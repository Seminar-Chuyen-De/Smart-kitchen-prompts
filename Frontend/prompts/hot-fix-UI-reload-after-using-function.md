**MỤC TIÊU:** Fix lỗi hiển thị ở trang `cookbook[id]`: Khi thêm công thức vào cookbook thông qua Modal AddRecipe, Modal chỉ tắt đi nhưng danh sách công thức trên trang `cookbook[id]` không tự động cập nhật. Cần sửa logic frontend để trang tự động tải lại dữ liệu (re-fetch) hoặc cập nhật state ngay sau khi thêm công thức thành công mà không cần người dùng phải bấm refresh trang. Cuối cùng, thực hiện quy trình Git để tạo nhánh và lưu code chuẩn xác.

**PHÂN TÍCH KIẾN TRÚC & CÁC BƯỚC THỰC THI:**
(Hãy viết code theo đúng thứ tự này, xong bước nào dừng lại để tôi test rồi mới làm tiếp)

- BƯỚC 1 - UI Component (`Frontend/`) - Đóng vai trò [AGENT-UI]:
  + Mở component hiển thị trang cookbook detail (vd: `Frontend/components/cookbook/CookbookDetail.tsx`) và component Modal AddRecipe.
  + Kiểm tra logic gọi custom hook fetching data (SWR/React Query) hoặc Server Action (nếu dùng Next.js App Router).
  + Sửa logic hàm submit trong AddRecipe Modal: Sau khi API thêm công thức trả về thành công (status 200/201), ngay lập tức gọi hàm trigger cập nhật UI (như `mutate()` của SWR, `router.refresh()` của Next.js, hoặc cập nhật state React tương ứng) để dữ liệu hiển thị mới nhất. Đóng Modal sau khi đã cập nhật UI.
  + Tối ưu giao diện: Khi người dùng bấm vào một công thức để thêm, thay đổi dấu "+" thành animation "rotation-loading" (vd: icon `Loader2` với `animate-spin`) và vô hiệu hóa nút bấm trong quá trình xử lý dữ liệu.
  + Đảm bảo component giữ nguyên cấu trúc chuẩn, có hiển thị loading (skeleton/spinner) trong thời gian chờ update dữ liệu.


- BƯỚC 2 - Lưu Code và Tạo nhánh (Git Flow) - Đóng vai trò [AGENT-GIT-WORKFLOW]:
  + Sau khi tôi test frontend hoạt động đúng, hãy xuất cho tôi kịch bản bash tuần tự để rẽ nhánh, commit và đẩy code.
  + Nhánh bắt buộc: `fix/hot-fix-UI-reload-after-using-function`
  + Câu lệnh commit mẫu: `fix: cập nhật tự động re-fetch danh sách cookbook sau khi thêm công thức`
  + Hướng dẫn ngắn gọn cho tôi copy lệnh bash chạy vào terminal.

- BƯỚC 3 - Cập nhật CHANGELOG:
  + Mở và cập nhật file `Smart-kitchen-prompts/CHANGELOG.md` theo chuẩn quy định tại `Smart-kitchen-prompts/README.md`.
  + Ghi chú lại lịch sử sửa lỗi UI (tăng version theo mức Patch 0.0.x). Phải đảm bảo format:
    ```markdown
    ## [X.Y.Z] — YYYY-MM-DD — Mô tả ngắn
    
    ### Loại thay đổi
    - Chi tiết thay đổi...
    ```

**RÀNG BUỘC (CONSTRAINTS BẮT BUỘC):**
- Tuân thủ quy tắc 3 thư mục, sử dụng đúng path aliases (`@/Frontend/*`, `@/Backend/*`).
- Không nhồi business logic hay gọi trực tiếp Prisma/AI API vào trong thư mục `app/`.
- [AGENT-UI]: CHỈ ĐƯỢC PHÉP tạo/sửa file trong thư mục `Frontend/`. Không gọi trực tiếp Database. Bắt buộc viết bằng TypeScript và dùng Tailwind CSS.
- [AGENT-GIT-WORKFLOW]: Không bao giờ dùng `git push --force` lên `main`. Luôn bắt đầu từ code mới nhất của `main`. Chỉ xuất bash code block để copy-paste trực tiếp.
