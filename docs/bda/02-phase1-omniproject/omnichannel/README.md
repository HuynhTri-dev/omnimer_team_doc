# Phân hệ OmniChannel — Danh mục Tài liệu Đặc tả theo 6 Phase Phát triển

Thư mục này chứa bộ tài liệu nghiệp vụ (**BRD & User Stories**), lộ trình phát triển (**Roadmap**) và đặc tả phần mềm chi tiết (**SRS IEEE 830, RTM CSV, FRD**) được tổ chức theo **6 Phase phát triển tuần tự (`phase_1/` → `phase_6/`)**, đồng bộ 100% với cấu trúc của `omniproject_core`.

## Tài liệu Tổng quan & Kế hoạch Kỹ thuật
- [**00_DEVELOPMENT_PHASES_ROADMAP.md**](./00_DEVELOPMENT_PHASES_ROADMAP.md) — Đồ thị phụ thuộc (DAG) & Work Breakdown Structure 6 Phase.
- [**07_omnichannel_implementation_changelog.md**](../../../../../../backend_otm/project_core/docs/change_log/07_omnichannel_implementation_changelog.md) — Kế hoạch triển khai kỹ thuật (19 Prisma Models, 6 Repositories, 5 Zod DTOs, Triad Lifecycle) & Phân công 5 AI Models (`Claude Opus`, `Gemini 4 Hydrogen`, `Neon`, `Gemini 4 Argon`, `Gemini Flash`).
- [BRD_OMNICHANNEL_USER_STORIES.md](./BRD_OMNICHANNEL_USER_STORIES.md) — Vision & Scope, Business Rules (`BR-CHN-001`..`009`), NFRs, 15 User Stories (`US-CHN-01`..`15`) và Risk Register.
- [diagram_omnichannel_sequence.md](./diagram_omnichannel_sequence.md) — Biểu đồ trình tự tổng quan xuyên suốt 4 luồng chính.

---

## Cấu trúc Thư mục theo 6 Phase Phát triển (`FR-CHN-001` → `FR-CHN-017`)

### 1. [Phase 1: Storage Foundation (BYOS) & Telegram Core Gateway](./phase_1/)
- [**01_SRS_PHASE1_STORAGE_GATEWAY.md**](./phase_1/01_SRS_PHASE1_STORAGE_GATEWAY.md) | [**02_RTM_PHASE1_STORAGE_GATEWAY.csv**](./phase_1/02_RTM_PHASE1_STORAGE_GATEWAY.csv)
- [FR-CHN-001: Channel Connection Management & Token Security](./phase_1/FR-CHN-001_channel_connection_management.md)
- [FR-CHN-002: Inbound Webhook Gateway (Signature, Idempotency, DLQ)](./phase_1/FR-CHN-002_inbound_webhook_gateway.md)
- *(Bao gồm `FR-CHN-017.1`: Pluggable Tenant Storage Adapter `MANAGED` / `BYOS`)*

### 2. [Phase 2: Unified 2-Way Inbox, Triage OCC, RBAC & Internal Note](./phase_2/)
- [**01_SRS_PHASE2_UNIFIED_INBOX.md**](./phase_2/01_SRS_PHASE2_UNIFIED_INBOX.md) | [**02_RTM_PHASE2_UNIFIED_INBOX.csv**](./phase_2/02_RTM_PHASE2_UNIFIED_INBOX.csv)
- [FR-CHN-003: 2-Way Messaging & Outbound Gateway](./phase_2/FR-CHN-003_2way_messaging_and_outbound.md)
- [FR-CHN-004: Conversation Triage, OCC, RBAC & Project Discussion](./phase_2/FR-CHN-004_conversation_triage_and_rbac.md)
- [FR-CHN-005: Internal Note Isolation & Server Hard Gate](./phase_2/FR-CHN-005_internal_note_isolation.md)

### 3. [Phase 3: Message-to-Task/Ticket, OPAgent Draft & Project Archival](./phase_3/)
- [**01_SRS_PHASE3_CONVERSION_ARCHIVAL.md**](./phase_3/01_SRS_PHASE3_CONVERSION_ARCHIVAL.md) | [**02_RTM_PHASE3_CONVERSION_ARCHIVAL.csv**](./phase_3/02_RTM_PHASE3_CONVERSION_ARCHIVAL.csv)
- [FR-CHN-006: Multi-Message Selection, Immutable Snapshot & Deep-Link](./phase_3/FR-CHN-006_multi_message_snapshot_deeplink.md)
- [FR-CHN-007: Support Project (`FR-SUP-000`) vs Delivery Project Conversion](./phase_3/FR-CHN-007_support_vs_delivery_conversion.md)
- [FR-CHN-008: OPAgent Draft Extraction, Timezone Anchor & PII Masking](./phase_3/FR-CHN-008_opagent_draft_and_pii_masking.md)
- [FR-CHN-017: Tenant Storage (BYOS), Important Message Preservation & Project Archival](./phase_3/FR-CHN-017_tenant_storage_byos_and_archival.md)

### 4. [Phase 4: Multi-Channel Expansion (Slack, Zalo OA 48h/ZNS), Auto-Notify & In-Chat Mention](./phase_4/)
- [**01_SRS_PHASE4_MULTICHANNEL_MENTIONS.md**](./phase_4/01_SRS_PHASE4_MULTICHANNEL_MENTIONS.md) | [**02_RTM_PHASE4_MULTICHANNEL_MENTIONS.csv**](./phase_4/02_RTM_PHASE4_MULTICHANNEL_MENTIONS.csv)
- [FR-CHN-009: Auto-Notification on Ticket/Task Lifecycle](./phase_4/FR-CHN-009_auto_status_notification.md)
- [FR-CHN-010: In-Chat `@member` & `#task` Mention with Public Reply Protection](./phase_4/FR-CHN-010_in_chat_member_task_mention.md)
- [FR-CHN-011: Task Chat Backlink Audit & Scoped Visibility](./phase_4/FR-CHN-011_task_chat_backlink_audit.md)

### 5. [Phase 5: Account Binding Governance & Internal AI ChatOps (Đợt 2A)](./phase_5/)
- [**01_SRS_PHASE5_BINDING_INTERNAL_CHATOPS.md**](./phase_5/01_SRS_PHASE5_BINDING_INTERNAL_CHATOPS.md) | [**02_RTM_PHASE5_BINDING_INTERNAL_CHATOPS.csv**](./phase_5/02_RTM_PHASE5_BINDING_INTERNAL_CHATOPS.csv)
- [FR-CHN-012: Account Binding via Short-Lived OTP / QR](./phase_5/FR-CHN-012_account_binding_otp_qr.md)
- [FR-CHN-013: Account Binding Governance, Customer Whitelist & Instant Revocation](./phase_5/FR-CHN-013_binding_governance_revocation.md)
- [FR-CHN-014: Read-Only AI ChatOps for Internal Staff (Đợt 2A)](./phase_5/FR-CHN-014_internal_ai_chatops_readonly.md)

### 6. [Phase 6: External Client AI ChatOps (Đợt 2B), NĐ 13 Compliance & Hardening](./phase_6/)
- [**01_SRS_PHASE6_CLIENT_CHATOPS_COMPLIANCE.md**](./phase_6/01_SRS_PHASE6_CLIENT_CHATOPS_COMPLIANCE.md) | [**02_RTM_PHASE6_CLIENT_CHATOPS_COMPLIANCE.csv**](./phase_6/02_RTM_PHASE6_CLIENT_CHATOPS_COMPLIANCE.csv)
- [FR-CHN-015: Read-Only AI ChatOps for External Clients & Prompt Injection Defense (Đợt 2B)](./phase_6/FR-CHN-015_external_client_ai_chatops_readonly.md)
- [FR-CHN-016: Data Retention, Soft-Delete Trash & NĐ 13/2023/NĐ-CP Compliance](./phase_6/FR-CHN-016_data_retention_privacy_nd13.md)
