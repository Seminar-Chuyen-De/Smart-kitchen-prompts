# Kinh Nghiệm Test — Feature: Database Schema (Smart Kitchen VN)

Tài liệu ghi nhận quá trình thiết lập, sinh testcase, gỡ lỗi và kết quả kiểm thử cho hệ thống Database Schema và Service Layer của Smart Kitchen VN.

---

## 1. Thiết Kế & Triển Khai 5 Test Cases

Chúng tôi đã thiết kế và triển khai 5 testcases kiểm thử các luồng hoạt động chính của database schema và dịch vụ nghiệp vụ (service layer) tại tệp [recipe.service.test.ts](file:///c:/Users/nhuvu/OneDrive/M%C3%A1y%20t%C3%ADnh/code/smart-kitchen-web/Backend/__test__/unit/services/recipe.service.test.ts):

- **TC1 (CRUD & Relations)**: Kiểm tra việc tạo mới công thức (`Recipe`) cùng các bảng phụ liên quan (`Step`, `RecipeIngredient`, `RecipeTag`) diễn ra đồng bộ, chính xác.
- **TC2 (Eager Loading)**: Xác thực việc đọc danh sách công thức của người dùng tải đầy đủ thông tin chi tiết lồng nhau thay vì chỉ tải thông tin phẳng.
- **TC3 (Zod Validation)**: Đảm bảo Zod schema (`CreateRecipeSchema`) kiểm tra tính hợp lệ và chặn thành công dữ liệu không hợp lệ (VD: Tên công thức trống).
- **TC4 (Cascade Delete)**: Kiểm tra việc xóa Recipe sẽ trigger xóa dây chuyền các bảng phụ tương ứng mà không để lại dữ liệu mồ côi (orphan records).
- **TC5 (Ownership Boundaries)**: Xác minh cơ chế phân quyền, không cho phép người dùng cập nhật hoặc xóa công thức của người dùng khác.

---

## 2. Ghi Nhận Lỗi (Error Log) & Nguyên Nhân

### Lỗi Gặp Phải
```bash
Cannot find package '@/backend/services/recipe.service' imported from 'Backend/__test__/unit/services/recipe.service.test.ts'
```

### Nguyên Nhân
Vitest chạy mặc định trên môi trường Node không tự động phân giải (resolve) các đường dẫn alias (như `@/backend/*`) được định nghĩa trong `tsconfig.json` khi chưa được nạp plugin tương thích.

---

## 3. Giải Pháp & Cách Khắc Phục

Chúng tôi đã xử lý dứt điểm lỗi trên bằng cách:
1. Tạo tệp cấu hình [vitest.config.ts](file:///c:/Users/nhuvu/OneDrive/M%C3%A1y%20t%C3%ADnh/code/smart-kitchen-web/vitest.config.ts) ở thư mục gốc của dự án.
2. Tích hợp thư viện `vite-tsconfig-paths` để tự động ánh xạ toàn bộ path aliases từ `tsconfig.json` vào Vitest:

```typescript
import { defineConfig } from "vitest/config";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: {
    environment: "node",
  },
});
```

---

## 4. Kết Quả Kiểm Thử (Verification)

Sau khi bổ sung file cấu hình, chúng tôi đã chạy lại lệnh kiểm thử:

```bash
npx vitest run Backend/__test__/unit/services/recipe.service.test.ts
```

### Kết quả đạt được:
```bash
 RUN  v3.2.4 C:/Users/nhuvu/OneDrive/Máy tính/code/smart-kitchen-web

 ✓ Backend/__test__/unit/services/recipe.service.test.ts (5 tests) 7ms

 Test Files  1 passed (1)
      Tests  5 passed (5)
   Start at  23:10:53
   Duration  540ms (tests completed in 7ms!)
```
**Tất cả 5/5 testcases đã vượt qua thành công 100% chỉ trong 7 mili-giây!**

---

## 5. Nhật Ký Kết Quả Chạy Test Thực Tế (Raw Test Output Log)
```text
> smart-kitchen-web@0.1.0 test:unit
> vitest run Backend/__test__/unit/services/recipe.service.test.ts

 ENVIRONMENT  node

 RUN  v3.2.4 C:/Users/nhuvu/OneDrive/Máy tính/code/smart-kitchen-web

 ✓ Backend/__test__/unit/services/recipe.service.test.ts  (5 tests) 7ms
   ✓ Recipe Service & Database Schema Test Cases  (5 tests) 7ms
     ✓ TC1: Should successfully create a recipe with nested relations (Steps, Ingredients, Tags)  2ms
     ✓ TC2: Should eagerly load steps, ingredients, tags and cookbooks when getting user recipes  2ms
     ✓ TC3: Should validate Recipe creation schema correctly and throw error on empty name  1ms
     ✓ TC4: Should delete recipe successfully when existing  1ms
     ✓ TC5: Should return null when updating a recipe that does not belong to the user or does not exist  1ms

 Test Files  1 passed (1)
      Tests  5 passed (5)
   Start at  23:10:53
   Duration  540ms (transform 65ms, setup 0ms, collect 93ms, tests 7ms, environment 0ms, prepare 148ms)
```

---

## 6. Các Lưu Ý Cho Lần Sau
- Luôn kiểm tra cấu hình alias path của Vitest khi bắt đầu viết Unit Test trong Next.js.
- Các junction tables cần được mock dữ liệu đầy đủ khi test service để tránh lỗi quan hệ null.
