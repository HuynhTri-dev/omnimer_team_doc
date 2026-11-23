# TÀI LIỆU GIẢI PHÁP KIẾN TRÚC (SOLUTION ARCHITECTURE)

**Hệ thống AI Agent (OPAgent) — Mô hình Python (LangGraph) & TypeScript Coarse-Grained Tools**

> [!NOTE]
> **Cập nhật kiến trúc (2026-11-23):** Một số quyết định hạ tầng và giao thức trong tài liệu gốc này đã được điều chỉnh qua bộ ADR tại [`02_opagent_technology_evaluation_adr.md`](../../../../../backend_otm/opagent_service/docs/architecture/02_opagent_technology_evaluation_adr.md): thay Qdrant bằng `pgvector` trong PostgreSQL Core, thay static header bằng Agent Delegation Token (`EdDSA`), dùng HTTP/1.1 Keep-Alive + SSE thay HTTP/2, và Core trực tiếp gửi thông báo Telegram. Phần thân tài liệu dưới đây sẽ được đồng bộ chi tiết ở Bước 11 của changelog.

---

## 1. Tổng quan & Nguyên tắc Thiết kế

Tài liệu này mô tả giải pháp kiến trúc tách biệt 2 dịch vụ (**OmniMer Core** bằng TypeScript và **OPAgent Service** bằng Python) nhằm phục vụ hệ sinh thái OmniMer Team và các hệ thống vệ tinh.

Kiến trúc tuân thủ 3 nguyên tắc nền tảng:

* **Tách biệt ranh giới dữ liệu (Database-per-Service):** OPAgent Service (Python) chạy ở trạng thái phi trạng thái (stateless) về mặt dữ liệu nghiệp vụ và không truy cập trực tiếp vào cơ sở dữ liệu của OmniMer Core. Mọi thao tác đọc/ghi nghiệp vụ đều đi qua lớp API nội bộ của TypeScript Core để bảo toàn quy tắc phân quyền (RBAC) và tính toàn vẹn dữ liệu.
* **Giao diện Tool dạng hạt thô (Coarse-Grained Remote Facade):** Dựa trên mẫu thiết kế *Remote Facade* (Martin Fowler, *Patterns of Enterprise Application Architecture*), các hàm nghiệp vụ bên trong TypeScript Core vẫn giữ dạng hạt mịn để tái sử dụng, nhưng lớp cổng phơi bày cho AI Agent phải gom nhiều truy vấn thành một điểm gọi duy nhất. Cách này giúp giảm số vòng lặp suy luận của LLM, hạ độ trễ đầu cuối và tiết kiệm token.
* **Ưu tiên mã nguồn tất định (Deterministic First):** Dựa trên hướng dẫn *Building Effective Agents* của Anthropic, các tác vụ có quy tắc rõ ràng (như quét hạn chót bằng SQL) được thực thi hoàn toàn bằng mã nguồn tất định tại Core Service; LLM chỉ được gọi ở các bước cần xử lý ngôn ngữ tự nhiên hoặc tổng hợp thông tin.

---

## 2. Kiến trúc Hệ thống Tổng thể

Sơ đồ dưới đây mô tả ranh giới giữa 2 dịch vụ, các kênh giao tiếp (đồng bộ HTTP/SSE và bất đồng bộ Redis Queue) và cách kết nối tới các hệ thống vệ tinh:

```mermaid
flowchart TB
    subgraph Clients["Tầng Giao diện & Kênh Tương tác"]
        UI["OmniMer Team UI<br/>(HTTP / WebSocket / SSE)"]
        SAT["Các Hệ thống Vệ tinh<br/>(HRM / EDU / App khác)"]
        TG["Telegram<br/>(DM Alert & Group Standup)"]
    end

    subgraph CoreService["Service 1: OmniMer Core (TypeScript - Node.js/Bun)"]
        GW["API Gateway & Auth<br/>(JWT / RBAC)"]
        WS["WebSocket Gateway<br/>(HITL Popup Confirm)"]
        CRON["Scheduler & Queue<br/>(BullMQ Cron 4h / 18:00)"]
        FACADE["Coarse-Grained Tool Facade<br/>(Internal REST / MCP Server)"]
        DOMAIN["Fine-Grained Domain Services<br/>(TaskService, FuzzySearch, UserRepo)"]
    end

    subgraph Infra["Hạ tầng Dữ liệu & Hàng đợi"]
        DB[("PostgreSQL<br/>(Task, draft_tasks, User)")]
        REDIS[("Redis<br/>(Queue, Cache, Idempotency)")]
    end

    subgraph AgentService["Service 2: OPAgent Service (Python - FastAPI + LangGraph)"]
        API["FastAPI Internal Router<br/>(OpenAPI Spec Auto-gen)"]
        GRAPH["LangGraph StateGraph<br/>(Orchestrator & Structured Output)"]
        TCLIENT["Async Tool Client<br/>(httpx HTTP/2 Keep-Alive / MCP)"]
        RAG["RAG & Heavy Processing<br/>(Qdrant / OCR / Voice STT)"]
    end

    subgraph External["External AI Providers"]
        LLM["LLM APIs<br/>(GPT-4 / Claude / Gemini)"]
    end

    UI <--> GW
    UI <--> WS
    SAT <--> GW
    CRON --> TG

    GW --> DOMAIN
    GW -- "HTTP/2 Sync (FR-AGT-001)" --> API
    CRON -- "Async Job (FR-AGT-002)" --> REDIS
    REDIS --> API

    FACADE --> DOMAIN
    DOMAIN <--> DB
    DOMAIN <--> REDIS
    DOMAIN --> WS

    API --> GRAPH
    GRAPH <--> LLM
    GRAPH <--> RAG
    GRAPH --> TCLIENT
    TCLIENT -- "Coarse-Grained Tool Call<br/>(X-User-Id, X-Internal-Secret)" --> FACADE

```

---

## 3. Luồng Xử lý Chi tiết (Sequence Diagrams)

### 3.1. Luồng FR-AGT-001: NLP Text/Voice-to-Task (Human-in-the-Loop)

Luồng này đảm bảo dữ liệu từ AI không bao giờ ghi thẳng vào bảng `Task` chính thức mà đi qua bảng tạm `draft_tasks` để Project Manager xác nhận qua WebSocket. Phần xử lý Fuzzy Search để ánh xạ tên người thực hiện sang `user_id` được giữ tại TypeScript Core để tối ưu tốc độ truy vấn cục bộ.

```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager (UI)
    participant Core as OmniMer Core (TypeScript)
    participant DB as PostgreSQL
    participant Agent as OPAgent (Python/LangGraph)
    participant LLM as LLM API (GPT-4/Claude)

    PM->>Core: Gửi câu lệnh tự nhiên ("Giao Lan làm báo cáo SEO, deadline chiều thứ 6")
    Core->>Core: Xác thực JWT & lấy current_datetime (UTC+7)
    Core->>Agent: POST /internal/v1/agent/extract-task (raw_input, current_datetime)
    Agent->>LLM: Gọi Structured Output theo JSON Schema
    LLM-->>Agent: JSON (task_title, description, assignee_name_hint, deadline_iso, extracted_subtasks)
    Agent-->>Core: Trả về JSON đã validate bằng Pydantic

    Core->>DB: Chạy Fuzzy Search map assignee_name_hint ("Lan") -> danh sách user_id
    DB-->>Core: Trả về danh sách ứng viên (VD: Lan Anh, Ngọc Lan)
    Core->>DB: INSERT bản nháp vào bảng tạm draft_tasks
    Core-->>PM: Push WebSocket (Popup Confirm + Dropdown chọn Assignee nếu trùng tên)

    PM->>Core: Xác nhận / Chỉnh sửa & nhấn "Tạo chính thức"
    Core->>DB: Chuyển dữ liệu từ draft_tasks -> INSERT bảng Task chính thức
    Core-->>PM: Trả về trạng thái thành công

```

### 3.2. Luồng FR-AGT-002: Proactive Health Monitor (Tách bạch SQL & AI)

Theo đặc tả kỹ thuật, quy tắc phát hiện rủi ro mỗi 4 giờ hoàn toàn có thể giải quyết bằng truy vấn SQL thuần mà không cần tốn chi phí LLM. Chỉ bước tổng hợp Standup lúc 18:00 hàng ngày mới cần gọi sang OPAgent Service để sinh văn bản tóm tắt.

```mermaid
sequenceDiagram
    autonumber
    participant Cron as BullMQ Scheduler (TS Core)
    participant DB as PostgreSQL
    participant Redis as Redis (Idempotency & Queue)
    participant Agent as OPAgent Worker (Python)
    participant LLM as LLM API
    participant TG as Telegram Bot API

    rect rgb(30, 64, 175, 0.08)
        Note over Cron, TG: Nhánh 1: BL-AGT-002.1 — Risk Detection (Chạy mỗi 4 giờ — Thuần SQL)
        Cron->>DB: Query Task (Status != Done AND DueDate - Now < 48h AND Cột đầu tiên)
        DB-->>Cron: Danh sách Task rủi ro + Telegram ID của Assignee
        Cron->>Redis: Kiểm tra key idempotency (alert:{task_id}) tránh spam
        Redis-->>Cron: Chưa cảnh báo trong 12h qua (Set TTL 12h)
        Cron->>TG: Gửi cảnh báo trực tiếp qua Telegram DM cho Assignee
    end

    rect rgb(16, 185, 129, 0.08)
        Note over Cron, TG: Nhánh 2: BL-AGT-002.2 — Standup Aggregation (18:00 hàng ngày)
        Cron->>DB: Aggregate 1 lần (Done X tasks, Đang làm Y tasks, Bị Block Z tasks)
        DB-->>Cron: Payload số liệu đã gom gọn (Coarse-grained)
        Cron->>Redis: Đẩy job vào Queue (kèm key standup:{project_id}:{date})
        Redis->>Agent: Worker Python nhận job
        Agent->>LLM: Tạo bản tóm tắt Standup tự nhiên từ payload số liệu
        LLM-->>Agent: Văn bản tóm tắt Standup
        Agent->>TG: Gửi báo cáo vào Group Telegram của Project
    end

```

### 3.3. Luồng Mở rộng: Agent Đa bước gọi Coarse-Grained TS Tools

Khi hệ thống mở rộng sang các bài toán Agent hỏi đáp đa bước trên nhiều hệ thống vệ tinh quanh OmniMer Team, LangGraph sẽ gọi các Tool hạt thô bên TypeScript Core:

```mermaid
sequenceDiagram
    autonumber
    participant Graph as LangGraph Agent (Python)
    participant Facade as Tool Facade (TS Core)
    participant S1 as ProjectService (Hạt mịn)
    participant S2 as TaskService (Hạt mịn)
    participant S3 as MemberService (Hạt mịn)

    Graph->>Facade: POST /internal/tools/get-project-health-snapshot<br/>(Headers: X-User-Id, X-Internal-Secret)
    Facade->>Facade: Kiểm tra quyền RBAC của X-User-Id trên project_id
    par Thực thi nội bộ song song (Promise.all)
        Facade->>S1: getProjectMeta(project_id)
        Facade->>S2: getTaskMetricsAndBlockers(project_id)
        Facade->>S3: getActiveMembersWorkload(project_id)
    end
    S1-->>Facade: Meta
    S2-->>Facade: Metrics & Blockers
    S3-->>Facade: Workload
    Facade->>Facade: Nén và lọc bỏ các trường thừa (tối ưu Context Window)
    Facade-->>Graph: Trả về 1 cục JSON tổng hợp duy nhất

```

---

## 4. Quy chuẩn Thiết kế Tool Dạng Hạt Thô & Hợp đồng API

### 4.1. Tiêu chuẩn phân tách Hạt mịn vs. Hạt thô

| Tiêu chí | Hạt mịn (Chỉ dùng nội bộ TS Core) | Hạt thô (Bắt buộc cho AI Tool Facade) |
| --- | --- | --- |
| **Mục đích** | Tái sử dụng cho CRUD, đơn vị nghiệp vụ nhỏ (`getUser`, `getTask`). | Phục vụ trọn vẹn một ý định/ngữ cảnh của Agent (`getDailyStandupPayload`, `resolveAssigneeCandidates`). |
| **Số lần gọi từ LangGraph** | Nhiều lần nối tiếp $\rightarrow$ phát sinh nhiều vòng lặp suy luận LLM. | 1 lần gọi duy nhất $\rightarrow$ giảm độ trễ từ nhiều giây xuống còn 1 bước suy luận. |
| **Cấu trúc JSON trả về** | Trả toàn bộ cột trong DB (gây nhiễu và tốn Token). | Đã qua lớp DTO lược bỏ metadata thừa (như `created_by_ip`, `updated_at` không cần thiết). |

### 4.2. Đồng bộ Kiểu dữ liệu Tự động (Contract-First)

Để loại bỏ rủi ro lệch schema giữa 2 ngôn ngữ (Python và TypeScript):

1. **Chiều Python $\rightarrow$ TypeScript:** FastAPI tự động sinh file `openapi.json` từ các Pydantic Model (ví dụ schema đầu ra của FR-AGT-001 gồm `task_title`, `description`, `assignee_name_hint`, `deadline_iso`, `extracted_subtasks`). TypeScript chạy lệnh `openapi-typescript` trong pipeline CI/CD để tự động sinh file `schema.d.ts` và HTTP Client có kiểm tra kiểu lúc biên dịch.


2. **Chiều TypeScript $\rightarrow$ Python:** Lớp Tool Facade bên TypeScript sử dụng **Zod** để định nghĩa Input/Output và xuất ra chuẩn **JSON Schema** (hoặc chạy dưới dạng **MCP Server** sử dụng `@modelcontextprotocol/sdk`). LangGraph phía Python nạp trực tiếp danh sách Tool thông qua `langchain-mcp-adapters` mà không cần viết tay lại định nghĩa tool.

---

## 5. Kiến trúc Triển khai, Bảo mật & Chốt chặn Vận hành

### 5.1. Tối ưu Hiệu năng Mạng nội bộ

* **Triển khai cùng mạng nội bộ (Private Docker Network / VPC):** Giữ độ trễ truyền tin giữa 2 service ở mức 1–5ms.
* **Connection Pooling (HTTP/2 Keep-Alive):** Phía Python khởi tạo duy nhất một instance `httpx.AsyncClient(http2=True)` trong vòng đời `lifespan` của FastAPI để tái sử dụng kết nối TCP khi gọi sang TypeScript Tools.
* **Chống nghẽn tài nguyên Python:** Chạy FastAPI với nhiều worker (Uvicorn/Gunicorn) và tách riêng các tác vụ nặng về CPU (như OCR tài liệu hoặc nạp embedding hàng loạt vào Vector DB) sang hàng đợi Redis Worker độc lập với API phục vụ người dùng trực tiếp.

### 5.2. Bảo mật & Chốt chặn An toàn (Guardrails)

* **Zero-Trust đối với tham số từ LLM:** Các thông tin định danh bảo mật (`user_id`, `team_id`, `role`) tuyệt đối không đưa vào schema để LLM tự điền. OmniMer Core trích xuất từ JWT và truyền qua Header nội bộ (`X-User-Id`, `X-Team-Id`, `X-Internal-Secret`). Khi LangGraph gọi ngược Tool sang TS Core, nó bắt buộc đính kèm lại đúng các header này để TS Core kiểm tra RBAC.
* **Giới hạn vòng lặp & Thời gian chờ:**
* Cấu hình `recursion_limit` trong LangGraph (tối đa 5–8 bước) để tránh vòng lặp gọi tool vô hạn.
* Cấu hình Timeout (10–15 giây) và **Circuit Breaker** phía TS Core khi gọi sang Python. Với FR-AGT-001, nếu OPAgent Service quá tải hoặc LLM API lỗi, hệ thống lập tức trả về fallback cho phép PM nhập tay trên form tạo task.


* **Chống gửi lặp thông báo (Idempotency):** Mọi tác vụ định kỳ trong FR-AGT-002 (cảnh báo DM mỗi 4 giờ và Standup lúc 18:00) đều phải kiểm tra khóa định danh trên Redis (`alert:{task_id}` với TTL 12h và `standup:{project_id}:{YYYY-MM-DD}`) để đảm bảo không spam Telegram khi worker thực hiện retry.



---

## 6. Lộ trình Triển khai Đề xuất

1. **Giai đoạn 1 (Nền tảng & FR-AGT-001):** Dựng khung FastAPI + LangGraph (Structured Output) và kết nối đồng bộ với TS Core. Thiết lập cơ chế tự sinh Type (`openapi-typescript`), bảng `draft_tasks`, Fuzzy Search ánh xạ `user_id` và luồng xác nhận WebSocket.


2. **Giai đoạn 2 (Tự động hóa & FR-AGT-002):** Triển khai BullMQ Cron Job tại TS Core cho nhánh quét rủi ro 4 giờ/lần (thuần SQL + Telegram DM) và kết nối Redis Queue đẩy payload tổng hợp lúc 18:00 sang Worker Python để tóm tắt Standup.


3. **Giai đoạn 3 (Mở rộng Đa hệ thống):** Chuẩn hóa lớp `Coarse-Grained Tool Facade` theo chuẩn MCP (Model Context Protocol) để các ứng dụng vệ tinh khác trong hệ sinh thái OmniMer có thể đăng ký Tool mới vào OPAgent mà không phải chỉnh sửa mã nguồn cốt lõi của Python Service.

---

## 7. Tài liệu Tham khảo

* Martin Fowler (2002), *Patterns of Enterprise Application Architecture* — Remote Facade Pattern (Nguyên tắc thiết kế giao diện hạt thô qua biên mạng).
* Anthropic Engineering (2024), *Building Effective Agents* — Phân tách Deterministic Workflows vs. Autonomous Agents và thiết kế Agent-Computer Interface (ACI) cho Tool.
* FastAPI & OpenAPI Specification — Cơ chế tự động sinh hợp đồng API từ Pydantic Models.
* Model Context Protocol (MCP) Specification — Chuẩn kết nối công cụ động giữa AI Agent và hệ thống lõi.