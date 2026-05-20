# Experience: Trang "Chỉnh sửa công thức" không cập nhật DB

> **Ngày**: 2026-05-20
> **Feature**: P3 — Edit User Recipe
> **Severity**: High (chức năng core bị broken)

---

## Mô tả lỗi

Sau khi hoàn thành trang edit recipe theo `prompts/p3-edit-user-recipe.md`, khi user submit form chỉnh sửa → trang redirect về chi tiết thành công nhưng **dữ liệu ingredients và steps không thay đổi trong DB**.

Metadata cơ bản (tên, mô tả, thời gian nấu...) cũng có thể không được lưu do mapping sai key (snake_case → camelCase).

---

## Nguyên nhân gốc rễ (3 điểm)

### 1. [CHÍNH] `RecipeEditPage.tsx` — bỏ qua `ingredients` và `steps`

`handleSubmit` nhận đầy đủ data từ `RecipeForm` (bao gồm `ingredients[]` và `steps[]`) nhưng **chỉ forward metadata** vào `updateRecipe`, không truyền 2 mảng quan trọng:

```tsx
// ❌ BUG:
await updateRecipe(recipeId, {
  recipes_name: data.recipes_name,
  // ... metadata only
  // ingredients và steps bị bỏ qua!
});
```

### 2. `useRecipes.ts` — `UpdateRecipeInput` thiếu slot cho `ingredients`/`steps`

```ts
// ❌ BUG:
export type UpdateRecipeInput = Partial<CreateRecipeInput>;
// CreateRecipeInput chỉ có metadata → TypeScript không báo lỗi khi bỏ sót
```

### 3. `useRecipes.ts` — `updateRecipe` gửi payload sai format

Hook gửi thẳng `data` (snake_case) nhưng Backend schema kỳ vọng camelCase (`recipesName`, `totalTime`, `ingredientId`, `stepNumber`...). Ngoài ra `time` trong step phải là `number` không phải `string`.

---

## Cách fix

### File 1: `Frontend/components/recipe/RecipeForm.tsx`
- Thêm `ingredient_id?: number` vào `FormIngredient` type
- Preserve `ingredient_id` khi map từ `initialData.ingredients`

### File 2: `Frontend/hooks/useRecipes.ts`
- Extend `UpdateRecipeInput` để nhận `ingredients[]` và `steps[]`
- Sửa optimistic update: destructure ra `_ing`, `_steps` trước khi spread (tránh type conflict)
- Sửa `updateRecipe` để build `apiPayload` với camelCase keys và map đúng kiểu:
  - `ingredients` → `{ ingredientId, quantity: Number, unit, note }`
  - `steps` → `{ stepNumber: idx+1, instruction, tip, time: Number }`

### File 3: `Frontend/components/pages/RecipeEditPage.tsx`
- Thêm `ingredients: data.ingredients` và `steps: data.steps` vào lời gọi `updateRecipe`

---

## Kết quả test

| Test | Kết quả |
|------|---------|
| `npx tsc --noEmit` — lỗi liên quan edit recipe | ✅ Không còn lỗi mới |
| Lỗi pre-existing (ProfilePage, test files) | ⚠️ Vẫn còn (không liên quan) |

**Browser test (cần thực hiện thủ công):**
- Verify Network tab: PUT request body phải có `steps[]` và `ingredients[]`
- Submit edit form → redirect → kiểm tra dữ liệu mới

---

## Lưu ý quan trọng

1. **Ingredient mới (không có `ingredient_id`)** bị bỏ qua khi update — filter `i.ingredient_id` trước khi map. Nếu user nhập tên nguyên liệu mới, nó sẽ không được lưu. Cần API riêng để tạo ingredient trước khi upsert.

2. **Backend đã đúng** — `recipe.service.ts` dùng `deleteMany` + `create` pattern cho steps/ingredients. Frontend là nguồn gốc 100% của bug.

3. **Optimistic update** chỉ cập nhật metadata (không cập nhật ingredients/steps trong state local) vì shape mismatch. Sau khi redirect về trang chi tiết, data sẽ được fetch lại từ API nên hiển thị đúng.

---

## Prompt mới để tạo

> Gửi [AGENT-PROMPT-ENGINEER]: Cập nhật `prompts/p3-edit-user-recipe.md` để bổ sung warning về 3 lỗi trên vào phần "RÀNG BUỘC" — đặc biệt nhấn mạnh:
> - `handleSubmit` PHẢI truyền `ingredients` và `steps` vào `updateRecipe`
> - `UpdateRecipeInput` cần được extend để nhận ingredients/steps từ form
> - Optimistic update không nên spread ingredients/steps (type conflict)
