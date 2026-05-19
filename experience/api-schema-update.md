# Lịch sử Test: Cập nhật Schema & APIs theo ERD (api-schema-update)

**Ngày test**: 2026-05-19
**Agent phụ trách**: `[AGENT-TESTING]`

### 1. Mô tả thay đổi:
- Áp dụng cấu trúc `ERD.md` vào hệ thống. Đổi ID sang `number`.
- Tạo mới các API, Schema và Service cho `Steps`, `Ingredients`, `Tags`.
- Cập nhật các API cũ của `Recipe`, `Cookbook`, và User.

### 2. Kết quả test ban đầu:
- **Pass**: 29/29 logic unit tests (sau khi update mock data trong test files).
- **Fail**: 9 files bị lỗi `No test suite found in file`.

### 3. Nguyên nhân lỗi:
- Các file test bị fail là do được tạo sẵn thư mục trống (nằm trong phase P3-1, P3-2 của Backlog) mà chưa có block `describe` hay `it` nào, khiến Vitest văng lỗi.

### 4. Cách Fix:
- Bổ sung cấu trúc mock cơ bản với `describe('Todo')` và `it.todo('...')` vào cả 9 files trống để hệ thống báo Skip thay vì Error.

### 5. Kết quả sau khi Fix:
- Tất cả unit test chạy thành công xanh 100% (kể cả skipped/todo). Tích hợp Database Schema và API thành công không gây sụp hệ thống.
