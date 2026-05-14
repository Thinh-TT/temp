# Tài Liệu Dự Án — Nền Tảng Tuyển Dụng & Cộng Đồng Nghề Nghiệp

## Cấu Trúc Thư Mục

```
docs/
├── README.md
├── 01-project/          # Tổng quan & kế hoạch dự án
├── 02-architecture/     # Kiến trúc hệ thống & thiết kế kỹ thuật
├── 03-functional/       # Yêu cầu chức năng & use case
├── 04-ui/               # Thiết kế giao diện & trải nghiệm người dùng
├── 05-tasks/            # Theo dõi công việc & task board
└── 06-logs/             # Nhật ký phát triển & quyết định kỹ thuật
```

## Danh Mục Tài Liệu

### 01 — Tổng Quan & Kế Hoạch Dự Án

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-ideas-and-scope.md](01-project/01-ideas-and-scope.md) | Ý tưởng sản phẩm, phạm vi MVP, công nghệ sử dụng |
| 2 | [02-non-functional-requirements.md](01-project/02-non-functional-requirements.md) | Yêu cầu phi chức năng: hiệu năng, bảo mật, khả dụng |
| 3 | [03-agent-instructions.md](01-project/03-agent-instructions.md) | Quy tắc code, naming convention, Clean Architecture cho AI agent |

### 02 — Kiến Trúc & Thiết Kế Kỹ Thuật

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-database-description.md](02-architecture/01-database-description.md) | Mô tả chi tiết các bảng trong cơ sở dữ liệu |
| 2 | [02-erd-and-db-rules.md](02-architecture/02-erd-and-db-rules.md) | Sơ đồ quan hệ thực thể & ràng buộc dữ liệu |
| 3 | [03-api-spec.md](02-architecture/03-api-spec.md) | Đặc tả API: endpoint, request/response, phân trang |
| 4 | [04-workflow-state-machine.md](02-architecture/04-workflow-state-machine.md) | Máy trạng thái cho Job, Application, Post |

### 03 — Yêu Cầu Chức Năng

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-functional-requirements.md](03-functional/01-functional-requirements.md) | Danh sách use case, acceptance criteria, business rules |

### 04 — Thiết Kế Giao Diện

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-ui-sitemap-and-wireframe.md](04-ui/01-ui-sitemap-and-wireframe.md) | Sitemap, danh sách màn hình, luồng điều hướng |

### 05 — Theo Dõi Công Việc

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-task-board.md](05-tasks/01-task-board.md) | Bảng theo dõi task theo phase & module |

### 06 — Nhật Ký Phát Triển

| # | Tài liệu | Mô tả |
|---|----------|-------|
| 1 | [01-dev-log.md](06-logs/01-dev-log.md) | Nhật ký quyết định kỹ thuật & vấn đề phát sinh |

## Quy Ước Đặt Tên File

- Tiền tố số thứ tự: `01-`, `02-`, ... để giữ thứ tự trong từng nhóm.
- Tên file ngắn gọn, dùng tiếng Anh, phân cách bằng dấu gạch ngang.
- File markdown (`.md`), mã hóa UTF-8.

## Liên Kết Nhanh Theo Module

| Module | Tài liệu liên quan |
|--------|-------------------|
| Auth & Phân quyền | 03-functional/01, 02-architecture/01, 02-architecture/02 |
| Quản lý Job | 03-functional/01, 02-architecture/03, 02-architecture/04 |
| Ứng tuyển | 03-functional/01, 02-architecture/01, 02-architecture/04 |
| Cộng đồng | 03-functional/01, 02-architecture/01, 02-architecture/03 |
| Admin Dashboard | 03-functional/01, 04-ui/01 |
| UI/UX | 04-ui/01 |
| Dev Conventions | 01-project/03-agent-instructions.md |
