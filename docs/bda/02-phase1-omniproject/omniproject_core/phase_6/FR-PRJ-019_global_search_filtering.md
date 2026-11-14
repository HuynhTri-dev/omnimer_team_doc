# Feature: Global Search & Advanced Filtering (FR-PRJ-019)

**Mô tả:** Hệ thống tìm kiếm toàn cục (Global Search) và bộ lọc nâng cao (Advanced Filtering) giúp người dùng nhanh chóng định vị Task, Comment, File đính kèm, hoặc Project giữa hàng nghìn dữ liệu xuyên suốt Workspace.
**Priority:** P1 (High)

---

## 1. Functional Requirements List

| FR ID | Requirement Description | Input | Output | Associated Business Rule | Priority |
|---|---|---|---|---|---|
| **FR-PRJ-019.1** | **Quick Search (Cmd+K):** Thanh tìm kiếm nhanh dạng pop-up ở mọi màn hình. Hỗ trợ tìm kiếm realtime (gõ tới đâu hiện tới đó). | Keyword (Tên task, ID) | Danh sách kết quả gợi ý | BL-PRJ-019.1 | Must-have |
| **FR-PRJ-019.2** | **Full-text Search:** Tìm kiếm nội dung bên trong Description, Comments, và tên file đính kèm. Hỗ trợ highlight từ khóa trùng khớp. | Keyword dài | Trang kết quả chi tiết | None | Must-have |
| **FR-PRJ-019.3** | **Advanced Filtering (OQL):** Cung cấp giao diện lọc nhiều lớp (Multi-layer filter) tại các View. Hỗ trợ Omni Query Language (OQL) dạng gõ text, VD: `assignee:me AND status:!done OR due_date:<today`. | Biểu thức OQL / Form lọc | Danh sách Task đã lọc | BL-PRJ-019.2 | Should-have |
| **FR-PRJ-019.4** | **Saved Filters:** Cho phép user lưu lại bộ lọc thường dùng (Ví dụ: "Task trễ hạn của team Dev") thành các Quick Tabs để bấm một phát ăn ngay. | Tên filter, Điều kiện lọc | Nút Filter hiển thị trên UI | None | Should-have |

---

## 2. Business Logic & Rules

* **BL-PRJ-019.1 (Search Access Control):** 
  * Kết quả tìm kiếm PHẢI tuân thủ RBAC. Nếu user tìm một từ khóa tồn tại trong Project A, nhưng user đó KHÔNG CÓ QUYỀN (Không phải thành viên) của Project A, thì tuyệt đối không được hiển thị kết quả đó.
  * Tương tự với Private Task trong Personal Space của người khác, phải bị loại khỏi kết quả tìm kiếm.
* **BL-PRJ-019.2 (OQL Constraints):** 
  * OQL (Omni Query Language) phải được validate cẩn thận để tránh lỗi parse.
  * Nếu query sai cú pháp (VD thiếu ngoặc), UI phải hiện lỗi thân thiện thay vì crash.

---

## 3. Kiến trúc Kỹ thuật (Gợi ý)
- Database RDBMS (PostgreSQL) kết hợp với **ElasticSearch** hoặc **Meilisearch** để xử lý Full-text search tốc độ cao.
- Cần có cơ chế Sync dữ liệu từ DB sang Search Engine mỗi khi có CRUD operation trên Task (thông qua Message Queue hoặc Debezium/CDC).

---

## 4. Tiêu chí Chấp nhận (Acceptance Criteria)
**Là** Team Member,
**Tôi muốn** có thể gõ phím tắt Cmd+K ở bất kỳ đâu để tìm lại một task cũ,
**Để** không phải mất công bấm vào từng dự án để mò mẫm.

**Acceptance Criteria:**
```gherkin
Given tôi đang ở màn hình Dashboard
When tôi bấm Cmd+K và gõ chữ "Fix bug login"
Then popup search hiện ra ngay lập tức
And danh sách gợi ý trả về đúng Task có chứa cụm từ đó trong vòng 200ms
```
