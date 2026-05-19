# 🧠 Bài Học Kinh Nghiệm: Khắc Phục Lỗi Clerk API Deprecation tại Middleware

*   **Tên Feature:** Xác thực người dùng bằng Clerk (Authentication)
*   **Được ghi nhận bởi:** `[AGENT-TESTING]` qua chu trình Vibe Coding

---

## 1. Mô Tả Lỗi (Error Description)
Trong quá trình khởi chạy môi trường phát triển Next.js 14, máy chủ Node.js ném ra ngoại lệ nghiêm trọng và treo luồng chạy:
```bash
TypeError: auth(...).protect is not a function
    at middleware.ts (c:\Users\ASUS\smart-kitchen-web\middleware.ts)
```

## 2. Phân Tích Nguyên Nhân (Root Cause)
*   Thư viện xác thực `@clerk/nextjs` trong tệp tin cấu hình dự án đã được nâng cấp lên phiên bản mới nhất (v5+).
*   Trong các phiên bản cũ (v4), đối tượng `auth()` trả về là một hàm có thể gọi trực tiếp và chuỗi phương thức bảo vệ `.protect()`. Tuy nhiên, trong phiên bản v5+, đối tượng `auth` đã được cấu trúc lại thành một đối tượng thuần túy (plain object) không thể gọi như một hàm. Việc cố gắng thực hiện lệnh `auth().protect()` dẫn đến lỗi kiểu dữ liệu nghiêm trọng.

## 3. Cách Khắc Phục (The Fix)
Điều chỉnh cú pháp gọi phương thức Clerk middleware bảo vệ route từ hàm gọi sang truy cập thuộc tính đối tượng:
```diff
- export default clerkMiddleware((auth, req) => {
-   if (isProtectedRoute(req)) auth().protect();
- });
+ export default clerkMiddleware((auth, req) => {
+   if (isProtectedRoute(req)) auth.protect();
+ });
```

## 4. Kết Quả Kiểm Thử (Test Verification)
*   Chạy lệnh xác minh cục bộ: `npm run dev`
*   Kết quả: Middleware của Clerk biên dịch thành công 100% trong 139ms, điều hướng chính xác các trang yêu cầu xác thực sang trang đăng nhập và cho phép truy cập các trang công khai.
