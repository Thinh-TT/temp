# API Specification

## 1. Mục tiêu tài liệu
- Chuẩn hóa contract API giữa backend và frontend.

## 2. Quy ước chung
- Base URL:
- API version: `v1`
- Content type: `application/json`
- Time format: `ISO 8601`
- Auth scheme: `Bearer JWT`

## 3. Chuẩn phản hồi
### 3.1 Success response
```json
{
  "success": true,
  "data": {},
  "message": "optional"
}
```

### 3.2 Error response
```json
{
  "success": false,
  "errorCode": "VALIDATION_ERROR",
  "message": "Human readable message",
  "details": []
}
```

## 4. Authentication API
### `POST /api/v1/auth/register`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```
- Error codes:
- `400`
- `409`

### `POST /api/v1/auth/login`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```
- Error codes:
- `400`
- `401`

## 5. Candidate API
### `GET /api/v1/candidates/me`
- Mô tả:
- Roles được phép gọi:
- Query params:
- Success response:
```json
{}
```

## 6. Employer API
### `POST /api/v1/jobs`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```

## 7. Job API
### `GET /api/v1/jobs`
- Mô tả:
- Query params: `keyword`, `categoryId`, `location`, `page`, `pageSize`
- Success response:
```json
{}
```

## 8. Application API
### `POST /api/v1/jobs/{jobId}/apply`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```

## 9. Community API
### `POST /api/v1/posts`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```

## 10. Admin API
### `PATCH /api/v1/admin/jobs/{jobId}/status`
- Mô tả:
- Roles được phép gọi:
- Request body:
```json
{}
```
- Success response:
```json
{}
```

## 11. Error code catalog
| Code | HTTP | Mô tả |
|---|---|---|
| VALIDATION_ERROR | 400 | |
| UNAUTHORIZED | 401 | |
| FORBIDDEN | 403 | |
| NOT_FOUND | 404 | |
| CONFLICT | 409 | |
| INTERNAL_ERROR | 500 | |

## 12. Changelog
| Version | Ngày | Người cập nhật | Nội dung |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Khởi tạo khung |

