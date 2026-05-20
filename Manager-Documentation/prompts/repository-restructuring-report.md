# Báo Cáo Tái Cấu Trúc Repository Theo 4 Vai Trò Đội Ngũ Phát Triển

## 1. Tổng Quan Chiến Lược (Strategic Overview)

Repository `Smart-kitchen-prompts` đóng vai trò là **ROM Tri thức (Context Engineering)** của toàn bộ hệ thống Multi-Agent Smart Kitchen VN. Để tối ưu hóa quy trình làm việc phối hợp giữa con người và các AI Agents chuyên biệt, repository đã được tái cấu trúc hoàn thiện từ sơ đồ phẳng sang phân vùng 4 vai trò cốt lõi đại diện cho một đội ngũ phát triển 4 thành viên thực tế (Frontend, Backend, AI-training, Manager-Documentation).

Đặc biệt, thư mục ngữ cảnh chung `context/` được duy trì tại gốc ngoài cùng làm cổng vào (Context Entrypoint) tối quan trọng để tất cả các AI Agents bắt buộc phải nạp trước tiên khi bắt đầu phiên làm việc.

---

## 2. Sơ Đồ Cấu Trúc Hoàn Thiện (Final Directory Architecture)

```
smart-kitchen-prompts/
│
├── 📄 README.md                     ← Tài liệu hướng dẫn cấu trúc & vận hành mới
├── 📄 CHANGELOG.md                  ← Nhật ký cập nhật phiên bản (Refactoring v1.0.0)
│
├── 📂 context/                      ← ⭐ GLOBAL CONTEXT (ROM hệ thống - AI đọc đầu tiên)
│   ├── CLAUDE.md                    ← Global rules & Quy tắc viết mã
│   ├── AGENTS.md                    ← Định nghĩa các AI Agents trong MSA
│   ├── FLOW.md                      ← Sơ đồ luồng nghiệp vụ & danh sách APIs
│   └── ERD.md                       ← Lược đồ thực thể quan hệ Database (8 bảng)
│
├── 📂 Frontend/                     ← 🎨 TÀI NGUYÊN FRONTEND DEVELOPER
│   ├── 📂 prompts/                  ← Prompts thiết kế UI/UX & CRUD (p1, p2, p3)
│   ├── 📂 skills/                   ← Skill card: 02-Agent-ui.md
│   └── 📂 experience/               ← Lịch sử gỡ lỗi client & form edit recipe
│
├── 📂 Backend/                      ← 🗄️ TÀI NGUYÊN BACKEND DEVELOPER & QA
│   ├── 📂 skills/                   ← Skill cards: 01-api-endpoint, 04-testing, 05-database-query
│   └── 📂 experience/               ← Sửa lỗi DB, Clerk middleware, NextConfig và API testing
│
├── 📂 AI-training/                  ← 🧠 TÀI NGUYÊN AI PROMPT ENGINEER
│   ├── 📂 prompts/                  ← Pipeline prompts: trích xuất nguyên liệu, gợi ý công thức
│   ├── 📂 skills/                   ← Skill card: 06-Agent-prompt.md (Phân tích Yêu cầu)
│   └── 📂 experience/               ← Test prompts, giải pháp rate limits & model switching
│
└── 📂 Manager-Documentation/        ← 📋 TÀI NGUYÊN PROJECT MANAGER & TECH WRITER
    ├── 📂 prompts/                  ← Backlogs, báo cáo tiến độ, viết báo cáo Word seminar
    ├── 📂 skills/                   ← Skill cards: 03-Agent-Manage, 07-Agent-git-workflow
    └── 📂 mcp-config/               ← Cấu hình MCP servers (mcp.json, Instruction)
```

---

## 3. Bản Đồ Phân Phối Tệp Tin Chi Tiết (Detailed File Mapping)

| Phân Vùng | Tệp Tin Nguồn (Cũ) | Đường Dẫn Mới |
|---|---|---|
| **Context (Root)** | `context/CLAUDE.md` | `context/CLAUDE.md` (Giữ ở gốc) |
| | `context/AGENTS.md` | `context/AGENTS.md` (Giữ ở gốc) |
| | `context/FLOW.md` | `context/FLOW.md` (Giữ ở gốc) |
| | `Backend/context/ERD.md` | `context/ERD.md` (Di chuyển từ Backend ra gốc) |
| **Frontend** | `prompts/p1-ui-core-features.md` | `Frontend/prompts/p1-ui-core-features.md` |
| | `prompts/p2-ux-enhancement-ui.md` | `Frontend/prompts/p2-ux-enhancement-ui.md` |
| | `prompts/p3-edit-user-recipe.md` | `Frontend/prompts/p3-edit-user-recipe.md` |
| | `prompts/hot-fix-ui.md` | `Frontend/prompts/hot-fix-ui.md` |
| | `prompts/hot-fix-UI-reload-after-using-function.md` | `Frontend/prompts/hot-fix-UI-reload-after-using-function.md` |
| | `skills/02-Agent-ui.md` | `Frontend/skills/02-Agent-ui.md` |
| | `experience/edit-recipe-not-saving.md` | `Frontend/experience/edit-recipe-not-saving.md` |
| **Backend** | `skills/01-Agent-api-endpoint.md` | `Backend/skills/01-Agent-api-endpoint.md` |
| | `skills/05-Agent-database-query.md` | `Backend/skills/05-Agent-database-query.md` |
| | `skills/04-Agent-testing.md` | `Backend/skills/04-Agent-testing.md` |
| | `experience/01-clerk-api-auth-deprecation.md` | `Backend/experience/01-clerk-api-auth-deprecation.md` |
| | `experience/02-next-config-compilation-standard.md` | `Backend/experience/02-next-config-compilation-standard.md` |
| | `experience/03-zod-schema-relative-path-flexibility.md` | `Backend/experience/03-zod-schema-relative-path-flexibility.md` |
| | `experience/api-endpoints-testing.md` | `Backend/experience/api-endpoints-testing.md` |
| | `experience/api-test-results-p2.md` | `Backend/experience/api-test-results-p2.md` |
| | `experience/cookbook-api-test.md` | `Backend/experience/cookbook-api-test.md` |
| | `experience/cookbook-service-test.md` | `Backend/experience/cookbook-service-test.md` |
| | `experience/database-schema.md` | `Backend/experience/database-schema.md` |
| **AI-training** | `skills/06-Agent-prompt.md` | `AI-training/skills/06-Agent-prompt.md` |
| | `prompts/ingredient-extraction-prompt/` | `AI-training/prompts/ingredient-extraction-prompt/` |
| | `prompts/recipe-generation-prompt/` | `AI-training/prompts/recipe-generation-prompt/` |
| | `experience/ingredient-extraction-prompt/test.md` | `AI-training/experience/ingredient-extraction-prompt/test.md` |
| | `experience/recipe-generation-prompt/test.md` | `AI-training/experience/recipe-generation-prompt/test.md` |
| | `experience/04-gemini-rate-limits-and-model-switching.md` | `AI-training/experience/04-gemini-rate-limits-and-model-switching.md` |
| **Manager-Doc** | `skills/03-Agent-Manage.md` | `Manager-Documentation/skills/03-Agent-Manage.md` |
| | `skills/07-Agent-git-workflow.md` | `Manager-Documentation/skills/07-Agent-git-workflow.md` |
| | `prompts/BACKLOG.md` | `Manager-Documentation/prompts/BACKLOG.md` |
| | `prompts/backlog-analysis.md` | `Manager-Documentation/prompts/backlog-analysis.md` |
| | `prompts/readme-documentation.md` | `Manager-Documentation/prompts/readme-documentation.md` |
| | `prompts/report-writing-prompt.md` | `Manager-Documentation/prompts/report-writing-prompt.md` |
| | `prompts/vibe-coding-agents-application.md` | `Manager-Documentation/prompts/vibe-coding-agents-application.md` |
| | `prompts/vibe-coding-prompts-log.md` | `Manager-Documentation/prompts/vibe-coding-prompts-log.md` |
| | `prompts/report/` | `Manager-Documentation/prompts/report/` |
| | `mcp-config/` | `Manager-Documentation/mcp-config/` |

---

## 4. Bảo Toàn Lịch Sử Git (Git History Preservation)

Để đảm bảo tính toàn vẹn của lịch sử phát triển dự án, toàn bộ quá trình tái cấu trúc được thực hiện nghiêm ngặt thông qua lệnh:
```bash
git mv <old_path> <new_path>
```
Hành động này giúp:
- Bảo toàn 100% lịch sử commit và tác giả (`git log` & `git blame`) trên từng dòng tài liệu.
- Git tự động ghi nhận dưới dạng `renamed` thay vì coi là hành vi xóa tệp cũ và tạo mới tệp trắng, giữ cho đồ thị Git (Git graph) luôn sạch và chính xác.

---

## 5. Quy Trình Vận Hành Mới Cho AI & Developer

1. **AI Agents**: Khi bắt đầu, Agent **PHẢI** đọc `context/CLAUDE.md` đầu tiên để hiểu quy chuẩn viết mã, sau đó tham chiếu đến phân vùng vai trò tương ứng trong `Frontend/`, `Backend/`, hoặc `AI-training/` để áp dụng skill card và xem bài học kinh nghiệm (`experience/`) để tránh mắc lại lỗi cũ.
2. **Developers**: Áp dụng quy trình Vibe Coding tuần tự từ dịch chuyển ý tưởng (`AI-training/skills/06-Agent-prompt.md`), viết code (`Backend/skills/...`, `Frontend/skills/...`), kiểm thử QA (`Backend/skills/04-Agent-testing.md`), đến quản lý phiên bản đẩy code (`Manager-Documentation/skills/07-Agent-git-workflow.md`).
