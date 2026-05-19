# [AGENT-UI] — Hot Fix UI & Logic Cập Nhật 

---

## MỤC TIÊU

Thực hiện chuỗi các bản vá lỗi (hot fixes) và cải thiện trải nghiệm người dùng (UX) cho trang Dashboard và Recipes, cũng như khắc phục vấn đề khởi tạo dữ liệu mẫu cho người dùng.

---

## CHI TIẾT CÁC LỖI ĐÃ FIX VÀ GIẢI PHÁP

### 1. Lỗi Hydration Mismatch ở Dashboard (Random Tip)
- **Vấn đề**: Việc sử dụng `Math.random()` trong `useState` initializer ở Server Component sinh ra lỗi "Text content does not match server-rendered HTML" do kết quả ở Server và Client khác nhau.
- **Giải pháp**: 
  - Khởi tạo `tipIndex = 0` trên Server để đảm bảo an toàn hydration.
  - Sử dụng `useEffect` chỉ chạy trên Client để ngẫu nhiên hóa vị trí ban đầu.
  - Nâng cấp UI: Biến "Mẹo hôm nay" thành một carousel tự động xoay mỗi 10 giây kèm theo hiệu ứng animation trượt từ trên xuống.

### 2. Dữ liệu giả (Mock Data) tồn tại trong Hook `useRecipes`
- **Vấn đề**: Hook `useRecipes` được khởi tạo ban đầu với dữ liệu giả `MOCK_RECIPES`. Nếu request fetch dữ liệu thất bại, nó vẫn âm thầm giữ lại dữ liệu giả mà không thông báo cho người dùng.
- **Giải pháp**:
  - Gỡ bỏ hoàn toàn `MOCK_RECIPES`. 
  - Khởi tạo state với array rỗng `[]` và `isLoading = true` để Skeleton hiển thị ngay lập tức.
  - Thêm tính năng tự động fetch (`useEffect`) ngay trong hook.
  - Chặn lỗi (catch error) và truyền lên bề mặt UI thay vì "nuốt lỗi" âm thầm.

### 3. Dư thừa logic ở trang `RecipesPage.tsx`
- **Vấn đề**: `useEffect` gọi `fetchRecipes()` dư thừa trong component do hook đã tự lo. Không có giao diện báo lỗi nếu fetch thất bại.
- **Giải pháp**:
  - Gỡ bỏ `useEffect` khỏi component.
  - Bổ sung một "Error Banner" màu đỏ hiển thị thông báo lỗi kèm theo nút "Thử lại".
  - Cập nhật bộ đếm công thức thành định dạng `[Đã lọc] / [Tổng số] công thức`.

### 4. Lỗi Seeding Default Recipes không tạo Cookbook
- **Vấn đề**: Chức năng tự động tạo 10 công thức mẫu không tự động sinh ra Cookbook và gán các công thức đó vào Cookbook, khiến bộ sưu tập của người dùng trống rỗng dù công thức đã được lưu. Nếu 1 recipe lỗi, toàn bộ tiến trình sẽ dừng lại.
- **Giải pháp**:
  - Đóng gói mỗi lần tạo recipe bằng `try/catch` để không block chu trình.
  - Cập nhật hàm `seedDefaultRecipesForUser` để sau khi hoàn tất seeding, sẽ tìm hoặc tự tạo một Cookbook có tên `📖 Công thức yêu thích` và gán (assign) tất cả 10 công thức vừa seed vào.

### 5. Seeding cho User cũ (Tính năng mới)
- **Vấn đề**: Do logic chỉ chạy tự động cho người dùng CHƯA CÓ công thức nào (recipes.length === 0), người dùng cũ sẽ không nhận được dữ liệu mẫu này.
- **Giải pháp**:
  - Tạo thêm một script CLI tại `scripts/seed-recipes-for-user.ts`.
  - Hỗ trợ câu lệnh chạy thủ công: `npx tsx scripts/seed-recipes-for-user.ts <USER_ID>` để chèn công thức mẫu cho bất kỳ người dùng nào theo yêu cầu quản trị.

### 6. Xoá Mock Data của Cookbook và Cập nhật Hook `useCookbooks`
- **Vấn đề**: Tương tự như Recipes, Cookbook vẫn đang sử dụng `MOCK_COOKBOOKS` làm dữ liệu tạm, và gọi fetch dư thừa ở các components.
- **Giải pháp**:
  - Gỡ bỏ hoàn toàn `MOCK_COOKBOOKS` khỏi `useCookbooks`.
  - Khởi tạo mặc định bằng mảng rỗng `[]` và tự động fetch (`useEffect`) ngay trong hook.
  - Sửa `CookbooksPage.tsx` và `CookbookDetail.tsx` để xoá bỏ `useEffect` gọi fetch thủ công.
  - Thêm Error Banner vào trang danh sách Cookbooks để hiển thị trạng thái lỗi nếu API gặp sự cố.
