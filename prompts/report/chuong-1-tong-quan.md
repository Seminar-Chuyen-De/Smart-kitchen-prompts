# CHƯƠNG 1: TỔNG QUAN ĐỀ TÀI

## 1.1 Đặt Vấn Đề

Trong nhịp sống hiện đại ngày càng bận rộn, việc nấu ăn tại nhà đang trở thành thách thức với nhiều người. Câu hỏi "Hôm nay nấu gì?" không chỉ đơn giản là sở thích — nó đòi hỏi người nấu phải biết mình đang có những nguyên liệu gì trong tủ lạnh, các nguyên liệu đó có thể kết hợp thành món ăn nào, và cách thực hiện ra sao. Với những người mới học nấu ăn, đây là rào cản đáng kể; với những người bận rộn, đây là nguồn gốc của sự lãng phí thực phẩm và chi phí đặt đồ ăn ngoài tăng cao.

Theo thống kê, một hộ gia đình trung bình lãng phí khoảng 30% thực phẩm mua về do không biết cách sử dụng hết. Trong khi đó, sự bùng nổ của trí tuệ nhân tạo — đặc biệt là các Mô hình Ngôn ngữ Lớn (Large Language Models - LLM) và Computer Vision — mở ra cơ hội xây dựng các ứng dụng thông minh có khả năng "nhìn" và "hiểu" nguyên liệu, từ đó gợi ý công thức nấu ăn phù hợp.

Chính từ thực tiễn đó, đề tài **Smart Kitchen VN** ra đời với tham vọng xây dựng một trợ lý nấu ăn thông minh: người dùng chỉ cần chụp ảnh các nguyên liệu đang có, hệ thống AI sẽ tự động nhận diện và tạo ra công thức nấu ăn cá nhân hóa, chi tiết từng bước. Điểm đột phá của đề tài nằm ở việc áp dụng kiến trúc **Multi-System Agent (MSA)** — một hướng tiếp cận hiện đại trong AI kỹ thuật — để phối hợp nhiều AI agent chuyên biệt cùng xử lý một tác vụ phức tạp một cách hiệu quả và có thể mở rộng.

---

## 1.2 Mục Tiêu Đề Tài

### 1.2.1 Mục tiêu tổng quát

Xây dựng ứng dụng web Smart Kitchen VN — một nền tảng nấu ăn thông minh ứng dụng kiến trúc Multi-System Agent, cho phép người dùng nhận diện nguyên liệu từ ảnh và tự động tạo công thức nấu ăn phù hợp thông qua AI.

### 1.2.2 Mục tiêu cụ thể

**Về mặt kỹ thuật:**
- Thiết kế và triển khai kiến trúc Multi-System Agent gồm 4 agent chuyên biệt: Orchestrator, Vision, Recipe, và Storage Agent.
- Tích hợp Google Cloud Vision API để nhận diện nguyên liệu từ ảnh người dùng upload.
- Tích hợp Claude Sonnet 4.5 (Anthropic) để sinh công thức nấu ăn dựa trên danh sách nguyên liệu.
- Xây dựng hệ thống backend với Next.js 14 App Router, PostgreSQL và Prisma ORM theo nguyên tắc phân tách concerns (Separation of Concerns).
- Triển khai hệ thống xác thực người dùng an toàn qua Clerk (OAuth 2.0).

**Về mặt sản phẩm:**
- Cung cấp tính năng scan ảnh nguyên liệu và tự động generate công thức.
- Cho phép người dùng tạo và quản lý Cookbook cá nhân.
- Hỗ trợ tạo công thức thủ công và chỉnh sửa công thức do AI tạo.
- Cung cấp thông tin dinh dưỡng (calo, protein, carbs, chất béo) tự động.

**Về mặt nghiên cứu:**
- Khảo sát và đánh giá hiệu quả của mô hình MSA trong bài toán xử lý đa bước với AI.
- Đề xuất pattern thiết kế cho các ứng dụng web tích hợp AI phức tạp.

---

## 1.3 Phạm Vi Đề Tài

### 1.3.1 Phạm vi thực hiện

Đề tài tập trung vào việc xây dựng ứng dụng web (web application) chạy trên nền tảng trình duyệt, với backend RESTful API. Phiên bản mobile (React Native / Expo) nằm trong roadmap phát triển nhưng không thuộc phạm vi báo cáo này.

Về mặt AI, đề tài sử dụng các dịch vụ AI thương mại (Google Vision API, Claude Sonnet 4.5) thay vì tự huấn luyện mô hình — phù hợp với quy mô dự án và mục tiêu nghiên cứu kiến trúc hệ thống nhiều hơn là nghiên cứu deep learning thuần túy.

### 1.3.2 Đối tượng người dùng

Ứng dụng hướng đến:
- Người nấu ăn tại gia (nội trợ, sinh viên, người đi làm bận rộn).
- Người mới học nấu ăn cần hướng dẫn chi tiết từng bước.
- Người muốn tận dụng thực phẩm trong tủ lạnh, giảm lãng phí.

### 1.3.3 Giới hạn

- Ngôn ngữ giao diện và công thức: tiếng Việt.
- Nhận diện nguyên liệu phụ thuộc chất lượng ảnh và độ chính xác của Google Vision API.
- Công thức được tạo bởi AI cần người dùng kiểm tra trước khi áp dụng.

---

## 1.4 Đối Tượng Nghiên Cứu

Đề tài nghiên cứu các vấn đề kỹ thuật sau:

1. **Kiến trúc Multi-System Agent (MSA)**: Cách thiết kế và tổ chức nhiều AI agent hoạt động phối hợp, phân chia nhiệm vụ rõ ràng, giao tiếp qua message thay vì shared state.

2. **Computer Vision trong nhận diện thực phẩm**: Ứng dụng Google Cloud Vision API (Label Detection) để trích xuất thông tin nguyên liệu từ ảnh thực tế.

3. **Prompt Engineering cho LLM**: Thiết kế hệ thống prompt để Claude Sonnet 4.5 sinh ra công thức có cấu trúc nhất quán, đầy đủ thông tin dinh dưỡng và các bước thực hiện chi tiết.

4. **Kiến trúc phần mềm web hiện đại**: Nguyên tắc Separation of Concerns trong Next.js 14, Thin Router pattern, Zod validation, Prisma ORM type-safety.

5. **Tích hợp hệ thống xác thực**: Clerk OAuth, JWT, Webhook-based user synchronization giữa auth provider và database nội bộ.

---

## 1.5 Ý Nghĩa Khoa Học và Thực Tiễn

### 1.5.1 Ý nghĩa khoa học

Đề tài đóng góp một case study thực tế về việc áp dụng kiến trúc Multi-System Agent trong phát triển ứng dụng web thương mại. Khác với các nghiên cứu lý thuyết về MAS (Multi-Agent System) trong học thuật, đề tài này chứng minh tính khả thi của MSA trong môi trường production với các ràng buộc thực tế: chi phí API call, latency, error handling, và maintainability của codebase.

Ngoài ra, đề tài đề xuất mô hình **Strict 3-Folder Rule** — một pattern kiến trúc phần mềm cụ thể cho ứng dụng Next.js tích hợp AI — có thể được tham khảo và áp dụng trong các dự án tương tự.

### 1.5.2 Ý nghĩa thực tiễn

Về mặt sản phẩm, Smart Kitchen VN giải quyết một nhu cầu thực tế của hàng triệu hộ gia đình Việt Nam: giảm lãng phí thực phẩm, tiết kiệm thời gian lên kế hoạch bữa ăn, và nâng cao chất lượng dinh dưỡng thông qua các công thức được cá nhân hóa.

Về mặt giáo dục công nghệ, đề tài là minh chứng cụ thể cho việc tích hợp các công nghệ AI tiên tiến (LLM, Computer Vision) vào ứng dụng web thực tế, giúp sinh viên và lập trình viên có tham chiếu thực tế khi tiếp cận lĩnh vực AI Engineering.

---

## 1.6 Cấu Trúc Báo Cáo

Báo cáo được tổ chức thành 5 chương:

- **Chương 1 — Tổng quan đề tài** (chương này): Trình bày bối cảnh, mục tiêu, phạm vi và ý nghĩa của đề tài. Đây là nền tảng để người đọc hiểu lý do dự án được xây dựng và những vấn đề nó giải quyết.

- **Chương 2 — Cơ sở lý thuyết**: Tóm tắt các kiến thức nền tảng cần thiết, bao gồm: LLM, Computer Vision, Multi-Agent System, Next.js, PostgreSQL/Prisma, và Clerk Authentication.

- **Chương 3 — Phương pháp MSA (Multi-System Agent)**: Trọng tâm kỹ thuật của báo cáo. Mô tả chi tiết kiến trúc hệ thống, thiết kế từng agent, luồng xử lý AI Scan, thiết kế database và API.

- **Chương 4 — Triển khai và kết quả đạt được**: Trình bày môi trường phát triển, các tính năng đã implement thực tế, kết quả đạt được và khó khăn gặp phải.

- **Chương 5 — Kết luận và hướng phát triển**: Đánh giá tổng thể, hạn chế hiện tại và định hướng phát triển trong tương lai.
