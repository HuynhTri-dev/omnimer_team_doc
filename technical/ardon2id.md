# Technical Specification & Decision Record (ADR): Argon2id vs Bcryptjs

> **Tài liệu:** Quyết định Kiến trúc Bảo mật (Architecture Decision Record - ADR)  
> **Trạng thái:** ACCEPTED  
> **Ngày phê duyệt:** 2026-09-21  
> **Phạm vi áp dụng:** `backend_otm` (`project_core`) — Module Xác thực & Danh tính (AuthN / Identity)  
> **Tiêu chuẩn đối chiếu:** OWASP Password Storage Cheat Sheet, RFC 9106, NIST SP 800-63B, `security_audit_plan.md` (Domain 1)

---

## 1. Bối cảnh & Vấn đề Kỹ thuật (Context & Problem Statement)

Trong đợt rà soát bảo mật mã nguồn ([`dependency_security_audit.md`](file:///Users/macbookair/Downloads/omnimer_team/backend_otm/project_core/docs/security/dependency_security_audit.md)), hệ thống phát hiện module xác thực đang sử dụng thư viện `bcryptjs@3.0.3` với cấu hình cost factor (salt rounds) = 10.

* **Vi phạm chuẩn bảo mật:** Tiêu chuẩn tại `security_audit_plan.md` và OWASP yêu cầu độ phức tạp tính toán tối thiểu của Bcrypt phải có **Cost Factor $\ge 12$** ($2^{12} = 4,096$ vòng băm) để đủ sức chống lại các dàn máy đào card đồ họa (GPU clusters) giải mã vét cạn (brute-force) khi cơ sở dữ liệu bị rò rỉ.
* **Xung đột kiến trúc Node.js:** `bcryptjs` là bản port 100% bằng JavaScript thuần (Pure JS), không có C++ binding. Nó chạy hoàn toàn trên **luồng chính (V8 Main Thread / Event Loop)** của Node.js:
  * Nếu nâng lên cost factor $\ge 12$: Mỗi thao tác băm mật khẩu ngốn từ **400ms – 1,200ms CPU luồng chính**.
  * Trong thời gian này, toàn bộ tiến trình Node.js bị **đóng băng (CPU Starvation DoS)**: không tiếp nhận thêm request mới, treo healthcheck `/healthz`, làm tê liệt API của toàn bộ doanh nghiệp.
* **Mục tiêu:** Chuyển đổi sang **`argon2` (biến thể `argon2id`)** nhằm giải quyết triệt để vấn đề hiệu năng Event Loop mà vẫn đạt chuẩn bảo mật cao nhất hiện nay.

---

## 2. Bảng Ma trận So sánh Toàn diện (Comparison Matrix)

| Tiêu chí So sánh | `bcryptjs` (Pure JavaScript) | `argon2` (`argon2id` - C++ Native Binding) | Đánh giá & Rút ra |
| :--- | :--- | :--- | :--- |
| **Bản chất Mã nguồn** | 100% Pure JavaScript (chạy qua V8 engine). | Lõi C/C++ biên dịch mã máy native, liên kết qua Node-API (`node-addon-api`). | **Argon2 vượt trội**: Chạy mã máy Assembly trực tiếp trên CPU, tận dụng tập lệnh AVX2/SSE. |
| **Mô hình Đa luồng (Threading)** | **Đơn luồng (Single-thread)** trên V8 Main Thread. Dù gọi bất đồng bộ vẫn chiếm dụng CPU luồng chính. | **Đa luồng (Multi-threaded)**: Đẩy toàn bộ tác vụ băm sang `libuv worker thread pool` của Node.js. | **Argon2 vượt trội**: Luồng chính JavaScript rảnh tay 100% để phục vụ các HTTP request khác. |
| **Tác động tới Event Loop** | ❌ **Block Event Loop** nặng nề nếu cost factor $\ge 12$ (400ms - 1.2s/request). Dễ bị tấn công CPU DoS. | ✅ **Non-blocking**: Luồng chính không bao giờ bị nghẽn, máy chủ luôn phản hồi mượt mà. | **Argon2 an toàn tuyệt đối** cho tính sẵn sàng (Availability - NFR-REL). |
| **Cơ chế Kháng bẻ khóa GPU/ASIC** | ⚠️ **Chỉ tốn CPU, không tốn RAM** (Memory-free). Các dàn GPU/ASIC hàng nghìn lõi crack rất nhanh. | ✅ **Memory-Hard**: Bắt buộc cấp phát RAM (ví dụ 64MB) cho mỗi phép tính băm. | **Argon2id vượt trội**: Dàn GPU bị nghẽn băng thông bộ nhớ, triệt tiêu ưu thế tính toán song song của hacker. |
| **Tính linh hoạt Tham số** | Chỉ có 1 tham số duy nhất: Cost Factor (số vòng lặp thời gian). | Cấu hình 3 chiều độc lập: **Time Cost** ($t$), **Memory Cost** ($m$), **Parallelism** ($p$). | **Argon2id vượt trội**: Tùy biến chính xác theo cấu hình phần cứng của server. |
| **Chuẩn Tổ chức & Năm ra đời** | 1999 (Niels Provos & David Mazières). Quá cũ so với năng lực tính toán hiện đại. | 2015 (Quán quân cuộc thi *Password Hashing Competition - PHC*). Được chuẩn hóa tại RFC 9106. | **Argon2id là tiêu chuẩn hiện đại nhất**. |
| **Khuyến nghị Quốc tế** | Không được khuyến nghị cho hệ thống mới. | OWASP Password Storage Cheat Sheet khuyến nghị số 1; NIST SP 800-63B khuyến nghị. | **Argon2id đạt chuẩn Enterprise**. |
| **Dependencies & Cài đặt** | ✅ **0 dependency ngoài**, không cần compiler C++, chạy được trên Edge Worker / Browser. | ⚠️ Phụ thuộc `node-addon-api`, `node-gyp-build`, `@phc/format`. Cần môi trường build C++ khi đóng gói Docker. | `bcryptjs` tiện dụng hơn khi cài đặt, nhưng `argon2` chuẩn chỉnh hơn cho máy chủ Backend. |

---

## 3. Lý giải Kỹ thuật: Vì sao Argon2 tối ưu hơn dù có nhiều dependencies?

Nhiều lập trình viên băn khoăn vì sao `argon2` kéo theo các dependencies như `@phc/format`, `node-addon-api`, `node-gyp-build` mà lại tối ưu hơn một thư viện "sạch bóng dependency" như `bcryptjs`:

```
                           YÊU CẦU BĂM MẬT KHẨU
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
┌─────────────────────────────────┐           ┌─────────────────────────────────┐
│     bcryptjs (0 dependencies)   │           │      argon2 (C++ Node-API)      │
├─────────────────────────────────┤           ├─────────────────────────────────┤
│ • Code JavaScript thuần         │           │ • Node-API đẩy sang libuv       │
│ • V8 Main Thread cày thuật toán │           │ • 4 luồng C++ chạy ngầm ở OS    │
│ • Chiếm 100% CPU Event Loop     │           │ • V8 Main Thread rảnh tay       │
│ • Máy chủ đóng băng 0.5s - 1s   │           │ • Máy chủ nhận tiếp 1,000 req   │
└─────────────────────────────────┘           └─────────────────────────────────┘
         │                                                     │
         ▼                                                     ▼
❌ Treo Server / Nghẽn API                      ✅ Mượt mà / Chuẩn Enterprise
```

1. **`node-addon-api`**: Là giao diện C++ chính thức được Node.js duy trì (N-API). Nó không phải là code JS thừa thãi, mà là cây cầu kết nối code C++ cấp thấp với hệ điều hành.
2. **`node-gyp-build`**: Công cụ nạp trực tiếp file nhị phân compiled `.node`. Lúc chạy production, ứng dụng nạp file binary chạy thẳng trên CPU, **hoàn toàn không thông qua bộ thông dịch JavaScript**.
3. **`@phc/format`**: Một module tiện ích siêu nhẹ (dưới 50 dòng code) chuẩn hóa chuỗi băm theo quy ước chung của cuộc thi quốc tế Password Hashing Competition.

---

## 4. Cấu hình Tiêu chuẩn Khuyến nghị cho OmniProject (`project_core`)

Tuân thủ khuyến nghị của OWASP Password Storage Cheat Sheet cho hệ thống Backend chạy Node.js:

```typescript
import argon2 from 'argon2';

export const ARGON2_CONFIG: argon2.Options = {
  type: argon2.argon2id,  // Biến thể argon2id: Chống cả Side-Channel Attacks và GPU Cracking
  memoryCost: 65536,      // 64 MB RAM (Khắc chế tuyệt đối phần cứng GPU của hacker)
  timeCost: 3,            // 3 iterations (Đảm bảo độ trễ tính toán an toàn ~100ms trên luồng C++)
  parallelism: 1,         // 1 luồng tính toán trên mỗi thao tác hash
};
```

---

## 5. Chiến lược Di chuyển Không Gián đoạn (Zero-Downtime Migration)

Khi nâng cấp từ `bcryptjs` sang `argon2id`, trong cơ sở dữ liệu đã có các tài khoản cũ lưu mật khẩu ở định dạng `$2a$` / `$2b$`. Hệ thống áp dụng cơ chế **Rehash on Login (Nâng cấp ngầm)**:

```
User bấm Đăng nhập ──► Tìm User trong DB
                           │
                           ▼
              Mật khẩu bắt đầu bằng '$argon2'?
                 │                       │
               [YES]                    [NO] (Mật khẩu cũ bcrypt)
                 │                       │
                 ▼                       ▼
           argon2.verify()         bcrypt.compare()
                 │                       │
                 │                 Khớp mật khẩu?
                 │                  ├── [SAI]: Báo lỗi 401
                 │                  └── [ĐÚNG]: 
                 │                        │
                 │                        ▼
                 │                 argon2.hash(password)
                 │                        │
                 │                        ▼
                 │                 Cập nhật hash mới vào DB
                 │                 (User được nâng cấp ngầm)
                 ▼                        │
          Cấp phát JWT Token ◄────────────┘
```

### Ưu điểm của chiến lược Rehash on Login:
* **Không cần reset mật khẩu người dùng**: Người dùng vẫn đăng nhập bình thường bằng mật khẩu cũ.
* **Nâng cấp lũy tiến (Progressive Upgrade)**: Bất kỳ người dùng nào đang hoạt động (active users) khi đăng nhập sẽ tự động được chuyển đổi sang `argon2id`.
* **Không làm gián đoạn cơ sở dữ liệu**: Không cần chạy script di chuyển hàng loạt dữ liệu mật khẩu khi chưa có bản rõ (plaintext password).

---

## 6. Kết luận & Quyết định

* Chấp thuận thay thế hoàn toàn `bcryptjs` bằng **`argon2`** trong toàn bộ module `project_core`.
* Khai báo thông số băm mật khẩu chuẩn OWASP trong service quản lý mật khẩu trung tâm (`src/core/services/PasswordService.ts`).
* Ghi nhận quyết định này vào tài liệu kiến trúc kỹ thuật của dự án OmniProject.
