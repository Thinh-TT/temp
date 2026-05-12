# UI Sitemap And Wireframe

## 1. Mục tiêu tài liệu
- Định nghĩa cấu trúc màn hình và điều hướng chính.

## 2. Quy ước
- Mã màn hình: `SCR-XXX`
- Role truy cập: `Guest`/`Candidate`/`Employer`/`Admin`

## 3. Sitemap tổng quan
1. Public
2. Candidate area
3. Employer area
4. Admin area

## 4. Danh sách màn hình
| Screen ID | Tên màn hình | URL | Vai trò | Ghi chú |
|---|---|---|---|---|
| SCR-001 | Home | `/` | All | |
| SCR-002 | Job List | `/jobs` | All | |
| SCR-003 | Job Detail | `/jobs/{slug}` | All | |
| SCR-004 | Login | `/auth/login` | Guest | |
| SCR-005 | Register | `/auth/register` | Guest | |

## 5. Luồng điều hướng chính
### 5.1 Candidate flow
1. Xem danh sách job.
2. Lọc và tìm job.
3. Xem chi tiết job.
4. Apply hoặc Save job.

### 5.2 Employer flow
1. Tạo job.
2. Chỉnh sửa job.
3. Theo dõi applications.
4. Cập nhật trạng thái ứng viên.

### 5.3 Admin flow
1. Duyệt job/post.
2. Ẩn nội dung vi phạm.
3. Xem dashboard tổng quan.

## 6. Wireframe checklist cho từng màn hình
### SCR-XXX - [Tên màn hình]
- Mục tiêu màn hình:
- Thành phần chính:
- Header
- Search/filter
- Main content
- Pagination
- Toast/alert
- Empty state:
- Error state:
- Loading state:
- CTA chính:

## 7. Quy tắc UI chung
- Responsive desktop/mobile.
- Điều hướng nhất quán.
- Hiển thị thông báo lỗi rõ ràng.
- Có phân trang ở danh sách lớn.

## 8. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

