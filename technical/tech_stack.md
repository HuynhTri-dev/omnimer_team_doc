# Tech Stack — OmniProject Core

> **Phiên bản:** 1.0
> **Ngày chốt:** 2026-09-15
> **Phạm vi:** Áp dụng cho `backend_otm` và `frontend_otm`
> **Tham chiếu SRS:** [`01_SRS_PHASE1_FOUNDATION.md`](../docs/bda/02-phase1-omniproject/omniproject_core/01_SRS_PHASE1_FOUNDATION.md), [`FR-PRJ-000`](../docs/bda/02-phase1-omniproject/omniproject_core/FR-PRJ-000_core_technical_architecture.md)

---

## 1. Frontend — Web SPA Client

*Yêu cầu gốc: SPA Responsive, Drag & Drop mượt mà, Optimistic Updates, hỗ trợ trình duyệt Chrome 90+ / Safari 14+ / Firefox 88+ / Edge.*

| Layer | Lựa chọn | Phiên bản | Lý do |
| :--- | :--- | :--- | :--- |
| **Core Framework** | **Next.js** (App Router) | 14+ | File-system routing chuẩn hóa, tránh lỗi cấu hình `react-router`. Server Components giúp ẩn config nhạy cảm an toàn. Ổn định cao, duy trì bởi Vercel. |
| **Language** | **TypeScript** | 5+ | Type-safety end-to-end, chia sẻ interface giữa Frontend và Backend. |
| **Styling** | **Tailwind CSS** | 3+ | Utility-first, dễ custom theme/dark mode, không cần thêm layer UI library ở Phase 1. |
| **Server State** | **TanStack Query (React Query)** | 5+ | Quản lý caching, refetching và **Optimistic Updates** — cần thiết cho OCC (HTTP 412 rollback). |
| **Client State** | **Zustand** | 4+ | Quản lý UI state nhẹ nhàng (ví dụ: trạng thái modal, sidebar, filter đang chọn). |
| **Drag & Drop** | **dnd-kit** | 6+ | Hiện đại, nhẹ hơn `react-beautiful-dnd` (đã ngừng maintain). Hỗ trợ Kanban board, sortable list. |
| **WebSocket Client** | **native `WebSocket` API** | — | Browser hỗ trợ sẵn, không cần dependency. Dùng custom hook để wrap reconnection logic (Exponential Backoff). |

---

## 2. Backend — API & WebSocket Server

*Yêu cầu gốc: REST API, WebSocket Pub/Sub, RBAC Middleware, SSO (SAML 2.0 + OIDC), OCC (Version field), Rate Limiting.*

| Layer | Lựa chọn | Phiên bản | Lý do |
| :--- | :--- | :--- | :--- |
| **Core Framework** | **Express.js** | 4+ | Nhẹ, battle-tested, không có overhead framework phức tạp. Phù hợp với team nhỏ cần onboard nhanh và maintain dễ. |
| **Language** | **TypeScript** | 5+ | Bắt lỗi tại compile-time, đảm bảo contract API type-safe. |
| **ORM** | **Prisma** | 5+ | Type-safe schema, Prisma Client tự generate, hỗ trợ row-level filter `workspace_id` cho Multi-tenant dễ dàng. Migration rõ ràng. |
| **Authentication** | **Passport.js** | — | Strategy pattern — dùng `passport-jwt`, `passport-saml`, `passport-openidconnect`. Giải quyết gọn 3 luồng Auth chỉ với config strategy. |
| **WebSocket Server** | **`ws`** | 8+ | Native WebSocket library cho Node.js. Không có overhead protocol riêng như Socket.io. Đủ để làm Pub/Sub cho Phase 1. |
| **Rate Limiting** | **`express-rate-limit`** + **Redis Store** | — | Áp dụng 100 req/min/user cho API chung, 10 req/min/IP cho Auth endpoint (theo NFR-SEC-02). |

> **NestJS bị loại:** NestJS giải quyết bài toán của team 10+ người với nhiều module phức tạp. Phase 1 chỉ cần REST + WebSocket + RBAC, Express + các middleware nhỏ là đủ. Evaluate lại khi scale sang Phase 3+.

> **Socket.io bị loại:** Socket.io mang overhead protocol riêng (custom frame, polling fallback cho IE11). SaaS B2B hiện đại không cần fallback này — `ws` library là đủ và nhẹ hơn nhiều.

---

## 3. Database & Caching

*Yêu cầu gốc: PostgreSQL (bắt buộc theo SRS), OCC bằng trường `version`, Rate Limiting, WebSocket scale.*

| Layer | Lựa chọn | Phiên bản | Lý do |
| :--- | :--- | :--- | :--- |
| **Primary Database** | **PostgreSQL** | 17 | Bắt buộc theo SRS. Hỗ trợ Row-Level Security cho Multi-tenant, JSONB cho Custom Fields (Phase 4+), Foreign Key chặt chẽ giữa các bảng cùng tenant. Tối ưu hiệu năng JSON/Memory của PG17. |
| **Caching & Pub/Sub** | **Redis** | 7+ | (1) Store cho Rate Limiting. (2) Pub/Sub để fan-out WebSocket events ra nhiều Express instances khi scale. (3) Cache session/token ngắn hạn. |

---

## 4. Infrastructure & Deployment

*Yêu cầu gốc: 99.9% Uptime, Zero-downtime Deployment, Auto-reconnection.*

| Giai đoạn | Strategy | Chi tiết & Lý do |
| :--- | :--- | :--- |
| **Local Dev** | **Docker Compose** | Chạy PostgreSQL + Redis + Express + Next.js local bằng 1 lệnh để dev & test nhanh. |
| **Dev & Testing (Cloud Free Tier)** | **Render Free (FE & BE) + Neon (DB) + Upstash (Redis)** | Tận dụng Cloud Free Tier chạy ổn định liên tục vài tháng không tốn phí:<br/>• **Frontend (Next.js):** Deploy **Render Web Service** (hoặc Static Site).<br/>• **Backend (Express + WebSocket `ws`):** Deploy **Render Free Web Service** (Cấp 750h/tháng, hỗ trợ tốt kết nối WebSocket dai dẳng; tự ngủ sau 15p idle, cold start ~30s)<br/>• **Database:** **Neon PostgreSQL** (Free vĩnh viễn 0.5GB — *vẫn giữ Neon thay vì Render Postgres vì DB của Render sẽ tự xóa sau 30 ngày*)<br/>• **Redis:** **Upstash Redis** (Free vĩnh viễn 10.000 req/ngày) |
| **Production (Nghiệm thu & Đóng gói)** | **Docker Containers → VPS** | Sau khi test OK toàn bộ dịch vụ, đóng gói thành Docker Containers và deploy trực tiếp lên VPS của Omnimer Team để tiết kiệm chi phí và chủ động hạ tầng. |
| **Scale-up (Phase 3+)** | **AWS ECS (Fargate)** hoặc **K8s (EKS)** | Evaluate lại khi hệ thống cần auto-scaling thực sự và multi-region. |

> **Kubernetes bị defer khỏi Phase 1.** K8s phù hợp khi có hàng chục microservice độc lập.

---

## 5. Third-party Services

| Dịch vụ | Provider | Ghi chú |
| :--- | :--- | :--- |
| **Payment / Subscription** | **Stripe** | Theo yêu cầu SRS (FR-ORG-001). Dùng Stripe Billing API + Webhook. |
| **Transactional Email** | **AWS SES** | Chi phí thấp hơn SendGrid khi volume tăng. Dùng cho Invite Links + Security Alerts. |
| **Identity Provider (SSO)** | **Azure AD / Okta / Google** | OmniProject đóng vai SP (SAML) và RP (OIDC). Cấu hình per-workspace qua Admin UI. |

---

## 6. Tổng quan Stack (Decision Summary)

```
Frontend:  Next.js 14 (App Router) + TypeScript + Tailwind CSS
           Zustand (Client State) + TanStack Query (Server State + Optimistic Updates)
           dnd-kit (Drag & Drop) + native WebSocket API

Backend:   Express.js + TypeScript
           Prisma ORM (PostgreSQL) + Passport.js (JWT / SAML / OIDC)
           ws (WebSocket Server) + express-rate-limit + Redis Store

Database:  PostgreSQL 17 (Primary) + Redis 7 (Cache / Rate Limit / Pub-Sub)

Infra:     Docker Compose (Local) → Cloud Free Tier (Dev/Test) → Docker + VPS (Prod) → AWS ECS (Phase 3+)

3rd Party: Stripe + AWS SES + IdP (Azure AD / Okta / Google)
```

---

*Tài liệu này là nguồn sự thật duy nhất (Single Source of Truth) cho quyết định công nghệ của OmniProject Core. Mọi thay đổi phải được ghi lại tại đây với lý do cụ thể.*
