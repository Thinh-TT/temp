# Ý Tưởng Và Phạm Vi Đề Tài

## 1. Tên Đề Tài
Nền tảng tuyển dụng và cộng đồng nghề nghiệp.

## 2. Ý Tưởng Sản Phẩm
Xây dựng ứng dụng web kết hợp:
- Nền tảng tuyển dụng (đăng tin, tìm việc, ứng tuyển).
- Cộng đồng chia sẻ kiến thức nghề nghiệp (bài viết, thảo luận, tương tác).

Định hướng tương tự:
- LinkedIn/TopCV cho tuyển dụng.
- Kết hợp tính năng cộng đồng như diễn đàn nghề nghiệp.

## 3. Phạm Vi Tổng Thể
- Core system: Recruitment Platform.
- Support module: Community.
- Tỷ trọng đề xuất: `70% Recruitment + 30% Community`.

## 4. Công Nghệ Sử Dụng
- ASP.NET Core Web API (.NET 8)
- Entity Framework Core
- SQL Server
- ASP.NET Identity
- Razor Pages + Bootstrap 5
- Quill.js
- Chart.js

## 5. Tính Năng Bắt Buộc (MVP)

### 5.1 Authentication & Authorization
- Register
- Login
- Logout
- Phân quyền cơ bản:
- `Candidate`
- `Employer`
- `Admin`

### 5.2 Hồ Sơ Người Dùng
- Candidate:
- Avatar
- Thông tin cá nhân
- Danh sách skills
- Upload CV PDF
- Employer:
- Company name
- Logo
- Description

### 5.3 Recruitment (Core)
- Employer:
- CRUD Job
- Mô tả công việc
- Yêu cầu
- Mức lương
- Hạn nộp hồ sơ
- Category
- Candidate:
- Xem danh sách job
- Xem chi tiết job
- Apply job
- Save/Bookmark job
- Admin:
- Duyệt job
- Ẩn/xóa job vi phạm

### 5.4 Job Search
- Tìm kiếm theo từ khóa
- Lọc theo category
- Lọc theo location

### 5.5 Application Workflow
- Trạng thái đơn ứng tuyển:
- `Pending`
- `Reviewed`
- `Interview`
- `Rejected`
- `Accepted`
- Employer có thể:
- Xem danh sách ứng viên
- Cập nhật trạng thái đơn

### 5.6 Community Module
- Bài viết:
- Tạo bài viết
- Sửa/xóa bài của chính mình
- Gắn category/tag
- Feed:
- Newest
- Trending đơn giản (ví dụ: `score = upvote + commentCount`)
- Tương tác:
- Comment
- Upvote
- Bookmark post

### 5.7 Admin Dashboard
- Chỉ số tổng quan:
- Tổng users
- Tổng jobs
- Tổng posts
- Tổng applications
- Biểu đồ cơ bản:
- Job theo category
- Application count theo thời gian

### 5.8 UI/UX
- Responsive (desktop + mobile)
- Clean layout
- Pagination
- Toast notification

## 6. Tính Năng Nên Làm (Sau MVP)

### 6.1 Rich Text Editor
- Tích hợp TinyMCE hoặc Quill cho:
- Job description
- Community posts

### 6.2 Nested Comment
- Hỗ trợ reply 1 cấp (Comment -> Reply).

### 6.3 User Profile Page
- Bài viết đã đăng
- Job đã apply
- Job đã save

### 6.4 Slug SEO
- Ví dụ:
- `jobs/backend-developer-hcm`
- `posts/how-to-pass-interview`

### 6.5 Image Upload
- Avatar
- Company logo
- Post thumbnail

### 6.6 Moderation Workflow
- Trạng thái bài viết:
- `Draft`
- `Pending`
- `Published`
- `Hidden`

### 6.7 Soft Delete
- Đánh dấu `IsDeleted`
- Hỗ trợ khôi phục dữ liệu

## 7. Tính Năng Nâng Cao (Khi MVP Ổn Định)

### 7.1 Notification System
- Ví dụ:
- Có người comment
- Job được duyệt
- Trạng thái apply thay đổi

### 7.2 Realtime
- SignalR cho:
- Live notification
- Live comment

### 7.3 Recommendation
- Related jobs
- Related posts

### 7.4 Full Text Search
- Tối ưu tìm kiếm nội dung lớn.

### 7.5 Analytics Nâng Cao
- Top company
- Active users
- Trending tags

### 7.6 Dark Mode
- Chế độ giao diện tối.

### 7.7 Follow Company
- Theo dõi doanh nghiệp yêu thích.

### 7.8 Reporting System
- Báo cáo bài viết vi phạm.

## 8. Các Tính Năng Loại Bỏ (Để Giữ Đúng Scope)

### 8.1 Realtime Chat
- Tốn thời gian triển khai
- Giá trị chưa cao trong phạm vi môn học

### 8.2 Video Upload
- Tăng chi phí lưu trữ và vận hành

### 8.3 AI Features
- AI matching
- AI CV parsing
- Không phù hợp phạm vi hiện tại

### 8.4 Microservices
- Overengineering cho quy mô đề tài

### 8.5 Mobile App
- Chưa cần trong giai đoạn này

### 8.6 Multi-language
- Hoãn sang giai đoạn mở rộng

### 8.7 Complex Permission Matrix
- Giữ mức đơn giản với 3 role:
- `Admin`
- `Employer`
- `Candidate`

## 9. Ghi Chú Chuẩn Hóa
- Ưu tiên hoàn thành toàn bộ `MVP` trước khi mở rộng.
- Mọi tính năng sau MVP chỉ triển khai khi không ảnh hưởng tiến độ chính.
- Trạng thái workflow cần đồng bộ với tài liệu database.
