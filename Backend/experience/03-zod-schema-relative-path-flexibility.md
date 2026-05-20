# 🧠 Bài Học Kinh Nghiệm: Khắc Phục Lỗi Zod Schema Chặn Đường Dẫn Tệp Ảnh Nội Bộ

*   **Tên Feature:** Lưu trữ công thức nấu ăn & Ảnh gốc (Cookbook Image Upload)
*   **Được ghi nhận bởi:** `[AGENT-TESTING]` qua chu trình Vibe Coding

---

## 1. Mô Tả Lỗi (Error Description)
Khi người dùng tải ảnh lên và bấm nút **"Lưu vào Cookbook"**, nút bấm bị vô hiệu hóa, hệ thống ném ra mã lỗi ZodError tại Backend và không thể tạo bản ghi trong DB:
```json
{
  "success": false,
  "error": "ZodValidationError: Invalid input data",
  "details": [
    {
      "code": "invalid_string",
      "path": ["imageRecipe"],
      "message": "Invalid url"
    }
  ]
}
```

## 2. Phân Tích Nguyên Nhân (Root Cause)
*   Zod Schema quản lý dữ liệu đầu vào công thức nấu ăn tại `Backend/schemas/recipe.schema.ts` định nghĩa trường ảnh đại diện `imageRecipe` nghiêm ngặt dưới dạng URL:
    `imageRecipe: z.string().url()`
*   Mục tiêu ban đầu của hệ thống là lưu ảnh Unsplash từ Internet (là liên kết tuyệt đối `https://...`). Tuy nhiên, khi chuyển sang tính năng lưu ảnh gốc của người dùng chụp cục bộ trên máy chủ, đường dẫn ảnh trả về là relative path dạng: `/uploads/1779093254543-pupb1k.jpg`.
*   Relative path không bắt đầu bằng giao thức mạng (`http/https`) nên không thỏa mãn định dạng `.url()` của Zod, dẫn đến lỗi validate chặn đứng nghiệp vụ lưu trữ món ăn.

## 3. Cách Khắc Phục (The Fix)
Nới lỏng ràng buộc Zod Schema đối với trường ảnh đại diện, thay thế kiểm duyệt `.url()` bằng kiểm duyệt độ dài chuỗi tối đa ký tự `.max(1000)` để chấp nhận cả đường dẫn ảnh nội bộ tương đối và ảnh tuyệt đối từ Internet:
```diff
export const CreateRecipeSchema = z.object({
  title: z.string().min(1, "Tiêu đề không được để trống"),
  description: z.string().optional(),
- imageRecipe: z.string().url("Đường dẫn ảnh không hợp lệ"),
+ imageRecipe: z.string().max(1000, "Đường dẫn ảnh quá dài").optional(),
  cookTime: z.number().int().nonnegative().optional(),
  servings: z.number().int().nonnegative().optional(),
  cookbookId: z.string().min(1, "ID Cookbook không được để trống"),
});
```

## 4. Kết Quả Kiểm Thử (Test Verification)
*   Thực hiện kiểm thử lưu món ăn với tệp tin ảnh tải lên thực tế.
*   Kết quả: Dữ liệu được xác thực thành công tức thì, Prisma ghi bản ghi PostgreSQL hợp lệ, trả về HTTP status `200 OK` và hiển thị ảnh gốc sắc nét trên giao diện Cookbook của người dùng!
