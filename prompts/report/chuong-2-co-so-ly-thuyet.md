# CHƯƠNG 2: CƠ SỞ LÝ THUYẾT

Chương này trình bày các kiến thức nền tảng kỹ thuật được sử dụng trong dự án Smart Kitchen VN. Mỗi công nghệ được mô tả súc tích tập trung vào lý do lựa chọn và vai trò trong hệ thống.

---

## 2.1 Mô Hình Ngôn Ngữ Lớn (LLM) — Claude Sonnet 4.5

Mô hình ngôn ngữ lớn (Large Language Model - LLM) là các mô hình học sâu được huấn luyện trên khối lượng văn bản khổng lồ, có khả năng hiểu và sinh ra ngôn ngữ tự nhiên theo ngữ cảnh. Không giống các hệ thống rule-based truyền thống, LLM có thể xử lý yêu cầu mở, suy luận đa bước và tạo ra nội dung có cấu trúc từ input phi cấu trúc.

**Claude Sonnet 4.5** (Anthropic) được lựa chọn cho dự án vì: (1) khả năng theo instruction chuẩn xác, phù hợp để sinh recipe có schema nhất quán; (2) context window lớn đủ để xử lý danh sách nguyên liệu dài và trả về JSON có cấu trúc; (3) API ổn định với SDK TypeScript chính thức (`@anthropic-ai/sdk`).

Trong hệ thống, Claude Sonnet 4.5 đóng vai trò **Recipe Agent** — nhận vào danh sách nguyên liệu (`string[]`) và sinh ra `RecipeDTO` đầy đủ bao gồm tên món, mô tả, các bước thực hiện, thông tin dinh dưỡng và nhãn phân loại.

---

## 2.2 Computer Vision — Google Cloud Vision API

Computer Vision là lĩnh vực AI cho phép máy tính "nhìn" và hiểu nội dung hình ảnh. Trong bài toán nhận diện thực phẩm, đây là bước quan trọng đầu tiên trong pipeline AI.

**Google Cloud Vision API** cung cấp dịch vụ phân tích ảnh theo yêu cầu (API call), trong đó tính năng **Label Detection** trả về danh sách các nhãn mô tả nội dung ảnh kèm điểm tin cậy (confidence score). Ưu điểm so với tự huấn luyện mô hình: không cần dataset, không cần hạ tầng GPU, độ chính xác cao với thực phẩm phổ biến, và chi phí hợp lý theo số lượng request.

Trong Smart Kitchen, Vision Agent gửi ảnh từ người dùng đến Google Vision API và lọc kết quả để lấy các nhãn có xác suất cao nhất, chuyển đổi thành danh sách nguyên liệu tiếng Việt.

---

## 2.3 Multi-Agent System (MAS/MSA)

Multi-Agent System là kiến trúc phần mềm gồm nhiều agent tự trị phối hợp để giải quyết vấn đề phức tạp mà một agent đơn lẻ không thể xử lý hiệu quả. Mỗi agent có nhiệm vụ chuyên biệt, giao tiếp qua message, và hoạt động độc lập với nhau.

**Ưu điểm của MSA so với monolithic AI:**
- **Chuyên biệt hóa**: Mỗi agent tối ưu cho một nhiệm vụ cụ thể.
- **Khả năng mở rộng**: Thêm agent mới mà không ảnh hưởng agent hiện tại.
- **Dễ debug**: Lỗi được cô lập trong từng agent, không lan rộng toàn hệ thống.
- **Tái sử dụng**: Một agent có thể được gọi từ nhiều workflow khác nhau.

Trong Smart Kitchen, MSA được thiết kế với 4 agent: Orchestrator (điều phối), Vision (nhận diện ảnh), Recipe (sinh công thức), và Storage (lưu trữ).

---

## 2.4 Next.js 14 App Router

Next.js là React framework full-stack, phiên bản 14 giới thiệu **App Router** — cơ chế routing mới dựa trên file-system với khái niệm Server Components và Client Components. Server Components chạy hoàn toàn trên server, không gửi JavaScript xuống client, phù hợp cho data fetching và rendering. Client Components (đánh dấu `'use client'`) chạy trên browser, xử lý tương tác người dùng.

Trong Smart Kitchen, `app/` chỉ đóng vai trò **routing layer siêu mỏng** — mỗi file page chỉ vài dòng import component từ `Frontend/`. Toàn bộ logic nằm trong `Frontend/`, `Backend/`, và `AI/`. Pattern này giúp tái sử dụng component dễ dàng và tránh coupling chặt với framework routing.

---

## 2.5 PostgreSQL và Prisma ORM

**PostgreSQL** là hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở, nổi bật với hỗ trợ kiểu dữ liệu phong phú (JSON, Array, ENUM), full-text search, và tính ổn định trong môi trường production. Dự án sử dụng kiểu `Json` của PostgreSQL để lưu `ingredients[]` và `instructions[]` — linh hoạt hơn các cột chuẩn trong giai đoạn MVP.

**Prisma ORM** cung cấp lớp truy cập database type-safe, tự động sinh TypeScript types từ schema, và CLI để quản lý migration. Prisma Client singleton pattern (`Backend/db/client.ts`) đảm bảo không tạo nhiều connection pool trong môi trường serverless của Next.js.

---

## 2.6 Xác Thực Người Dùng với Clerk

**Clerk** là dịch vụ Authentication-as-a-Service hỗ trợ OAuth 2.0 (Google, GitHub...), email/password, và magic link. Thay vì tự xây dựng auth system phức tạp, Clerk cung cấp: UI components sẵn có (SignIn, SignUp), session management, JWT verification middleware, và Webhook để thông báo sự kiện user đến hệ thống.

Trong Smart Kitchen, flow auth hoạt động: Clerk xử lý đăng ký/đăng nhập → phát Webhook event `user.created` → hệ thống tạo record trong bảng `users` nội bộ → middleware Clerk kiểm tra JWT trên mọi request đến `app/api/`. Cách tiếp cận này tách biệt hoàn toàn auth logic ra khỏi business logic.
