# Báo cáo Kết quả Chiến dịch Tấn công & Thử thách Phát hiện Lỗi Toàn diện (OmniMer System-Wide Defect Discovery Report)

**Ngày thực hiện tấn công ban đầu:** 2026-11-16 (Pre-fix baseline: `NO-GO`)  
**Ngày hoàn tất khắc phục & tái kiểm thử (Fix Verification):** 2026-11-16 (`GO`)  
**Phạm vi kiểm thử:** `backend_otm/project_core`, `backend_otm/omnimer_mcp_server`, `frontend_otm/website_otm`  
**Kế hoạch tham chiếu:** [12_SYSTEMATIC_DEFECT_DISCOVERY_ATTACK_PLAN.md](file:///Users/macbookair/Downloads/source/omnimer_team/omnimer_team_doc/docs/bda/02-phase1-omniproject/omniproject_core/12_SYSTEMATIC_DEFECT_DISCOVERY_ATTACK_PLAN.md)  
**Trạng thái hiện tại:** Toàn bộ **55 lỗi** (thuộc 25 nhóm `P0–P3`) phát hiện trong chiến dịch đã được vá trong mã nguồn cục bộ, đồng thời xác minh **100% PASS** trên toàn bộ test tấn công, baseline regression, mutation gate, build TypeScript và ESLint.

---

## 1. Tóm tắt Điều hành & Quyết định Cổng Phát hành (Release Gate Scorecard)

> **KẾT LUẬN RELEASE GATE SAU KHẮC PHỤC: `GO` (ĐẠT TOÀN BỘ NGƯỠNG KIỂM THỬ CỤC BỘ — SẴN SÀNG TRIỂN KHAI PRODUCTION)**  
> Toàn bộ **7/7 nhóm lỗi `P0`**, **10/10 nhóm `P1`**, **7/7 nhóm `P2`** và **1/1 nhóm `P3`** đã được xử lý triệt để.  
> *Lưu ý vận hành:* Các thay đổi CORS whitelist (`backend_otm/project_core/src/app.ts`) và Security Headers / CSP (`frontend_otm/website_otm/next.config.mjs`) đã hoàn thiện và xác minh trên môi trường kiểm thử cục bộ; cần deploy nhánh mã nguồn mới lên Render & Cloudflare Workers để áp dụng lên endpoint live.

| Tiêu chí Release Gate | Ngưỡng yêu cầu | Trước khắc phục (Baseline) | Sau khắc phục (Verified) | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| Lỗi nghiêm trọng `P0` | `= 0` | **7 nhóm lỗi `P0`** | **0 lỗi `P0` mở** | **PASS** |
| Lỗi mức cao `P1` | `<= 3` (0 BOLA/Auth/SSRF/DoS) | **10 nhóm lỗi `P1`** | **0 lỗi `P1` mở** | **PASS** |
| Lỗi mức trung bình `P2` | Đã vá & kiểm chứng | **7 nhóm lỗi `P2`** | **0 lỗi `P2` mở** | **PASS** |
| Lỗi mức thấp `P3` | Hợp đồng error code / status | **1 nhóm lỗi `P3`** | **0 lỗi `P3` mở** | **PASS** |
| Backend Attack Suites (Waves 1–3, 5) | 100% Pass | 22 PASS / 44 FAIL | **66/66 PASS (5/5 suites, 27.335s)** | **PASS** |
| MCP Attack & Contract Suites | 100% Pass | 4 PASS / 4 FAIL | **8/8 PASS (0 FAIL, `npm run build` ok)** | **PASS** |
| Frontend Playwright Wave 4 & Chaos | 100% Pass | 2 PASS / 7 FAIL | **9/9 PASS (`http://localhost:3100`, 36.2s)** | **PASS** |
| Mutation Score (`MUT-01..04`) | `>= 80%` trên module lõi | `HealthScore` 60%, `Workload` 60%, `OqlCompiler` 20% | `HealthScore` **100% (10/10)**, `Workload` **100% (10/10)**, `Dependency` **80% (8/10)**, `OqlCompiler` **90% (9/10)** | **PASS (4/4)** |
| Hồi quy Baseline & Static Checks | 100% pass `tsc`, `lint`, unit/integration | BE `tsc` ok, 65/65 suites pass; FE `tsc` & `lint` 0 errors | BE: `tsc` ok, **65/65 suites (705/705 tests)** pass (`--runInBand`, 33.942s); FE: `tsc` ok & `next lint` **No ESLint warnings or errors** | **PASS** |

---

## 2. Tổng hợp Kết quả Theo Từng Làn Sóng Tấn công (Sau Khắc phục)

### 2.1. Tổng quan Định lượng Từng Bộ Test Tấn công

| Làn sóng / Bộ kiểm thử | Công cụ / Tệp tái lập | Tổng Test | Trước Fix (PASS/FAIL) | Sau Fix (PASS/FAIL) | Ghi chú xác minh |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Wave 1A:** AuthZ Matrix, BOLA/IDOR, AuthN, Mass Assignment | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) | 19 | 4 / 15 | **19 / 0** | Authz matrix, BOLA 404, GUEST RBAC, strict DTOs, rateLimiter |
| **Wave 1B:** WebSocket Authorization & Eviction | [wave1.ws-authz.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.ws-authz.test.ts) | 3 | 1 / 2 | **3 / 0** | Immediate socket eviction & `ws:error` (`FORBIDDEN`) |
| **Wave 2A:** Public Intake, Automations, Files, Importer & SSRF | [wave2.intake-automations-files.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave2.intake-automations-files.test.ts) | 17 | 6 / 11 | **17 / 0** | reCAPTCHA replay, SVG block, S3 verify, loop/WIP/Queue, Confused Deputy |
| **Wave 2B:** MCP Security, Path Traversal & Prompt Injection | [mcp-attack-plan.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/omnimer_mcp_server/src/tests/mcp-attack-plan.test.ts) | 6 *(+2 contract)* | 2 / 4 *(+2)* | **6 / 0** *(+2 contract)* | `additionalProperties: false`, safe path segments, `<untrusted_data>` |
| **Wave 3:** Concurrency Races, Pagination/Date DoS & DST | [wave3.race-abuse-time.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave3.race-abuse-time.test.ts) | 14 | 2 / 12 | **14 / 0** | Keyed locks, advisory/`FOR UPDATE`, partial indexes, TZ/DST |
| **Wave 4 & FE Chaos:** XSS, CSP/Headers, Token, OCC, Tab Sync, Virtual DOM, IME | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) | 9 | 2 / 7 | **9 / 0** | Playwright Chromium trên `http://localhost:3100` |
| **Wave 5:** Triple-Box BVA/EP, Graybox, AST Mutation & Chaos/ReDoS | [wave5.triple-box-chaos.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave5.triple-box-chaos.test.ts) | 13 | 9 / 4 | **13 / 0** | NFC OQL normalization, AST mutation `>= 80%` trên 4/4 modules |
| **Tổng cộng chiến dịch tấn công (không tính 2 contract test gốc)** | — | **81** | **26 / 55** | **81 / 0** | **100% Passed (0 failures)** |

### 2.2. Xác minh Toàn vẹn & Baseline Regression Sau Các Bản Vá Cuối Cùng

* `backend_otm/project_core`:
  * `npx tsc --noEmit`: 0 lỗi biên dịch TypeScript.
  * `npx jest src/__tests__/attack_plan/ --no-cache --forceExit`: **5/5 suites passed, 66/66 tests passed** (`Time: 27.335 s`).
  * `npx jest --testPathIgnorePatterns=attack_plan --runInBand --silent --forceExit`: **65/65 suites passed, 705/705 tests passed** (`Time: 33.942 s`).
* `backend_otm/omnimer_mcp_server`:
  * `npm run build` (`tsc`) thành công và `node --test dist/tests/mcp-contract.test.js dist/tests/mcp-attack-plan.test.js` đạt **8/8 tests passed, 0 failed**.
* `frontend_otm/website_otm`:
  * `npx tsc --noEmit`: 0 lỗi TypeScript.
  * `npm run lint` (`next lint`): **`No ESLint warnings or errors`**.
  * `BASE_URL=http://localhost:3100 npx playwright test tests/e2e/attack-wave4-5.spec.ts --reporter=list`: **9/9 tests passed (`36.2s`)**.
* Memory hệ thống đã được ghi nhận theo quy tắc tại [system_defect_remediation.md](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/memory/16-11-2026/system_defect_remediation.md) và [frontend_defect_remediation.md](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/memory/16-11-2026/frontend_defect_remediation.md).

---

## 3. Danh mục Chi tiết Lỗi Đã Chứng minh Theo Mức độ Nghiêm trọng (`P0` – `P3`)

### 3.1. Lỗi Mức `P0` (Release Blockers — Bắt buộc Vá Trước Mọi Phát hành)

| Mã Lỗi | Attack IDs | Thành phần & Vị trí Mã nguồn | Hành vi Thực tế (Quan sát) vs Kỳ vọng | Tệp Chứng minh |
| :--- | :--- | :--- | :--- | :--- |
| **DEF-P0-01** | `AUTHZ-OBJ-01` | Backend `project.routes.ts:41-45` (`GET /api/v1/projects/:projectId`) | Thành viên cùng Workspace nhưng **không** thuộc Project A2 (`USER_NON_PROJ_A`) đọc được chi tiết Project A2 với **`HTTP 200 OK`** thay vì `404 NOT_FOUND` (BOLA Read). | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |
| **DEF-P0-02** | `AUTHZ-OBJ-04` | Backend `portfolio.routes.ts:20-38`, `capacity.routes.ts:31`, `project.routes.ts:27-38` | Tài khoản role `GUEST` vượt rào RBAC: gọi `GET /workspaces/:wsId/portfolio` trả **`200 OK`**, `GET /workload` trả **`200 OK`**, `POST /projects` tạo được dự án mới (**`201 Created`**), và `GET /projects` lộ cả `PROJ_A2` không thuộc quyền (`33330002-...`). | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |
| **DEF-P0-03** | `CORS-01` *(Live Probe)* | Backend Production Render `https://omnimer-project-core.onrender.com` | Gửi `OPTIONS /api/v1/auth/login` với header `Origin: https://evil-attacker.workers.dev` nhận về **`HTTP/2 204`** phản xạ nguyên `access-control-allow-origin: https://evil-attacker.workers.dev` cùng **`access-control-allow-credentials: true`**. | Live read-only `curl -sI -X OPTIONS` |
| **DEF-P0-04** | `AUTO-03` | Backend `AutomationEngine.ts` | **Confused Deputy:** Automation rule do `PM_A` tạo vẫn thực thi thành công (`status: "SUCCESS"`) dù `PM_A` đã bị hạ xuống role `MEMBER`/`GUEST` trước khi kích hoạt, thay vì ngắt với `PERMISSION_REVOKED`. | [wave2.intake-automations-files.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave2.intake-automations-files.test.ts) |
| **DEF-P0-05** | `PUB-04a`, `PUB-04b`, `FILE-01b` | Backend `attachment.dto.ts`, `PublicIntakeService.ts:136`, `markdownSanitizer.ts:19,97` | **Stored/Reflected XSS vectors:** (1) `POST /public/forms/:slug/uploads` nhận `{ fileName: "xss.svg", mimeType: "image/png" }` -> **`200 OK`**; (2) `embedSnippet` nội suy thô `form.title` tạo `<iframe ... title=""/><script>alert("xss")</script>...`; (3) `RAW_HTML_TAG_REGEX` (`markdownSanitizer.ts:19`) yêu cầu khoảng trắng `\s` nên giữ nguyên `<a/href="javascript:alert(1)">click`. | [wave2.intake-automations-files.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave2.intake-automations-files.test.ts) |
| **DEF-P0-06** | `MCP-02c` | MCP `src/resources/index.ts`, `src/client/OmniMerClient.ts` | URI `omnimer://tasks/id%2F..%2F..%2Fworkspaces%2FWS_B%2Faudit-logs` bị decode `%2F` và không validate UUID, làm MCP client gửi HTTP request tới đường dẫn traversal **`/api/v1/workspaces/WS_B/audit-logs`**. | [mcp-attack-plan.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/omnimer_mcp_server/src/tests/mcp-attack-plan.test.ts) |
| **DEF-P0-07** | `RC-02`, `RC-03`, `RC-07` | Backend `TaskService.ts`, `DependencyService.ts` | **3 Concurrency Data Integrity Defects:** (1) `PATCH /tasks/:id/move` không gửi `currentVersion`/`expectedVersion` vẫn trả **`200 OK`** (OCC bypass); (2) 2 move đồng thời vào cột `wipLimit=1` đều `200 OK` -> `finalInColumn: 2`, `lockedStatus: false`; (3) 2 request tạo dependency `A->B` và `B->A` đồng thời đều **`201 Created`** lưu `depsInserted: 2` tạo vòng lặp DAG (`locked: false`). | [wave3.race-abuse-time.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave3.race-abuse-time.test.ts) |

---

### 3.2. Lỗi Mức `P1` (High Severity — Cần Vá Trước GA)

| Mã Lỗi | Attack IDs | Thành phần & Vị trí Mã nguồn | Mô tả Lỗi & Bằng chứng Thực tế | Tệp Chứng minh |
| :--- | :--- | :--- | :--- | :--- |
| **DEF-P1-01** | `SEC-04`, `AUTHZ-OBJ-01`, `AUTHZ-OBJ-03` | Backend Task / Subtask / Comment / Attachment & Project nested routes | **Existence Oracle:** Truy cập trái phép private task (`PATCH /tasks/:id` bởi `MEMBER_A`) hoặc project entity khác (`board`, `tasks`, `metrics`, `subtasks`, `comments`, `attachments/download-url`) trả **`403`** thay vì **`404 NOT_FOUND`**, làm lộ ID tồn tại cho kẻ thăm dò. | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |
| **DEF-P1-02** | `AUTHN-02`, `AUTHN-06`, `AUTHN-05b` | Backend `auth.routes.ts` / `auth.controller.ts` | (1) `POST /api/v1/auth/refresh` trả **`404`**; (2) `POST /auth/sso/saml/callback` và `GET /auth/sso/oidc/callback` trả **`404`**; (3) `/auth/login` và `/auth/register` chưa gắn `rateLimitMiddleware` chống brute-force theo IP/tuyến đường. | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |
| **DEF-P1-03** | `MASS-01a`, `MASS-01b` | Backend `src/api/dtos/*` (`invite.dto.ts`, `member.dto.ts`, `data-migration.dto.ts`, v.v.) | **14/30 file DTO** có `z.object()` chưa gọi `.strict()` (`invite.dto.ts` `0/4`, `member.dto.ts` `0/4`, `data-migration.dto.ts` còn `1 .passthrough()`). Thực thi live cho thấy `POST /workspaces/:wsId/invites` nhận thêm `{ organizationId, status }` vẫn **`201`** và `PATCH /members/:id/role` nhận thêm `{ workspaceId }` vẫn **`200`**. | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |
| **DEF-P1-04** | `WS-AUTH-03`, `WS-AUTH-01` | Backend `WebSocketGateway.ts:234-300,380,606-610` | **WS Post-Revoke Frame Leak:** Thu hồi thành viên dùng `setTimeout(() => socket.close(1008), 100)` mà không gỡ socket khỏi room ngay (`removeSocket`), làm lọt bản tin `leakAfterRevoke`. Ngoài ra WS lỗi quyền trả `{ event: 'ERROR', code: 403 }` thay vì `{ event: 'ws:error', code: 'FORBIDDEN' }`. | [wave1.ws-authz.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.ws-authz.test.ts) |
| **DEF-P1-05** | `PUB-01a`, `PUB-04c`, `FILE-01a`, `FILE-04c` | Backend `PublicIntakeService.ts:61`, `S3StorageService.ts`, `crypto.util.ts` | (1) `verifyRecaptcha` cho phép **replay** cùng token (`0.9` cả 2 lần); (2) `POST /public/forms/:slug/submit` tạo task (`201`) với attachment chưa qua `verifyUploadedObject` (`0` lần gọi); (3) `verifyMagicBytes` với `text/plain` chỉ kiểm offset 0 (`/* log */\n<script>` pass `true`); (4) `verifyOAuthState` cho phép **replay** state CSRF đã ký. | [wave2.intake-automations-files.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave2.intake-automations-files.test.ts) |
| **DEF-P1-06** | `AUTO-01a`, `AUTO-01b`, `AUTO-02` | Backend `AutomationEngine.ts`, `ActionExecutor.ts:59-60`, `automation.queue.ts:20` | (1) Ping-pong `Rule1 <-> Rule2` giữa 2 task chạy hết 5/5 lượt `SUCCESS`, thiếu ngắt `ABORTED_LOOP` ở độ sâu 3; (2) `CHANGE_STATUS` bỏ qua giới hạn WIP cột; (3) `automation.queue.ts` dùng mảng in-memory không giới hạn `const pending: AutomationJob[] = []` thay vì BullMQ/rate-limiter. | [wave2.intake-automations-files.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave2.intake-automations-files.test.ts) |
| **DEF-P1-07** | `RC-04`, `RC-05`, `RC-10`, `RC-11`, `RC-12` | Backend `SubtaskService.ts`, `BillingService.ts:99`, `prisma/schema.prisma`, Task/Project POST | (1) Roll-up subtask thiếu khóa `FOR UPDATE` trên task cha (`locksUsed: false`); (2) Schema & migrations thiếu partial unique index `WHERE status = 'ACTIVE'` cho `sprints` và unique `(recurrenceSeriesId, occurrenceIndex/date)`; (3) 2 webhook Stripe song song văng lỗi `P2002` vì check ngoài tx; (4) Header HTTP `Idempotency-Key` bị bỏ qua trên `POST /tasks` (tạo 2 task trùng). | [wave3.race-abuse-time.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave3.race-abuse-time.test.ts) |
| **DEF-P1-08** | `ABUSE-01`, `ABUSE-02` | Backend Pagination & `AnalyticsService` / `PortfolioController` | (1) `members`, `audit-logs`, `automations` chấp nhận **`200 OK`** cho `limit=1000000`, `limit=0`, `page=-5`; (2) Khoảng ngày **130 năm** (`1970-01-01..2099-12-31`) trả **`200 OK`** trên `portfolio/timeline`, `flow-metrics`, `throughput?groupBy=DAY` (**sinh 47,482 daily buckets!**) và `202` trên `analytics/export`. | [wave3.race-abuse-time.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave3.race-abuse-time.test.ts) |
| **DEF-P1-09** | `MCP-03a`, `MCP-03b`, `MCP-02b` | MCP `src/prompts/index.ts`, `src/tools/*.ts` | (1) Toàn bộ **4/4 prompt MCP** thiếu chỉ thị phân tách untrusted-content chống Indirect Prompt Injection; (2) `omnimer_get_task_details` trả nguyên `[SYSTEM OVERRIDE]` / `<script>` không bọc boundary `<untrusted_data>`; (3) **64/67 tool** thiếu `additionalProperties: false` & ràng buộc UUID ở `inputSchema`. | [mcp-attack-plan.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/omnimer_mcp_server/src/tests/mcp-attack-plan.test.ts) |
| **DEF-P1-10** | `FE-SEC-03`, `FE-SEC-04` *(+ Live Workers Probe)* | Frontend `next.config.mjs`, Cloudflare Workers edge, `src/stores/auth-store.ts:49,59,75` | (1) Cả Next.js local lẫn production Cloudflare Workers (`/login`, `/intake/demo`) thiếu hoàn toàn **`Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`** (và HSTS); (2) JWT được lưu trực tiếp vào `localStorage.getItem/setItem('access_token')`. | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) & Live `curl -sI` |

---

### 3.3. Lỗi Mức `P2` (Medium Severity — Chất lượng State, Hiệu năng UI, I18N, Timezone & Mutation Gate)

| Mã Lỗi | Attack IDs | Thành phần & Vị trí Mã nguồn | Mô tả Lỗi & Số liệu Thực nghiệm | Tệp Chứng minh |
| :--- | :--- | :--- | :--- | :--- |
| **DEF-P2-01** | `FE-STATE-01` | Frontend `task-detail-drawer-content.tsx` | Khi sửa task trên Task Detail Drawer bị backend trả `412 OCC_VERSION_CONFLICT` (`patchTaskCalls >= 1`), `OCCConflictModal` không hiển thị (`occModalVisible = false`). | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) |
| **DEF-P2-02** | `FE-STATE-02` | Frontend `src/stores/auth-store.ts` (toàn bộ `src/` thiếu `BroadcastChannel` / `storage` listener) | Đăng xuất ở Tab A xóa token trong `localStorage` nhưng Tab B đang mở không nhận sự kiện đồng bộ và vẫn ở lại trang dashboard (`redirectedToLogin: false`). | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) |
| **DEF-P2-03** | `FE-STATE-03` | Frontend `src/hooks/use-websocket.ts:241-270` (`ws.onopen`) | Khi WS bị ngắt và kết nối lại (`wsConnections.length >= 2`), `ws.onopen` reset `reconnectAttemptRef` và `SUBSCRIBE` lại room nhưng **không invalidate/refetch** dữ liệu (`taskListCallsAfterMount` vẫn là `1` == `beforeCalls`). Đồng thời `handleDomainEvent` (`lines 71-74`) chỉ dedup khi có `sequence_id`. | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) |
| **DEF-P2-04** | `FE-PERF-01`, `FE-PERF-01b` | Frontend `task-table-view.tsx:241-298`, `kanban-board.tsx` | Không sử dụng ảo hóa DOM (`@tanstack/react-virtual` / `react-window` vắng mặt): (1) Table 1,000 tasks mount đủ **`1,000` `<tr>`** (ngưỡng `<= 80`); (2) Kanban 500 thẻ phát sinh **`5` Long Tasks > 100ms** (ngưỡng `<= 2`). | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) |
| **DEF-P2-05** | `FE-I18N-01a`, `FE-I18N-01c` | Frontend `oql-query-input.tsx:129`, `subtask-tree.tsx`, `task-detail-drawer-content.tsx` & Backend `search/OqlCompiler.ts` | (1) Toàn bộ frontend thiếu kiểm tra `!e.nativeEvent.isComposing`: nhấn Enter gõ tiếng Việt IME trong `OqlQueryInput` kích hoạt sớm `POST /search/oql/validate` (`callsDuringComposing: 1`); (2) `OqlCompiler.compile` giữ nguyên chuỗi Unicode NFD `"Công việc Tiếng Việt"` thay vì chuẩn hóa `.normalize('NFC')`. | [attack-wave4-5.spec.ts](file:///Users/macbookair/Downloads/source/omnimer_team/frontend_otm/website_otm/tests/e2e/attack-wave4-5.spec.ts) & [wave5.triple-box-chaos.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave5.triple-box-chaos.test.ts) |
| **DEF-P2-06** | `TIME-01a`, `TIME-01b` | Backend `TimeTrackingService.ts`, `TaskService.ts` / `RecurrenceService` | (1) Stop timer lúc Chủ Nhật `17:30 UTC` (`00:30` Thứ Hai `+07:00`) bị gán `loggedDate = "2026-11-01"` theo ngày UTC thay vì `"2026-11-02"`; (2) Lặp lịch qua US DST `2026-03-08 America/New_York` bị dịch từ `09:00` (`13:00Z`) thành `10:00` (`14:00:00.000Z`). | [wave3.race-abuse-time.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave3.race-abuse-time.test.ts) |
| **DEF-P2-07** | `MUT-01`, `MUT-02`, `MUT-04` | Backend unit tests cho `HealthScoreEngine.ts`, `WorkloadCalculator.ts`, `search/OqlCompiler.ts` | **Mutation Score dưới ngưỡng `>= 80%`:** `HealthScoreEngine` đạt **`60%`** (`6/10`, sót `M4-ROR@L75`, `M5-ROR@L76`, `M6-ROR@L77`, `M7-LCR@L81`); `WorkloadCalculator` đạt **`60%`** (`6/10`, sót `M3-LCR@L223`, `M4-ROR@L223`, `M5-ROR@L223`, `M9-ROR@L248`); `OqlCompiler` chỉ đạt **`20%`** (`2/10`, sót 8 mutant `M2..M10`). *(Riêng `DependencyService` đạt `80%`).* | [wave5.triple-box-chaos.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave5.triple-box-chaos.test.ts) |

---

### 3.4. Lỗi Mức `P3` (Low / Contract Semantics & Cleanup Hygiene)

| Mã Lỗi | Attack IDs | Thành phần & Vị trí Mã nguồn | Mô tả Lỗi | Tệp Chứng minh |
| :--- | :--- | :--- | :--- | :--- |
| **DEF-P3-01** | `SEC-01`, `SEC-02`, `SEC-03`, `AUTHN-04`, `AUTHN-03`, `AUTHZ-XWS-FALLBACK` | `authorize.ts`, `workspaceContext.ts:52-97`, `InviteService.ts` | (1) `403` từ `SEC-01/02/03` trả `code: undefined` thay vì `'WORKSPACE_ACCESS_DENIED'` / `'INSUFFICIENT_PERMISSIONS'`; (2) Invite token hết hạn/đã dùng trả `400` (`code: undefined`) thay vì `410 GONE` / `'INVITE_TOKEN_INVALID_OR_EXPIRED'`; (3) User deactivated bị chặn đúng nhưng trả `403` thay vì `401`; (4) Slug workspace không tồn tại trả `400` thay vì `404` và `workspaceContext.ts:96-97` còn nhánh fallback sang `req.defaultWorkspaceId`. | [wave1.authz-matrix.test.ts](file:///Users/macbookair/Downloads/source/omnimer_team/backend_otm/project_core/src/__tests__/attack_plan/wave1.authz-matrix.test.ts) |

---

## 4. Các Cơ chế Phòng vệ Đã Đạt (Passed Security Controls)

Quá trình tấn công cũng ghi nhận những hàng rào đã hoạt động hiệu quả và chính xác:
1. **Authentication Coverage & JWT (`AUTHN-01`, `AUTHN-05`, Route Audit):** Toàn bộ `>= 150` endpoints được quét tự động đều gắn `authenticateJwt` và trả `401` khi thiếu token; token `alg:none`, sửa 1 byte chữ ký hoặc hết hạn đều bị chặn `401`; sai mật khẩu >= 5 lần khóa tài khoản `423` và chặn cả mật khẩu đúng sau đó.
2. **Path-Confusion IDOR (`AUTHZ-OBJ-02`) & WS Token Expiry (`WS-AUTH-02`):** Truy vấn chéo entity Project A2 qua path Project A1 trả `404`; WebSocket tự đóng `4401` khi JWT hết hạn.
3. **Public Form Rate Limit & Pinning (`PUB-02`, `PUB-03`), Export/File Security (`FILE-01c/02`, `FILE-03a`, `FILE-04a`, `FILE-04b`):** Public form giới hạn `429` sau 5 request/phút; spoofing `projectId`/`statusId` bị chặn `400` hoặc ghim về cấu hình form; Presigned URL TTL `<= 300s` (export `<= 24h` + `attachment`); XLSX Zip Bomb, XXE `<!ENTITY>`, CSV > 5,000 dòng & CSV Formula (`=CMD`) đều bị chặn/escape; Zip-Slip `../../app/.env` bị sanitize; 11 URL SSRF (`169.254.169.254`, `127.0.0.1`, RFC1918, decimal/hex localhost) bị chặn hoàn toàn.
4. **Single-Task OCC & Delete Race (`RC-01`, `RC-08`):** 10 request `PATCH /tasks/:id` đồng thời (`expectedVersion=1`) chỉ có đúng **1 `200 OK` và 9 `412`**; sửa song song với `DELETE` trả `404` trên task đã xóa.
5. **Triple-Box BVA, Cache, DTO Redaction, Frontend XSS & ReDoS (`BVA-01..03`, `GRAY-01..03`, `FE-SEC-01/02`, `FE-I18N-01b`, `CHAOS-03/04`, `MUT-03`):** Fibonacci `{1,2,3,5,8,13}`, biên file `25MB`/`20MB`, `HealthScoreEngine` `NO_DATA` / `[0..100]`, xóa cache metrics khi tạo task (`GRAY-01`), ẩn attachment `PENDING` (`GRAY-02`), kiểm tra đối chiếu mã nguồn (`WorkspaceService.ts:182-191`, `ProjectRepository.ts:150-156`, `AuthController.ts:124`) & chạy lại mock harness sau khi hỗ trợ nested `.select` xác nhận **không lộ `passwordHash`/`encryptedToken`/`lockedUntil`/`stripeCustomerId`** (`GRAY-03`); Frontend React renderer chặn thực thi `<script>`/`javascript:`/`<svg onload>` (`FE-SEC-01/02`) và giữ viewport ổn định trước chuỗi 10,000 ký tự không dấu cách (`FE-I18N-01b`); 40 OQL/FTS đồng thời chạy trong 89ms (`CHAOS-03`), ReDoS <= 100ms (`CHAOS-04`), `DependencyService` đạt `80%` mutation score (`MUT-03`), MCP từ chối `401` không lộ secret & chặn `VIEWER` trên 5 write/export tools (`MCP-01`, `MCP-02a`).

---

## 5. Giới hạn Công cụ & Phạm vi Ngoài Tầm (Environment Gaps)

Các công cụ CLI chuyên dụng sau **không được cài đặt** trong môi trường cục bộ tại thời điểm kiểm thử (`command -v` -> `MISSING`):
* `docker`, `k6`, `toxiproxy`, `semgrep`, `zap.sh`, `stryker`.

Do đó:
* Load/Soak test ở quy mô `k6` & fault injection mạng `toxiproxy` được thay bằng concurrent burst in-process (`Promise.all`, 40 OQL/FTS queries, 2,500 WS frames trên Playwright + CDP heap measurement).
* AST Mutation testing được thực thi bằng TypeScript AST mutant engine nội bộ trong `wave5.triple-box-chaos.test.ts` (mô phỏng các toán tử `ROR`/`LCR`, loại trừ điều kiện vòng lặp để tránh infinite loop) thay cho `@stryker-mutator/core`.
* Hệ thống sản xuất (Neon PostgreSQL, Render API, Cloudflare Workers) được bảo toàn tuyệt đối — mọi bài test ghi/phá tải/fuzz chạy cục bộ, chỉ gửi request read-only (`HEAD`/header/`OPTIONS`) tới production.

---

## 6. Lộ trình Khắc phục Ưu tiên (Actionable Remediation Roadmap)

1. **P0 Ngay lập tức (Wave 1–3 Hotfixes):**
   * Bổ sung middleware kiểm tra thành viên dự án/anti-IDOR (`404 NOT_FOUND`) trên `GET /projects/:projectId` và toàn bộ nested project/task routes; gắn `requireRole(['ADMIN', 'PM', 'MEMBER'])` vào `POST /projects`, `GET /portfolio*`, `GET /workload*`.
   * Khóa danh sách `CORS_ALLOWED_ORIGINS` tường minh trên backend thay vì phản xạ mọi `Origin` khi `credentials: true`.
   * Bắt buộc `expectedVersion` trên `PATCH /tasks/:id/move`; bọc chuyển trạng thái WIP và thêm dependency DAG trong transaction + `pg_advisory_xact_lock` / `SELECT ... FOR UPDATE`; kiểm tra quyền creator còn hiệu lực trong `AutomationEngine` (`PERMISSION_REVOKED`).
   * Vá `markdownSanitizer.ts` cho trường hợp `<tag/attr=...>`, encode `encodeURIComponent` + UUID strict check tại `OmniMerClient.ts` & `resources/index.ts`, chặn `.svg`/`.html` + escape HTML trong `PublicIntakeService.embedSnippet`.
2. **P1 Trước khi mở Beta/GA:**
   * Thêm `.strict()` cho 14 file DTO còn thiếu và loại bỏ `.passthrough()` ở `data-migration.dto.ts`; hoàn thiện `/auth/refresh` + SSO callback + gắn `authRateLimiter`; thực thi one-time cache cho reCAPTCHA token & OAuth `state` nonce; ép `verifyUploadedObject` khi submit intake; scan toàn bộ buffer `text/plain` chống padded HTML/script.
   * Chuyển `automation.queue.ts` sang BullMQ + Redis rate limiter; thêm `automationDepth <= 3` (`ABORTED_LOOP`); khóa parent task khi roll-up subtask; bổ sung partial unique index Sprint `ACTIVE` & Recurrence series trong Prisma; bọc `BillingService` xử lý webhook idempotent trong transaction/P2002 catch; chặn `limit > 100` & date-range `> 365` ngày; thêm CSP/XFO/nosniff/Referrer-Policy vào Next.js/Workers & chuyển JWT sang `HttpOnly` cookie; thêm `<untrusted_data>` framing + prompt boundary & `additionalProperties: false` trên MCP server.
3. **P2 Tối ưu hóa Sản phẩm & Bộ Test:**
   * Nối `OCCConflictModal` vào `TaskDetailDrawerContent` khi gặp `412`; đồng bộ logout đa tab bằng `BroadcastChannel` / `storage` event; invalidates query khi `useWebSocket.onopen` reconnect; áp dụng `@tanstack/react-virtual` cho Table/Kanban; thêm guard `!e.nativeEvent.isComposing` tại UI & `.normalize('NFC')` tại `OqlCompiler`; chuẩn hóa TZ/DST cho TimeTracking & Recurrence; bổ sung unit test tiêu diệt các mutant còn sống ở `OqlCompiler`, `HealthScoreEngine`, `WorkloadCalculator`.
