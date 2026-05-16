<!-- Mô tả nghiệp vụ (từ tài liệu PRD) -->

# FLOW — Luồng Nghiệp Vụ Smart Kitchen VN

> Tài liệu này mô tả các user journey chính và luồng dữ liệu tương ứng trong hệ thống.
> Đọc cùng với `ERD.md` (schema) và `AGENTS.md` (multi-agent architecture).

---

## Flow 1 — AI Recipe Generation (Core Feature)

> **Mô tả**: User chụp/upload ảnh nguyên liệu → AI nhận diện → generate recipe → lưu vào Cookbook.

```
[USER]
  │
  ▼ Upload ảnh nguyên liệu
[Frontend — IngredientUpload Component]
  │  POST /api/ai/analyze-image
  ▼
[app/api/ai/analyze-image] ← (Route handler mỏng)
  │  Gọi
  ▼
[AI — Orchestrator Agent]  (ai/agents/orchestrator.agent.ts)
  │
  ├──▶ [Vision Agent]  (ai/agents/vision.agent.ts)
  │       │  Gửi ảnh tới Google Cloud Vision API
  │       │  Trả về: string[] ingredients
  │       ▼
  │    ["cà chua", "trứng", "hành tây", ...]
  │
  ├──▶ [Recipe Agent]  (ai/agents/recipe.agent.ts)
  │       │  Gửi ingredients list tới Claude Sonnet 4.5
  │       │  Trả về: RecipeDTO (name, steps, nutrition, tags)
  │       ▼
  │    { recipeName, steps[], calories, protein, ... }
  │
  └──▶ [Storage Agent]  (ai/agents/storage.agent.ts)
          │  Gọi backend/services/recipe.service.ts
          │  Lưu Recipe + Steps + Ingredients + Tags vào PostgreSQL
          ▼
       Recipe saved ✅ → recipe_id trả về

  ▼
[Frontend] Nhận recipe_id → Redirect tới trang Recipe Detail
[USER] Xem công thức, chỉnh sửa, lưu vào Cookbook
```

### Data flow chi tiết

| Bước | Actor | Input | Output | DB Table |
|---|---|---|---|---|
| 1 | User | Upload ảnh | FormData | — |
| 2 | Vision Agent | Image binary | `string[]` ingredients | — |
| 3 | Recipe Agent | `string[]` ingredients | `RecipeDTO` | — |
| 4 | Storage Agent | `RecipeDTO` | `recipe_id` | `recipes`, `steps`, `recipe_ingredients`, `recipe_tags` |
| 5 | Frontend | `recipe_id` | Hiển thị Recipe Detail | — |

---

## Flow 2 — Manual Recipe Creation

> **Mô tả**: User tự nhập công thức (không dùng AI).

```
[USER] Điền form tạo công thức
  │
  ▼ POST /api/recipes
[app/api/recipes/route.ts]
  │  Validate với Zod schema (backend/schemas/recipe.schema.ts)
  ▼
[backend/services/recipe.service.ts]
  │  source_type = "MANUAL"
  │  Lưu Recipe + Steps + Ingredients
  ▼
[PostgreSQL]  ✅ Trả về recipe_id
  │
  ▼
[Frontend] Toast success → Redirect Recipe Detail
```

---

## Flow 3 — Authentication (Clerk)

> **Mô tả**: User đăng ký / đăng nhập qua Clerk. Sync user vào DB nội bộ.

```
[USER] Đăng ký / Đăng nhập
  │
  ▼ Clerk handles authentication
[Clerk Webhook] → POST /api/webhooks/clerk
  │  Event: user.created / user.updated
  ▼
[backend/services/user.service.ts]
  │  Upsert user vào bảng users (userId = Clerk userId)
  ▼
[PostgreSQL] ✅ User synced

[Subsequent requests]
  │  middleware.ts — Clerk middleware kiểm tra session
  ▼
[app/api/*] — Route handlers lấy userId từ Clerk session
```

---

## Flow 4 — Recipe Search & Filter

> **Mô tả**: User tìm kiếm công thức theo tên, nguyên liệu, hoặc tag.

```
[USER] Nhập từ khóa tìm kiếm
  │
  ▼ GET /api/recipes?q=cà+chua&tag=chay&userId=...
[app/api/recipes/route.ts]
  │
  ▼
[backend/services/recipe.service.ts]
  │  Query: WHERE recipes_name ILIKE '%cà chua%'
  │         AND recipe_tags.tag.name = 'chay'
  │         AND user_id = userId
  ▼
[PostgreSQL] Trả về danh sách recipes (với ingredients & tags)
  │
  ▼
[Frontend — RecipeList Component] Render danh sách
```

---

## Tóm Tắt API Endpoints

| Method | Path | Service | Mô tả |
|---|---|---|---|
| `POST` | `/api/ai/analyze-image` | Orchestrator Agent | Upload ảnh → generate recipe AI |
| `GET` | `/api/recipes` | recipe.service | Danh sách / tìm kiếm recipes |
| `POST` | `/api/recipes` | recipe.service | Tạo recipe thủ công |
| `GET` | `/api/recipes/:id` | recipe.service | Chi tiết recipe |
| `PUT` | `/api/recipes/:id` | recipe.service | Cập nhật recipe |
| `DELETE` | `/api/recipes/:id` | recipe.service | Xóa recipe |
| `POST` | `/api/webhooks/clerk` | user.service | Sync user từ Clerk |

---

## State Machine — Recipe Status

```
[Draft] ──── User nhập dữ liệu ────▶ [Saved]
                                        │
                         AI generate ───┘
                                        │
                              ┌─────────▼──────────┐
                              │  source_type:       │
                              │  MANUAL / IMPORTED  │
                              │  AI_GENERATED       │
                              └────────────────────┘
                                        │
                         Thêm vào Cookbook
                                        │
                                        ▼
                                  [In Cookbook]
```

---

## Ghi Chú Nghiệp Vụ

1. **AI recipes** tự động set `source_type = AI_GENERATED` và user có thể chỉnh sửa sau.
2. **Cascade delete**: Xóa user → xóa toàn bộ recipes và cookbooks của user đó.
3. **Cookbook**: Một recipe có thể thuộc nhiều cookbook (N:M).
4. **Nutrition data** (calories, protein, carbs, fats) do Recipe Agent tự động tính dựa trên ingredients.
5. **Tags** được tạo sẵn trong DB (seeded), không để user tự tạo tag mới để tránh dữ liệu rác.