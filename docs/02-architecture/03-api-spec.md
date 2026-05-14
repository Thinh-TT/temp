# API Specification

## 1. Mục tiêu tài liệu

- Chuẩn hóa contract API giữa backend, frontend và tester.
- Đồng bộ với `2-functional-requirements.md` và `4-erd-and-db-rules.md`.
- Bao phủ toàn bộ endpoint chính cho các use case `FR-001` đến `FR-022`.

## 2. Phạm vi API

### 2.1 API trong MVP (`Must`)

- Authentication
- Candidate profile
- Employer profile
- Job CRUD, job list, job detail
- Apply job, save job
- Employer application management
- Community post CRUD, feed, interaction
- Admin job moderation
- Admin dashboard

### 2.2 API sau MVP (`Should`)

- Reply comment 1 cấp
- User profile page
- Upload avatar/logo/thumbnail
- Post moderation workflow
- Restore soft-deleted data
- SEO slug routing hoàn chỉnh

## 3. Quy ước chung

- Base URL đề xuất: `/api/v1`
- Content type:
  - Request: `application/json`
  - Response: `application/json`
- Auth scheme: `Bearer JWT`
- Time format: `ISO 8601` UTC, ví dụ: `2026-05-13T08:30:00Z`
- String enum trong request/response chỉ dùng khi đã chốt rõ; trong database hệ thống vẫn có thể map về `tinyint`.
- API list bắt buộc có phân trang.
- **Cơ chế serve file**: Avatar, logo, thumbnail là file public — được serve trực tiếp từ thư mục tĩnh (VD: `/static/uploads/images/...`). CV là file riêng tư — được serve qua endpoint được bảo vệ `GET /api/v1/files/cv/{fileId}`, backend kiểm tra quyền truy cập (chính chủ, employer của job đã apply, admin) trước khi trả file. URL tuyệt đối của CV không được lộ ra ngoài response API công khai; các response chứa CV trả về `fileId` để frontend gọi qua endpoint protected.

### 3.1 Header chuẩn

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

### 3.2 Quy ước pagination

- Query params chung:
  - `page`: số trang, bắt đầu từ `1`
  - `pageSize`: số bản ghi trên trang
- Response list chuẩn:

```json
{
  "success": true,
  "data": {
    "items": [],
    "page": 1,
    "pageSize": 10,
    "totalItems": 0,
    "totalPages": 0
  },
  "message": "optional"
}
```

### 3.3 Chuẩn phản hồi thành công

```json
{
  "success": true,
  "data": {},
  "message": "optional"
}
```

### 3.4 Chuẩn phản hồi lỗi

```json
{
  "success": false,
  "errorCode": "VALIDATION_ERROR",
  "message": "Human readable message",
  "details": [
    {
      "field": "email",
      "message": "Email is required"
    }
  ]
}
```

## 4. Kiểu dữ liệu dùng chung

### 4.1 `JobStatus`

- `Pending`
- `Approved`
- `Hidden`

### 4.2 `ApplicationStatus`

- `Pending`
- `Reviewed`
- `Interview`
- `Rejected`
- `Accepted`

### 4.3 `PostStatus`

- `Draft`
- `Pending`
- `Published`
- `Hidden`

### 4.4 `EmploymentType`

- `FullTime`
- `PartTime`

### 4.5 `UserRole`

- `Candidate`
- `Employer`
- `Admin`

## 5. Authentication API

### `POST /api/v1/auth/register`

- Mô tả: Đăng ký tài khoản `Candidate` hoặc `Employer`.
- Roles được phép gọi: `Guest`
- Auth: Không yêu cầu
- Request body:

```json
{
  "userName": "john.dev",
  "email": "john@example.com",
  "password": "StrongPassword123!",
  "confirmPassword": "StrongPassword123!",
  "role": "Candidate"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
    "role": "Candidate"
  },
  "message": "Register successful"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `409 USERNAME_ALREADY_EXISTS`
  - `409 EMAIL_ALREADY_EXISTS`

### `POST /api/v1/auth/login`

- Mô tả: Đăng nhập bằng email hoặc username và mật khẩu.
- Roles được phép gọi: `Guest`
- Auth: Không yêu cầu
- Request body:

```json
{
  "login": "john@example.com",
  "password": "StrongPassword123!"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt-access-token",
    "expiresAt": "2026-05-13T10:30:00Z",
    "user": {
      "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
      "userName": "john.dev",
      "email": "john@example.com",
      "role": "Candidate"
    }
  }
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 INVALID_CREDENTIALS`
  - `403 ACCOUNT_INACTIVE`

### `POST /api/v1/auth/logout`

- Mô tả: Đăng xuất phiên hiện tại.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": null,
  "message": "Logout successful"
}
```

- Error codes:
  - `401 UNAUTHORIZED`

## 6. Candidate API

### `GET /api/v1/candidates/me`

- Mô tả: Lấy hồ sơ candidate hiện tại.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Query params: Không có
- Success response:

```json
{
  "success": true,
  "data": {
    "candidateId": 12,
    "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
    "fullName": "Nguyen Van A",
    "avatarUrl": "/uploads/avatars/a.png",
    "dateOfBirth": "2002-08-15",
    "gender": 1,
    "address": "Ho Chi Minh City",
    "bio": "Backend developer",
    "cvFileId": "cv_candidate_12",
    "experienceYears": 2,
    "skills": [
      {
        "skillId": 1,
        "name": "ASP.NET Core"
      }
    ]
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 PROFILE_NOT_FOUND`

### `PUT /api/v1/candidates/me`

- Mô tả: Cập nhật hồ sơ candidate hiện tại.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Request body:

```json
{
  "fullName": "Nguyen Van A",
  "avatarUrl": "/uploads/avatars/a.png",
  "dateOfBirth": "2002-08-15",
  "gender": 1,
  "address": "Ho Chi Minh City",
  "bio": "Backend developer",
  "cvFileId": "cv_candidate_12",
  "experienceYears": 2,
  "skillIds": [1, 3, 5]
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "candidateId": 12
  },
  "message": "Profile updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `400 INVALID_FILE_TYPE`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 SKILL_NOT_FOUND`

### `GET /api/v1/candidates/me/saved-jobs`

- Mô tả: Lấy danh sách job đã lưu của candidate hiện tại.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Query params:
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "savedJobId": 3,
        "jobId": 101,
        "jobTitle": "Backend Developer",
        "jobSlug": "backend-developer-hcm",
        "companyName": "ABC Tech",
        "location": "Ho Chi Minh City",
        "savedAt": "2026-05-13T08:30:00Z"
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `GET /api/v1/candidates/me/activity`

- Mô tả: Lấy hoạt động của candidate hiện tại cho trang profile sau MVP.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Phase: `Should`
- Query params:
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "appliedJobs": [
      {
        "applicationId": 8,
        "jobId": 101,
        "jobTitle": "Backend Developer",
        "status": "Pending",
        "appliedAt": "2026-05-13T08:30:00Z"
      }
    ],
    "savedJobs": [
      {
        "savedJobId": 3,
        "jobId": 101,
        "jobTitle": "Backend Developer"
      }
    ]
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

## 7. Employer API

### `GET /api/v1/employers/me`

- Mô tả: Lấy hồ sơ employer hiện tại.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "employerId": 5,
    "userId": "f8807a6e-28a6-4e3f-bf90-123456789abc",
    "companyName": "ABC Tech",
    "logoUrl": "/uploads/logos/abc.png",
    "description": "Software company",
    "website": "https://abctech.example",
    "address": "Ho Chi Minh City",
    "companySize": "51-200"
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 PROFILE_NOT_FOUND`

### `PUT /api/v1/employers/me`

- Mô tả: Cập nhật hồ sơ employer hiện tại.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "companyName": "ABC Tech",
  "logoUrl": "/uploads/logos/abc.png",
  "description": "Software company",
  "website": "https://abctech.example",
  "address": "Ho Chi Minh City",
  "companySize": "51-200"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "employerId": 5
  },
  "message": "Employer profile updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `400 INVALID_FILE_TYPE`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `GET /api/v1/employer/jobs`

- Mô tả: Lấy danh sách job do employer hiện tại sở hữu.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Query params:
  - `status` (optional)
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "jobId": 101,
        "title": "Backend Developer",
        "status": "Pending",
        "deadline": "2026-06-01T00:00:00Z",
        "createdAt": "2026-05-13T08:30:00Z"
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `GET /api/v1/employer/jobs/{jobId}/applications`

- Mô tả: Lấy danh sách ứng viên của một job thuộc employer.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Query params:
  - `status` (optional)
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "applicationId": 8,
        "candidateId": 12,
        "candidateName": "Nguyen Van A",
        "cvFileId": "cv_apply_8",
        "coverLetter": "I am interested in this role",
        "status": "Pending",
        "appliedAt": "2026-05-13T08:30:00Z",
        "reviewedAt": null,
        "note": null
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`

## 8. Job API

### `POST /api/v1/jobs`

- Mô tả: Tạo job mới.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "jobCategoryId": 2,
  "title": "Backend Developer",
  "description": "<p>Develop APIs</p>",
  "requirement": "<ul><li>.NET 8</li></ul>",
  "benefits": "<p>Flexible working time</p>",
  "salaryMin": 1000,
  "salaryMax": 2000,
  "location": "Ho Chi Minh City",
  "experienceLevel": "Junior",
  "employmentType": "FullTime",
  "deadline": "2026-06-01T00:00:00Z"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "jobId": 101,
    "status": "Pending",
    "slug": "backend-developer-hcm"
  },
  "message": "Job created successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_CATEGORY_NOT_FOUND`

### `PUT /api/v1/jobs/{jobId}`

- Mô tả: Cập nhật job của employer sở hữu job đó.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "jobCategoryId": 2,
  "title": "Backend Developer Updated",
  "description": "<p>Updated description</p>",
  "requirement": "<p>Updated requirement</p>",
  "benefits": "<p>Updated benefits</p>",
  "salaryMin": 1200,
  "salaryMax": 2200,
  "location": "Ho Chi Minh City",
  "experienceLevel": "Junior",
  "employmentType": "FullTime",
  "deadline": "2026-06-15T00:00:00Z"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "jobId": 101,
    "status": "Pending"
  },
  "message": "Job updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`

### `DELETE /api/v1/jobs/{jobId}`

- Mô tả: Xóa mềm hoặc ẩn job của employer sở hữu job đó.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": null,
  "message": "Job deleted successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`

### `GET /api/v1/jobs`

- Mô tả: Lấy danh sách job công khai.
- Roles được phép gọi: `Guest`, `Candidate`, `Employer`
- Auth: Không bắt buộc
- Query params:
  - `keyword`
  - `categoryId`
  - `location`
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "jobId": 101,
        "slug": "backend-developer-hcm",
        "title": "Backend Developer",
        "companyName": "ABC Tech",
        "location": "Ho Chi Minh City",
        "salaryMin": 1000,
        "salaryMax": 2000,
        "deadline": "2026-06-01T00:00:00Z",
        "jobCategory": {
          "jobCategoryId": 2,
          "name": "Backend"
        }
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `400 VALIDATION_ERROR`

### `GET /api/v1/jobs/{slug}`

- Mô tả: Lấy chi tiết một job công khai bằng slug.
- Roles được phép gọi: `Guest`, `Candidate`, `Employer`
- Auth: Không bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "jobId": 101,
    "slug": "backend-developer-hcm",
    "title": "Backend Developer",
    "description": "<p>Develop APIs</p>",
    "requirement": "<ul><li>.NET 8</li></ul>",
    "benefits": "<p>Flexible working time</p>",
    "salaryMin": 1000,
    "salaryMax": 2000,
    "location": "Ho Chi Minh City",
    "experienceLevel": "Junior",
    "employmentType": "FullTime",
    "deadline": "2026-06-01T00:00:00Z",
    "viewCount": 15,
    "company": {
      "employerId": 5,
      "companyName": "ABC Tech",
      "logoUrl": "/uploads/logos/abc.png",
      "description": "Software company"
    },
    "category": {
      "jobCategoryId": 2,
      "name": "Backend",
      "slug": "backend"
    }
  }
}
```

- Error codes:
  - `404 JOB_NOT_FOUND`

### `POST /api/v1/jobs/{jobId}/save`

- Mô tả: Lưu job vào danh sách cá nhân.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "savedJobId": 3
  },
  "message": "Job saved successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`
  - `409 JOB_ALREADY_SAVED`

### `DELETE /api/v1/jobs/{jobId}/save`

- Mô tả: Bỏ lưu job khỏi danh sách cá nhân.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": null,
  "message": "Saved job removed successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 SAVED_JOB_NOT_FOUND`

## 9. Application API

### `POST /api/v1/jobs/{jobId}/apply`

- Mô tả: Candidate nộp đơn ứng tuyển cho một job.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Request body:

```json
{
  "cvFileId": "cv_candidate_12",
  "coverLetter": "I am interested in this role"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "applicationId": 8,
    "status": "Pending",
    "appliedAt": "2026-05-13T08:30:00Z"
  },
  "message": "Application submitted successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`
  - `409 APPLICATION_ALREADY_EXISTS`
  - `422 JOB_EXPIRED`

### `PATCH /api/v1/applications/{applicationId}/status`

- Mô tả: Employer cập nhật trạng thái application theo workflow hợp lệ.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "status": "Reviewed",
  "note": "Candidate profile matches requirements"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "applicationId": 8,
    "status": "Reviewed",
    "reviewedAt": "2026-05-13T09:00:00Z"
  },
  "message": "Application status updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 APPLICATION_NOT_FOUND`
  - `409 INVALID_STATE_TRANSITION`

## 10. Community API

### `POST /api/v1/posts`

- Mô tả: Tạo bài viết cộng đồng.
- Roles được phép gọi: `Candidate`, `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "postCategoryId": 1,
  "title": "How to pass backend interview",
  "summary": "Some useful interview tips",
  "content": "<p>Prepare your fundamentals</p>",
  "thumbnailUrl": "/uploads/posts/interview.png",
  "tagIds": [1, 2]
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "status": "Published",
    "slug": "how-to-pass-backend-interview"
  },
  "message": "Post created successfully"
}
```

- Ghi chú:
  - Trong MVP có thể publish trực tiếp.
  - Sau MVP có thể khởi tạo `Draft` hoặc `Pending` tùy workflow moderation được bật.
- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_CATEGORY_NOT_FOUND`
  - `404 TAG_NOT_FOUND`

### `POST /api/v1/posts/{postId}/submit`

- Mô tả: Gửi bài viết từ `Draft` lên `Pending` để chờ admin duyệt.
- Roles được phép gọi: `Candidate`, `Employer`
- Auth: Bắt buộc
- Phase: `Should`
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "status": "Pending"
  },
  "message": "Post submitted for review"
}
```

- Ghi chú:
  - Chỉ hoạt động khi moderation workflow (Sau MVP) được bật.
  - Chỉ tác giả mới được submit bài của mình.
  - Chỉ có hiệu lực khi post đang ở trạng thái `Draft`.
- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_NOT_FOUND`
  - `409 INVALID_STATE_TRANSITION`

### `PUT /api/v1/posts/{postId}`

- Mô tả: Cập nhật bài viết của chính tác giả.
- Roles được phép gọi: `Candidate`, `Employer`
- Auth: Bắt buộc
- Request body:

```json
{
  "postCategoryId": 1,
  "title": "How to pass backend interview - updated",
  "summary": "Updated summary",
  "content": "<p>Updated content</p>",
  "thumbnailUrl": "/uploads/posts/interview.png",
  "tagIds": [1, 3]
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "status": "Published"
  },
  "message": "Post updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_NOT_FOUND`

### `DELETE /api/v1/posts/{postId}`

- Mô tả: Xóa mềm bài viết của tác giả.
- Roles được phép gọi: `Candidate`, `Employer`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": null,
  "message": "Post deleted successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_NOT_FOUND`

### `GET /api/v1/posts`

- Mô tả: Lấy feed cộng đồng.
- Roles được phép gọi: `Guest`, `Candidate`, `Employer`
- Auth: Không bắt buộc
- Query params:
  - `sort`: `newest` | `trending`
  - `categoryId` (optional)
  - `tagId` (optional)
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "postId": 21,
        "slug": "how-to-pass-backend-interview",
        "title": "How to pass backend interview",
        "summary": "Some useful interview tips",
        "thumbnailUrl": "/uploads/posts/interview.png",
        "upvoteCount": 8,
        "commentCount": 4,
        "bookmarkCount": 2,
        "viewCount": 30,
        "createdAt": "2026-05-13T08:30:00Z",
        "author": {
          "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
          "displayName": "Nguyen Van A"
        }
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `400 VALIDATION_ERROR`

### `GET /api/v1/posts/{slug}`

- Mô tả: Lấy chi tiết bài viết công khai bằng slug.
- Roles được phép gọi: `Guest`, `Candidate`, `Employer`
- Auth: Không bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "slug": "how-to-pass-backend-interview",
    "title": "How to pass backend interview",
    "summary": "Some useful interview tips",
    "content": "<p>Prepare your fundamentals</p>",
    "thumbnailUrl": "/uploads/posts/interview.png",
    "status": "Published",
    "upvoteCount": 8,
    "commentCount": 4,
    "bookmarkCount": 2,
    "viewCount": 30,
    "createdAt": "2026-05-13T08:30:00Z",
    "author": {
      "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
      "displayName": "Nguyen Van A"
    },
    "category": {
      "postCategoryId": 1,
      "name": "Career",
      "slug": "career"
    },
    "tags": [
      {
        "tagId": 1,
        "name": "Interview",
        "slug": "interview"
      }
    ]
  }
}
```

- Error codes:
  - `404 POST_NOT_FOUND`

### `POST /api/v1/posts/{postId}/comments`

- Mô tả: Tạo comment gốc cho bài viết.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Request body:

```json
{
  "content": "Thanks for sharing"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postCommentId": 15,
    "content": "Thanks for sharing",
    "createdAt": "2026-05-13T09:15:00Z"
  },
  "message": "Comment created successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `404 POST_NOT_FOUND`

### `POST /api/v1/posts/{postId}/comments/{commentId}/reply`

- Mô tả: Reply trực tiếp vào comment gốc.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Request body:

```json
{
  "content": "I agree with you"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postCommentId": 16,
    "parentCommentId": 15,
    "content": "I agree with you",
    "createdAt": "2026-05-13T09:20:00Z"
  },
  "message": "Reply created successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `404 COMMENT_NOT_FOUND`
  - `409 COMMENT_REPLY_DEPTH_EXCEEDED`

### `POST /api/v1/posts/{postId}/votes`

- Mô tả: Upvote bài viết.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "upvoteCount": 9
  },
  "message": "Post upvoted successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `404 POST_NOT_FOUND`
  - `409 POST_ALREADY_UPVOTED`

### `DELETE /api/v1/posts/{postId}/votes`

- Mô tả: Bỏ upvote bài viết.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "upvoteCount": 8
  },
  "message": "Post upvote removed successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `404 POST_VOTE_NOT_FOUND`

### `POST /api/v1/posts/{postId}/bookmarks`

- Mô tả: Bookmark bài viết.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "bookmarkCount": 3
  },
  "message": "Post bookmarked successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `404 POST_NOT_FOUND`
  - `409 POST_ALREADY_BOOKMARKED`

### `DELETE /api/v1/posts/{postId}/bookmarks`

- Mô tả: Bỏ bookmark bài viết.
- Roles được phép gọi: `Candidate`, `Employer`, `Admin`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "bookmarkCount": 2
  },
  "message": "Post bookmark removed successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `404 POST_BOOKMARK_NOT_FOUND`

## 11. User Profile API

### `GET /api/v1/users/{userId}/profile`

- Mô tả: Lấy trang profile công khai hoặc profile mở rộng tùy quyền.
- Roles được phép gọi: `Guest`, `Candidate`, `Employer`
- Auth: Không bắt buộc
- Phase: `Should`
- Query params:
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "userId": "b1a8c995-2f6b-4c0a-9f20-123456789abc",
    "role": "Candidate",
    "displayName": "Nguyen Van A",
    "avatarUrl": "/uploads/avatars/a.png",
    "bio": "Backend developer",
    "posts": [
      {
        "postId": 21,
        "title": "How to pass backend interview",
        "slug": "how-to-pass-backend-interview"
      }
    ]
  }
}
```

- Ghi chú:
  - Nếu người gọi là chính chủ candidate, backend có thể bổ sung `appliedJobs` và `savedJobs`.
  - Guest không được nhận `cvFileId` hoặc lịch sử apply riêng tư. CV chỉ được truy cập qua `GET /api/v1/files/cv/{fileId}` với authorization phù hợp.
- Error codes:
  - `404 USER_NOT_FOUND`

## 12. Upload API

### `POST /api/v1/uploads/avatar`

- Mô tả: Upload avatar cho candidate.
- Roles được phép gọi: `Candidate`
- Auth: Bắt buộc
- Phase: `Should`
- Content type: `multipart/form-data`
- Form fields:
  - `file`
- Success response:

```json
{
  "success": true,
  "data": {
    "url": "/uploads/avatars/avatar-12.png"
  },
  "message": "Avatar uploaded successfully"
}
```

- Error codes:
  - `400 INVALID_FILE_TYPE`
  - `400 FILE_TOO_LARGE`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `POST /api/v1/uploads/logo`

- Mô tả: Upload logo công ty cho employer.
- Roles được phép gọi: `Employer`
- Auth: Bắt buộc
- Phase: `Should`
- Content type: `multipart/form-data`
- Form fields:
  - `file`
- Success response:

```json
{
  "success": true,
  "data": {
    "url": "/uploads/logos/logo-5.png"
  },
  "message": "Logo uploaded successfully"
}
```

- Error codes:
  - `400 INVALID_FILE_TYPE`
  - `400 FILE_TOO_LARGE`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `POST /api/v1/uploads/post-thumbnail`

- Mô tả: Upload thumbnail bài viết.
- Roles được phép gọi: `Candidate`, `Employer`
- Auth: Bắt buộc
- Phase: `Should`
- Content type: `multipart/form-data`
- Form fields:
  - `file`
- Success response:

```json
{
  "success": true,
  "data": {
    "url": "/uploads/posts/post-21.png"
  },
  "message": "Thumbnail uploaded successfully"
}
```

- Error codes:
  - `400 INVALID_FILE_TYPE`
  - `400 FILE_TOO_LARGE`
  - `401 UNAUTHORIZED`

## 12b. File Access API

### `GET /api/v1/files/cv/{fileId}`

- Mô tả: Lấy nội dung file CV (PDF) với kiểm soát quyền truy cập.
- Roles được phép gọi: `Candidate` (chính chủ), `Employer` (sở hữu job mà candidate đã apply), `Admin`
- Auth: Bắt buộc
- Quy tắc truy cập:
  - Candidate chỉ xem được CV của chính mình.
  - Employer chỉ xem được CV của ứng viên đã apply vào job do employer sở hữu.
  - Admin có quyền xem tất cả CV.
- Response content type: `application/pdf`
- Response: binary stream của file PDF.
- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 FILE_NOT_FOUND`

> **Ghi chú**: Avatar, logo, thumbnail không cần endpoint riêng vì là file public — được serve trực tiếp từ thư mục tĩnh. CV là file riêng tư duy nhất cần protected endpoint này. Các response API chứa CV trả về `cvFileId` (định danh để gọi endpoint này) thay vì URL trực tiếp.

## 13. Admin API

### `GET /api/v1/admin/jobs`

- Mô tả: Lấy danh sách job để kiểm duyệt.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Query params:
  - `status` (optional)
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "jobId": 101,
        "title": "Backend Developer",
        "companyName": "ABC Tech",
        "status": "Pending",
        "createdAt": "2026-05-13T08:30:00Z"
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `PATCH /api/v1/admin/jobs/{jobId}/status`

- Mô tả: Duyệt hoặc ẩn job.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Request body:

```json
{
  "status": "Approved"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "jobId": 101,
    "status": "Approved"
  },
  "message": "Job status updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`
  - `409 INVALID_STATE_TRANSITION`

### `DELETE /api/v1/admin/jobs/{jobId}`

- Mô tả: Admin xóa mềm job vi phạm.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": null,
  "message": "Job deleted successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`

### `GET /api/v1/admin/dashboard`

- Mô tả: Lấy dữ liệu tổng quan dashboard quản trị.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Success response:

```json
{
  "success": true,
  "data": {
    "summary": {
      "totalUsers": 100,
      "totalJobs": 45,
      "totalPosts": 32,
      "totalApplications": 76
    },
    "jobByCategory": [
      {
        "categoryName": "Backend",
        "jobCount": 12
      }
    ],
    "applicationCountByDate": [
      {
        "date": "2026-05-13",
        "count": 5
      }
    ]
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `GET /api/v1/admin/posts`

- Mô tả: Lấy danh sách bài viết để kiểm duyệt (hỗ trợ moderation workflow).
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Query params:
  - `status` (optional): `Pending` | `Published` | `Hidden`
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "postId": 21,
        "title": "How to pass backend interview",
        "authorName": "Nguyen Van A",
        "categoryName": "Career",
        "status": "Pending",
        "createdAt": "2026-05-13T08:30:00Z"
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `PATCH /api/v1/admin/posts/{postId}/status`

- Mô tả: Duyệt hoặc ẩn bài viết theo moderation workflow.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Request body:

```json
{
  "status": "Published"
}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "status": "Published"
  },
  "message": "Post status updated successfully"
}
```

- Error codes:
  - `400 VALIDATION_ERROR`
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_NOT_FOUND`
  - `409 INVALID_STATE_TRANSITION`

### `GET /api/v1/admin/trash`

- Mô tả: Lấy danh sách dữ liệu đã xóa mềm để admin khôi phục.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Query params:
  - `entityType`: `job` | `post`
  - `page`
  - `pageSize`
- Success response:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "entityType": "job",
        "entityId": 101,
        "title": "Backend Developer",
        "deletedAt": "2026-05-13T10:00:00Z"
      }
    ],
    "page": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`

### `PATCH /api/v1/admin/jobs/{jobId}/restore`

- Mô tả: Khôi phục job đã xóa mềm.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "jobId": 101,
    "isDeleted": false,
    "status": "Hidden"
  },
  "message": "Job restored successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 JOB_NOT_FOUND`
  - `409 JOB_NOT_DELETED`

### `PATCH /api/v1/admin/posts/{postId}/restore`

- Mô tả: Khôi phục post đã xóa mềm.
- Roles được phép gọi: `Admin`
- Auth: Bắt buộc
- Phase: `Should`
- Request body:

```json
{}
```

- Success response:

```json
{
  "success": true,
  "data": {
    "postId": 21,
    "isDeleted": false,
    "status": "Hidden"
  },
  "message": "Post restored successfully"
}
```

- Error codes:
  - `401 UNAUTHORIZED`
  - `403 FORBIDDEN`
  - `404 POST_NOT_FOUND`
  - `409 POST_NOT_DELETED`

## 14. SEO slug và route notes

- Job detail public dùng: `GET /api/v1/jobs/{slug}`
- Post detail public dùng: `GET /api/v1/posts/{slug}`
- Ở giai đoạn MVP, nếu slug chưa bật hoàn toàn, backend có thể tạm dùng `jobId`/`postId` nội bộ nhưng frontend route công khai vẫn nên thiết kế theo slug.
- Khi title đổi và slug đổi, backend nên trả slug mới trong response của endpoint update.

## 15. Error code catalog

| Code                           | HTTP  | Mô tả                              |
| ------------------------------ | ----- | ---------------------------------- |
| `VALIDATION_ERROR`             | `400` | Dữ liệu đầu vào không hợp lệ       |
| `INVALID_FILE_TYPE`            | `400` | Loại file không hợp lệ             |
| `FILE_TOO_LARGE`               | `400` | File vượt giới hạn kích thước      |
| `UNAUTHORIZED`                 | `401` | Thiếu hoặc sai token đăng nhập     |
| `INVALID_CREDENTIALS`          | `401` | Sai thông tin đăng nhập            |
| `FORBIDDEN`                    | `403` | Không có quyền truy cập tài nguyên |
| `ACCOUNT_INACTIVE`             | `403` | Tài khoản bị vô hiệu hóa           |
| `NOT_FOUND`                    | `404` | Không tìm thấy tài nguyên          |
| `USER_NOT_FOUND`               | `404` | Không tìm thấy user                |
| `PROFILE_NOT_FOUND`            | `404` | Không tìm thấy hồ sơ               |
| `JOB_NOT_FOUND`                | `404` | Không tìm thấy job                 |
| `POST_NOT_FOUND`               | `404` | Không tìm thấy post                |
| `COMMENT_NOT_FOUND`            | `404` | Không tìm thấy comment             |
| `APPLICATION_NOT_FOUND`        | `404` | Không tìm thấy application         |
| `SAVED_JOB_NOT_FOUND`          | `404` | Không tìm thấy bản ghi save job    |
| `POST_VOTE_NOT_FOUND`          | `404` | Không tìm thấy upvote              |
| `POST_BOOKMARK_NOT_FOUND`      | `404` | Không tìm thấy bookmark            |
| `JOB_CATEGORY_NOT_FOUND`       | `404` | Không tìm thấy category của job    |
| `POST_CATEGORY_NOT_FOUND`      | `404` | Không tìm thấy category của post   |
| `TAG_NOT_FOUND`                | `404` | Không tìm thấy tag                 |
| `CONFLICT`                     | `409` | Xung đột dữ liệu                   |
| `USERNAME_ALREADY_EXISTS`      | `409` | Username đã tồn tại                |
| `EMAIL_ALREADY_EXISTS`         | `409` | Email đã tồn tại                   |
| `APPLICATION_ALREADY_EXISTS`   | `409` | Candidate đã apply job này         |
| `JOB_ALREADY_SAVED`            | `409` | Candidate đã lưu job này           |
| `POST_ALREADY_UPVOTED`         | `409` | User đã upvote post                |
| `POST_ALREADY_BOOKMARKED`      | `409` | User đã bookmark post              |
| `INVALID_STATE_TRANSITION`     | `409` | Chuyển trạng thái không hợp lệ     |
| `COMMENT_REPLY_DEPTH_EXCEEDED` | `409` | Vượt quá số cấp reply cho phép     |
| `JOB_NOT_DELETED`              | `409` | Job chưa ở trạng thái xóa mềm      |
| `POST_NOT_DELETED`             | `409` | Post chưa ở trạng thái xóa mềm     |
| `JOB_EXPIRED`                  | `422` | Job đã quá hạn apply               |
| `INTERNAL_ERROR`               | `500` | Lỗi hệ thống không mong muốn       |

## 16. Changelog

| Version | Ngày       | Người cập nhật / Tên Model | Nội dung                                                                                                                    |
| ------- | ---------- | -------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| 0.1     | YYYY-MM-DD |                            | Khởi tạo khung                                                                                                              |
| 0.2     | 2026-05-13 | Codex                      | Đồng bộ API spec với functional requirements và ERD/DB rules, bổ sung đầy đủ endpoint, request/response, role và error code |
