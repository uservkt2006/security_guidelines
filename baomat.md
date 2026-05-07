# AI Security Skill: React Hardening & Auto-Vulnerability Fixer 
# Tác giả: @vokhactam | vokhactam.user@gmail.com
# Version: 1.0.0
# Ngày: 2026-05-07

## Mục tiêu
Skill này cung cấp cho AI khả năng tự động:
- Phát hiện các lỗ hổng bảo mật thường gặp trong code React.
- Sửa chữa (vá) lỗ hổng ngay khi sinh mã hoặc khi review code.
- Áp dụng cấu hình và thực hành bảo mật mặc định cho mọi dự án React.
- Đảm bảo ứng dụng được tạo ra an toàn trước các tấn công thực tế (XSS, CSRF, injection, RCE, dependency vulnerabilities...).

## Phạm vi áp dụng
- Code frontend React (JSX, functional components, hooks).
- Cấu hình build (Webpack, Vite, Next.js).
- Quản lý dependencies (`package.json`).
- Tương tác với backend APIs (cảnh báo và hướng dẫn).

## Nguyên tắc cốt lõi (Security Golden Rules)
1. **Không bao giờ tin dữ liệu từ người dùng** – luôn escape, validate, sanitize.
2. **Cấm dùng `dangerouslySetInnerHTML`** trừ khi bắt buộc và đã sanitize bằng DOMPurify.
3. **Chặn URL giao thức `javascript:`** trong các thuộc tính `href`, `src`, `action`.
4. **CSRF protection bắt buộc** nếu API dùng cookie-based auth.
5. **Cập nhật dependencies liên tục** – tự động audit và đề xuất nâng cấp.
6. **Không hardcode secret/key** trong client code.
7. **Sử dụng TypeScript** để ngăn chặn XSS qua kiểu dữ liệu.

## Quy trình tự động (Auto-Fix Pipeline)
Khi AI sinh code hoặc nhận yêu cầu tạo website React, thực hiện tuần tự:

### Bước 0: Kiểm tra phiên bản React và dependencies
- Chạy lệnh ảo: `npm list react react-dom next` → xác định version.
- Nếu phát hiện phiên bản có lỗ hổng nghiêm trọng (ví dụ React 19.0.0 → 19.2.2):
    - Tự động nâng cấp trong `package.json` lên phiên bản đã vá (19.0.3, 19.1.4, 19.2.3 trở lên).
    - Thêm script `"audit": "npm audit fix"` vào `package.json`.
- Quét toàn bộ dependencies bằng `npm audit` (giả lập) – nếu có lỗ hổng high/critical, tự động cập nhật phiên bản an toàn.

### Bước 1: Phát hiện và sửa XSS
**Pattern cần tìm trong code:**
- `dangerouslySetInnerHTML={{ __html: ... }}` với biến chưa qua sanitize.
- `<a href={userInput}>`, `<img src={userInput}>` không có validation.
- `eval(userInput)`, `new Function(userInput)`.

**Hành động tự động:**
- Thay thế `dangerouslySetInnerHTML` bằng component an toàn:
```javascript
import DOMPurify from 'dompurify';
const SafeHTML = ({ html }) => <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />;
```
- Thêm hàm `sanitizeUrl` để kiểm tra giao thức:
```javascript
const sanitizeUrl = (url) => {
  const safe = ['http:', 'https:', 'mailto:', 'tel:'];
  try {
    const parsed = new URL(url);
    if (!safe.includes(parsed.protocol)) return '#';
  } catch { return '#'; }
  return url;
};
```
- Tự động wrap tất cả `href={variable}` bằng `href={sanitizeUrl(variable)}`.

### Bước 2: CSRF protection
- Nếu phát hiện dùng `fetch` hoặc `axios` với `credentials: 'include'`:
    - Tự động thêm header `X-CSRF-Token` lấy từ cookie hoặc meta tag.
    - Hướng dẫn backend implement CSRF token (nếu AI có thể sửa cả backend).
- Thêm middleware: sử dụng thư viện `csurf` (Express) hoặc `django.middleware.csrf`.

### Bước 3: Ngăn chặn injection (gián tiếp)
- React frontend không trực tiếp SQL, nhưng nếu AI sinh API endpoint:
    - Bắt buộc dùng parameterized queries (ví dụ: không nối chuỗi `"SELECT * FROM users WHERE id = " + userId`).
    - Sử dụng ORM (Prisma, TypeORM) hoặc query builder an toàn.
- Nếu phát hiện code nối chuỗi trong fetch URL: chuyển sang dùng `URLSearchParams` hoặc `encodeURIComponent`.

### Bước 4: Xác thực và phân quyền mạnh
- Luôn thêm logic kiểm tra token JWT hoặc session ở mọi request cần bảo vệ.
- Không cho phép IDOR: kiểm tra quyền truy cập tài nguyên (ví dụ: `userId !== req.user.id`).
- Sinh code mặc định cho `AuthContext` và `ProtectedRoute`.

### Bước 5: Bảo mật API từ frontend
- Rate limiting đề xuất: Thêm `axios-retry` và thông báo lỗi 429.
- Validate tất cả input ngay trên client (dùng Zod hoặc Yup) nhưng không thay thế server validation.
- Tự động che giấu API keys – không cho phép hardcode; nhắc dùng biến môi trường `VITE_*` hoặc `NEXT_PUBLIC_*` nhưng cảnh báo rằng không an toàn 100%.

### Bước 6: Vá lỗ hổng React2Shell và RSC (nếu dùng Next.js App Router)
- Kiểm tra `next.config.js`: có `experimental.reactCompiler` hoặc dùng Server Components?
- Nếu có, bắt buộc:
    - Nâng cấp lên `react@19.0.3+`, `react-dom@19.0.3+`, `next@15.0.4+` (hoặc phiên bản vá tương ứng).
    - Thêm cấu hình CSP header để giảm thiểu tác động nếu chưa kịp vá.
    - Không sinh code gửi dữ liệu người dùng trực tiếp vào Server Actions mà chưa validate.

### Bước 7: Dependency auto-healing
- Tự động thêm vào `package.json`:
```json
"scripts": {
  "security-check": "npm audit && npx snyk test",
  "preinstall": "npx only-allow npm"
}
```
- Nếu phát hiện package lỗi (ví dụ lodash 4.17.x có CVE), tự động nâng lên phiên bản vá.
- Sử dụng `overrides` in `package.json` để ép phiên bản an toàn khi không thể nâng trực tiếp.

### Bước 8: Tạo báo cáo bảo mật cuối cùng
Sau khi sinh code, AI xuất ra file `SECURITY_AUDIT_REPORT.md` bao gồm:
- Danh sách lỗ hổng đã phát hiện và tự động sửa.
- Phiên bản các package đã nâng cấp.
- Các khuyến nghị chưa tự động làm được (ví dụ: thiết lập CSP header, https chuyển hướng).

## Ví dụ: AI tự động sửa lỗi trực tiếp trong code

**Input code (không an toàn):**
```javascript
function Comment({ userInput }) {
  return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
}
```

**Output sau khi áp dụng skill:**
```javascript
import DOMPurify from 'dompurify';

function Comment({ userInput }) {
  const sanitized = DOMPurify.sanitize(userInput);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

**Input code (XSS qua href):**
```javascript
<a href={userProvidedLink}>Click</a>
```

**Output:**
```javascript
import { sanitizeUrl } from './utils/security';

<a href={sanitizeUrl(userProvidedLink)}>Click</a>
```

## Cấu hình mặc định cho một dự án React mới (template an toàn)
Khi AI tạo mới project, tự động thêm:
- `package.json` với các script security.
- File `.env.example` chứa biến môi trường mẫu (không có secret thật).
- File `utils/security.js` chứa `sanitizeUrl`, `sanitizeHtml`.
- File `middleware/csrf.js` nếu dùng Next.js.
- File `security-headers.js` (CSP, HSTS, X-Frame-Options) cho phần server (Next.js hoặc Express).

## Kiểm tra cuối cùng (Zero-trust checklist)
Trước khi xuất code, AI tự hỏi:
- [ ] Có bất kỳ `dangerouslySetInnerHTML` nào không được bao bọc bởi DOMPurify không?
- [ ] Có thuộc tính `href`/`src` nào nhận biến người dùng mà không qua `sanitizeUrl` không?
- [ ] Dependency đã được update lên phiên bản không có CVE critical chưa?
- [ ] Server components có dùng React 19.x nhỏ hơn 19.0.3 không?
- [ ] CSRF token được gửi trong các request thay đổi dữ liệu?
- [ ] Không có secret hardcode trong client code?

Nếu tất cả đều OK → xuất code kèm file báo cáo bảo mật.
