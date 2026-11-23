<!--
  name: sa.md
  description: Solution Architecture specification for OPAgent Multi-Agent Ecosystem, defining Dual-Service boundaries (TypeScript project_core PEP/PDP + Python opagent_service Reasoning Engine), HTTP/1.1 Keep-Alive + SSE protocol, Agent Delegation Token (EdDSA Ed25519), pgvector vector(1024) RAG, 20-tool catalog, PostgreSQL opagent_runtime checkpointer, and Google AI Studio / OpenRouter / AWS Bedrock serving strategy.
-->

# TÀI LIỆU GIẢI PHÁP KIẾN TRÚC (SOLUTION ARCHITECTURE)

**Hệ sinh thái AI Agent (OPAgent) — Mô hình Dual-Service: TypeScript Core (System of Record & PEP/PDP) & Python OPAgent (LangGraph Reasoning Engine)**

> [!NOTE]
> **Phiên bản kiến trúc:** `v2.0.0` (Đồng bộ toàn diện theo bộ ADR [`02_opagent_technology_evaluation_adr.md`](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/02_opagent_technology_evaluation_adr.md), Kiến trúc mã nguồn [`01_opagent_source_architecture.md`](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/01_opagent_source_architecture.md) và Kiến trúc An toàn Thông tin [`03_opagent_security_authorization_architecture.md`](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/03_opagent_security_authorization_architecture.md)).
> *Quyết định kỹ thuật cốt lõi:* Tích hợp `pgvector (vector(1024))` ngay trong Neon PostgreSQL Core thay cho Qdrant độc lập; xác thực liên dịch vụ bằng Agent Delegation Token (`EdDSA Ed25519`); truyền thông bằng HTTP/1.1 Keep-Alive + SSE; lưu checkpoint LangGraph tại schema cô lập `opagent_runtime` (role `opagent_svc`); toàn bộ thông báo kênh ngoài (Telegram/Slack/Zalo OA) do Core trực tiếp phát; và tầng phục vụ Model chuyển đổi linh hoạt `Google AI Studio` (1500 RPD) $\rightarrow$ `OpenRouter` $\rightarrow$ `AWS Bedrock`.

---

## 1. Tổng quan & Nguyên tắc Thiết kế

Tài liệu này mô tả giải pháp kiến trúc tách biệt 2 dịch vụ (**OmniMer Core** bằng TypeScript và **OPAgent Service** bằng Python) nhằm phục vụ hệ sinh thái OmniMer Team và các hệ thống vệ tinh:

1. **Service 1 — OmniMer Core (`backend_otm/project_core` — TypeScript / Express 5 / Prisma):**
   - Đóng vai trò **System of Record duy nhất** và là **Policy Enforcement Point (PEP) / Policy Decision Point (PDP)** duy nhất của toàn hệ thống.
   - Sở hữu cơ sở dữ liệu chính (PostgreSQL schema `omnimer`), quản lý toàn bộ phân quyền (RBAC/ABAC), bảo mật PII lớp 1, cổng kiểm duyệt Strict DoD (`FR-PRJ-008`), Anti-Gaming (`FR-PRJ-007.4`), kho tri thức `pgvector` (`agent_knowledge_chunks`), các bảng nháp HITL (`draft_tasks`, `agent_action_drafts`, `setup_blueprint_drafts`), và trực tiếp gửi thông báo kênh ngoài tới Telegram, Slack, Zalo OA.
2. **Service 2 — OPAgent Service (`backend_otm/opagent_service` — Python 3.14 + FastAPI + LangGraph):**
   - Đóng vai trò **Reasoning Engine thuần túy**, chạy dưới dạng Render Private Service (`pserv`) tại region Singapore, không mở public URL ra Internet.
   - Hoàn toàn **stateless** đối với dữ liệu nghiệp vụ: không có kết nối tới schema `omnimer`, chỉ kết nối tới schema cô lập `opagent_runtime` qua role chuyên biệt `opagent_svc` (đã bị revoke toàn bộ quyền trên `omnimer` và `public`) để lưu trữ LangGraph checkpoint ngắn hạn (tối đa 24h, xóa khi có dữ liệu `> INTERNAL`).
   - Tách biệt **6 Single-Shot Chains** (không checkpoint: trích xuất task, phân loại triage, tìm ứng viên dispatch, tóm tắt standup, kiểm tra release, tóm tắt ca trực) và **4 LangGraph StateGraphs** (`support_copilot`, `pm_copilot`, `setup_wizard`, `asset_guardian` với `recursion_limit = 6..8`).

### Ba nguyên tắc nền tảng:
* **Tách biệt ranh giới dữ liệu & Không có quyền ghi trực tiếp (Zero Direct Commit):** Toàn bộ 20 công cụ của Agent chỉ thuộc 2 nhóm `READ` và `STAGE`. Agent không bao giờ có quyền tự ý `COMMIT` thay đổi vào các bảng nghiệp vụ chính (`tasks`, `users`, `projects`). Mọi thao tác ghi đều phải dừng lại ở bảng nháp Staging (`draft_tasks`, `agent_action_drafts`) để người dùng xác nhận (Human-in-the-Loop) hoặc kích hoạt qua rule tất định phía Core.
* **Giao diện Tool dạng hạt thô (Coarse-Grained Remote Facade):** Dựa trên mẫu thiết kế *Remote Facade* (Martin Fowler), các API nội bộ phía TypeScript Core gom nhiều truy vấn phức tạp (Workload, Skills, Ownership, Leave) thành 1 lần gọi duy nhất (`POST /internal/v1/agent-tools/*`), giúp giảm số vòng lặp suy luận của LLM, hạ độ trễ và tiết kiệm token.
* **Ưu tiên mã nguồn tất định (Deterministic First):** Các tác vụ có quy tắc rõ ràng (như quét hạn chót 4h bằng SQL, kiểm tra cửa sổ 48h Zalo OA, tính điểm điều phối Dev `40/35/25`, tính SPI/EVM Sprint, và phản hồi ngoài giờ bằng mẫu đã duyệt `ApprovedAutoReplyTemplate`) được thực thi 100% bằng mã nguồn tất định tại Core Service; LLM chỉ được gọi khi thực sự cần xử lý ngôn ngữ tự nhiên.

---

## 2. Kiến trúc Hệ thống Tổng thể

```mermaid
flowchart TB
    subgraph Clients["Tầng Giao Diện & Kênh Tương Tác"]
        UI["OmniMer Web UI / Cmd+K<br/>(HTTP / WebSocket HITL Cards)"]
        SAT["Hệ thống Vệ tinh<br/>(HRM / EDU / Partner Apps)"]
        EXT_CHANNELS["Telegram / Slack / Zalo OA<br/>(Unified Inbox & Webhook Gateway)"]
    end

    subgraph CoreService["Service 1: OmniMer Core (TypeScript — Node.js Express 5)"]
        PEP_PDP["Security Gatekeeper, PEP/PDP<br/>(JWT Auth, WorkspaceContext, RBAC, PII Masking)"]
        ADT_GEN["Agent Delegation Token Issuer<br/>(EdDSA Ed25519 kid_v1, scp[], fr, jti, exp)"]
        CRON["Advisory Lock Schedulers<br/>(DeadlineScannerCron, ProactiveHealthMonitorCron)"]
        FACADE["20 Coarse-Grained Agent Tools<br/>(Zod .strict(), Strict DoD, Anti-Gaming, HITL Staging)"]
        NOTIFIER["Centralized Channel Notifier<br/>(Telegram, Slack Webhook, Zalo ZNS)"]
        CORE_REPOS["Repositories & Knowledge Services<br/>(DraftTaskRepo, KnowledgeIngestionService)"]
    end

    subgraph Infra["Hạ Tầng Dữ Liệu Neon PostgreSQL & Redis"]
        DB_OMNIMER[("Neon PostgreSQL: schema 'omnimer'<br/>• tasks, users, projects, draft_tasks<br/>• agent_knowledge_chunks (vector 1024 + HNSW)<br/>• approved_auto_reply_templates")]
        DB_RUNTIME[("Neon PostgreSQL: schema 'opagent_runtime'<br/>• langgraph_checkpoints (TTL 24h)<br/>• role 'opagent_svc' ONLY")]
        REDIS[("Redis Cache & Rate Limiting<br/>• Rate limit 20r/10w, Idempotency<br/>• Daily Workspace Budget opa:budget:*<br/>• ADT JTI Revocation blacklist")]
    end

    subgraph AgentService["Service 2: OPAgent Service (Python 3.14 — FastAPI + LangGraph)"]
        SEC_MIDDLEWARE["Security Guard & ADT Verification<br/>(api/deps.py: Ed25519 Public Key, scp[], fr check)"]
        SINGLE_CHAINS["6 Single-Shot Chains<br/>(no checkpointer: task-extract, triage, dispatch)"]
        LANG_GRAPHS["4 LangGraph StateGraphs<br/>(support_copilot, pm_copilot, setup_wizard, asset_guardian)"]
        TOOL_CALLER["Async Tool Client<br/>(httpx HTTP/1.1 Pool + Keep-Alive + ADT Token)"]
        LLM_ADAPTER["LLM Provider Strategy Factory<br/>(Google AI Studio / OpenRouter / Ollama / Bedrock)"]
    end

    subgraph ExternalLLM["Tầng Model Serving Ngoài (Model Inference Layer)"]
        GOOGLE_AI["Google AI Studio (1500 RPD Free Tier)<br/>• Router: gemini-3.5-flash-lite<br/>• Reasoning: gemini-3.5-flash"]
        OPENROUTER["OpenRouter (Dev/Test Fallback)<br/>• google/gemma-4-26b-a4b-it:free<br/>• google/gemma-4-31b-it:free"]
        OLLAMA_CLOUD["Ollama Cloud (Staging/Prod Option)<br/>• gemma4:31b-cloud (Text + Image)"]
        AWS_BEDROCK["AWS Bedrock ap-southeast-1 (Enterprise)<br/>• Claude 3.5 Haiku / Sonnet + Titan Embed v2"]
    end

    UI <--> PEP_PDP
    SAT <--> PEP_PDP
    EXT_CHANNELS <--> PEP_PDP
    CRON --> NOTIFIER --> EXT_CHANNELS

    PEP_PDP --> ADT_GEN
    PEP_PDP -- "HTTP/1.1 Keep-Alive / SSE + ADT" --> SEC_MIDDLEWARE
    SEC_MIDDLEWARE --> SINGLE_CHAINS
    SEC_MIDDLEWARE --> LANG_GRAPHS
    LANG_GRAPHS <--> DB_RUNTIME

    LANG_GRAPHS --> TOOL_CALLER
    TOOL_CALLER -- "POST /internal/v1/agent-tools/*<br/>(Bearer ADT + X-Agent-Service-Key)" --> FACADE
    FACADE --> CORE_REPOS
    CORE_REPOS <--> DB_OMNIMER
    PEP_PDP <--> REDIS

    SINGLE_CHAINS <--> LLM_ADAPTER
    LANG_GRAPHS <--> LLM_ADAPTER
    LLM_ADAPTER --> GOOGLE_AI
    LLM_ADAPTER --> OPENROUTER
    LLM_ADAPTER --> OLLAMA_CLOUD
    LLM_ADAPTER --> AWS_BEDROCK
```

---

## 3. Luồng Xử lý Trọng yếu (Sequence Diagrams)

### 3.1. Luồng FR-AGT-001: NLP Text/Voice-to-Task (Human-in-the-Loop)

Toàn bộ thông tin từ AI chỉ được bóc tách và ghi tạm vào `draft_tasks`. Không bao giờ ghi trực tiếp vào bảng `tasks` trước khi PM bấm xác nhận trên giao diện.

```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager (Web UI)
    participant Core as OmniMer Core (TypeScript)
    participant DB as Neon DB (schema omnimer)
    participant Agent as OPAgent Service (FastAPI)
    participant LLM as Google AI Studio (gemini-3.5-flash-lite)

    PM->>Core: Gửi câu lệnh tiếng Việt ("Giao Lan làm báo cáo tiến độ tuần, hạn chót 17h thứ 6 tuần này")
    Core->>Core: Xác thực JWT, kiểm tra quyền Workspace & múi giờ UTC+7
    Core->>Core: Che PII lớp 1 (maskVietnamesePii) -> sinh ADT Token (EdDSA, fr="FR-AGT-001")
    Core->>Agent: POST /internal/v1/chains/task-extraction (masked_text, current_datetime_utc7)<br/>Header: Authorization: Bearer <ADT>, X-Agent-Service-Key
    Agent->>Agent: Verify chữ ký Ed25519, kiểm tra fr claim & pii_guard
    Agent->>LLM: Gọi chat/completions (OpenAI format, temperature 0, response_format: json_object)
    LLM-->>Agent: JSON: { task_title, assignee_name_hint: "Lan", deadline_iso: "2026-11-27T17:00:00+07:00" }
    Agent-->>Core: Trả JSON Structured Extraction đã validate qua Pydantic (< 3s)
    
    Core->>DB: Fuzzy Search tìm nhân sự khớp "Lan" (Lan Anh, Ngọc Lan) theo match_score
    DB-->>Core: Danh sách ứng viên nhân sự
    Core->>DB: INSERT bản nháp vào draft_tasks (ai_status = DRAFT_READY)
    Core-->>PM: Gửi WebSocket sự kiện draft_tasks.ready (kèm Popup Confirm + Dropdown chọn người)

    PM->>Core: Bấm Confirm (chọn Ngọc Lan & bấm Lưu)
    Core->>DB: Transaction: Đổi draft_tasks -> INSERT tasks chính thức
    Core-->>PM: Phản hồi tạo Task thành công
```

### 3.2. Luồng FR-AGT-002: Proactive Health Monitor (100% Core Sở Hữu Kênh Ngoại)

Tuân thủ nguyên tắc **Core gửi mọi thông báo kênh ngoài**: OPAgent không nắm giữ Telegram Bot Token, không gửi tin nhắn trực tiếp ra bên ngoài.

```mermaid
sequenceDiagram
    autonumber
    participant Cron as ProactiveHealthMonitorCron (Core)
    participant DB as Neon DB (schema omnimer)
    participant Redis as Redis (Idempotency)
    participant Agent as OPAgent Service (FastAPI)
    participant LLM as LLM Inference
    participant TG as Telegram Bot API

    rect rgb(30, 64, 175, 0.08)
        Note over Cron, TG: Nhánh 1: Cảnh báo rủi ro 4 giờ & SLA 48h (Thuần SQL — Zero-LLM)
        Cron->>DB: Truy vấn Task (status != DONE AND due_date - now < 48h AND total_float == 0)
        DB-->>Cron: Danh sách Task đường găng rủi ro + Telegram Chat ID của Assignee
        Cron->>Redis: Kiểm tra khóa chống spam (alert:{task_id}, TTL 12h)
        Redis-->>Cron: Khóa hợp lệ (SET NX TTL 12h)
        Cron->>TG: Core trực tiếp gửi tin nhắn DM cảnh báo cho Assignee
    end

    rect rgb(16, 185, 129, 0.08)
        Note over Cron, TG: Nhánh 2: Tổng hợp Daily Standup lúc 18:00
        Cron->>DB: Gom số liệu Standup trong ngày (Done X, In-Progress Y, Blocked Z)
        DB-->>Cron: Coarse-grained payload số liệu dự án
        Cron->>Redis: Kiểm tra khóa standup:{project_id}:{YYYY-MM-DD}
        Redis-->>Cron: Chưa gửi trong ngày
        Cron->>Agent: POST /internal/v1/chains/standup-summary (kèm ADT Token)<br/>Gọi đồng bộ lấy văn bản tóm tắt
        Agent->>LLM: Sinh văn bản tóm tắt ngôn ngữ tự nhiên từ số liệu
        LLM-->>Agent: Nội dung tóm tắt hoàn chỉnh
        Agent-->>Cron: Trả chuỗi text tóm tắt về Core
        Cron->>TG: Core trực tiếp gửi báo cáo vào Group Telegram của Dự án
    end
```

### 3.3. Luồng Agent Đa bước gọi 20 Coarse-Grained Tools qua ADT Token

```mermaid
sequenceDiagram
    autonumber
    participant Graph as LangGraph Node (Python)
    participant Client as Tool Client (httpx HTTP/1.1 Pool)
    participant CoreGuard as agentDelegation.middleware.ts (Core)
    participant Facade as OpAgentToolFacadeRepo (Core)
    participant DB as Neon DB (schema omnimer)

    Graph->>Client: Cần dữ liệu: Gọi tool query_tasks_by_nl_filters
    Client->>CoreGuard: POST /internal/v1/agent-tools/query-tasks-by-nl-filters<br/>Header: Authorization: Bearer <ADT>, X-Agent-Service-Key
    CoreGuard->>CoreGuard: 1. Kiểm tra X-Agent-Service-Key bằng timingSafeEqual<br/>2. Verify chữ ký Ed25519 (kid_v1, alg=EdDSA)<br/>3. Kiểm tra jti trên Redis (chống replay)<br/>4. Kiểm tra scp[] chứa "query_tasks_by_nl_filters"<br/>5. Kiểm tra fr claim khớp với luồng gọi<br/>6. Truy vấn live WorkspaceMember từ DB (chống stale token)
    CoreGuard->>Facade: Ủy quyền thực thi (tiêm req.workspaceContext an toàn)
    Facade->>DB: Thực thi SQL tham số hóa trong phạm vi allowed_project_ids
    DB-->>Facade: Kết quả thô
    Facade->>Facade: Lọc bỏ metadata thừa (nén context window)
    Facade-->>Client: Trả về JSON kết quả hạt thô (< 100ms)
    Client-->>Graph: LangGraph Node nhận kết quả và tiếp tục suy luận
```

---

## 4. Danh mục 20 Agent Tools & Chuẩn Hóa Hợp Đồng Dữ Liệu

### 4.1. Quy chuẩn Phân Loại Tool (READ & STAGE ONLY — 0 COMMIT)

Hệ thống rút hoàn toàn quyền `COMMIT` của LLM (`commit_task_assignment` đã bị loại bỏ) để triệt tiêu lỗ hổng **Excessive Agency (OWASP LLM06)**:
* **Tool loại `READ` (13 tools):** Đọc số liệu tổng hợp, tra cứu danh bạ, tìm kiếm vector RAG, kiểm tra DoD, đánh giá EVM/SPI.
* **Tool loại `STAGE` (7 tools):** Ghi tạm vào các bảng nháp Staging (`draft_tasks`, `agent_action_drafts`, `setup_blueprint_drafts`) có kiểm tra Anti-Gaming (`FR-PRJ-007.4`), khóa Strict DoD (`FR-PRJ-008`) và thời gian sống ngắn (TTL 15 phút, OCC versioning).

### 4.2. Cơ chế Phản hồi Ngoài giờ Tất định (After-Hours Template-Based Auto-Reply)
Đối với `FR-AGT-003` (Support Bot):
* **Trong giờ làm việc:** Bot chỉ gợi ý câu trả lời để nhân viên CSKH duyệt trước khi gửi khách.
* **Ngoài giờ làm việc:** Khi Workspace bật cờ `after_hours_auto_reply_enabled = true`:
  - LLM chỉ được phép phân loại `intent`, đánh giá `severity` và chọn `matched_template_id`.
  - Core kiểm tra điều kiện khắt khe: `intent === 'FAQ'`, `severity` không phải `P1/P2`, và độ tin cậy $\ge$ ngưỡng cấu hình.
  - Core trích xuất nguyên văn nội dung từ bảng `ApprovedAutoReplyTemplate` (`status = APPROVED`) và gửi cho khách hàng.
  - **Tuyệt đối không gửi bất kỳ câu chữ tự do nào do LLM tự sinh ra.**
  - Nếu là sự cố `P1_CRITICAL`, hệ thống chuyển ngay sang luồng cảnh báo khẩn cấp (Emergency Alert) tới trực ban.

### 4.3. Quy trình Đồng bộ Hợp đồng Hai Chiều (Contract-First Pipeline)
1. **TypeScript $\rightarrow$ Python:** Toàn bộ Input/Output DTO của 20 Tools được định nghĩa bằng **Zod `.strict()`** tại Core, xuất ra `/manifest`. Script `sync_tool_contracts.py` tự động chuyển đổi sang các Pydantic model trong `_generated.py` tại `opagent_service`.
2. **Python $\rightarrow$ TypeScript:** FastAPI tự động xuất `openapi.json`. Core chạy `openapi-typescript` để sinh kiểu dữ liệu `opagent.d.ts` đồng bộ 100%.

---

## 5. Kiến trúc Bảo mật & Chốt chặn Vận hành (Security & Guardrails)

### 5.1. Ma trận Phòng Thủ STRIDE & OWASP LLM Top 10 (2025)
Hệ thống triển khai 16 chốt chặn kiểm thử an toàn thông tin chuyên sâu (**`SEC-AGT-01..16`**):
* **Bảo vệ danh tính & ủy quyền:** Bắt buộc ADT token ký `Ed25519 (kid_v1)`, kiểm tra danh sách quyền `scp[]`, kiểm tra claim `fr`, kiểm tra trạng thái hoạt động thực tế của người dùng trực tiếp từ DB tại thời điểm gọi.
* **Cô lập RAG đa tầng & Zero Drift:** Lưu vector 1024 chiều ngay trong bảng `agent_knowledge_chunks` của Neon DB; lọc cứng 2 tầng `classification = 'INTERNAL'` và `faq_eligible = true`; khi tài liệu bị hạ độ mật hoặc gỡ cờ FAQ, toàn bộ vector chunk bị xóa **ngay trong cùng một DB transaction** với sự kiện đổi độ mật, đảm bảo 0ms drift window.
* **Bảo mật PII 2 lớp:** Lớp 1 tại Core (`maskVietnamesePii` bằng Regex tất định che SĐT, Email, CCCD, STK ngân hàng trước khi gửi sang Python); Lớp 2 tại Python (`pii_guard.py` kiểm tra lại trước khi gửi External LLM).
* **Kiểm soát vòng lặp & Ngân sách:** LangGraph đặt giới hạn `recursion_limit = 6..8`; Redis kiểm soát ngân sách token hàng ngày của Workspace (`opa:budget:{workspace_id}:{date}`); Circuit Breaker ngắt tự động sau 5s/15s.

### 5.2. Chiến lược Phục Vụ Model Đa Tầng (ADR-OPA-007)
* **Môi trường Dev / Test (Google AI Studio [Chính] & OpenRouter [Dự phòng]):**
  - **Google AI Studio (`LLM_PROVIDER=google_ai_studio`):** Kết nối chuẩn OpenAI-compatible tới `https://generativelanguage.googleapis.com/v1beta/openai` với hạn mức **1,500 requests/ngày** miễn phí. Router Model: `gemini-3.5-flash-lite` (~2.1s); Reasoning Model: `gemini-3.5-flash` (~3.6s).
  - **OpenRouter (`LLM_PROVIDER=openrouter`):** Phục vụ test các model mã nguồn mở `google/gemma-4-26b-a4b-it:free` và `google/gemma-4-31b-it:free` (hoặc kết nối qua Google BYOK).
* **Môi trường Staging / Prod Option:** Sẵn sàng kết nối **Ollama Cloud** (`gemma4:31b-cloud`) qua `langchain-ollama`.
* **Môi trường Enterprise Scale (AWS Bedrock):** Kết nối cụm dịch vụ tại **Singapore (`ap-southeast-1`)** qua VPC Endpoint, xác thực bằng **IAM Role / IRSA**, sử dụng `amazon.titan-embed-text-v2:0` (`1024-d`) khớp chuẩn 100% với schema `vector(1024)` của Neon PostgreSQL mà không cần chỉnh sửa database.

---

## 6. Lộ trình Triển khai Kế thừa & Tài liệu Liên quan

* **Giai đoạn 0: Nền tảng Bảo mật & Hạ tầng** *(Đã hoàn tất Bước 0.1 & Bước 0.2 — Neon pgvector 0.8.7, role `opagent_svc`, cặp khóa `Ed25519 kid_v1`, Render Environment Groups, benchmark LLM PASS)*.
* **Giai đoạn I: Nền tảng Dữ liệu Core, ADT Security Guard & Khởi tạo `opagent_service`** *(Bước 1 $\rightarrow$ Bước 4)*.
* **Giai đoạn II: Triển khai 7 Bộ Tính năng & 4 StateGraphs** *(Bước 5 $\rightarrow$ Bước 8)*.
* **Giai đoạn III: Kiểm thử Tải, 16 SEC-AGT Security Gate & Tích hợp UI** *(Bước 9 $\rightarrow$ Bước 11)*.

👉 **Các tài liệu kỹ thuật liên kết trực tiếp:**
* [01_opagent_source_architecture.md](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/01_opagent_source_architecture.md)
* [02_opagent_technology_evaluation_adr.md](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/02_opagent_technology_evaluation_adr.md)
* [03_opagent_security_authorization_architecture.md](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/architecture/03_opagent_security_authorization_architecture.md)
* [09_opagent_multi_agent_ecosystem_changelog.md](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/opagent_service/docs/change_log/09_opagent_multi_agent_ecosystem_changelog.md)
* [01_SRS_OPAGENT_MULTI_AGENT_ECOSYSTEM.md](./01_SRS_OPAGENT_MULTI_AGENT_ECOSYSTEM.md)
* [02_RTM_OPAGENT_MULTI_AGENT_ECOSYSTEM.csv](./02_RTM_OPAGENT_MULTI_AGENT_ECOSYSTEM.csv)
