# Non-Functional Requirements

## 1. Mục tiêu tài liệu
- Xác định các yêu cầu chất lượng hệ thống ngoài chức năng nghiệp vụ.

## 2. Hiệu năng (Performance)
- Thời gian phản hồi API trung bình: `< 500ms` cho truy vấn phổ biến.
- P95 response time: `< 1200ms`.
- Hỗ trợ concurrent users mục tiêu:

## 3. Bảo mật (Security)
- JWT + role-based authorization.
- Hash mật khẩu theo chuẩn Identity mặc định.
- Validate input, chống SQL Injection/XSS/CSRF.
- Giới hạn kích thước upload.
- Audit log cho thao tác nhạy cảm.

## 4. Sẵn sàng và tin cậy (Availability & Reliability)
- Uptime mục tiêu:
- Cơ chế retry cho tác vụ ngoài hệ thống.
- Backup database định kỳ:

## 5. Logging & Monitoring
- Log level: `Information`, `Warning`, `Error`.
- Correlation ID cho request.
- Dashboard theo dõi lỗi runtime.

## 6. Khả năng mở rộng (Scalability)
- Tách read/write khi vượt ngưỡng.
- Cache cho dữ liệu đọc nhiều.
- Phân trang bắt buộc cho API list.

## 7. Dễ bảo trì (Maintainability)
- Coding conventions thống nhất.
- PR review bắt buộc.
- Unit test coverage tối thiểu:

## 8. Tương thích và UX
- Hỗ trợ trình duyệt mục tiêu:
- Responsive mobile/desktop.
- Accessibility cơ bản (semantic HTML, contrast).

## 9. Tuân thủ dữ liệu
- Quy định lưu trữ dữ liệu cá nhân.
- Thời gian giữ log và dữ liệu xóa mềm.

## 10. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

