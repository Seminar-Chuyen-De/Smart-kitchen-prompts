# 📋 BACKLOG — Smart Kitchen Web

> **Phân tích thực hiện bởi**: [AGENT-MANAGE] / Orchestrator
> **Ngày phân tích**: 2026-05-16
> **Dựa trên prompt**: `prompts/backlog-analysis.md`
> **Codebase**: `smart-kitchen-web` (Next.js 14, Clerk, PostgreSQL, Claude Sonnet 4.5, Google Vision)

---

## 🗂️ Trạng Thái Hiện Tại (Snapshot)

| Layer | Đã có | Còn thiếu |
|---|---|---|
| **Config** | Đầy đủ (tsconfig, tailwind, next, prisma, vitest.config.ts) | — |
| **DB Schema** | Đầy đủ (User, Cookbook, CookbookRecipe, Step, Ingredient, RecipeIngredient, Tag, RecipeTag, ScanLog) | — |
| **Backend/services** | recipe.service.ts (Nested CRUD chuẩn ERD) | cookbook.service.ts, user.service.ts, scan.service.ts |
| **Backend/schemas** | Đầy đủ (recipe, user, ingredient, tag, cookbook, step) | scan.schema.ts |
| **app/api** | ❌ Chưa có route nào | Toàn bộ API routes |
| **app/(routes)** | layout, page, sign-in, sign-up | /dashboard và toàn bộ sub-routes |
| **Frontend/components** | HomePage.tsx | Dashboard, Recipe, Cookbook, AI Scan, Layout |
| **Frontend/hooks** | ❌ Trống | useRecipes, useCookbooks, useScan |
| **AI/agents** | 4 files (nội dung cần verify) | Cần implement đầy đủ logic |
| **AI/workflows** | scan-generative-save.ts (cần verify) | — |
| **Tests** | Đã có 5/5 unit tests cho recipe service | Toàn bộ các test cases còn lại |

---

## 🚨 P0 — Blocking (Làm trước, phải xong mới có thể test feature)

### P0-1 · Clerk Webhook — Sync User vào DB

**Mục tiêu**: Khi user đăng ký qua Clerk, tự động tạo record trong bảng `users`.**Agent**: `[AGENT-DB]` → `[AGENT-API]`**Dependency**: Không

- [ ] **[AGENT-DB]** Viết `Backend/services/user.service.ts`:
  - `upsertUser(clerkId, email, name, avatarUrl)`
  - `getUserById(clerkId)`
- [ ] **[AGENT-API]** Tạo `app/api/webhooks/clerk/route.ts`:
  - Verify Clerk webhook signature (dùng `svix`)
  - Handle event `user.created` → gọi `upsertUser`
  - Handle event `user.updated` → update user info

---

### P0-2 · Dashboard Layout & Route

**Mục tiêu**: Tạo layout cho authenticated area (`/dashboard`). HomePage đang link đến `/dashboard` nhưng route chưa tồn tại.**Agent**: `[AGENT-UI]` → `[AGENT-API]`**Dependency**: P0-1

- [x] **[AGENT-UI]** Tạo `Frontend/components/layout/Navbar.tsx` (dashboard nav):
  - Logo, link Recipe/Cookbook, UserButton Clerk
- [x] **[AGENT-UI]** Tạo `Frontend/components/layout/Sidebar.tsx` (optional mobile nav)
- [x] **[AGENT-UI]** Tạo `app/dashboard/layout.tsx`:
  - Clerk `auth().protect()` guard
  - Import Navbar
- [x] **[AGENT-UI]** Tạo `app/dashboard/page.tsx` → import `DashboardPage`
- [x] **[AGENT-UI]** Tạo `Frontend/components/pages/DashboardPage.tsx`:
  - Summary cards: số recipe, số cookbook, lần scan gần nhất
  - Quick actions: "Scan mới", "Xem Cookbook"

---

## 🔥 P1 — Core Features (Tính năng cốt lõi)

### P1-1 · AI Scan Flow (Feature chính)

**Mục tiêu**: User upload ảnh → Vision API nhận diện → Claude generate recipe → lưu DB.
**Agent**: `[AGENT-DB]` → `[AGENT-API]` → `[AGENT-UI]`
**Dependency**: P0-1, P0-2

#### Backend Layer

- [ ] **[AGENT-DB]** Implement `AI/agents/vision.agent.ts`:
  - Nhận `imageBuffer` hoặc `imageUrl`
  - Gọi Google Cloud Vision API (label detection)
  - Trả về `string[]` ingredients
- [ ] **[AGENT-DB]** Implement `AI/agents/recipe.agent.ts`:
  - Nhận `string[]` ingredients
  - Gọi Claude Sonnet 4.5 (Anthropic SDK) với prompt từ `AI/prompts/`
  - Trả về `RecipeDTO { title, description, ingredients, instructions, cookTime, servings }`
- [ ] **[AGENT-DB]** Implement `AI/agents/storage.agent.ts`:
  - Nhận `RecipeDTO` + `userId`
  - Gọi `Backend/services/recipe.service.ts` → `createRecipe()`
  - Trả về `recipe_id`
- [ ] **[AGENT-DB]** Implement `AI/agents/orchestrations.agent.ts`:
  - Orchestrate: vision → recipe → storage
  - Xử lý lỗi từng bước, trả về status rõ ràng
- [ ] **[AGENT-DB]** Implement `AI/workflows/scan-generative-save.ts`:
  - Entry point workflow gọi orchestrator
- [ ] **[AGENT-DB]** Tạo `AI/prompts/recipe-generation.ts`:
  - System prompt cho Claude generate recipe
- [ ] **[AGENT-DB]** Tạo `AI/types/recipe.types.ts`:
  - `RecipeDTO`, `ScanResult`, `AgentResponse` interfaces
- [ ] **[AGENT-DB]** Viết `Backend/services/scan.service.ts`:
  - `createScanLog(imageUrl, detectedItems, recipeId?)`
  - `getScansByUser(userId)`
- [ ] **[AGENT-DB]** Thêm `Backend/schemas/scan.schema.ts`:
  - `ScanRequestSchema` (validate upload request)

#### API Layer

- [ ] **[AGENT-API]** Tạo `app/api/scan/route.ts` (POST):
  - Nhận `multipart/form-data` (image file)
  - Gọi `AI/workflows/scan-generative-save.ts`
  - Trả về `{ recipe, scanLogId }`

#### Frontend Layer

- [x] **[AGENT-UI]** Tạo `Frontend/components/ai/ImageUploader.tsx`:
  - Drag & drop + click to upload
  - Preview ảnh sau khi chọn
  - Progress indicator khi đang scan
- [x] **[AGENT-UI]** Tạo `Frontend/components/ai/ScanResultCard.tsx`:
  - Hiển thị ingredients detected
  - Preview recipe được generate
  - Nút "Lưu vào Cookbook" / "Làm lại"
- [x] **[AGENT-UI]** Tạo `Frontend/hooks/useScan.ts`:
  - State management cho scan flow
  - Upload image, poll status, nhận kết quả
- [x] **[AGENT-UI]** Tạo `Frontend/components/pages/ScanPage.tsx`
- [x] **[AGENT-UI]** Tạo `app/dashboard/scan/page.tsx` → import `ScanPage`

---

### P1-2 · Recipe CRUD

**Mục tiêu**: User xem, tạo thủ công, sửa, xóa công thức.
**Agent**: `[AGENT-API]` → `[AGENT-UI]`
**Dependency**: P0-1, P0-2 (recipe.service.ts & schema đã có)

#### API Layer

- [ ] **[AGENT-API]** Tạo `app/api/recipes/route.ts`:
  - `GET` → `getRecipesByUser(userId)` → trả về list
  - `POST` → validate `CreateRecipeSchema` → `createRecipe(userId, data)`
- [ ] **[AGENT-API]** Tạo `app/api/recipes/[id]/route.ts`:
  - `GET` → `getRecipeById(id, userId)`
  - `PUT` → validate `UpdateRecipeSchema` → `updateRecipe(id, userId, data)`
  - `DELETE` → `deleteRecipe(id, userId)`

#### Frontend Layer

- [x] **[AGENT-UI]** Tạo `Frontend/components/recipe/RecipeCard.tsx`:
  - Ảnh, tên, thời gian, số khẩu phần, source badge (AI/Manual)
  - Hover actions: Edit, Delete, Add to Cookbook
- [x] **[AGENT-UI]** Tạo `Frontend/components/recipe/RecipeList.tsx`:
  - Grid/list view, skeleton loading, empty state
- [x] **[AGENT-UI]** Tạo `Frontend/components/recipe/RecipeDetail.tsx`:
  - Full recipe view: ingredients list, steps, nutrition info
- [x] **[AGENT-UI]** Tạo `Frontend/components/recipe/RecipeForm.tsx`:
  - Form tạo/sửa recipe thủ công
  - Dynamic ingredient fields, step fields
- [x] **[AGENT-UI]** Tạo `Frontend/hooks/useRecipes.ts`:
  - `useRecipes()` — fetch, create, update, delete với optimistic updates
- [x] **[AGENT-UI]** Tạo `Frontend/components/pages/RecipesPage.tsx`
- [x] **[AGENT-UI]** Tạo `Frontend/components/pages/RecipeDetailPage.tsx`
- [x] **[AGENT-UI]** Tạo `app/dashboard/recipes/page.tsx` → import `RecipesPage`
- [x] **[AGENT-UI]** Tạo `app/dashboard/recipes/[id]/page.tsx` → import `RecipeDetailPage`
- [x] **[AGENT-UI]** Tạo `app/dashboard/recipes/new/page.tsx` — form tạo recipe thủ công

---

### P1-3 · Cookbook Management

**Mục tiêu**: User tạo/quản lý cookbook, thêm/xóa recipe vào cookbook.
**Agent**: `[AGENT-DB]` → `[AGENT-API]` → `[AGENT-UI]`
**Dependency**: P1-2

#### Backend Layer

- [ ] **[AGENT-DB]** Viết `Backend/services/cookbook.service.ts`:
  - `getCookbooksByUser(userId)`
  - `getCookbookById(id, userId)` (include recipes)
  - `createCookbook(userId, data)`
  - `updateCookbook(id, userId, data)`
  - `deleteCookbook(id, userId)`
  - `addRecipeToCookbook(cookbookId, recipeId, userId)`
  - `removeRecipeFromCookbook(cookbookId, recipeId, userId)`
- [x] **[AGENT-DB]** Tạo `Backend/schemas/cookbook.schema.ts`:
  - `CreateCookbookSchema`, `UpdateCookbookSchema`

#### API Layer

- [ ] **[AGENT-API]** Tạo `app/api/cookbooks/route.ts`:
  - `GET` → list cookbooks
  - `POST` → tạo cookbook mới
- [ ] **[AGENT-API]** Tạo `app/api/cookbooks/[id]/route.ts`:
  - `GET`, `PUT`, `DELETE`
- [ ] **[AGENT-API]** Tạo `app/api/cookbooks/[id]/recipes/route.ts`:
  - `POST` → thêm recipe
  - `DELETE` → xóa recipe

#### Frontend Layer

- [x] **[AGENT-UI]** Tạo `Frontend/components/cookbook/CookbookCard.tsx`
- [x] **[AGENT-UI]** Tạo `Frontend/components/cookbook/CookbookList.tsx`
- [x] **[AGENT-UI]** Tạo `Frontend/components/cookbook/CookbookDetail.tsx`
- [x] **[AGENT-UI]** Tạo `Frontend/hooks/useCookbooks.ts`
- [x] **[AGENT-UI]** Tạo `Frontend/components/pages/CookbooksPage.tsx`
- [x] **[AGENT-UI]** Tạo `app/dashboard/cookbooks/page.tsx`
- [x] **[AGENT-UI]** Tạo `app/dashboard/cookbooks/[id]/page.tsx`

---

## 🎨 P2 — UX Enhancement (Trải nghiệm người dùng)

### P2-1 · Shared UI Components

**Dependency**: P0-2

- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/Toast.tsx` — Notification system
- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/Modal.tsx` — Reusable modal/dialog
- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/Skeleton.tsx` — Loading skeleton
- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/Button.tsx` — Unified button variants
- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/Badge.tsx` — Source type badge (AI/Manual)
- [ ] **[AGENT-UI]** Tạo `Frontend/components/ui/EmptyState.tsx` — Empty state placeholder
- [ ] **[AGENT-UI]** Tạo `Frontend/contexts/ToastContext.tsx` — Global toast state

### P2-2 · Error Handling & Loading States

**Dependency**: P1-1, P1-2, P1-3

- [ ] **[AGENT-UI]** Tạo `app/error.tsx` — Global error boundary
- [ ] **[AGENT-UI]** Tạo `app/not-found.tsx` — 404 page
- [ ] **[AGENT-UI]** Tạo `app/loading.tsx` — Global loading page
- [ ] **[AGENT-API]** Tạo `Backend/middleware/error-handler.ts`:
  - Chuẩn hóa format error response: `{ error: string, code: string }`
- [ ] **[AGENT-API]** Tạo `Backend/constants/error-codes.ts`:
  - Enum các error code: `UNAUTHORIZED`, `NOT_FOUND`, `VALIDATION_ERROR`, etc.

### P2-3 · Search & Filter Recipes

**Dependency**: P1-2

- [ ] **[AGENT-DB]** Cập nhật `Backend/services/recipe.service.ts`:
  - Thêm `searchRecipes(userId, query, filters)` với full-text search
- [ ] **[AGENT-API]** Cập nhật `app/api/recipes/route.ts`:
  - Thêm query params: `?q=`, `?source=`, `?cookbookId=`
- [ ] **[AGENT-UI]** Tạo `Frontend/components/recipe/RecipeSearch.tsx`:
  - Input tìm kiếm + filter dropdowns
- [ ] **[AGENT-UI]** Cập nhật `useRecipes.ts` — thêm search/filter state

### P2-4 · User Profile Page

**Dependency**: P0-1

- [ ] **[AGENT-API]** Tạo `app/api/user/profile/route.ts` (GET, PUT)
- [ ] **[AGENT-UI]** Tạo `Frontend/components/pages/ProfilePage.tsx`
- [ ] **[AGENT-UI]** Tạo `app/dashboard/profile/page.tsx`

---

## 🏭 P3 — Production Readiness

### P3-1 · Unit Tests (Backend)

**Dependency**: P0-1, P1-1, P1-2, P1-3

- [x] **[AGENT-TESTING]** Tạo `Backend/__test__/unit/services/recipe.service.test.ts` (Passed 5/5)
- [ ] **[AGENT-TESTING]** Tạo `Backend/__test__/cookbook.service.test.ts`
- [ ] **[AGENT-TESTING]** Tạo `Backend/__test__/user.service.test.ts`
- [ ] **[AGENT-TESTING]** Tạo `AI/__test__/vision.agent.test.ts` (mock Google Vision)
- [ ] **[AGENT-TESTING]** Tạo `AI/__test__/recipe.agent.test.ts` (mock Claude)

### P3-2 · E2E Tests (Frontend)

**Dependency**: P1-1, P1-2, P1-3

- [ ] **[AGENT-TESTING]** Tạo `__test__/e2e/auth.spec.ts` — Sign-in/sign-up flow
- [ ] **[AGENT-TESTING]** Tạo `__test__/e2e/recipe.spec.ts` — CRUD recipes
- [ ] **[AGENT-TESTING]** Tạo `__test__/e2e/scan.spec.ts` — Upload → generate → save flow
- [ ] **[AGENT-TESTING]** Tạo `__test__/e2e/cookbook.spec.ts` — Cookbook management

### P3-3 · SEO & Performance

**Dependency**: P1-2, P1-3

- [ ] **[AGENT-UI]** Thêm `generateMetadata()` cho từng route page
- [ ] **[AGENT-UI]** Tạo `app/sitemap.ts` — Dynamic sitemap
- [ ] **[AGENT-UI]** Tạo `app/robots.ts` — Robots.txt
- [ ] **[AGENT-UI]** Thêm `next/image` optimization cho recipe images
- [ ] **[AGENT-MANAGE]** Audit và tối ưu Prisma queries (N+1, index)

### P3-4 · CI/CD & Deploy

**Dependency**: P3-1, P3-2

- [ ] **[AGENT-MANAGE]** Tạo `.github/workflows/ci.yml` — Auto test on PR
- [ ] **[AGENT-MANAGE]** Cấu hình Vercel deployment
- [ ] **[AGENT-MANAGE]** Setup database migration script cho production
- [ ] **[AGENT-MANAGE]** Kiểm tra và bổ sung tất cả env variables vào `.env.example`

---

## 🗺️ Roadmap & Thứ Tự Thực Hiện

```
TUẦN 1 — Foundation
├── P0-1: Clerk Webhook (User sync)
├── P0-2: Dashboard Layout
└── P2-1: Shared UI Components

TUẦN 2 — Core AI Feature
├── P1-1-Backend: Vision Agent + Recipe Agent + Storage Agent + Orchestrator
└── P1-1-API: Scan API route

TUẦN 3 — Recipe & AI UI
├── P1-1-Frontend: ImageUploader, ScanResultCard, ScanPage
├── P1-2-API: Recipe CRUD routes
└── P1-2-Frontend: RecipeCard, RecipeList, RecipeDetail, useRecipes

TUẦN 4 — Cookbook & Polish
├── P1-3: Cookbook đầy đủ (Backend → API → Frontend)
├── P2-2: Error handling & Loading states
├── P2-3: Search & Filter
└── P2-4: User Profile

TUẦN 5 — Production
├── P3-1: Unit Tests
├── P3-2: E2E Tests
├── P3-3: SEO & Performance
└── P3-4: CI/CD & Deploy
```

---

## 📊 Summary

| Priority               | Số tasks     | Ước tính    |
| ---------------------- | ------------ | ----------- |
| 🚨 P0 — Blocking       | 8 tasks      | ~1 ngày     |
| 🔥 P1 — Core Features  | 35 tasks     | ~2 tuần     |
| 🎨 P2 — UX Enhancement | 20 tasks     | ~1 tuần     |
| 🏭 P3 — Production     | 18 tasks     | ~1 tuần     |
| **Tổng**               | **81 tasks** | **~5 tuần** |

---

## 🤖 Cách Dùng File Này

**Để bắt đầu một task:**

1. Chọn task theo thứ tự Priority (P0 → P1 → P2 → P3)
2. Xác định Agent phụ trách (`[AGENT-DB]`, `[AGENT-API]`, `[AGENT-UI]`, `[AGENT-TESTING]`)
3. Copy skill card tương ứng từ `Smart-kitchen-prompts/skills/`
4. Paste vào đầu conversation với AI + mô tả task cụ thể
5. Sau khi hoàn thành, đánh dấu `[x]` vào checkbox
6. Cập nhật `CHANGELOG.md`

**Để tạo Actionable Prompt:**

- Dùng `skills/06-Agent-prompt.md` để refine task trước khi giao cho AI

---

> **Cập nhật lần cuối**: 2026-05-16 | **Version**: 1.0.0
