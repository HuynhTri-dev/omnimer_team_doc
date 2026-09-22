# Software Requirements Specification (SRS) - OmniProject Core Phase 2
> **Reference Standard:** IEEE Std 830-1998  
> **Project:** OmniProject Core  
> **Phase:** Phase 2 (Dynamic Workflows, Custom Fields, Subtasks, Lifecycle & Templates)

---

## 1. Introduction

### 1.1 Purpose
Tài liệu SRS này đặc tả toàn bộ các yêu cầu phần mềm, yêu cầu phi chức năng (NFR), cấu trúc dữ liệu và các luồng logic chi tiết (Sequence Diagrams) cho Phase 2 của hệ thống OmniProject Core. Tài liệu này đóng vai trò là "Design Contract" chuẩn mực kỹ thuật giữa Đội ngũ Phân tích Nghiệp vụ (BA), Kỹ sư Phát triển (Backend & Frontend), và Đội ngũ Kiểm thử Chất lượng (QA/QC).

### 1.2 Product Scope
Phạm vi Phase 2 tập trung cung cấp năng lực quản trị linh hoạt và tự động hóa cho Project Manager (PM) và các thành viên trong dự án:
- **FR-PRJ-002 (Dynamic Workflows & WIP Limits):** Cấu hình luồng trạng thái tùy chỉnh đa dạng cho từng dự án, cơ chế Meta-Status phục vụ báo cáo liên phòng ban, kiểm soát tắc nghẽn bằng giới hạn WIP (Work In Progress), quy trình bàn giao (Handoff), và quản lý điểm nghẽn qua cờ Blocked.
- **FR-PRJ-004 (Dynamic Custom Fields):** Mở rộng thuộc tính dữ liệu nghiệp vụ của Task tùy biến theo dự án mà không cần thay đổi source code, hỗ trợ ép kiểu an toàn (Type Safety), kiểm tra dữ liệu động và bộ lọc (Filter/Sort).
- **FR-PRJ-006 (Subtask Management):** Phân rã khối lượng công việc thành các đầu mục con (Subtasks) với cơ chế bảo vệ tối đa 1 cấp độ, phân quyền người phụ trách độc lập và tự động cộng dồn tiến độ (Progress Roll-up).
- **FR-PRJ-018 (Project Lifecycle & Templates):** Quản lý vòng đời dự án (Active, Paused, Archived), nhân bản nhanh quy trình chuẩn thông qua Templates, và đóng băng dữ liệu dự án cũ (Archived Immutability).

### 1.3 Definitions & Abbreviations
- **WIP (Work In Progress):** Số lượng công việc tối đa được phép xử lý đồng thời trong một trạng thái/cột nhằm tránh quá tải.
- **Meta-Status:** Danh mục trạng thái trừu tượng cố định của hệ thống (`OPEN`, `TODO`, `IN_PROGRESS`, `IN_REVIEW`, `DONE`, `CANCELED`) để chuẩn hóa báo cáo tổng thể.
- **EAV (Entity-Attribute-Value):** Mô hình lưu trữ cơ sở dữ liệu cho các trường dữ liệu động.
- **Progress Roll-up:** Cơ chế tính toán tự động tỷ lệ % hoàn thành của Task cha dựa trên tỷ lệ Subtask hoàn thành.
- **DoD (Definition of Done):** Tiêu chuẩn hoàn thành của một công việc.
- **RBAC (Role-Based Access Control):** Kiểm soát truy cập dựa trên vai trò người dùng.

### 1.4 References
- [FR-PRJ-002: Dynamic Workflows & WIP Limits](./FR-PRJ-002_dynamic_workflows_wip.md)
- [FR-PRJ-004: Dynamic Custom Fields](./FR-PRJ-004_dynamic_custom_fields.md)
- [FR-PRJ-006: Subtask Management](./FR-PRJ-006_subtask_management.md)
- [FR-PRJ-018: Project Lifecycle Templates](./FR-PRJ-018_project_lifecycle_templates.md)
- [01_SRS_PHASE1_FOUNDATION.md](../phase_1/01_SRS_PHASE1_FOUNDATION.md)
- [03_RTM_PHASE2_OMNIPROJECT_CORE.csv](./03_RTM_PHASE2_OMNIPROJECT_CORE.csv)

---

## 2. Overall Description

### 2.1 Product Perspective
OmniProject Phase 2 vận hành trên hạ tầng Multi-tenant đã được thiết lập ở Phase 1. Hệ thống cung cấp REST API và WebSocket Event Bus kết nối cơ sở dữ liệu PostgreSQL (kết hợp JSONB Indexing và quan hệ chặt chẽ). Dữ liệu được cô lập tuyệt đối theo `workspace_id` và `project_id`.

```mermaid
graph TD
    Client[Web Client SPA / Mobile] -->|HTTPS REST API / WebSocket| Gateway[API Gateway & Auth Middleware]
    Gateway --> WF_Module[Workflow & WIP Engine]
    Gateway --> CF_Module[Custom Field Engine]
    Gateway --> ST_Module[Subtask & Rollup Service]
    Gateway --> TPL_Module[Lifecycle & Template Service]
    WF_Module --> DB[(PostgreSQL DB)]
    CF_Module --> DB
    ST_Module --> DB
    TPL_Module --> DB
    WF_Module -.->|Pub/Sub Event| Realtime[Realtime WebSocket Server]
    Realtime -.->|Push Notifications| Client
```

### 2.2 Product Functions
1. **Quy trình luồng linh hoạt (Dynamic Workflows):**
   - Tạo, sửa, xóa, sắp xếp cột trạng thái (Custom Status).
   - Bắt buộc ánh xạ vào 1 trong 6 Meta-Status cốt lõi.
   - Thiết lập và cưỡng chế kiểm tra WIP Limit khi kéo thả Task.
   - Cho phép PM vượt rào WIP (WIP Override).
   - Tự động gợi ý chuyển giao người phụ trách (Handoff Assignee) và bắn thông báo.
   - Gán cờ cảnh báo kẹt công việc (Blocked Flag) kèm lý do.
2. **Trường dữ liệu tùy chỉnh (Dynamic Custom Fields):**
   - Định nghĩa trường mới với các kiểu: `TEXT`, `NUMBER`, `CURRENCY`, `DATE`, `DROPDOWN`, `CHECKBOX`.
   - Cấu hình validation: `is_required`, giá trị min/max, danh sách lựa chọn Dropdown.
   - Lưu trữ, kiểm tra định dạng dữ liệu phía server (Server-side Type Enforcement).
   - Tìm kiếm, lọc (Filter) và sắp xếp (Sort) Task dựa trên Custom Field.
   - Xóa mềm (Soft Delete) Custom Field để bảo toàn lịch sử dữ liệu.
3. **Quản lý công việc con (Subtasks):**
   - Tạo, chỉnh sửa, xóa Subtasks bên trong Task gốc.
   - Gán người phụ trách (Assignee) và Deadline độc lập với Task cha.
   - Tự động cộng dồn tiến độ % của Task cha (Progress Roll-up).
   - Hỗ trợ hiển thị mở rộng cây công việc (Tree-view) trên bảng danh sách.
4. **Vòng đời dự án & Template chuẩn hóa (Project Lifecycle & Templates):**
   - Khởi tạo dự án mới độc lập hoặc sao chép nguyên mẫu từ Template (System Template & Workspace Template).
   - Lưu trữ một dự án đang vận hành thành Template tái sử dụng.
   - Đóng/Lưu trữ (Archive) dự án: đưa về trạng thái đóng băng chỉ đọc (Read-only) và ẩn khỏi giao diện vận hành hằng ngày.
   - Khôi phục (Unarchive) dự án về trạng thái hoạt động (Active).

### 2.3 User Classes & Characteristics
| User Class | Mô tả đặc điểm | Quyền hạn trong Phase 2 |
|---|---|---|
| **Workspace Admin** | Chủ sở hữu hoặc Quản trị viên Workspace | Quản lý toàn bộ Projects, quản lý System/Workspace Templates, can thiệp cấu hình hệ thống, kiểm soát hạn mức lưu trữ/dự án của gói dịch vụ. |
| **Project Manager (PM)** | Người điều phối dự án cụ thể | Tạo và cấu hình Workflows, quy định WIP Limits, Override WIP, tạo Custom Fields, lưu dự án thành Template, Archive/Unarchive dự án, phân bổ thành viên. |
| **Team Member** | Thành viên thực thi trực tiếp các đầu việc | Kéo thả chuyển trạng thái Task (trong giới hạn WIP), cập nhật giá trị Custom Fields, đánh dấu cờ Blocked, tạo/hoàn thành Subtasks. Không có quyền sửa cấu hình Workflow/Custom Field. |
| **Observer / Guest** | Khách hàng hoặc đối tác giám sát tiến độ | Quyền chỉ đọc (Read-only): Xem bảng Kanban, xem tiến độ Task/Subtask, xem các trường Custom Fields. Không được phép chỉnh sửa. |

### 2.4 General Constraints & Business Rules
1. **Hierarchy Depth Constraint (BL-PRJ-006.2):** Cấu trúc phân cấp Task chỉ hỗ trợ tối đa **1 cấp độ con** (`Parent Task` -> `Child Subtask`). Hệ thống nghiêm cấm tạo Subtask bên trong một Subtask (No Grandchildren).
2. **Parent-Child Completion Constraint (BL-PRJ-006.3):** Không thể chuyển Task cha sang trạng thái hoàn thành (`DONE`) nếu vẫn còn ít nhất 1 Subtask chưa hoàn thành (`Incomplete`). Hệ thống ném lỗi HTTP 400 Bad Request.
3. **Workflow Integrity Constraint (BL-PRJ-002.3):** Không được phép xóa một cột trạng thái nếu bên trong cột đó vẫn còn tồn tại Task. Bắt buộc phải di chuyển toàn bộ Task sang cột khác trước khi thực hiện xóa.
4. **WIP Enforcement & Privilege Constraint (BL-PRJ-002.1 & BL-PRJ-002.2):** Khi cột đạt ngưỡng WIP, thao tác kéo thả của Member bị từ chối và đẩy thẻ về vị trí cũ. Chỉ PM hoặc Admin mới có quyền xác nhận "WIP Override".
5. **Task Blocked Constraint (BL-PRJ-002.5):** Khi Task bị gán cờ `is_blocked = true`, Task bị khóa tại cột hiện tại. Thành viên bắt buộc phải giải tỏa cờ ("Resolve Block") trước khi có thể kéo Task sang cột khác.
6. **Workspace Quota Constraint (BL-PRJ-018.1):** Kiểm tra giới hạn số lượng dự án theo gói cước (Gói Free tối đa 3 dự án Active). Nếu vượt quá, chặn tạo mới và yêu cầu nâng cấp gói cước.
7. **Archived Immutability Constraint (BL-PRJ-018.3):** Dự án ở trạng thái `ARCHIVED` bị đóng băng hoàn toàn. Mọi hành vi sửa đổi Task, Comment, Subtask, Custom Field đều bị từ chối (HTTP 403 / Read-only). Đồng thời các Task này bị loại trừ khỏi các màn hình "My Tasks", "Portfolio Dashboard" và "Workload View".

### 2.5 Lean Metrics Calculation (Quản trị tinh gọn)
Hệ thống tính toán tự động 4 chỉ số hiệu suất căn cứ trên thời điểm chuyển đổi Meta-Status:
- **Điểm cam kết (Commitment Point):** Thời điểm Task được di chuyển từ Meta-Status `OPEN` (Backlog) sang `TODO` (Kanban Board cam kết).
- **System Lead Time:** $T_{\text{DONE}} - T_{\text{CREATED}}$ (Tổng thời gian từ lúc tạo thẻ tại Backlog đến khi hoàn thành).
- **Delivery Lead Time:** $T_{\text{DONE}} - T_{\text{TODO}}$ (Tốc độ giao hàng thực tế của đội ngũ tính từ điểm cam kết).
- **Cycle Time:** $T_{\text{DONE}} - T_{\text{IN\_PROGRESS}}$ (Thời gian thao tác kỹ thuật thực tế để giải quyết Task).

---

## 3. Specific Non-Functional Requirements (NFRs)
> Toàn bộ NFR được định lượng bằng chỉ số đo lường cụ thể (Measurable Thresholds) theo chuẩn IEEE 830-1998.

### 3.1 Performance Requirements (Hiệu năng)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-PRF-001** | Thời gian phản hồi API CRUD thông thường | 95% số lượng request (P95) $\le 200\text{ms}$; 99% request (P99) $\le 400\text{ms}$ dưới tải 500 req/sec. |
| **NFR-PRF-002** | Thời gian nhân bản Dự án từ Template | Quá trình Clone Template chứa $\le 200$ Task mẫu, Workflow và Custom Fields hoàn tất trong thời gian $\le 1.8\text{s}$. |
| **NFR-PRF-003** | Độ trễ kích hoạt Progress Roll-up | Thời gian từ lúc Subtask chuyển trạng thái đến khi Task cha tính toán lại `% progress` và ghi DB $\le 45\text{ms}$. |
| **NFR-PRF-004** | Hiệu năng Tìm kiếm & Lọc (Filter/Sort) theo Custom Field | Thời gian truy vấn lọc dữ liệu phức tạp $\le 350\text{ms}$ trên tập dữ liệu $50,000$ Tasks nhờ PostgreSQL GIN Indexing. |
| **NFR-PRF-005** | Độ trễ thông báo thời gian thực (WebSocket Latency) | Sự kiện Handoff Assignee hoặc Cập nhật Kanban được phát tới Client trong thời gian $\le 80\text{ms}$. |

### 3.2 Security & Data Protection (Bảo mật)
| ID | Requirement Description | Measurable Threshold & Reference Standard |
|---|---|---|
| **NFR-SEC-001** | Kiểm soát phân quyền đa tầng (RBAC Enforcement) | 100% API cấu hình (Workflow, Custom Field, WIP Override, Archive) chặn đứng truy cập trái phép với mã phản hồi `HTTP 403 Forbidden` trong thời gian $\le 15\text{ms}$. (OWASP Broken Object Level Authorization) |
| **NFR-SEC-002** | Kiểm duyệt dữ liệu đầu vào (Input Sanitization & Type Safety) | Kiểm tra dữ liệu Custom Field phía Server. Lọc mã độc XSS (DOMPurify/Sanitizer). Từ chối payload sai kiểu với mã `HTTP 422 Unprocessable Entity`. (OWASP A03: Injection) |
| **NFR-SEC-003** | Bảo vệ dữ liệu Đa khách thuê (Multi-tenant Isolation) | Mọi câu truy vấn Database bắt buộc phải kèm cặp điều kiện khóa ngoại `workspace_id = context.current_workspace` và `project_id = context.current_project`. |
| **NFR-SEC-004** | Nhật ký kiểm toán hành vi trọng yếu (Audit Logging) | 100% hành vi Override WIP, Thay đổi quyền Task, Xóa Custom Field, và Archive Project phải được lưu trữ vào bảng `Audit_Log` (User ID, IP, Timestamp, Old Value, New Value) với thời gian lưu trữ tối thiểu 365 ngày. |

### 3.3 Reliability & Availability (Độ tin cậy & Sẵn sàng)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-AVL-001** | Tính khả dụng của Hệ thống (Uptime) | Đạt tối thiểu $99.9\%$ Uptime hằng tháng (tổng thời gian gián đoạn ngoài kế hoạch $\le 43.8$ phút/tháng). |
| **NFR-AVL-002** | Tính toàn vẹn giao dịch (ACID Transaction Integrity) | Quá trình Clone Template và Cập nhật Roll-up phải thực thi trong Database Transaction. Nếu có bất kỳ lỗi nào xảy ra, 100% trạng thái phải được Rollback nguyên vẹn, không để sót dữ liệu rác (Orphan Records). |
| **NFR-AVL-003** | Khả năng tự phục hồi kết nối Real-time | Khi WebSocket bị ngắt kết nối đột ngột, Client tự động kết nối lại theo thuật toán Exponential Backoff ($1\text{s}, 2\text{s}, 4\text{s}, 8\text{s}... \max 30\text{s}$) và gọi API đồng bộ bù sai lệch trạng thái. |

### 3.4 Scalability (Khả năng mở rộng)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-SCA-001** | Khả năng mở rộng dung lượng Task & Fields | Kiến trúc cơ sở dữ liệu hỗ trợ tối thiểu $2,000,000$ bản ghi Task Custom Values và $500,000$ Tasks trên mỗi Workspace mà không làm suy giảm tốc độ query quá $15\%$. |
| **NFR-SCA-002** | Khả năng mở rộng không trạng thái (Stateless Scaling) | Backend API Server hoàn toàn Stateless, cho phép Horizontal Pod Autoscaling (HPA) từ 2 lên 20 pods khi mức chiếm dụng CPU vượt ngưỡng $70\%$. |

### 3.5 Maintainability & Extensibility (Khả năng bảo trì)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-MNT-001** | Thiết kế mở rộng kiểu Custom Field | Áp dụng Strategy Pattern cho Dynamic Validator; thời gian bổ sung một kiểu dữ liệu mới (ví dụ: `FORMULA`, `GEO_LOCATION`) chỉ cần thêm 1 Validator Class độc lập mà không phải sửa logic controller hiện có. |
| **NFR-MNT-002** | Độ bao phủ kiểm thử tự động (Test Coverage) | Mã nguồn các module cốt lõi (Workflow, WIP, Roll-up, Template) phải đạt độ bao phủ Unit & Integration Test tối thiểu $\ge 85\%$. |

### 3.6 Usability (Trải nghiệm người dùng)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-USB-001** | Phản hồi giao diện kéo thả (Drag-and-Drop Responsiveness) | Thao tác kéo thả Task trên Kanban hiển thị Optimistic UI ngay lập tức trong vòng $\le 16\text{ms}$ (tương đương chuẩn 60fps). Nếu Server trả về lỗi WIP, thẻ tự động trượt mượt mà (smooth transition) về vị trí cũ kèm Toast thông báo trong $\le 200\text{ms}$. |
| **NFR-USB-002** | Độ rõ ràng của thông báo lỗi (Error Clarity) | 100% thông báo lỗi nghiệp vụ hiển thị bằng tiếng Việt rõ nghĩa, chỉ đích danh nguyên nhân (ví dụ: "Cột Đang làm đã đạt giới hạn tối đa 3 công việc. Chỉ Quản lý dự án mới có quyền đưa thêm"). |

### 3.7 Compatibility (Tương thích môi trường)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-CMP-001** | Hỗ trợ trình duyệt Web | Tương thích và hoạt động ổn định không lỗi hiển thị trên: Chrome $\ge 90$, Microsoft Edge $\ge 90$, Mozilla Firefox $\ge 88$, Safari $\ge 14$. |
| **NFR-CMP-002** | Đáp ứng màn hình (Responsive Display) | Giao diện tối ưu hoàn toàn cho độ phân giải màn hình từ Desktop tiêu chuẩn ($1920\times1080$, $1440\times900$, $1366\times768$) đến Laptop và Tablet ($1024\times768$). |

---

## 4. Data Models & Schema Design

### 4.1 Entity Relationship Diagram (Phase 2 Entities)

```mermaid
erDiagram
    PROJECT ||--o{ WORKFLOW_STATUS : "contains"
    PROJECT ||--o{ CUSTOM_FIELD_DEFINITION : "configures"
    PROJECT ||--o{ TASK : "owns"
    WORKFLOW_STATUS ||--o{ TASK : "groups"
    TASK ||--o{ TASK : "subtask_of (1-level)"
    TASK ||--o{ TASK_CUSTOM_FIELD_VALUE : "has"
    CUSTOM_FIELD_DEFINITION ||--o{ TASK_CUSTOM_FIELD_VALUE : "defines"

    PROJECT {
        uuid project_id PK
        uuid workspace_id FK
        string name
        enum status "ACTIVE, PAUSED, ARCHIVED"
        boolean is_template
        timestamp created_at
    }

    WORKFLOW_STATUS {
        uuid status_id PK
        uuid project_id FK
        string name
        int order_index
        int wip_limit "Nullable"
        enum meta_category "OPEN, TODO, IN_PROGRESS, IN_REVIEW, DONE, CANCELED"
    }

    CUSTOM_FIELD_DEFINITION {
        uuid field_id PK
        uuid project_id FK
        string name
        enum type "TEXT, NUMBER, CURRENCY, DATE, DROPDOWN, CHECKBOX"
        json options "Nullable (dropdown choices)"
        boolean is_required
        boolean is_deleted "Soft delete flag"
    }

    TASK {
        uuid task_id PK
        uuid project_id FK
        uuid status_id FK
        uuid parent_task_id FK "Nullable (Self-ref)"
        string title
        uuid assignee_id "Nullable"
        int progress_percent "0 to 100"
        boolean is_blocked "Default false"
        string block_reason "Nullable"
        date due_date "Nullable"
        timestamp updated_at
    }

    TASK_CUSTOM_FIELD_VALUE {
        uuid value_id PK
        uuid task_id FK
        uuid field_id FK
        string value_string "Nullable"
        numeric value_number "Nullable"
        timestamp value_date "Nullable"
        boolean value_boolean "Nullable"
    }
```

---

## 5. Detailed Sequence Diagrams (CRUD & Operational Flows)

### 5.1 Dynamic Workflows & WIP Management (FR-PRJ-002)

#### 5.1.1 Create Workflow Status (Tạo cột quy trình)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Nhập tên "Ready for QA", chọn Meta-Status "IN_REVIEW", WIP Limit = 3
    Client->>API: POST /projects/{project_id}/workflow-statuses
    Note over Client,API: Headers: Authorization Bearer Token
    API->>API: 1. Kiểm tra quyền PM trong Project (RBAC)
    API->>DB: 2. Kiểm tra trùng tên status trong Project
    DB-->>API: Tên hợp lệ, chưa tồn tại
    API->>DB: 3. Lấy max order_index hiện tại và INSERT Status mới
    DB-->>API: Trả về status_id
    API-->>Client: HTTP 201 Created (Status Metadata)
    Client-->>PM: Hiển thị cột mới trên bảng Kanban
```

#### 5.1.2 Read / Get Workflow Statuses
```mermaid
sequenceDiagram
    autonumber
    actor User as Team Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Truy cập màn hình Kanban Board của Dự án
    Client->>API: GET /projects/{project_id}/workflow-statuses
    API->>DB: SELECT * FROM Workflow_Status WHERE project_id = {id} ORDER BY order_index ASC
    DB-->>API: Danh sách Workflow_Status kèm thống kê số Tasks hiện tại
    API-->>Client: HTTP 200 OK
    Client-->>User: Hiển thị đầy đủ các cột trạng thái và chỉ số WIP
```

#### 5.1.3 Update Workflow Status (Đổi tên, Thứ tự, WIP Limit)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Cập nhật WIP Limit cột "Đang làm" từ 3 lên 5
    Client->>API: PUT /projects/{project_id}/workflow-statuses/{status_id}
    Note over Client,API: Payload: { wip_limit: 5 }
    API->>API: Kiểm tra quyền PM
    API->>DB: UPDATE Workflow_Status SET wip_limit = 5 WHERE status_id = {status_id}
    DB-->>API: Update thành công
    API-->>Client: HTTP 200 OK
    Client-->>PM: Cập nhật header cột "Đang làm (x/5)"
```

#### 5.1.4 Delete Workflow Status (Ràng buộc toàn vẹn BL-PRJ-002.3)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Bấm "Xóa cột" trạng thái "Kiểm thử"
    Client->>API: DELETE /projects/{project_id}/workflow-statuses/{status_id}
    API->>DB: SELECT COUNT(*) FROM Tasks WHERE status_id = {status_id}
    DB-->>API: count = 3
    alt Cột vẫn còn Task (count > 0)
        API-->>Client: HTTP 400 Bad Request ("Không thể xóa trạng thái đang chứa Task. Vui lòng di dời Task trước.")
        Client-->>PM: Hiển thị Popup cảnh báo đỏ, từ chối xóa
    else Cột trống rỗng (count == 0)
        API->>DB: DELETE FROM Workflow_Status WHERE status_id = {status_id}
        DB-->>API: Xóa thành công
        API-->>Client: HTTP 200 OK
        Client-->>PM: Xóa cột khỏi giao diện bảng
    end
```

#### 5.1.5 Move Task with WIP Check, PM Override & Handoff (FR-PRJ-002.2 -> 002.4)
```mermaid
sequenceDiagram
    autonumber
    actor Dev as Developer (Assignee hiện tại)
    participant Client as Web Client
    participant API as API Server
    participant DB as Database
    actor PM as Project Manager
    actor Tester as Tester (Assignee mới)

    Dev->>Client: Kéo Task A sang cột "Testing" (Meta-Status: IN_REVIEW)
    Client->>Client: Kiểm tra cờ is_blocked của Task A
    alt Task đang bị Blocked (BL-PRJ-002.5)
        Client-->>Dev: Chặn kéo thẻ! Báo lỗi "Task đang bị nghẽn, cần gỡ cờ trước."
    else Task bình thường
        Client->>API: PATCH /tasks/{task_id}/move (new_status_id: "Testing")
        API->>DB: Đếm số Task hiện tại ở cột "Testing" so với wip_limit
        DB-->>API: Current = 3, Limit = 3 (Đã đạt trần)
        alt Vượt WIP Limit & User là Dev thông thường (BL-PRJ-002.1 & 002.2)
            API-->>Client: HTTP 403 Forbidden ("WIP Limit Exceeded - Quyền hạn không đủ để Override")
            Client->>Client: Revert Task về cột cũ
            Client-->>Dev: Báo Toast đỏ: "Cột đã quá tải WIP. Chỉ PM mới có thể bổ sung việc."
        else Vượt WIP Limit nhưng User là PM (Override Allowed)
            Client-->>PM: Hiển thị Modal xác nhận: "Cột đã đạt tối đa 3 Task. Bạn có muốn Override không?"
            PM->>Client: Bấm "Xác nhận Override"
            Client->>API: PATCH /tasks/{task_id}/move (new_status_id: "Testing", override_wip: true)
            API->>DB: Kiểm tra quyền PM -> Hợp lệ
            API-->>Client: HTTP 200 OK (Status Updated, Yêu cầu Handoff Assignee)
            Client-->>Dev: Bật Popup: "Chuyển giao cho ai tiếp nhận kiểm thử?"
            Dev->>Client: Chọn "Tester B"
            Client->>API: PATCH /tasks/{task_id} (assignee_id: Tester_B)
            API->>DB: Cập nhật Assignee mới & Lưu Activity History
            DB-->>API: OK
            API-->>Client: HTTP 200 OK
            API-xTester: Bắn WebSocket Notification: "Bạn được giao kiểm thử Task A"
            Client-->>Dev: Cập nhật UI hoàn tất
        end
    end
```

#### 5.1.6 Toggle Task Blocked State (FR-PRJ-002.5)
```mermaid
sequenceDiagram
    autonumber
    actor Member as Team Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    Member->>Client: Bấm nút "Mark as Blocked", nhập lý do "Chờ API bên thứ 3"
    Client->>API: POST /tasks/{task_id}/block (reason: "Chờ API bên thứ 3")
    API->>DB: UPDATE Tasks SET is_blocked = true, block_reason = "..." WHERE task_id = {task_id}
    API->>DB: Ghi log vào Activity_Stream
    DB-->>API: Success
    API-->>Client: HTTP 200 OK
    Client-->>Member: Hiển thị huy hiệu ĐỎ CẢNH BÁO trên Task Card
```

---

### 5.2 Dynamic Custom Fields (FR-PRJ-004)

#### 5.2.1 Create Custom Field Definition
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Nhập tên "Ngân sách dự tính", Type: "CURRENCY", is_required: true
    Client->>API: POST /projects/{project_id}/custom-fields
    Note over Client,API: Payload: { name: "Ngân sách dự tính", type: "CURRENCY", is_required: true }
    API->>API: Validate kiểu dữ liệu hợp lệ (TEXT, NUMBER, CURRENCY, DATE, DROPDOWN, CHECKBOX)
    API->>DB: INSERT INTO Custom_Field_Definition
    DB-->>API: field_id mới
    API-->>Client: HTTP 201 Created
    Client-->>PM: Hiển thị trường mới trong danh sách Cấu hình Dự án
```

#### 5.2.2 Read Custom Fields & Task Values
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Mở Modal chi tiết Task 101
    Client->>API: GET /projects/{project_id}/custom-fields (Lấy metadata định nghĩa)
    Client->>API: GET /tasks/{task_id}/custom-field-values (Lấy dữ liệu đã điền)
    API->>DB: Query bảng Custom_Field_Definition và Task_Custom_Field_Value
    DB-->>API: Data Definitions & Stored Values
    API-->>Client: HTTP 200 OK
    Client-->>User: Render các input form tương ứng theo từng Type trên giao diện Task
```

#### 5.2.3 Update / Set Custom Field Value with Type Safety (BL-PRJ-004.1)
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant Validator as Dynamic Validator Engine
    participant DB as Database

    User->>Client: Nhập giá trị "Mười lăm triệu" vào ô "Ngân sách dự tính"
    Client->>API: PUT /tasks/{task_id}/custom-fields/{field_id} (value: "Mười lăm triệu")
    API->>DB: Lấy định nghĩa kiểu của field_id
    DB-->>API: type = "CURRENCY", is_required = true
    API->>Validator: Thực thi Validate("Mười lăm triệu", CURRENCY)
    Validator-->>API: Validation Failed (Giá trị không phải số)
    API-->>Client: HTTP 422 Unprocessable Entity ("Giá trị trường Ngân sách bắt buộc là dạng số tiền")
    Client-->>User: Hiển thị báo lỗi đỏ dưới ô input
    
    User->>Client: Sửa lại nhập "15000000"
    Client->>API: PUT /tasks/{task_id}/custom-fields/{field_id} (value: "15000000")
    API->>Validator: Thực thi Validate("15000000", CURRENCY)
    Validator-->>API: Validation Passed (Số tiền hợp lệ)
    API->>DB: UPSERT INTO Task_Custom_Field_Value (task_id, field_id, value_number = 15000000)
    DB-->>API: Success
    API-->>Client: HTTP 200 OK
    Client-->>User: Hiển thị giá trị đã lưu "15,000,000 VND"
```

#### 5.2.4 Filter and Sort Tasks by Custom Field (FR-PRJ-004.4 & BL-PRJ-004.3)
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Chọn Filter: Ngân sách > 10,000,000 & Sắp xếp theo Ngày phát hành (DATE)
    Client->>API: GET /projects/{id}/tasks?filter[field_budget][gt]=10000000&sort=field_release_date:asc
    API->>DB: Query Tasks JOIN Task_Custom_Field_Value (Tận dụng Indexing trên value_number/value_date)
    DB-->>API: Danh sách Tasks thỏa mãn điều kiện
    API-->>Client: HTTP 200 OK (Paginated Task List)
    Client-->>User: Cập nhật danh sách bảng Table View
```

#### 5.2.5 Delete Custom Field (Soft Delete BL-PRJ-004.2)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Nhấn xóa trường "Ngân sách dự tính"
    Client->>API: DELETE /projects/{project_id}/custom-fields/{field_id}
    API->>API: Kiểm tra quyền PM
    API->>DB: UPDATE Custom_Field_Definition SET is_deleted = true WHERE field_id = {field_id}
    DB-->>API: Update thành công
    API-->>Client: HTTP 200 OK ("Đã ẩn trường tùy chỉnh thành công")
    Client-->>PM: Ẩn trường khỏi cấu hình; dữ liệu cũ trên Task vẫn được bảo toàn
```

---

### 5.3 Subtask Management (FR-PRJ-006)

#### 5.3.1 Create Subtask (Kiểm soát 1 cấp độ BL-PRJ-006.2)
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Trong Task A, bấm "Thêm Subtask" -> Nhập tên "Thiết kế DB", Assignee: Dev B
    Client->>API: POST /tasks (parent_task_id: {task_a_id}, title: "Thiết kế DB", assignee_id: Dev_B)
    API->>DB: SELECT parent_task_id FROM Tasks WHERE task_id = {task_a_id}
    DB-->>API: Parent_Task_ID = NULL (Task A là Task gốc cấp 1)
    alt Nếu Task A đã có parent_task_id != NULL (Đang cố tạo Subtask cấp 2)
        API-->>Client: HTTP 400 Bad Request ("Hệ thống chỉ hỗ trợ Subtask 1 cấp độ con")
        Client-->>User: Chặn tạo và báo lỗi
    else Hợp lệ (Cấp độ 1)
        API->>DB: BEGIN TRANSACTION
        API->>DB: INSERT INTO Tasks (title: "Thiết kế DB", parent_task_id: {task_a_id}, status: "TODO")
        API->>DB: Đếm lại tổng Subtasks và số Subtasks DONE của Task A
        API->>DB: Cập nhật progress_percent của Task A
        API->>DB: COMMIT TRANSACTION
        DB-->>API: Success
        API-->>Client: HTTP 201 Created (Subtask metadata)
        Client-->>User: Thêm Subtask mới vào danh sách dưới Task A
    end
```

#### 5.3.2 Read Subtasks (Tree-view Expansion FR-PRJ-006.4)
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Trên Table View, click icon mũi tên mở rộng (Expand) Task A
    Client->>API: GET /tasks/{task_a_id}/subtasks
    API->>DB: SELECT * FROM Tasks WHERE parent_task_id = {task_a_id} ORDER BY created_at ASC
    DB-->>API: Danh sách 4 Subtasks con
    API-->>Client: HTTP 200 OK
    Client-->>User: Hiển thị 4 hàng Subtask thụt lề dưới Task A kèm checkbox và status
```

#### 5.3.3 Update Subtask & Auto Progress Roll-up (FR-PRJ-006.3 & BL-PRJ-006.1)
```mermaid
sequenceDiagram
    autonumber
    actor Assignee as Subtask Assignee
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    Assignee->>Client: Tích chọn hoàn thành Subtask 1 (Đổi status sang DONE)
    Client->>API: PATCH /tasks/{subtask_1_id} (status: "DONE")
    API->>DB: BEGIN TRANSACTION
    API->>DB: 1. UPDATE Tasks SET status = 'DONE' WHERE task_id = {subtask_1_id}
    API->>DB: 2. Lấy parent_task_id của Subtask 1 -> Trả về ID_Task_A
    API->>DB: 3. SELECT COUNT(*) as total, COUNT(*) FILTER (WHERE status = 'DONE') as completed FROM Tasks WHERE parent_task_id = ID_Task_A
    DB-->>API: total = 4, completed = 2
    API->>API: 4. Tính toán Roll-up: Progress = (2 / 4) * 100 = 50%
    API->>DB: 5. UPDATE Tasks SET progress_percent = 50 WHERE task_id = ID_Task_A
    API->>DB: COMMIT TRANSACTION
    DB-->>API: Hoàn tất
    API-->>Client: HTTP 200 OK (Subtask: DONE, Parent Progress: 50%)
    Client->>Client: Gạch ngang tên Subtask 1 & Cập nhật thanh tiến độ Task A lên 50%
```

#### 5.3.4 Attempt to Complete Parent Task with Incomplete Subtasks (BL-PRJ-006.3)
```mermaid
sequenceDiagram
    autonumber
    actor Dev as Developer
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    Dev->>Client: Kéo Task A (Task cha) sang cột "DONE"
    Client->>API: PATCH /tasks/{task_a_id}/move (new_status: "DONE")
    API->>DB: SELECT COUNT(*) FROM Tasks WHERE parent_task_id = {task_a_id} AND status != 'DONE'
    DB-->>API: incomplete_subtasks = 2
    alt Còn Subtask chưa hoàn thành (incomplete_subtasks > 0)
        API-->>Client: HTTP 400 Bad Request ("Không thể đóng Task cha khi còn 2 Subtasks chưa hoàn tất.")
        Client->>Client: Revert Task A về cột cũ trên Kanban
        Client-->>Dev: Bật Modal thông báo: "Vui lòng hoàn thành hoặc đóng toàn bộ Subtasks trước khi chuyển sang Done."
    else Toàn bộ Subtask đã Done
        API->>DB: UPDATE Tasks SET status = 'DONE', progress_percent = 100 WHERE task_id = {task_a_id}
        DB-->>API: Success
        API-->>Client: HTTP 200 OK
        Client-->>Dev: Chuyển Task A sang cột DONE thành công
    end
```

#### 5.3.5 Delete Subtask & Recalculate Roll-up
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Nhấn icon Xóa Subtask 1 của Task A
    Client->>API: DELETE /tasks/{subtask_1_id}
    API->>DB: BEGIN TRANSACTION
    API->>DB: 1. Lấy parent_task_id -> ID_Task_A
    API->>DB: 2. DELETE FROM Tasks WHERE task_id = {subtask_1_id}
    API->>DB: 3. Tính toán lại Progress Roll-up cho Task A dựa trên số Subtask còn lại
    API->>DB: 4. UPDATE Tasks SET progress_percent = new_progress WHERE task_id = ID_Task_A
    API->>DB: COMMIT TRANSACTION
    DB-->>API: Hoàn tất
    API-->>Client: HTTP 200 OK
    Client-->>User: Xóa dòng Subtask khỏi giao diện & cập nhật lại % tiến độ của Task A
```

---

### 5.4 Project Lifecycle & Templates (FR-PRJ-018)

#### 5.4.1 Create Project from Template (Hạn mức BL-PRJ-018.1 & Sao chép BL-PRJ-018.2)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Chọn "Tạo dự án mới", chọn Template "Scrum Chuẩn", đặt tên "Dự Án Mobile v1"
    Client->>API: POST /projects (template_id: "TPL_SCRUM", name: "Dự Án Mobile v1")
    API->>DB: 1. Kiểm tra số dự án ACTIVE hiện tại của Workspace so với Plan Limit (BL-PRJ-018.1)
    DB-->>API: Active Projects = 3, Free Plan Limit = 3 (Đã đạt giới hạn)
    alt Vượt quá hạn mức gói cước Workspace
        API-->>Client: HTTP 402 Payment Required ("Workspace đã đạt giới hạn 3 dự án của gói Free. Vui lòng nâng cấp.")
        Client-->>PM: Hiển thị Popup nâng cấp tài khoản
    else Hạn mức hợp lệ
        API->>DB: BEGIN TRANSACTION
        API->>DB: 2. Tạo bản ghi Projects mới (status: 'ACTIVE', is_template: false)
        API->>DB: 3. Sao chép toàn bộ Workflow_Status từ Template sang Project mới
        API->>DB: 4. Sao chép toàn bộ Custom_Field_Definition từ Template sang Project mới
        API->>DB: 5. Đọc danh sách Tasks mẫu trong Template
        API->>API: 6. Transform dữ liệu Tasks mẫu: Status gán về cột đầu (TODO), Assignee = NULL, xóa Comments, shift due_date tịnh tiến theo ngày tạo
        API->>DB: 7. Bulk INSERT Tasks mẫu vào Project mới
        API->>DB: COMMIT TRANSACTION
        DB-->>API: Tạo thành công, trả về new_project_id
        API-->>Client: HTTP 201 Created (new_project_id)
        Client-->>PM: Điều hướng trực tiếp vào Kanban Board của "Dự Án Mobile v1"
    end
```

#### 5.4.2 Read / List Projects (Phân tách Active & Archived)
```mermaid
sequenceDiagram
    autonumber
    actor User as Member
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    User->>Client: Truy cập trang Danh sách Dự án (Portfolio View)
    Client->>API: GET /projects?include_archived=false
    API->>DB: SELECT * FROM Projects WHERE workspace_id = {ws_id} AND status != 'ARCHIVED'
    DB-->>API: Danh sách các dự án đang hoạt động
    API-->>Client: HTTP 200 OK
    Client-->>User: Hiển thị danh sách các dự án Active (Không bị lẫn dự án đã đóng)
```

#### 5.4.3 Save Active Project as Template (FR-PRJ-018.3)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Bấm "Lưu thành Template", đặt tên "Mẫu Dự án Marketing 2026"
    Client->>API: POST /projects/{project_id}/save-as-template (template_name: "Mẫu Dự án Marketing 2026")
    API->>API: Kiểm tra quyền PM/Admin
    API->>DB: BEGIN TRANSACTION
    API->>DB: 1. Tạo bản ghi Project mới với is_template = true, status = 'ACTIVE'
    API->>DB: 2. Sao chép toàn bộ cấu hình Workflow và Custom Fields sang Template mới
    API->>DB: 3. Sao chép danh sách Tasks mẫu (đã xóa dữ liệu thực thi, lịch sử cá nhân)
    API->>DB: COMMIT TRANSACTION
    DB-->>API: Hoàn tất
    API-->>Client: HTTP 201 Created ("Lưu Template thành công")
    Client-->>PM: Template xuất hiện trong Thư viện Template của Workspace
```

#### 5.4.4 Archive Project (Đóng băng chỉ đọc BL-PRJ-018.3)
```mermaid
sequenceDiagram
    autonumber
    actor PM as Project Manager
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    PM->>Client: Nhấn chọn "Archive Project"
    Client->>Client: Hiển thị Modal xác nhận cảnh báo đóng băng dự án
    PM->>Client: Xác nhận Archive
    Client->>API: POST /projects/{project_id}/archive
    API->>API: Kiểm tra quyền PM
    API->>DB: UPDATE Projects SET status = 'ARCHIVED', updated_at = NOW() WHERE project_id = {id}
    DB-->>API: Success
    API-->>Client: HTTP 200 OK ("Dự án đã được lưu trữ")
    Client-->>PM: Đổi nhãn dự án thành "Archived (Read-only)", ẩn khỏi Sidebar và thanh điều hướng chính
```

#### 5.4.5 Unarchive Project (Khôi phục dự án FR-PRJ-018.5)
```mermaid
sequenceDiagram
    autonumber
    actor Admin as Workspace Admin
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    Admin->>Client: Vào mục Cài đặt -> Thùng lưu trữ dự án (Archived Projects) -> Bấm "Khôi phục"
    Client->>API: POST /projects/{project_id}/unarchive
    API->>API: Kiểm tra quyền Admin
    API->>DB: UPDATE Projects SET status = 'ACTIVE', updated_at = NOW() WHERE project_id = {id}
    DB-->>API: Success
    API-->>Client: HTTP 200 OK ("Khôi phục dự án thành công")
    Client-->>Admin: Dự án mở khóa quyền chỉnh sửa, xuất hiện trở lại trên Portfolio Dashboard
```

#### 5.4.6 Soft Delete Project
```mermaid
sequenceDiagram
    autonumber
    actor Admin as Workspace Admin
    participant Client as Web Client
    participant API as API Server
    participant DB as Database

    Admin->>Client: Bấm "Xóa vĩnh viễn/Xóa dự án"
    Client->>API: DELETE /projects/{project_id}
    API->>API: Kiểm tra quyền Workspace Admin
    API->>DB: UPDATE Projects SET is_deleted = true WHERE project_id = {id}
    DB-->>API: Success
    API-->>Client: HTTP 200 OK
    Client-->>Admin: Xóa hoàn toàn dự án khỏi giao diện người dùng
```

---

## 6. External Interface Requirements

### 6.1 User Interfaces (UI)
- **Kanban Board:** Kéo thả linh hoạt mượt mà, phản hồi màu sắc trạng thái (Xanh = Bình thường, Vàng = Gần chạm WIP, Đỏ = Đạt/Vượt trần WIP). Cảnh báo Icon tam giác đỏ nổi bật khi Task bị Blocked.
- **Task Detail Modal:** Bố cục linh hoạt tự động render các trường Custom Fields theo Type; kiểm tra định dạng tức thì trên Client (Client-side fast feedback) kết hợp Server-side validation.
- **Tree-view Table:** Hỗ trợ Expand/Collapse các Subtask mượt mà, hiển thị thanh tiến độ trực quan (% Progress Bar).

### 6.2 Software Interfaces
- **PostgreSQL Database:** Tối ưu hóa truy vấn EAV/JSONB qua extension `pg_trgm` hoặc GIN indexing trên trường JSONB.
- **WebSocket Gateway:** Giao tiếp kênh hai chiều thời gian thực để bắn các sự kiện: `TASK_MOVED`, `WIP_OVERRIDDEN`, `ASSIGNEE_HANDOFF`, `PROGRESS_UPDATED`.

---

## 7. Requirements Traceability Reference
Toàn bộ ma trận truy xuất từ Business Requirements -> Functional Requirements -> User Stories -> Test Cases đã được ánh xạ chi tiết và lưu trữ độc lập tại file [`03_RTM_PHASE2_OMNIPROJECT_CORE.csv`](./03_RTM_PHASE2_OMNIPROJECT_CORE.csv). Mọi thay đổi ở tài liệu này bắt buộc phải cập nhật đồng bộ sang RTM.
