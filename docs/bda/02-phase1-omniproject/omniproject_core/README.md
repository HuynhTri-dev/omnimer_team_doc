# OmniProject Core — Business & Domain Architecture (BDA)

Chào mừng đến với hệ thống tài liệu Đặc tả Yêu cầu Nghiệp vụ (FR) của **OmniProject**. Thư mục này chứa toàn bộ thiết kế tính năng cho phân hệ cốt lõi của sản phẩm, được thiết kế theo chuẩn Enterprise SaaS (hỗ trợ Multi-tenant, RBAC, Real-time Sync).

Hệ thống bao gồm **24 Phân hệ Tính năng lớn** (cùng 1 tài liệu Kiến trúc cốt lõi & 1 tài liệu Từ điển thuật ngữ) với tổng cộng **107 Tính năng nhỏ (Sub-features)** được thiết kế chi tiết theo từng cấp độ **Macro to Micro**:

---

## 🗺️ Lộ trình Triển khai Lập trình (Development Phases)
> Xem chi tiết kế hoạch phân chia 6 giai đoạn phát triển từ Nền tảng tới Nâng cao tại: [**00_DEVELOPMENT_PHASES_ROADMAP.md**](./00_DEVELOPMENT_PHASES_ROADMAP.md).

---

## 📚 0. Thuật ngữ cốt lõi (Domain Terminology)
Trước khi đọc các tài liệu bên dưới, hãy tham khảo [**00_domain_terminology.md**](./00_domain_terminology.md) để hiểu rõ định nghĩa về Organization, Workspace, Portfolio, Project, Sprint, và Task.

---

## 🏢 1. Level 1: Organization & Identity (Quản trị Tổ chức & Danh tính)
> **Tài liệu tổng hợp (Phase 1):** Xem bản đặc tả phần mềm chuẩn IEEE 830 tại [**01_SRS_PHASE1_FOUNDATION.md**](./01_SRS_PHASE1_FOUNDATION.md) và Ma trận truy vết yêu cầu (CSV) tại [**02_RTM_PHASE1_FOUNDATION.csv**](./02_RTM_PHASE1_FOUNDATION.csv).


### * [**FR-ORG-001: Workspace, Organization Management & Billing**](./FR-ORG-001_workspace_organization_management.md) — *(5 tính năng con)*
  - **FR-ORG-001.1 - Tạo Organization & Workspace:** Khi user đăng ký mới, hệ thống tự động tạo 1 Organization và 1 Default Workspace. User trở thành Org Owner.
  - **FR-ORG-001.2 - Quản lý Cấu hình Workspace:** Cho phép Workspace Admin chỉnh sửa tên, logo, timezone mặc định, và format ngày/giờ cho toàn bộ Workspace.
  - **FR-ORG-001.3 - Subscription & Billing:** Quản lý gói cước (Free, Pro, Enterprise). Hiển thị số lượng seat (tài khoản) đang sử dụng, chu kỳ thanh toán, và xuất hóa đơn VAT.
  - **FR-ORG-001.4 - Multi-Workspace Switcher:** Cho phép 1 User có thể tham gia nhiều Workspace khác nhau và dễ dàng chuyển đổi qua lại từ Menu góc trái màn hình.
  - **FR-ORG-001.5 - Audit Log (Cấp độ Org):** Lưu vết toàn bộ các hành động mang tính quản trị (Tạo project mới, xóa project, đổi gói cước, xuất dữ liệu hàng loạt).

### * [**FR-ORG-002: Identity, Access Management (IAM), SSO & Invites**](./FR-ORG-002_identity_access_management.md) — *(5 tính năng con)*
  - **FR-ORG-002.1 - Mời thành viên (Invite Members):** Cho phép Workspace Admin gửi email mời người dùng mới vào Workspace. Có thể chọn trước Role cấp Workspace.
  - **FR-ORG-002.2 - Quản lý Directory (Member List):** Màn hình hiển thị toàn bộ thành viên trong Workspace. Cho phép Admin tìm kiếm, lọc, thay đổi Role, hoặc vô hiệu hóa (Deactivate) tài khoản.
  - **FR-ORG-002.3 - Quản lý Nhóm (User Groups):** Cho phép tạo các Nhóm người dùng (Ví dụ: "Backend Team", "Designers"). Nhóm này có thể được dùng để `assignee` hoặc `@mention` hàng loạt.
  - **FR-ORG-002.4 - Cấu hình SSO (SAML/OIDC):** Màn hình cho phép Admin điền các thông tin IdP (Identity Provider) như ACS URL, Entity ID, x509 Certificate để bật SSO cho Workspace.
  - **FR-ORG-002.5 - Khóa tài khoản tự động (Auto-Lock):** Tự động khóa tài khoản hoặc buộc đổi mật khẩu nếu phát hiện đăng nhập sai quá 5 lần (Brute-force protection).

## 📊 2. Level 2: Cross-Project & Analytics (Quản trị Liên dự án & Báo cáo)

### * [**FR-PRJ-010: Portfolio & Multi-Project Dashboard**](./phase_6/FR-PRJ-010_portfolio_multi_project.md) — *(5 tính năng con)*
  - **FR-PRJ-010.1 - Portfolio Board:** Hiển thị tất cả Projects mà user có quyền truy cập dưới dạng thẻ (Card) hoặc danh sách (List), kèm trạng thái tổng thể (On Track / At Risk / Off Track), % tiến độ và ngày deadline gần nhất.
  - **FR-PRJ-010.2 - Cross-Project Timeline (Program Gantt):** Hiển thị Gantt Chart cấp Portfolio, mỗi hàng là 1 Project, các mốc Sprint/Milestone hiển thị trên timeline chung.
  - **FR-PRJ-010.3 - Risk Radar tổng hợp:** Tổng hợp và hiển thị danh sách Tasks/Projects đang "At Risk" (quá hạn, blocked, hoặc gần deadline < 48h) trên toàn bộ portfolio.
  - **FR-PRJ-010.4 - Project Health Score:** Hệ thống tự động tính điểm sức khỏe (0–100) cho từng project dựa trên: % task đúng hạn, % task bị blocked, tốc độ hoàn thành.
  - **FR-PRJ-010.5 - Milestone Tracking:** Cho phép tạo Milestone (mốc quan trọng) gắn vào Project, hiển thị trên Program Gantt và Portfolio card.

### * [**FR-PRJ-011: Resource & Capacity Management (Workload)**](./phase_6/FR-PRJ-011_resource_capacity.md) — *(5 tính năng con)*
  - **FR-PRJ-011.1 - Workload View:** Hiển thị tổng số giờ/số Task được giao cho mỗi thành viên trong một khoảng thời gian (tuần/sprint). Cho phép so sánh với capacity tối đa đã cấu hình (VD: 40h/tuần).
  - **FR-PRJ-011.2 - Availability Calendar:** Mỗi thành viên tự đăng ký thời gian nghỉ (Leave), giảm capacity (Partial). Hệ thống trừ ngày nghỉ vào tổng capacity khi PM lên kế hoạch.
  - **FR-PRJ-011.3 - Cảnh báo Overload:** Hệ thống tự động cảnh báo (badge đỏ) khi tổng workload của thành viên vượt quá 100% capacity trong bất kỳ tuần nào.
  - **FR-PRJ-011.4 - Cross-Project Workload:** Tổng hợp workload của 1 thành viên xuyên suốt tất cả Projects họ tham gia (không chỉ 1 project).
  - **FR-PRJ-011.5 - Re-assign gợi ý:** Khi một thành viên overload, hệ thống gợi ý danh sách thành viên khác còn capacity để PM xem xét chuyển Task.

### * [**FR-PRJ-012: Reporting & Analytics (Velocity, Burndown, Cycle Time)**](./phase_6/FR-PRJ-012_reporting_analytics.md) — *(5 tính năng con)*
  - **FR-PRJ-012.1 - Burndown Chart:** Hiển thị đồ thị số lượng Story Points (hoặc số Task) còn lại theo ngày trong Sprint. So sánh đường thực tế (Actual) với đường lý tưởng (Ideal).
  - **FR-PRJ-012.2 - Velocity Chart:** Biểu đồ cột hiển thị tổng Story Points hoàn thành (Done) qua mỗi Sprint. Tính Velocity trung bình trong 3 Sprint gần nhất (rolling average).
  - **FR-PRJ-012.3 - Cycle Time Distribution:** Biểu đồ phân phối (histogram hoặc scatter) thời gian từ khi Task vào `IN_PROGRESS` đến khi `DONE`.
  - **FR-PRJ-012.4 - Throughput Chart:** Biểu đồ số lượng Task hoàn thành mỗi ngày/tuần trong một khoảng thời gian.
  - **FR-PRJ-012.5 - Export Báo cáo:** Xuất dữ liệu báo cáo (bảng Task, log thời gian, burndown data) sang định dạng CSV hoặc PDF.

### * [**FR-PRJ-019: Global Search & Advanced Filtering (Cmd+K, OQL)**](./phase_6/FR-PRJ-019_global_search_filtering.md) — *(4 tính năng con)*
  - **FR-PRJ-019.1 - Quick Search (Cmd+K):** Thanh tìm kiếm nhanh dạng pop-up ở mọi màn hình. Hỗ trợ tìm kiếm realtime (gõ tới đâu hiện tới đó).
  - **FR-PRJ-019.2 - Full-text Search:** Tìm kiếm nội dung bên trong Description, Comments, và tên file đính kèm. Hỗ trợ highlight từ khóa trùng khớp.
  - **FR-PRJ-019.3 - Advanced Filtering (OQL):** Cung cấp giao diện lọc nhiều lớp (Multi-layer filter) tại các View. Hỗ trợ Omni Query Language (OQL) dạng gõ text, VD: `assignee:me AND status:!done OR due_date:<today`.
  - **FR-PRJ-019.4 - Saved Filters:** Cho phép user lưu lại bộ lọc thường dùng (Ví dụ: "Task trễ hạn của team Dev") thành các Quick Tabs để bấm một phát ăn ngay.

## 📁 3. Level 3: Project Lifecycle & Governance (Vòng đời & Quy trình Dự án)

### * [**FR-PRJ-018: Project Lifecycle & Templates (Khởi tạo, Archive)**](./FR-PRJ-018_project_lifecycle_templates.md) — *(5 tính năng con)*
  - **FR-PRJ-018.1 - Tạo mới Project (Khởi tạo):** Cho phép Workspace Admin hoặc PM tạo dự án mới. Yêu cầu nhập Tên, Mô tả, Ngày bắt đầu/kết thúc dự kiến và gán Members ban đầu.
  - **FR-PRJ-018.2 - Sử dụng Project Templates:** Khi tạo Project, cung cấp danh sách Template (VD: "Mẫu Phát triển Phần mềm", "Mẫu Marketing Campaign"). Khi chọn, hệ thống tự động copy toàn bộ cấu trúc: Custom Workflow, Custom Fields, DoD config, và các Task mẫu.
  - **FR-PRJ-018.3 - Lưu Project thành Template:** Cho phép PM lưu lại một cấu trúc dự án đang chạy thành Template dùng chung cho toàn Workspace để tái sử dụng sau này.
  - **FR-PRJ-018.4 - Archive / Đóng Project:** Khi dự án kết thúc, PM có thể chuyển trạng thái sang `ARCHIVED`. Dự án sẽ bị khóa `Read-only` và ẩn khỏi màn hình chính để không gây nhiễu.
  - **FR-PRJ-018.5 - Khôi phục Project (Unarchive):** Cho phép Admin tìm lại các dự án đã Archive và mở lại (chuyển về `ACTIVE`) nếu có nhu cầu phát sinh.

### * [**FR-PRJ-000: Core Technical Architecture, NFRs & RBAC/RACI Matrix**](./FR-PRJ-000_core_technical_architecture.md) — *(Kiến trúc Nền tảng & Tiêu chuẩn)*
  - **RBAC Matrix & RACI:** Ma trận phân quyền 4 cấp vai trò và bảng phân định trách nhiệm RACI cho Task.
  - **Real-time Sync & Conflict Resolution:** Kiến trúc WebSocket/SSE, Event Sourcing, Last-Write-Wins và Offline Queue.
  - **Non-Functional Requirements (NFRs):** SLA phản hồi <100ms, Uptime 99.9%, mã hóa AES-256 / TLS 1.3.

### * [**FR-PRJ-002: Dynamic Workflow & WIP Limits**](./FR-PRJ-002_dynamic_workflows_wip.md) — *(4 tính năng con)*
  - **FR-PRJ-002.1 - Định nghĩa Workflow (Custom Workflow):** Cho phép Project Manager tạo và cấu hình các trạng thái (Status) tùy chỉnh cho luồng công việc (VD: To Do -> Dev -> Test -> Done) và thiết lập giới hạn WIP cho từng trạng thái.
  - **FR-PRJ-002.2 - Quy trình Giao việc & Chuyển giao (Assignment & Handoff):** Khi chuyển Task sang một trạng thái mới (VD: từ Dev sang Test), hệ thống hỗ trợ tự động gợi ý/yêu cầu cập nhật người phụ trách (Assignee) phù hợp với vòng đời đó.
  - **FR-PRJ-002.3 - Cảnh báo giới hạn WIP (WIP Block):** Hệ thống ngăn chặn hoặc cảnh báo khi kéo Task vào một trạng thái (Cột) đã đạt ngưỡng giới hạn WIP (Work In Progress).
  - **FR-PRJ-002.4 - Vượt rào WIP (WIP Override):** Cấp quyền cho Project Manager (hoặc role được ủy quyền) xác nhận bỏ qua giới hạn WIP trong trường hợp khẩn cấp.

### * [**FR-PRJ-008: Strict DoD (Definition of Done) & Output Contract**](./FR-PRJ-008_strict_dod_output.md) — *(3 tính năng con)*
  - **FR-PRJ-008.1 - Thiết lập Output Contract:** Khi tạo/sửa Task, Project Manager hoặc Reporter có thể chọn định dạng đầu ra bắt buộc (VD: File đính kèm, Link Figma, Link Pull Request).
  - **FR-PRJ-008.2 - Chặn hoàn thành (Done Blocker):** Ngăn chặn người dùng chuyển trạng thái Task sang `Done` nếu chưa cung cấp tài liệu đầu ra đúng định dạng đã yêu cầu.
  - **FR-PRJ-008.3 - Tự động lưu trữ (Asset Sync):** Khi Task hoàn thành hợp lệ, tài liệu đầu ra được copy một bản (Event-driven) sang hệ thống Asset Governance (Kho tri thức) để bảo lưu dài hạn.

### * [**FR-PRJ-009: Scrum & Sprint Management (Optional Module)**](./FR-PRJ-009_scrum_sprint_management.md) — *(4 tính năng con)*
  - **FR-PRJ-009.1 - Sprint Planning:** Cho phép tạo Sprint mới, định nghĩa mục tiêu (Goal) và khoảng thời gian (Start Date - End Date).
  - **FR-PRJ-009.2 - Kéo thả Task vào Sprint:** Kéo các Task từ Product Backlog vào các Sprint `PLANNED` hoặc `ACTIVE` để lên kế hoạch thực hiện.
  - **FR-PRJ-009.3 - Start Sprint:** Bắt đầu một Sprint. Hệ thống sẽ khóa các thay đổi lớn và bắt đầu tính toán báo cáo (Burndown Chart).
  - **FR-PRJ-009.4 - Complete Sprint & Rollover:** Đóng một Sprint khi hết thời gian. Xử lý các Task chưa hoàn thành bằng cách đẩy chúng về Backlog hoặc đẩy sang Sprint kế tiếp (Rollover).

## ✅ 4. Level 4: Task Execution & Collaboration (Thực thi & Phối hợp)

### * [**FR-PRJ-001: Multi-View Engine (Kanban, Gantt, Table)**](./FR-PRJ-001_multi_view_engine.md) — *(4 tính năng con)*
  - **FR-PRJ-001.1 - Chuyển đổi View (View Switcher):** Hệ thống cho phép người dùng chuyển đổi qua lại giữa 4 góc nhìn: Kanban, Gantt, Table, Scrum mà không làm thay đổi dữ liệu gốc.
  - **FR-PRJ-001.2 - Đồng bộ Dữ liệu thời gian thực (View Sync):** Mọi thay đổi dữ liệu trên một View (VD: Kéo thả trên Kanban) phải được cập nhật tức thì tới các client đang mở View khác (VD: Gantt).
  - **FR-PRJ-001.3 - Bảo lưu Trạng thái (View State Preservation):** Hệ thống phải giữ nguyên các bộ lọc (Filter) và sắp xếp (Sort) khi người dùng chuyển đổi View.
  - **FR-PRJ-001.4 - Admin Radar (Giám sát cảnh báo):** Hệ thống tự động làm nhấp nháy đỏ các thẻ Task có tính chất khẩn cấp hoặc đang bị "Blocked".

### * [**FR-PRJ-016: Calendar View**](./FR-PRJ-016_calendar_view.md) — *(5 tính năng con)*
  - **FR-PRJ-016.1 - Calendar View Mode:** Thêm tab "Calendar" vào View Switcher (FR-PRJ-001). Hỗ trợ 3 chế độ hiển thị: **Month** (lịch tháng), **Week** (lịch tuần), **Day** (lịch ngày).
  - **FR-PRJ-016.2 - Task trên Calendar:** Mỗi Task hiển thị dưới dạng "sự kiện" (Event chip) trên ngày `due_date`. Task kéo dài nhiều ngày (có `start_date` và `end_date`) hiển thị như event block trải dài.
  - **FR-PRJ-016.3 - Kéo thả Task trên Calendar:** Cho phép kéo Task từ ngày này sang ngày khác để thay đổi `due_date` (hoặc `end_date`). Tuân thủ RBAC giống Gantt View (FR-PRJ-000).
  - **FR-PRJ-016.4 - Quick-create trên Calendar:** Click trực tiếp vào ô ngày để tạo Task mới với `due_date` mặc định là ngày được click.
  - **FR-PRJ-016.5 - Cross-project Calendar:** Trong màn hình "My Tasks" + Calendar View, hiển thị Task từ TẤT CẢ Projects của user trên cùng 1 lịch, phân biệt màu theo Project.

### * [**FR-PRJ-003: Dependency & Critical Path (Đường găng)**](./FR-PRJ-003_dependency_critical_path.md) — *(4 tính năng con)*
  - **FR-PRJ-003.1 - Thiết lập Phụ thuộc (Dependencies):** Cho phép nối các Task theo 4 loại: FS (Finish-to-Start), SS (Start-to-Start), FF (Finish-to-Finish), SF (Start-to-Finish). Có thể thiết lập độ trễ (Lag days).
  - **FR-PRJ-003.2 - Chống Vòng lặp (Cycle Detection):** Ngăn chặn người dùng tạo vòng lặp phụ thuộc (A -> B -> C -> A).
  - **FR-PRJ-003.3 - Auto-Scheduling (Lan truyền):** Khi thay đổi ngày của Task tiền nhiệm, các Task hậu nhiệm liên quan tự động dịch chuyển tương ứng theo kiểu liên kết.
  - **FR-PRJ-003.4 - Tính toán Đường Găng (Critical Path):** Nút bật/tắt (Toggle) hiển thị đường găng trên Gantt Chart. Làm nổi bật các Task không có thời gian dự trữ (Total Float = 0).

### * [**FR-PRJ-004: Dynamic Custom Fields (EAV Model)**](./FR-PRJ-004_dynamic_custom_fields.md) — *(4 tính năng con)*
  - **FR-PRJ-004.1 - Tạo Trường tùy chỉnh:** Cho phép định nghĩa Custom Field mới với các kiểu dữ liệu: `TEXT`, `NUMBER`, `CURRENCY`, `DATE`, `DROPDOWN`, `CHECKBOX`.
  - **FR-PRJ-004.2 - Quản lý giá trị Field:** Cho phép nhập và hiển thị giá trị của Custom Field trên UI của Task Modal, Kanban (trên thẻ Task), và Table View.
  - **FR-PRJ-004.3 - Validation linh hoạt:** Project Manager có thể thiết lập cờ `Required`, `Min/Max` (với số), hoặc tập giá trị cho phép (Drop-down options).
  - **FR-PRJ-004.4 - Lọc và Sắp xếp:** Người dùng có thể Filter (Lọc) và Sort (Sắp xếp) danh sách Task theo các Custom Field vừa tạo.

### * [**FR-PRJ-005: Task Collaboration (Comments, @mentions, Activity Log)**](./FR-PRJ-005_task_collaboration.md) — *(4 tính năng con)*
  - **FR-PRJ-005.1 - Mô tả chi tiết (Rich Text / Markdown):** Hỗ trợ viết mô tả Task bằng định dạng Markdown (headings, lists, code blocks, bold/italic, tables). Hỗ trợ preview trực tiếp.
  - **FR-PRJ-005.2 - Quản lý Đính kèm (Attachments):** Cho phép upload file (PDF, Image, Excel) trực tiếp vào Task. File được quản lý chung trong hệ thống Asset Governance.
  - **FR-PRJ-005.3 - Bình luận & Tag (Mentions):** Hỗ trợ thảo luận đa chiều, người dùng có thể gõ `@username` để gọi tên người khác vào Task.
  - **FR-PRJ-005.4 - Nhật ký hoạt động (Activity Log):** Tự động ghi lại toàn bộ lịch sử thay đổi của Task (VD: Ai đổi Status, đổi Assignee, đổi Deadline, thêm Subtask) kèm theo timestamp.

### * [**FR-PRJ-006: Subtask Management (Parent-Child Task)**](./FR-PRJ-006_subtask_management.md) — *(4 tính năng con)*
  - **FR-PRJ-006.1 - Tạo và Quản lý Subtask:** Cho phép người dùng tạo, sửa, xóa Subtask bên trong một Task. Subtask có thể được gán cho người khác (Assignee khác với Task cha) và có Deadline riêng.
  - **FR-PRJ-006.2 - Cập nhật trạng thái Subtask:** Người thực thi có thể đánh dấu hoàn thành (Done) hoặc cập nhật trạng thái của Subtask độc lập với Task cha.
  - **FR-PRJ-006.3 - Progress Roll-up (Cộng dồn tiến độ):** Hệ thống tự động tính toán tiến độ hoàn thành (%) của Task cha dựa trên số lượng Subtask đã Done.
  - **FR-PRJ-006.4 - Subtask View Mode:** Trên màn hình List/Table, người dùng có thể mở rộng (Expand) Task cha để xem danh sách Subtasks ở dạng cây (Tree-view).

### * [**FR-PRJ-007: Smart Prioritization (Action Score / Auto-sort)**](./FR-PRJ-007_smart_prioritization.md) — *(4 tính năng con)*
  - **FR-PRJ-007.1 - Giao việc phân quyền (Delegation):** Hỗ trợ mô hình 2 vai trò trên một Task: `Owner` (Người chịu trách nhiệm cuối cùng) và `Assignee` (Người trực tiếp thực thi).
  - **FR-PRJ-007.2 - Tính toán Action Score:** Hệ thống tự động tính toán điểm ưu tiên của mỗi Task dựa trên độ quan trọng và thời gian còn lại (Deadline).
  - **FR-PRJ-007.3 - Tự động sắp xếp (Smart Sort):** Màn hình "My Work" / Kanban cột To-Do tự động sắp xếp danh sách Task từ trên xuống dưới theo Action Score giảm dần.
  - **FR-PRJ-007.4 - Bảo vệ Deadline (Anti-Gaming):** Khóa quyền thay đổi Due Date của `Assignee` để tránh tình trạng cố tình lùi Deadline làm giảm Action Score.

### * [**FR-PRJ-017: Recurring Tasks (Cron/RRULE)**](./FR-PRJ-017_recurring_tasks.md) — *(4 tính năng con)*
  - **FR-PRJ-017.1 - Thiết lập Recurrence:** Khi tạo hoặc sửa Task, user có thể bật chế độ "Repeat" và cấu hình lịch lặp: Daily / Weekly (chọn ngày trong tuần) / Monthly (ngày cố định) / Custom (cron-style).
  - **FR-PRJ-017.2 - Tự động tạo Task mới khi Done:** Khi Task có recurrence được chuyển sang `DONE`, hệ thống tự động tạo instance Task mới cho chu kỳ tiếp theo, giữ nguyên: Title, Assignee, Custom Fields, DoD config.
  - **FR-PRJ-017.3 - Quản lý chuỗi (Series Management):** Cho phép user chỉnh sửa "Chỉ Task này" hoặc "Task này và tất cả Task sau" trong chuỗi. Tương tự mô hình của Google Calendar.
  - **FR-PRJ-017.4 - Dừng chuỗi (End Recurrence):** Cho phép đặt ngày kết thúc chuỗi lặp (`end_date`) hoặc số lần lặp tối đa (`max_occurrences`). Khi đến điều kiện kết thúc, không tạo thêm instance mới.

## ⚡ 5. Level 5: Utilities & Personal Productivity (Tiện ích & Tự động hóa)

### * [**FR-PRJ-015: Personal Space & My Tasks (Không gian cá nhân)**](./FR-PRJ-015_personal_space.md) — *(5 tính năng con)*
  - **FR-PRJ-015.1 - Personal Space tự động:** Khi user đăng ký tài khoản, hệ thống tự động tạo 1 Project đặc biệt "My Space" thuộc về user đó và chỉ mình user đó thấy. Không thể xóa Personal Space.
  - **FR-PRJ-015.2 - My Tasks Dashboard:** Màn hình tập trung hiển thị TẤT CẢ các Task đang được giao cho user, xuyên suốt mọi Projects (bao gồm cả Personal Space), được nhóm theo: Hôm nay / Tuần này / Sau đó / Không có deadline.
  - **FR-PRJ-015.3 - Quick-add Task:** Cho phép user thêm Task nhanh từ bất kỳ màn hình nào bằng phím tắt `Q` hoặc nút `+` nổi (Floating Action Button), không cần mở Project. Task mới mặc định được thêm vào Personal Space.
  - **FR-PRJ-015.4 - Private Tasks:** Cho phép đánh dấu Task là Private trong bất kỳ Project nào. Private Task chỉ hiển thị với chính user đó và Admin, ẩn với mọi thành viên khác kể cả PM.
  - **FR-PRJ-015.5 - "My Tasks" Smart Sort:** Tích hợp cơ chế Smart Prioritization (FR-PRJ-007) vào "My Tasks" để tự động sắp xếp task theo Action Score, kết hợp filter nhanh: Hôm nay / Sắp tới / Đã xong / Không có hạn.

### * [**FR-PRJ-013: Notification Center & Integrations (Slack/Teams, Webhooks)**](./FR-PRJ-013_notification_center.md) — *(5 tính năng con)*
  - **FR-PRJ-013.1 - Notification Inbox (In-app):** Hiển thị danh sách tất cả thông báo của user trong ứng dụng (Chuông 🔔), gồm: Mention, Task giao mới, Deadline sắp đến, Task bị Blocked, Sprint bắt đầu/kết thúc. Hỗ trợ đánh dấu đã đọc và xóa.
  - **FR-PRJ-013.2 - Notification Preference:** Mỗi user tự cấu hình loại thông báo nào nhận qua kênh nào (In-app / Email / Slack / Teams). Hỗ trợ tắt hoàn toàn từng loại thông báo.
  - **FR-PRJ-013.3 - Email Digest:** Thay vì gửi email từng cái một, hệ thống gom (batch) các thông báo non-urgent trong ngày thành 1 email Digest (digest 8h sáng và 18h chiều).
  - **FR-PRJ-013.4 - Slack / Teams Webhook:** Cho phép PM cấu hình Webhook URL vào channel Slack/Teams để nhận thông báo cấp dự án (Task Blocked, Sprint bắt đầu, Risk Alert) tự động.
  - **FR-PRJ-013.5 - Do Not Disturb (DND):** User cài đặt khoảng giờ yên tĩnh (VD: 22:00–07:00). Trong khung giờ này, thông báo non-urgent được giữ lại và gửi sau.

### * [**FR-PRJ-014: Time Tracking (Timer & Manual Log)**](./FR-PRJ-014_time_tracking.md) — *(5 tính năng con)*
  - **FR-PRJ-014.1 - Bấm giờ (Timer):** Cung cấp nút Start/Stop Timer trên Task Modal. Khi bấm Start, hệ thống đếm thời gian real-time. Khi Stop, tự động tạo Time Log entry cho khoảng thời gian vừa làm.
  - **FR-PRJ-014.2 - Nhập thủ công (Manual Log):** Cho phép nhập thời gian đã làm (VD: 2h 30m) trực tiếp mà không cần dùng Timer. Hỗ trợ nhập cho ngày trong quá khứ (tối đa 30 ngày).
  - **FR-PRJ-014.3 - Xem tổng Logged Time:** Hiển thị trên Task Modal: tổng giờ đã log, so sánh với `estimated_hours` (% tiến độ dựa trên giờ).
  - **FR-PRJ-014.4 - Timesheet View:** Màn hình tổng hợp thời gian làm việc của user trong tuần, nhóm theo ngày và dự án. Cho phép PM xem timesheet của toàn team.
  - **FR-PRJ-014.5 - Time Report xuất khẩu:** Xuất Timesheet ra CSV/Excel để tính lương, invoice khách hàng.

### * [**FR-PRJ-020: No-Code Automations (Rule Engine: If-This-Then-That)**](./FR-PRJ-020_nocode_automations.md) — *(5 tính năng con)*
  - **FR-PRJ-020.1 - Automation Builder (UI):** Giao diện kéo thả cho phép chọn Trigger (Khi nào), Condition (Điều kiện), và Action (Làm gì).
  - **FR-PRJ-020.2 - Triggers (Sự kiện kích hoạt):** Hỗ trợ các Trigger: Task created, Status changed, Due date approaches (trước 1 ngày/1 giờ), Custom Field updated.
  - **FR-PRJ-020.3 - Conditions (Điều kiện lọc):** Hỗ trợ check điều kiện: Trạng thái hiện tại là gì, Assignee là ai, Custom field giá trị bao nhiêu.
  - **FR-PRJ-020.4 - Actions (Hành động thực thi):** Hỗ trợ: Đổi Status, Gán người (Assign), Add Comment, Set Due Date, Gửi tin nhắn Slack/Teams, Gửi Email.
  - **FR-PRJ-020.5 - Automation Logs:** Giao diện xem lịch sử chạy của các Rule (Thành công/Thất bại), báo lỗi chi tiết nếu Action fail (VD: Slack token hết hạn).

### * [**FR-PRJ-021: Public Forms & Task Intake (Cổng tiếp nhận yêu cầu)**](./FR-PRJ-021_public_forms_intake.md) — *(5 tính năng con)*
  - **FR-PRJ-021.1 - Form Builder:** Màn hình kéo thả để PM xây dựng Form. Có thể map các trường trong Form với các trường mặc định của Task (Name, Description) hoặc Custom Fields.
  - **FR-PRJ-021.2 - Form Sharing (Public Link):** Sinh ra một đường link URL công khai (hoặc mã iframe để nhúng vào website công ty) không cần đăng nhập.
  - **FR-PRJ-021.3 - Task Auto-Creation:** Khi người dùng bên ngoài nhấn Submit form, hệ thống tự động sinh ra một Task nằm ở cột mặc định (thường là cột đầu tiên - Backlog/Inbox) của dự án.
  - **FR-PRJ-021.4 - Spam Protection:** Tích hợp reCAPTCHA v3 ẩn vào Public Form để ngăn chặn bot spam tạo rác vào hệ thống.
  - **FR-PRJ-021.5 - File Upload trên Form:** Cho phép người điền form đính kèm file (VD: ảnh chụp màn hình lỗi). File này sẽ tự động gắn vào Attachments của Task.

## 🔄 6. Phụ lục: Giai đoạn 2 (Phase 2)

### * [**FR-PRJ-022: Data Import, Export & Migration (Trello/Jira)**](./phase_6/FR-PRJ-022_data_import_export.md) — *(4 tính năng con)*
  - **FR-PRJ-022.1 - CSV/Excel Import:** Cho phép PM tải lên một file CSV. Cung cấp UI Mapping (Khớp cột CSV với các trường dữ liệu của Task như Tên, Assignee, Ngày đến hạn).
  - **FR-PRJ-022.2 - Trello Importer:** Nhập trực tiếp dữ liệu từ Trello Board qua API (Sử dụng OAuth để xác thực). Đảm bảo giữ nguyên các Cột (Lists), Labels, và Members nếu trùng email.
  - **FR-PRJ-022.3 - Jira Importer:** Tương tự Trello nhưng phức tạp hơn do phải map Issue Types (Epic, Story, Bug) thành cấu trúc Task/Subtask của OmniProject.
  - **FR-PRJ-022.4 - Full Project Export:** Nút cho phép PM tải về toàn bộ dự án dưới định dạng CSV, JSON. Kèm theo link nén (ZIP) các file đính kèm.
