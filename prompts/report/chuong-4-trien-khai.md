# CHƯƠNG 4: TRIỂN KHAI VÀ KẾT QUẢ ĐẠT ĐƯỢC

Chương này mô tả môi trường phát triển, các tính năng đã được triển khai thực tế trong codebase, kết quả đạt được, cũng như những khó khăn gặp phải và cách giải quyết trong quá trình phát triển.

---

## 4.1 Môi Trường Phát Triển

| Thành phần | Công cụ / Phiên bản |
|---|---|
| Hệ điều hành | Windows 11 |
| IDE | Visual Studio Code |
| Runtime | Node.js 20 LTS |
| Package Manager | npm / bun |
| Version Control | Git + GitHub |
| Database | PostgreSQL 16 (local) |
| AI APIs | Google Cloud Vision API, Anthropic Claude Sonnet 4.5 |
| Testing | Vitest (unit), Playwright (E2E) |
| Framework | Next.js 14.2.29 |

---

## 4.2 Các Tính Năng Đã Triển Khai

### 4.2.1 Cấu hình dự án (Foundation)

Toàn bộ cấu hình dự án đã được thiết lập hoàn chỉnh:

- **`tsconfig.json`**: TypeScript strict mode với path aliases đầy đủ (`@/frontend/*`, `@/backend/*`, `@/ai/*`).
- **`tailwind.config.ts`**: Brand color palette, dark theme tokens, custom utilities (`glass-card`, `btn-primary`).
- **`next.config.mjs`**: Image optimization domains, server-only packages.
- **`.env.example`**: Template đầy đủ các biến môi trường cần thiết (Clerk keys, Google Vision credentials, Anthropic API key, Database URL).
- **`prisma/schema.prisma`**: Schema đầy đủ 4 models chính (User, Cookbook, Recipe, ScanLog) với relations và enums.
- **`Backend/db/client.ts`**: Prisma singleton pattern, tránh connection pool leak trong môi trường serverless.

### 4.2.2 Hệ thống xác thực (Authentication)

Hệ thống xác thực được tích hợp hoàn toàn với Clerk:

- **`middleware.ts`**: Clerk middleware bảo vệ toàn bộ route trừ landing page và các route public. Sử dụng `clerkMiddleware` với `createRouteMatcher` để định nghĩa public routes.
- **`app/sign-in/[[...sign-in]]/page.tsx`** và **`app/sign-up/[[...sign-up]]/page.tsx`**: Trang đăng nhập và đăng ký với Clerk UI components, tự động xử lý OAuth flow.
- **`app/layout.tsx`**: `ClerkProvider` wrap toàn bộ app, cung cấp session context cho mọi component.

### 4.2.3 Giao diện Landing Page

Landing page đã được implement hoàn chỉnh tại `Frontend/components/pages/HomePage.tsx`:

- **Navigation**: Logo, nút "Đăng nhập" (hiển thị khi chưa đăng nhập), nút "Dashboard" và `UserButton` của Clerk (hiển thị khi đã đăng nhập). Navigation sử dụng Clerk's `SignedIn`/`SignedOut` components để conditional rendering.
- **Hero Section**: Tiêu đề lớn với gradient text, mô tả ngắn về sản phẩm, CTA button dẫn đến trang Scan (cho user đã đăng nhập) hoặc Sign-in (cho user mới).
- **Feature Cards**: 3 card giới thiệu tính năng chính: Scan thông minh, AI Generate, Cookbook cá nhân — với hiệu ứng hover và glass-morphism design.
- **Design system**: Dark theme với palette zinc/brand, gradient background, blur effects, smooth hover transitions.

### 4.2.4 Backend Services & Schemas

Lớp Backend đã có nền tảng vững chắc:

- **`Backend/schemas/recipe.schema.ts`**: Zod validation schema đầy đủ cho `CreateRecipeSchema` (title, description, ingredients array, instructions array, cookTime, servings, imageUrl, cookbookId, source enum) và `UpdateRecipeSchema` (partial version).
- **`Backend/services/recipe.service.ts`**: 5 hàm CRUD hoàn chỉnh: `getRecipesByUser`, `getRecipeById`, `createRecipe`, `updateRecipe`, `deleteRecipe` — tất cả đều có ownership check, tránh user A truy cập recipe của user B.

### 4.2.5 AI Agent Files

Cấu trúc AI agents đã được tạo sẵn với 4 files agent và 1 workflow:

- `AI/agents/orchestrations.agent.ts` — Orchestrator điều phối pipeline
- `AI/agents/vision.agent.ts` — Giao tiếp Google Cloud Vision API
- `AI/agents/recipe.agent.ts` — Giao tiếp Claude Sonnet 4.5
- `AI/agents/storage.agent.ts` — Lưu trữ recipe qua Backend service
- `AI/workflows/scan-generative-save.ts` — Entry point workflow

### 4.2.6 API Quét Ảnh Nhận Diện Nguyên Liệu Thông Minh (Gemini AI integration)

Xây dựng endpoint API `/api/scan` tích hợp trực tiếp SDK Google Gemini 2.5 Flash / 1.5 Flash:
- Tiếp nhận hình ảnh dạng Base64 từ Frontend.
- Thiết kế Prompt Engineering tối ưu để AI phân tích chính xác nguyên liệu bằng tiếng Việt, tự động trả về định dạng JSON chuẩn chứa: Tên món ăn gợi ý, mô tả, danh sách nguyên liệu, các bước nấu nướng chi tiết và từ khóa Unsplash liên quan.

### 4.2.7 Đồng bộ người dùng tự động và Tự khởi tạo Cookbook mặc định

Giải quyết triệt để rào cản xác thực và đồng bộ dữ liệu:
- Khi người dùng đăng nhập bằng Clerk và thực hiện quét, backend tự động kiểm tra và chèn thông tin tài khoản `userId` vào bảng `User` nội bộ nếu chưa có.
- Tự động kiểm tra và tạo mới Cookbook mặc định `"📖 Công thức yêu thích"` khi người dùng lưu món ăn đầu tiên, loại bỏ hoàn toàn các lỗi mâu thuẫn quan hệ khóa ngoại (foreign key constraints).

### 4.2.8 Lưu trữ ảnh gốc cục bộ và thư viện Emoji đa dạng

Tối ưu hóa trải nghiệm thị giác sống động:
- Viết API chuyển đổi dữ liệu ảnh Base64 thành file ảnh tĩnh vật lý lưu trong thư mục `/public/uploads/` trên máy chủ cục bộ, giúp hiển thị chính xác bức ảnh gốc mà người dùng vừa tải lên/chụp.
- Xây dựng thuật toán phân tích từ khóa nguyên liệu tiếng Việt để tự động gán Emoji phong phú tương ứng (`🍚` cơm, `🐷` thịt lợn, `🥕` cà rốt, `🥬` rau xanh, `🥚` trứng, v.v.), mang lại giao diện sống động và bắt mắt.

---

## 4.3 Kết Quả Đạt Được

### 4.3.1 Kiến trúc vững chắc

Thành công lớn nhất của dự án trong giai đoạn này là thiết lập được **kiến trúc MSA chuẩn** với Strict 3-Folder Rule được thực thi nghiêm ngặt. Toàn bộ team (và AI agents) đều tuân thủ quy tắc này — không có file nào vi phạm boundary giữa các layer sau khi có cơ chế kiểm soát qua skill cards và context engineering.

### 4.3.2 Developer Experience (DX) tốt

- Path aliases (`@/frontend/*`, `@/backend/*`, `@/ai/*`) hoạt động xuyên suốt, giảm relative import dài dòng.
- Prisma type-safety: tất cả database query đều được TypeScript check tại compile time.
- Zod validation: lỗi input được bắt tại boundary trước khi đến service, error message rõ ràng.

### 4.3.3 Hệ thống Context Engineering

Song song với codebase, nhóm đã xây dựng hệ thống **smart-kitchen-prompts** — một Context Engineering Repository gồm:
- 4 context files (`CLAUDE.md`, `AGENTS.md`, `ERD.md`, `FLOW.md`) làm "ROM" cho AI agents
- 6 skill cards định nghĩa vai trò và quy tắc từng agent
- Backlog 81 tasks phân loại theo priority (P0 → P3)
- Changelog theo dõi lịch sử thay đổi

---

## 4.4 Khó Khăn và Hướng Giải Quyết

### Vấn đề 1: Clerk API thay đổi giữa các phiên bản

**Lỗi gặp phải**: `auth(...).protect is not a function` tại `middleware.ts`.  
**Nguyên nhân**: `@clerk/nextjs` v5+ thay đổi API — `auth` bây giờ là plain object, không phải function.  
**Giải pháp**: Đổi `auth().protect()` → `auth.protect()`. Ghi lại vào CHANGELOG để tránh tái diễn.

### Vấn đề 2: Next.js không hỗ trợ TypeScript config file

**Lỗi gặp phải**: `Error: Configuring Next.js via 'next.config.ts' is not supported`.  
**Nguyên nhân**: Next.js 14 chỉ hỗ trợ `.mjs` hoặc `.js`, TypeScript config chỉ từ Next.js 15+.  
**Giải pháp**: Tạo mới `next.config.mjs` với JSDoc type annotation thay vì TypeScript import.

### Vấn đề 3: Prisma connection pool trong môi trường serverless

**Vấn đề**: Mỗi API route tạo Prisma Client mới → cạn kiệt connection pool.  
**Giải pháp**: Singleton pattern trong `Backend/db/client.ts` — kiểm tra `global.prisma` trước khi tạo mới, cache instance vào global object trong development mode.

### Vấn đề 4: Quản lý ngữ cảnh AI trong nhóm

**Vấn đề**: Các thành viên và AI agents thiếu context chung → code inconsistent, vi phạm kiến trúc.  
**Giải pháp**: Xây dựng hệ thống `smart-kitchen-prompts` với context files, skill cards, và quy trình Vibe Coding (Analyse → Code → Test) bắt buộc cho mọi task.

### Vấn đề 5: Zod Schema kiểm duyệt chặn đường dẫn tệp ảnh nội bộ

**Lỗi gặp phải**: Lỗi validate ZodError chặn không cho lưu công thức nấu ăn vào Cookbook cục bộ.  
**Nguyên nhân**: Zod Schema tại `recipe.schema.ts` thiết lập kiểm duyệt thuộc tính `imageRecipe` bằng định dạng URL tuyệt đối (`z.string().url()`). Trong khi đó, ảnh gốc do người dùng tải lên được lưu nội bộ trên server dưới dạng relative path `/uploads/...`. Do đó, Zod ném ra lỗi ZodError chặn hành động tạo bản ghi.  
**Giải pháp**: Nới lỏng kiểm duyệt Zod cho `imageRecipe` sang `z.string().max(1000)` để tương thích hoàn toàn cả ảnh cục bộ lẫn ảnh mạng tuyệt đối, đồng thời bọc try-catch an toàn quanh logger để tránh Node.js crash.

### Vấn đề 6: Giới hạn hạn mức API của mô hình Gemini (429 Rate Limit) và lỗi cú pháp khóa

**Lỗi gặp phải**: Quét ảnh thất bại với lỗi `429 RESOURCE_EXHAUSTED` hoặc `400 API Key Not Valid`.  
**Nguyên nhân**: Google AI Studio giới hạn 20 yêu cầu/ngày trên dải IP dùng chung đối với mô hình `gemini-2.5-flash` ở gói miễn phí. Đồng thời, khóa API bị lỗi cú pháp dán dính ký tự ngắt dòng `\n` trong file `.env`.  
**Giải pháp**: Dọn dẹp dứt điểm file `.env` về chuẩn dòng đơn. Ở Backend, tối ưu hóa thuật toán tự động chuyển đổi linh hoạt sang mô hình `gemini-1.5-flash` để mở rộng băng thông hạn mức độc lập khi bị rate limit, đồng thời hiển thị hướng dẫn chi tiết tiếng Việt (mẹo dùng 4G, đổi Wifi, hoặc bật/tắt VPN) để người dùng chủ động xử lý sự cố IP.

---

## 4.5 Demo Giao Diện

### Màn hình Landing Page

Trang chủ hiển thị trên nền dark gradient (zinc-950 → zinc-900), với:
- Navigation bar trong suốt (glassmorphism) ở top với logo "🍳 Smart Kitchen"
- Hero section center-aligned với badge "✨ Powered by Claude Sonnet 4.5 & Google Vision"
- Tiêu đề lớn với gradient text cam-vàng: "Chụp nguyên liệu, AI lo phần còn lại"
- 3 feature cards bên dưới với hiệu ứng hover border glow

### Màn hình Sign-in / Sign-up

Trang xác thực sử dụng Clerk UI Components sẵn có — form đăng nhập/đăng ký với Google OAuth, email/password. Clerk tự động handle UI, validation và flow.

### Màn hình Dashboard và AI Scan (Đã hoàn thiện 100%)

Dashboard và trang quét nguyên liệu đã hoạt động hoàn chỉnh:
- **Scan Live**: Cho phép tải ảnh lên từ camera/máy tính, hiển thị preview thời gian thực.
- **AI Đề xuất**: Nhận diện nguyên liệu cực nhạy bằng Gemini AI, gợi ý món ăn kèm Emoji phong phú cực kỳ trực quan (`🍚`, `🐷`, `🥕`, `🥚`).
- **Cookbook Integration**: Nút **"Lưu vào Cookbook"** lưu trữ món ăn kèm ảnh gốc siêu nét cục bộ tức thì, tự động chuyển hướng người dùng sang trang chi tiết món ăn trong vòng chưa đầy 1 giây.
