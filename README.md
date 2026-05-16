# 🍳 Smart Kitchen Prompts

> **Context Engineering Repository** — Não bộ của hệ thống Multi-Agent Smart Kitchen VN.
> Repo này **không chứa code chạy được**. Nó quản lý ngữ cảnh, kỹ năng, và prompt templates cho toàn bộ hệ thống AI.

---

## 🎯 Mục Đích

`smart-kitchen-prompts` là nơi tập trung toàn bộ **tri thức ngữ cảnh (context knowledge)** của dự án Smart Kitchen VN, được thiết kế để:

| Đối tượng | Mục đích |
|---|---|
| 🤖 **AI Agent** | Đọc `context/` để nạp ngữ cảnh dự án trước khi làm việc |
| 👨‍💻 **Developer** | Tra cứu skill cards để biết cách giao đúng task cho đúng agent |
| 📝 **Prompt Engineer** | Thêm/cập nhật prompts và skills theo chuẩn Vibe Coding |

> **Tại sao cần repo này?** Thay vì lặp lại context dự án trong mỗi cuộc hội thoại với AI, chúng ta lưu tất cả vào đây và AI chỉ cần đọc một lần để hiểu toàn bộ hệ thống.

---

## 📁 Cấu Trúc Thư Mục

```
smart-kitchen-prompts/
│
├── 📄 README.md              ← Bạn đang đọc file này
├── 📄 CHANGELOG.md           ← Lịch sử thay đổi prompt/context (luôn cập nhật)
│
├── 📂 context/               ← "ROM" của hệ thống — AI ĐỌC ĐẦU TIÊN
│   ├── CLAUDE.md             ← ⭐ Global rules — AI đọc TRƯỚC TIÊN
│   ├── AGENTS.md             ← Định nghĩa agents & kiến trúc MSA
│   ├── ERD.md                ← Database schema (PostgreSQL, 8 bảng)
│   └── FLOW.md               ← 5 luồng nghiệp vụ chính (user journeys)
│
├── 📂 skills/                ← Skill cards: vai trò & quy tắc từng AI agent
│   ├── 01-Agent-api-endpoint.md   ← [AGENT-API] Người gác cổng API
│   ├── 02-Agent-ui.md             ← [AGENT-UI] Chuyên gia Frontend
│   ├── 03-Agent-Manage.md         ← [AGENT-MANAGE] Tech Lead & Reviewer
│   ├── 04-Agent-testing.md        ← [AGENT-TESTING] QA & Error Fixer
│   ├── 05-Agent-database-query.md ← [AGENT-DB] Kiến trúc sư Dữ liệu
│   └── 06-Agent-prompt.md         ← [AGENT-PROMPT-ENGINEER] Kỹ sư Phân tích YC
│
├── 📂 prompts/               ← Actionable prompts đã được tinh chỉnh
│   ├── ingredient-extraction-prompt/   ← v1.md → v2.md (versioned)
│   ├── recipe-generation-prompt/
│   └── readme-documentation.md        ← Prompt tạo file README này
│
├── 📂 mcp-config/            ← Cấu hình MCP servers (PostgreSQL MCP + Context7)
│   ├── mcp.json              ← Config file cho MCP client
│   └── Instruction.md        ← Hướng dẫn cài đặt & kết nối MCP
│
└── 📂 experience/            ← Bài học kinh nghiệm, gotchas, best practices
```

---

## 🤖 Hướng Dẫn Sử Dụng — Dành cho AI Agent

### Thứ tự đọc file bắt buộc

Khi bắt đầu một phiên làm việc mới, AI **PHẢI** đọc theo thứ tự sau:

```
1. context/CLAUDE.md    → Global rules (đọc TRƯỚC TIÊN, không bỏ qua)
2. context/AGENTS.md    → Kiến trúc MSA & STRICT 3-FOLDER RULE
3. context/ERD.md       → Schema DB (khi làm task liên quan đến dữ liệu)
4. context/FLOW.md      → Luồng nghiệp vụ (khi làm feature mới)
5. skills/{agent}.md    → Quy tắc cụ thể cho role hiện tại
```

### Nguyên tắc khi làm việc

- ✅ **Luôn** tham chiếu `context/` trước khi đề xuất giải pháp.
- ✅ **Hoạt động đúng vai** — mỗi agent chỉ được phép hoạt động trong phạm vi skill card của mình.
- ✅ **Vibe Coding cycle**: Analyse → Code → Test (xong bước nào báo cho user test trước khi làm tiếp).
- ❌ **Không bịa đặt** thông tin về schema, tech stack, hay kiến trúc — chỉ dựa vào `context/`.

---

## 👨‍💻 Hướng Dẫn Sử Dụng — Dành cho Developer

### Cách giao task đúng cách

**Bước 1**: Copy skill card tương ứng vào đầu cuộc hội thoại với AI:
```
# Ví dụ: Giao task tạo API mới
→ Copy toàn bộ nội dung `skills/01-Agent-api-endpoint.md` vào prompt đầu tiên
→ Sau đó mô tả task cần làm
```

**Bước 2**: Nếu task chưa rõ ràng, dùng `06-Agent-prompt.md` để refine trước:
```
→ Copy `skills/06-Agent-prompt.md` + mô tả ý tưởng mơ hồ
→ AI sẽ sinh ra Actionable Prompt chuẩn cho bạn copy đi dùng
```

### Cách thêm skill mới

1. Tạo file mới trong `skills/` theo format: `{số thứ tự}-Agent-{tên}.md`
2. Cấu trúc file phải bao gồm:
   - `# ROLE: [AGENT-TÊN] - Mô tả vai trò`
   - `# QUY TẮC BẮT BUỘC (STRICT RULES):`
   - `# QUY TRÌNH THỰC THI (VIBE CODING):`
3. Cập nhật `CHANGELOG.md` với entry mới.

### Cách thêm/cập nhật prompt

- Mỗi feature/chức năng có **một thư mục riêng** trong `prompts/`
- Dùng versioning: `v1.md`, `v2.md`, ... khi prompt được cải tiến
- Prompt phải theo format Actionable Prompt chuẩn (xem `skills/06-Agent-prompt.md`)

---

## 🧠 Multi-Agent System — Tóm Tắt

| Agent | Role | Phạm vi hoạt động |
|---|---|---|
| `[AGENT-API]` | Người gác cổng API | `app/api/` — Thin Router ≤20 dòng |
| `[AGENT-UI]` | Chuyên gia Frontend | `Frontend/components/`, `Frontend/hooks/` |
| `[AGENT-MANAGE]` | Tech Lead & Reviewer | Kiểm soát kiến trúc toàn dự án |
| `[AGENT-TESTING]` | QA & Error Fixer | Debug, fix lỗi, verify |
| `[AGENT-DB]` | Kiến trúc sư Dữ liệu | `Backend/services/`, `Backend/schemas/`, `prisma/` |
| `[AGENT-PROMPT-ENGINEER]` | Kỹ sư Phân tích YC | Chuyển idea → Actionable Prompt |

### Orchestrator Flow (thứ tự gọi agents cho một feature mới)

```
[AGENT-PROMPT-ENGINEER]  →  [AGENT-DB]  →  [AGENT-API]  →  [AGENT-UI]
       Refine idea            Schema &         Thin          UI &
       thành prompt           Service          Router        Hooks
                                    ↕
                           [AGENT-TESTING]  ←  Review mọi bước
                           [AGENT-MANAGE]   ←  Giám sát kiến trúc
```

---

## 📐 Quy Ước & Quy Tắc

### Naming Convention

| Loại file | Quy ước | Ví dụ |
|---|---|---|
| Skill card | `{số}-Agent-{tên}.md` | `01-Agent-api-endpoint.md` |
| Prompt | `{feature-name}.md` | `ingredient-extraction-prompt/v2.md` |
| Context | `ALL_CAPS.md` | `AGENTS.md`, `ERD.md` |

### Cập nhật CHANGELOG

Sau mỗi lần thêm/sửa file trong repo này, **bắt buộc** thêm entry vào `CHANGELOG.md`:
```markdown
## [X.Y.Z] — YYYY-MM-DD — Mô tả ngắn

### Loại thay đổi
- Chi tiết thay đổi
```

Phiên bản tăng theo quy tắc:
- **Patch** (`0.0.x`): Sửa lỗi typo, cập nhật nhỏ
- **Minor** (`0.x.0`): Thêm skill/prompt mới
- **Major** (`x.0.0`): Thay đổi kiến trúc agent, refactor lớn

---

## 🔗 Các Repo Liên Quan

| Repo | Mô tả | Quan hệ |
|---|---|---|
| `smart-kitchen-vn` | React Native mobile app | Consumer của API |
| `smart-kitchen-api` | Express.js REST API + Prisma | Backend chính, chứa `schema.prisma` gốc |
| `smart-kitchen-web` | Next.js 14 web app | Frontend chính, áp dụng context từ repo này |
| `smart-kitchen-ai-main` | AI agent implementations | Implement MSA được định nghĩa trong `context/AGENTS.md` |

> **Lưu ý**: File `context/ERD.md` trong repo này được sync từ `smart-kitchen-api/prisma/schema.prisma`. Khi schema DB thay đổi, cần cập nhật `ERD.md` và thêm entry vào `CHANGELOG.md`.

---

<div align ="center">

**Smart Kitchen VN** — *AI-powered cooking, made simple.*

`context/` → `skills/` → `prompts/` → **Code**

</div>
