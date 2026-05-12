# Test Plan

## 1. Mục tiêu tài liệu
- Định nghĩa chiến lược và phạm vi kiểm thử cho MVP.

## 2. Phạm vi kiểm thử
- In scope:
- Out of scope:

## 3. Môi trường kiểm thử
- Backend:
- Database:
- Frontend:
- Test accounts:

## 4. Test strategy
### 4.1 Unit test
- Service layer business rules.
- Validator.

### 4.2 Integration test
- API + DB integration.
- Auth + authorization flows.

### 4.3 UI/Manual test
- Luồng người dùng chính.
- Responsive cơ bản.

## 5. Danh sách test scenario chính
| ID | Module | Scenario | Loại test | Priority |
|---|---|---|---|---|
| TC-001 | Auth | Login hợp lệ | Integration | High |
| TC-002 | Recruitment | Employer tạo job | Integration | High |
| TC-003 | Recruitment | Candidate apply job | Integration | High |
| TC-004 | Community | Tạo post và comment | Integration | Medium |
| TC-005 | Admin | Duyệt job/post | Integration | High |

## 6. Checklist UAT
- Đăng ký/đăng nhập theo từng role chạy đúng.
- Employer đăng job và quản lý applications.
- Candidate tìm job, apply, save job.
- Cộng đồng đăng bài, bình luận, tương tác.
- Admin duyệt và ẩn nội dung vi phạm.

## 7. Defect management
- Mức độ lỗi: `Critical`, `Major`, `Minor`.
- Trạng thái lỗi: `Open`, `In Progress`, `Resolved`, `Closed`.
- SLA fix đề xuất:
- Critical:
- Major:
- Minor:

## 8. Entry/Exit criteria
### Entry criteria
- Môi trường test sẵn sàng.
- API/spec đã chốt cho phase.

### Exit criteria
- Không còn lỗi `Critical`.
- Tỷ lệ pass test case mục tiêu:
- UAT sign-off:

## 9. Test report template
| Build | Tổng test | Pass | Fail | Blocked | Ghi chú |
|---|---|---|---|---|---|
| | | | | | |

## 10. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

