# Sequence Diagrams: OmniChannel (Phase 1)

Tài liệu mô tả chi tiết luồng xử lý kỹ thuật (bao gồm Happy Path và Alternate/Exception Flows) cho 4 nhóm nghiệp vụ chính của phân hệ OmniChannel, bám sát [BRD_OMNICHANNEL_USER_STORIES.md](file:///Users/macbookair/Downloads/source/omnimer_team/omnimer_team_doc/docs/bda/02-phase1-omniproject/omnichannel/BRD_OMNICHANNEL_USER_STORIES.md).

---

## 1. Luồng Inbound Webhook & Outbound 2 Chiều (`FR-CHN-002`, `FR-CHN-003`, `FR-CHN-005`)

```mermaid
sequenceDiagram
    autonumber
    actor Ext as Khách hàng (Telegram/Slack/Zalo OA)
    participant GW as OmniChannel Gateway
    participant Redis as Redis (Idempotency & Cache)
    participant Queue as RabbitMQ / DLQ
    participant Store as Tenant Storage (Managed/BYOS)
    participant UI as Unified Inbox UI
    actor Staff as Team Member

    %% Inbound Flow
    Ext->>GW: Gửi tin nhắn / Webhook POST
    GW->>GW: Xác thực Signature / Secret Header
    alt Chữ ký không hợp lệ
        GW-->>Ext: HTTP 401/403 (Ghi log bảo mật)
    else Chữ ký hợp lệ
        GW->>Redis: Check Key (tenant_id + channel + message_id)
        alt Đã tồn tại (Retry trùng lặp)
            GW-->>Ext: HTTP 200 OK (Skip)
        else Tin nhắn mới
            GW->>Queue: Enqueue Normalized Event
            GW-->>Ext: HTTP 200 OK (< 100ms)
            Queue->>Store: Worker tải Media -> Lưu Tenant Storage (SHA-256)
            Store-->>UI: Lưu DB + Push WebSocket (p95 < 3s)
        end
    end

    %% Outbound Flow & Internal Note Hard Gate
    Staff->>UI: Soạn tin nhắn & bấm Gửi
    alt Chế độ INTERNAL_NOTE
        UI->>GW: POST /messages (visibility = INTERNAL_NOTE)
        Note over GW: Server Hard Gate: Chặn gọi API ngoài
        GW-->>UI: Lưu DB nội bộ + Push nội bộ (Không gửi Ext)
    else Chế độ PUBLIC Reply
        UI->>GW: POST /messages (visibility = PUBLIC)
        alt Kênh Zalo OA & Quá cửa sổ 48h
            GW-->>UI: Block & trả mã ZALO_WINDOW_EXPIRED (Gợi ý ZNS)
        else Hợp lệ (Telegram / Slack / Zalo còn hạn)
            GW->>Ext: Gọi API gửi tin nhắn ra kênh ngoài
            GW-->>UI: Cập nhật trạng thái SENT
        end
    end
```

---

## 2. Luồng Gom Nhiều Tin Nhắn → Task/Ticket với OPAgent & Snapshot (`FR-CHN-006`, `FR-CHN-007`, `FR-CHN-008`)

```mermaid
sequenceDiagram
    autonumber
    actor PM as PM / Team Member
    participant UI as Unified Inbox UI
    participant API as OmniProject API
    participant Mask as PII Masking Middleware
    participant Agent as OPAgent (LLM)
    participant DB as Postgres DB

    PM->>UI: Tick chọn 1..20 tin nhắn (cùng 1 hội thoại) -> Bấm "Tạo Task/Ticket"
    UI->>API: POST /drafts/from-messages (message_ids[], tenant_tz)
    API->>Mask: Che PII (SĐT, Email, CCCD) trước khi gọi LLM ngoài
    Mask->>Agent: Gửi nội dung đã che + anchor_timestamp (received_at)
    Agent-->>Mask: Trả JSON Draft (Title, Assignee hint, Deadline ISO)
    Mask-->>API: Khôi phục PII hiển thị cho PM
    API-->>UI: Hiển thị Modal Draft (Human-in-the-Loop)

    PM->>UI: Chọn Project (SUPPORT hoặc DELIVERY), kiểm tra/sửa & bấm "Lưu"
    UI->>API: POST /tasks
    alt Project loại SUPPORT
        API->>DB: INSERT Ticket (task_type=TICKET, parent_fr_id=FR-SUP-000)
    else Project loại DELIVERY
        API->>DB: Kiểm tra bắt buộc parent_fr_id (BR-001) -> INSERT Task
    end
    API->>DB: INSERT task_message_snapshots (Nội dung + File bất biến + Deep-link)
    DB-->>UI: 200 OK (Task/Ticket Created)
```

---

## 3. Luồng AI ChatOps Read-Only, Account Binding & Phòng thủ Prompt Injection (`FR-CHN-012`..`015`)

```mermaid
sequenceDiagram
    autonumber
    actor User as Nhân viên / Khách hàng (Telegram/Zalo)
    participant BotGW as AI ChatOps Gateway
    participant Auth as Binding & RateLimit (DB/Redis)
    participant Tool as Read-Only Query API (RBAC/Public View)
    participant LLM as OPAgent LLM

    User->>BotGW: Nhắn hỏi tiến độ (VD: "Tiến độ Ticket banner 20/10" hoặc Prompt Injection)
    BotGW->>Auth: Kiểm tra Binding (tenant_id, channel, external_sender_id) & Rate Limit

    alt Chưa liên kết hoặc đã bị Admin thu hồi (Zero-stale-cache)
        Auth-->>BotGW: DENIED (Unbound / Revoked)
        BotGW-->>User: Từ chối truy vấn & yêu cầu liên kết OTP/liên hệ Admin
    else Binding ACTIVE & Trong Rate Limit
        BotGW->>BotGW: Intent Gate (Kiểm tra chỉ cho phép Read-only)
        alt Phát hiện ý định Ghi/Sửa/Xóa Task
            BotGW-->>User: Từ chối thao tác ghi & hướng dẫn mở OmniMer Web App
        else Ý định Tra cứu (Read-only)
            BotGW->>Tool: Gọi Tool API với (tenant_id, user_id, customer_scope) tiêm cứng từ Server
            Note over Tool: Lọc cứng RBAC & Field-level Public View<br/>(Loại bỏ Internal Note, tên Dev, chi phí)
            Tool-->>BotGW: Trả về dữ liệu đã lọc (bọc thẻ untrusted nếu có mô tả)
            BotGW->>LLM: Tổng hợp câu trả lời tự nhiên từ dữ liệu đã lọc
            LLM-->>BotGW: Phản hồi an toàn
            BotGW-->>User: Gửi kết quả tiến độ (< 5s)
        end
    end
```

---

## 4. Luồng Lưu trữ BYOS, Lưu vào Dự án (`archive.md`) & Tuân thủ NĐ 13/2023/NĐ-CP (`FR-CHN-016`, `FR-CHN-017`)

```mermaid
sequenceDiagram
    autonumber
    actor Member as Team Member
    actor Admin as Tenant Admin / PM
    participant API as Storage & Archival API
    participant Store as Tenant Storage (Managed / BYOS)
    participant KB as Knowledge Base (FR-DOC-001)
    participant DB as Postgres DB

    Member->>API: Chọn tin nhắn + file bất kỳ định dạng -> Bấm "Lưu vào Dự án"
    API->>API: Đóng gói nội dung thành archive.md & tính SHA-256
    API->>Store: Ghi archive.md + Copy file gốc lên Storage của Tenant (storage_mode_used)
    alt Lỗi ghi BYOS Storage
        Store-->>API: Storage Write Error
        API-->>Member: Báo lỗi lưu trữ (Không ghi Asset rỗng)
    else Thành công
        API->>KB: Đồng bộ Asset mới (CHAT_ARCHIVE / DESIGN_FIGMA / ...)
        API->>DB: INSERT project_chat_archives + UPDATE messages (is_preserved=true, is_archived_to_project=true)
        API-->>Member: Lưu thành công (Miễn trừ Job Retention)
    end

    Admin->>API: Thực thi Yêu cầu Xóa PII theo Nghị định 13/2023/NĐ-CP
    API->>DB: Revoke Bindings + Ẩn danh PII trên Inbox (Không xóa file đã Archive vào Dự án)
    API-->>Admin: Biên bản hoàn tất (PII đã xóa, Tài sản dự án được bảo toàn)
```
