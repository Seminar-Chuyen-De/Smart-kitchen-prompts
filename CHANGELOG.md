<!-- Lịch sử thay đổi prompt/context -->

# CHANGELOG — Smart Kitchen Web

---

## [0.5.0] — 2026-05-18 — feat: P2 UX Enhancement — Shared UI, Error Pages, Profile

### 🔀 Git Workflow — [AGENT-GIT-WORKFLOW]

**Mục tiêu**: Hợp nhất toàn bộ công việc đã làm trên các nhánh riêng lẻ vào một nhánh chức năng mới trước khi bắt đầu P2.

| Nhánh nguồn | Nhánh đích | Nội dung |
|---|---|---|
| `origin/feature/database-schema` | `feature/p2-ux-enhancement` | DB schema, Zod schemas, seed, service layer |
| `origin/feature/api-endpoints-tests` | `feature/p2-ux-enhancement` | API routes, unit tests, user.service |
| `origin/feature/p0-p1-core-ui` | `feature/p2-ux-enhancement` | Toàn bộ P0-2 + P1 UI components |

**Bonus fix**: Cập nhật `.gitignore` để loại `.next/`, `tsconfig.tsbuildinfo` và các build artifacts khỏi git tracking.

---

### 🎨 P2-1 · Shared UI Components — [AGENT-UI]

**Mục tiêu**: Xây dựng bộ primitive UI components tái sử dụng làm nền tảng cho toàn bộ dashboard.

| File | Mô tả |
|---|---|
| `Frontend/components/ui/Button.tsx` | 4 variants (primary/secondary/ghost/danger), 3 sizes, `isLoading` spinner, `leftIcon`/`rightIcon`, `forwardRef` |
| `Frontend/components/ui/Badge.tsx` | 5 variants: `ai` (violet), `manual` (blue), `pending` (amber), `success` (green), `error` (red) |
| `Frontend/components/ui/Skeleton.tsx` | 3 variants: `text`, `card`, `avatar` — animation pulse, prop `count` để render nhiều |
| `Frontend/components/ui/EmptyState.tsx` | Icon + title + description + optional `action` button |
| `Frontend/components/ui/Modal.tsx` | Backdrop blur, đóng bằng ESC/click-outside, 4 sizes (sm/md/lg/full), `aria-modal` |
| `Frontend/components/ui/Toast.tsx` | Fixed bottom-right, slide-in animation, 4 loại có màu riêng, nút dismiss |
| `Frontend/contexts/ToastContext.tsx` | `useReducer` state, auto-remove sau 4000ms, export `useToast()` hook và `ToastProvider` |

**Tích hợp**: `app/dashboard/layout.tsx` được cập nhật để bọc `ToastProvider` + render `ToastContainer`.

---

### 🛡️ P2-2 · Error Handling & Loading States — [AGENT-UI]

**Mục tiêu**: Cung cấp UX nhất quán khi xảy ra lỗi hoặc đang tải dữ liệu ở cấp ứng dụng.

| File | Mô tả |
|---|---|
| `app/loading.tsx` | Global loading skeleton với thương hiệu Smart Kitchen (icon 🍳 + Skeleton component) |
| `app/not-found.tsx` | 404 page: `UtensilsCrossed` icon, text "404", 2 navigation buttons (Dashboard / Trang chủ) |
| `app/error.tsx` | Next.js Error Boundary (`'use client'`): nhận `error.digest`, nút "Thử lại" gọi `reset()` |

---

### 👤 P2-4 · User Profile Page — [AGENT-UI]

**Mục tiêu**: Trang hồ sơ người dùng dùng dữ liệu từ Clerk, với stat cards và modal chỉnh sửa.

| File | Mô tả |
|---|---|
| `Frontend/components/pages/ProfilePage.tsx` | Dùng `useUser()` Clerk: avatar, tên, email, ngày tham gia, stat cards (recipe/cookbook), edit Modal |
| `app/dashboard/profile/page.tsx` | Thin shell + `export const metadata` |

> **Ghi chú**: Stat cards hiển thị giá trị `0` với comment `// TODO: fetch từ /api/user/profile`. Form edit chưa submit API — chờ P2-4 API layer.

---

### 📋 Prompt Engineering

- Thêm `prompts/p2-ux-enhancement-ui.md` — Actionable Prompt theo format `06-Agent-prompt.md`, gồm 2 phần:
  - **PHẦN 1** — `[AGENT-UI]`: 3 bước thực thi với spec chi tiết từng component
  - **PHẦN 2** — `[AGENT-GIT-WORKFLOW]`: Script merge 3 nhánh + script push cuối

### ✅ Verification

- Dev server `npm run dev` → **Chạy thành công**
- `app/error.tsx`, `app/loading.tsx`, `app/not-found.tsx` → hiển thị trong `.next/app-build-manifest.json` ✅
- `/dashboard/profile` → route render đúng với Clerk user data ✅
- `ToastProvider` + `ToastContainer` tích hợp vào dashboard layout ✅

---

## [0.4.0] — 2026-05-18 — feat: P0-2 + P1 Core Features UI hoàn chỉnh

### 🎨 Frontend Layer — [AGENT-UI]

**Mục tiêu**: Xây dựng toàn bộ giao diện (UI layer) cho Dashboard, AI Scan Flow, Recipe CRUD và Cookbook Management. Tất cả tuân thủ STRICT 3-FOLDER RULE — `app/` chỉ là routing shells siêu mỏng, toàn bộ UI nằm trong `Frontend/`.

**Dependency đã cài thêm:** `lucide-react` (icons)

#### P0-2 · Dashboard Layout & Route

| File | Mô tả |
|---|---|
| `Frontend/components/layout/Navbar.tsx` | Fixed navbar: logo, nav links có active state, UserButton Clerk, hamburger mobile |
| `Frontend/components/layout/Sidebar.tsx` | Mobile slide-in sidebar tích hợp trong Navbar |
| `app/dashboard/layout.tsx` | Clerk `auth()` guard + Navbar wrapper |
| `app/dashboard/page.tsx` | Thin shell → `DashboardPage` |
| `Frontend/components/pages/DashboardPage.tsx` | Stat cards (recipes/cookbooks/scan), Quick Actions, Recent Recipes grid, Skeleton loading |

#### P1-1 · AI Scan Flow UI

| File | Mô tả |
|---|---|
| `Frontend/hooks/useScan.ts` | State machine: idle → uploading → scanning → done/error. Gọi `POST /api/scan` |
| `Frontend/components/ai/ImageUploader.tsx` | Drag & drop zone, file preview, progress bar animation, status messages |
| `Frontend/components/ai/ScanResultCard.tsx` | Hiển thị recipe AI: badge, ingredients chips, steps preview, nút Lưu/Scan lại. Export `ScanResultSkeleton` |
| `Frontend/components/pages/ScanPage.tsx` | Layout 2 cột (uploader \| result), How-it-works guide, error state |
| `app/dashboard/scan/page.tsx` | Thin shell → `ScanPage` |

#### P1-2 · Recipe CRUD UI

| File | Mô tả |
|---|---|
| `Frontend/hooks/useRecipes.ts` | Types đầy đủ (Recipe, Step, Tag, Ingredient), CRUD với optimistic updates, mock data cho dev |
| `Frontend/components/recipe/RecipeCard.tsx` | Card: image/fallback, source badge, hover actions (edit/delete/cookbook), tags. Export `RecipeCardSkeleton` |
| `Frontend/components/recipe/RecipeList.tsx` | Grid/List toggle view, skeleton loading (6 cards), empty state với CTA |
| `Frontend/components/recipe/RecipeDetail.tsx` | Full view: hero image, nutrition grid (4 cards), ingredients, numbered steps, tags. Export `RecipeDetailSkeleton` |
| `Frontend/components/recipe/RecipeForm.tsx` | Form tạo/sửa: dynamic ingredient rows, step list, nutrition fields, image URL preview |
| `Frontend/components/pages/RecipesPage.tsx` | Search bar + source filter, delete confirmation dialog, dùng `useRecipes` |
| `Frontend/components/pages/RecipeDetailPage.tsx` | Fetch by ID, loading/error states, mock data fallback |
| `app/dashboard/recipes/page.tsx` | Thin shell → `RecipesPage` |
| `app/dashboard/recipes/[id]/page.tsx` | Thin shell → `RecipeDetailPage` |
| `app/dashboard/recipes/new/page.tsx` | Full page: dùng `RecipeForm`, xử lý create & redirect |

#### P1-3 · Cookbook Management UI

| File | Mô tả |
|---|---|
| `Frontend/hooks/useCookbooks.ts` | Cookbook type + CRUD + addRecipe/removeRecipe, optimistic updates, mock data |
| `Frontend/components/cookbook/CookbookCard.tsx` | Card: mosaic image preview hoặc icon, recipe count, hover edit/delete. Export `CookbookCardSkeleton` |
| `Frontend/components/cookbook/CookbookList.tsx` | Grid với skeleton (3 cards), empty state |
| `Frontend/components/cookbook/CookbookDetail.tsx` | Inline name edit, add-recipe search modal, remove recipe per card, delete cookbook confirm |
| `Frontend/components/pages/CookbooksPage.tsx` | Inline create form, edit-name modal, delete confirm |
| `app/dashboard/cookbooks/page.tsx` | Thin shell → `CookbooksPage` |
| `app/dashboard/cookbooks/[id]/page.tsx` | Thin shell → `CookbookDetail` |

### 📋 Prompt Engineering

- Thêm `prompts/p1-ui-core-features.md` — Actionable Prompt đầy đủ cho [AGENT-UI] bao gồm spec chi tiết từng component, constraints, và thứ tự thực hiện.

### ✅ Verification

- `npx tsc --noEmit` → **0 errors**
- Dev server `npm run dev` → **Ready in 2.6s**
- `/dashboard` → Clerk auth guard hoạt động, redirect sign-in ✅
- Tất cả routes render đúng với mock data ✅

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