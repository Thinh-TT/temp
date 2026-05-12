# Workflow State Machine

## 1. Mục tiêu tài liệu
- Chuẩn hóa luồng trạng thái cho các thực thể có workflow.

## 2. Nguyên tắc chung
- Mọi thay đổi trạng thái phải được kiểm soát theo transition hợp lệ.
- Ghi log lịch sử trạng thái nếu nghiệp vụ yêu cầu audit.

## 3. Workflow: Job
### 3.1 Trạng thái
- `Pending`
- `Approved`
- `Hidden`
- `Rejected` (nếu áp dụng)

### 3.2 Transition hợp lệ
| From | To | Ai được phép |
|---|---|---|
| `Pending` | `Approved` | `Admin` |
| `Pending` | `Hidden` | `Admin` |
| `Approved` | `Hidden` | `Admin` |
| `Hidden` | `Approved` | `Admin` |

### 3.3 Rule
- Chỉ `Approved` mới hiển thị công khai.

## 4. Workflow: Application
### 4.1 Trạng thái
- `Pending`
- `Reviewed`
- `Interview`
- `Rejected`
- `Accepted`

### 4.2 Transition hợp lệ
| From | To | Ai được phép |
|---|---|---|
| `Pending` | `Reviewed` | `Employer` |
| `Reviewed` | `Interview` | `Employer` |
| `Interview` | `Accepted` | `Employer` |
| `Interview` | `Rejected` | `Employer` |
| `Reviewed` | `Rejected` | `Employer` |

### 4.3 Rule
- Không cho phép `Accepted` chuyển ngược về trạng thái trước.

## 5. Workflow: Post
### 5.1 Trạng thái
- `Draft`
- `Pending`
- `Published`
- `Hidden`

### 5.2 Transition hợp lệ
| From | To | Ai được phép |
|---|---|---|
| `Draft` | `Pending` | `Candidate`/`Employer` |
| `Pending` | `Published` | `Admin` |
| `Pending` | `Hidden` | `Admin` |
| `Published` | `Hidden` | `Admin` |
| `Hidden` | `Published` | `Admin` |

### 5.3 Rule
- `Draft` chỉ chủ bài viết nhìn thấy.

## 6. Pseudocode validate transition
```text
if (nextState not in allowedTransitions[currentState]) {
  throw InvalidStateTransition;
}
```

## 7. Audit log đề xuất
| Entity | EntityId | FromState | ToState | ChangedBy | ChangedAt | Note |
|---|---|---|---|---|---|---|
| | | | | | | |

## 8. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

