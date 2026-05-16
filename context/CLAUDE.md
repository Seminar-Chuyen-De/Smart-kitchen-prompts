<!-- Global rules cho Claude Code -->
# CLAUDE — Global Rules for AI Agents

> "ROM" của toàn hệ thống — AI đọc file này TRƯỚC TIÊN trước khi làm bất kỳ việc gì.

---

## 1. Project Identity

- **Project**: Smart Kitchen VN
- **Type**: AI-powered cooking web app (Next.js 14 + Mobile React Native)
- **Core flow**: User upload ảnh nguyên liệu → AI nhận diện → generate recipe → lưu Cookbook.

---

## 2. Kiến trúc BẮT BUỘC — STRICT 3-FOLDER RULE

**TUYỆT ĐỐI** tuân thủ cấu trúc dưới đây. Không được tạo `lib/` hay `components/` ở root.

| Thư mục | Vai trò | Được phép gọi |
|---|---|---|
| `app/` | Routing siêu mỏng (page, layout, API route shell) | `frontend/`, `backend/`, `ai/` |
| `frontend/` | Mọi UI chạy trên browser (`components/`, `hooks/`, `styles/`) | `backend/` (qua fetch), không gọi trực tiếp DB |
| `backend/` | Logic server (`services/`, `schemas/`, `db/`) | DB (Prisma), không gọi AI trực tiếp |
| `ai/` | Multi-Agent System (`agents/`, `tools/`, `workflows/`) | `backend/services/` để lưu dữ liệu |

### Path Aliases (bắt buộc dùng)

```
@/frontend/*   → ./frontend/
@/backend/*    → ./backend/
@/ai/*         → ./ai/
@/prisma       → ./prisma/schema.prisma
```

---

## 3. Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 App Router |
| Styling | Tailwind CSS |
| Database | PostgreSQL + Prisma ORM |
| Auth | Clerk |
| AI Core | Claude Sonnet 4.5 |
| Vision | Google Cloud Vision API |
| MCP | PostgreSQL MCP Server + Context7 |
| Mobile | React Native (Expo) |
| API | RESTful (smart-kitchen-api — Express/Prisma) |

---

## 4. Multi-Agent Architecture (MSA)

```
[Orchestrator Agent]  (ai/agents/orchestrator.agent.ts)
    ├── [Vision Agent]     → Nhận diện nguyên liệu từ ảnh (Google Vision API)
    ├── [Recipe Agent]     → Generate recipe từ danh sách nguyên liệu (Claude Sonnet)
    └── [Storage Agent]   → Gọi backend/services/recipe.service.ts để lưu DB
```

- Agent giao tiếp **qua message**, không share state trực tiếp.
- Orchestrator là entry point duy nhất cho AI workflow.

---

## 5. Coding Rules — DO NOT

- ❌ Không mock DB trong API tests (dùng test DB thực).
- ❌ Không share state giữa các tests.
- ❌ Không hardcode API keys — luôn dùng `process.env.*`.
- ❌ Không viết UI hay business logic trong Next.js Route Handlers (`app/api/`).
- ❌ Không gọi Prisma trực tiếp từ `frontend/` hay `ai/`.
- ❌ Không tạo thư mục mới ngoài 3 thư mục lõi mà không có lý do rõ ràng.

---

## 6. Coding Rules — MUST DO

- ✅ Mỗi service phải có Zod schema validation tương ứng tại `backend/schemas/`.
- ✅ Custom hooks chỉ được đặt trong `frontend/hooks/`.
- ✅ Mọi lỗi phải được xử lý và trả về error response chuẩn.
- ✅ Dùng TypeScript strict mode.
- ✅ Tên file: `kebab-case.ts`, tên class/component: `PascalCase`.
- ✅ Commit message format: `feat|fix|chore|refactor: mô tả ngắn`.

---

## 7. Vibe Coding Workflow

Với mỗi task mới, tuân thủ chu trình **Analyse → Code → Test**:

1. **Analyse**: Đọc `AGENTS.md`, `ERD.md`, `FLOW.md` trước khi code.
2. **Code**: Viết theo thứ tự: Schema → Service → API Route → UI Component.
3. **Test**: Chạy unit test (backend) + E2E test (frontend) trước khi PR.

---

## 8. Files Context Quan Trọng

| File | Nội dung |
|---|---|
| `AGENTS.md` | Định nghĩa agents & kiến trúc hệ thống |
| `ERD.md` | Database schema (Entity Relationship Diagram) |
| `FLOW.md` | Luồng nghiệp vụ chính (user journey & data flow) |
| `CLAUDE.md` | File này — Global rules (đọc đầu tiên) |
