# Báo cáo kết quả Unit Test: P2-3 (Search) & P2-4 (User Profile)

## 1. Môi trường kiểm thử
- **Framework:** Vitest
- **Lệnh chạy:** `npm run test:unit -- __test__/unit/api/`
- **Ngày thực hiện:** 2026-05-18

## 2. Các Test Cases Mới Đã Thêm

### 2.1. API `/api/recipes` (GET) - Hỗ trợ Search & Filter (P2-3)
Đã cập nhật mock để hỗ trợ `NextRequest` thay vì request rỗng, đảm bảo API có thể trích xuất `query`, `source`, và `cookbookId`.
- ✅ `GET should return 401 if not authenticated`: Kiểm tra chặn người dùng chưa đăng nhập.
- ✅ `GET should return recipes if authenticated`: Kiểm tra việc gọi `searchRecipes` và trả về đúng danh sách recipe khi có xác thực.

### 2.2. API `/api/user/profile` (GET, PUT) - (P2-4)
Tạo mới hoàn toàn file `__test__/unit/api/user-profile.test.ts`.

**GET Method:**
- ✅ `should return 401 if not authenticated`: Kiểm tra middleware `auth()`.
- ✅ `should return 404 if user not found`: Đảm bảo xử lý đúng khi Clerk id không tồn tại trong DB.
- ✅ `should return user profile if authenticated and found`: Kiểm tra lấy thông tin profile chuẩn xác.

**PUT Method:**
- ✅ `should return 401 if not authenticated`
- ✅ `should return 400 if validation fails`: Kiểm tra `UpdateUserSchema` của Zod hoạt động (vd: email sai định dạng).
- ✅ `should update and return user profile`: Kiểm tra việc gọi `updateUser` thành công và trả về dữ liệu đã update.

## 3. Tổng hợp Kết Quả
- **Tổng số Test Files:** 8 (đã thêm 1 file `user-profile.test.ts`).
- **Tổng số Test Cases:** 28 (đã thêm 6 cases cho User Profile).
- **Tỉ lệ Pass:** 100% (28/28 tests passed).
- **Thời gian chạy:** ~755ms.

Mọi chức năng Backend (Service, Schema, Route, Error Handler) cho phần tìm kiếm công thức và quản lý thông tin cá nhân đều đã được đảm bảo hoạt động đúng nghiệp vụ.
