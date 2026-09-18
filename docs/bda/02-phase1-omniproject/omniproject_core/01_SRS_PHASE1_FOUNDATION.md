# Software Requirements Specification (SRS) - Phase 1: Foundation
> **Reference Standard:** IEEE Std 830-1998
> **Project:** OmniProject Core
> **Phase:** 1 (Foundation, Multi-Tenancy & IAM)

## 1. Introduction

### 1.1 Purpose
Tài liệu SRS này đặc tả các yêu cầu phần mềm cho Giai đoạn 1 (Phase 1) của hệ thống OmniProject Core. Mục đích của Phase 1 là thiết lập hạ tầng kỹ thuật cốt lõi (Multi-tenancy), quản trị tổ chức (Organization/Workspace), và hệ thống định danh/phân quyền (IAM & RBAC). Tài liệu này phục vụ cho đội ngũ phát triển (Backend, Frontend, DevOps) và đội ngũ kiểm thử (QA) làm cơ sở thiết kế, lập trình và nghiệm thu.

### 1.2 Product Scope
Phạm vi của Phase 1 bao gồm:
- Xây dựng kiến trúc Multi-tenant (Data Isolation) để phục vụ mô hình SaaS B2B.
- Quản lý vòng đời của Organization và Workspace (Khởi tạo, Cấu hình, Thanh toán - Billing).
- Quản lý Danh tính và Truy cập (Identity & Access Management - IAM): Mời thành viên, User Groups, bảo vệ đăng nhập (Auto-lock chống Brute-force).
- Tích hợp Đăng nhập một lần (SSO) qua hai giao thức: SAML 2.0 và OIDC (OAuth 2.0).
- Thiết lập khung giao tiếp thời gian thực qua WebSocket kết hợp cơ chế Optimistic Concurrency Control (OCC).

Các tính năng Quản lý Dự án, Task, Bảng Kanban sẽ KHÔNG nằm trong phạm vi tài liệu này (chuyển sang Phase 2 & 3).

### 1.3 Definitions & Abbreviations
- **SSO:** Single Sign-On (Đăng nhập một lần).
- **SAML 2.0:** Security Assertion Markup Language (Giao thức xác thực chuẩn doanh nghiệp).
- **OIDC:** OpenID Connect (Dựa trên OAuth 2.0).
- **IdP:** Identity Provider (Nhà cung cấp định danh, ví dụ: Azure AD, Okta, Google).
- **JWT:** JSON Web Token (Sử dụng để lưu trữ phiên đăng nhập không trạng thái - stateless session).
- **OCC:** Optimistic Concurrency Control (Kiểm soát đồng thời lạc quan - xử lý xung đột dữ liệu bằng version thay vì database lock).
- **Tenant:** Một khách hàng (Organization/Workspace) độc lập trên hệ thống chia sẻ.
- **RBAC:** Role-Based Access Control (Kiểm soát truy cập dựa trên vai trò).

### 1.4 References
- [FR-PRJ-000: Core Technical Architecture & Governance](./FR-PRJ-000_core_technical_architecture.md)
- [FR-ORG-001: Workspace, Organization Management & Billing](./FR-ORG-001_workspace_organization_management.md)
- [FR-ORG-002: Identity, Access Management (IAM), SSO & Invites](./FR-ORG-002_identity_access_management.md)
- [00_DEVELOPMENT_PHASES_ROADMAP](./00_DEVELOPMENT_PHASES_ROADMAP.md)

---

## 2. Overall Description

### 2.1 Product Perspective
OmniProject Phase 1 đóng vai trò là "nền móng" (Foundation) cho một ứng dụng Web SaaS độc lập. Nó cung cấp API bảo mật và cơ sở dữ liệu Multi-tenant cho Frontend Web Client (React/Vue/Angular) tương tác. Hệ thống cũng giao tiếp với các hệ thống bên ngoài như cổng thanh toán (Stripe) để xử lý Subscription và các IdP để xác thực SSO.

### 2.2 Product Functions
- Khởi tạo Tenant (Organization & Workspace) khi user đăng ký.
- Quản lý gói cước (Free, Pro, Enterprise) qua Stripe.
- Mời thành viên mới qua Email JWT Token.
- Vô hiệu hóa (Deactivate) tài khoản nhân viên nghỉ việc.
- Cưỡng chế đăng nhập qua SSO (Force SSO) để tuân thủ bảo mật doanh nghiệp.
- Xử lý đồng bộ thời gian thực (Real-time sync) qua WebSocket Pub/Sub.

### 2.3 User Classes and Characteristics
Dựa trên kiến trúc RBAC, hệ thống phân loại người dùng thành 4 nhóm (Roles) cấp Workspace:

| User Class | Đặc điểm & Hành vi (Characteristics) | Access Rights (Quyền hạn Phase 1) |
|---|---|---|
| **Workspace Admin** | Là chủ doanh nghiệp hoặc IT Admin. Cần giao diện quản trị rõ ràng, ưu tiên tính bảo mật và khả năng kiểm soát toàn diện. | Toàn quyền: Cấu hình Workspace, Mời/Khóa thành viên, Nâng cấp gói cước, Cấu hình IdP (SAML/OIDC). Xem Audit Logs. |
| **Project Manager (PM)** | Người quản lý dự án. Trong Phase 1, PM đóng vai trò như một người dùng thông thường đối với các cấu hình cấp Org/Workspace. | Cập nhật hồ sơ cá nhân. Không có quyền mời thành viên (trừ khi được ủy quyền) hoặc đổi cấu hình SSO. |
| **Team Member** | Nhân viên công ty. Số lượng đông đảo nhất, thao tác chủ yếu là đăng nhập hàng ngày (Login). | Đăng nhập qua Email/Password hoặc SSO. Xem danh sách Directory nhưng không thể chỉnh sửa. |
| **Guest / Client** | Đối tác hoặc Khách hàng bên ngoài. | Tham gia vào Workspace qua lời mời, quyền hạn chỉ ở mức xem (Read-only) dữ liệu được chia sẻ, không tiếp cận cấu hình hệ thống. |

### 2.4 General Constraints
- **Regulatory:** Hệ thống phải tuân thủ chuẩn an toàn thông tin cơ bản (mật khẩu mã hóa bcrypt/Argon2, không lưu trữ thông tin thẻ tín dụng nguyên bản - tuân thủ PCI-DSS level 4 thông qua Stripe).
- **Hardware/Software Limits:** Database PostgreSQL cần có cấu trúc khóa ngoại (Foreign Key) chặt chẽ giữa các bảng thuộc cùng Tenant (Organization ID).

---

## 3. Specific Requirements (NFRs & Security)

### 3.1 External Interface Requirements
- **User Interfaces (UI):** Giao diện Web (SPA) hỗ trợ Responsive, tương thích với các trình duyệt hiện đại (Chrome 90+, Safari 14+, Firefox 88+, Edge).
- **Software Interfaces:**
  - **Stripe API:** Sử dụng Stripe Checkout / Billing API để tạo Subscription, nâng cấp gói cước. Webhook từ Stripe sẽ cập nhật trạng thái thanh toán về Backend.
  - **Email Service (SendGrid/AWS SES):** Sử dụng API để gửi email mời tham gia (Invite Links) và email cảnh báo bảo mật.
  - **Identity Providers (SAML/OIDC):** Giao tiếp qua HTTP Redirects và POST Bindings. Hệ thống sẽ đóng vai trò là Service Provider (SP) hoặc Relying Party (RP).

### 3.2 Non-Functional Requirements (Theo chuẩn IEEE 830)

#### 3.2.1 Performance Requirements (Hiệu năng)
| ID | Requirement Description | Measurable Threshold |
|---|---|---|
| **NFR-PERF-01** | Thời gian phản hồi API (Response Time) | 95% request API thông thường phải phản hồi $\le 200ms$. API liên kết bên thứ 3 (SSO, Stripe) $\le 1.5s$. |
| **NFR-PERF-02** | Độ trễ Real-time (WebSocket Latency) | Thời gian từ lúc phát sinh thay đổi ở Server đến lúc Client nhận được event phải $\le 100ms$ (điều kiện mạng chuẩn). |

#### 3.2.2 Security Requirements (Bảo mật)
| ID | Requirement Description | Reference Standard |
|---|---|---|
| **NFR-SEC-01** | Mã hóa Dữ liệu (Encryption) | Dữ liệu truyền tải qua mạng bắt buộc dùng HTTPS (TLS 1.2+). Mật khẩu lưu trữ dùng thuật toán `bcrypt` (work factor 10+) hoặc `Argon2id`. |
| **NFR-SEC-02** | Chống Brute-force & Rate Limiting | Khóa IP & Tài khoản 5 phút sau 5 lần nhập sai mật khẩu liên tiếp. Rate limit 100 req/min/IP cho API chung, 10 req/min/IP cho API Authentication. (OWASP Broken Auth) |
| **NFR-SEC-03** | Chống Injection & XSS | Mọi input phải được validate/sanitize ở Server-side. Dữ liệu xuất ra UI phải qua bộ lọc DOMPurify hoặc cơ chế Auto-escaping của Framework (React/Vue). (OWASP Injection & XSS) |
| **NFR-SEC-04** | Kiểm soát truy cập Multi-tenant | Mọi truy vấn DB phải bao gồm điều kiện `workspace_id = context.current_workspace`. Phân quyền RBAC qua Middleware trước khi vào Controller. (OWASP BOLA/IDOR) |

#### 3.2.3 Reliability & Availability (Độ tin cậy & Sẵn sàng)
| ID | Requirement Description | Threshold |
|---|---|---|
| **NFR-REL-01** | Thời gian hoạt động (Uptime) | Đạt chuẩn 99.9% uptime (thời gian chết tối đa ~43.8 phút/tháng). |
| **NFR-REL-02** | Kiểm soát đồng thời (Concurrency) | Áp dụng OCC (Optimistic Concurrency Control) bằng trường `version` trên các bảng cấu hình lớn, tránh mất dữ liệu khi 2 Admin cùng sửa một lúc. Trả về mã lỗi HTTP 412. |
| **NFR-REL-03** | Auto-reconnection (Phục hồi kết nối) | Nếu WebSocket đứt, Client tự động thử lại (Exponential backoff 1s, 2s, 4s...) và fetch API fallback để đồng bộ dữ liệu lỡ nhịp. |

#### 3.2.4 Maintainability (Bảo trì)
- Logs hệ thống (Application Logs) phải tuân thủ chuẩn JSON và được đẩy về Centralized Logging System (ví dụ: ELK Stack hoặc Datadog). 
- Audit Logs cho cấp độ Tổ chức (Organization Audit Logs) phải được lưu trữ riêng trong DB để phục vụ truy xuất tuân thủ luật lệ, không thể bị sửa đổi (Immutable).

---

## 4. Functional Requirements Traceability
Các yêu cầu chức năng (Functional Requirements) được đặc tả chi tiết kèm theo Sequence Diagrams và Acceptance Criteria tại các tài liệu sau:

- **Tạo & Quản lý Workspace, Subscription:** Xem chi tiết tại [FR-ORG-001](./FR-ORG-001_workspace_organization_management.md)
- **SSO, Mời thành viên, Auto-lock:** Xem chi tiết tại [FR-ORG-002](./FR-ORG-002_identity_access_management.md)
- **Kiến trúc RBAC & WebSockets:** Xem chi tiết tại [FR-PRJ-000](./FR-PRJ-000_core_technical_architecture.md)
