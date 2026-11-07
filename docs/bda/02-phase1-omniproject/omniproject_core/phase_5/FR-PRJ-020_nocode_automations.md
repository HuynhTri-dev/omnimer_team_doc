# Feature: No-Code Automations (FR-PRJ-020)

**Mô tả:** Bộ máy Tự động hóa (Rule Engine) cho phép người dùng cấu hình các logic nghiệp vụ tùy chỉnh theo mô hình "If-This-Then-That" (Trigger -> Condition -> Action) mà không cần viết code. Giúp giảm thiểu các thao tác lặp đi lặp lại.
**Priority:** P1 (High) - Tính năng Core Enterprise

---

## 1. Functional Requirements List

| FR ID | Requirement Description | Input | Output | Associated Business Rule | Priority |
|---|---|---|---|---|---|
| **FR-PRJ-020.1** | **Automation Builder (UI):** Giao diện kéo thả cho phép chọn Trigger (Khi nào), Condition (Điều kiện), và Action (Làm gì). | Các block kéo thả | Rule Definition JSON | None | Must-have |
| **FR-PRJ-020.2** | **Triggers (Sự kiện kích hoạt):** Hỗ trợ các Trigger: Task created, Status changed, Due date approaches (trước 1 ngày/1 giờ), Custom Field updated. | Event Payload | Kích hoạt Rule | BL-PRJ-020.1 | Must-have |
| **FR-PRJ-020.3** | **Conditions (Điều kiện lọc):** Hỗ trợ check điều kiện: Trạng thái hiện tại là gì, Assignee là ai, Custom field giá trị bao nhiêu. | Rule logic | True / False | None | Must-have |
| **FR-PRJ-020.4** | **Actions (Hành động thực thi):** Hỗ trợ: Đổi Status, Gán người (Assign), Add Comment, Set Due Date, Gửi tin nhắn Slack/Teams, Gửi Email. | Execution command | Dữ liệu bị thay đổi / Message gửi đi | BL-PRJ-020.2 | Must-have |
| **FR-PRJ-020.5** | **Automation Logs:** Giao diện xem lịch sử chạy của các Rule (Thành công/Thất bại), báo lỗi chi tiết nếu Action fail (VD: Slack token hết hạn). | Lịch sử chạy | Danh sách Logs | None | Should-have |

---

## 2. Business Logic & Rules

* **BL-PRJ-020.1 (Infinite Loop Protection):** 
  * Cực kỳ quan trọng: Hệ thống phải có cơ chế phát hiện và chặn vòng lặp vô hạn (Ví dụ: Rule A đổi status sang X, Rule B thấy status X lại đổi về Y, Rule A lại kích hoạt...). 
  * Giới hạn: Một task chỉ được phép trigger tối đa 10 chuỗi automation trong vòng 1 phút. Vượt quá sẽ bị chặn (Circuit Breaker) và gửi cảnh báo cho PM.
* **BL-PRJ-020.2 (Permissions in Automation):** 
  * Bot Automation chạy dưới quyền "System". Tuy nhiên, nếu Rule được tạo bởi PM, nó chỉ có tác dụng trong phạm vi Project đó.

---

## 3. Data Structure

### Entity: Automation_Rule
| Field | Data Type | Required | Constraints |
|---|---|---|---|
| `rule_id` | UUID | Yes | Primary Key |
| `project_id` | UUID | Yes | |
| `name` | String | Yes | VD: "Auto assign QA khi Done" |
| `is_active` | Boolean | Yes | Default: True |
| `trigger_config` | JSONB | Yes | `{ "event": "STATUS_CHANGED", "target": "Done" }` |
| `action_config` | JSONB | Yes | `[{ "action": "SET_ASSIGNEE", "user_id": "uuid" }]` |

---

## 4. Tiêu chí Chấp nhận (Acceptance Criteria)
**Là** Project Manager,
**Tôi muốn** hệ thống tự động gán Task cho nhân viên QA khi Developer kéo Task sang cột "Ready for Test",
**Để** tối ưu thời gian liên lạc và tránh việc quên assign người test.

**Acceptance Criteria:**
```gherkin
Given tôi đã tạo một Rule: [Trigger: Status = Ready for Test] -> [Action: Assign to QA_User]
When Developer kéo Task-A sang cột "Ready for Test"
Then hệ thống tự động loại bỏ Assignee cũ và gán Task-A cho QA_User
And một Notification được bắn về máy của QA_User báo rằng "Bạn vừa được assign một Task tự động"
```
