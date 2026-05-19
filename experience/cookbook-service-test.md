# Cookbook Service Backend Test

## Nội dung thay đổi
- Tạo các unit test case cho `Backend/services/cookbook.service.ts` tại `Backend/__test__/unit/services/cookbook.service.test.ts`.
- Bổ sung `vitest.config.ts` với plugin `vite-tsconfig-paths` để Vitest có thể resolve được path alias (`@/backend/...`).

## Các Test Case Đã Thực Hiện
1. `getCookbooksByUser`: Verify trả về list cookbooks đúng user.
2. `getCookbookById`: Verify trả về đúng cookbook kèm theo related recipes.
3. `createCookbook`: Verify tạo cookbook thành công.
4. `updateCookbook`: Verify update cookbook thành công nếu đúng owner, thất bại nếu sai owner.
5. `deleteCookbook`: Verify delete cookbook thành công nếu đúng owner, thất bại nếu sai owner.
6. `addRecipeToCookbook`: Verify add recipe vào cookbook khi owner đúng cả 2.
7. `removeRecipeFromCookbook`: Verify remove recipe khỏi cookbook thành công.

## Lỗi Gặp Phải & Cách Giải Quyết
- **Lỗi**: Khi chạy test báo lỗi `Error: Failed to load url @/backend/services/cookbook.service`. Nguyên nhân do Vitest mặc định không nhận diện path alias trong `tsconfig.json` nếu thiếu cấu hình plugin tương ứng.
- **Cách Fix**: 
  1. Tạo file `vitest.config.ts` tại root.
  2. Import và dùng `vite-tsconfig-paths`.
  3. Cấu hình `environment: 'node'`.

## Kết Quả
- Lệnh chạy: `npx vitest run Backend/__test__/unit/services/cookbook.service.test.ts`
- Trạng thái: **PASS** (11/11 tests pass).

## Lưu Ý
- Do dự án dùng path alias viết hoa (`@/backend/`) khác với các framework Next.js thông thường (`@/Backend/`), cần luôn đảm bảo import đúng chuẩn chữ thường đã setup ban đầu, hoặc sửa alias nếu cần thiết.
