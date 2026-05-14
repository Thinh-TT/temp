# Non-Functional Requirements

## 1. Mục tiêu tài liệu

- Xác định các yêu cầu chất lượng hệ thống ngoài chức năng nghiệp vụ.
- Đồng bộ với công nghệ đã chọn trong `1-ideas-and-scope.md`: ASP.NET Core Web API (.NET 8), Entity Framework Core, SQL Server, Razor Pages + Bootstrap 5.
- Làm căn cứ cho kiến trúc sư, devops và tester đánh giá chất lượng hệ thống.

## 2. Hiệu năng (Performance)

### 2.1 Thời gian phản hồi

| Chỉ số                       | Mục tiêu   | Ghi chú                                   |
| ---------------------------- | ---------- | ----------------------------------------- |
| API response time trung bình | `< 300ms`  | Cho endpoint GET list/GET detail phổ biến |
| API response time P95        | `< 800ms`  | Tính trên 95% request                     |
| API response time P99        | `< 1500ms` | Cho phép spike với query nặng (dashboard) |
| Trang Razor Page load        | `< 2s`     | First contentful paint                    |
| Time to Interactive          | `< 3s`     | Trên desktop                              |

### 2.2 Database query

| Chỉ số                                              | Mục tiêu  |
| --------------------------------------------------- | --------- |
| Truy vấn đơn giản (lookup by ID, unique index)      | `< 50ms`  |
| Truy vấn danh sách có phân trang (có join 2-3 bảng) | `< 200ms` |
| Truy vấn dashboard tổng hợp (aggregate)             | `< 500ms` |

### 2.3 Concurrent users mục tiêu

| Mức           | Concurrent Users | Ghi chú                      |
| ------------- | ---------------- | ---------------------------- |
| Giai đoạn MVP | 50-100           | Đủ cho phạm vi đồ án/môn học |
| Mở rộng       | 500-1000         | Nếu triển khai thực tế       |

### 2.4 Biện pháp tối ưu

- Phân trang bắt buộc cho mọi API list (pageSize mặc định 10, tối đa 50).
- Index database theo đúng thiết kế trong `4-erd-and-db-rules.md` mục 9.
- Sử dụng `AsNoTracking()` cho Entity Framework query read-only.
- Cache dữ liệu đọc nhiều: `JobCategories`, `PostCategories`, `Skills`, `Tags` (dữ liệu ít thay đổi).
- Nén response (gzip/brotli) cho nội dung HTML và JSON lớn.
- Bundle & minify CSS/JS cho Razor Pages.

## 3. Bảo mật (Security)

### 3.1 Authentication & Authorization

- JWT Bearer token với thời hạn hợp lý (access token: 1-24h, đề xuất 8h cho phiên làm việc).
- Refresh token (tùy chọn) với thời hạn dài hơn (7 ngày).
- Role-based authorization: `Admin`, `Employer`, `Candidate`.
- Policy-based authorization cho các rule sở hữu (chỉ employer sở hữu job mới được sửa).

### 3.2 Mật khẩu

- Hash mật khẩu theo chuẩn ASP.NET Identity (PBKDF2 với salt, iterations mặc định).
- Password policy:
  - Độ dài tối thiểu: 8 ký tự.
  - Yêu cầu: ít nhất 1 chữ hoa, 1 chữ thường, 1 số, 1 ký tự đặc biệt.
- Không lưu password plaintext ở bất kỳ đâu.

### 3.3 Input validation & injection prevention

- Validate toàn bộ input tại tầng API (Data Annotation / FluentValidation).
- Entity Framework Core parameterized query — chống SQL Injection.
- Sanitize HTML từ rich text editor trước khi lưu (chống XSS stored).
- Mã hóa HTML entities khi render nội dung người dùng (chống XSS reflected).
- Anti-forgery token cho tất cả Razor Page form POST (ASP.NET tự động).

### 3.4 File upload security

- Giới hạn kích thước file:
  - CV PDF: tối đa `5MB`.
  - Avatar: tối đa `2MB`.
  - Logo: tối đa `2MB`.
  - Thumbnail: tối đa `2MB`.
- Chỉ chấp nhận extension:
  - CV: `.pdf`.
  - Ảnh: `.jpg`, `.jpeg`, `.png`, `.webp`.
- Validate MIME type thực tế của file (không chỉ dựa vào extension).
- Lưu file với tên được rename (GUID hoặc hash) để tránh path traversal.
- Không lưu file trong thư mục web root; dùng thư mục riêng ngoài `wwwroot` hoặc blob storage.

### 3.5 HTTPS & headers

- Bắt buộc HTTPS trong môi trường production.
- Security headers:
  - `Strict-Transport-Security` (HSTS)
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Content-Security-Policy` (CSP) cơ bản
  - `Referrer-Policy: strict-origin-when-cross-origin`

### 3.6 Rate limiting (cho production)

- Giới hạn request cho endpoint nhạy cảm:
  - `/auth/login`: 5 lần/phút/IP.
  - `/auth/register`: 3 lần/phút/IP.
- Giới hạn upload file: 10 lần/phút/user.

### 3.7 Audit trail

- Ghi log cho các thao tác nhạy cảm:
  - Thay đổi trạng thái job/post (ai duyệt, duyệt lúc nào).
  - Thay đổi trạng thái application.
  - Xóa mềm và khôi phục dữ liệu.
  - Thay đổi role người dùng (nếu có).

## 4. Sẵn sàng và tin cậy (Availability & Reliability)

### 4.1 Uptime mục tiêu

| Môi trường           | Uptime        | Ghi chú                                           |
| -------------------- | ------------- | ------------------------------------------------- |
| Development          | Không cam kết | Môi trường dev local                              |
| Production (MVP)     | 99.5%         | ~3.6 giờ downtime/tháng, chấp nhận được cho đồ án |
| Production (mở rộng) | 99.9%         | ~43 phút downtime/tháng                           |

### 4.2 Xử lý lỗi

- Global exception handler: bắt tất cả exception chưa handled, trả `500 INTERNAL_ERROR`, không leak stack trace.
- Validation error: trả `400` với details theo chuẩn `5-api-spec.md` mục 3.4.
- Circuit breaker cho external service (nếu có tích hợp email, storage).
- Retry policy cho transient failure (SQL connection, file I/O).

### 4.3 Backup & Recovery

- Backup database định kỳ: hàng ngày (daily full backup).
- Giữ backup 7 ngày gần nhất.
- Soft delete (thay vì hard delete) cho dữ liệu nghiệp vụ chính: `Jobs`, `Posts`, `PostComments`.
- Khả năng restore dữ liệu đã xóa mềm thông qua admin interface.

### 4.4 Graceful degradation

- Dashboard admin: nếu một data source lỗi, hiển thị thông báo lỗi cục bộ thay vì crash toàn trang.
- Upload file: nếu storage tạm lỗi, giữ nguyên URL cũ, không ghi đè bằng null.
- Community feed: nếu trending query chậm, fallback về newest.

## 5. Logging & Monitoring

### 5.1 Log levels

| Level         | Sử dụng cho                                                           |
| ------------- | --------------------------------------------------------------------- |
| `Information` | Request/response, login thành công, CRUD thành công, state transition |
| `Warning`     | Validation error, 404, 401, 403, file upload bị từ chối               |
| `Error`       | Exception chưa handled, database connection failure, storage failure  |
| `Critical`    | Hệ thống không thể khởi động, migration failure                       |

### 5.2 Cấu trúc log

- Dùng structured logging (Serilog hoặc tương tự).
- Mỗi log entry phải có:
  - `Timestamp` (ISO 8601 UTC)
  - `Level`
  - `CorrelationId` (xuyên suốt 1 request)
  - `UserId` (nếu đã xác thực)
  - `Message`
  - `Exception` (nếu có)

### 5.3 Correlation ID

- Tạo GUID cho mỗi request từ middleware đầu tiên.
- Gắn vào response header `X-Correlation-Id`.
- Log correlation ID trong tất cả log entry của request đó.

### 5.4 Monitoring (cho production)

- Health check endpoint: `GET /health` trả `200 OK` nếu database và storage khả dụng.
- Application performance monitoring (APM) cơ bản: thời gian response, error rate.
- Dashboard lỗi runtime: tổng hợp 5xx, 4xx theo endpoint.

## 6. Khả năng mở rộng (Scalability)

### 6.1 Kiến trúc hiện tại

- Monolithic ASP.NET Core — phù hợp với quy mô MVP.
- Razor Pages server-rendered + REST API cho tương tác động.

### 6.2 Điểm mở rộng tương lai

- Tách read/write: dùng read replica cho query list/dashboard khi traffic tăng.
- Cache phân tán (Redis) thay cho in-memory cache khi có nhiều instance.
- CDN cho static assets và file upload.
- Background job (Hangfire/Quartz) cho tác vụ bất đồng bộ: gửi email notification, cleanup.

### 6.3 Giới hạn hiện tại đã áp dụng

- Phân trang bắt buộc cho API list — tránh load toàn bộ bảng.
- `pageSize` tối đa: 50 bản ghi/trang.
- Chỉ hỗ trợ filter cơ bản: keyword, category, location, status.
- Không hỗ trợ full-text search khi chưa bật tính năng nâng cao.

## 7. Dễ bảo trì (Maintainability)

### 7.1 Coding conventions

- Tuân thủ C# coding conventions của Microsoft.
- Đặt tên rõ ràng: PascalCase cho class/method, camelCase cho variable/param.
- Controller: mỏng, chỉ xử lý HTTP concern. Business logic nằm trong Service layer.
- DTO/ViewModel riêng cho từng endpoint, không expose entity trực tiếp.

### 7.2 Tổ chức source code

```
/src
  /Project.Web          # Razor Pages, Controllers
  /Project.Application  # Service interfaces & implementations, DTOs
  /Project.Domain       # Entity models, Enums, Business rules
  /Project.Infrastructure # EF Core DbContext, Migrations, Repositories
```

### 7.3 Testing

| Loại test                 | Coverage mục tiêu | Ghi chú                                                             |
| ------------------------- | ----------------- | ------------------------------------------------------------------- |
| Unit test (Service layer) | `>= 70%`          | Test business logic, validation, state machine                      |
| Integration test (API)    | `>= 50%`          | Test endpoint critical: auth, job CRUD, apply, application workflow |
| UI test                   | Không bắt buộc    | Manual test với checklist là đủ cho quy mô hiện tại                 |

### 7.4 Database migration

- Mỗi thay đổi schema có migration riêng, tên mô tả rõ thay đổi.
- Không sửa migration đã chạy trên môi trường dùng chung.
- Seed data tách riêng, có thể chạy lại nhiều lần không lỗi.

### 7.5 Code review

- PR review bắt buộc trước khi merge vào `main`.
- Checklist review tối thiểu:
  - Không có secret/credential trong code.
  - Input validation đầy đủ.
  - Đúng authorization role.
  - Có xử lý lỗi phù hợp.
  - Không có migration thừa hoặc sai.

## 8. Tương thích và UX

### 8.1 Trình duyệt hỗ trợ

| Trình duyệt             | Phiên bản tối thiểu |
| ----------------------- | ------------------- |
| Chrome                  | 100+                |
| Firefox                 | 100+                |
| Edge                    | 100+                |
| Safari                  | 15+                 |
| Mobile Chrome (Android) | 100+                |
| Mobile Safari (iOS)     | 15+                 |

### 8.2 Responsive design

- Breakpoints Bootstrap 5 mặc định:
  - `xs`: `< 576px`
  - `sm`: `≥ 576px`
  - `md`: `≥ 768px`
  - `lg`: `≥ 992px`
  - `xl`: `≥ 1200px`
- Mobile-first approach.
- Hamburger menu cho navigation trên mobile.
- Bảng dữ liệu (table) chuyển thành card/list trên mobile.

### 8.3 Accessibility (WCAG 2.1 Level AA cơ bản)

- Semantic HTML (`<nav>`, `<main>`, `<header>`, `<footer>`, `<article>`).
- `alt` text cho tất cả hình ảnh.
- `aria-label` cho button chỉ chứa icon.
- Color contrast ratio: tối thiểu `4.5:1` cho normal text, `3:1` cho large text.
- Focus indicator rõ ràng cho keyboard navigation.
- Form input có `<label>` liên kết đúng.
- Toast notification có `role="alert"`.

### 8.4 Trải nghiệm người dùng

- Thời gian toast hiển thị: 5 giây cho success/info, không tự động ẩn cho error.
- Confirm modal cho hành động phá hủy (xóa, thay đổi trạng thái không thể hoàn tác).
- Button CTA luôn visible, không bị ẩn dưới fold trên mobile.
- Form validation: validate inline khi blur field, validate toàn bộ khi submit.
- Không scroll hijacking, không auto-play video.

## 9. Tuân thủ dữ liệu

### 9.1 Dữ liệu cá nhân

- Dữ liệu cá nhân (họ tên, email, CV, ảnh đại diện) chỉ được xử lý theo mục đích nghiệp vụ.
- CV chỉ hiển thị cho: chính chủ, employer của job mà candidate đã apply, admin.
- CV snapshot được lưu riêng cho từng lần apply; candidate có thể cập nhật CV trong hồ sơ mà không ảnh hưởng đến CV đã nộp.

### 9.2 Lưu trữ và xóa dữ liệu

- Soft delete data được giữ trong hệ thống để phục vụ audit/khôi phục.
- Thời gian giữ dữ liệu đã xóa mềm: 90 ngày, sau đó có thể hard delete thủ công bởi admin.
- Log file: giữ tối thiểu 30 ngày.
- Backup database: giữ 7 ngày.

### 9.3 Quyền riêng tư

- Không share dữ liệu người dùng với bên thứ ba.
- Không tracking/analytics third-party (trong phạm vi đồ án).
- Cookie chỉ dùng cho mục đích xác thực (session/token), không dùng cho quảng cáo.

## 10. Ràng buộc công nghệ

| Ràng buộc         | Chi tiết                                                              |
| ----------------- | --------------------------------------------------------------------- |
| Backend framework | ASP.NET Core Web API (.NET 8)                                         |
| ORM               | Entity Framework Core 8                                               |
| Database          | SQL Server                                                            |
| Authentication    | ASP.NET Core Identity + JWT                                           |
| Frontend          | Razor Pages + Bootstrap 5                                             |
| Rich text editor  | Quill.js                                                              |
| Chart             | Chart.js                                                              |
| File storage      | Local file system (dev) / Azure Blob hoặc thư mục shared (production) |
| Version control   | Git                                                                   |

## 11. Ma trận rủi ro kỹ thuật

| Rủi ro                                  | Mức độ     | Biện pháp giảm thiểu                                         |
| --------------------------------------- | ---------- | ------------------------------------------------------------ |
| SQL injection                           | Thấp       | EF Core parameterized query, input validation                |
| XSS từ rich text editor                 | Trung bình | HTML sanitize trước lưu và render                            |
| Lộ CV/dữ liệu riêng tư                  | Trung bình | Authorization check kỹ ở API, không lộ CV qua public profile |
| File upload malicious                   | Trung bình | Validate MIME type, giới hạn kích thước, lưu ngoài web root  |
| Performance degradation với dữ liệu lớn | Thấp       | Phân trang, index database, AsNoTracking                     |
| JWT token bị đánh cắp                   | Thấp       | HTTPS, token expiry hợp lý                                   |
| Soft delete data leak                   | Thấp       | Query mặc định luôn lọc `IsDeleted = false`                  |

## 12. Changelog

| Version | Ngày       | Người cập nhật / Tên Model | Nội dung                                                                                                                                                                   |
| ------- | ---------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.1     | YYYY-MM-DD |                            | Khởi tạo khung                                                                                                                                                             |
| 1.0     | 2026-05-14 | DeepSeek V4 Pro            | Hoàn thiện non-functional requirements: hiệu năng, bảo mật, availability, logging, scalability, maintainability, UX, tuân thủ dữ liệu, ràng buộc công nghệ, ma trận rủi ro |
