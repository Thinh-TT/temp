# UI Sitemap And Wireframe

## 1. Mục tiêu tài liệu

- Định nghĩa cấu trúc màn hình, URL pattern và điều hướng chính cho toàn bộ ứng dụng.
- Đồng bộ với `2-functional-requirements.md` (traceability section) và `5-api-spec.md`.
- Làm nền cho thiết kế Razor Pages, phân chia partial view, và kiểm thử UI.

## 2. Quy ước

- Mã màn hình: `SCR-XXX`
- URL pattern: thống nhất với `5-api-spec.md`, dùng slug cho route công khai khi tính năng slug được bật.
- Role truy cập: `Guest`, `Candidate`, `Employer`, `Admin`.
- Mỗi màn hình phải xử lý 4 trạng thái UI: **Loading**, **Empty**, **Error**, **Success**.
- Component tái sử dụng được đánh dấu `[C]`.

## 3. Sitemap tổng quan

```
/ (Home)
├── /jobs (Job List)
│   └── /jobs/{slug} (Job Detail)
├── /posts (Community Feed)
│   └── /posts/{slug} (Post Detail)
├── /auth/login (Login)
├── /auth/register (Register)
├── /candidate/
│   ├── /candidate/profile (Candidate Profile - edit)
│   └── /candidate/saved-jobs (Saved Jobs)
├── /employer/
│   ├── /employer/profile (Employer Profile - edit)
│   ├── /employer/jobs (Employer Job List)
│   │   ├── /employer/jobs/create (Job Form - create)
│   │   ├── /employer/jobs/{jobId}/edit (Job Form - edit)
│   │   └── /employer/jobs/{jobId}/applications (Employer Applications)
├── /posts/create (Post Editor - create)
├── /posts/{postId}/edit (Post Editor - edit)
├── /users/{userId} (User Profile - public)
└── /admin/
    ├── /admin/dashboard (Admin Dashboard)
    ├── /admin/jobs (Admin Job Moderation)
    ├── /admin/posts (Admin Post Moderation)
    └── /admin/trash (Admin Trash Restore)
```

## 4. Danh sách màn hình đầy đủ

| Screen ID | Tên màn hình          | URL                                                    | Vai trò             | Phase  | Ghi chú                                               |
| --------- | --------------------- | ------------------------------------------------------ | ------------------- | ------ | ----------------------------------------------------- |
| SCR-001   | Home                  | `/`                                                    | All                 | Must   | Landing page, hero + featured jobs + trending posts   |
| SCR-002   | Job List              | `/jobs`                                                | All                 | Must   | Danh sách job + filter + pagination                   |
| SCR-003   | Job Detail            | `/jobs/{slug}`                                         | All                 | Must   | Chi tiết job + apply button (candidate) + save button |
| SCR-004   | Login                 | `/auth/login`                                          | Guest               | Must   | Form đăng nhập                                        |
| SCR-005   | Register              | `/auth/register`                                       | Guest               | Must   | Form đăng ký, chọn role Candidate/Employer            |
| SCR-006   | Candidate Profile     | `/candidate/profile`                                   | Candidate           | Must   | Form chỉnh sửa hồ sơ cá nhân; avatar upload (FR-020) là Should |
| SCR-007   | Employer Profile      | `/employer/profile`                                    | Employer            | Must   | Form chỉnh sửa hồ sơ công ty; logo upload (FR-020) là Should |
| SCR-008   | Employer Job List     | `/employer/jobs`                                       | Employer            | Must   | Danh sách job của employer + filter theo status       |
| SCR-009   | Job Form              | `/employer/jobs/create`, `/employer/jobs/{jobId}/edit` | Employer            | Must   | Form tạo/sửa job; MVP: textarea thường; Sau MVP: rich text editor (FR-016) |
| SCR-010   | Saved Jobs            | `/candidate/saved-jobs`                                | Candidate           | Must   | Danh sách job đã lưu                                  |
| SCR-011   | Employer Applications | `/employer/jobs/{jobId}/applications`                  | Employer            | Must   | Danh sách ứng viên + cập nhật trạng thái              |
| SCR-012   | Community Feed        | `/posts`                                               | All                 | Must   | Feed bài viết + sort newest/trending                  |
| SCR-013   | Post Detail           | `/posts/{slug}`                                        | All                 | Must   | Chi tiết bài viết + comment + upvote + bookmark       |
| SCR-014   | Post Editor           | `/posts/create`, `/posts/{postId}/edit`                | Candidate, Employer | Must   | Form tạo/sửa bài viết; thumbnail upload (FR-020) là Should |
| SCR-015   | User Profile          | `/users/{userId}`                                      | All                 | Should | Hồ sơ công khai + bài viết đã đăng                    |
| SCR-016   | Admin Job Moderation  | `/admin/jobs`                                          | Admin               | Must   | Danh sách job chờ duyệt + approve/hide/delete         |
| SCR-017   | Admin Dashboard       | `/admin/dashboard`                                     | Admin               | Must   | Tổng quan + biểu đồ                                   |
| SCR-018   | Admin Post Moderation | `/admin/posts`                                         | Admin               | Should | Danh sách post chờ duyệt + publish/hide               |
| SCR-019   | Admin Trash Restore   | `/admin/trash`                                         | Admin               | Should | Danh sách dữ liệu đã xóa mềm + restore                |

## 5. Luồng điều hướng chính

### 5.1 Guest flow (chưa đăng nhập)

```
Home (SCR-001)
 ├──► Job List (SCR-002)
 │     └──► Job Detail (SCR-003)
 ├──► Community Feed (SCR-012)
 │     └──► Post Detail (SCR-013)
 ├──► Login (SCR-004) → sau login → Home (nếu Candidate) / Employer Job List (nếu Employer)
 └──► Register (SCR-005) → sau register → Login (SCR-004)
```

### 5.2 Candidate flow

```
Login → Home / Job List
 ├──► Job Detail (SCR-003)
 │     ├──► Apply (tại SCR-003, toast thành công)
 │     └──► Save/Unsave (tại SCR-003, toggle button)
 ├──► Candidate Profile (SCR-006) → chỉnh sửa + upload avatar + CV
 ├──► Saved Jobs (SCR-010) → xem + bỏ lưu
 ├──► Community Feed (SCR-012)
 │     └──► Post Detail (SCR-013)
 │           ├──► Comment
 │           ├──► Reply (Sau MVP)
 │           ├──► Upvote / Bỏ upvote
 │           └──► Bookmark
 ├──► Post Editor (SCR-014) → tạo bài mới từ navbar
 │     └──► Post Editor (SCR-014) → sửa bài từ Post Detail
 └──► User Profile (SCR-015) → xem hồ sơ của mình hoặc người khác
```

### 5.3 Employer flow

```
Login → Employer Job List (SCR-008)
 ├──► Job Form (SCR-009)
 │     ├──► Tạo job mới → chờ duyệt → redirect về SCR-008
 │     └──► Sửa job → chờ duyệt lại → redirect về SCR-008
 ├──► Employer Applications (SCR-011)
 │     ├──► Xem CV snapshot
 │     ├──► Cập nhật trạng thái: Reviewed / Interview / Rejected / Accepted
 │     └──► Thêm ghi chú
 ├──► Employer Profile (SCR-007) → chỉnh sửa + upload logo
 ├──► Community Feed (SCR-012) → tương tác như Candidate
 ├──► Post Editor (SCR-014) → tạo/sửa bài viết
 └──► User Profile (SCR-015) → xem hồ sơ công khai
```

### 5.4 Admin flow

```
Login → Admin Dashboard (SCR-017)
 ├──► Admin Job Moderation (SCR-016)
 │     ├──► Approve job
 │     ├──► Hide job
 │     └──► Delete job
 ├──► Admin Post Moderation (SCR-018) [Sau MVP]
 │     ├──► Publish post
 │     └──► Hide post
 └──► Admin Trash Restore (SCR-019) [Sau MVP]
       └──► Restore job hoặc post đã xóa mềm
```

## 6. Wireframe chi tiết từng màn hình

### SCR-001 - Home

- **Mục tiêu**: Trang đích, giới thiệu nền tảng, dẫn hướng nhanh đến job và bài viết nổi bật.
- **Thành phần chính**:
  - `[C]` Navbar (logo, menu, login/register hoặc user dropdown)
  - Hero section (tagline + CTA "Tìm việc ngay" → `/jobs`)
  - Featured Jobs (top 6 job `Approved`, gần deadline nhất)
  - Trending Posts (top 4 post `Published`, theo trending score)
  - Footer
- **Empty state**: Hiển thị placeholder "Chưa có tin tuyển dụng nào" nếu không có job.
- **Error state**: Fallback section với thông báo lỗi nhẹ, không crash toàn trang.
- **Loading state**: Skeleton card cho job và post.

### SCR-002 - Job List

- **Mục tiêu**: Danh sách job công khai với tìm kiếm và lọc.
- **Thành phần chính**:
  - `[C]` Navbar
  - Search bar (keyword input + nút search)
  - Filter bar: dropdown `Category`, dropdown `Location`, nút "Clear filter"
  - Sort: mặc định theo `CreatedAt DESC`
  - Job card list: mỗi card hiển thị title, company name, location, salary range, deadline, category tag
  - `[C]` Pagination
  - Footer
- **Empty state**: "Không tìm thấy công việc phù hợp." + minh họa + CTA "Xóa bộ lọc".
- **Error state**: Toast error nếu API lỗi.
- **Loading state**: Skeleton job cards (6 item).
- **CTA chính**: Click vào job card → `SCR-003`.

### SCR-003 - Job Detail

- **Mục tiêu**: Hiển thị đầy đủ thông tin job, cho phép candidate apply và save.
- **Thành phần chính**:
  - `[C]` Navbar
  - Breadcrumb: Home > Jobs > [Job Title]
  - Job header: title, company name + logo, location, salary, employment type, experience level, deadline, view count
  - Tab/pane: Description | Requirements | Benefits
  - Company info sidebar: logo, company name, description, website link
  - Action buttons (cho Candidate):
    - "Apply Now" button → mở modal xác nhận apply
    - "Save" / "Saved" toggle button
    - "Ứng tuyển ngay" disabled nếu đã apply (hiển thị "Đã ứng tuyển") hoặc job hết hạn
  - Login prompt cho Guest: "Đăng nhập để ứng tuyển" → link tới `SCR-004`
  - Related jobs (cùng category, tối đa 4)
  - Footer
- **Empty state**: Không áp dụng (luôn có nội dung từ API).
- **Error state**: 404 page nếu job không tồn tại.
- **Loading state**: Skeleton cho nội dung job.
- **CTA chính**: Apply Now (Candidate), Save Job (Candidate), Đăng nhập (Guest).

### SCR-004 - Login

- **Mục tiêu**: Xác thực người dùng.
- **Thành phần chính**:
  - Minimal header (chỉ logo)
  - Form: email/username input, password input, "Remember me" checkbox, "Login" button
  - Link: "Chưa có tài khoản? Đăng ký" → `SCR-005`
  - Validation message inline (email sai định dạng, thiếu mật khẩu)
  - Toast error nếu sai thông tin đăng nhập
- **Error state**: Toast "Sai email hoặc mật khẩu" / "Tài khoản bị khóa".
- **Loading state**: Button loading spinner khi đang gọi API.
- **CTA chính**: Login button.

### SCR-005 - Register

- **Mục tiêu**: Tạo tài khoản mới.
- **Thành phần chính**:
  - Minimal header (chỉ logo)
  - Form: username, email, password, confirm password, role selector (radio: Candidate / Employer)
  - Validation inline: password strength, confirm password match, email format, username length
  - Link: "Đã có tài khoản? Đăng nhập" → `SCR-004`
- **Error state**: Toast "Email đã tồn tại" / "Username đã tồn tại".
- **Loading state**: Button loading spinner.
- **CTA chính**: Register button.

### SCR-006 - Candidate Profile

- **Mục tiêu**: Quản lý hồ sơ ứng viên.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Avatar: MVP dùng text input URL; Sau MVP dùng upload + preview (FR-020)
  - Form fields: FullName, DateOfBirth (datepicker), Gender (dropdown), Address, Bio (textarea), ExperienceYears (number)
  - Skills selector: multi-select dropdown hoặc tag input, chọn từ danh sách `Skills` có sẵn
  - CV upload: input file (chỉ nhận PDF), hiển thị tên file hiện tại, nút "Tải CV mới"
  - "Lưu thay đổi" button
- **Error state**: Toast lỗi validation / file không hợp lệ.
- **Loading state**: Skeleton form.
- **CTA chính**: Lưu thay đổi.

### SCR-007 - Employer Profile

- **Mục tiêu**: Quản lý hồ sơ doanh nghiệp.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Logo: MVP dùng text input URL; Sau MVP dùng upload + preview (FR-020)
  - Form fields: CompanyName (required), Description (textarea), Website (url), Address, CompanySize (dropdown: 1-10, 11-50, 51-200, 201-500, 500+)
  - "Lưu thay đổi" button
- **Error state**: Toast "Vui lòng nhập tên công ty" nếu thiếu CompanyName.
- **Loading state**: Skeleton form.
- **CTA chính**: Lưu thay đổi.

### SCR-008 - Employer Job List

- **Mục tiêu**: Quản lý tin tuyển dụng của employer.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - "Đăng tin mới" button → `SCR-009` (create mode)
  - Filter tabs: All / Pending / Approved / Hidden
  - Job table/card list: title, status badge (màu: Pending=vàng, Approved=xanh, Hidden=đỏ), deadline, createdAt, action buttons (Edit, Delete)
  - Xác nhận modal khi xóa
  - `[C]` Pagination
- **Empty state**: "Bạn chưa có tin tuyển dụng nào. Đăng tin ngay!"
- **Error state**: Toast lỗi.
- **Loading state**: Skeleton table rows.
- **CTA chính**: Đăng tin mới.

### SCR-009 - Job Form

- **Mục tiêu**: Tạo hoặc chỉnh sửa tin tuyển dụng.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Form fields:
    - Title (text input, required)
    - JobCategory (dropdown, required)
    - Description (MVP: textarea; Sau MVP: rich text editor Quill.js, required)
    - Requirement (MVP: textarea; Sau MVP: rich text editor Quill.js, required)
    - Benefits (MVP: textarea; Sau MVP: rich text editor Quill.js)
    - SalaryMin, SalaryMax (number inputs)
    - Location (text input)
    - ExperienceLevel (dropdown: Fresher/Junior/Senior)
    - EmploymentType (dropdown: FullTime/PartTime)
    - Deadline (datepicker, required)
  - Conditional: nếu đang sửa, hiển thị trạng thái hiện tại + cảnh báo "Sửa nội dung sẽ gửi lại kiểm duyệt"
  - "Lưu" và "Hủy" buttons
- **Empty state**: Form trống (create mode).
- **Error state**: Inline validation errors, toast lỗi server.
- **Loading state**: Skeleton form.
- **CTA chính**: Lưu / Cập nhật.

### SCR-010 - Saved Jobs

- **Mục tiêu**: Xem danh sách job đã lưu.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Job card list (tương tự SCR-002 nhưng có thêm nút "Bỏ lưu")
  - `[C]` Pagination
- **Empty state**: "Bạn chưa lưu công việc nào. Khám phá việc làm ngay!" + link tới `/jobs`.
- **Loading state**: Skeleton job cards.
- **CTA chính**: Click vào job card → `SCR-003`.

### SCR-011 - Employer Applications

- **Mục tiêu**: Quản lý ứng viên cho một job.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Breadcrumb: Employer Jobs > [Job Title] > Applications
  - Filter tabs: All / Pending / Reviewed / Interview / Accepted / Rejected
  - Application table:
    - Candidate name, avatar
    - CV snapshot link (mở PDF)
    - Cover letter preview (expandable)
    - Status badge
    - Applied date
    - Action: dropdown chọn trạng thái mới + nút "Cập nhật"
    - Note input (inline, để thêm ghi chú)
  - `[C]` Pagination
- **Empty state**: "Chưa có ứng viên nào cho tin tuyển dụng này."
- **Loading state**: Skeleton table rows.
- **CTA chính**: Cập nhật trạng thái.

### SCR-012 - Community Feed

- **Mục tiêu**: Hiển thị feed bài viết cộng đồng.
- **Thành phần chính**:
  - `[C]` Navbar
  - Sort toggle: "Mới nhất" / "Xu hướng"
  - Filter: dropdown Category, dropdown Tag (tùy chọn)
  - "Viết bài" button (chỉ Candidate/Employer) → `SCR-014`
  - Post card list: thumbnail, title, summary, author (avatar + display name), category tag, upvote count, comment count, bookmark count, view count, thời gian đăng
  - `[C]` Pagination
- **Empty state**: "Chưa có bài viết nào. Hãy là người đầu tiên chia sẻ!" + CTA viết bài.
- **Loading state**: Skeleton post cards.
- **CTA chính**: Click vào post card → `SCR-013`.

### SCR-013 - Post Detail

- **Mục tiêu**: Hiển thị đầy đủ nội dung bài viết và tương tác.
- **Thành phần chính**:
  - `[C]` Navbar
  - Breadcrumb: Community > [Post Title]
  - Post header: title, author (avatar + name + link tới SCR-015), category, tags (badge), thời gian đăng
  - Post content (HTML đã sanitize)
  - Interaction bar:
    - Upvote button (với count)
    - Comment icon (với count, anchor tới comment section)
    - Bookmark button (với count)
    - Share button (copy link)
  - Comment section:
    - Comment input (cho Candidate/Employer/Admin)
    - Danh sách comment gốc, mỗi comment có: avatar, tên, thời gian, nội dung, nút Reply
    - Reply inline form (hiện dưới comment gốc khi bấm Reply) [Sau MVP]
    - Reply list (thụt lề 1 cấp)
  - Author actions (nếu là tác giả): Edit button → `SCR-014`, Delete button (kèm confirm)
  - Admin actions (nếu là admin): Hide button
- **Empty state**: Không áp dụng.
- **Error state**: 404 nếu post không tồn tại.
- **Loading state**: Skeleton cho nội dung và comment.
- **CTA chính**: Upvote, Comment, Bookmark.

### SCR-014 - Post Editor

- **Mục tiêu**: Tạo hoặc chỉnh sửa bài viết cộng đồng.
- **Thành phần chính**:
  - `[C]` Navbar (user menu)
  - Form fields:
    - Title (text input, required)
    - Summary (textarea, max 500 ký tự, hiển thị counter)
    - PostCategory (dropdown, required)
    - Content (rich text editor - Quill.js, required)
    - Tags (multi-select / tag input từ danh sách `Tags` có sẵn)
    - Thumbnail: MVP dùng text input URL; Sau MVP dùng upload + preview (FR-020)
  - Action buttons:
    - MVP: "Đăng bài" (publish trực tiếp)
    - Sau MVP: "Lưu nháp" + "Gửi duyệt"
    - "Hủy"
- **Empty state**: Form trống (create mode).
- **Error state**: Inline validation errors.
- **Loading state**: Skeleton form.
- **CTA chính**: Đăng bài / Lưu nháp / Gửi duyệt.

### SCR-015 - User Profile

- **Mục tiêu**: Trang hồ sơ công khai của người dùng (Sau MVP).
- **Thành phần chính**:
  - `[C]` Navbar
  - Profile header: avatar, display name, role badge, bio
  - Nếu là Candidate: skills list
  - Nếu là Employer: company name, logo, description, website, company size
  - Tab: "Bài viết" (mặc định)
  - Post list của user (card đơn giản)
  - Nếu là chính chủ Candidate: tab "Đã ứng tuyển" (danh sách job đã apply) + tab "Đã lưu" (danh sách job đã save)
  - `[C]` Pagination
- **Empty state**: "Người dùng chưa có bài viết nào."
- **Error state**: 404 nếu user không tồn tại.
- **Loading state**: Skeleton profile header + post list.
- **CTA chính**: Xem bài viết.

### SCR-016 - Admin Job Moderation

- **Mục tiêu**: Kiểm duyệt tin tuyển dụng.
- **Thành phần chính**:
  - `[C]` Admin sidebar / navbar
  - Filter tabs: Pending / Approved / Hidden / All
  - Job table: title, company name, category, status badge, createdAt, action buttons
  - Action buttons theo trạng thái:
    - Pending: [Approve] [Hide] [Delete]
    - Approved: [Hide] [Delete]
    - Hidden: [Approve]
  - Modal xác nhận cho mỗi action
  - `[C]` Pagination
- **Empty state**: "Không có tin tuyển dụng nào cần duyệt."
- **Loading state**: Skeleton table.
- **CTA chính**: Approve / Hide / Delete.

### SCR-017 - Admin Dashboard

- **Mục tiêu**: Tổng quan hệ thống.
- **Thành phần chính**:
  - `[C]` Admin sidebar / navbar
  - Summary cards (4 card):
    - Tổng Users (icon người dùng)
    - Tổng Jobs (icon công việc)
    - Tổng Posts (icon bài viết)
    - Tổng Applications (icon đơn ứng tuyển)
  - Biểu đồ 1: Bar chart - Job theo Category (Chart.js)
  - Biểu đồ 2: Line chart - Application count theo ngày (Chart.js, mặc định 7-30 ngày gần nhất)
  - Có thể thêm date range filter
- **Empty state**: Card hiển thị 0, biểu đồ rỗng với trục tọa độ (không crash).
- **Error state**: Toast lỗi cục bộ nếu một biểu đồ không tải được.
- **Loading state**: Skeleton card + skeleton chart area.
- **CTA chính**: Không có CTA chính, đây là màn hình xem.

### SCR-018 - Admin Post Moderation

- **Mục tiêu**: Kiểm duyệt bài viết cộng đồng (Sau MVP).
- **Thành phần chính**:
  - `[C]` Admin sidebar / navbar
  - Filter tabs: Pending / Published / Hidden / All
  - Post table: title, author, category, status badge, createdAt, action buttons
  - Action buttons theo trạng thái:
    - Pending: [Publish] [Hide]
    - Published: [Hide]
    - Hidden: [Publish]
  - Modal xác nhận cho mỗi action
  - `[C]` Pagination
- **Empty state**: "Không có bài viết nào cần duyệt."
- **Loading state**: Skeleton table.
- **CTA chính**: Publish / Hide.

### SCR-019 - Admin Trash Restore

- **Mục tiêu**: Khôi phục dữ liệu đã xóa mềm (Sau MVP).
- **Thành phần chính**:
  - `[C]` Admin sidebar / navbar
  - Filter: dropdown Entity Type (Job / Post)
  - Table: entity type icon, title, thời gian xóa, nút [Restore]
  - Modal xác nhận restore
  - `[C]` Pagination
- **Empty state**: "Không có dữ liệu nào trong thùng rác."
- **Loading state**: Skeleton table.
- **CTA chính**: Restore.

## 7. Component Inventory (tái sử dụng)

| Component ID | Tên                | Mô tả                                                                                            | Sử dụng trong                                                          |
| ------------ | ------------------ | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| C-001        | Navbar             | Thanh điều hướng chính, responsive, user dropdown khi đã login                                   | Tất cả màn hình                                                        |
| C-002        | Admin Sidebar      | Sidebar navigation cho admin area                                                                | SCR-016, SCR-017, SCR-018, SCR-019                                     |
| C-003        | Pagination         | Phân trang với page number, prev/next, hiển thị tổng                                             | SCR-002, SCR-008, SCR-010, SCR-011, SCR-012, SCR-016, SCR-018, SCR-019 |
| C-004        | Job Card           | Card hiển thị job (title, company, location, salary, deadline)                                   | SCR-002, SCR-010                                                       |
| C-005        | Post Card          | Card hiển thị bài viết (thumbnail, title, summary, author, stats)                                | SCR-012                                                                |
| C-006        | Status Badge       | Badge màu theo trạng thái (Pending=vàng, Approved/Published=xanh, Hidden/Rejected=đỏ, Draft=xám) | Nhiều màn hình                                                         |
| C-007        | Toast Notification | Toast thông báo thành công/lỗi, auto-dismiss                                                     | Tất cả màn hình                                                        |
| C-008        | Confirm Modal      | Modal xác nhận hành động nguy hiểm (xóa, thay đổi trạng thái)                                    | Nhiều màn hình                                                         |
| C-009        | Rich Text Editor   | Quill.js editor wrapper, sanitize HTML trước khi gửi                                             | SCR-009, SCR-014                                                       |
| C-010        | File Upload        | Input file với preview, validate type/size                                                       | SCR-006, SCR-007, SCR-014                                              |
| C-011        | Empty State        | Minh họa + message + CTA khi danh sách rỗng                                                      | Tất cả màn hình list                                                   |
| C-012        | Skeleton Loader    | Loading placeholder hình chữ nhật xám, animate pulse                                             | Tất cả màn hình                                                        |
| C-013        | Breadcrumb         | Đường dẫn phân cấp                                                                               | SCR-003, SCR-011, SCR-013                                              |
| C-014        | User Dropdown      | Menu dropdown khi đã login: Profile, Saved Jobs (candidate), My Jobs (employer), Logout          | Navbar (C-001)                                                         |

## 8. Quy tắc UI chung

### 8.1 Responsive breakpoints

- Mobile: `< 768px` — single column, hamburger menu
- Tablet: `768px - 1024px` — 2 column where applicable
- Desktop: `> 1024px` — full layout

### 8.2 Quy tắc hiển thị

- Phân trang mặc định: `pageSize = 10`.
- Toast auto-dismiss sau 5 giây.
- Button hành động chính màu primary (Bootstrap `btn-primary`), button hủy màu secondary.
- Status badge màu: Pending=vàng (`warning`), Approved/Published=xanh (`success`), Hidden=xám (`secondary`), Rejected=đỏ (`danger`), Draft=xám nhạt (`light`).
- Empty state luôn có minh họa + CTA dẫn hướng.

### 8.3 Accessibility cơ bản

- Semantic HTML (`<nav>`, `<main>`, `<article>`, `<section>`).
- `alt` cho tất cả ảnh.
- `aria-label` cho button chỉ có icon.
- Contrast ratio tối thiểu `4.5:1` cho văn bản.
- Focus visible cho keyboard navigation.

### 8.4 Trạng thái UI bắt buộc cho mọi màn hình

| Trạng thái | Mô tả                           | Component                      |
| ---------- | ------------------------------- | ------------------------------ |
| Loading    | Hiển thị khi đang fetch dữ liệu | Skeleton Loader (C-012)        |
| Empty      | Không có dữ liệu để hiển thị    | Empty State (C-011)            |
| Error      | API trả lỗi hoặc network error  | Toast (C-007) + message inline |
| Success    | Dữ liệu hiển thị bình thường    | Nội dung chính                 |

## 9. Ma trận màn hình × API endpoint

| Screen ID | API endpoint(s) chính                                                                                                                                                                                                                                                                        |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SCR-001   | `GET /api/v1/jobs` (featured), `GET /api/v1/posts` (trending)                                                                                                                                                                                                                                |
| SCR-002   | `GET /api/v1/jobs`                                                                                                                                                                                                                                                                           |
| SCR-003   | `GET /api/v1/jobs/{slug}`, `POST /api/v1/jobs/{jobId}/apply`, `POST /api/v1/jobs/{jobId}/save`, `DELETE /api/v1/jobs/{jobId}/save`                                                                                                                                                           |
| SCR-004   | `POST /api/v1/auth/login`                                                                                                                                                                                                                                                                    |
| SCR-005   | `POST /api/v1/auth/register`                                                                                                                                                                                                                                                                 |
| SCR-006   | `GET /api/v1/candidates/me`, `PUT /api/v1/candidates/me`, `POST /api/v1/uploads/avatar`                                                                                                                                                                                                      |
| SCR-007   | `GET /api/v1/employers/me`, `PUT /api/v1/employers/me`, `POST /api/v1/uploads/logo`                                                                                                                                                                                                          |
| SCR-008   | `GET /api/v1/employer/jobs`                                                                                                                                                                                                                                                                  |
| SCR-009   | `POST /api/v1/jobs`, `PUT /api/v1/jobs/{jobId}`                                                                                                                                                                                                                                              |
| SCR-010   | `GET /api/v1/candidates/me/saved-jobs`, `DELETE /api/v1/jobs/{jobId}/save`                                                                                                                                                                                                                   |
| SCR-011   | `GET /api/v1/employer/jobs/{jobId}/applications`, `PATCH /api/v1/applications/{applicationId}/status`                                                                                                                                                                                        |
| SCR-012   | `GET /api/v1/posts`                                                                                                                                                                                                                                                                          |
| SCR-013   | `GET /api/v1/posts/{slug}`, `POST /api/v1/posts/{postId}/comments`, `POST /api/v1/posts/{postId}/comments/{commentId}/reply`, `POST /api/v1/posts/{postId}/votes`, `DELETE /api/v1/posts/{postId}/votes`, `POST /api/v1/posts/{postId}/bookmarks`, `DELETE /api/v1/posts/{postId}/bookmarks` |
| SCR-014   | `POST /api/v1/posts`, `PUT /api/v1/posts/{postId}`, `POST /api/v1/uploads/post-thumbnail`, `POST /api/v1/posts/{postId}/submit`                                                                                                                                                              |
| SCR-015   | `GET /api/v1/users/{userId}/profile`, `GET /api/v1/candidates/me/activity`                                                                                                                                                                                                                   |
| SCR-016   | `GET /api/v1/admin/jobs`, `PATCH /api/v1/admin/jobs/{jobId}/status`, `DELETE /api/v1/admin/jobs/{jobId}`                                                                                                                                                                                     |
| SCR-017   | `GET /api/v1/admin/dashboard`                                                                                                                                                                                                                                                                |
| SCR-018   | `GET /api/v1/admin/posts`, `PATCH /api/v1/admin/posts/{postId}/status`                                                                                                                                                                                                                       |
| SCR-019   | `GET /api/v1/admin/trash`, `PATCH /api/v1/admin/jobs/{jobId}/restore`, `PATCH /api/v1/admin/posts/{postId}/restore`                                                                                                                                                                          |

## 10. Changelog

| Version | Ngày       | Người cập nhật/ Tên Model | Nội dung                                                                                                        |
| ------- | ---------- | ------------------------- | --------------------------------------------------------------------------------------------------------------- |
| 0.1     | YYYY-MM-DD |                           | Khởi tạo khung                                                                                                  |
| 1.0     | 2026-05-14 | DeepSeek V4 Pro           | Hoàn thiện sitemap 19 màn hình, wireframe chi tiết, component inventory, quy tắc UI chung, ma trận màn hình-API |
