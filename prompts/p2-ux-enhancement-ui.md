# Prompt: P2 — UX Enhancement UI + Git Workflow

> **Sinh bởi**: [AGENT-PROMPT-ENGINEER]  
> **Ngày**: 2026-05-18  
> **Dành cho Agent**: `[AGENT-UI]` + `[AGENT-GIT-WORKFLOW]`  
> **BACKLOG Ref**: P2-1, P2-2, P2-4

---

## PHẦN 1 — ACTIONABLE PROMPT: P2 UX Enhancement UI

---

**MỤC TIÊU:**  
Hoàn thành toàn bộ lớp **P2 — UX Enhancement** (Frontend-only tasks) cho dự án Smart Kitchen VN, bao gồm:
1. **P2-1**: Tạo bộ Shared UI Components tái sử dụng (Toast, Modal, Skeleton, Button, Badge, EmptyState) + ToastContext.
2. **P2-2**: Tạo các trang xử lý lỗi/loading cấp `app/` (error.tsx, not-found.tsx, loading.tsx).
3. **P2-4**: Tạo trang Profile người dùng (`ProfilePage.tsx` + route).

Sau khi hoàn thành tất cả UI, thực hiện **PHẦN 2** để merge các nhánh hiện có và tạo nhánh feature mới.

---

**PHÂN TÍCH KIẾN TRÚC & CÁC BƯỚC THỰC THI:**  
*(Xong bước nào, dừng lại để test rồi mới làm tiếp)*

---

### BƯỚC 1 — Shared UI Components (`Frontend/components/ui/`)  
> **BACKLOG**: P2-1

Tạo 6 component cơ sở tái sử dụng. **Chưa có thư mục `ui/` nào**, cần tạo mới.

#### 1.1 — `Frontend/components/ui/Button.tsx`
- Unified button với các variants: `primary` | `secondary` | `ghost` | `danger`
- Prop: `variant`, `size` (`sm` | `md` | `lg`), `isLoading` (hiển thị spinner), `disabled`, `leftIcon`, `rightIcon`
- Dùng `forwardRef`, typed đầy đủ TypeScript
- **Test ngay**: Import vào `DashboardPage.tsx` thay thế 1 button hiện có, kiểm tra render đúng variant

#### 1.2 — `Frontend/components/ui/Badge.tsx`
- Hiển thị nhãn nguồn gốc: `AI` (màu tím/violet), `Manual` (màu xanh/blue), `Pending` (vàng/amber)
- Prop: `variant: 'ai' | 'manual' | 'pending' | 'success' | 'error'`, `label: string`
- **Test ngay**: Dùng trong `RecipeCard.tsx` để hiển thị nguồn gốc recipe

#### 1.3 — `Frontend/components/ui/Skeleton.tsx`
- Skeleton placeholder với animation `pulse`
- Variants: `text` (dòng chữ), `card` (card box), `avatar` (hình tròn)
- Prop: `variant`, `width?`, `height?`, `className?`
- **Test ngay**: Thay thế loading state trong `RecipeList.tsx`

#### 1.4 — `Frontend/components/ui/EmptyState.tsx`
- Empty state với icon, tiêu đề, mô tả, và optional action button
- Prop: `icon: ReactNode`, `title: string`, `description: string`, `action?: { label: string; onClick: () => void }`
- **Test ngay**: Dùng trong `RecipeList.tsx` khi danh sách rỗng

#### 1.5 — `Frontend/components/ui/Modal.tsx`
- Reusable dialog/modal với backdrop blur
- Prop: `isOpen: boolean`, `onClose: () => void`, `title?: string`, `size?: 'sm' | 'md' | 'lg' | 'full'`, `children: ReactNode`
- Hỗ trợ đóng bằng ESC key và click outside
- `'use client'` — dùng `useEffect` để trap focus
- **Test ngay**: Bọc `RecipeForm.tsx` trong Modal, kiểm tra open/close

#### 1.6 — `Frontend/components/ui/Toast.tsx` + `Frontend/contexts/ToastContext.tsx`
- `ToastContext.tsx`: Context Provider quản lý global toast state
  - `addToast(message, type: 'success' | 'error' | 'info' | 'warning')`
  - `removeToast(id)`
  - Auto-remove sau 4000ms
- `Toast.tsx`: Component render list toast (fixed bottom-right), animation slide-in/out
- **Export hook**: `useToast()` từ `ToastContext.tsx`
- **Test ngay**: Wrap `app/dashboard/layout.tsx` với `ToastProvider`, gọi `useToast().addToast(...)` từ DashboardPage

---

### BƯỚC 2 — Error Handling & Loading States (`app/`)  
> **BACKLOG**: P2-2 (Frontend tasks only — bỏ qua `Backend/middleware/` và `Backend/constants/`)

#### 2.1 — `app/loading.tsx`
- Global loading page cho toàn bộ `app/`
- Hiển thị centered spinner + text "Đang tải..."
- Dùng component `Skeleton` đã tạo ở Bước 1

#### 2.2 — `app/not-found.tsx`
- Custom 404 page
- Hiển thị: icon lớn (ví dụ 🍳 hoặc Lucide `UtensilsCrossed`), text "404 — Trang không tồn tại", link về `/dashboard`
- Dùng component `Button` đã tạo ở Bước 1

#### 2.3 — `app/error.tsx`
- `'use client'` — đây là Error Boundary của Next.js
- Props: `error: Error & { digest?: string }`, `reset: () => void`
- Hiển thị: icon cảnh báo, mô tả lỗi, nút "Thử lại" gọi `reset()`
- **Test ngay**: Truy cập route không hợp lệ để kiểm tra 404, force throw error để kiểm tra error boundary

---

### BƯỚC 3 — User Profile Page  
> **BACKLOG**: P2-4 (Frontend tasks only — bỏ qua API route vì chưa có `app/api/user/profile`)

#### 3.1 — `Frontend/components/pages/ProfilePage.tsx`
- `'use client'`
- Hiển thị thông tin user từ Clerk hook `useUser()` (KHÔNG fetch từ API — đây là placeholder trước khi có API)
- Layout: Avatar lớn, tên, email, ngày tham gia
- Section thống kê: "Số công thức đã tạo", "Số cookbook" — dùng giá trị static/mock `(0)` kèm chú thích `// TODO: fetch từ /api/user/profile`
- Nút "Chỉnh sửa hồ sơ" mở `Modal` (dùng component đã tạo ở Bước 1) — form chỉ có trường `displayName` (mock, chưa submit API)
- Dùng `Button`, `Modal`, `Skeleton` từ `@/Frontend/components/ui/`

#### 3.2 — `app/dashboard/profile/page.tsx`
- Route page mỏng, chỉ import và render `ProfilePage`
- Thêm `export const metadata = { title: 'Hồ sơ — Smart Kitchen VN' }`

---

**RÀNG BUỘC (CONSTRAINTS BẮT BUỘC):**
- **Không gian làm việc**: CHỈ được tạo/sửa file trong `Frontend/` và `app/` (error.tsx, not-found.tsx, loading.tsx). KHÔNG động đến `Backend/`.
- **Path alias**: Chỉ dùng `@/Frontend/*` để import nội bộ giữa các component.
- **TypeScript**: Tất cả file phải có type annotation đầy đủ, không dùng `any`.
- **Công nghệ**: Tailwind CSS, Lucide Icons, Radix UI (nếu cần cho Modal/Toast). KHÔNG cài thêm thư viện mới nếu Radix UI đã đủ.
- **Client/Server**: Mọi component có state/effect phải có `'use client'` ở đầu file.
- **KHÔNG gọi DB/API trực tiếp**: Mọi dữ liệu đến từ `useUser()` (Clerk) hoặc giá trị mock có comment `// TODO`.
- **Không nhồi business logic vào `app/`**: `app/dashboard/profile/page.tsx` chỉ import và render component từ `Frontend/`.

---

## PHẦN 2 — GIT WORKFLOW: Merge Nhánh & Tạo Nhánh Feature P2

> **Agent**: `[AGENT-GIT-WORKFLOW]`

---

### Bối cảnh hiện tại (Trạng thái Repo `smart-kitchen-web`)

| Nhánh | Commit mới nhất | Nội dung |
|---|---|---|
| `main` | `6d1a625` | Foundation khởi tạo |
| `feature/database-schema` *(remote only)* | `6f80e0b` | DB schema, Zod schemas, seed, service layer |
| `feature/api-endpoints-tests` *(local + remote)* | `d7036bb` | API endpoints + unit tests |
| `feature/p0-p1-core-ui` *(current, local + remote)* | `fa1b153` | P0-2 + P1 core UI |

**Nhiệm vụ**: Merge `feature/database-schema` + `feature/api-endpoints-tests` + `feature/p0-p1-core-ui` vào một nhánh chức năng mới tên `feature/p2-ux-enhancement` để làm việc tiếp với P2 UI. **Chưa push nhánh mới lên remote** cho đến khi toàn bộ tiến trình P2 hoàn tất.

---

### Script Git — Thực hiện tuần tự

```bash
# Bước 1: Cập nhật main và fetch toàn bộ remote
git checkout main && git pull origin main && git fetch --all

# Bước 2: Tạo nhánh feature/p2-ux-enhancement từ main
git checkout -b feature/p2-ux-enhancement

# Bước 3: Merge feature/database-schema (remote only)
git merge origin/feature/database-schema --no-ff -m "feat: merge feature/database-schema vào p2-ux-enhancement"

# Bước 4: Merge feature/api-endpoints-tests
git merge origin/feature/api-endpoints-tests --no-ff -m "feat: merge feature/api-endpoints-tests vào p2-ux-enhancement"

# Bước 5: Merge feature/p0-p1-core-ui
git merge origin/feature/p0-p1-core-ui --no-ff -m "feat: merge feature/p0-p1-core-ui vào p2-ux-enhancement"

# Bước 6: Kiểm tra trạng thái sau merge
git log --oneline --graph -10
git status
```

> ⚠️ **Nếu có conflict**: Resolve thủ công, ưu tiên giữ code mới nhất từ `feature/p0-p1-core-ui`, rồi chạy `git add . && git merge --continue`.

---

### Script Git — Sau khi hoàn thành toàn bộ P2 UI (Bước cuối cùng)

Sau khi tất cả P2 tasks đã implement xong và test OK, commit và push:

```bash
# Stage tất cả thay đổi P2
git add .

# Kiểm tra lần cuối trước khi commit
git status

# Commit theo Conventional Commits
git commit -m "feat: implement P2 UX Enhancement — shared UI components, error pages, profile page"

# Push nhánh lên remote (chỉ chạy sau khi hoàn tất)
git push -u origin feature/p2-ux-enhancement
```

> 💡 Sau khi push, lên GitHub tạo Pull Request từ `feature/p2-ux-enhancement` → `main` để review và merge.

---

### Thực hiện tương tự cho repo `Smart-kitchen-prompts`

```bash
# Chuyển sang repo prompts
cd ../Smart-kitchen-prompts

# Kiểm tra nhánh hiện tại
git branch -a && git status

# Cập nhật main
git checkout main && git pull origin main

# Tạo nhánh tương ứng để lưu prompts của P2
git checkout -b feature/p2-ux-enhancement-prompts

# Stage file prompt vừa tạo
git add prompts/p2-ux-enhancement-ui.md

# Commit
git commit -m "docs: thêm actionable prompt cho P2 UX Enhancement UI và git workflow"

# (Chưa push — chờ đến khi toàn bộ P2 hoàn tất)
```

---

## 📌 Checklist hoàn thành P2 Frontend

Đánh dấu `[x]` sau khi hoàn thành từng item:

### P2-1 · Shared UI Components
- [ ] `Frontend/components/ui/Button.tsx`
- [ ] `Frontend/components/ui/Badge.tsx`
- [ ] `Frontend/components/ui/Skeleton.tsx`
- [ ] `Frontend/components/ui/EmptyState.tsx`
- [ ] `Frontend/components/ui/Modal.tsx`
- [ ] `Frontend/components/ui/Toast.tsx`
- [ ] `Frontend/contexts/ToastContext.tsx`

### P2-2 · Error & Loading Pages
- [ ] `app/loading.tsx`
- [ ] `app/not-found.tsx`
- [ ] `app/error.tsx`

### P2-4 · Profile Page
- [ ] `Frontend/components/pages/ProfilePage.tsx`
- [ ] `app/dashboard/profile/page.tsx`

### Git Workflow
- [ ] Nhánh `feature/p2-ux-enhancement` đã được tạo từ merge 3 nhánh
- [ ] Nhánh `feature/p2-ux-enhancement-prompts` đã được tạo trong repo `Smart-kitchen-prompts`
- [ ] **(Chờ)** Push cả 2 nhánh sau khi P2 hoàn tất
