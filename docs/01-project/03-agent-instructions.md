# Agent Instructions

Hướng dẫn cho AI agent khi làm việc trên dự án. Tuân thủ các quy tắc dưới đây để đảm bảo code nhất quán và đúng kiến trúc.

---

## General Rules

- Use ASP.NET Core 8
- Use Entity Framework Core
- Use Repository + UnitOfWork pattern
- Use MediatR for CQRS
- Use FluentValidation
- Use AutoMapper only when mapping is complex
- Prefer async/await everywhere
- Avoid business logic inside controllers

---

## Kiến Trúc Clean Architecture

Dự án tổ chức theo Clean Architecture với 4 tầng:

```
/src
  /Domain          # Entities, Enums, Interfaces (không phụ thuộc tầng nào)
  /Application     # Use Cases, Commands, Queries, DTOs, Validators
  /Infrastructure  # EF Core, Repositories, External Services
  /WebAPI          # Controllers, Middleware, Filters, Razor Pages
```

### Quy tắc phụ thuộc

- `Domain` không tham chiếu đến bất kỳ tầng nào.
- `Application` chỉ tham chiếu đến `Domain`.
- `Infrastructure` tham chiếu đến `Application` và `Domain`.
- `WebAPI` tham chiếu đến `Infrastructure`, `Application`, và `Domain`.

---

## Naming Conventions

### Classes

| Loại | Pattern | Ví dụ |
|------|---------|-------|
| Service | `{Entity}Service` | `UserService` |
| Repository | `{Entity}Repository` | `UserRepository` |
| DTO (Request) | `{Action}{Entity}Request` | `CreateUserRequest` |
| DTO (Response) | `{Entity}{Action}Response` | `UserDetailResponse` |
| Command | `{Action}{Entity}Command` | `CreateUserCommand` |
| Query | `{Action}{Entity}Query` | `GetUserByIdQuery` |
| Validator | `{Entity}{Action}Validator` | `CreateUserValidator` |
| Controller | `{Entity}Controller` | `UsersController` |

### Interfaces

Always prefix with `I`:

- `IUserRepository`
- `IUnitOfWork`
- `IJobService`

### File naming

- One class per file.
- File name matches class name: `UserService.cs`, `IUserRepository.cs`.

---

## Controller Rules

Controllers should:
- Only receive HTTP request
- Call MediatR `ISender` or service interface
- Return `IActionResult` / `ActionResult<T>`
- Map DTOs if needed (delegate to AutoMapper when complex)

Controllers must NOT:
- Contain business logic
- Access DbContext directly
- Call repositories directly

```csharp
// ĐÚNG
[HttpPost]
public async Task<IActionResult> Create(CreateJobRequest request)
{
    var command = _mapper.Map<CreateJobCommand>(request);
    var result = await _sender.Send(command);
    return result.IsSuccess ? Ok(result) : BadRequest(result);
}
```

---

## Entity Rules

Entities must:
- Reside in `Domain` layer
- Contain domain logic only (validation, business invariants)
- Avoid infrastructure dependencies (no EF attributes if possible, use Fluent configuration)
- Use private setters for properties that should not be mutated externally

```csharp
// ĐÚNG
public class Job
{
    public Guid Id { get; private set; }
    public string Title { get; private set; }
    public JobStatus Status { get; private set; }

    public void Approve()
    {
        if (Status != JobStatus.Pending)
            throw new DomainException("Only pending jobs can be approved");
        Status = JobStatus.Approved;
    }
}
```

---

## Database Rules

- Use `Guid` for primary keys.
- Use UTC for all timestamps.
- Every entity must include:
  - `CreatedAt` (`datetime2(7)` — mặc định `GETUTCDATE()`)
  - `UpdatedAt` (`datetime2(7)` — cập nhật tự động hoặc qua interceptor)
- Use soft delete (`IsDeleted` + `DeletedAt`) cho dữ liệu nghiệp vụ.
- Configure relationships via Fluent API (`IEntityTypeConfiguration<T>`), không dùng Data Annotations.
- Tạo index cho các cột thường xuyên query (FK, filter, sort).

---

## API Response Format

### Success

```json
{
  "success": true,
  "data": {}
}
```

### Error

```json
{
  "success": false,
  "message": "Validation failed"
}
```

### Paginated list

```json
{
  "success": true,
  "data": {
    "items": [],
    "page": 1,
    "pageSize": 10,
    "totalItems": 0,
    "totalPages": 0
  }
}
```

---

## Logging

Use **Serilog** cho structured logging.

Các sự kiện bắt buộc phải log:
- Exceptions (qua middleware toàn cục)
- Important business actions (tạo job, duyệt job, thay đổi trạng thái application)
- Authentication events (đăng nhập thành công, thất bại, khóa tài khoản)
- Admin moderation actions (duyệt/ẩn/xóa job, post)

Cấp độ log:
- `Information`: business action quan trọng
- `Warning`: validation failure, attempt truy cập trái phép
- `Error`: exception không mong đợi

---

## Important — Quy Trình Làm Việc

Trước khi implement bất kỳ tính năng nào:

1. **Đọc tài liệu liên quan** — Kiểm tra `03-functional/` để hiểu use case và `02-architecture/` để biết API contract, DB schema, workflow.
2. **Kiểm tra convention hiện có** — Đọc code tương tự đã implement để tái sử dụng pattern.
3. **Tái sử dụng pattern có sẵn** — Không sáng tạo pattern mới nếu đã có pattern giải quyết cùng vấn đề.
4. **Cập nhật dev log** — Sau khi hoàn thành, ghi entry vào `06-logs/01-dev-log.md` nếu có quyết định kỹ thuật quan trọng hoặc vấn đề phát sinh đáng ghi nhận.

---

## Commit Convention

```
<type>: <short description>

Type:
- feat     — tính năng mới
- fix      — sửa lỗi
- refactor — tái cấu trúc code
- docs     — cập nhật tài liệu
- style    — format, thiếu dấu cách (không thay đổi logic)
- test     — thêm/sửa test
- chore    — build, config, dependencies
```
