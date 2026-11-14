# Feature: Data Import, Export & Third-Party Integrations (FR-PRJ-022)

**Mô tả:** Cung cấp công cụ di chuyển dữ liệu (Migration) để giúp khách hàng mới onboard dễ dàng từ các nền tảng khác (Trello, Jira, Asana) hoặc xuất dữ liệu dự án ra file CSV/Excel để báo cáo.
**Priority:** Phase 2 (Backlog) - *Lưu ý: Tính năng này đã được phê duyệt nhưng dời sang Phase 2 để tránh rủi ro Scope Creep cho team Dev ở giai đoạn hiện tại.*

---

## 1. Functional Requirements List (Dự kiến)

| FR ID | Requirement Description | Input | Output | Associated Business Rule | Priority |
|---|---|---|---|---|---|
| **FR-PRJ-022.1** | **CSV/Excel Import:** Cho phép PM tải lên một file CSV. Cung cấp UI Mapping (Khớp cột CSV với các trường dữ liệu của Task như Tên, Assignee, Ngày đến hạn). | File CSV | Sinh ra danh sách Tasks | None | Phase 2 |
| **FR-PRJ-022.2** | **Trello Importer:** Nhập trực tiếp dữ liệu từ Trello Board qua API (Sử dụng OAuth để xác thực). Đảm bảo giữ nguyên các Cột (Lists), Labels, và Members nếu trùng email. | Trello API Token | Chuyển đổi toàn bộ Board thành Project | None | Phase 2 |
| **FR-PRJ-022.3** | **Jira Importer:** Tương tự Trello nhưng phức tạp hơn do phải map Issue Types (Epic, Story, Bug) thành cấu trúc Task/Subtask của OmniProject. | Jira Cloud Token | Project mới với Data Jira | None | Phase 2 |
| **FR-PRJ-022.4** | **Full Project Export:** Nút cho phép PM tải về toàn bộ dự án dưới định dạng CSV, JSON. Kèm theo link nén (ZIP) các file đính kèm. | Nút Export | File tải về (.zip, .csv) | None | Phase 2 |

---

## 2. Business Logic & Rules (Sơ bộ)
* **Xử lý bất đồng bộ (Async Processing):** Tiến trình Import/Export dữ liệu lớn (hàng ngàn tasks) không được phép chạy đồng bộ chặn (Block) luồng API chính. Phải đẩy vào Message Queue (RabbitMQ) và xử lý ngầm (Background Worker). UI sẽ hiển thị thanh Progress Bar, sau khi xong sẽ báo Notification/Email.
* **Member Mapping Fallback:** Trong quá trình Import từ Trello/Jira, nếu có một user trong file không tồn tại (chưa có tài khoản) trong OmniProject, hệ thống sẽ tự động gán Task đó cho "Unassigned" và ghi chú vào description: "Originally assigned to [Tên user cũ]".

---

## 3. Tiêu chí Chấp nhận (Tham khảo cho Phase 2)
**Là** một công ty mới mua phần mềm OmniProject,
**Tôi muốn** chuyển toàn bộ dữ liệu dự án đang có sẵn trên Trello sang hệ thống mới,
**Để** đội ngũ không phải tạo lại hàng trăm thẻ công việc bằng tay.

**Acceptance Criteria:**
```gherkin
Given tôi bấm vào nút "Import từ Trello"
When tôi cấp quyền OAuth cho OmniProject đọc dữ liệu Trello của tôi
Then hệ thống hiển thị danh sách các Board hiện có bên Trello
When tôi chọn Board "Marketing 2026"
Then hệ thống chạy background ngầm và sinh ra một Project "Marketing 2026" trong OmniProject, giữ nguyên thứ tự 5 cột trạng thái và toàn bộ thẻ bên trong
```
