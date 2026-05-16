# CHƯƠNG 3: PHƯƠNG PHÁP MSA (MULTI-SYSTEM AGENT)

Chương này là trọng tâm kỹ thuật của báo cáo, trình bày chi tiết kiến trúc hệ thống Multi-System Agent được thiết kế cho Smart Kitchen VN, bao gồm vai trò từng agent, nguyên tắc phân tách concerns, luồng xử lý AI Scan, thiết kế database và API.

---

## 3.1 Thiết Kế Tổng Thể Hệ Thống

Smart Kitchen VN được tổ chức theo mô hình phân tầng rõ ràng với 4 layer chính:

```
┌─────────────────────────────────────────────┐
│              CLIENT (Browser)               │
│         Frontend/ — React Components        │
└──────────────────┬──────────────────────────┘
                   │ HTTP Request
┌──────────────────▼──────────────────────────┐
│           ROUTING LAYER (app/)              │
│    Next.js App Router — Thin Shell Only     │
└───────┬─────────────────┬───────────────────┘
        │                 │
┌───────▼───────┐  ┌──────▼────────────────────┐
│  Backend/     │  │        AI/                │
│  Services     │  │  Multi-Agent System       │
│  Schemas      │◄─┤  Orchestrator             │
│  DB Client    │  │  → Vision Agent           │
└───────┬───────┘  │  → Recipe Agent           │
        │          │  → Storage Agent          │
┌───────▼───────┐  └───────────────────────────┘
│  PostgreSQL   │
│  (Prisma ORM) │
└───────────────┘
```

Nguyên tắc thiết kế cốt lõi: **Không một layer nào được "nhảy cấp"**. Frontend không được gọi Prisma trực tiếp. AI agents không được gọi database trực tiếp — phải đi qua Backend services. Route handlers trong `app/api/` chỉ được gọi service, không chứa business logic.

---

## 3.2 Kiến Trúc Multi-System Agent

### 3.2.1 Tổng quan

Hệ thống MSA của Smart Kitchen gồm 4 agent, phối hợp theo pattern **Orchestrator-Worker**:

```
[Orchestrator Agent]
    ├──► [Vision Agent]   ─── Google Cloud Vision API
    ├──► [Recipe Agent]   ─── Claude Sonnet 4.5 (Anthropic)
    └──► [Storage Agent]  ─── Backend/services/recipe.service.ts
```

Các agent giao tiếp **qua message (function call)**, không chia sẻ state trực tiếp. Orchestrator là entry point duy nhất — ngoài hệ thống không được gọi Vision/Recipe/Storage agent trực tiếp.

### 3.2.2 Orchestrator Agent (`AI/agents/orchestrations.agent.ts`)

Orchestrator là "não bộ" điều phối toàn bộ pipeline AI. Nhiệm vụ:
- Nhận input từ API route (`imageBuffer`, `userId`)
- Gọi Vision Agent → nhận `string[]` ingredients
- Gọi Recipe Agent với ingredients list → nhận `RecipeDTO`
- Gọi Storage Agent với `RecipeDTO` + `userId` → nhận `recipeId`
- Xử lý lỗi từng bước: nếu Vision fail → trả về error sớm, không chạy tiếp
- Trả về kết quả tổng hợp cho API route

**Design decision**: Orchestrator xử lý lỗi theo mô hình **fail-fast** — dừng pipeline ngay khi một bước thất bại, không tiếp tục xử lý tốn kém API call.

### 3.2.3 Vision Agent (`AI/agents/vision.agent.ts`)

Vision Agent chịu trách nhiệm duy nhất: chuyển đổi ảnh thành danh sách nguyên liệu.

**Input**: `imageBuffer: Buffer` hoặc `imageUrl: string`  
**Output**: `string[]` — danh sách tên nguyên liệu tiếng Việt  
**Process**:
1. Gửi ảnh đến Google Cloud Vision API (Label Detection feature)
2. Nhận về danh sách labels với confidence score
3. Lọc labels có score > ngưỡng (ví dụ: 0.7)
4. Map label tiếng Anh sang tên nguyên liệu tiếng Việt
5. Trả về `string[]` cho Orchestrator

**Lý do chọn Label Detection**: So với Object Detection, Label Detection cho kết quả mô tả chung hơn nhưng đủ để Recipe Agent hiểu và sinh công thức. Object Detection cần training data thực phẩm tiếng Việt — phức tạp hơn nhiều.

### 3.2.4 Recipe Agent (`AI/agents/recipe.agent.ts`)

Recipe Agent là agent quan trọng nhất về chất lượng output. Nó đóng vai trò **Prompt Engineer tự động** — biến danh sách nguyên liệu thô thành công thức hoàn chỉnh.

**Input**: `ingredients: string[]`  
**Output**: `RecipeDTO { title, description, ingredients, instructions, cookTime, servings, calories, protein, carbs, fats }`  

**Prompt Engineering Strategy**:
- System prompt định nghĩa output schema JSON cố định, buộc Claude trả về đúng cấu trúc
- Few-shot examples trong prompt giúp Claude hiểu format mong muốn
- Constraint rõ ràng: "Chỉ sử dụng nguyên liệu trong danh sách, không thêm nguyên liệu không có"
- Yêu cầu ước tính dinh dưỡng dựa trên khẩu phần tiêu chuẩn

### 3.2.5 Storage Agent (`AI/agents/storage.agent.ts`)

Storage Agent đóng vai trò **cầu nối** giữa lớp AI và lớp Backend. Đây là agent duy nhất được phép tương tác với database thông qua service.

**Input**: `RecipeDTO` + `userId: string`  
**Output**: `recipeId: string`  
**Process**:
1. Validate `RecipeDTO` trước khi lưu (sử dụng Zod schema từ `Backend/schemas/`)
2. Gọi `createRecipe(userId, data)` từ `Backend/services/recipe.service.ts`
3. Set `source_type = "AI_GENERATED"` tự động
4. Trả về `recipeId` cho Orchestrator

**Design decision**: Storage Agent không được import Prisma Client trực tiếp — phải gọi qua Backend service. Điều này đảm bảo business logic (ownership check, cascade) luôn được áp dụng nhất quán.

---

## 3.3 Nguyên Tắc STRICT 3-FOLDER RULE

Đây là nguyên tắc kiến trúc quan trọng nhất của dự án, đảm bảo Separation of Concerns nghiêm ngặt:

| Thư mục | Vai trò | Được gọi bởi | Không được gọi |
|---|---|---|---|
| `app/` | Routing shell | Browser, Next.js | Prisma, AI API trực tiếp |
| `Frontend/` | UI (browser-side) | app/, user | Prisma, AI API |
| `Backend/` | Business logic (server) | app/api/, AI/ | Frontend/ |
| `AI/` | Agent system | app/api/ | Prisma trực tiếp |

**Lý do quan trọng**: Vi phạm nguyên tắc này gây ra:
- **Circular dependency**: A import B, B import A → runtime error
- **Security leak**: Business logic lộ ra client bundle
- **Khó test**: Logic rải rác khắp nơi, không isolate được

---

## 3.4 Luồng Xử Lý AI Scan (Core Workflow)

Đây là workflow trung tâm của Smart Kitchen, thực hiện qua 5 bước:

**Bước 1 — User Upload**: Người dùng chọn ảnh từ device, frontend gửi `multipart/form-data` đến `POST /api/scan`.

**Bước 2 — Vision Processing**: Route handler gọi Orchestrator → Orchestrator gọi Vision Agent → Vision Agent gửi ảnh đến Google Cloud Vision API → nhận về danh sách labels như `["tomato", "egg", "onion", "garlic"]` → map sang tiếng Việt `["cà chua", "trứng", "hành tây", "tỏi"]`.

**Bước 3 — Recipe Generation**: Orchestrator gọi Recipe Agent với danh sách nguyên liệu → Recipe Agent gọi Claude Sonnet 4.5 với prompt được thiết kế sẵn → Claude trả về JSON có cấu trúc với đầy đủ công thức, các bước và thông tin dinh dưỡng.

**Bước 4 — Storage**: Orchestrator gọi Storage Agent → Storage Agent validate data → gọi `recipe.service.ts` → Prisma lưu vào PostgreSQL với `source_type = AI_GENERATED`.

**Bước 5 — Response**: API trả về `{ recipeId, recipe }` → Frontend redirect người dùng đến trang Recipe Detail để xem và chỉnh sửa.

---

## 3.5 Thiết Kế Database

### 3.5.1 Lựa chọn PostgreSQL

PostgreSQL được chọn thay vì NoSQL vì: công thức nấu ăn có quan hệ phức tạp giữa các entity (Recipe ↔ Ingredient ↔ Tag ↔ Cookbook), cần JOIN query hiệu quả; hỗ trợ `ENUM` type cho `source_type`; `Json` column cho ingredients/instructions linh hoạt; và full-text search tích hợp sẵn cho tính năng tìm kiếm về sau.

### 3.5.2 Schema 8 Bảng

Hệ thống gồm 8 bảng với quan hệ rõ ràng:

- **users**: Lưu profile người dùng, liên kết với Clerk user ID
- **recipes**: Bảng trung tâm — lưu công thức, thông tin dinh dưỡng, `source_type` enum
- **cookbooks**: Bộ sưu tập công thức, thuộc về một user
- **cookbook_recipes**: Bảng junction N:M giữa Cookbook và Recipe
- **steps**: Các bước thực hiện công thức, có `step_number` để sắp xếp
- **ingredients**: Catalog nguyên liệu dùng chung toàn hệ thống
- **recipe_ingredients**: Junction N:M giữa Recipe và Ingredient, lưu `quantity` và `unit`
- **tags**: Nhãn phân loại (Chay, Keto, Dễ làm...), được seed sẵn
- **recipe_tags**: Junction N:M giữa Recipe và Tag

### 3.5.3 Quyết Kế Cascade Delete

Toàn bộ foreign keys sử dụng `onDelete: Cascade` — xóa User kéo theo xóa toàn bộ Recipes và Cookbooks của user đó. Điều này đảm bảo không có orphan records trong database, tuân thủ quy tắc integrity.

---

## 3.6 Thiết Kế API — Thin Router Pattern

Mỗi route handler trong `app/api/` tuân thủ quy tắc **không quá 20 dòng code** và chỉ làm 4 việc:

1. **Lấy `userId`** từ Clerk session (`auth()`)
2. **Validate input** bằng Zod schema (import từ `Backend/schemas/`)
3. **Gọi service function** (import từ `Backend/services/`)
4. **Trả về JSON** chuẩn (`NextResponse.json()`)

Pattern này đảm bảo route handler không bao giờ chứa business logic — dễ test, dễ thay thế framework nếu cần, và enforce separation rõ ràng.

**Ví dụ cấu trúc route handler:**
```
POST /api/recipes
  → auth().userId
  → CreateRecipeSchema.parse(body)
  → createRecipe(userId, data)
  → NextResponse.json({ recipe }, 201)
```
