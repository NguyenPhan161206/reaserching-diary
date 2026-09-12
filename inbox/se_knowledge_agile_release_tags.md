# Software Engineering Fundamentals: Agile, Release, and Git Tags

**Created Date:** 2026-09-12
**Tags:** `#SoftwareEngineering`, `#Agile`, `#Git`, `#CI_CD`
**Source:** Q&A with Agent

---

## 1. Agility (Sự linh hoạt) trong Phát triển Phần mềm
**Khái niệm & Tư duy:**
- Không phải là quy trình cứng nhắc, mà là **mindset**: liên tục lặp lại (iterative), chia nhỏ công việc, giao sản phẩm (delivery) liên tục, thu thập phản hồi và thích ứng.
- **Bài học cốt lõi:** Đón nhận thay đổi, ưu tiên giao tiếp trực tiếp hơn quy trình, thà có sản phẩm chạy được sớm (fail fast, learn fast) còn hơn code một dự án hoàn hảo trong bí mật rồi thất bại.

**Lợi ích trong quản lý dự án & AI:**
- Khắc phục nhược điểm của Waterfall (chốt yêu cầu từ đầu nhưng thị trường thay đổi).
- Chia dự án thành các Sprints (1-4 tuần) giúp giảm thiểu rủi ro (risk mitigation).
- **Trong thời đại AI:** AI có tính bất định (uncertainty) cực cao (không chắc model có hội tụ hay không). Agile giúp thiết kế hệ thống dạng Modular (tách biệt Data, Training, Inference) để dễ thay thế, ngăn chặn ngụy biện chi phí chìm (sunk cost fallacy) khi cố đấm ăn xôi với một model tồi, và tương thích hoàn hảo với vòng lặp thu thập data (RLHF).

## 2. Release & Bản Release
**Đứng dưới góc độ Kỹ thuật (Software Engineering):**
- **Release (Động từ - Triển khai):** Hành động đưa code từ nhánh dev/staging đẩy thẳng lên môi trường Production để user sử dụng (Deploy). Thường được tự động hóa qua hệ thống CI/CD (GitHub Actions, Jenkins).
- **Bản Release (Danh từ - Artifact):** Một bản chụp (snapshot) toàn bộ source code của dự án tại một thời điểm đủ ổn định. Nó là gói mã nguồn đã được build sẵn (như file `.jar`, `.apk`, Docker image) kèm theo "Release Note" (nhật ký cập nhật: tính năng mới, fix bug).

## 3. Git Tags (Name Tags)
"Tag" không phải là một file vật lý, mà là metadata (dữ liệu siêu liên kết) của Git dùng để đánh dấu một mốc lịch sử quan trọng (ví dụ: `v1.2.0`). Tag là bất di bất dịch (read-only).

**Vị trí tìm thấy Tag:**
- **Trên Giao diện Web (GitHub/GitLab):** Ở mục "Releases" bên phải màn hình repository, hoặc trong dropdown chọn Branch (chuyển sang tab Tags).
- **Dưới Terminal (Local):** Dùng lệnh `git tag` để liệt kê, hoặc `git show v1.2.0` để xem chi tiết. Git lưu ngầm dữ liệu này trong thư mục ẩn `.git/refs/tags/`.
- **Trong IDE (VS Code):** Nằm ở panel "Source Control", mục "TAGS" ở dưới cùng.
