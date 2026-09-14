# Projects Review — UMT-Truck-Blind-Sight · Snake_ANN · Joblink

**Created Date:** 2026-09-14
**Tags:** `#Projects`, `#Portfolio`, `#IoT`, `#ML`, `#Fullstack`, `#CareerPath`
**Status:** Published

---

## 🚀 Tổng quan 3 dự án

| Dự án | Loại | Mô tả ngắn | Stack | Kiến thức rút ra lớn nhất |
|---|---|---|---|---|
| [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) | IoT (cá nhân/ACLAB) | Hệ thống cảnh báo điểm mù xe tải: cảm biến siêu âm JSN-SR04T → ESP32-S3 → cloud CoreIoT (ThingsBoard) → hiển thị cảnh báo real-time trên LCD 7" LVGL | C/C++, ESP32-S3, JSN-SR04T, MQTT, CoreIoT Rule-Chain, ESP-IDF, LVGL v9, FreeRTOS, PlatformIO | Toàn trình IoT end-to-end: sensor → firmware (FreeRTOS task) → cloud processing → UI nhúng |
| [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) | AI/ML (cá nhân) | ANN học cách thay thế heuristic Manhattan (do người định nghĩa) trong A* tìm đường cho game Snake, biết né chướng ngại vật + thân rắn | Python, TensorFlow, Pygame, NumPy, Matplotlib | Pipeline dữ liệu → train → tích hợp → evaluation; so sánh được "đoán bằng công thức" vs "học từ dữ liệu" |
| [Joblink](https://github.com/qminh77/joblink) | Full-stack (team — SE005) | Mạng xã hội việc làm & tuyển dụng: bài viết, kết nối, messaging real-time, tuyển dụng, CV, admin panel | Next.js 16, React 19, TypeScript, Tailwind, Supabase (PostgreSQL + RLS), TanStack Query, Zod | Kiến trúc layered security + tổ chức code feature-based trong team lớn hơn |

---

## 🧠 Kiến thức rút ra từ dự án

### 1. UMT-Truck-Blind-Sight — bài học hệ thống nhúng + IoT

- **Phân tầng rõ ràng:** tách `sensor-node` (đọc/lọc dữ liệu, buzzer cục bộ) khỏi `waveshare-screen` (hiển thị LVGL) — mỗi board độc lập, giao tiếp qua cloud. Giống **separation of concerns** áp dụng ở cấp phần cứng.
- **Chọn framework theo driver, không theo sở thích:** board màn hình phải dùng ESP-IDF thuần vì driver LVGL 9.1/GT911 cần ESP-IDF ≥ 5.5, tổ hợp `arduino,espidf` của PlatformIO bị khóa ở 4.4.7. → Quyết định kỹ thuật phải xuất phát từ **ràng buộc thực tế**, ghi [ADR](docs/adr/) và log lý do.
- **Bảo mật mặc định:** token/core key đặt trong `config/keys.json` (gitignore), chỉ commit template — không bao giờ đẩy secret (đúng rule B4 của diary này).
- **Độ trễ thấp ⇒ không round-trip cloud:** buzzer cảnh báo chạy cục bộ theo ngưỡng (<50cm WARNING, <20cm DANGER) thay vì đi cloud rồi về — latency-sensitive task phải xử lý ở edge.

### 2. Snake_ANN — bài học ML + heuristic

- **Ý tưởng cốt lõi:** thay vì con người *viết* distance metric (Manhattan), để mạng *học* khoảng cách thực tế có tính obstacles từ dữ liệu BFS. Đây chính là tinh thần "learned heuristic" mà các hệ thống search/planning hiện đại dùng.
- **Pipeline rõ ràng là chìa khóa:** `data_generation → training → integration → evaluation → visualization` — tách rời từng giai đoạn giúp debug và tái sử dụng.
- **So sánh phải có benchmark:** có chế độ so sánh thời gian thực (phím 1/2) + benchmark win rate/số bước/thời gian → tuyên bố "ANN tốt hơn" phải đi kèm số liệu, không cảm tính.

### 3. Joblink — bài học full-stack + bảo mật

- **Layered defense:** Zod validate input → action wrapper (requireRole) → ActionError không rò rỉ SQL/stack trace → **Row Level Security (RLS)** ở DB. Security được trải nhiều lớp chứ không chỉ "kiểm tra ở UI".
- **RSC-first + feature-based:** mặc định Server Component, chỉ `use client` khi cần; code chia theo module tính năng (posts, jobs, network...) thay vì theo layer → dễ scale khi team lớn.
- **Zero ORM + RPC:** logic phức tạp chạy thẳng RPC function ở DB — giảm round-trip, tận dụng PostgreSQL.
- **Làm việc nhóm:** 164 commits, message rõ ràng, quy ước chung — version control + commit convention là chuẩn giao tiếp trong team (liên hệ trực tiếp note CI/CD vừa ghi).

---

## 🔗 Liên hệ với mục tiêu AI Engineer

- **Blind-Sight** ≈ AIoT sau này: mọi hệ thống AI thực tế đều chạy qua sensor → edge → cloud; nắm hardware bên dưới giúp thiết kế data pipeline đúng.
- **Snake_ANN** ≈ cơ sở của ML Engineering: pipeline dữ liệu, train, evaluate, deploy heuristic. Mở rộng tự nhiên sang MLOps (CI cho model, drift test — xem [SUGGESTED] trong note version-control).
- **Joblink** ≈ nền tảng kỹ sư backend/AI product: hiểu auth/RLS/API là điều kiện để đưa model ra production an toàn. Hướng Economics/Business: Joblink là social graph việc làm — nơi có thể gắn recommendation/matchmaking bằng ML sau này.

## 📝 Research Journey

- **Why:** 3 dự án nằm đúng 3 mảng tôi theo đuổi (IoT, ML, Full-stack) — review lại giúp thấy mình đã có được piece nào trong bức tranh AI Engineer.
- **Struggle:** Lúc làm chưa nhìn ra sự liên thông — tưởng là 3 dự án tách biệt (phần cứng, game AI, web).
- **Aha moment:** Cả ba đều cùng một nguyên lý timeout: **chia nhỏ + pipeline + feedback sớm** — đúng tinh thần Agile/DevOps vừa học. "Làm" và "học lý thuyết" cuối cùng gặp nhau.
- **Career link:** Portfolio này chính là bằng chứng thực chiến cho mục tiêu AI Engineer trong Economics/Business — nhất là Joblink (social/job graph) + Snake_ANN (learned heuristic), kết hợp với Blind-Sight (AIoT edge).

## 🔗 Related Topics

- [[../books_summaries/agile_project_management/knowledge/version-control-devops-ci-cd.md]] — Joblink 164 commits / pipeline quality gate chính là Agile + CI/CD hiện thực
- [[../ai_ml/04-xp-engineering-practices-for-ml.md]] — Snake_ANN nên phát triển thêm TDD/CI cho ML
- [[../iot_aiot/]] — Blind-Sight thuộc hệ AIoT edge
- [SUGGESTED] **Bridge note: "From snake heuristic to learned recommenders"** — nối Snake_ANN (learned heuristic A*) với recommendation/matchmaking trong Joblink — nên nằm trong `random_ideas/`

## ✅ Todo cải tiến

- [ ] Blind-Sight: thêm TLS cho MQTT + dùng token rotation (xóa security note trong SECURITY.md)
- [ ] Snake_ANN: viết CI pipeline test data validation + smoke train; log metrics chuẩn (MLflow/DVC)
- [ ] Joblink: thử gắn feature recommendation đơn giản (content-based) để thử học-lặp-label từ tương tác

---

*Làm dự án là cái giá phải trả để biến lý thuyết thành kỹ năng.*