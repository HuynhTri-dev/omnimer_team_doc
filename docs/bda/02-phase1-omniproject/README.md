# Phase 1: OmniProject & OmniChannel

Đây là thư mục chứa toàn bộ đặc tả chức năng (FRD) và Tiêu chí chấp nhận (AC) cho **Phase 1**. Nhằm hỗ trợ việc theo dõi và bảo trì dễ dàng (Vertical Slicing), các tài liệu ở đây được chia nhỏ theo từng Component và Feature độc lập. Mỗi Feature file chứa đầy đủ cả luật hệ thống (Backend) lẫn giao diện/Gherkin (Frontend/QA).

## 🗂 Cấu trúc Thành phần (Components)

### 1. [OmniProject Core](./omniproject_core/)
Nhóm tính năng quản lý dự án linh hoạt và lập lịch.
- **[FR-PRJ-000: Core Technical Architecture & Governance](./omniproject_core/FR-PRJ-000_core_technical_architecture.md)** (Hiến pháp kỹ thuật)
- [FR-PRJ-001: Multi-View Engine](./omniproject_core/FR-PRJ-001_multi_view_engine.md)
- [FR-PRJ-002: Dynamic Workflows & WIP Limits](./omniproject_core/FR-PRJ-002_dynamic_workflows_wip.md)
- [FR-PRJ-003: Dependency & Critical Path](./omniproject_core/FR-PRJ-003_dependency_critical_path.md)
- [FR-PRJ-004: Dynamic Custom Fields](./omniproject_core/FR-PRJ-004_dynamic_custom_fields.md)
- [FR-PRJ-005: Task Collaboration & Attachments](./omniproject_core/FR-PRJ-005_task_collaboration.md)
- [FR-PRJ-006: Subtask Management](./omniproject_core/FR-PRJ-006_subtask_management.md)
- [FR-PRJ-007: Smart Prioritization & Delegation Chain](./omniproject_core/FR-PRJ-007_smart_prioritization.md)
- [FR-PRJ-008: Strict Definition of Done & Output Contract](./omniproject_core/FR-PRJ-008_strict_dod_output.md)
- [FR-PRJ-009: Scrum & Sprint Management](./omniproject_core/FR-PRJ-009_scrum_sprint_management.md)

### 2. [OmniChannel](./omnichannel/)
Cổng giao tiếp 2 chiều hợp nhất (Telegram, Slack, Zalo OA), BYOS Storage & AI ChatOps Read-only.
- [**00_DEVELOPMENT_PHASES_ROADMAP.md (Lộ trình 6 Giai đoạn)**](./omnichannel/00_DEVELOPMENT_PHASES_ROADMAP.md)
- [README OmniChannel (Danh mục 6 Phase & 17 FRs)](./omnichannel/README.md)
- [BRD & User Stories (15 US + Risk Register)](./omnichannel/BRD_OMNICHANNEL_USER_STORIES.md)
- [Phase 1: Storage Foundation (BYOS) & Telegram Gateway (`FR-CHN-001, 002, 017.1`)](./omnichannel/phase_1/)
- [Phase 2: Unified 2-Way Inbox, Triage OCC & Internal Note (`FR-CHN-003..005`)](./omnichannel/phase_2/)
- [Phase 3: Message-to-Task, OPAgent Draft & Project Archival (`FR-CHN-006..008, 017.2..4`)](./omnichannel/phase_3/)
- [Phase 4: Slack/Zalo 48h, Auto-Notify & In-Chat Mention (`FR-CHN-009..011`)](./omnichannel/phase_4/)
- [Phase 5: Account Binding & Internal AI ChatOps 2A (`FR-CHN-012..014`)](./omnichannel/phase_5/)
- [Phase 6: External Client AI ChatOps 2B & NĐ 13 Compliance (`FR-CHN-015..016`)](./omnichannel/phase_6/)
- [Biểu đồ Trình tự Tổng quan (Sequence Diagrams)](./omnichannel/diagram_omnichannel_sequence.md)

### 3. [Agent Project (OPAgent)](./agent_project/)
Trợ lý AI bóc tách thông tin và cảnh báo rủi ro.
- [FR-AGT-001: NLP Text/Voice-to-Task](./agent_project/FR-AGT-001_nlp_text_to_task.md)
- [FR-AGT-002: Proactive Health Monitor](./agent_project/FR-AGT-002_proactive_health_monitor.md)

### 4. [Asset Governance](./asset_governance/)
Quản trị tri thức và lưu trữ tài liệu dự án tự động.
- [FR-DOC-001: Automated Asset Governance](./asset_governance/FR-DOC-001_automated_asset_governance.md)
