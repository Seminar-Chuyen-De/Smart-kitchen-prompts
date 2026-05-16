<!-- "ROM" của toàn hệ thống -->
# Smart Kitchen Web — Project Context

## Project Overview
AI-powered cooking web app sử dụng Multi-Agent Architecture.
Users upload ảnh nguyên liệu → AI nhận diện → generate recipe → lưu Cookbook.

## Tech Stack
- Framework: Next.js 14 App Router (app/ chỉ dùng làm routing mỏng)
- Styling: Tailwind CSS
- Database: PostgreSQL + Prisma ORM
- Auth: Clerk
- AI Core: Claude Sonnet 4.5
- Vision: Google Cloud Vision API
- MCP: PostgreSQL MCP Server + Context7

## Architecture: STRICT 3-FOLDER RULE
TUYỆT ĐỐI tuân thủ cấu trúc sau, không tạo thư mục `lib/` hay `components/` ở root:
1. `app/`: Lớp routing siêu mỏng. Không chứa logic. Chỉ import từ 3 thư mục bên dưới.
2. `frontend/`: Chứa mọi thứ chạy trên browser (`components/`, `hooks/`, `styles/`).
3. `backend/`: Chứa logic server (`services/`, `schemas/`, `db/`). Không gọi AI từ đây.
4. `ai/`: Chứa Multi-Agent System (`agents/`, `tools/`, `workflows/`).

## MSA Architecture
[Orchestrator Agent] (ai/agents/orchestrator.agent.ts)
    ├── [Vision Agent]     → Nhận diện nguyên liệu từ ảnh
    ├── [Recipe Agent]     → Generate recipe từ ingredients list
    └── [Storage Agent]    → Gọi backend/services/recipe.service.ts để lưu

## Path Aliases
Bắt buộc dùng: `@/frontend/*`, `@/backend/*`, `@/ai/*`, `@/prisma`

## DO NOT
- Không mock DB trong API tests.
- Không share state giữa các tests.
- Không hardcode API keys (dùng ENV).
- Không viết UI/Business logic trực tiếp vào Next.js Route Handlers.

