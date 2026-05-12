# Implementation Plan

## 1. Mục tiêu
- Lập kế hoạch triển khai từ MVP đến mở rộng.

## 2. Giả định
- Team size:
- Thời lượng dự án:
- Công nghệ đã chốt:

## 3. Milestone tổng quan
| Milestone | Mục tiêu | Thời gian | Output |
|---|---|---|---|
| M1 | Setup nền tảng | Week 1 | Repo, CI, auth base |
| M2 | Recruitment core | Week 2-3 | Jobs, applications |
| M3 | Community core | Week 4 | Posts, comments |
| M4 | Admin + polish | Week 5 | Dashboard, moderation |
| M5 | Test + demo | Week 6 | UAT pass, release candidate |

## 4. Kế hoạch theo phase
### Phase 1 - Foundation
- Setup solution structure.
- Auth + roles + seed dữ liệu cơ bản.
- Base layout UI.

### Phase 2 - Recruitment Core
- CRUD jobs.
- Job search/filter.
- Apply workflow.

### Phase 3 - Community
- CRUD posts.
- Tags/categories.
- Comment/upvote/bookmark.

### Phase 4 - Admin & Dashboard
- Duyệt job/post.
- Báo cáo vi phạm.
- Dashboard số liệu.

### Phase 5 - Stabilize
- Fix bug.
- Tối ưu hiệu năng.
- Hoàn tất tài liệu demo.

## 5. Dependency map
| Hạng mục | Phụ thuộc |
|---|---|
| Application workflow | Auth, CandidateProfile, Jobs |
| Community moderation | Posts, Roles, Admin module |
| Dashboard | Dữ liệu từ Jobs/Applications/Posts |

## 6. Rủi ro và phương án
| Rủi ro | Mức độ | Ảnh hưởng | Giảm thiểu |
|---|---|---|---|
| Chậm tiến độ backend | High | Trễ tích hợp FE | Chốt API sớm, mock API |
| Scope creep | Medium | Vỡ kế hoạch | Bám MVP, change request rõ ràng |

## 7. Definition of Done
- Có code + test cơ bản.
- Pass review.
- Tài liệu cập nhật.
- Demo được end-to-end.

## 8. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

