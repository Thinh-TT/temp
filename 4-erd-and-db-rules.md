# ERD And DB Rules

## 1. Mục tiêu tài liệu

- Chốt mô hình dữ liệu, quan hệ giữa các bảng và các ràng buộc ở mức database.
- Đồng bộ với `2-functional-requirements.md`, đặc biệt cho các use case từ `FR-001` đến `FR-022`.
- Làm nền cho thiết kế entity, migration, index, seed data và kiểm thử dữ liệu.

## 2. Nguồn tham chiếu

- Scope và tính năng: `1-ideas-and-scope.md`
- Mô tả bảng chi tiết: `3-database-description.md`
- Use case và business rule: `2-functional-requirements.md`

## 3. Danh sách bảng

| Nhóm           | Bảng                                         | Vai trò nghiệp vụ chính                  |
| -------------- | -------------------------------------------- | ---------------------------------------- |
| User & Auth    | `AspNetUsers`, `AspNetRoles`                 | Xác thực, phân quyền                     |
| User & Auth    | `CandidateProfiles`, `EmployerProfiles`      | Hồ sơ ứng viên, hồ sơ doanh nghiệp       |
| User & Auth    | `Skills`, `CandidateSkills`                  | Kỹ năng và liên kết kỹ năng ứng viên     |
| Recruitment    | `JobCategories`, `Jobs`                      | Danh mục job và tin tuyển dụng           |
| Recruitment    | `Applications`, `SavedJobs`                  | Đơn ứng tuyển và job đã lưu              |
| Community      | `PostCategories`, `Posts`                    | Danh mục bài viết và bài viết cộng đồng  |
| Community      | `Tags`, `PostTags`                           | Tag bài viết                             |
| Community      | `PostComments`, `PostVotes`, `PostBookmarks` | Tương tác bài viết                       |
| Admin & System | `Notifications`, `Reports`                   | Mở rộng cho thông báo và báo cáo vi phạm |

## 4. Quan hệ chính (ERD text)

### 4.1 User & profile

- `AspNetUsers (1) - (0..1) CandidateProfiles`
- `AspNetUsers (1) - (0..1) EmployerProfiles`
- `CandidateProfiles (N) - (N) Skills` qua `CandidateSkills`

### 4.2 Recruitment

- `EmployerProfiles (1) - (N) Jobs`
- `JobCategories (1) - (N) Jobs`
- `Jobs (1) - (N) Applications`
- `CandidateProfiles (1) - (N) Applications`
- `CandidateProfiles (1) - (N) SavedJobs`
- `Jobs (1) - (N) SavedJobs`

### 4.3 Community

- `AspNetUsers (1) - (N) Posts`
- `PostCategories (1) - (N) Posts`
- `Posts (N) - (N) Tags` qua `PostTags`
- `Posts (1) - (N) PostComments`
- `AspNetUsers (1) - (N) PostComments`
- `PostComments (1) - (0..N) PostComments` qua `ParentCommentId`
- `Posts (1) - (N) PostVotes`
- `AspNetUsers (1) - (N) PostVotes`
- `Posts (1) - (N) PostBookmarks`
- `AspNetUsers (1) - (N) PostBookmarks`

### 4.4 Admin & system

- `AspNetUsers (1) - (N) Notifications`
- `Posts (1) - (N) Reports`
- `AspNetUsers (1) - (N) Reports` qua `ReporterId`

## 5. Thực thể trọng tâm cần ưu tiên

### 5.1 Recruitment core

- `Jobs`
- `Applications`
- `SavedJobs`
- `JobCategories`

### 5.2 Community support

- `Posts`
- `PostComments`
- `PostVotes`
- `PostBookmarks`
- `Tags`
- `PostCategories`

### 5.3 User identity

- `AspNetUsers`
- `CandidateProfiles`
- `EmployerProfiles`

## 6. Khóa và ràng buộc

### 6.1 Primary key

- Quy ước: `<EntityName>Id` cho tất cả bảng nghiệp vụ, trừ các bảng Identity kế thừa chuẩn ASP.NET Identity.
- Tất cả primary key nên dùng clustered primary key mặc định.

### 6.2 Foreign key

- Dùng `ON DELETE RESTRICT` mặc định cho các bảng nghiệp vụ chính để tránh mất dữ liệu dây chuyền ngoài ý muốn.
- Có thể dùng `ON DELETE CASCADE` cho bảng liên kết thuần nếu parent bị hard delete trong môi trường dev/test:
  - `CandidateSkills`
  - `PostTags`
  - `PostVotes`
  - `PostBookmarks`
- Với `PostComments.ParentCommentId`, ưu tiên `ON DELETE RESTRICT` hoặc `NO ACTION`; xóa comment nên dùng soft delete thay vì hard delete.

### 6.3 One-to-one và uniqueness theo user

| Bảng                | Cột      | Ràng buộc |
| ------------------- | -------- | --------- |
| `CandidateProfiles` | `UserId` | Unique    |
| `EmployerProfiles`  | `UserId` | Unique    |

Ghi chú:

- Điều này bảo đảm một user không có nhiều hồ sơ candidate hoặc nhiều hồ sơ employer.
- Quy tắc “một tài khoản chỉ có một vai trò nghiệp vụ chính” hiện được chốt ở tầng ứng dụng; database hiện tại chưa có ràng buộc chéo trực tiếp giữa `CandidateProfiles` và `EmployerProfiles`.

### 6.4 Composite unique constraints bắt buộc

| Bảng              | Cột                      | Lý do                                             |
| ----------------- | ------------------------ | ------------------------------------------------- |
| `CandidateSkills` | `CandidateId`, `SkillId` | Không trùng kỹ năng trên cùng ứng viên            |
| `Applications`    | `JobId`, `CandidateId`   | Mỗi ứng viên chỉ apply một lần cho một job        |
| `SavedJobs`       | `CandidateId`, `JobId`   | Mỗi ứng viên chỉ lưu một lần cho một job          |
| `PostTags`        | `PostId`, `TagId`        | Không trùng tag trên cùng bài viết                |
| `PostVotes`       | `PostId`, `UserId`       | Mỗi user chỉ upvote tối đa một lần cho một post   |
| `PostBookmarks`   | `PostId`, `UserId`       | Mỗi user chỉ bookmark tối đa một lần cho một post |

### 6.5 Unique constraints đơn

| Bảng             | Cột        | Lý do                                              |
| ---------------- | ---------- | -------------------------------------------------- |
| `AspNetUsers`    | `UserName` | Tránh trùng tên đăng nhập                          |
| `AspNetUsers`    | `Email`    | Tránh trùng email đăng nhập                        |
| `Skills`         | `Name`     | Tránh trùng kỹ năng                                |
| `JobCategories`  | `Slug`     | Route SEO duy nhất                                 |
| `PostCategories` | `Slug`     | Route SEO duy nhất                                 |
| `Tags`           | `Slug`     | Route SEO duy nhất                                 |
| `Jobs`           | `Slug`     | Route SEO duy nhất cho job khi bật tính năng slug  |
| `Posts`          | `Slug`     | Route SEO duy nhất cho post khi bật tính năng slug |

Ghi chú:

- Với `Jobs.Slug` và `Posts.Slug`, nên dùng unique index có điều kiện `WHERE Slug IS NOT NULL` để hỗ trợ giai đoạn MVP chưa bật slug.
- Với `Name` và `Slug`, nên chuẩn hóa lower-case/trim tại tầng ứng dụng trước khi lưu để tránh trùng logic nhưng khác kiểu chữ.

### 6.6 Check constraints đề xuất

| Bảng                | Check constraint                              | Mục đích                                |
| ------------------- | --------------------------------------------- | --------------------------------------- |
| `Jobs`              | `SalaryMin >= 0`                              | Không cho lương âm                      |
| `Jobs`              | `SalaryMax IS NULL OR SalaryMax >= SalaryMin` | Tránh khoảng lương sai                  |
| `Jobs`              | `Deadline > CreatedAt`                        | Không cho hạn nộp nhỏ hơn thời điểm tạo |
| `CandidateProfiles` | `ExperienceYears >= 0`                        | Không cho số năm kinh nghiệm âm         |
| `Applications`      | `Status IN (0,1,2,3,4)`                       | Giới hạn enum workflow apply            |
| `Posts`             | `Status IN (0,1,2,3)`                         | Giới hạn enum workflow post             |
| `Jobs`              | `Status IN (0,1,2)`                           | Giới hạn enum workflow job hiện tại     |
| `PostComments`      | `ParentCommentId <> PostCommentId`            | Chặn self-reference                     |

Ghi chú:

- Rule “reply chỉ 1 cấp” của `PostComments.ParentCommentId` khó enforce sạch hoàn toàn bằng check constraint; nên validate ở tầng ứng dụng.
- Rule “job sửa nội dung quan trọng phải quay lại `Pending`” cũng là business rule ở tầng ứng dụng.

## 7. Enum và mã trạng thái chuẩn hóa

### 7.1 `Jobs.Status`

Đồng bộ theo functional requirements và workflow hiện tại:

- `0 = Pending`
- `1 = Approved`
- `2 = Hidden`

Ghi chú:

- Nếu sau này cần `Rejected`, phải cập nhật đồng bộ ở database description, workflow và API spec.

### 7.2 `Applications.Status`

- `0 = Pending`
- `1 = Reviewed`
- `2 = Interview`
- `3 = Rejected`
- `4 = Accepted`

### 7.3 `Posts.Status`

- `0 = Draft`
- `1 = Pending`
- `2 = Published`
- `3 = Hidden`

### 7.4 `Jobs.EmploymentType`

Giá trị enum nên được khai báo rõ ở tầng code, ví dụ:

- `0 = FullTime`
- `1 = PartTime`

Ghi chú:

- Nếu cần thêm `Remote`, `Contract`, `Internship`, phải chốt lại ở tài liệu scope và API trước khi mở rộng schema hoặc enum.

## 8. Quy tắc dữ liệu theo bảng

### 8.1 `AspNetUsers`

- Bắt buộc lưu `CreatedAt`.
- `LastLoginAt` nullable nhưng phải được cập nhật sau đăng nhập thành công.
- `IsActive = false` phải chặn luồng đăng nhập ở tầng ứng dụng.

### 8.2 `CandidateProfiles`

- `UserId` là unique FK tới `AspNetUsers`.
- `CVUrl` phải trỏ tới file PDF hợp lệ theo rule nghiệp vụ.
- `AvatarUrl` chỉ lưu URL/path, không lưu binary trực tiếp trong bảng.

### 8.3 `EmployerProfiles`

- `UserId` là unique FK tới `AspNetUsers`.
- `CompanyName` là dữ liệu bắt buộc trước khi cho phép tạo `Jobs`.
- `LogoUrl` chỉ lưu URL/path.

### 8.4 `Jobs`

- Mỗi job phải thuộc đúng một employer và một category.
- Job public chỉ lấy từ bản ghi `Status = Approved` và `IsDeleted = 0`.
- Khi employer chỉnh sửa nội dung quan trọng, tầng ứng dụng phải reset `Status = Pending`.
- `ViewCount` mặc định `0`.
- `CreatedAt`, `UpdatedAt` bắt buộc.

### 8.5 `Applications`

- Mỗi cặp `JobId + CandidateId` là duy nhất.
- `Status` mặc định `Pending`.
- `AppliedAt` bắt buộc.
- `ReviewedAt` có thể null cho đến khi employer mở/đổi trạng thái.
- `CVUrl` lưu snapshot tại thời điểm apply, không phụ thuộc hoàn toàn vào `CandidateProfiles.CVUrl`.

### 8.6 `SavedJobs`

- Mỗi cặp `CandidateId + JobId` là duy nhất.
- Không cần soft delete; thao tác bỏ lưu có thể hard delete bản ghi.

### 8.7 `Posts`

- `Content` có thể lưu HTML nhưng phải được sanitize trước khi insert/update.
- `UpvoteCount`, `CommentCount`, `BookmarkCount`, `ViewCount` là các trường cache số liệu, mặc định `0`.
- Feed công khai chỉ lấy `Status = Published` và `IsDeleted = 0`.
- `Slug` có thể null ở giai đoạn chưa bật tính năng slug.

### 8.8 `PostComments`

- Hỗ trợ soft delete bằng `IsDeleted`.
- `ParentCommentId` nullable; null nghĩa là comment gốc.
- Reply chỉ hỗ trợ 1 cấp, enforce ở tầng ứng dụng.

### 8.9 `PostVotes` và `PostBookmarks`

- Mỗi user tối đa một bản ghi trên mỗi post.
- Không cần soft delete; có thể hard delete khi người dùng bỏ upvote hoặc bỏ bookmark.

### 8.10 `Notifications`

- Bảng này là future-ready cho giai đoạn sau MVP.
- Chưa bắt buộc triển khai ngay, nhưng schema có thể giữ sẵn nếu nhóm muốn chuẩn bị mở rộng.

### 8.11 `Reports`

- Bảng này thuộc nhóm mở rộng.
- Nếu chưa làm reporting workflow, có thể giữ schema ở mức dự phòng và chưa expose API.

## 9. Index đề xuất

### 9.1 Index cho luồng đăng nhập và hồ sơ

| Bảng                | Cột index            | Loại   | Mục đích                |
| ------------------- | -------------------- | ------ | ----------------------- |
| `AspNetUsers`       | `NormalizedUserName` | Unique | Đăng nhập theo username |
| `AspNetUsers`       | `NormalizedEmail`    | Unique | Kiểm tra trùng email    |
| `CandidateProfiles` | `UserId`             | Unique | Truy hồ sơ theo user    |
| `EmployerProfiles`  | `UserId`             | Unique | Truy hồ sơ theo user    |

### 9.2 Index cho recruitment

| Bảng           | Cột index                             | Loại            | Mục đích                                |
| -------------- | ------------------------------------- | --------------- | --------------------------------------- |
| `Jobs`         | `Status`, `IsDeleted`, `CreatedAt`    | Nonclustered    | Danh sách job công khai và chờ duyệt    |
| `Jobs`         | `JobCategoryId`, `Location`, `Status` | Nonclustered    | Filter category/location                |
| `Jobs`         | `EmployerId`, `CreatedAt`             | Nonclustered    | Danh sách job theo employer             |
| `Jobs`         | `Slug`                                | Unique filtered | Truy cập chi tiết bằng slug             |
| `Applications` | `JobId`, `CandidateId`                | Unique          | Chặn apply trùng                        |
| `Applications` | `JobId`, `Status`, `AppliedAt`        | Nonclustered    | Danh sách ứng viên theo job và workflow |
| `Applications` | `CandidateId`, `AppliedAt`            | Nonclustered    | Lịch sử apply của candidate             |
| `SavedJobs`    | `CandidateId`, `JobId`                | Unique          | Chặn lưu trùng                          |

### 9.3 Index cho community

| Bảng            | Cột index                                            | Loại            | Mục đích                    |
| --------------- | ---------------------------------------------------- | --------------- | --------------------------- |
| `Posts`         | `Status`, `IsDeleted`, `CreatedAt`                   | Nonclustered    | Feed newest                 |
| `Posts`         | `Status`, `IsDeleted`, `UpvoteCount`, `CommentCount` | Nonclustered    | Feed trending đơn giản      |
| `Posts`         | `UserId`, `CreatedAt`                                | Nonclustered    | Bài viết theo tác giả       |
| `Posts`         | `Slug`                                               | Unique filtered | Truy cập bài viết bằng slug |
| `PostTags`      | `PostId`, `TagId`                                    | Unique          | Chặn tag trùng              |
| `PostComments`  | `PostId`, `CreatedAt`                                | Nonclustered    | Load comment theo bài       |
| `PostComments`  | `ParentCommentId`                                    | Nonclustered    | Load reply                  |
| `PostVotes`     | `PostId`, `UserId`                                   | Unique          | Chặn upvote trùng           |
| `PostBookmarks` | `PostId`, `UserId`                                   | Unique          | Chặn bookmark trùng         |

### 9.4 Index cho admin

| Bảng            | Cột index                       | Loại         | Mục đích                                |
| --------------- | ------------------------------- | ------------ | --------------------------------------- |
| `Jobs`          | `Status`, `UpdatedAt`           | Nonclustered | Kiểm duyệt job                          |
| `Posts`         | `Status`, `UpdatedAt`           | Nonclustered | Kiểm duyệt post                         |
| `Reports`       | `PostId`, `Status`, `CreatedAt` | Nonclustered | Theo dõi báo cáo vi phạm khi triển khai |
| `Notifications` | `UserId`, `IsRead`, `CreatedAt` | Nonclustered | Danh sách thông báo của user            |

## 10. Soft delete và restore rules

### 10.1 Bảng áp dụng soft delete

- `Jobs`
- `Posts`
- `PostComments`

### 10.2 Nguyên tắc áp dụng

- Query mặc định của ứng dụng phải loại trừ bản ghi `IsDeleted = 1`, trừ các màn hình admin hoặc luồng restore.
- Restore chỉ áp dụng cho bảng có cột `IsDeleted`.
- Sau khi restore:
  - `Jobs` quay về bản ghi chưa xóa mềm nhưng vẫn phải tuân theo `Status` hiện tại.
  - `Posts` quay về bản ghi chưa xóa mềm nhưng chỉ công khai nếu `Status = Published`.

### 10.3 Những bảng không cần soft delete

- `SavedJobs`
- `PostVotes`
- `PostBookmarks`
- `CandidateSkills`
- `PostTags`

Lý do:

- Đây là bảng quan hệ/interaction đơn giản; thao tác bỏ lưu, bỏ vote, bỏ tag có thể hard delete an toàn.

## 11. Quy tắc thời gian và audit

- Tất cả cột thời gian chuẩn hóa theo hậu tố `At`.
- Dùng UTC cho `CreatedAt`, `UpdatedAt`, `AppliedAt`, `ReviewedAt`, `LastLoginAt`.
- `UpdatedAt` phải cập nhật mỗi lần chỉnh sửa bản ghi chính như `Jobs`, `Posts`.
- Với các thao tác kiểm duyệt quan trọng, nên có audit log ở tầng ứng dụng hoặc thêm bảng lịch sử riêng khi triển khai:
  - đổi `Jobs.Status`
  - đổi `Posts.Status`
  - restore dữ liệu

## 12. Quy tắc migration

- Mỗi thay đổi schema phải có migration riêng, tên migration phản ánh đúng mục đích nghiệp vụ.
- Không sửa migration đã chạy ở môi trường dùng chung hoặc production.
- Với thay đổi rủi ro cao như đổi enum, đổi unique index, đổi kiểu cột, cần có script rollback hoặc kế hoạch rollback rõ ràng.
- Nếu thêm filtered unique index cho `Slug`, cần kiểm tra dữ liệu cũ trước khi migration chạy.

## 13. Seed data

### 13.1 Bắt buộc

- Roles mặc định:
  - `Admin`
  - `Employer`
  - `Candidate`
- Job categories cơ bản, ví dụ:
  - `Backend`
  - `Frontend`
  - `Fullstack`
  - `DevOps`
  - `QA`
- Post categories cơ bản, ví dụ:
  - `Career`
  - `Interview`
  - `Tech Sharing`
  - `CV & Portfolio`

### 13.2 Khuyến nghị

- Seed một admin mặc định cho môi trường development/demo.
- Seed một số `Skills` phổ biến để tránh candidate phải tạo thủ công từ đầu.

## 14. Quy tắc triển khai ở DB vs tầng ứng dụng

### 14.1 Nên enforce ở database

- PK, FK, unique constraints
- Composite unique cho bảng liên kết
- Check constraint số học và enum cơ bản
- Default value cho count/status/thời gian tạo nếu phù hợp
- Index cho truy vấn chính

### 14.2 Nên enforce ở tầng ứng dụng

- Phân quyền theo role
- Chỉ employer sở hữu job mới được sửa job hoặc xem application
- Reset `Jobs.Status` về `Pending` khi sửa nội dung quan trọng
- Transition workflow hợp lệ cho `Applications.Status`, `Jobs.Status`, `Posts.Status`
- Reply comment tối đa 1 cấp
- Sanitize HTML rich text
- Kiểm tra file upload và lưu trữ an toàn
- Ẩn dữ liệu riêng tư trên user profile

## 15. Changelog

| Version | Ngày       | Người cập nhật / Tên Model | Nội dung                                                                                                                     |
| ------- | ---------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 0.1     | YYYY-MM-DD |                            | Khởi tạo khung                                                                                                               |
| 0.2     | 2026-05-13 | Codex                      | Đồng bộ ERD và DB rules với functional requirements, bổ sung enum, unique constraints, index, soft delete và migration rules |
