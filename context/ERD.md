<!-- Mô tả database schema bằng text -->
# ERD — Database Schema (Smart Kitchen)

> Mô tả toàn bộ database schema. Source of truth: `smart-kitchen-api/prisma/schema.prisma`.
> Database: **PostgreSQL** | ORM: **Prisma**

---

## Entity Relationship Diagram

```
┌─────────────┐       ┌──────────────────┐       ┌──────────────┐
│    users    │       │     recipes      │       │  cookbooks   │
├─────────────┤       ├──────────────────┤       ├──────────────┤
│ user_id (PK)│──1:N──│ recipe_id (PK)   │       │cookbook_id(PK│
│ email       │       │ user_id (FK)     │──N:M──│ user_id (FK) │
│ username    │──1:N──│ recipes_name     │       │ name         │
│ avatar_url  │       │ description      │       │ created_at   │
│ created_at  │       │ image_recipe     │       │ updated_at   │
│ updated_at  │       │ total_time       │       └──────────────┘
└─────────────┘       │ calories         │              │
                      │ protein          │              │ (via)
                      │ carbs            │   ┌──────────────────────┐
                      │ fats             │   │   cookbook_recipes   │
                      │ source_type      │   ├──────────────────────┤
                      │ number_of_serves │   │ recipe_id (FK, PK)   │
                      │ created_at       │   │ cookbook_id (FK, PK) │
                      │ updated_at       │   │ created_at           │
                      └──────────────────┘   │ updated_at           │
                             │               └──────────────────────┘
              ┌──────────────┼──────────────┐
              │              │              │
        ┌───────────┐  ┌──────────┐  ┌─────────────────────┐
        │   steps   │  │recipe_tag│  │ recipe_ingredients  │
        ├───────────┤  ├──────────┤  ├─────────────────────┤
        │step_id(PK)│  │recipe_id │  │ recipe_id (FK, PK)  │
        │recipe_id  │  │  (FK,PK) │  │ingredient_id(FK, PK)│
        │step_number│  │tag_id    │  │ quantity            │
        │instruction│  │  (FK,PK) │  │ unit                │
        │ tip       │  └──────────┘  │ note                │
        │ time      │       │        └─────────────────────┘
        │created_at │  ┌──────────┐           │
        │updated_at │  │   tags   │  ┌─────────────────┐
        └───────────┘  ├──────────┤  │  ingredients    │
                       │tag_id(PK)│  ├─────────────────┤
                       │ name     │  │ingredient_id(PK)│
                       │ category │  │ name            │
                       │ emoji    │  │ icon            │
                       │created_at│  │ bg_color        │
                       │updated_at│  │ created_at      │
                       └──────────┘  │ updated_at      │
                                     └─────────────────┘
```

---

## Mô tả Chi Tiết Từng Bảng

### `users` — Người dùng

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `user_id` | `String` (UUID) | PK | ID người dùng (tích hợp Clerk) |
| `email` | `String` | UNIQUE, NOT NULL | Email đăng nhập |
| `username` | `String` | NOT NULL | Tên hiển thị |
| `avatar_url` | `String?` | NULL | URL ảnh đại diện |
| `created_at` | `DateTime` | DEFAULT now() | Thời điểm tạo |
| `updated_at` | `DateTime` | AUTO UPDATE | Thời điểm cập nhật cuối |

**Relations**: 1 User → N Recipes, 1 User → N Cookbooks

---

### `recipes` — Công thức nấu ăn

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `recipe_id` | `Int` | PK, AUTOINCREMENT | ID công thức |
| `user_id` | `String` | FK → users | Chủ sở hữu |
| `recipes_name` | `String` | NOT NULL | Tên công thức |
| `description` | `String?` | TEXT | Mô tả |
| `image_recipe` | `String?` | | URL ảnh món ăn |
| `total_time` | `Int?` | | Thời gian nấu (phút) |
| `calories` | `Float?` | | Calo |
| `protein` | `Float?` | | Protein (g) |
| `carbs` | `Float?` | | Carbohydrate (g) |
| `fats` | `Float?` | | Chất béo (g) |
| `source_type` | `SourceType?` | ENUM | Nguồn gốc công thức |
| `number_of_serves` | `Int?` | | Số khẩu phần |
| `created_at` | `DateTime` | DEFAULT now() | |
| `updated_at` | `DateTime` | AUTO UPDATE | |

**Enum `source_type`**: `MANUAL` | `IMPORTED` | `AI_GENERATED`

**Relations**: Recipe → Steps (1:N), Recipe → Tags (N:M via recipe_tags), Recipe → Ingredients (N:M via recipe_ingredients), Recipe → Cookbooks (N:M via cookbook_recipes)

---

### `cookbooks` — Bộ sưu tập công thức

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `cookbook_id` | `Int` | PK, AUTOINCREMENT | ID cookbook |
| `user_id` | `String` | FK → users | Chủ sở hữu |
| `name` | `String` | NOT NULL | Tên cookbook |
| `created_at` | `DateTime` | DEFAULT now(), INDEX | |
| `updated_at` | `DateTime` | AUTO UPDATE | |

---

### `cookbook_recipes` — Junction: Cookbook ↔ Recipe

| Column | Type | Constraints |
|---|---|---|
| `recipe_id` | `Int` | FK → recipes, PK composite |
| `cookbook_id` | `Int` | FK → cookbooks, PK composite |
| `created_at` | `DateTime` | |
| `updated_at` | `DateTime` | |

---

### `steps` — Các bước thực hiện công thức

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `step_id` | `Int` | PK, AUTOINCREMENT | ID bước |
| `recipe_id` | `Int` | FK → recipes | Thuộc công thức |
| `step_number` | `Int` | NOT NULL | Thứ tự bước |
| `instruction` | `String` | TEXT, NOT NULL | Hướng dẫn |
| `tip` | `String?` | TEXT | Mẹo (tùy chọn) |
| `time` | `Int?` | | Thời gian thực hiện (phút) |

---

### `ingredients` — Nguyên liệu

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `ingredient_id` | `Int` | PK, AUTOINCREMENT | ID nguyên liệu |
| `name` | `String` | NOT NULL | Tên nguyên liệu |
| `icon` | `String?` | | Emoji/icon |
| `bg_color` | `String?` | DEFAULT `#F5F5F5` | Màu nền hiển thị |

---

### `recipe_ingredients` — Junction: Recipe ↔ Ingredient

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `recipe_id` | `Int` | FK → recipes, PK composite | |
| `ingredient_id` | `Int` | FK → ingredients, PK composite | |
| `quantity` | `Float?` | | Số lượng |
| `unit` | `String?` | | Đơn vị (g, ml, cái…) |
| `note` | `String?` | | Ghi chú thêm |

---

### `tags` — Nhãn phân loại

| Column | Type | Constraints | Mô tả |
|---|---|---|---|
| `tag_id` | `Int` | PK, AUTOINCREMENT | ID tag |
| `name` | `String` | NOT NULL | Tên tag (VD: "Chay", "Keto") |
| `category` | `String?` | | Nhóm tag |
| `emoji` | `String?` | | Emoji đại diện |

### `recipe_tags` — Junction: Recipe ↔ Tag

| Column | Type | Constraints |
|---|---|---|
| `recipe_id` | `Int` | FK → recipes, PK composite |
| `tag_id` | `Int` | FK → tags, PK composite |

---

## Quy tắc Cascade Delete

| Bảng | Hành vi khi xóa parent |
|---|---|
| `recipes` | Xóa User → Xóa Recipe (Cascade) |
| `steps` | Xóa Recipe → Xóa Steps (Cascade) |
| `recipe_tags` | Xóa Recipe hoặc Tag → Xóa junction (Cascade) |
| `recipe_ingredients` | Xóa Recipe hoặc Ingredient → Xóa junction (Cascade) |
| `cookbook_recipes` | Xóa Recipe hoặc Cookbook → Xóa junction (Cascade) |
| `cookbooks` | Xóa User → Xóa Cookbook (Cascade) |

---

## Ghi Chú Kiến Trúc

- Tất cả foreign keys được **cascade delete** để tránh orphan records.
- `recipe_id`, `cookbook_id`, `step_id`, `tag_id`, `ingredient_id` dùng **Integer autoincrement** (internal IDs).
- `user_id` dùng **UUID string** để tích hợp với Clerk Auth.
- `source_type` enum giúp phân biệt recipe do user nhập tay, import từ nơi khác, hay AI tạo ra.
