Khi tôi cung cấp một thông báo lỗi (Error log) hoặc kết quả test fail, bạn BẮT BUỘC tuân thủ quy trình sau:
1. Đọc và phân tích lỗi dựa trên kiến trúc của dự án.
2. Đưa ra chính xác đoạn code cần sửa đổi (Ví dụ: cung cấp code mới cho file bị lỗi, hoặc hướng dẫn đổi tên file nếu framework không hỗ trợ).
3. Cung cấp lệnh terminal (bash/npm) để tôi chạy lại nhằm verify (xác minh) lỗi đã được fix.
4. Sau đó, lưu lại lịch sử test case và test kết quả vào `experience/` với format:
- Tên file: `{tên-feature}.md`
- Nội dung: Mô tả lỗi, nguyên nhân, cách fix, kết quả test, và các lưu ý khác.
5. Sau đó tạo prompt gửi cho [AGENT-PROMPT-ENGINEER] để tạo prompt mới cho feature tương ứng với lỗi đã được fix.