# [AGENT-UI] — Actionable Prompt: P3 Chỉnh sửa công thức tự tạo

> **Sinh bởi**: [AGENT-PROMPT-ENGINEER]
> **Ngày**: 2026-05-20
> **Skill áp dụng**: `skills/02-Agent-ui.md`
> **Phụ thuộc**: `prompts/p1-ui-core-features.md` (P1 đã hoàn thành)

---

## MỤC TIÊU

Thêm chức năng **chỉnh sửa công thức do người dùng tự tạo** (`source_type === "MANUAL"`) vào ứng dụng Smart Kitchen VN. Công thức có nhãn `AI_GENERATED` **không được phép chỉnh sửa**.

**Flow hoạt động:**
1. User thấy nút ✏️ trên `RecipeCard` (trang danh sách) → chỉ hiện nút nếu `source_type !== "AI_GENERATED"`, nếu là AI thì nút bị mờ (disabled/opacity) và không click được.
2. User thấy nút "Chỉnh sửa" trong `RecipeDetail` (trang chi tiết) → đặt đối xứng với nút "Quay lại danh sách" ở cùng một hàng layout.
3. Khi bấm nút chỉnh sửa (từ cả hai nơi) → hiện dialog xác nhận **"Bạn có muốn chỉnh sửa công thức này?"**.
4. User xác nhận → điều hướng đến route `/dashboard/recipes/[id]/edit`.
5. Trang edit tái sử dụng `RecipeForm` với `initialData` được populate từ dữ liệu của `recipe_id`.
6. Sau khi submit thành công → `updateRecipe(id, data)` từ `useRecipes` → redirect về `/dashboard/recipes/[id]`.

---

## ROLE & CONSTRAINTS (BẮT BUỘC)

```
# ROLE: [AGENT-UI] - Chuyên gia Frontend (React/Next.js Client)

STRICT RULES:
1. CHỈ tạo/sửa file trong thư mục `Frontend/` (components/, hooks/)
   - app/ chỉ tạo page.tsx siêu mỏng (import component từ Frontend/)
2. Stack: Tailwind CSS + Lucide Icons + TypeScript
3. KHÔNG gọi Prisma hay AI API trực tiếp — mọi data qua custom hooks → fetch /api/*
4. Path alias: `@/frontend/*` (maps to ./Frontend/*)
5. Hook đã sẵn sàng: `updateRecipe(id, data)` có sẵn trong `Frontend/hooks/useRecipes.ts`
```

---

## PHÂN TÍCH KIẾN TRÚC & CÁC BƯỚC THỰC THI

> **Không cần đụng vào Backend/ hay app/api/**. Toàn bộ là UI-only vì `updateRecipe` hook đã gọi `PUT /api/recipes/:id`.

---

### BƯỚC 1 — Sửa `RecipeCard.tsx`: Nút chỉnh sửa ngoài danh sách

**File**: `Frontend/components/recipe/RecipeCard.tsx`

**Thay đổi cần làm:**

Trong phần `{/* Hover actions overlay */}` (hiện tại có nút `onEdit`), áp dụng logic:

```
- Nếu recipe.source_type === "AI_GENERATED":
    Nút ✏️ PHẢI HIỂN THỊ nhưng:
      * disabled={true}
      * opacity-40 cursor-not-allowed
      * title="Không thể chỉnh sửa công thức AI"
      * KHÔNG gọi onEdit khi click
- Nếu recipe.source_type === "MANUAL" (hoặc "IMPORTED"):
    Nút ✏️ bình thường, gọi onEdit() khi click
```

**Spec chi tiết nút disabled:**
```tsx
// Trong phần hover overlay, thay thế nút onEdit hiện tại:
{onEdit && (
  <button
    onClick={isAI ? undefined : onEdit}
    disabled={isAI}
    className={`w-8 h-8 rounded-lg bg-zinc-900/90 backdrop-blur border border-zinc-700
      flex items-center justify-center text-zinc-400 transition-colors
      ${isAI
        ? "opacity-40 cursor-not-allowed"
        : "hover:bg-zinc-800 hover:text-brand-300 cursor-pointer"
      }`}
    title={isAI ? "Không thể chỉnh sửa công thức AI" : "Chỉnh sửa công thức"}
  >
    <Edit2 className="w-3.5 h-3.5" />
  </button>
)}

// Khai báo biến isAI ở đầu component:
const isAI = recipe.source_type === "AI_GENERATED";
```

> **Quan trọng:** `onEdit` prop trong `RecipeCard` vẫn giữ nguyên, chỉ thêm logic điều kiện disabled.

---

### BƯỚC 2 — Sửa `RecipeDetail.tsx`: Nút chỉnh sửa trong trang chi tiết

**File**: `Frontend/components/recipe/RecipeDetail.tsx`

**Thay đổi cần làm:**

Hiện tại phần "Back button" ở đầu component (`RecipeDetail`) chỉ có nút "Quay lại danh sách" đơn lẻ:

```tsx
// Hiện tại (line ~60-66):
<Link href="/dashboard/recipes" className="inline-flex items-center gap-2 ...">
  <ArrowLeft className="w-4 h-4 ..." />
  Quay lại danh sách
</Link>
```

**Cần thay thành layout flex row có 2 phần tử đối xứng hai bên:**

```tsx
// Sau khi sửa:
<div className="flex items-center justify-between">
  {/* Trái: Quay lại danh sách */}
  <Link
    href="/dashboard/recipes"
    className="inline-flex items-center gap-2 text-sm text-brand-500 bg-brand-500/20
      hover:text-white hover:bg-white/10 rounded-xl px-3 py-1.5 transition-colors group"
  >
    <ArrowLeft className="w-4 h-4 group-hover:-translate-x-1 transition-transform" />
    Quay lại danh sách
  </Link>

  {/* Phải: Chỉnh sửa (chỉ render khi onEdit được truyền vào) */}
  {onEdit && (
    <button
      onClick={onEdit}
      className="inline-flex items-center gap-2 text-sm text-brand-400 bg-brand-500/10
        hover:text-white hover:bg-brand-500/20 border border-brand-500/30 rounded-xl
        px-3 py-1.5 transition-colors group"
    >
      <Edit2 className="w-4 h-4" />
      Chỉnh sửa công thức
    </button>
  )}
</div>
```

**Lưu ý:**
- Nút "Sửa" trong action overlay ảnh hero (top-right) vẫn giữ nguyên — đây là nút chỉnh sửa bổ sung.
- Nút chỉnh sửa mới này là CTA riêng ở đầu trang, đối xứng với nút "Quay lại".
- Không hiển thị nút nếu `recipe.source_type === "AI_GENERATED"` — logic này xử lý tại `RecipeDetailPage` (xem Bước 3).

---

### BƯỚC 3 — Sửa `RecipeDetailPage.tsx`: Thêm confirm dialog + routing edit

**File**: `Frontend/components/pages/RecipeDetailPage.tsx`

**Thay đổi cần làm:**

Thêm state `showEditConfirm` và logic điều hướng:

```tsx
"use client";

import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import { RecipeDetail, RecipeDetailSkeleton } from "@/frontend/components/recipe/RecipeDetail";
import type { Recipe } from "@/frontend/hooks/useRecipes";

// ... (mock data giữ nguyên)

export function RecipeDetailPage({ recipeId }: RecipeDetailPageProps) {
  const router = useRouter();
  const [recipe, setRecipe] = useState<Recipe | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [showEditConfirm, setShowEditConfirm] = useState(false); // ← THÊM MỚI

  // ... (useEffect fetch recipe giữ nguyên)

  // Chỉ cho phép sửa công thức MANUAL
  const isEditable = recipe?.source_type === "MANUAL";

  const handleEditClick = () => {
    setShowEditConfirm(true);
  };

  const confirmEdit = () => {
    setShowEditConfirm(false);
    router.push(`/dashboard/recipes/${recipeId}/edit`);
  };

  if (isLoading) return <RecipeDetailSkeleton />;
  if (error || !recipe) { /* ... error state giữ nguyên ... */ }

  return (
    <>
      <RecipeDetail
        recipe={recipe}
        onEdit={isEditable ? handleEditClick : undefined}  // ← chỉ truyền onEdit nếu isEditable
        // onDelete, onAddToCookbook giữ nguyên nếu có
      />

      {/* Confirm Edit Dialog */}
      {showEditConfirm && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
          <div
            className="absolute inset-0 bg-zinc-950/80 backdrop-blur-sm"
            onClick={() => setShowEditConfirm(false)}
          />
          <div className="relative glass-card p-6 max-w-sm w-full space-y-4">
            <h3 className="font-semibold text-white text-lg">Chỉnh sửa công thức?</h3>
            <p className="text-zinc-400 text-sm">
              Bạn có muốn chỉnh sửa công thức{" "}
              <span className="text-white font-medium">"{recipe.recipes_name}"</span>?
            </p>
            <div className="flex gap-3">
              <button
                onClick={confirmEdit}
                className="flex-1 px-4 py-2 bg-brand-600 hover:bg-brand-500 text-white
                  rounded-xl text-sm font-medium transition-colors"
              >
                ✏️ Chỉnh sửa
              </button>
              <button
                onClick={() => setShowEditConfirm(false)}
                className="btn-ghost text-sm"
              >
                Hủy
              </button>
            </div>
          </div>
        </div>
      )}
    </>
  );
}
```

---

### BƯỚC 4 — Sửa `RecipesPage.tsx`: Thêm confirm dialog cho nút edit từ danh sách

**File**: `Frontend/components/pages/RecipesPage.tsx`

**Thay đổi cần làm:**

Hiện tại `RecipesPage` đã có `onDelete` nhưng chưa có `onEdit`. Cần thêm:

```tsx
// Thêm state:
const [editTarget, setEditTarget] = useState<Recipe | null>(null);

// Handler:
const handleEdit = (recipe: Recipe) => {
  if (recipe.source_type === "AI_GENERATED") return; // guard thêm
  setEditTarget(recipe);
};

const confirmEdit = () => {
  if (!editTarget) return;
  setEditTarget(null);
  router.push(`/dashboard/recipes/${editTarget.recipe_id}/edit`);
};

// Import useRouter:
import { useRouter } from "next/navigation";
const router = useRouter();

// Truyền onEdit vào RecipeList:
<RecipeList
  recipes={filtered}
  isLoading={isLoading}
  onEdit={handleEdit}       // ← THÊM
  onDelete={handleDelete}
/>

// Thêm Confirm Edit Dialog (sau Delete Dialog, tương tự cấu trúc):
{editTarget && (
  <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
    <div
      className="absolute inset-0 bg-zinc-950/80 backdrop-blur-sm"
      onClick={() => setEditTarget(null)}
    />
    <div className="relative glass-card p-6 max-w-sm w-full space-y-4">
      <h3 className="font-semibold text-white text-lg">Chỉnh sửa công thức?</h3>
      <p className="text-zinc-400 text-sm">
        Bạn có muốn chỉnh sửa công thức{" "}
        <span className="text-white font-medium">"{editTarget.recipes_name}"</span>?
      </p>
      <div className="flex gap-3">
        <button
          onClick={confirmEdit}
          className="flex-1 px-4 py-2 bg-brand-600 hover:bg-brand-500 text-white
            rounded-xl text-sm font-medium transition-colors"
        >
          ✏️ Chỉnh sửa
        </button>
        <button onClick={() => setEditTarget(null)} className="btn-ghost text-sm">
          Hủy
        </button>
      </div>
    </div>
  </div>
)}
```

---

### BƯỚC 5 — Tạo trang Edit Route mới

#### 5a. [NEW] `Frontend/components/pages/RecipeEditPage.tsx`

Đây là component page chính cho chức năng edit, tái sử dụng `RecipeForm` với `initialData`:

```
Spec:
- "use client"
- Props: recipeId: number
- useEffect: fetch GET /api/recipes/:id → populate initialData
- Hiển thị skeleton khi loading (dùng lại RecipeDetailSkeleton hoặc skeleton form tương tự)
- Render RecipeForm với:
    * initialData={recipe}   ← truyền toàn bộ recipe object (RecipeForm đã nhận Partial<Recipe>)
    * onSubmit={handleSubmit}
    * onCancel={() => router.push(`/dashboard/recipes/${recipeId}`)}
    * isLoading={isSaving}
- handleSubmit:
    * Gọi PUT /api/recipes/:id thông qua updateRecipe(recipeId, data)
    * Sau khi thành công → router.push(`/dashboard/recipes/${recipeId}`)
    * Nếu lỗi → hiển thị toast/error message
- Tiêu đề trang: "✏️ Chỉnh sửa công thức" (thay vì "Tạo công thức mới")
- Nút "Quay lại" → router.push(`/dashboard/recipes/${recipeId}`)

Lưu ý về RecipeForm & initialData:
- RecipeForm hiện đã nhận initialData?: Partial<Recipe>
- Các trường cơ bản (name, description, image_recipe, total_time...) đã được populate
- Với ingredients/steps: RecipeForm hiện khởi tạo state rỗng khi không có dữ liệu.
  Cần cải thiện: nếu recipe.ingredients có dữ liệu, map sang FormIngredient[] để populate;
  tương tự với recipe.steps → map sang FormStep[].
```

**Spec cụ thể cho việc populate ingredients/steps trong `RecipeForm.tsx`:**

Sửa `useState` khởi tạo trong `RecipeForm.tsx`:

```tsx
// Thay thế:
const [ingredients, setIngredients] = useState<FormIngredient[]>([
  { name: "", quantity: "", unit: "", note: "" },
]);
const [steps, setSteps] = useState<FormStep[]>([{ instruction: "", tip: "", time: "" }]);

// Thành:
const [ingredients, setIngredients] = useState<FormIngredient[]>(() => {
  if (initialData?.ingredients && initialData.ingredients.length > 0) {
    return initialData.ingredients.map((ing) => ({
      name: ing.name ?? "",
      quantity: String(ing.quantity ?? ""),
      unit: ing.unit ?? "",
      note: ing.note ?? "",
    }));
  }
  return [{ name: "", quantity: "", unit: "", note: "" }];
});

const [steps, setSteps] = useState<FormStep[]>(() => {
  if (initialData?.steps && initialData.steps.length > 0) {
    return initialData.steps.map((s) => ({
      instruction: s.instruction ?? "",
      tip: s.tip ?? "",
      time: String(s.time ?? ""),
    }));
  }
  return [{ instruction: "", tip: "", time: "" }];
});
```

#### 5b. [NEW] `app/dashboard/recipes/[id]/edit/page.tsx`

Route shell siêu mỏng:

```tsx
import { RecipeEditPage } from "@/frontend/components/pages/RecipeEditPage";

export default function Page({ params }: { params: { id: string } }) {
  return <RecipeEditPage recipeId={Number(params.id)} />;
}
```

---

## RÀNG BUỘC (CONSTRAINTS BẮT BUỘC)

```
ARCHITECTURE:
✅ app/dashboard/recipes/[id]/edit/page.tsx  → Route shell siêu mỏng
✅ Frontend/components/pages/RecipeEditPage.tsx → Logic + UI
✅ Tái sử dụng RecipeForm với initialData (không duplicate component)
✅ Tái sử dụng updateRecipe từ useRecipes hook (đã có sẵn)
❌ KHÔNG tạo API route mới — PUT /api/recipes/:id đã tồn tại
❌ KHÔNG chỉnh sửa useRecipes.ts — hook updateRecipe đã đúng

LOGIC PHÂN QUYỀN:
✅ source_type === "MANUAL" → cho phép edit
✅ source_type === "AI_GENERATED" → disabled nút, không navigate đến edit route
✅ source_type === "IMPORTED" → tùy thiết kế (đề xuất: cho phép edit, vì user đã import về)

CONFIRM DIALOG:
✅ Cả 2 entry point (danh sách & chi tiết) đều hiển thị dialog "Bạn có muốn chỉnh sửa?"
✅ Dialog dùng inline modal (fixed overlay), cùng style với dialog xóa đang có
✅ Nút confirm: brand-600 color | Nút hủy: btn-ghost

PATH ALIASES:
✅ @/frontend/components/...
✅ @/frontend/hooks/...
❌ Không dùng relative import ../../
```

---

## THỨ TỰ THỰC HIỆN ĐỀ XUẤT

```
Làm tuần tự, xong bước nào test ngay trên browser:

1. BƯỚC 5b: Tạo route app/dashboard/recipes/[id]/edit/page.tsx (chỉ là shell)
   └── Test: navigate thủ công đến /dashboard/recipes/1/edit → thấy import error → OK

2. BƯỚC 5a: Tạo Frontend/components/pages/RecipeEditPage.tsx
   └── Test: /dashboard/recipes/1/edit → thấy form với tiêu đề "Chỉnh sửa công thức"
   └── Test: form populate dữ liệu recipe (nếu fetch API thành công)

3. BƯỚC 1a (trong RecipeForm.tsx): Sửa useState ingredients/steps để populate từ initialData
   └── Test: form edit hiển thị đúng nguyên liệu và bước của recipe gốc

4. BƯỚC 2: Sửa RecipeDetail.tsx — thêm layout 2 nút đối xứng
   └── Test: /dashboard/recipes/1 → thấy nút "Chỉnh sửa" bên phải, "Quay lại" bên trái

5. BƯỚC 3: Sửa RecipeDetailPage.tsx — thêm confirm dialog + isEditable logic
   └── Test: click "Chỉnh sửa" → thấy dialog confirm → click "Chỉnh sửa" → vào edit page
   └── Test: recipe AI_GENERATED → không truyền onEdit → không thấy nút chỉnh sửa

6. BƯỚC 4: Sửa RecipesPage.tsx — thêm handleEdit + confirm dialog
   └── Test: hover recipe card MANUAL → nút ✏️ active → click → thấy dialog confirm
   └── Test: hover recipe card AI_GENERATED → nút ✏️ mờ → không click được

7. KIỂM TRA E2E:
   a. Tạo 1 công thức MANUAL → edit → lưu → xem lại chi tiết → data đã cập nhật
   b. Hover recipe AI → nút edit bị disabled
   c. Vào trang chi tiết recipe AI → không có nút chỉnh sửa ở hàng đầu
```

---

## CÁCH TEST SAU KHI HOÀN THÀNH

```bash
# Dev server (đã đang chạy)
# npm run dev trong d:/Smart_Kitchen_project/smart-kitchen-web

# TypeScript check
npx tsc --noEmit

# Các scenario cần test thủ công trên browser:

SCENARIO 1: Edit từ danh sách (RecipesPage)
  → /dashboard/recipes
  → Hover recipe có source_type = "MANUAL"
  → Thấy nút ✏️ sáng bình thường
  → Click ✏️ → Dialog "Bạn có muốn chỉnh sửa?"
  → Click "Chỉnh sửa" → vào /dashboard/recipes/:id/edit
  → Form hiển thị đúng data của recipe
  → Sửa tên → bấm "Lưu công thức" → redirect về /dashboard/recipes/:id
  → Tên đã được cập nhật

SCENARIO 2: Edit từ trang chi tiết (RecipeDetailPage)
  → /dashboard/recipes/:id (recipe MANUAL)
  → Thấy hàng đầu: [← Quay lại danh sách] ... [✏️ Chỉnh sửa công thức →]
  → Click "Chỉnh sửa công thức" → Dialog confirm
  → Xác nhận → vào edit page

SCENARIO 3: AI recipe — không cho edit
  → /dashboard/recipes (hover recipe AI_GENERATED)
  → Nút ✏️ mờ (opacity-40), cursor not-allowed, không click được
  → /dashboard/recipes/:id (recipe AI_GENERATED)
  → Không có nút "Chỉnh sửa" ở hàng navigation đầu trang
```

---

## TÓM TẮT CÁC FILE CẦN THAY ĐỔI

| Hành động | File | Mô tả |
|-----------|------|-------|
| **MODIFY** | `Frontend/components/recipe/RecipeCard.tsx` | Nút edit: disabled + mờ nếu AI_GENERATED |
| **MODIFY** | `Frontend/components/recipe/RecipeForm.tsx` | Populate ingredients/steps từ initialData |
| **MODIFY** | `Frontend/components/recipe/RecipeDetail.tsx` | Layout 2 nút đối xứng ở đầu trang |
| **MODIFY** | `Frontend/components/pages/RecipeDetailPage.tsx` | isEditable logic + confirm dialog |
| **MODIFY** | `Frontend/components/pages/RecipesPage.tsx` | Thêm handleEdit + confirm dialog |
| **NEW** | `Frontend/components/pages/RecipeEditPage.tsx` | Page component cho edit |
| **NEW** | `app/dashboard/recipes/[id]/edit/page.tsx` | Route shell siêu mỏng |

---

> **Cập nhật lần cuối**: 2026-05-20 | **Saved**: `Smart-kitchen-prompts/prompts/p3-edit-user-recipe.md`
