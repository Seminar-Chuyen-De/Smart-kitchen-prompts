# 🍳 Smart Kitchen Prompts

> **Context Engineering Repository** — Não bộ & Ngữ cảnh học thuật của hệ thống Multi-Agent Smart Kitchen VN.
> Repo này **không chứa code chạy được**. Nó quản lý ngữ cảnh, kỹ năng, các Actionable Prompt, và bài học kinh nghiệm cho toàn bộ hệ thống AI.

---

## 🎯 Bản Đồ Vai Trò & Cấu Trúc Repository

Repository này được cấu trúc lại hoàn chỉnh theo quy trình làm việc thực tế của một **Đội ngũ phát triển 4 thành viên (Con người + AI Agents)**, giữ nguyên thư mục `context/` ở gốc ngoài cùng để làm điểm bắt đầu tối quan trọng cho mọi AI Agent nạp ngữ cảnh:

```
smart-kitchen-prompts/
│
├── 📄 README.md                     ← Bạn đang đọc file này
├── 📄 CHANGELOG.md                  ← Ghi nhận lịch sử thay đổi (phiên bản v1.0.0)
│
├── 📂 context/                      ← ⭐ ROM TỐI QUAN TRỌNG CỦA HỆ THỐNG — AI ĐỌC ĐẦU TIÊN
│   ├── CLAUDE.md                    ← Hướng dẫn chung và cấu trúc tech stack toàn dự án
│   ├── AGENTS.md                    ← Định nghĩa các AI Agents & kiến trúc Multi-Agent (MSA)
│   ├── FLOW.md                      ← Sơ đồ và mô tả chi tiết 5 luồng nghiệp vụ cốt lõi
│   └── ERD.md                       ← Bản thiết kế cơ sở dữ liệu quan hệ thực thể (8 bảng)
│
├── 📂 Frontend/                     ← 🎨 Tài nguyên dành cho Frontend Developer
│   ├── 📂 prompts/                  ← Prompts UI/UX, Core features (p1, p2, p3, hotfixes)
│   ├── 📂 skills/                   ← Skill card: 02-Agent-ui.md
│   └── 📂 experience/               ← Lịch sử & giải pháp sửa lỗi giao diện client
│
├── 📂 Backend/                      ← 🗄️ Tài nguyên dành cho Backend Developer & QA
│   ├── 📂 skills/                   ← Skill cards: 01-Agent-api-endpoint.md, 04-Agent-testing.md, 05-Agent-database-query.md
│   └── 📂 experience/               ← Lịch sử sửa lỗi DB, Clerk middleware, NextConfig và API testing
│
├── 📂 AI-training/                  ← 🧠 Tài nguyên dành cho AI Prompt Engineer
│   ├── 📂 prompts/                  ← Versioned prompts: nhận diện nguyên liệu, gợi ý công thức (v1, v2, Eval)
│   ├── 📂 skills/                   ← Skill card: 06-Agent-prompt.md (Kỹ sư Phân tích & Phác thảo Yêu cầu)
│   └── 📂 experience/               ← Kết quả test prompts, giải pháp rate limits & model switching
│
└── 📂 Manager-Documentation/        ← 📋 Tài nguyên dành cho Project Manager & Tech Writer
    ├── 📂 prompts/                  ← Backlogs, báo cáo tiến độ, viết báo cáo Word (seminar 5 chương)
    ├── 📂 skills/                   ← Skill cards: 03-Agent-Manage.md, 07-Agent-git-workflow.md
    └── 📂 mcp-config/               ← Cấu hình kết nối Model Context Protocol (mcp.json, Instruction.md)
```

---

## 📁 Chi Tiết Phân Phối Tệp Tin Theo Vai Trò

### ⭐ 0. Global Context Layer (ROM hệ thống)
*Thư mục tối quan trọng nằm tại gốc ngoài cùng để mọi AI-Skill khi khởi chạy đều bắt buộc phải đọc trước tiên để nắm bắt toàn cảnh cấu trúc, tech stack, sơ đồ database và nghiệp vụ cốt lõi của Smart Kitchen VN.*
*   `context/CLAUDE.md`: Global rules quy định tech stack, cấu trúc 3 thư mục cứng (`Frontend/`, `Backend/`, `AI/`), và quy tắc viết mã.
*   `context/AGENTS.md`: Thiết kế và phân chia nhiệm vụ cho 4 Agents trong hệ thống Multi-Agent (MSA).
*   `context/FLOW.md`: Mô tả chi tiết 5 luồng hoạt động chính (AI Scan, Manual Form, Cookbook, Clerk Auth, Search & Filter) kèm danh sách APIs.
*   `context/ERD.md`: Bản thiết kế cơ sở dữ liệu quan hệ thực thể (8 bảng PostgreSQL, kho dữ liệu chính).

### 🎨 1. Frontend Developer (UI/UX & Client Logic)
*Chịu trách nhiệm thiết kế giao diện React/Next.js, hiệu ứng hoạt hoạt ảnh, custom hooks, đồng bộ dữ liệu client, và xử lý các hotfix hiển thị.*
*   `Frontend/prompts/p1-ui-core-features.md`: Thiết lập giao diện trang chủ, scan ảnh, danh sách món ăn.
*   `Frontend/prompts/p2-ux-enhancement-ui.md`: Thêm loading state, thông báo toast, trang cá nhân (profile).
*   `Frontend/prompts/p3-edit-user-recipe.md`: Biểu mẫu chỉnh sửa chi tiết món ăn.
*   `Frontend/prompts/hot-fix-ui.md`: Sửa lỗi hiển thị UI.
*   `Frontend/prompts/hot-fix-UI-reload-after-using-function.md`: Sửa lỗi đồng bộ giao diện sau khi trigger hàm.
*   `Frontend/skills/02-Agent-ui.md`: **[AGENT-UI]** - Chuyên gia viết mã giao diện React và CSS Tailwind.
*   `Frontend/experience/edit-recipe-not-saving.md`: Bài học kinh nghiệm xử lý lỗi form edit recipe không đồng bộ payload.

### 🗄️ 2. Backend Developer & QA (Data, Security & Server Routing)
*Chịu trách nhiệm thiết kế schema PostgreSQL, truy vấn Prisma, xác thực Clerk, API route thin controller, và thực hiện kiểm thử an toàn/hiệu năng.*
*   `Backend/skills/01-Agent-api-endpoint.md`: **[AGENT-API]** - Viết route controller mỏng đảm bảo an toàn biên dịch.
*   `Backend/skills/05-Agent-database-query.md`: **[AGENT-DB]** - Truy vấn cơ sở dữ liệu Prisma an toàn.
*   `Backend/skills/04-Agent-testing.md`: **[AGENT-TESTING]** - QA/QC chạy lệnh verify và viết log kinh nghiệm gỡ lỗi.
*   `Backend/experience/01-clerk-api-auth-deprecation.md`: Khắc phục lỗi deprecation cú pháp hàm `auth()` của Clerk Middleware v5.
*   `Backend/experience/02-next-config-compilation-standard.md`: Chuẩn hóa tệp tin cấu hình NextConfig mjs trong môi trường TS.
*   `Backend/experience/03-zod-schema-relative-path-flexibility.md`: Khắc phục lỗi validate Zod schema chặn đường dẫn ảnh nội bộ tương đối.
*   `Backend/experience/database-schema.md`: Lịch sử thiết kế và đồng bộ database.
*   `Backend/experience/*testing.md` & `*test.md`: Nhật ký kiểm thử API, Cookbook service, và kết quả kiểm thử pha P2.

### 🧠 3. AI-training (Prompt Engineer & LLM Pipeline Integrator)
*Chịu trách nhiệm tối ưu hóa prompt hệ thống, tinh chỉnh pipeline trích xuất nguyên liệu từ ảnh chụp, cấu trúc logic tạo công thức nấu ăn, và xây dựng cơ chế dự phòng API.*
*   `AI-training/skills/06-Agent-prompt.md`: **[AGENT-PROMPT-ENGINEER]** - Kỹ sư dịch ý tưởng thành Actionable Prompt chuẩn.
*   `AI-training/prompts/ingredient-extraction-prompt/`: Thư mục prompts versioned + file đánh giá (Eval) nhận diện nguyên liệu.
*   `AI-training/prompts/recipe-generation-prompt/`: Thư mục prompts versioned + file đánh giá (Eval) gợi ý món ăn.
*   `AI-training/experience/ingredient-extraction-prompt/test.md`: Kết quả test trích xuất nhãn ảnh (label detection).
*   `AI-training/experience/recipe-generation-prompt/test.md`: Kết quả test đề xuất công thức.
*   `AI-training/experience/04-gemini-rate-limits-and-model-switching.md`: Giải pháp xử lý lỗi API rate limits (429) và cơ chế chuyển đổi thông minh giữa Gemini 2.5 Flash và 1.5 Flash.

### 📋 4. Manager & Documentation (PM, Architect & Tech Writer)
*Chịu trách nhiệm quản lý tiến độ backlog, quy định quy trình làm việc nhóm, cấu hình MCP để mở rộng tri thức AI, và soạn thảo báo cáo học thuật.*
*   `Manager-Documentation/prompts/BACKLOG.md`: Bảng quản lý 80+ đầu việc cần làm (chia theo thứ tự P0 -> P3).
*   `Manager-Documentation/prompts/backlog-analysis.md`: Prompt phân tích mã nguồn để tự động gen ra backlog.
*   `Manager-Documentation/prompts/readme-documentation.md`: Prompt sinh tệp README gốc này.
*   `Manager-Documentation/prompts/report-writing-prompt.md`: Prompt sinh tài liệu báo cáo seminar.
*   `Manager-Documentation/prompts/report/`: Bản thảo báo cáo seminar 5 chương (từ Tổng quan đến Kết luận).
*   `Manager-Documentation/prompts/vibe-coding-agents-application.md`: Báo cáo ứng dụng thực tế Multi-Agent System môn Vibe Coding.
*   `Manager-Documentation/prompts/vibe-coding-prompts-log.md`: Nhật ký hội thoại (Prompts Log) minh chứng quy trình Vibe Coding.
*   `Manager-Documentation/skills/03-Agent-Manage.md`: **[AGENT-MANAGE]** - Giám sát cấu trúc folder và review Pull Request.
*   `Manager-Documentation/skills/07-Agent-git-workflow.md`: **[AGENT-GIT-WORKFLOW]** - Quản lý nhánh, commit chuẩn Conventional, và đẩy code.
*   `Manager-Documentation/mcp-config/`: Cấu hình kết nối cổng database/Context7 MCP.

---

## 🤖 Hướng Dẫn Sử Dụng Dành Cho AI Agent

Khi bắt đầu một phiên làm việc mới, AI **PHẢI** nạp ngữ cảnh dự án theo đúng thứ tự ưu tiên:

1. Đọc tệp cấu trúc toàn cục ở gốc ngoài cùng: `context/CLAUDE.md`.
2. Đọc kiến trúc MSA để hiểu cách phối hợp: `context/AGENTS.md`.
3. Tùy thuộc vào phân vùng công việc được giao, truy cập vào thư mục tương ứng (Frontend / Backend / AI-training / Manager-Documentation) để đọc các tài liệu hướng dẫn (`skills/`) và các bài học kinh nghiệm (`experience/`) để tránh lặp lại các bug cũ.

---

## 👨‍💻 Hướng Dẫn Sử Dụng Dành Cho Developer

### Quy trình Vibe Coding chuẩn hóa:
1. **Bước 1 (Phân tích)**: Sử dụng kỹ sư phân tích `AI-training/skills/06-Agent-prompt.md` để chuyển đổi ý tưởng thô sơ của bạn thành **Actionable Prompt** chi tiết.
2. **Bước 2 (Thực thi)**: Gửi Actionable Prompt cho các Agent chuyên môn (`Backend/skills/...`, `Frontend/skills/...`, `AI-training/skills/...`) để viết code lớp Schema, Service, API routes và UI Components theo đúng thứ tự.
3. **Bước 3 (Kiểm thử & Bàn giao)**: Dùng QA Agent `Backend/skills/04-Agent-testing.md` để test, ghi kinh nghiệm sửa lỗi vào `experience/` nếu có, và dùng `Manager-Documentation/skills/07-Agent-git-workflow.md` để checkout nhánh và push PR lên GitHub.

---

<div align ="center">

**Smart Kitchen VN** — *Hệ thống Multi-Agent nấu ăn thông minh hàng đầu.*
*Quy chuẩn - Chuyên nghiệp - Hiệu suất cao.*

</div>
