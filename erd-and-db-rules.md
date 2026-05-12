# ERD And DB Rules

## 1. Mục tiêu tài liệu
- Chốt mô hình dữ liệu và các ràng buộc ở mức database.

## 2. Danh sách bảng
| Nhóm | Bảng |
|---|---|
| User & Auth | `AspNetUsers`, `AspNetRoles`, `CandidateProfiles`, `EmployerProfiles`, `Skills`, `CandidateSkills` |
| Recruitment | `JobCategories`, `Jobs`, `Applications`, `SavedJobs` |
| Community | `PostCategories`, `Posts`, `Tags`, `PostTags`, `PostComments`, `PostVotes`, `PostBookmarks` |
| Admin & System | `Notifications`, `Reports` |

## 3. Quan hệ chính (ERD text)
- `AspNetUsers (1) - (0..1) CandidateProfiles`
- `AspNetUsers (1) - (0..1) EmployerProfiles`
- `EmployerProfiles (1) - (N) Jobs`
- `JobCategories (1) - (N) Jobs`
- `Jobs (1) - (N) Applications`
- `CandidateProfiles (1) - (N) Applications`
- `CandidateProfiles (N) - (N) Skills` qua `CandidateSkills`
- `Posts (N) - (N) Tags` qua `PostTags`
- `Posts (1) - (N) PostComments`
- `Posts (1) - (N) PostVotes`
- `Posts (1) - (N) PostBookmarks`

## 4. Khóa và ràng buộc
### 4.1 Primary key
- Quy ước: `<EntityName>Id` (trừ bảng Identity).

### 4.2 Foreign key
- Dùng `ON DELETE RESTRICT` mặc định.
- Chỉ dùng cascade khi thực sự cần và đã review tác động dữ liệu.

### 4.3 Unique constraints
| Bảng | Cột | Lý do |
|---|---|---|
| `AspNetUsers` | `UserName`, `Email` | Tránh trùng tài khoản |
| `Skills` | `Name` | Tránh trùng kỹ năng |
| `JobCategories` | `Slug` | SEO route duy nhất |
| `Tags` | `Slug` | SEO route duy nhất |

### 4.4 Index đề xuất
| Bảng | Cột index | Mục đích |
|---|---|---|
| `Jobs` | `Status`, `CreatedAt` | Danh sách job |
| `Jobs` | `JobCategoryId`, `Location` | Filter nhanh |
| `Applications` | `JobId`, `CandidateId` | Truy vấn workflow |
| `Posts` | `Status`, `CreatedAt` | Feed cộng đồng |

## 5. Quy tắc dữ liệu
- Soft delete bằng `IsDeleted` cho bảng nội dung chính.
- Luôn ghi `CreatedAt`, `UpdatedAt` ở UTC.
- Enum trạng thái phải map rõ ở tầng code.
- Không lưu dữ liệu nhạy cảm dạng plain text.

## 6. Migration rules
- Mỗi thay đổi schema phải có migration riêng.
- Không sửa migration đã chạy production.
- Có script rollback tương ứng cho thay đổi rủi ro cao.

## 7. Seed data
- Roles mặc định: `Admin`, `Employer`, `Candidate`.
- Dữ liệu danh mục cơ bản: `JobCategories`, `PostCategories`.

## 8. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

