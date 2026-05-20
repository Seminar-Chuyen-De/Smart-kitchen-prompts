# Kinh nghiệm: Thực hiện Unit Test cho Cookbook API Endpoints

## Lịch sử Test Case
Các unit tests đã được triển khai cho 3 API routes phục vụ tính năng Cookbook Management (P1-3).
Các test files được tạo tại `__test__/unit/api/`:
1. `cookbooks.test.ts`
   - GET `/api/cookbooks`: 401 nếu chưa login, 200 kèm danh sách nếu thành công.
   - POST `/api/cookbooks`: 201 tạo thành công, 400 nếu truyền sai dữ liệu Zod schema.
2. `cookbook-id.test.ts`
   - GET `/api/cookbooks/[id]`: 401, 404 nếu không tìm thấy, 200 trả về chi tiết.
   - PUT `/api/cookbooks/[id]`: 200 update thành công.
   - DELETE `/api/cookbooks/[id]`: 200 xóa thành công.
3. `cookbook-recipes.test.ts`
   - POST `/api/cookbooks/[id]/recipes`: 200 thêm thành công, 400 nếu thiếu recipeId.
   - DELETE `/api/cookbooks/[id]/recipes`: 200 gỡ recipe thành công (hỗ trợ đọc `recipeId` từ JSON body hoặc `searchParams`).

## Kết Quả
- Lệnh chạy: `npx vitest run __test__/unit/api/cookbooks.test.ts __test__/unit/api/cookbook-id.test.ts __test__/unit/api/cookbook-recipes.test.ts`
- Trạng thái: **PASS** (13/13 tests pass).

## Lưu ý (Lessons Learned)
- Bài học từ việc Inject `userId` trước khi parse Zod Schema đã phát huy tác dụng. Ở API này, `userId` không được đặt trong `CreateCookbookSchema`, do đó ta chỉ cần gọi parse payload từ client gửi lên mà không gặp lỗi validation giả, sau đó truyền trực tiếp `userId` vào `cookbookService` là được.
- Ở route `DELETE` recipe khỏi cookbook, việc hỗ trợ nhận `recipeId` qua 2 cách (`body` và `searchParams`) giúp Frontend linh hoạt hơn khi fetch data, vì một số HTTP Client không hỗ trợ gửi Body kèm với method DELETE. Test cases cũng đã cover đầy đủ 2 trường hợp này.
