# Feature: Public Forms & Task Intake (FR-PRJ-021)

**Mô tả:** Chức năng cho phép PM biến một Project thành một "Cổng tiếp nhận yêu cầu". Bằng cách tạo ra một Form web chia sẻ công khai (Public Link), người ngoài tổ chức (Client, Customer) có thể điền thông tin. Mỗi lượt Submit sẽ tự động tạo ra một Task mới trong dự án.
**Priority:** P1 (High) - Cần thiết cho các team Service Desk, IT Helpdesk, Marketing Request.

---

## 1. Functional Requirements List

| FR ID | Requirement Description | Input | Output | Associated Business Rule | Priority |
|---|---|---|---|---|---|
| **FR-PRJ-021.1** | **Form Builder:** Màn hình kéo thả để PM xây dựng Form. Có thể map các trường trong Form với các trường mặc định của Task (Name, Description) hoặc Custom Fields. | Thao tác kéo thả | Form Layout JSON | None | Must-have |
| **FR-PRJ-021.2** | **Form Sharing (Public Link):** Sinh ra một đường link URL công khai (hoặc mã iframe để nhúng vào website công ty) không cần đăng nhập. | Bấm nút Share | Public URL | BL-PRJ-021.1 | Must-have |
| **FR-PRJ-021.3** | **Task Auto-Creation:** Khi người dùng bên ngoài nhấn Submit form, hệ thống tự động sinh ra một Task nằm ở cột mặc định (thường là cột đầu tiên - Backlog/Inbox) của dự án. | Form Data | Một Task mới | None | Must-have |
| **FR-PRJ-021.4** | **Spam Protection:** Tích hợp reCAPTCHA v3 ẩn vào Public Form để ngăn chặn bot spam tạo rác vào hệ thống. | Hành vi click submit | Chặn bot | BL-PRJ-021.2 | Must-have |
| **FR-PRJ-021.5** | **File Upload trên Form:** Cho phép người điền form đính kèm file (VD: ảnh chụp màn hình lỗi). File này sẽ tự động gắn vào Attachments của Task. | File (Max 20MB) | File lưu Cloud | None | Should-have |

---

## 2. Business Logic & Rules

* **BL-PRJ-021.1 (Form Access Control):** 
  * Bất kỳ ai có Public Link đều có thể submit form.
  * Form có thể bị PM đóng lại bất cứ lúc nào (Deactivate link). Nếu truy cập link đã đóng, hiện thông báo "Form này không còn nhận phản hồi".
* **BL-PRJ-021.2 (Spam & Rate Limit):** 
  * Áp dụng Rate Limit chặt cho API nhận submit form: Tối đa 5 requests / IP / phút. Vượt quá sẽ trả về HTTP 429.

---

## 3. Tiêu chí Chấp nhận (Acceptance Criteria)
**Là** Trưởng phòng IT (IT Helpdesk),
**Tôi muốn** nhân viên các phòng ban khác có thể báo lỗi máy tính qua một Form web đơn giản,
**Để** tôi không phải cấp tài khoản OmniProject cho toàn bộ nhân viên công ty nhưng vẫn quản lý được danh sách các lỗi dưới dạng Task.

**Acceptance Criteria:**
```gherkin
Given tôi đã tạo một Form "Báo cáo lỗi IT", có trường "Mô tả lỗi" (bắt buộc) và "Ảnh chụp" (không bắt buộc)
When nhân viên phòng Kế toán truy cập Public Link, điền form và bấm Submit
Then trên màn hình của Kế toán báo "Gửi yêu cầu thành công"
And trên Kanban Board của dự án IT Helpdesk, lập tức xuất hiện một Task mới có tên tương ứng ở cột "Inbox"
```
