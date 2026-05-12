# Functional Requirements

## 1. Mục tiêu tài liệu
- Mô tả yêu cầu chức năng đủ chi tiết để đội BA, Dev, Tester và UI cùng bám vào triển khai.
- Bám theo phạm vi sản phẩm trong `1-ideas-and-scope.md` và cấu trúc dữ liệu trong `3-database-description.md`.
- Bao phủ toàn bộ tính năng `MVP` và `Sau MVP` trong mục 5 và mục 6 của tài liệu ý tưởng.

## 2. Phạm vi
### 2.1 In scope - MVP (`Must`)
- Đăng ký, đăng nhập, đăng xuất và phân quyền theo 3 role: `Candidate`, `Employer`, `Admin`.
- Quản lý hồ sơ ứng viên và hồ sơ doanh nghiệp.
- Quản lý tin tuyển dụng: tạo, sửa, xóa, duyệt, hiển thị công khai.
- Danh sách job, chi tiết job, tìm kiếm, lọc, ứng tuyển, lưu job.
- Quản lý quy trình ứng tuyển theo các trạng thái trong database.
- Module cộng đồng: bài viết, feed, comment, upvote, bookmark.
- Dashboard quản trị với chỉ số tổng quan và biểu đồ cơ bản.
- Responsive, phân trang, thông báo thao tác thành công/thất bại.

### 2.2 In scope - Sau MVP (`Should`)
- Rich text editor cho nội dung job và bài viết.
- Nested comment 1 cấp cho bài viết cộng đồng.
- Trang hồ sơ người dùng và hoạt động liên quan.
- Slug SEO cho job, bài viết, category, tag.
- Upload hình ảnh cho avatar, logo công ty, thumbnail bài viết.
- Moderation workflow cho bài viết cộng đồng.
- Soft delete và khôi phục dữ liệu.

### 2.3 Out of scope
- Realtime chat.
- Video upload.
- AI matching, AI CV parsing.
- Microservices.
- Mobile app native.
- Multi-language.
- Permission matrix phức tạp ngoài 3 role chính.
- Notification realtime, recommendation, analytics nâng cao và reporting system nâng cao trong giai đoạn hiện tại.

### 2.4 Giả định phân tích
- `Admin` không tự đăng ký công khai, tài khoản quản trị được seed hoặc tạo bởi hệ thống.
- Một tài khoản chỉ mang một vai trò nghiệp vụ chính tại một thời điểm: `Candidate` hoặc `Employer` hoặc `Admin`.
- Khi người dùng "xóa" job hoặc post, hệ thống ưu tiên xóa mềm ở tầng dữ liệu nếu bảng có `IsDeleted`.
- Trạng thái duyệt job trong phạm vi hiện tại gồm tối thiểu: `Pending`, `Approved`, `Hidden`.

## 3. Vai trò người dùng
- `Guest`: người chưa đăng nhập, chỉ xem nội dung công khai.
- `Candidate`: ứng viên tìm việc, quản lý hồ sơ cá nhân, ứng tuyển và tham gia cộng đồng.
- `Employer`: doanh nghiệp/nhà tuyển dụng đăng tin, quản lý ứng viên và tham gia cộng đồng.
- `Admin`: kiểm duyệt nội dung và theo dõi chỉ số hệ thống.

## 4. Danh sách use case
| ID | Use case | Vai trò | Mức ưu tiên | Ghi chú |
|---|---|---|---|---|
| FR-001 | Đăng ký tài khoản | `Guest` | Must | Chọn loại tài khoản `Candidate` hoặc `Employer` |
| FR-002 | Đăng nhập và đăng xuất | `Guest`, `Candidate`, `Employer`, `Admin` | Must | Duy trì phiên làm việc an toàn |
| FR-003 | Quản lý hồ sơ ứng viên | `Candidate` | Must | Gồm avatar, thông tin cá nhân, skills, CV PDF |
| FR-004 | Quản lý hồ sơ nhà tuyển dụng | `Employer` | Must | Gồm tên công ty, logo, mô tả, website, quy mô |
| FR-005 | Quản lý tin tuyển dụng | `Employer` | Must | CRUD job và theo dõi trạng thái duyệt |
| FR-006 | Xem và tìm kiếm danh sách job | `Guest`, `Candidate`, `Employer` | Must | Keyword, category, location, pagination |
| FR-007 | Xem chi tiết job | `Guest`, `Candidate`, `Employer` | Must | Chỉ job `Approved` được công khai |
| FR-008 | Ứng tuyển công việc | `Candidate` | Must | Dùng CV hiện tại hoặc CV đính kèm tại thời điểm apply |
| FR-009 | Lưu và bỏ lưu job | `Candidate` | Must | Job bookmark cá nhân |
| FR-010 | Quản lý danh sách ứng viên và trạng thái đơn | `Employer` | Must | Theo workflow `Pending -> Reviewed -> Interview -> Rejected/Accepted` |
| FR-011 | Tạo và quản lý bài viết cộng đồng | `Candidate`, `Employer` | Must | Tạo, sửa, xóa bài của chính mình; gắn category/tag |
| FR-012 | Xem feed cộng đồng | `Guest`, `Candidate`, `Employer` | Must | `Newest` và `Trending` đơn giản |
| FR-013 | Tương tác với bài viết cộng đồng | `Candidate`, `Employer`, `Admin` | Must | Comment, upvote, bookmark |
| FR-014 | Kiểm duyệt job | `Admin` | Must | Duyệt, ẩn, xóa mềm job vi phạm |
| FR-015 | Xem dashboard quản trị | `Admin` | Must | Tổng quan users, jobs, posts, applications |
| FR-016 | Soạn nội dung với rich text editor | `Employer`, `Candidate` | Should | Cho mô tả job và bài viết |
| FR-017 | Trả lời bình luận 1 cấp | `Candidate`, `Employer`, `Admin` | Should | Reply trực tiếp vào comment gốc |
| FR-018 | Xem trang hồ sơ người dùng | `Candidate`, `Employer`, `Guest` | Should | Bài viết đã đăng, job đã apply, job đã save |
| FR-019 | Tạo và sử dụng slug SEO | Hệ thống | Should | Route thân thiện cho job, post, category, tag |
| FR-020 | Upload ảnh nội dung | `Candidate`, `Employer` | Should | Avatar, logo công ty, thumbnail bài viết |
| FR-021 | Kiểm duyệt bài viết cộng đồng | `Candidate`, `Employer`, `Admin` | Should | Workflow `Draft`, `Pending`, `Published`, `Hidden` |
| FR-022 | Khôi phục dữ liệu đã xóa mềm | `Admin` | Should | Khôi phục job/post đã xóa mềm |

## 5. Use case chi tiết

### FR-001 - Đăng ký tài khoản
- Mô tả: Người dùng tạo tài khoản mới và chọn loại tài khoản `Candidate` hoặc `Employer`.
- Actor: `Guest`
- Preconditions:
  - Người dùng chưa đăng nhập.
  - Email và username chưa tồn tại.
- Trigger: Người dùng gửi form đăng ký.
- Main flow:
  1. Người dùng nhập username, email, mật khẩu và chọn loại tài khoản.
  2. Hệ thống kiểm tra dữ liệu bắt buộc và tính duy nhất.
  3. Hệ thống tạo bản ghi trong `AspNetUsers`.
  4. Hệ thống gán role tương ứng trong `AspNetRoles`.
  5. Hệ thống tạo hồ sơ rỗng trong `CandidateProfiles` hoặc `EmployerProfiles`.
  6. Hệ thống trả kết quả đăng ký thành công và điều hướng tới đăng nhập hoặc đăng nhập tự động.
- Alternative flows:
  1. Nếu email hoặc username đã tồn tại, hệ thống từ chối và hiển thị lỗi.
  2. Nếu role được chọn không hợp lệ, hệ thống không tạo tài khoản.
- Postconditions:
  - Tài khoản mới được tạo ở trạng thái hoạt động.
  - Hồ sơ nghiệp vụ tương ứng được khởi tạo.
- Business rules liên quan:
  - `BR-001`, `BR-002`, `BR-003`
- Acceptance criteria:
  1. Given email chưa tồn tại, When người dùng đăng ký với role `Candidate`, Then hệ thống tạo `AspNetUsers`, gán role `Candidate` và tạo `CandidateProfiles`.
  2. Given email đã tồn tại, When người dùng gửi form đăng ký, Then hệ thống trả lỗi `409` và không tạo thêm bản ghi nào.

### FR-002 - Đăng nhập và đăng xuất
- Mô tả: Người dùng xác thực vào hệ thống và kết thúc phiên làm việc khi cần.
- Actor: `Guest`, `Candidate`, `Employer`, `Admin`
- Preconditions:
  - Tài khoản tồn tại và đang hoạt động.
- Trigger:
  - Đăng nhập: người dùng gửi form login.
  - Đăng xuất: người dùng chọn chức năng logout.
- Main flow:
  1. Người dùng nhập email/username và mật khẩu.
  2. Hệ thống xác thực thông tin đăng nhập.
  3. Hệ thống cập nhật `LastLoginAt`.
  4. Hệ thống tạo token hoặc session và trả thông tin role.
  5. Khi logout, hệ thống hủy token hoặc session hiện tại.
- Alternative flows:
  1. Nếu thông tin đăng nhập sai, hệ thống trả lỗi `401`.
  2. Nếu tài khoản bị khóa hoặc không hoạt động, hệ thống từ chối đăng nhập.
- Postconditions:
  - Người dùng có hoặc không còn phiên đăng nhập hợp lệ.
- Business rules liên quan:
  - `BR-004`, `BR-005`
- Acceptance criteria:
  1. Given tài khoản hợp lệ, When người dùng đăng nhập đúng mật khẩu, Then hệ thống cấp phiên đăng nhập và trả role hiện tại.
  2. Given người dùng đã đăng nhập, When người dùng bấm đăng xuất, Then phiên hiện tại không còn dùng được cho request tiếp theo.

### FR-003 - Quản lý hồ sơ ứng viên
- Mô tả: Ứng viên cập nhật hồ sơ cá nhân phục vụ tìm việc và ứng tuyển.
- Actor: `Candidate`
- Preconditions:
  - Người dùng đã đăng nhập với role `Candidate`.
  - Hồ sơ `CandidateProfiles` đã tồn tại.
- Trigger: Người dùng mở trang hồ sơ và lưu thay đổi.
- Main flow:
  1. Hệ thống hiển thị thông tin hồ sơ hiện tại.
  2. Người dùng cập nhật họ tên, ngày sinh, giới tính, địa chỉ, bio, kinh nghiệm.
  3. Người dùng chọn avatar, danh sách kỹ năng và CV PDF nếu cần.
  4. Hệ thống kiểm tra định dạng dữ liệu và tệp tải lên.
  5. Hệ thống cập nhật `CandidateProfiles` và `CandidateSkills`.
  6. Hệ thống trả thông báo lưu thành công.
- Alternative flows:
  1. Nếu tệp CV không phải PDF hoặc vượt giới hạn, hệ thống từ chối tải lên.
  2. Nếu kỹ năng được chọn không tồn tại, hệ thống yêu cầu chọn lại.
- Postconditions:
  - Hồ sơ ứng viên được cập nhật.
  - CV và avatar mới khả dụng cho các chức năng liên quan.
- Business rules liên quan:
  - `BR-006`, `BR-007`, `BR-008`, `BR-018`
- Acceptance criteria:
  1. Given ứng viên đã đăng nhập, When cập nhật hồ sơ với dữ liệu hợp lệ, Then hệ thống lưu thông tin mới và danh sách skills tương ứng.
  2. Given ứng viên tải lên tệp không hợp lệ, When bấm lưu, Then hệ thống không cập nhật hồ sơ và hiển thị lỗi phù hợp.

### FR-004 - Quản lý hồ sơ nhà tuyển dụng
- Mô tả: Nhà tuyển dụng cập nhật thông tin công ty để sử dụng trong tin tuyển dụng và hồ sơ công khai.
- Actor: `Employer`
- Preconditions:
  - Người dùng đã đăng nhập với role `Employer`.
  - Hồ sơ `EmployerProfiles` đã tồn tại.
- Trigger: Người dùng lưu thay đổi tại trang hồ sơ doanh nghiệp.
- Main flow:
  1. Hệ thống hiển thị thông tin công ty hiện tại.
  2. Người dùng cập nhật tên công ty, mô tả, website, địa chỉ, quy mô và logo.
  3. Hệ thống kiểm tra dữ liệu và tệp logo nếu có.
  4. Hệ thống lưu thay đổi vào `EmployerProfiles`.
  5. Hệ thống dùng thông tin mới cho các tin tuyển dụng hiển thị sau đó.
- Alternative flows:
  1. Nếu thiếu `CompanyName`, hệ thống không cho lưu.
  2. Nếu logo tải lên không hợp lệ, hệ thống từ chối riêng phần logo.
- Postconditions:
  - Hồ sơ nhà tuyển dụng được cập nhật.
- Business rules liên quan:
  - `BR-009`, `BR-018`
- Acceptance criteria:
  1. Given employer hợp lệ, When cập nhật đầy đủ thông tin công ty, Then hệ thống lưu hồ sơ mới và phản ánh trên trang công khai.
  2. Given employer chưa nhập tên công ty, When bấm lưu, Then hệ thống báo lỗi validation.

### FR-005 - Quản lý tin tuyển dụng
- Mô tả: Nhà tuyển dụng tạo, sửa, xem danh sách và xóa job của chính mình.
- Actor: `Employer`
- Preconditions:
  - Người dùng đã đăng nhập với role `Employer`.
  - Hồ sơ doanh nghiệp đã được hoàn thiện tối thiểu.
- Trigger:
  - Người dùng tạo mới, chỉnh sửa hoặc xóa một job.
- Main flow:
  1. Người dùng mở danh sách job của doanh nghiệp.
  2. Người dùng tạo mới hoặc chọn một job để chỉnh sửa.
  3. Người dùng nhập tiêu đề, category, mô tả, yêu cầu, quyền lợi, lương, location, experience level, employment type, deadline.
  4. Hệ thống kiểm tra dữ liệu hợp lệ và gán `Status = Pending` cho job mới hoặc job đã thay đổi cần duyệt lại.
  5. Hệ thống lưu `Jobs`, cập nhật `UpdatedAt` và tạo `Slug` nếu tính năng đã bật.
  6. Khi xóa, hệ thống ẩn job khỏi giao diện người dùng và đánh dấu xóa mềm nếu áp dụng.
- Alternative flows:
  1. Nếu deadline nhỏ hơn thời điểm hiện tại, hệ thống không cho lưu.
  2. Nếu employer cố sửa job không thuộc sở hữu của mình, hệ thống trả `403`.
- Postconditions:
  - Job được tạo, cập nhật hoặc bị xóa mềm.
  - Job mới/chỉnh sửa chờ admin duyệt trước khi công khai.
- Business rules liên quan:
  - `BR-010`, `BR-011`, `BR-012`, `BR-013`, `BR-014`, `BR-019`
- Acceptance criteria:
  1. Given employer hợp lệ, When tạo job với đầy đủ thông tin, Then hệ thống lưu job ở trạng thái `Pending`.
  2. Given employer sở hữu một job, When xóa job, Then job không còn hiển thị công khai và dữ liệu được giữ để phục vụ khôi phục nếu bật soft delete.

### FR-006 - Xem và tìm kiếm danh sách job
- Mô tả: Người dùng duyệt danh sách job công khai và áp dụng bộ lọc tìm kiếm.
- Actor: `Guest`, `Candidate`, `Employer`
- Preconditions:
  - Hệ thống có job ở trạng thái `Approved` và chưa bị xóa mềm.
- Trigger: Người dùng mở trang danh sách job hoặc thay đổi bộ lọc.
- Main flow:
  1. Hệ thống tải danh sách job công khai theo phân trang.
  2. Người dùng nhập keyword hoặc chọn category, location.
  3. Hệ thống áp dụng điều kiện tìm kiếm và sắp xếp.
  4. Hệ thống trả danh sách phù hợp cùng thông tin tổng số bản ghi.
- Alternative flows:
  1. Nếu không có kết quả, hệ thống hiển thị empty state.
  2. Nếu tham số phân trang không hợp lệ, hệ thống dùng giá trị mặc định an toàn.
- Postconditions:
  - Người dùng nhìn thấy danh sách job phù hợp với bộ lọc.
- Business rules liên quan:
  - `BR-012`, `BR-015`, `BR-020`
- Acceptance criteria:
  1. Given có nhiều job đã duyệt, When người dùng lọc theo category và location, Then hệ thống chỉ trả job phù hợp và có phân trang.
  2. Given job chưa được duyệt, When guest mở danh sách job, Then job đó không xuất hiện.

### FR-007 - Xem chi tiết job
- Mô tả: Người dùng xem đầy đủ nội dung một job công khai.
- Actor: `Guest`, `Candidate`, `Employer`
- Preconditions:
  - Job tồn tại, ở trạng thái công khai và chưa bị xóa mềm.
- Trigger: Người dùng chọn một job từ danh sách hoặc mở bằng URL slug.
- Main flow:
  1. Hệ thống nhận `jobId` hoặc `slug`.
  2. Hệ thống tải thông tin job, công ty, category và metadata liên quan.
  3. Hệ thống tăng `ViewCount`.
  4. Hệ thống hiển thị mô tả, yêu cầu, quyền lợi, lương, deadline và thông tin công ty.
- Alternative flows:
  1. Nếu job không tồn tại hoặc đã bị ẩn, hệ thống trả `404`.
  2. Nếu truy cập bằng slug cũ, hệ thống điều hướng về slug hiện tại nếu có hỗ trợ.
- Postconditions:
  - Lượt xem job được ghi nhận.
- Business rules liên quan:
  - `BR-012`, `BR-016`
- Acceptance criteria:
  1. Given một job `Approved`, When người dùng mở trang chi tiết, Then hệ thống hiển thị đầy đủ thông tin và tăng `ViewCount`.
  2. Given một job `Hidden` hoặc đã xóa mềm, When guest truy cập URL, Then hệ thống không hiển thị nội dung công khai.

### FR-008 - Ứng tuyển công việc
- Mô tả: Ứng viên nộp đơn ứng tuyển cho một job.
- Actor: `Candidate`
- Preconditions:
  - Người dùng đã đăng nhập với role `Candidate`.
  - Job ở trạng thái `Approved`, chưa hết hạn và chưa bị xóa mềm.
  - Ứng viên có hồ sơ và có CV hợp lệ.
- Trigger: Người dùng chọn chức năng apply trên trang job detail.
- Main flow:
  1. Hệ thống hiển thị form apply với CV hiện tại và ô nhập cover letter.
  2. Người dùng xác nhận CV dùng để ứng tuyển và nhập thư giới thiệu nếu muốn.
  3. Hệ thống kiểm tra trùng đơn theo `CandidateId` và `JobId`.
  4. Hệ thống tạo bản ghi trong `Applications` với `Status = Pending`.
  5. Hệ thống lưu CV snapshot tại thời điểm apply.
  6. Hệ thống thông báo nộp đơn thành công.
- Alternative flows:
  1. Nếu job đã hết hạn, hệ thống không cho apply.
  2. Nếu ứng viên đã apply job đó trước đó, hệ thống trả `409`.
  3. Nếu candidate chưa có CV, hệ thống yêu cầu cập nhật hồ sơ trước.
- Postconditions:
  - Đơn ứng tuyển mới được tạo.
- Business rules liên quan:
  - `BR-017`, `BR-021`, `BR-022`, `BR-023`
- Acceptance criteria:
  1. Given candidate có CV hợp lệ, When apply vào job còn hạn, Then hệ thống tạo `Applications` ở trạng thái `Pending`.
  2. Given candidate đã apply job trước đó, When apply lần nữa, Then hệ thống từ chối và không tạo đơn mới.

### FR-009 - Lưu và bỏ lưu job
- Mô tả: Ứng viên lưu job để xem lại sau hoặc bỏ lưu khi không còn nhu cầu.
- Actor: `Candidate`
- Preconditions:
  - Người dùng đã đăng nhập với role `Candidate`.
  - Job tồn tại và còn hiển thị công khai.
- Trigger:
  - Người dùng bấm `Save`.
  - Người dùng bấm `Unsave`.
- Main flow:
  1. Hệ thống xác định candidate hiện tại và job mục tiêu.
  2. Nếu chưa có bản ghi lưu, hệ thống tạo `SavedJobs`.
  3. Nếu đã có bản ghi và người dùng chọn bỏ lưu, hệ thống xóa bản ghi lưu.
  4. Hệ thống cập nhật trạng thái hiển thị trên UI.
- Alternative flows:
  1. Nếu job không còn công khai, hệ thống không cho lưu mới.
  2. Nếu người dùng chưa đăng nhập, hệ thống điều hướng tới đăng nhập.
- Postconditions:
  - Danh sách `SavedJobs` của ứng viên được cập nhật.
- Business rules liên quan:
  - `BR-024`
- Acceptance criteria:
  1. Given candidate chưa lưu job, When bấm `Save`, Then hệ thống tạo đúng một bản ghi trong `SavedJobs`.
  2. Given candidate đã lưu job, When bấm `Unsave`, Then bản ghi lưu bị xóa và UI phản ánh ngay trạng thái mới.

### FR-010 - Quản lý danh sách ứng viên và trạng thái đơn
- Mô tả: Nhà tuyển dụng xem ứng viên của từng job và cập nhật trạng thái đơn theo workflow hợp lệ.
- Actor: `Employer`
- Preconditions:
  - Người dùng đã đăng nhập với role `Employer`.
  - Job thuộc sở hữu employer và có ít nhất một application.
- Trigger: Employer mở màn hình quản lý application hoặc thực hiện cập nhật trạng thái.
- Main flow:
  1. Hệ thống hiển thị danh sách application theo từng job.
  2. Employer xem CV snapshot, cover letter và trạng thái hiện tại.
  3. Employer chọn trạng thái tiếp theo hợp lệ: `Reviewed`, `Interview`, `Rejected`, `Accepted`.
  4. Hệ thống kiểm tra quyền sở hữu job và transition hợp lệ.
  5. Hệ thống cập nhật `Applications.Status`, `ReviewedAt` và `Note` nếu có.
  6. Hệ thống trả thông báo cập nhật thành công.
- Alternative flows:
  1. Nếu employer cố cập nhật application của job không thuộc doanh nghiệp mình, hệ thống trả `403`.
  2. Nếu transition không hợp lệ, hệ thống từ chối và giữ nguyên trạng thái cũ.
- Postconditions:
  - Trạng thái application được cập nhật đúng workflow.
- Business rules liên quan:
  - `BR-021`, `BR-025`, `BR-026`
- Acceptance criteria:
  1. Given application đang ở `Pending`, When employer đổi sang `Reviewed`, Then hệ thống cập nhật thành công và ghi `ReviewedAt`.
  2. Given application đã ở `Accepted`, When employer cố chuyển về `Reviewed`, Then hệ thống từ chối do vi phạm workflow.

### FR-011 - Tạo và quản lý bài viết cộng đồng
- Mô tả: Người dùng đã đăng nhập tạo, sửa và xóa bài viết cộng đồng của chính mình.
- Actor: `Candidate`, `Employer`
- Preconditions:
  - Người dùng đã đăng nhập.
  - Có quyền tạo bài trong module cộng đồng.
- Trigger: Người dùng tạo mới, chỉnh sửa hoặc xóa bài viết.
- Main flow:
  1. Người dùng mở trình soạn thảo bài viết.
  2. Người dùng nhập tiêu đề, summary, content, category và tag.
  3. Nếu tính năng upload ảnh đã bật, người dùng có thể chọn thumbnail.
  4. Hệ thống kiểm tra dữ liệu bắt buộc và định dạng nội dung.
  5. Hệ thống lưu bài viết vào `Posts` và quan hệ `PostTags`.
  6. Khi chỉnh sửa hoặc xóa, hệ thống chỉ cho phép tác giả thao tác trên bài của mình.
- Alternative flows:
  1. Nếu thiếu tiêu đề hoặc nội dung, hệ thống từ chối lưu.
  2. Nếu người dùng sửa bài không phải của mình, hệ thống trả `403`.
- Postconditions:
  - Bài viết được tạo, cập nhật hoặc xóa mềm theo cấu hình.
- Business rules liên quan:
  - `BR-027`, `BR-028`, `BR-029`, `BR-030`
- Acceptance criteria:
  1. Given người dùng đăng nhập hợp lệ, When tạo bài với title, content, category hợp lệ, Then hệ thống lưu bài viết và gắn tag tương ứng.
  2. Given người dùng không phải tác giả, When sửa bài của người khác, Then hệ thống từ chối thao tác.

### FR-012 - Xem feed cộng đồng
- Mô tả: Người dùng xem danh sách bài viết cộng đồng theo `Newest` hoặc `Trending`.
- Actor: `Guest`, `Candidate`, `Employer`
- Preconditions:
  - Hệ thống có bài viết ở trạng thái công khai.
- Trigger: Người dùng mở trang feed hoặc đổi kiểu sắp xếp.
- Main flow:
  1. Hệ thống tải danh sách bài viết công khai, chưa bị xóa mềm.
  2. Người dùng chọn `Newest` hoặc `Trending`.
  3. Với `Trending`, hệ thống sắp xếp theo điểm đơn giản từ `UpvoteCount` và `CommentCount`.
  4. Hệ thống trả kết quả có phân trang.
- Alternative flows:
  1. Nếu không có bài viết, hệ thống hiển thị empty state.
  2. Nếu tham số sort không hợp lệ, hệ thống dùng `Newest`.
- Postconditions:
  - Người dùng xem được feed cộng đồng theo tiêu chí mong muốn.
- Business rules liên quan:
  - `BR-031`, `BR-032`
- Acceptance criteria:
  1. Given có nhiều bài `Published`, When guest chọn `Newest`, Then hệ thống hiển thị bài theo thứ tự tạo mới nhất.
  2. Given có dữ liệu comment và upvote, When người dùng chọn `Trending`, Then hệ thống dùng điểm xu hướng đơn giản để sắp xếp.

### FR-013 - Tương tác với bài viết cộng đồng
- Mô tả: Người dùng tương tác với bài viết thông qua comment, upvote, bookmark và reply 1 cấp nếu tính năng đã bật.
- Actor: `Candidate`, `Employer`, `Admin`
- Preconditions:
  - Người dùng đã đăng nhập.
  - Bài viết ở trạng thái `Published`.
- Trigger:
  - Người dùng gửi comment.
  - Người dùng bấm upvote hoặc bookmark.
  - Người dùng reply vào comment gốc.
- Main flow:
  1. Hệ thống nhận hành động tương tác từ người dùng.
  2. Với comment hoặc reply, hệ thống tạo `PostComments` và cập nhật `CommentCount`.
  3. Với upvote, hệ thống tạo hoặc hủy `PostVotes` theo trạng thái hiện tại và cập nhật `UpvoteCount`.
  4. Với bookmark, hệ thống tạo hoặc hủy `PostBookmarks` và cập nhật `BookmarkCount`.
  5. Hệ thống trả trạng thái tương tác mới cho UI.
- Alternative flows:
  1. Nếu bài viết không ở trạng thái `Published`, hệ thống không cho tương tác mới.
  2. Nếu reply vượt quá 1 cấp, hệ thống từ chối.
  3. Nếu comment đã bị xóa mềm, hệ thống không cho reply vào comment đó.
- Postconditions:
  - Dữ liệu tương tác và bộ đếm cache của bài viết được cập nhật.
- Business rules liên quan:
  - `BR-033`, `BR-034`, `BR-035`, `BR-036`
- Acceptance criteria:
  1. Given người dùng đã đăng nhập, When gửi comment hợp lệ, Then hệ thống tạo `PostComments` và tăng `CommentCount`.
  2. Given người dùng đã upvote một bài, When bấm upvote lần nữa, Then hệ thống toggle hoặc từ chối theo thiết kế đã chọn và dữ liệu luôn chỉ có tối đa một bản ghi active cho mỗi người dùng.

### FR-014 - Kiểm duyệt job
- Mô tả: Admin duyệt, ẩn hoặc xóa mềm tin tuyển dụng vi phạm.
- Actor: `Admin`
- Preconditions:
  - Người dùng đã đăng nhập với role `Admin`.
  - Job tồn tại.
- Trigger: Admin thao tác trên màn hình kiểm duyệt job.
- Main flow:
  1. Hệ thống hiển thị danh sách job chờ duyệt hoặc cần xử lý.
  2. Admin xem thông tin job và thông tin doanh nghiệp liên quan.
  3. Admin chọn hành động `Approve`, `Hide` hoặc `Delete`.
  4. Hệ thống cập nhật `Jobs.Status` hoặc `IsDeleted` tương ứng.
  5. Hệ thống ghi nhận thời điểm và người thao tác cho audit nếu có.
- Alternative flows:
  1. Nếu job đã bị xóa mềm, hệ thống không cho duyệt tiếp.
  2. Nếu admin chọn trạng thái không hợp lệ, hệ thống từ chối.
- Postconditions:
  - Job ở trạng thái kiểm duyệt mới.
- Business rules liên quan:
  - `BR-012`, `BR-013`, `BR-037`
- Acceptance criteria:
  1. Given job đang `Pending`, When admin duyệt, Then job chuyển sang `Approved` và xuất hiện công khai.
  2. Given job vi phạm, When admin chọn `Hide` hoặc `Delete`, Then job không còn hiển thị trên danh sách công khai.

### FR-015 - Xem dashboard quản trị
- Mô tả: Admin theo dõi tổng quan hệ thống qua các chỉ số và biểu đồ cơ bản.
- Actor: `Admin`
- Preconditions:
  - Người dùng đã đăng nhập với role `Admin`.
- Trigger: Admin mở trang dashboard.
- Main flow:
  1. Hệ thống tổng hợp số lượng users, jobs, posts, applications.
  2. Hệ thống tổng hợp số job theo category.
  3. Hệ thống tổng hợp application count theo thời gian.
  4. Hệ thống hiển thị dữ liệu dưới dạng card và chart.
- Alternative flows:
  1. Nếu chưa có dữ liệu, hệ thống hiển thị giá trị 0 và trạng thái trống.
  2. Nếu một nguồn dữ liệu tạm lỗi, hệ thống hiển thị thông báo lỗi cục bộ nhưng không làm hỏng toàn bộ dashboard.
- Postconditions:
  - Admin xem được snapshot tổng quan hệ thống.
- Business rules liên quan:
  - `BR-038`
- Acceptance criteria:
  1. Given admin đăng nhập hợp lệ, When mở dashboard, Then hệ thống hiển thị 4 chỉ số tổng và 2 biểu đồ cơ bản.
  2. Given chưa có application, When mở biểu đồ theo thời gian, Then hệ thống hiển thị biểu đồ rỗng hợp lệ thay vì lỗi.

### FR-016 - Soạn nội dung với rich text editor
- Mô tả: Người dùng soạn mô tả job và bài viết bằng editor hỗ trợ định dạng cơ bản.
- Actor: `Employer`, `Candidate`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Người dùng đã đăng nhập và có quyền tạo nội dung tương ứng.
- Trigger: Người dùng mở form tạo hoặc sửa job/post.
- Main flow:
  1. Hệ thống hiển thị trình soạn thảo rich text.
  2. Người dùng định dạng tiêu đề phụ, danh sách, đoạn văn, liên kết.
  3. Hệ thống lưu HTML đã được sanitize vào `Jobs.Description`, `Jobs.Requirement`, `Jobs.Benefits` hoặc `Posts.Content`.
- Alternative flows:
  1. Nếu nội dung chứa script hoặc HTML nguy hiểm, hệ thống loại bỏ trước khi lưu.
- Postconditions:
  - Nội dung định dạng được lưu và hiển thị an toàn.
- Business rules liên quan:
  - `BR-039`
- Acceptance criteria:
  1. Given employer tạo job, When nhập mô tả bằng rich text editor, Then hệ thống lưu được HTML hợp lệ và hiển thị đúng ở trang chi tiết.
  2. Given người dùng dán nội dung có script, When lưu, Then hệ thống sanitize và không render mã nguy hiểm.

### FR-017 - Trả lời bình luận 1 cấp
- Mô tả: Người dùng reply trực tiếp vào một bình luận gốc trong bài viết cộng đồng.
- Actor: `Candidate`, `Employer`, `Admin`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Người dùng đã đăng nhập.
  - Comment cha là comment cấp gốc.
- Trigger: Người dùng bấm `Reply` tại một comment.
- Main flow:
  1. Hệ thống hiển thị ô nhập phản hồi dưới comment gốc.
  2. Người dùng nhập nội dung và gửi.
  3. Hệ thống tạo `PostComments` mới với `ParentCommentId` trỏ về comment gốc.
  4. Hệ thống cập nhật `CommentCount`.
- Alternative flows:
  1. Nếu comment cha đã là reply, hệ thống từ chối tạo reply cấp 2.
  2. Nếu comment cha đã bị xóa mềm, hệ thống không cho reply.
- Postconditions:
  - Reply 1 cấp được tạo thành công nếu hợp lệ.
- Business rules liên quan:
  - `BR-034`
- Acceptance criteria:
  1. Given comment gốc tồn tại, When người dùng reply, Then hệ thống tạo comment con với `ParentCommentId` hợp lệ.
  2. Given comment cha đã là reply, When người dùng tiếp tục reply, Then hệ thống từ chối vì vượt quá số cấp hỗ trợ.

### FR-018 - Xem trang hồ sơ người dùng
- Mô tả: Người dùng xem trang hồ sơ công khai và hoạt động liên quan của mình hoặc của người khác theo quyền cho phép.
- Actor: `Candidate`, `Employer`, `Guest`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Người dùng mục tiêu tồn tại.
- Trigger: Người dùng mở trang profile.
- Main flow:
  1. Hệ thống hiển thị thông tin cơ bản của ứng viên hoặc doanh nghiệp.
  2. Nếu là chủ sở hữu hồ sơ candidate, hệ thống hiển thị thêm job đã apply và job đã save.
  3. Hệ thống hiển thị danh sách bài viết đã đăng của người dùng.
  4. Hệ thống áp dụng phân trang cho các danh sách dài.
- Alternative flows:
  1. Nếu guest truy cập hồ sơ candidate, hệ thống không hiển thị dữ liệu riêng tư như danh sách job đã apply và CV.
  2. Nếu hồ sơ không tồn tại, hệ thống trả `404`.
- Postconditions:
  - Người dùng xem được hồ sơ phù hợp với quyền truy cập.
- Business rules liên quan:
  - `BR-040`
- Acceptance criteria:
  1. Given candidate mở hồ sơ của chính mình, When tải trang profile, Then hệ thống hiển thị bài viết đã đăng, job đã apply và job đã save.
  2. Given guest mở hồ sơ candidate khác, When tải trang, Then hệ thống chỉ hiển thị phần công khai và không lộ CV hoặc lịch sử apply.

### FR-019 - Tạo và sử dụng slug SEO
- Mô tả: Hệ thống tạo slug thân thiện URL cho job, bài viết, category và tag.
- Actor: Hệ thống
- Preconditions:
  - Tính năng sau MVP đã được bật.
- Trigger: Bản ghi mới được tạo hoặc tiêu đề/tên thay đổi.
- Main flow:
  1. Hệ thống chuẩn hóa title hoặc name thành slug.
  2. Hệ thống kiểm tra tính duy nhất trong bảng tương ứng.
  3. Nếu trùng, hệ thống thêm hậu tố phân biệt.
  4. Hệ thống lưu slug vào cột `Slug`.
  5. Router dùng slug để render URL công khai.
- Alternative flows:
  1. Nếu title rỗng, hệ thống không tạo slug và trả lỗi validation cho thực thể liên quan.
- Postconditions:
  - Mỗi bản ghi có slug hợp lệ và duy nhất trong ngữ cảnh của nó.
- Business rules liên quan:
  - `BR-041`
- Acceptance criteria:
  1. Given job có title mới, When hệ thống lưu job, Then slug được tạo và có thể dùng trong URL chi tiết.
  2. Given hai bài viết có title giống nhau, When tạo slug, Then hệ thống vẫn đảm bảo hai slug là duy nhất.

### FR-020 - Upload ảnh nội dung
- Mô tả: Người dùng tải lên avatar, logo công ty và thumbnail bài viết.
- Actor: `Candidate`, `Employer`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Người dùng đã đăng nhập.
- Trigger: Người dùng chọn và tải lên một tệp ảnh.
- Main flow:
  1. Hệ thống kiểm tra định dạng và kích thước tệp.
  2. Hệ thống lưu tệp tại nơi lưu trữ đã cấu hình.
  3. Hệ thống cập nhật `AvatarUrl`, `LogoUrl` hoặc `ThumbnailUrl`.
  4. Hệ thống hiển thị ảnh mới cho người dùng.
- Alternative flows:
  1. Nếu tệp không phải định dạng ảnh hợp lệ, hệ thống từ chối.
  2. Nếu upload thất bại, hệ thống giữ nguyên URL cũ.
- Postconditions:
  - URL ảnh mới được lưu nếu upload thành công.
- Business rules liên quan:
  - `BR-018`, `BR-042`
- Acceptance criteria:
  1. Given candidate chọn avatar hợp lệ, When upload thành công, Then `AvatarUrl` được cập nhật.
  2. Given employer chọn tệp vượt giới hạn, When upload, Then hệ thống báo lỗi và không ghi đè logo hiện tại.

### FR-021 - Kiểm duyệt bài viết cộng đồng
- Mô tả: Bài viết cộng đồng đi qua workflow kiểm duyệt sau MVP trước khi công khai.
- Actor: `Candidate`, `Employer`, `Admin`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Bài viết tồn tại.
- Trigger:
  - Tác giả lưu nháp hoặc gửi duyệt.
  - Admin duyệt hoặc ẩn bài.
- Main flow:
  1. Tác giả tạo bài và lưu với `Status = Draft` hoặc gửi duyệt với `Status = Pending`.
  2. Admin xem danh sách bài chờ duyệt.
  3. Admin chọn `Publish` hoặc `Hide`.
  4. Hệ thống cập nhật `Posts.Status` tương ứng.
  5. Feed công khai chỉ hiển thị bài `Published`.
- Alternative flows:
  1. Nếu người dùng không phải tác giả, hệ thống không cho đổi trạng thái từ `Draft` sang `Pending`.
  2. Nếu admin thao tác trạng thái không hợp lệ, hệ thống từ chối.
- Postconditions:
  - Bài viết ở đúng trạng thái workflow.
- Business rules liên quan:
  - `BR-029`, `BR-031`, `BR-043`
- Acceptance criteria:
  1. Given tác giả lưu nháp, When mở lại bài viết, Then chỉ tác giả nhìn thấy bài ở trạng thái `Draft`.
  2. Given bài viết ở `Pending`, When admin duyệt, Then bài chuyển sang `Published` và xuất hiện trên feed công khai.

### FR-022 - Khôi phục dữ liệu đã xóa mềm
- Mô tả: Admin khôi phục job hoặc bài viết đã bị xóa mềm nhưng còn lưu trong database.
- Actor: `Admin`
- Preconditions:
  - Tính năng sau MVP đã được bật.
  - Bản ghi mục tiêu có `IsDeleted = true`.
- Trigger: Admin chọn thao tác restore trên màn hình quản lý dữ liệu đã xóa.
- Main flow:
  1. Hệ thống hiển thị danh sách job/post đã xóa mềm.
  2. Admin xem chi tiết bản ghi cần khôi phục.
  3. Admin chọn `Restore`.
  4. Hệ thống đặt lại `IsDeleted = false`.
  5. Hệ thống đưa bản ghi về trạng thái hiển thị phù hợp theo workflow hiện tại.
- Alternative flows:
  1. Nếu bản ghi không hỗ trợ soft delete, hệ thống không hiển thị trong danh sách khôi phục.
  2. Nếu admin cố restore bản ghi không tồn tại, hệ thống trả `404`.
- Postconditions:
  - Bản ghi được khôi phục khỏi trạng thái xóa mềm.
- Business rules liên quan:
  - `BR-044`
- Acceptance criteria:
  1. Given job đã bị xóa mềm, When admin restore, Then `IsDeleted` về `false` và job có thể tiếp tục được xử lý theo workflow.
  2. Given post không bị xóa mềm, When gọi restore, Then hệ thống từ chối thao tác không hợp lệ.

## 6. Business rules tổng hợp
| Rule ID | Mô tả | Áp dụng cho |
|---|---|---|
| `BR-001` | `Email` và `UserName` phải duy nhất trong hệ thống. | Đăng ký tài khoản |
| `BR-002` | `Admin` không đăng ký qua luồng public. | Đăng ký tài khoản |
| `BR-003` | Sau khi đăng ký phải tạo hồ sơ nghiệp vụ tương ứng với role. | Đăng ký tài khoản |
| `BR-004` | Chỉ tài khoản `IsActive = true` mới được đăng nhập. | Đăng nhập |
| `BR-005` | Hệ thống phải cập nhật `LastLoginAt` sau đăng nhập thành công. | Đăng nhập |
| `BR-006` | Chỉ `Candidate` mới được cập nhật `CandidateProfiles`. | Hồ sơ ứng viên |
| `BR-007` | `CandidateSkills` là quan hệ nhiều-nhiều; không được lưu trùng cùng một cặp `CandidateId` và `SkillId`. | Hồ sơ ứng viên |
| `BR-008` | `CVUrl` phải trỏ tới tệp PDF hợp lệ và còn truy cập được tại thời điểm apply. | Hồ sơ ứng viên, ứng tuyển |
| `BR-009` | Employer phải có `CompanyName` trước khi tạo job. | Hồ sơ employer, job |
| `BR-010` | Mỗi job phải thuộc đúng một `EmployerProfiles` và một `JobCategories`. | Job |
| `BR-011` | `Deadline` phải lớn hơn thời điểm lưu job. | Job |
| `BR-012` | Chỉ job `Approved` và chưa bị xóa mềm mới được hiển thị công khai. | Job list, job detail, kiểm duyệt job |
| `BR-013` | Job mới tạo hoặc job sửa nội dung quan trọng phải quay lại trạng thái `Pending`. | Job |
| `BR-014` | Employer chỉ được sửa/xóa job do mình sở hữu. | Job |
| `BR-015` | API danh sách job phải hỗ trợ phân trang và chỉ nhận bộ lọc `keyword`, `category`, `location` trong scope hiện tại. | Job search |
| `BR-016` | Mỗi lượt mở job detail hợp lệ sẽ tăng `ViewCount` thêm 1. | Job detail |
| `BR-017` | Chỉ `Candidate` mới được apply job. | Ứng tuyển |
| `BR-018` | Upload file phải kiểm tra loại tệp, kích thước và đường dẫn lưu trữ an toàn. | Upload CV, avatar, logo, thumbnail |
| `BR-019` | Xóa job trong nghiệp vụ được hiểu là ẩn khỏi người dùng cuối; ở tầng dữ liệu ưu tiên soft delete nếu có `IsDeleted`. | Job |
| `BR-020` | Bộ lọc job không được trả về bản ghi `Hidden`, `Pending` hoặc `IsDeleted = true`. | Job search |
| `BR-021` | Mỗi candidate chỉ được có tối đa một application cho mỗi job. | Ứng tuyển |
| `BR-022` | Không cho apply job khi đã quá `Deadline`. | Ứng tuyển |
| `BR-023` | `Applications.Status` phải khởi tạo là `Pending`. | Ứng tuyển |
| `BR-024` | Mỗi candidate chỉ được lưu tối đa một bản ghi `SavedJobs` cho mỗi job. | Save job |
| `BR-025` | Chỉ employer sở hữu job mới được xem application của job đó. | Quản lý application |
| `BR-026` | Workflow application chỉ cho phép: `Pending -> Reviewed -> Interview -> Accepted/Rejected`, không cho rollback từ `Accepted`. | Quản lý application |
| `BR-027` | Chỉ tác giả mới được sửa/xóa bài viết của mình, trừ admin trong vai trò kiểm duyệt. | Bài viết cộng đồng |
| `BR-028` | `Posts` phải thuộc một `PostCategories`; tag là tùy chọn nhưng nếu có phải tồn tại trong `Tags`. | Bài viết cộng đồng |
| `BR-029` | Trong MVP có thể publish trực tiếp; khi moderation workflow bật thì phải dùng `Draft/Pending/Published/Hidden`. | Bài viết cộng đồng |
| `BR-030` | Xóa bài viết trong nghiệp vụ ưu tiên xóa mềm bằng `IsDeleted` nếu đã bật soft delete. | Bài viết cộng đồng |
| `BR-031` | Feed cộng đồng chỉ hiển thị bài `Published` và chưa bị xóa mềm. | Feed cộng đồng |
| `BR-032` | `Trending` được tính tối thiểu từ `UpvoteCount + CommentCount`. | Feed cộng đồng |
| `BR-033` | Mỗi user chỉ có tối đa một upvote active cho mỗi post. | Upvote post |
| `BR-034` | `ParentCommentId` chỉ hỗ trợ reply 1 cấp. | Comment/reply |
| `BR-035` | Mỗi user chỉ có tối đa một bookmark active cho mỗi post. | Bookmark post |
| `BR-036` | Khi tạo/xóa comment, upvote hoặc bookmark, hệ thống phải đồng bộ lại các trường cache count liên quan trong `Posts`. | Tương tác bài viết |
| `BR-037` | Hành động kiểm duyệt job phải được ghi log hoặc lưu dấu vết audit khi hệ thống có hỗ trợ. | Kiểm duyệt job |
| `BR-038` | Dashboard quản trị dùng dữ liệu tổng hợp theo thời điểm truy vấn, không yêu cầu realtime. | Dashboard |
| `BR-039` | Nội dung HTML từ rich text editor phải được sanitize trước khi lưu và render. | Rich text editor |
| `BR-040` | Dữ liệu riêng tư như CV, danh sách job đã apply chỉ hiển thị cho chính chủ hoặc admin nếu có quyền. | User profile page |
| `BR-041` | `Slug` phải duy nhất trong từng bảng đích và được tái tạo khi title/name thay đổi nếu thiết kế cho phép. | SEO slug |
| `BR-042` | Ảnh upload phải là định dạng web an toàn như `jpg`, `jpeg`, `png`, `webp`. | Upload ảnh |
| `BR-043` | `Draft` chỉ tác giả nhìn thấy; `Pending` chờ admin; `Published` công khai; `Hidden` không công khai. | Moderation workflow post |
| `BR-044` | Chỉ admin mới được restore bản ghi đã xóa mềm. | Soft delete/restore |

## 7. Traceability
| Use case ID | API | DB table | UI screen | Test case |
|---|---|---|---|---|
| `FR-001` | `POST /api/v1/auth/register` | `AspNetUsers`, `AspNetRoles`, `CandidateProfiles`, `EmployerProfiles` | `SCR-005 Register` | `TC-FR-001` |
| `FR-002` | `POST /api/v1/auth/login`, `POST /api/v1/auth/logout` | `AspNetUsers` | `SCR-004 Login` | `TC-FR-002` |
| `FR-003` | `GET /api/v1/candidates/me`, `PUT /api/v1/candidates/me` | `CandidateProfiles`, `Skills`, `CandidateSkills` | `SCR-006 Candidate Profile` | `TC-FR-003` |
| `FR-004` | `GET /api/v1/employers/me`, `PUT /api/v1/employers/me` | `EmployerProfiles` | `SCR-007 Employer Profile` | `TC-FR-004` |
| `FR-005` | `GET /api/v1/employer/jobs`, `POST /api/v1/jobs`, `PUT /api/v1/jobs/{jobId}`, `DELETE /api/v1/jobs/{jobId}` | `Jobs`, `JobCategories` | `SCR-008 Employer Job List`, `SCR-009 Job Form` | `TC-FR-005` |
| `FR-006` | `GET /api/v1/jobs` | `Jobs`, `JobCategories`, `EmployerProfiles` | `SCR-002 Job List` | `TC-FR-006` |
| `FR-007` | `GET /api/v1/jobs/{slug}` | `Jobs`, `EmployerProfiles`, `JobCategories` | `SCR-003 Job Detail` | `TC-FR-007` |
| `FR-008` | `POST /api/v1/jobs/{jobId}/apply` | `Applications`, `Jobs`, `CandidateProfiles` | `SCR-003 Job Detail` | `TC-FR-008` |
| `FR-009` | `POST /api/v1/jobs/{jobId}/save`, `DELETE /api/v1/jobs/{jobId}/save`, `GET /api/v1/candidates/me/saved-jobs` | `SavedJobs` | `SCR-003 Job Detail`, `SCR-010 Saved Jobs` | `TC-FR-009` |
| `FR-010` | `GET /api/v1/employer/jobs/{jobId}/applications`, `PATCH /api/v1/applications/{applicationId}/status` | `Applications`, `Jobs` | `SCR-011 Employer Applications` | `TC-FR-010` |
| `FR-011` | `POST /api/v1/posts`, `PUT /api/v1/posts/{postId}`, `DELETE /api/v1/posts/{postId}` | `Posts`, `PostCategories`, `Tags`, `PostTags` | `SCR-014 Post Editor` | `TC-FR-011` |
| `FR-012` | `GET /api/v1/posts`, `GET /api/v1/posts/{slug}` | `Posts`, `PostCategories`, `PostVotes`, `PostComments`, `PostBookmarks` | `SCR-012 Community Feed`, `SCR-013 Post Detail` | `TC-FR-012` |
| `FR-013` | `POST /api/v1/posts/{postId}/comments`, `POST /api/v1/posts/{postId}/votes`, `DELETE /api/v1/posts/{postId}/votes`, `POST /api/v1/posts/{postId}/bookmarks`, `DELETE /api/v1/posts/{postId}/bookmarks` | `PostComments`, `PostVotes`, `PostBookmarks`, `Posts` | `SCR-013 Post Detail` | `TC-FR-013` |
| `FR-014` | `GET /api/v1/admin/jobs`, `PATCH /api/v1/admin/jobs/{jobId}/status`, `DELETE /api/v1/admin/jobs/{jobId}` | `Jobs` | `SCR-016 Admin Job Moderation` | `TC-FR-014` |
| `FR-015` | `GET /api/v1/admin/dashboard` | `AspNetUsers`, `Jobs`, `Posts`, `Applications`, `JobCategories` | `SCR-017 Admin Dashboard` | `TC-FR-015` |
| `FR-016` | Tích hợp trong API tạo/sửa `jobs` và `posts` | `Jobs`, `Posts` | `SCR-009 Job Form`, `SCR-014 Post Editor` | `TC-FR-016` |
| `FR-017` | `POST /api/v1/posts/{postId}/comments/{commentId}/reply` | `PostComments`, `Posts` | `SCR-013 Post Detail` | `TC-FR-017` |
| `FR-018` | `GET /api/v1/users/{userId}/profile`, `GET /api/v1/candidates/me/activity` | `CandidateProfiles`, `EmployerProfiles`, `Posts`, `Applications`, `SavedJobs` | `SCR-015 User Profile` | `TC-FR-018` |
| `FR-019` | Tích hợp trong API tạo/sửa `jobs`, `posts`, `categories`, `tags` | `Jobs`, `Posts`, `JobCategories`, `PostCategories`, `Tags` | `SCR-002 Job List`, `SCR-003 Job Detail`, `SCR-012 Community Feed`, `SCR-013 Post Detail` | `TC-FR-019` |
| `FR-020` | `POST /api/v1/uploads/avatar`, `POST /api/v1/uploads/logo`, `POST /api/v1/uploads/post-thumbnail` | `CandidateProfiles`, `EmployerProfiles`, `Posts` | `SCR-006 Candidate Profile`, `SCR-007 Employer Profile`, `SCR-014 Post Editor` | `TC-FR-020` |
| `FR-021` | `POST /api/v1/posts/{postId}/submit`, `PATCH /api/v1/admin/posts/{postId}/status` | `Posts` | `SCR-014 Post Editor`, `SCR-018 Admin Post Moderation` | `TC-FR-021` |
| `FR-022` | `GET /api/v1/admin/trash`, `PATCH /api/v1/admin/jobs/{jobId}/restore`, `PATCH /api/v1/admin/posts/{postId}/restore` | `Jobs`, `Posts` | `SCR-019 Admin Trash Restore` | `TC-FR-022` |

## 8. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD |  | Khởi tạo khung |
| 0.2 | 2026-05-13 | Codex | Hoàn thiện functional requirements bám theo scope và database description, bao phủ toàn bộ mục 5 và 6 |
