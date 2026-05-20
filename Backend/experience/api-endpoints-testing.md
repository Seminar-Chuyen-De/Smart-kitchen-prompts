# Kinh nghiệm: Xử lý lỗi Unit Test API Endpoints

## Mô tả lỗi
Khi viết và chạy Unit Test cho các API endpoints (cụ thể là `app/api/recipes/route.ts` - phương thức `POST`), test case `POST should create recipe` trả về lỗi HTTP status 400 thay vì 201 như kỳ vọng.
Thêm vào đó, Vitest báo lỗi không tìm thấy module import do config đường dẫn absolute alias (`@/app`) không được setup trong `tsconfig.json`.

## Nguyên nhân
1. **Lỗi 400 Bad Request**: Schema `CreateRecipeSchema` (Zod) yêu cầu trường `userId` không được trống. Tuy nhiên, request payload từ Frontend/Test không truyền `userId` mà lấy thông qua `auth()` của Clerk ở Middleware. Dẫn đến hàm `CreateRecipeSchema.parse(body)` bị Zod báo lỗi thiếu field, và API throw về mã 400.
2. **Lỗi Module Resolution**: `vitest.config.ts` đọc alias từ `tsconfig.json`. Nhưng dự án hiện chỉ cài đặt `@/backend/*`, `@/frontend/*`, `@/ai/*` mà KHÔNG có `@/app/*`. Việc import file test bằng path `@/app/api/...` khiến Vite/Vitest crash.

## Cách Fix
1. **Đối với lỗi 400**: Thay vì yêu cầu Payload phải có `userId`, API Route đã tự động lấy `userId` từ Clerk `auth()` và *gắn (inject)* thẳng vào Object body trước khi ném vào Zod Parser:
   ```typescript
   const body = await req.json();
   body.userId = userId; // Inject userId vào body
   const data = CreateRecipeSchema.parse(body);
   ```
2. **Đối với lỗi Module Resolution**: Cập nhật lại đường dẫn import trong các file tests để sử dụng relative path (e.g. `../../../app/api/recipes/route`) thay vì dựa vào alias không tồn tại.

## Kết quả Test
Cả 4 suites và tổng số 9 test cases cho API Recipes, Clerk Webhooks và Analyze Image đã chạy Pass toàn bộ 100%. Lệnh đã chạy: `npm run test:unit -- __test__/unit/api/`.

## Các lưu ý khác
- Khi viết các Zod schema có tái sử dụng (`CreateRecipeSchema = RecipeSchema.omit(...)`), hãy lưu ý các trường như `userId`. Nếu không omit nó, bắt buộc phải tự inject data vào trước khi parse.
- Hạn chế sử dụng `@/app` trong files test nếu tsconfig chưa được cấu hình, hãy dùng relative path hoặc cập nhật `tsconfig.json` một cách nhất quán.
