# [AGENT-UI] — Actionable Prompt: P1 Core Features UI

> **Sinh bởi**: [AGENT-PROMPT-ENGINEER]
> **Ngày**: 2026-05-18
> **Skill áp dụng**: `skills/02-Agent-ui.md`
> **Dựa trên backlog**: `prompts/BACKLOG.md`

---

## MỤC TIÊU

Xây dựng **toàn bộ giao diện (UI layer)** cho các tính năng cốt lõi P1 của Smart Kitchen VN web app. Trước khi làm P1, cần kiểm tra P0 đã hoàn thành chưa.

**Flow tổng quát:**
1. User đăng nhập (Clerk) → vào `/dashboard`
2. Dashboard hiển thị summary cards + quick actions
3. User scan ảnh → AI generate recipe → lưu vào Cookbook
4. User xem/tạo/sửa/xóa công thức thủ công
5. User quản lý Cookbook (thêm/xóa recipe)

---

## ROLE & CONSTRAINTS (BẮT BUỘC)

```
# ROLE: [AGENT-UI] - Chuyên gia Frontend (React/Next.js Client)

STRICT RULES:
1. CHỈ tạo/sửa file trong thư mục `Frontend/` (components/, hooks/)
   - app/ chỉ tạo page.tsx / layout.tsx siêu mỏng (import component từ Frontend/)
2. Stack: Tailwind CSS + Lucide Icons + TypeScript (strict mode)
3. KHÔNG gọi Prisma hay AI API trực tiếp — mọi data qua custom hooks → fetch /api/*
4. Path alias: `@/frontend/*` (maps to ./Frontend/*)
5. Tên file: kebab-case.tsx | Tên component: PascalCase
6. Skeleton loading cho mọi async state
```

---

## KIỂM TRA P0 TRƯỚC KHI LÀM P1

Trước tiên, hãy kiểm tra các file sau đã tồn tại chưa. Nếu **chưa có**, phải tạo P0 trước:

### ✅ Checklist P0

| Task | File cần tồn tại | Hành động nếu thiếu |
|------|-----------------|---------------------|
| P0-2a | `Frontend/components/layout/Navbar.tsx` | **Tạo ngay** (xem spec bên dưới) |
| P0-2b | `Frontend/components/layout/Sidebar.tsx` | **Tạo ngay** (optional mobile nav) |
| P0-2c | `app/dashboard/layout.tsx` | **Tạo ngay** — Clerk `auth().protect()` + import Navbar |
| P0-2d | `app/dashboard/page.tsx` | **Tạo ngay** — import `DashboardPage` |
| P0-2e | `Frontend/components/pages/DashboardPage.tsx` | **Tạo ngay** — summary cards + quick actions |

> **Lưu ý P0-1** (Clerk Webhook `app/api/webhooks/clerk/route.ts`): Đây là backend task — [AGENT-UI] **bỏ qua**, nhưng cần biết rằng `userId` lấy từ Clerk session trong các API calls.

---

## PHÂN TÍCH KIẾN TRÚC & CÁC BƯỚC THỰC THI

> Làm **tuần tự** theo thứ tự dưới. Xong mỗi bước, dừng lại để test trên browser trước khi tiếp tục.

---

### BƯỚC 0 — P0 Dashboard Layout (nếu chưa có)

#### `Frontend/components/layout/Navbar.tsx`

```
Spec:
- Logo: "🍳 Smart Kitchen" (link về /)
- Navigation links: /dashboard, /dashboard/recipes, /dashboard/cookbooks, /dashboard/scan
- Active link styling (highlight current route)
- UserButton từ @clerk/nextjs (avatar + sign out)
- Responsive: hamburger menu trên mobile
- Design: dark glass morphism (bg-zinc-900/80 backdrop-blur), border-b border-zinc-800
```

#### `Frontend/components/layout/Sidebar.tsx` (optional mobile)

```
Spec:
- Mobile slide-in sidebar với overlay
- Các link: Dashboard, Recipes, Cookbooks, Scan
- Trigger: hamburger icon trong Navbar
- Đóng khi click overlay hoặc link
```

#### `app/dashboard/layout.tsx` (routing shell)

```tsx
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";
import { Navbar } from "@/frontend/components/layout/Navbar";

export default async function DashboardLayout({ children }: { children: React.ReactNode }) {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");
  return (
    <div className="min-h-screen bg-zinc-950">
      <Navbar />
      <main className="pt-16">{children}</main>
    </div>
  );
}
```

#### `app/dashboard/page.tsx` (routing shell)

```tsx
import { DashboardPage } from "@/frontend/components/pages/DashboardPage";
export default function Page() { return <DashboardPage />; }
```

#### `Frontend/components/pages/DashboardPage.tsx`

```
Spec:
- "use client"
- Header: "Xin chào, [username]! 👋" (lấy từ useUser của Clerk)
- Summary cards (3 cards ngang):
  * Tổng số recipes (icon: BookOpen)
  * Tổng số cookbooks (icon: Library)
  * Lần scan gần nhất (icon: Camera, hiển thị relative time)
- Quick action buttons:
  * "📸 Scan nguyên liệu mới" → link /dashboard/scan
  * "📖 Xem Cookbook" → link /dashboard/cookbooks
  * "✏️ Tạo recipe thủ công" → link /dashboard/recipes/new
- Recent recipes section: hiển thị 3-4 recipe cards gần nhất
- Skeleton loading khi đang fetch data
- Data fetch: GET /api/recipes?limit=4 và GET /api/cookbooks (thông qua hooks)
```

---

### BƯỚC 1 — P1-1 AI Scan Flow UI

#### 1a. `Frontend/hooks/useScan.ts`

```
Spec:
- State: imageFile (File | null), previewUrl (string), status ('idle' | 'uploading' | 'scanning' | 'done' | 'error'), result (ScanResult | null), error (string | null)
- Function uploadAndScan(file: File):
  * Tạo FormData, append file
  * POST /api/scan (multipart/form-data)
  * Set status từng bước
  * Nhận response: { recipe: RecipeDTO, scanLogId: string }
  * Set result
- Function reset(): clear toàn bộ state
- Return: { imageFile, previewUrl, status, result, error, uploadAndScan, reset }

Types cần (inline hoặc import từ AI/types):
  interface ScanResult {
    recipe: {
      title: string;
      description: string;
      ingredients: string[];
      instructions: string[];
      cookTime: number;
      servings: number;
    };
    scanLogId: string;
  }
```

#### 1b. `Frontend/components/ai/ImageUploader.tsx`

```
Spec:
- "use client"
- Drag & drop zone: border-dashed, hover highlight, icon Camera/Upload từ lucide
- Click to select file (input[type=file] hidden, accept="image/*")
- Preview ảnh sau khi chọn (URL.createObjectURL)
- Progress bar khi uploading (animated)
- Status messages: "Đang nhận diện nguyên liệu...", "AI đang tạo công thức..."
- Nút "Chọn ảnh khác" để reset
- Props: onFileSelect?: (file: File) => void; status: string; previewUrl: string | null
```

#### 1c. `Frontend/components/ai/ScanResultCard.tsx`

```
Spec:
- "use client"
- Hiển thị kết quả scan:
  * Badge "✨ AI Generated"
  * Tên recipe (h2, bold)
  * Description
  * Ingredients detected: danh sách chip/tag
  * Thời gian nấu, số khẩu phần
  * Preview 3 bước đầu tiên của instructions
- Action buttons:
  * "💾 Lưu vào Cookbook" → gọi POST /api/recipes (sau đó redirect)
  * "🔄 Scan lại" → gọi reset()
- Props: result: ScanResult; onReset: () => void
- Skeleton: hiển thị khi status === 'scanning'
```

#### 1d. `Frontend/components/pages/ScanPage.tsx`

```
Spec:
- "use client"
- Sử dụng useScan() hook
- Layout: 2 cột trên desktop (uploader | result), 1 cột trên mobile
- Khi status === 'idle': chỉ hiện ImageUploader
- Khi status === 'uploading'/'scanning': hiện ImageUploader với progress + ScanResultCard skeleton
- Khi status === 'done': hiện ScanResultCard với kết quả
- Khi status === 'error': hiện error message + nút thử lại
- Page title: "🔍 Scan Nguyên Liệu"
```

#### 1e. `app/dashboard/scan/page.tsx` (routing shell)

```tsx
import { ScanPage } from "@/frontend/components/pages/ScanPage";
export default function Page() { return <ScanPage />; }
```

---

### BƯỚC 2 — P1-2 Recipe CRUD UI

#### 2a. `Frontend/hooks/useRecipes.ts`

```
Spec:
- State: recipes (Recipe[]), isLoading, error
- CRUD functions (với optimistic updates):
  * fetchRecipes(query?: string): GET /api/recipes?q=...
  * createRecipe(data: CreateRecipeInput): POST /api/recipes
  * updateRecipe(id: number, data: UpdateRecipeInput): PUT /api/recipes/:id
  * deleteRecipe(id: number): DELETE /api/recipes/:id
- Return: { recipes, isLoading, error, fetchRecipes, createRecipe, updateRecipe, deleteRecipe }

Type Recipe (dựa trên ERD):
  interface Recipe {
    recipe_id: number;
    recipes_name: string;
    description?: string;
    image_recipe?: string;
    total_time?: number;
    calories?: number;
    protein?: number;
    carbs?: number;
    fats?: number;
    source_type?: 'MANUAL' | 'IMPORTED' | 'AI_GENERATED';
    number_of_serves?: number;
    ingredients?: RecipeIngredient[];
    steps?: Step[];
    tags?: Tag[];
    created_at: string;
  }
```

#### 2b. `Frontend/components/recipe/RecipeCard.tsx`

```
Spec:
- Card layout: ảnh (aspect-video, fallback emoji 🍽️), tên, description truncated, badges
- Source badge: "🤖 AI" (green) | "✏️ Manual" (blue) | "📥 Imported" (gray)
- Info row: ⏱️ {total_time} phút | 👥 {number_of_serves} người
- Tags: chip list (tối đa 3, còn lại hiển thị "+N")
- Hover overlay: 3 icon buttons (Edit ✏️, Delete 🗑️, Add to Cookbook 📚)
- Click card → navigate /dashboard/recipes/:id
- Props: recipe: Recipe; onEdit?: () => void; onDelete?: () => void; onAddToCookbook?: () => void
```

#### 2c. `Frontend/components/recipe/RecipeList.tsx`

```
Spec:
- Grid responsive: 1 col mobile, 2 col tablet, 3 col desktop
- Toggle view: Grid | List (icon buttons)
- Empty state: icon 🍳, "Chưa có công thức nào", nút "Tạo công thức đầu tiên"
- Skeleton: 6 skeleton cards khi loading
- Props: recipes: Recipe[]; isLoading: boolean; onEdit, onDelete, onAddToCookbook callbacks
```

#### 2d. `Frontend/components/recipe/RecipeDetail.tsx`

```
Spec:
- Full-page recipe view:
  * Hero image (next/image nếu có, fallback gradient)
  * Source badge + created date
  * h1: recipe name
  * Description
  * Nutrition info grid: Calories | Protein | Carbs | Fats (4 cards ngang)
  * Ingredients section: list với quantity + unit + ingredient name
  * Steps section: numbered list, mỗi step có time estimate nếu có
  * Tags section: chip list
- Action buttons (chỉ hiện nếu là recipe của user): Edit | Delete | Add to Cookbook
- Back button → /dashboard/recipes
- Props: recipe: Recipe; onEdit, onDelete, onAddToCookbook
```

#### 2e. `Frontend/components/recipe/RecipeForm.tsx`

```
Spec:
- "use client"
- Form tạo/sửa recipe thủ công
- Fields:
  * recipes_name (required, text input)
  * description (textarea)
  * image_recipe (URL input, preview)
  * total_time (number, phút)
  * number_of_serves (number)
  * calories, protein, carbs, fats (number, optional)
- Dynamic fields:
  * Ingredients: [ ingredient_name | quantity | unit | note ] — thêm/xóa dòng
  * Steps: [ step_number | instruction | tip | time ] — reorderable, thêm/xóa
- Tags: multi-select từ danh sách có sẵn (fetch GET /api/tags)
- Submit: "Lưu công thức" | Cancel
- Props: initialData?: Partial<Recipe>; onSubmit: (data) => void; onCancel: () => void; isLoading: boolean
```

#### 2f. Pages & Routes cho Recipe

```
Files cần tạo:

Frontend/components/pages/RecipesPage.tsx:
  - "use client"
  - Dùng useRecipes() hook
  - Thanh search + filter (source_type, tag)
  - RecipeList component
  - Nút "➕ Tạo Recipe" → navigate /dashboard/recipes/new
  - Delete confirmation dialog (inline, không cần Modal riêng ở bước này)

Frontend/components/pages/RecipeDetailPage.tsx:
  - "use client"
  - Props: recipeId: number
  - Fetch recipe: GET /api/recipes/:id
  - RecipeDetail component
  - Loading skeleton + error state

app/dashboard/recipes/page.tsx:
  import { RecipesPage } from "@/frontend/components/pages/RecipesPage";
  export default function Page() { return <RecipesPage />; }

app/dashboard/recipes/[id]/page.tsx:
  import { RecipeDetailPage } from "@/frontend/components/pages/RecipeDetailPage";
  export default function Page({ params }: { params: { id: string } }) {
    return <RecipeDetailPage recipeId={Number(params.id)} />;
  }

app/dashboard/recipes/new/page.tsx:
  - Import RecipeForm
  - On submit: POST /api/recipes → redirect /dashboard/recipes/:newId
  - On cancel: navigate back
```

---

### BƯỚC 3 — P1-3 Cookbook Management UI

#### 3a. `Frontend/hooks/useCookbooks.ts`

```
Spec:
- State: cookbooks (Cookbook[]), isLoading, error
- Functions:
  * fetchCookbooks(): GET /api/cookbooks
  * createCookbook(name: string): POST /api/cookbooks
  * updateCookbook(id: number, name: string): PUT /api/cookbooks/:id
  * deleteCookbook(id: number): DELETE /api/cookbooks/:id
  * addRecipeToCookbook(cookbookId: number, recipeId: number): POST /api/cookbooks/:id/recipes
  * removeRecipeFromCookbook(cookbookId: number, recipeId: number): DELETE /api/cookbooks/:id/recipes

Type Cookbook:
  interface Cookbook {
    cookbook_id: number;
    name: string;
    recipes?: Recipe[];
    created_at: string;
    _count?: { recipes: number };
  }
```

#### 3b. `Frontend/components/cookbook/CookbookCard.tsx`

```
Spec:
- Card với:
  * Icon 📚 (lớn) hoặc grid preview ảnh từ recipes (tối đa 4 ảnh dạng mosaic)
  * Tên cookbook (h3)
  * Số lượng recipes: "X công thức"
  * Ngày tạo
- Hover: Edit name (✏️), Delete (🗑️) icon buttons
- Click → navigate /dashboard/cookbooks/:id
- Props: cookbook: Cookbook; onEdit, onDelete
```

#### 3c. `Frontend/components/cookbook/CookbookList.tsx`

```
Spec:
- Grid: 1 col mobile, 2 col tablet, 3 col desktop
- Empty state: "Chưa có cookbook nào", nút "Tạo cookbook đầu tiên"
- Skeleton: 3 skeleton cards khi loading
- Props: cookbooks: Cookbook[]; isLoading: boolean; onEdit, onDelete
```

#### 3d. `Frontend/components/cookbook/CookbookDetail.tsx`

```
Spec:
- Header: tên cookbook + edit name inline (click ✏️ → input), nút Delete cookbook
- Recipes in cookbook: dùng RecipeList component (read-only, kèm nút "Xóa khỏi cookbook" per card)
- Nút "➕ Thêm công thức" → mở modal search recipe để chọn
- Add recipe modal: search input + danh sách recipes của user → click để add
- Back button → /dashboard/cookbooks
- Props: cookbookId: number
```

#### 3e. Pages & Routes cho Cookbook

```
Files cần tạo:

Frontend/components/pages/CookbooksPage.tsx:
  - "use client"
  - Dùng useCookbooks() hook
  - Nút "➕ Tạo Cookbook" → inline form (input tên + submit)
  - CookbookList component
  - Edit cookbook name: inline modal với input

app/dashboard/cookbooks/page.tsx:
  import { CookbooksPage } from "@/frontend/components/pages/CookbooksPage";
  export default function Page() { return <CookbooksPage />; }

app/dashboard/cookbooks/[id]/page.tsx:
  import { CookbookDetail } from "@/frontend/components/cookbook/CookbookDetail";
  export default function Page({ params }: { params: { id: string } }) {
    return <CookbookDetail cookbookId={Number(params.id)} />;
  }
```

---

## RÀNG BUỘC (CONSTRAINTS BẮT BUỘC)

```
ARCHITECTURE:
✅ app/     → Chỉ chứa page.tsx / layout.tsx siêu mỏng (import từ Frontend/)
✅ Frontend/ → Toàn bộ UI components, hooks
❌ Không gọi Prisma từ Frontend
❌ Không gọi AI API từ Frontend
❌ Không đặt business logic trong app/api route handlers

PATH ALIASES (bắt buộc):
✅ @/frontend/components/...
✅ @/frontend/hooks/...
❌ Không dùng relative import ../../

TYPESCRIPT:
✅ Strict mode
✅ Mọi props phải có type/interface rõ ràng
✅ Không dùng 'any'

STYLING:
✅ Tailwind CSS
✅ Dark theme (zinc-950 background)
✅ Skeleton loading cho async states
✅ Responsive (mobile-first)
✅ Lucide Icons cho icons
```

---

## THỨ TỰ THỰC HIỆN ĐỀ XUẤT

```
Xong bước nào → test trên browser → mới làm bước tiếp:

1. BƯỚC 0: Tạo P0 nếu thiếu
   └── Navbar → DashboardLayout → DashboardPage

2. BƯỚC 1: AI Scan Flow
   ├── useScan.ts (hook)
   ├── ImageUploader.tsx
   ├── ScanResultCard.tsx
   └── ScanPage.tsx + route

3. BƯỚC 2: Recipe CRUD
   ├── useRecipes.ts (hook)
   ├── RecipeCard.tsx
   ├── RecipeList.tsx
   ├── RecipeDetail.tsx
   ├── RecipeForm.tsx
   └── RecipesPage, RecipeDetailPage + routes

4. BƯỚC 3: Cookbook
   ├── useCookbooks.ts (hook)
   ├── CookbookCard.tsx
   ├── CookbookList.tsx
   ├── CookbookDetail.tsx
   └── CookbooksPage + routes
```

---

## CÁCH TEST SAU MỖI BƯỚC

```bash
# Dev server
cd smart-kitchen-web && npm run dev

# Kiểm tra TypeScript
npx tsc --noEmit

# Các route cần test:
/ → HomePage (đã có)
/dashboard → DashboardPage
/dashboard/scan → ScanPage
/dashboard/recipes → RecipesPage
/dashboard/recipes/new → RecipeForm (tạo mới)
/dashboard/recipes/[id] → RecipeDetailPage
/dashboard/cookbooks → CookbooksPage
/dashboard/cookbooks/[id] → CookbookDetail
```

> **Lưu ý**: API routes (`/api/scan`, `/api/recipes`, `/api/cookbooks`) chưa được implement ở bước này. Hãy dùng **mock data** trong hooks (trả về data cứng) để có thể test UI ngay lập tức. Sau đó replace bằng real API calls khi backend sẵn sàng.

---

> **Cập nhật lần cuối**: 2026-05-18 | **Saved**: `Smart-kitchen-prompts/prompts/p1-ui-core-features.md`
