# Mô Tả Cơ Sở Dữ Liệu

Hệ thống được chia thành 5 nhóm chính:
- User & Authentication Domain
- Recruitment Domain (Core)
- Community Domain
- Admin & System Domain
- Báo cáo và vận hành

## I. User & Authentication Domain

### 1. `ApplicationUser` (`AspNetUsers`, kế thừa `IdentityUser`)
Lưu thông tin tài khoản đăng nhập hệ thống.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `Id` | `string` | Khóa chính |
| `UserName` | `nvarchar` | Tên đăng nhập |
| `Email` | `nvarchar` | Email đăng nhập |
| `PasswordHash` | `nvarchar` | Mật khẩu đã mã hóa |
| `PhoneNumber` | `nvarchar` | Số điện thoại |
| `EmailConfirmed` | `bit` | Trạng thái xác nhận email |
| `LockoutEnabled` | `bit` | Cho phép khóa tài khoản |
| `CreatedAt` | `datetime` | Ngày tạo tài khoản |
| `LastLoginAt` | `datetime` | Lần đăng nhập gần nhất |
| `IsActive` | `bit` | Trạng thái hoạt động |

### 2. `Roles` (`AspNetRoles`, dùng `IdentityRole`)
Lưu nhóm quyền người dùng.

Các role mặc định:
- `Admin`
- `Employer`
- `Candidate`

### 3. `CandidateProfiles`
Lưu hồ sơ ứng viên.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `CandidateId` | `int` | Khóa chính |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `FullName` | `nvarchar` | Họ tên |
| `AvatarUrl` | `nvarchar` | Ảnh đại diện |
| `DateOfBirth` | `date` | Ngày sinh |
| `Gender` | `tinyint` | Giới tính |
| `Address` | `nvarchar` | Địa chỉ |
| `Bio` | `nvarchar(max)` | Giới thiệu bản thân |
| `CVUrl` | `nvarchar` | Đường dẫn CV PDF |
| `ExperienceYears` | `int` | Số năm kinh nghiệm |
| `CreatedAt` | `datetime` | Ngày tạo hồ sơ |

### 4. `Skills`
Danh mục kỹ năng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `SkillId` | `int` | Khóa chính |
| `Name` | `nvarchar` | Tên kỹ năng (ví dụ: ASP.NET, React) |

### 5. `CandidateSkills`
Bảng liên kết nhiều-nhiều giữa ứng viên và kỹ năng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `CandidateId` | `int` | FK -> `CandidateProfiles.CandidateId` |
| `SkillId` | `int` | FK -> `Skills.SkillId` |

### 6. `EmployerProfiles`
Lưu thông tin doanh nghiệp tuyển dụng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `EmployerId` | `int` | Khóa chính |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `CompanyName` | `nvarchar` | Tên công ty |
| `LogoUrl` | `nvarchar` | Logo công ty |
| `Description` | `nvarchar(max)` | Mô tả công ty |
| `Website` | `nvarchar` | Website |
| `Address` | `nvarchar` | Địa chỉ |
| `CompanySize` | `nvarchar` | Quy mô công ty |
| `CreatedAt` | `datetime` | Ngày tạo hồ sơ công ty |

## II. Recruitment Domain (Core)

### 7. `JobCategories`
Danh mục ngành nghề/vị trí.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `JobCategoryId` | `int` | Khóa chính |
| `Name` | `nvarchar` | Tên danh mục (ví dụ: Backend, DevOps) |
| `Slug` | `nvarchar` | Chuỗi thân thiện URL |

### 8. `Jobs`
Lưu tin tuyển dụng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `JobId` | `int` | Khóa chính |
| `EmployerId` | `int` | FK -> `EmployerProfiles.EmployerId` |
| `JobCategoryId` | `int` | FK -> `JobCategories.JobCategoryId` |
| `Title` | `nvarchar` | Tiêu đề tin tuyển dụng |
| `Slug` | `nvarchar` | Chuỗi SEO URL |
| `Description` | `nvarchar(max)` | Mô tả công việc |
| `Requirement` | `nvarchar(max)` | Yêu cầu tuyển dụng |
| `Benefits` | `nvarchar(max)` | Quyền lợi |
| `SalaryMin` | `decimal` | Mức lương tối thiểu |
| `SalaryMax` | `decimal` | Mức lương tối đa |
| `Location` | `nvarchar` | Địa điểm làm việc |
| `ExperienceLevel` | `nvarchar` | Cấp độ kinh nghiệm (Fresher/Junior/Senior) |
| `EmploymentType` | `tinyint` | Loại hình làm việc (Fulltime/Parttime) |
| `Deadline` | `datetime` | Hạn nộp hồ sơ |
| `Status` | `tinyint` | Trạng thái duyệt bài đăng |
| `ViewCount` | `int` | Số lượt xem |
| `CreatedAt` | `datetime` | Ngày tạo |
| `UpdatedAt` | `datetime` | Ngày cập nhật |
| `IsDeleted` | `bit` | Soft delete |

### 9. `Applications`
Lưu đơn ứng tuyển của ứng viên cho từng công việc.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `ApplicationId` | `int` | Khóa chính |
| `JobId` | `int` | FK -> `Jobs.JobId` |
| `CandidateId` | `int` | FK -> `CandidateProfiles.CandidateId` |
| `CVUrl` | `nvarchar` | CV tại thời điểm ứng tuyển |
| `CoverLetter` | `nvarchar(max)` | Thư giới thiệu |
| `Status` | `tinyint` | Trạng thái quy trình ứng tuyển |
| `AppliedAt` | `datetime` | Thời điểm nộp đơn |
| `ReviewedAt` | `datetime` | Thời điểm nhà tuyển dụng xem |
| `Note` | `nvarchar(max)` | Ghi chú từ nhà tuyển dụng |

Enum trạng thái `Applications.Status`:
- `0 = Pending`
- `1 = Reviewed`
- `2 = Interview`
- `3 = Rejected`
- `4 = Accepted`

### 10. `SavedJobs`
Danh sách công việc đã lưu của ứng viên.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `SavedJobId` | `int` | Khóa chính |
| `CandidateId` | `int` | FK -> `CandidateProfiles.CandidateId` |
| `JobId` | `int` | FK -> `Jobs.JobId` |
| `CreatedAt` | `datetime` | Thời điểm lưu |

## III. Community Domain

### 11. `PostCategories`
Danh mục bài viết cộng đồng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostCategoryId` | `int` | Khóa chính |
| `Name` | `nvarchar` | Tên danh mục |
| `Slug` | `nvarchar` | Chuỗi SEO URL |

### 12. `Posts`
Lưu nội dung bài viết cộng đồng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostId` | `int` | Khóa chính |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `PostCategoryId` | `int` | FK -> `PostCategories.PostCategoryId` |
| `Title` | `nvarchar` | Tiêu đề bài viết |
| `Slug` | `nvarchar` | Chuỗi SEO URL |
| `ThumbnailUrl` | `nvarchar` | Ảnh đại diện bài viết |
| `Content` | `nvarchar(max)` | Nội dung HTML |
| `Summary` | `nvarchar(500)` | Tóm tắt ngắn |
| `UpvoteCount` | `int` | Số lượng upvote (cache) |
| `CommentCount` | `int` | Số lượng bình luận (cache) |
| `BookmarkCount` | `int` | Số lượng bookmark (cache) |
| `ViewCount` | `int` | Số lượt xem |
| `Status` | `tinyint` | Trạng thái bài viết |
| `CreatedAt` | `datetime` | Ngày tạo |
| `UpdatedAt` | `datetime` | Ngày cập nhật |
| `IsDeleted` | `bit` | Soft delete |

Enum trạng thái `Posts.Status`:
- `0 = Draft`
- `1 = Pending`
- `2 = Published`
- `3 = Hidden`

### 13. `Tags`
Thẻ gắn cho bài viết cộng đồng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `TagId` | `int` | Khóa chính |
| `Name` | `nvarchar` | Tên tag |
| `Slug` | `nvarchar` | Chuỗi SEO URL |

### 14. `PostTags`
Bảng liên kết nhiều-nhiều giữa bài viết và tag.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostTagId` | `int` | Khóa chính |
| `PostId` | `int` | FK -> `Posts.PostId` |
| `TagId` | `int` | FK -> `Tags.TagId` |

### 15. `PostComments`
Bình luận của người dùng trên bài viết.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostCommentId` | `int` | Khóa chính |
| `PostId` | `int` | FK -> `Posts.PostId` |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `ParentCommentId` | `int` (nullable) | Bình luận cha (hỗ trợ reply 1 cấp) |
| `Content` | `nvarchar(max)` | Nội dung bình luận |
| `CreatedAt` | `datetime` | Ngày tạo |
| `IsDeleted` | `bit` | Soft delete |

### 16. `PostVotes`
Lượt upvote bài viết.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostVoteId` | `int` | Khóa chính |
| `PostId` | `int` | FK -> `Posts.PostId` |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `CreatedAt` | `datetime` | Thời điểm upvote |

### 17. `PostBookmarks`
Lưu bài viết cộng đồng.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `PostBookmarkId` | `int` | Khóa chính |
| `PostId` | `int` | FK -> `Posts.PostId` |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `CreatedAt` | `datetime` | Thời điểm lưu |

## IV. Admin & System Domain

### 18. `Notifications`
Thông báo hệ thống (có thể kết hợp SignalR cho realtime).

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `NotificationId` | `int` | Khóa chính |
| `UserId` | `string` | FK -> `AspNetUsers.Id` |
| `Type` | `tinyint` | Loại thông báo |
| `Title` | `nvarchar` | Tiêu đề thông báo |
| `Content` | `nvarchar` | Nội dung thông báo |
| `IsRead` | `bit` | Trạng thái đã đọc/chưa đọc |
| `CreatedAt` | `datetime` | Thời điểm tạo |

### 19. `Reports`
Báo cáo vi phạm nội dung bài viết.

| Cột | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `ReportId` | `int` | Khóa chính |
| `PostId` | `int` | FK -> `Posts.PostId` |
| `ReporterId` | `string` | FK -> `AspNetUsers.Id` |
| `Reason` | `nvarchar` | Lý do báo cáo |
| `Status` | `tinyint` | Trạng thái xử lý |
| `CreatedAt` | `datetime` | Thời điểm báo cáo |

## Ghi Chú Chuẩn Hóa

- Thống nhất tên cột thời gian theo hậu tố `At` (`CreatedAt`, `UpdatedAt`, ...).
- Dùng `IsDeleted` cho soft delete ở các bảng nội dung chính.
- Các trường `Status` nên đi kèm enum ở tầng code để tránh magic number.
