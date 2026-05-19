# 🧠 Bài Học Kinh Nghiệm: Chuẩn Hóa Tệp Cấu Hình Next.js 14 trong Môi Trường TypeScript

*   **Tên Feature:** Cấu hình môi trường phát triển (Next.js Config)
*   **Được ghi nhận bởi:** `[AGENT-TESTING]` qua chu trình Vibe Coding

---

## 1. Mô Tả Lỗi (Error Description)
Trình biên dịch của Next.js ném ra lỗi cú pháp cấu hình khi khởi động dev server:
```bash
Error: Configuring Next.js via 'next.config.ts' is not supported. Please use 'next.config.js' or 'next.config.mjs' instead.
```

## 2. Phân Tích Nguyên Nhân (Root Cause)
*   Next.js phiên bản 14.x trở xuống chưa hỗ trợ tính năng biên dịch tệp tin cấu hình mở rộng trực tiếp bằng TypeScript `.ts` (`next.config.ts`). Tính năng cấu hình TypeScript trực tiếp chỉ bắt đầu được hỗ trợ từ Next.js phiên bản 15+.
*   Do đó, khi tệp tin cấu hình khởi tạo là `next.config.ts`, Next.js 14 không thể tự parse và ném ra lỗi ngoại lệ chặn tiến trình biên dịch.

## 3. Cách Khắc Phục (The Fix)
*   Đổi tên tệp cấu hình từ `next.config.ts` sang `next.config.mjs`.
*   Sử dụng cú pháp định nghĩa kiểu JSDoc của TypeScript (`/** @type {import('next').NextConfig} */`) để giữ nguyên khả năng kiểm tra lỗi cú pháp và gợi ý code tự động của IDE:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
      },
    ],
  },
};

export default nextConfig;
```

## 4. Kết Quả Kiểm Thử (Test Verification)
*   Tiến trình khởi động dev server chạy mượt mà, Next.js nhận diện tệp `next.config.mjs` tức thì và hoàn tất biên dịch trang chủ trong vòng chưa đầy 1900ms.
