# Task Board

## 1. Mục Tiêu

Bảng theo dõi công việc triển khai dự án, phân chia theo phase và module chức năng. Mỗi task gắn với ID use case tương ứng từ `03-functional/01-functional-requirements.md`.

## 2. Quy Ước Trạng Thái

| Ký hiệu | Trạng thái | Ý nghĩa |
|---------|-----------|---------|
| `[ ]`   | Todo      | Chưa bắt đầu |
| `[~]`   | In Progress | Đang thực hiện |
| `[x]`   | Done      | Hoàn thành |
| `[!]`   | Blocked   | Bị chặn, cần xử lý dependency |

## 3. Phase 1 — Foundation

| ID | Task | Use Case | Status | Assignee | Ghi chú |
|----|------|----------|--------|----------|---------|
| T-001 | Khởi tạo project ASP.NET Core (.NET 8) | — | `[ ]` | | Solution structure |
| T-002 | Cấu hình EF Core + SQL Server | — | `[ ]` | | DbContext, connection string |
| T-003 | Cấu hình ASP.NET Identity | — | `[ ]` | | ApplicationUser, Roles |
| T-004 | Thiết lập Razor Pages + Bootstrap 5 | — | `[ ]` | | Layout, assets |
| T-005 | Tạo migration khởi tạo DB | — | `[ ]` | | Từ 02-architecture |

## 4. Phase 2 — Auth & Profile

| ID | Task | Use Case | Status | Assignee | Ghi chú |
|----|------|----------|--------|----------|---------|
| T-006 | Đăng ký tài khoản | FR-001 | `[ ]` | | Candidate + Employer |
| T-007 | Đăng nhập / Đăng xuất | FR-002 | `[ ]` | | JWT Bearer |
| T-008 | Quản lý hồ sơ ứng viên | FR-003 | `[ ]` | | Avatar, skills, CV |
| T-009 | Quản lý hồ sơ nhà tuyển dụng | FR-004 | `[ ]` | | Company info, logo |

## 5. Phase 3 — Recruitment Core

| ID | Task | Use Case | Status | Assignee | Ghi chú |
|----|------|----------|--------|----------|---------|
| T-010 | CRUD Job (Employer) | FR-005 | `[ ]` | | Tạo/sửa/xóa job |
| T-011 | Job List + Search + Filter | FR-006 | `[ ]` | | Pagination |
| T-012 | Job Detail | FR-007 | `[ ]` | | Slug routing |
| T-013 | Apply Job | FR-008 | `[ ]` | | CV attachment |
| T-014 | Save/Unsave Job | FR-009 | `[ ]` | | Bookmark |
| T-015 | Application Management | FR-010 | `[ ]` | | Status workflow |
| T-016 | Admin Job Moderation | FR-014 | `[ ]` | | Approve/Hide |

## 6. Phase 4 — Community

| ID | Task | Use Case | Status | Assignee | Ghi chú |
|----|------|----------|--------|----------|---------|
| T-017 | CRUD Post | FR-011 | `[ ]` | | Category, tag |
| T-018 | Community Feed | FR-012 | `[ ]` | | Newest + Trending |
| T-019 | Comment + Upvote + Bookmark | FR-013 | `[ ]` | | Interaction |
| T-020 | Rich Text Editor | FR-016 | `[ ]` | | Quill.js |
| T-021 | Nested Comment (1 cấp) | FR-017 | `[ ]` | | Reply |

## 7. Phase 5 — Admin & Polish

| ID | Task | Use Case | Status | Assignee | Ghi chú |
|----|------|----------|--------|----------|---------|
| T-022 | Admin Dashboard | FR-015 | `[ ]` | | Chart.js |
| T-023 | User Profile Page | FR-018 | `[ ]` | | Public profile |
| T-024 | SEO Slug | FR-019 | `[ ]` | | Friendly URL |
| T-025 | File Upload | FR-020 | `[ ]` | | Avatar, logo, CV |
| T-026 | Post Moderation | FR-021 | `[ ]` | | Admin workflow |
| T-027 | Soft Delete Restore | FR-022 | `[ ]` | | Trash page |

## 8. Ghi Chú Blockers

| Ngày | Task | Vấn đề | Hướng giải quyết |
|------|------|--------|-----------------|
| | | | |
