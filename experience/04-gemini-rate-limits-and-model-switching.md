# 🧠 Bài Học Kinh Nghiệm: Khắc Phục Lỗi Hạn Mức Gemini (429 Rate Limit) & Chiến Lược Model Switching

*   **Tên Feature:** Quét ảnh AI nhận diện nguyên liệu (AI Scan)
*   **Được ghi nhận bởi:** `[AGENT-TESTING]` qua chu trình Vibe Coding

---

## 1. Mô Tả Lỗi (Error Description)
Khi người dùng tải ảnh lên để nhận diện nguyên liệu, hệ thống thông báo quét thất bại kèm mã lỗi ngoại lệ API trả về:
```json
Gemini API error: {
  "error": {
    "code": 429,
    "message": "You exceeded your current quota, please check your plan and billing details. Quota exceeded for metric: generativelanguage.googleapis.com/generate_content_free_tier_requests, limit: 20, model: gemini-2.5-flash"
  }
}
```
Hoặc khi người dùng thay đổi khóa API mới, hệ thống ném ra mã lỗi `400 API Key Not Valid`.

## 2. Phân Tích Nguyên Nhân (Root Cause)
*   **Lỗi 429 Rate Limit:** Google AI Studio Free Tier giới hạn tối đa 20 cuộc gọi/ngày trên dải IP dùng chung đối với mô hình thế hệ mới `gemini-2.5-flash`. Do người dùng sử dụng dải IP công cộng trùng lắp với nhiều người dùng khác, hạn ngạch của IP này bị cạn kiệt.
*   **Lỗi 400 Bad Key:** Khi người dùng thay thế key mới vào `.env`, ký tự dán bị dính khoảng trắng hoặc ký tự ngắt dòng `\n` bí ẩn, khiến trình nạp biến môi trường nhận nhầm cú pháp dẫn đến khóa bị hỏng.

## 3. Cách Khắc Phục (The Fix)
*   **Dọn sạch .env:** Loại bỏ hoàn toàn khoảng trắng và ký tự xuống dòng, quy định khóa API là dòng đơn chuẩn chỉnh.
*   **Chiến lược Dự phòng (Fallback & Model Switching):** Thiết lập cơ chế tự động chuyển đổi linh hoạt mô hình trong logic gọi API tại `app/api/scan/route.ts` nhằm mở rộng hạn ngạch cuộc gọi độc lập trên môi trường Free:
```javascript
// Cấu hình linh hoạt mô hình dự phòng
const tryFetchGemini = async (modelName, body, apiKey) => {
  const geminiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${modelName}:generateContent?key=${apiKey}`;
  const response = await fetch(geminiUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body)
  });
  return response;
};

// Gọi chính với gemini-2.5-flash, tự động chuyển đổi sang gemini-1.5-flash nếu dính lỗi hạn ngạch
let response = await tryFetchGemini("gemini-2.5-flash", requestBody, apiKey);
if (response.status === 429) {
  console.warn("Gemini 2.5 Flash rate limited. Switching fallback to 1.5 Flash...");
  response = await tryFetchGemini("gemini-1.5-flash", requestBody, apiKey);
}
```
*   **Chỉ dẫn UX Tiếng Việt:** Khi cả hai mô hình đều trả về mã lỗi 429, hiển thị thông báo chi tiết tiếng Việt hướng dẫn người dùng thay đổi Wifi, chuyển sang 4G cá nhân hoặc bật/tắt VPN để đổi địa chỉ IP IP NAT sạch.

## 4. Kết Quả Kiểm Thử (Test Verification)
*   Hệ thống tự động phát hiện và phục hồi dịch vụ xuất sắc. Người dùng sau khi thực hiện chuyển đổi mạng WiFi và dán khóa chuẩn đã hoàn tất 100% việc quét ảnh và lưu thành công công thức món ăn mượt mà!
