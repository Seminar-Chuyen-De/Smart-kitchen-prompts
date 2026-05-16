<!-- Lịch sử thay đổi prompt/context -->

# CHANGELOG — Smart Kitchen Web

---

## [0.3.0] — 2026-05-16 — Documentation: README.md chi tiết & chuyên nghiệp

### 📖 README.md Authored

**Mục tiêu:** Viết README.md hoàn chỉnh cho repo `smart-kitchen-prompts` — tài liệu "lời chào đầu tiên" phục vụ cả AI agent lẫn developer.

**Nội dung README bao gồm:**
- **Mục đích repo**: Giải thích rõ đây là Context Engineering Repository, không phải code repo.
- **Cấu trúc thư mục**: Tree view đầy đủ với mô tả vai trò từng file/folder.
- **Hướng dẫn AI Agent**: Thứ tự đọc file bắt buộc, nguyên tắc hoạt động.
- **Hướng dẫn Developer**: Cách giao task đúng cách, cách thêm skill/prompt mới.
- **Multi-Agent System Table**: Bảng tóm tắt 6 agents + Orchestrator flow diagram.
- **Quy ước đặt tên**: Naming convention cho skill cards, prompts, context files.
- **Versioning CHANGELOG**: Quy tắc tăng phiên bản patch/minor/major.
- **Liên kết repo liên quan**: 4 repos trong hệ thống Smart Kitchen.

### 🤖 Actionable Prompt (theo skill 06)

Prompt được sinh bởi `[AGENT-PROMPT-ENGINEER]` và lưu tại:
`Smart-kitchen-prompts/prompts/readme-documentation.md`

**MỤC TIÊU:** Viết README.md chi tiết cho `smart-kitchen-prompts` phục vụ cả AI lẫn developer.

**CÁC BƯỚC:**
- BƯỚC 1: Phân tích cấu trúc repo thực tế (context/, skills/, prompts/, mcp-config/, experience/).
- BƯỚC 2: Viết 8 phần theo thứ tự: Header → Mục đích → Cấu trúc → Hướng dẫn AI → Hướng dẫn Dev → MSA table → Quy ước → Liên kết.
- BƯỚC 3: Overwrite `README.md` với nội dung mới.
- BƯỚC 4: Cập nhật `CHANGELOG.md`.

**RÀNG BUỘC:** Nội dung phải phản ánh đúng cấu trúc thực tế, viết bằng tiếng Việt + tiếng Anh cho thuật ngữ kỹ thuật.

---

## [0.2.0] — 2026-05-16 — Context Engineering: CLAUDE.md + ERD.md + FLOW.md



### 📚 Context Files Authored

**Mục tiêu:** Xây dựng bộ "ROM" đầy đủ cho toàn hệ thống AI agent, giúp Claude Code và các agent khác có đủ ngữ cảnh để làm việc chính xác ngay từ đầu.

#### [CLAUDE.md](../../../smart-kitchen/smart-kitchen-prompts/context/CLAUDE.md) — Global Rules

Nội dung bao gồm:
- **Project identity**: Smart Kitchen VN, AI-powered cooking app.
- **STRICT 3-FOLDER RULE**: Quy tắc bắt buộc cho `app/`, `frontend/`, `backend/`, `ai/`.
- **Path Aliases**: `@/frontend/*`, `@/backend/*`, `@/ai/*`, `@/prisma`.
- **Tech Stack table**: Next.js 14, Tailwind, PostgreSQL, Prisma, Clerk, Claude Sonnet 4.5, Google Vision.
- **Multi-Agent Architecture (MSA)**: Orchestrator → Vision → Recipe → Storage.
- **DO NOT / MUST DO rules**: Các ràng buộc cứng khi code.
- **Vibe Coding Workflow**: Cycle Analyse → Code → Test.

#### [ERD.md](../../../smart-kitchen/smart-kitchen-prompts/context/ERD.md) — Database Schema

Nội dung bao gồm:
- **ASCII ERD diagram** toàn bộ 8 bảng: `users`, `recipes`, `cookbooks`, `cookbook_recipes`, `steps`, `tags`, `recipe_tags`, `ingredients`, `recipe_ingredients`.
- **Mô tả chi tiết từng bảng**: columns, types, constraints, mô tả.
- **Enum `source_type`**: `MANUAL | IMPORTED | AI_GENERATED`.
- **Cascade delete rules** cho tất cả relations.
- Source of truth: `smart-kitchen-api/prisma/schema.prisma`.

#### [FLOW.md](./context/FLOW.md) — Luồng Nghiệp Vụ

Nội dung bao gồm:
- **Flow 1 — AI Recipe Generation**: Upload ảnh → Vision Agent → Recipe Agent → Storage Agent → Lưu DB.
- **Flow 2 — Manual Recipe Creation**: User form → Zod validate → recipe.service → PostgreSQL.
- **Flow 3 — Cookbook Management**: Tạo/xóa cookbook, thêm/xóa recipe vào cookbook.
- **Flow 4 — Authentication (Clerk)**: Webhook sync, middleware guard.
- **Flow 5 — Recipe Search & Filter**: Query theo tên, tag, userId.
- **API Endpoints table**: Tổng hợp 10+ endpoints.
- **State machine** cho Recipe status.
- **Ghi chú nghiệp vụ**: Cascade, nutrition auto-calc, tags seeded.

### 🤖 Prompt Kỹ Thuật Sử Dụng (theo skill 06-Agent-prompt.md)

**MỤC TIÊU:** Tạo bộ context engineering hoàn chỉnh cho hệ thống Smart Kitchen VN, gồm 3 file `CLAUDE.md`, `ERD.md`, `FLOW.md`, dựa trên `AGENTS.md` và schema Prisma thực tế của dự án.

**PHÂN TÍCH KIẾN TRÚC & CÁC BƯỚC THỰC THI:**
- BƯỚC 1: Đọc `AGENTS.md` (cả 2 repo) + `schema.prisma` + `06-Agent-prompt.md` để hiểu context.
- BƯỚC 2: Viết `CLAUDE.md` — "ROM" global rules cho mọi AI agent.
- BƯỚC 3: Viết `ERD.md` — Schema từ Prisma schema thực tế.
- BƯỚC 4: Viết `FLOW.md` — 5 user flows chính của hệ thống.
- BƯỚC 5: Cập nhật `CHANGELOG.md` để ghi lại lịch sử.

**RÀNG BUỘC:**
- Nội dung phải phản ánh đúng codebase thực tế (không bịa đặt).
- Dùng bảng Markdown, ASCII diagram, và code blocks để dễ đọc.

---

## [0.1.2] — 2026-05-16 — Bugfix: `auth().protect is not a function` (Clerk v6)

### 🐛 Bug
- **Lỗi:** `auth(...).protect is not a function` tại `middleware.ts:7`
- **Nguyên nhân:** `@clerk/nextjs` v5+ thay đổi API của `clerkMiddleware`.  
  Tham số `auth` bây giờ là **plain object**, không còn là function nữa.

### ✅ Fix — `middleware.ts`
```diff
- auth().protect();
+ auth.protect();
```

### 📝 Bài học
> **Rule:** Với `@clerk/nextjs` v5+, trong `clerkMiddleware` callback: `auth` là object → dùng `auth.protect()`, không phải `auth().protect()`.

---

## [0.1.1] — 2026-05-16 — Bugfix: next.config.ts không được hỗ trợ

### 🐛 Bug
- **Lỗi:** `Error: Configuring Next.js via 'next.config.ts' is not supported.`
- **Nguyên nhân:** Next.js 14.x chỉ hỗ trợ `next.config.js` hoặc `next.config.mjs`. TypeScript config (`.ts`) chỉ có từ Next.js 15+.

### ✅ Fix
- Tạo mới `next.config.mjs` (ESM, dùng JSDoc `@type` thay vì TypeScript import).
- Deprecated `next.config.ts` (giữ lại với comment cảnh báo).

### 📝 Bài học
> **Rule:** Với Next.js 14, LUÔN dùng `next.config.mjs` hoặc `next.config.js`. Không bao giờ dùng `.ts`.

---

## [0.1.0] — 2026-05-16 — Foundation Setup

### 🚀 Khởi tạo Foundation — Config → DB → Backend → Frontend → App

| File | Layer | Mục đích |
|---|---|---|
| `package.json` | Config | Tất cả dependencies |
| `tsconfig.json` | Config | Full strict mode + path aliases |
| `next.config.mjs` | Config | Image domains, server packages |
| `tailwind.config.ts` | Config | Brand palette, content paths |
| `postcss.config.js` | Config | Required by Tailwind |
| `.env.example` | Config | Template biến môi trường |
| `middleware.ts` | App | Clerk auth guard |
| `app/layout.tsx` | App | Root layout + ClerkProvider |
| `app/page.tsx` | App | 3 dòng → delegate HomePage |
| `app/sign-in/[...]/page.tsx` | App | Clerk sign-in |
| `app/sign-up/[...]/page.tsx` | App | Clerk sign-up |
| `prisma/schema.prisma` | DB | User, Cookbook, Recipe, ScanLog |
| `prisma/seed.ts` | DB | Demo data (idempotent) |
| `Backend/db/client.ts` | Backend | Prisma singleton |
| `Backend/schemas/recipe.schema.ts` | Backend | Zod validation |
| `Backend/services/recipe.service.ts` | Backend | CRUD + ownership check |
| `Frontend/styles/globals.css` | Frontend | Dark theme + utilities |
| `Frontend/components/pages/HomePage.tsx` | Frontend | Landing page UI |