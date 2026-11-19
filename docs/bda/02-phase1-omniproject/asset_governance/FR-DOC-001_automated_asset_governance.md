# Feature: Automated Asset Governance (FR-DOC-001)

**Mô tả:** Hệ thống Kho Tri thức (Knowledge Base) tự động phân loại, gắn thẻ và lưu trữ các tài liệu đầu ra từ Task để phục vụ đào tạo (Training), bảo trì (Maintenance) và chuyển giao (Onboarding) cho nhân viên mới.
**Priority:** P1 (High)

## 1. Đặc tả Kỹ thuật (FRD)

### 1.1 Data Contract (Asset Document Schema)

```jsonc
{
  "asset_id": "uuid",
  "project_id": "uuid",
  "source_type": "TASK | OMNICHANNEL_ARCHIVE", // Nguồn từ Task Done hoặc Nút "Lưu vào Dự án" (FR-CHN-017)
  "source_task_id": "uuid | null",
  "source_archive_id": "uuid | null",       // FK -> project_chat_archives.id
  "source_conversation_id": "uuid | null",  // Deep-link về hội thoại OmniChannel nếu có
  "author_id": "uuid",                      // Người nộp tài liệu hoặc người bấm Lưu vào Dự án
  "asset_type": "string",                   // CODE_PR, DESIGN_FIGMA, FRD_DOC, SPREADSHEET, IMAGE, CHAT_ARCHIVE, OTHER
  "storage_key": "string | null",           // Key trên Storage Tenant (Managed / BYOS)
  "url": "string",                          // Presigned S3/BYOS Link hoặc External URL
  "sha256": "string | null",                // Checksum toàn vẹn (từ FR-CHN-017)
  "created_at": "ISO8601"
}
```

### 1.2 Business Logic & Rules

* **BL-DOC-001.1 (Auto-Tagging Pipeline):** Khi nhận event `TaskDone_WithOutput` từ OmniProject Core, hệ thống tự động bóc tách thông tin: Tên Project, ID người làm (Author), Ngày hoàn thành (Timestamp) để lưu metadata cho Asset mà không cần User nhập thủ công.
* **BL-DOC-001.2 (Asset Retrieval & Filtering):** Cung cấp giao diện tìm kiếm (Search UI) cho phép lọc tài liệu theo đa chiều. Ví dụ: "Tìm tất cả DESIGN_FIGMA do nhân viên A tạo trong năm 2026 thuộc Project B".
* **BL-DOC-001.3 (Onboarding Export):** PM có quyền gom nhiều Assets lại thành một "Onboarding Package" (Gói bàn giao) dưới dạng 1 link chia sẻ duy nhất để gửi cho nhân viên mới đọc hiểu hệ thống.
* **BL-DOC-001.4 (OmniChannel Chat Archive Ingestion):** Tiếp nhận bản ghi đồng bộ từ [FR-CHN-017](file:///Users/macbookair/Downloads/source/omnimer_team/omnimer_team_doc/docs/bda/02-phase1-omniproject/omnichannel/phase_3/FR-CHN-017_tenant_storage_byos_and_archival.md) khi thành viên bấm *"Lưu vào Dự án"* trên Unified Inbox (`source_type = OMNICHANNEL_ARCHIVE`), hiển thị file `archive.md` cùng các tệp đính kèm từ Storage Tenant (Managed hoặc BYOS) trực tiếp trong Knowledge Base của Project và hỗ trợ gom vào Onboarding Package.

---

## 2. Tiêu chí Chấp nhận (Acceptance Criteria)

**Là** Project Manager / Admin,
**Tôi muốn** mọi tài liệu đầu ra của dự án được lưu trữ tập trung và tự động phân loại theo người làm và ngày tháng,
**Để** khi một nhân sự cứng nghỉ việc, tôi có sẵn trọn bộ tài liệu chi tiết (đã chống vibe code) để gửi cho nhân sự mới tự học (training).

**Acceptance Criteria (Gherkin):**
```gherkin
Given Nhân viên A vừa hoàn thành Task "Viết API Login" và nộp kèm link PR (Pull Request)
When Tôi vào màn hình "Knowledge Base" của Dự án
Then Hệ thống hiển thị 1 thẻ tài liệu mới với nhãn [Type: CODE_PR], [Author: Nhân viên A], [Time: Vừa xong]
And Khi tôi click vào thẻ đó, hệ thống trỏ thẳng đến link PR gốc
```
