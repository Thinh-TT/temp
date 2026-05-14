# Workflow State Machine

## 1. Mục tiêu tài liệu

- Chuẩn hóa luồng trạng thái cho các thực thể có workflow: `Jobs`, `Applications`, `Posts`.
- Đồng bộ với `2-functional-requirements.md`, `4-erd-and-db-rules.md` và `5-api-spec.md`.
- Làm tài liệu tham khảo cho dev backend khi implement validate transition, dev frontend khi render action button theo trạng thái, và tester khi viết test case cho luồng trạng thái.

## 2. Nguyên tắc chung

- Mọi thay đổi trạng thái phải được kiểm soát theo transition hợp lệ được định nghĩa trong tài liệu này.
- Transition không hợp lệ trả về HTTP `409 INVALID_STATE_TRANSITION`.
- Actor thực hiện transition phải có quyền tương ứng (`Candidate`, `Employer`, `Admin`).
- Ghi log lịch sử trạng thái nếu nghiệp vụ yêu cầu audit (job approval, post moderation, application status change).
- Các trường `Status` được lưu dưới dạng `tinyint` trong database, mapping với enum string ở tầng API.

## 3. Workflow: Job

### 3.1 Sơ đồ trạng thái

```
        ┌─────────────────────────────────────┐
        │                                     │
        ▼                                     │
    ┌─────────┐    Admin     ┌──────────┐     │
    │ Pending │ ──────────►  │ Approved │     │
    │  (0)    │              │   (1)    │     │
    └─────────┘              └──────────┘     │
         │                       │    ▲       │
         │                       │    │       │
         │ Admin                 │ Admin      │ Admin
         │                       │    │       │
         ▼                       ▼    │       │
    ┌─────────┐    Admin     ┌──────────┐     │
    │ Hidden  │ ◄──────────  │ Hidden   │     │
    │  (2)    │              │  (2)     │     │
    └─────────┘              └──────────┘     │
         │                       │            │
         │ Admin                 │ Admin      │
         │                       │            │
         └───────┬───────────────┘            │
                 │                            │
                 │ Admin (restore)            │
                 ▼                            │
            [IsDeleted = false] ──────────────┘
```

### 3.2 Bảng trạng thái

| Value | Enum       | Mô tả                          |
| ----- | ---------- | ------------------------------ |
| `0`   | `Pending`  | Chờ admin duyệt                |
| `1`   | `Approved` | Đã duyệt, hiển thị công khai   |
| `2`   | `Hidden`   | Bị ẩn khỏi danh sách công khai |

### 3.3 Transition hợp lệ

| From       | To         | Actor    | Điều kiện                                                                    | Ghi chú                                        |
| ---------- | ---------- | -------- | ---------------------------------------------------------------------------- | ---------------------------------------------- |
| `Pending`  | `Approved` | `Admin`  | Job chưa bị xóa mềm                                                          | API: `PATCH /api/v1/admin/jobs/{jobId}/status` |
| `Pending`  | `Hidden`   | `Admin`  | Job chưa bị xóa mềm                                                          | Ẩn job vi phạm                                 |
| `Approved` | `Hidden`   | `Admin`  | Job chưa bị xóa mềm                                                          | Ẩn tạm thời                                    |
| `Hidden`   | `Approved` | `Admin`  | Job chưa bị xóa mềm                                                          | Mở hiển thị lại                                |
| `Approved` | `Pending`  | Hệ thống | Employer sửa nội dung quan trọng (title, description, requirement, benefits) | Tự động reset về chờ duyệt                     |
| `Hidden`   | `Pending`  | Hệ thống | Employer sửa nội dung quan trọng khi job đang bị ẩn                          | Tự động reset về chờ duyệt                     |
| `Pending`  | `Pending`  | Hệ thống | Employer sửa job đang chờ duyệt                                              | Giữ nguyên trạng thái, cập nhật `UpdatedAt`    |

### 3.4 Transition bị cấm

| From       | To                    | Lý do                                                                           |
| ---------- | --------------------- | ------------------------------------------------------------------------------- |
| `Approved` | `Pending` (bởi Admin) | Admin không reset về pending, việc này chỉ do hệ thống tự động khi employer sửa |
| `Hidden`   | `Pending` (bởi Admin) | Admin không reset về pending                                                    |
| `Pending`  | `Pending` (bởi Admin) | Admin phải chọn Approve hoặc Hide, không được giữ nguyên khi duyệt              |

### 3.5 Quy tắc hiển thị

| Trạng thái | `IsDeleted = 0`                                 | `IsDeleted = 1` |
| ---------- | ----------------------------------------------- | --------------- |
| `Pending`  | Chỉ hiển thị cho employer sở hữu và admin       | Không hiển thị  |
| `Approved` | Hiển thị công khai (guest, candidate, employer) | Không hiển thị  |
| `Hidden`   | Chỉ hiển thị cho employer sở hữu và admin       | Không hiển thị  |

### 3.6 Tương tác với Soft Delete

```
Job bình thường (IsDeleted=0)
    │
    ├── Employer xóa ──► IsDeleted = 1 (ẩn hoàn toàn)
    │                     Status giữ nguyên
    │
    └── Admin xóa ────► IsDeleted = 1 (ẩn hoàn toàn)
                         Status giữ nguyên

Job đã xóa mềm (IsDeleted=1)
    │
    └── Admin restore ──► IsDeleted = 0
                          Status = Hidden (an toàn, không tự động public)
```

> **Chốt quy tắc restore**: Sau khi restore, entity luôn được gán `Status = Hidden`, bất kể trạng thái trước khi bị xóa mềm là gì. Đây là quy tắc safety-first của state machine, tránh trường hợp content từng bị xóa vô tình được public trở lại. DB rules mục 10.2 trong `4-erd-and-db-rules.md` cần được đọc cùng với quy tắc này: "vẫn phải tuân theo Status hiện tại" nghĩa là sau restore entity sẽ bắt đầu từ `Hidden` và admin có thể chuyển tiếp sang `Approved`/`Published` nếu cần.

### 3.7 Audit log đề xuất cho Job

| Cột          | Mô tả                          |
| ------------ | ------------------------------ |
| `JobId`      | FK tới job bị thay đổi         |
| `FromStatus` | Trạng thái cũ                  |
| `ToStatus`   | Trạng thái mới                 |
| `ChangedBy`  | UserId của admin hoặc employer |
| `ChangedAt`  | Thời điểm thay đổi (UTC)       |
| `Reason`     | Lý do thay đổi (tùy chọn)      |

## 4. Workflow: Application

### 4.1 Sơ đồ trạng thái

```
                    ┌──────────┐
                    │ Pending  │
                    │   (0)    │
                    └────┬─────┘
                         │
                         │ Employer
                         ▼
                    ┌──────────┐
              ┌─────│ Reviewed │─────┐
              │     │   (1)    │     │
              │     └────┬─────┘     │
              │          │           │
              │          │ Employer  │ Employer
              │          ▼           │
              │     ┌──────────┐     │
              │     │ Interview│     │
              │     │   (2)    │     │
              │     └────┬─────┘     │
              │          │           │
              │          │ Employer  │
              │     ┌────┴─────┐     │
              │     ▼          ▼     ▼
              │ ┌────────┐ ┌────────────┐
              │ │Accepted│ │ Rejected   │
              │ │  (4)   │ │   (3)      │
              │ └────────┘ └────────────┘
              │     ▲          ▲
              │     │ Final    │ Final
              │     │ (không   │ (không
              │     │ thể đổi) │ thể đổi)
              │     │          │
              └─────┴──────────┘
               Từ Reviewed có thể
               reject trực tiếp
```

### 4.2 Bảng trạng thái

| Value | Enum        | Mô tả                        |
| ----- | ----------- | ---------------------------- |
| `0`   | `Pending`   | Đơn mới nộp, chưa được xem   |
| `1`   | `Reviewed`  | Đã được employer xem         |
| `2`   | `Interview` | Được mời phỏng vấn           |
| `3`   | `Rejected`  | Bị từ chối (trạng thái cuối) |
| `4`   | `Accepted`  | Được nhận (trạng thái cuối)  |

### 4.3 Transition hợp lệ

| From        | To          | Actor      | Điều kiện           | API                                      |
| ----------- | ----------- | ---------- | ------------------- | ---------------------------------------- |
| `Pending`   | `Reviewed`  | `Employer` | Employer sở hữu job | `PATCH /api/v1/applications/{id}/status` |
| `Reviewed`  | `Interview` | `Employer` | Employer sở hữu job | `PATCH /api/v1/applications/{id}/status` |
| `Reviewed`  | `Rejected`  | `Employer` | Employer sở hữu job | `PATCH /api/v1/applications/{id}/status` |
| `Interview` | `Accepted`  | `Employer` | Employer sở hữu job | `PATCH /api/v1/applications/{id}/status` |
| `Interview` | `Rejected`  | `Employer` | Employer sở hữu job | `PATCH /api/v1/applications/{id}/status` |

### 4.4 Transition bị cấm

| From       | To          | Lý do                               |
| ---------- | ----------- | ----------------------------------- |
| `Accepted` | Bất kỳ      | Trạng thái cuối, không thể thay đổi |
| `Rejected` | Bất kỳ      | Trạng thái cuối, không thể thay đổi |
| `Pending`  | `Interview` | Phải qua `Reviewed` trước           |
| `Pending`  | `Accepted`  | Phải qua `Reviewed` và `Interview`  |
| `Pending`  | `Rejected`  | Phải qua `Reviewed` trước           |
| `Reviewed` | `Accepted`  | Phải qua `Interview` trước          |

### 4.5 Side effects khi chuyển trạng thái

| Transition          | Side effect                                        |
| ------------------- | -------------------------------------------------- |
| Bất kỳ → `Reviewed` | Cập nhật `ReviewedAt = UTC now`                    |
| Bất kỳ → bất kỳ     | Cho phép cập nhật `Note`                           |
| Mọi transition      | Cập nhật `Applications.UpdatedAt` (nếu có cột này) |

### 4.6 Audit log đề xuất cho Application

| Cột             | Mô tả                    |
| --------------- | ------------------------ |
| `ApplicationId` | FK tới application       |
| `FromStatus`    | Trạng thái cũ            |
| `ToStatus`      | Trạng thái mới           |
| `ChangedBy`     | UserId của employer      |
| `ChangedAt`     | Thời điểm thay đổi (UTC) |
| `Note`          | Ghi chú từ employer      |

## 5. Workflow: Post (Community)

### 5.1 Sơ đồ trạng thái

```
    ┌─────────┐   Author    ┌──────────┐   Admin    ┌───────────┐
    │ Draft   │ ──────────► │ Pending  │ ──────────►│ Published │
    │  (0)    │             │   (1)    │            │    (2)    │
    └─────────┘             └──────────┘            └───────────┘
         ▲                       │                       │
         │                       │ Admin                 │ Admin
         │ Author                ▼                       ▼
         │ (save)           ┌──────────┐            ┌──────────┐
         └──────────────────│ Hidden   │◄───────────│ Hidden   │
                            │   (3)    │            │   (3)    │
                            └──────────┘            └──────────┘
                                 ▲                       │
                                 │ Admin                 │ Admin
                                 │ (restore              │
                                 │  + publish)           │
                                 └───────────────────────┘
```

### 5.2 Bảng trạng thái

| Value | Enum        | Mô tả                  | Ai nhìn thấy         |
| ----- | ----------- | ---------------------- | -------------------- |
| `0`   | `Draft`     | Bản nháp               | Chỉ tác giả          |
| `1`   | `Pending`   | Chờ admin duyệt        | Tác giả và admin     |
| `2`   | `Published` | Đã xuất bản, công khai | Tất cả (kể cả guest) |
| `3`   | `Hidden`    | Bị ẩn                  | Tác giả và admin     |

### 5.3 Transition hợp lệ

| From        | To          | Actor                    | Điều kiện                             | API                                     |
| ----------- | ----------- | ------------------------ | ------------------------------------- | --------------------------------------- |
| `Draft`     | `Pending`   | `Candidate` / `Employer` | Là tác giả của bài                    | `POST /api/v1/posts/{id}/submit`        |
| `Draft`     | `Draft`     | `Candidate` / `Employer` | Là tác giả, lưu nháp tiếp             | `PUT /api/v1/posts/{id}`                |
| `Pending`   | `Published` | `Admin`                  | Bài chưa bị xóa mềm                   | `PATCH /api/v1/admin/posts/{id}/status` |
| `Pending`   | `Hidden`    | `Admin`                  | Bài chưa bị xóa mềm                   | `PATCH /api/v1/admin/posts/{id}/status` |
| `Published` | `Hidden`    | `Admin`                  | Ẩn bài vi phạm                        | `PATCH /api/v1/admin/posts/{id}/status` |
| `Hidden`    | `Published` | `Admin`                  | Mở hiển thị lại                       | `PATCH /api/v1/admin/posts/{id}/status` |
| `Hidden`    | `Pending`   | `Candidate` / `Employer` | Là tác giả, gửi duyệt lại sau khi sửa | `POST /api/v1/posts/{id}/submit`        |

### 5.4 Transition bị cấm

| From        | To          | Lý do                                     |
| ----------- | ----------- | ----------------------------------------- |
| `Draft`     | `Published` | Phải qua bước gửi duyệt (`Pending`)       |
| `Draft`     | `Hidden`    | Không có ý nghĩa với bản nháp             |
| `Published` | `Draft`     | Không thể quay về nháp sau khi đã publish |
| `Published` | `Pending`   | Không thể gửi duyệt lại bài đã publish    |

### 5.5 Quy tắc hiển thị Post

| Trạng thái  | `IsDeleted = 0`                                      | `IsDeleted = 1` |
| ----------- | ---------------------------------------------------- | --------------- |
| `Draft`     | Chỉ tác giả                                          | Không hiển thị  |
| `Pending`   | Tác giả + Admin (trong màn hình kiểm duyệt)          | Không hiển thị  |
| `Published` | Công khai toàn bộ (feed, detail)                     | Không hiển thị  |
| `Hidden`    | Tác giả + Admin; không xuất hiện trên feed công khai | Không hiển thị  |

### 5.6 MVP vs Sau MVP

| Giai đoạn   | Hành vi                                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **MVP**     | Bỏ qua moderation workflow. Khi tạo bài, mặc định `Status = Published`. Không dùng `Draft` và `Pending`. Người dùng tạo/xóa trực tiếp. |
| **Sau MVP** | Bật đầy đủ moderation workflow. Bài mới mặc định `Draft`. Tác giả chủ động gửi duyệt. Admin duyệt trước khi public.                    |

### 5.7 Tương tác với Soft Delete

```
Post bình thường (IsDeleted=0)
    │
    ├── Tác giả xóa ────► IsDeleted = 1
    │                      Status giữ nguyên
    │
    └── Admin xóa ──────► IsDeleted = 1
                           Status giữ nguyên

Post đã xóa mềm (IsDeleted=1)
    │
    └── Admin restore ──► IsDeleted = 0
                          Status = Hidden (an toàn)
```

> **Quy tắc restore cho Post**: Giống Job — restore luôn ép `Status = Hidden`. Lý do và cách đọc chung: xem chú thích tại mục 3.6.

### 5.8 Audit log đề xuất cho Post

| Cột          | Mô tả                                      |
| ------------ | ------------------------------------------ |
| `PostId`     | FK tới post                                |
| `FromStatus` | Trạng thái cũ                              |
| `ToStatus`   | Trạng thái mới                             |
| `ChangedBy`  | UserId của tác giả hoặc admin              |
| `ChangedAt`  | Thời điểm thay đổi (UTC)                   |
| `Reason`     | Lý do (đặc biệt quan trọng với admin hide) |

## 6. Pseudocode validate transition

### 6.1 Cấu trúc dữ liệu transition map

```csharp
// Job transitions
var jobTransitions = new Dictionary<JobStatus, HashSet<JobStatus>>
{
    { JobStatus.Pending,  new() { JobStatus.Approved, JobStatus.Hidden } },
    { JobStatus.Approved, new() { JobStatus.Hidden } },
    { JobStatus.Hidden,   new() { JobStatus.Approved } }
};

// Application transitions
var applicationTransitions = new Dictionary<ApplicationStatus, HashSet<ApplicationStatus>>
{
    { ApplicationStatus.Pending,   new() { ApplicationStatus.Reviewed } },
    { ApplicationStatus.Reviewed,  new() { ApplicationStatus.Interview, ApplicationStatus.Rejected } },
    { ApplicationStatus.Interview, new() { ApplicationStatus.Accepted, ApplicationStatus.Rejected } }
    // Accepted và Rejected không có entry -> không có transition đi tiếp
};

// Post transitions (Sau MVP)
var postTransitions = new Dictionary<PostStatus, HashSet<PostStatus>>
{
    { PostStatus.Draft,     new() { PostStatus.Pending } },
    { PostStatus.Pending,   new() { PostStatus.Published, PostStatus.Hidden } },
    { PostStatus.Published, new() { PostStatus.Hidden } },
    { PostStatus.Hidden,    new() { PostStatus.Published, PostStatus.Pending } }
};
```

### 6.2 Hàm validate chung

```csharp
public void ValidateTransition<TStatus>(
    TStatus currentStatus,
    TStatus nextStatus,
    Dictionary<TStatus, HashSet<TStatus>> allowedTransitions)
{
    if (!allowedTransitions.TryGetValue(currentStatus, out var allowedNext))
    {
        throw new InvalidStateTransitionException(
            $"Trạng thái '{currentStatus}' là trạng thái cuối, " +
            $"không thể chuyển sang trạng thái khác.");
    }

    if (!allowedNext.Contains(nextStatus))
    {
        throw new InvalidStateTransitionException(
            $"Không thể chuyển từ '{currentStatus}' sang '{nextStatus}'. " +
            $"Các trạng thái hợp lệ tiếp theo: {string.Join(", ", allowedNext)}.");
    }
}
```

### 6.3 Tích hợp với endpoint

```text
PATCH /api/v1/admin/jobs/{jobId}/status
  └── Controller
       └── Service.UpdateJobStatus(jobId, newStatus, adminUserId)
            ├── Load job (kiểm tra tồn tại, IsDeleted)
            ├── ValidateTransition(job.Status, newStatus, jobTransitions)
            ├── Cập nhật job.Status = newStatus
            ├── Ghi audit log
            └── Save changes
```

## 7. Ma trận tổng hợp quyền theo Actor

| Actor       | Job: Pending→Approved     | Job: Approve→Hidden | App: Pending→Reviewed | App: Interview→Accepted | Post: Draft→Pending | Post: Pending→Published |
| ----------- | ------------------------- | ------------------- | --------------------- | ----------------------- | ------------------- | ----------------------- |
| `Candidate` | ❌                        | ❌                  | ❌                    | ❌                      | ✅ (bài của mình)   | ❌                      |
| `Employer`  | ❌                        | ❌                  | ✅ (job của mình)     | ✅ (job của mình)       | ✅ (bài của mình)   | ❌                      |
| `Admin`     | ✅                        | ✅                  | ❌                    | ❌                      | ❌                  | ✅                      |
| Hệ thống    | ❌ (chỉ reset về Pending) | ❌                  | ❌                    | ❌                      | ❌                  | ❌                      |

## 8. Quy tắc tự động (System-triggered transitions)

| Trigger                 | Entity         | Transition               | Điều kiện kích hoạt                                                                    |
| ----------------------- | -------------- | ------------------------ | -------------------------------------------------------------------------------------- |
| Employer sửa job        | `Jobs`         | `Approved` → `Pending`   | Sửa title, description, requirement, hoặc benefits                                     |
| Employer sửa job        | `Jobs`         | `Hidden` → `Pending`     | Sửa title, description, requirement, hoặc benefits                                     |
| Employer sửa job        | `Jobs`         | `Pending` → `Pending`    | Sửa các trường khác (salary, location, etc.), giữ nguyên Pending, cập nhật `UpdatedAt` |
| Candidate apply         | `Applications` | (khởi tạo) = `Pending`   | Luôn khởi tạo ở `Pending`                                                              |
| User tạo post (MVP)     | `Posts`        | (khởi tạo) = `Published` | MVP không có moderation                                                                |
| User tạo post (Sau MVP) | `Posts`        | (khởi tạo) = `Draft`     | Khi moderation workflow được bật                                                       |

## 9. Tổng quan API endpoint liên quan đến state machine

| Endpoint                                            | Entity      | Actor              | Transition                                                    |
| --------------------------------------------------- | ----------- | ------------------ | ------------------------------------------------------------- |
| `POST /api/v1/jobs`                                 | Job         | Employer           | Khởi tạo → `Pending`                                          |
| `PUT /api/v1/jobs/{jobId}`                          | Job         | Employer           | `Approved`/`Hidden` → `Pending` (nếu sửa nội dung quan trọng) |
| `PATCH /api/v1/admin/jobs/{jobId}/status`           | Job         | Admin              | `Pending` → `Approved`/`Hidden`; `Approved` ↔ `Hidden`        |
| `POST /api/v1/jobs/{jobId}/apply`                   | Application | Candidate          | Khởi tạo → `Pending`                                          |
| `PATCH /api/v1/applications/{applicationId}/status` | Application | Employer           | Theo application transition map                               |
| `POST /api/v1/posts`                                | Post        | Candidate/Employer | Khởi tạo → `Published` (MVP) hoặc `Draft` (Sau MVP)           |
| `POST /api/v1/posts/{postId}/submit`                | Post        | Candidate/Employer | `Draft` → `Pending` (Sau MVP)                                 |
| `PATCH /api/v1/admin/posts/{postId}/status`         | Post        | Admin              | `Pending` → `Published`/`Hidden`; `Published` ↔ `Hidden`      |

## 10. Changelog

| Version | Ngày       | Người cập nhật / Tên Model | Nội dung                                                                                                                                       |
| ------- | ---------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.1     | YYYY-MM-DD |                            | Khởi tạo khung                                                                                                                                 |
| 1.0     | 2026-05-14 | DeepSeek V4 Pro            | Hoàn thiện state machine cho Job, Application, Post; bổ sung sơ đồ, transition table, pseudocode, audit log, ma trận quyền, và quy tắc tự động |
