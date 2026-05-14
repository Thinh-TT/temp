# Nhật Ký Phát Triển

## 1. Mục Tiêu

Ghi lại các quyết định kỹ thuật quan trọng, vấn đề phát sinh và bài học kinh nghiệm trong quá trình phát triển.

## 2. Quy Ước

- Mỗi entry ghi: ngày, người thực hiện, loại (Decision / Issue / Lesson).
- Tham chiếu đến task liên quan từ `05-tasks/01-task-board.md`.
- Ghi ngắn gọn, tập trung vào WHY hơn là WHAT.

## 3. Nhật Ký

### Template

```markdown
## YYYY-MM-DD — Tiêu đề ngắn gọn

**Loại:** [Decision | Issue | Lesson]
**Task:** [T-XXX] (nếu có)
**Người thực hiện:**

**Mô tả:**

**Quyết định / Giải pháp:**

**Hệ quả:**
```

---

<!-- Bắt đầu ghi log từ đây -->

### 2026-05-14 — Khởi tạo cấu trúc tài liệu

**Loại:** Decision
**Task:** —
**Người thực hiện:** ThinhTT

**Mô tả:** Tổ chức lại toàn bộ tài liệu dự án theo nhóm chức năng thay vì đánh số phẳng.

**Quyết định:** Cấu trúc 6 nhóm: Project (tổng quan), Architecture (kiến trúc), Functional (yêu cầu chức năng), UI (thiết kế giao diện), Tasks (theo dõi công việc), Logs (nhật ký).

**Hệ quả:** Tài liệu dễ tra cứu theo module, giảm thời gian onboarding cho thành viên mới.
