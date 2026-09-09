# Phân Tích Kỹ Thuật: Hệ Thống Giả Lập, Kiểm Thử & Giao Thức MQTT

*Ngày tạo: 2026-09-09*

Tài liệu này tổng hợp các kiến thức kỹ thuật, phân tích hiện trạng hệ thống và liệt kê các vấn đề cần đào sâu liên quan đến công cụ giả lập dữ liệu mạng (MQTT), mô phỏng giao diện PC, kiểm thử tự động và hệ thống Record & Replay.

---

## 1. Phân Tích Kỹ Thuật (Technical Breakdown)

### 1.1. Công Cụ Giả Lập Dữ Liệu Mạng (MQTT Data Emulator)
*   **Bản chất (Core Idea):** Một phần mềm chạy trên PC đóng vai trò là *MQTT Publisher* (thiết bị ảo). Nó kết nối đến *MQTT Broker* (như EMQX, Mosquitto) và liên tục đẩy (publish) các payload chứa thông số đo đạc cảm biến và thông tin khu vực giả lập lên các Topic tương ứng.
*   **Hiện trạng (Current Status):** Framework cơ bản đã hoàn thiện. Giao tiếp TCP/IP đến Broker thông suốt. Payload đúng schema.
*   **Điểm nghẽn kỹ thuật (Technical Gap):** Thiếu một state machine (máy trạng thái) hoặc kịch bản (scenario engine) để tự động sinh dữ liệu theo chuỗi sự kiện không gian/thời gian. Ví dụ:
    *   *Kịch bản vật thể tiến lại gần:* Giá trị khoảng cách giảm dần tuyến tính hoặc phi tuyến theo thời gian.
    *   *Kịch bản cắt ngang:* Cảm biến 1 trigger, sau đó cảm biến 2 trigger với một độ trễ nhất định.

### 1.2. Mô Phỏng Giao Diện Màn Hình trên PC (PC-based UI Simulator)
*   **Bản chất:** Tách rời logic UI/UX khỏi phần cứng nhúng (như STM32/ESP32 + LCD). Sử dụng các framework như LVGL (Light and Versatile Graphics Library) với PC Simulator (SDL2) hoặc Qt để biên dịch và chạy thẳng UI layer trên HĐH máy tính (Windows/Linux) dưới dạng một GUI application.
*   **Hiện trạng:** Chưa triển khai (0%).
*   **Tác động:** Chu kỳ build-flash-test trên vi điều khiển thực tế quá chậm (tốn phút/lần). Có simulator trên PC sẽ rút ngắn xuống tính bằng giây, cho phép hot-reload hoặc test nhanh các animation, layout.

### 1.3. Kiểm Thử Tự Động (Automated Unit Testing)
*   **Bản chất:** Áp dụng CI/CD và TDD (Test-Driven Development) cho firmware. Viết các test case cô lập logic nghiệp vụ khỏi phần cứng (Hardware Abstraction Layer - HAL).
*   **Hiện trạng:**
    *   *Thuật toán lọc khoảng cách:* Coverage 100%, chạy tốt trên môi trường PC.
    *   *Mô hình phân loại tín hiệu cảm biến (Hiển thị/Hardware-bound):* Chưa có test. Việc logic phân loại đang bị trói buộc với code phần cứng/hiển thị khiến nó không thể test độc lập.
*   **Giải pháp:** Cần tái cấu trúc (Refactor) áp dụng Design Pattern (ví dụ: MVC hoặc Observer) để tách biệt lớp xử lý tín hiệu (Business Logic) và lớp hiển thị (View/Hardware).

### 1.4. Công Cụ Ghi Hình và Phát Lại Dữ Liệu (Record & Replay)
*   **Bản chất:**
    *   *Record (Ghi):* Một MQTT Subscriber hoặc Serial logger chạy ngầm, lắng nghe toàn bộ raw data từ cảm biến và lưu thành dạng file log/time-series database (như InfluxDB, CSV, hoặc rosbag trong ROS).
    *   *Replay (Phát lại):* Công cụ đọc file log và inject (bơm) lại dữ liệu vào processing pipeline với đúng timestamp (hoặc có thể tua nhanh/chậm) để tái tạo chính xác lỗi (deterministic debugging).
*   **Hiện trạng:** Chưa có. Khó bắt bug những lỗi dạng corner-case hoặc data-race (hiếm gặp trên thực tế).

### 1.5. Kiến Trúc Giao Thức MQTT (MQTT Architecture)
*   **Mô hình Pub/Sub:** Decoupled (giải quyết sự phụ thuộc) giữa người gửi và người nhận thông qua **Broker**. Thiết bị phần cứng (Client) không cần biết server nào xử lý, chỉ cần biết Topic.
*   **Ưu điểm:** Băng thông cực thấp (header nhỏ gọn cỡ 2 bytes), giữ kết nối tốt qua mạng chập chờn (cơ chế Keep-Alive, LWT - Last Will and Testament).
*   **Routing & Wildcards:** Cấu trúc topic dạng cây (`/`). Hỗ trợ wildcard `+` (một cấp) và `#` (đa cấp). Ví dụ: `sensor/+/temp` nhận cả phòng khách và phòng ngủ.
*   **QoS (Quality of Service):**
    *   `QoS 0` (At most once): Gửi và quên (Fire and forget).
    *   `QoS 1` (At least once): Đảm bảo đến, nhưng có thể trùng lặp (cần ACK).
    *   `QoS 2` (Exactly once): Đảm bảo đến và không trùng lặp (handshake 4 bước, chậm nhất).

---

## 2. Các Vấn Đề Tồn Đọng & Câu Hỏi Cần Đào Sâu (Backlog & Deep-dive)

Dưới đây là danh sách các hạng mục cần nghiên cứu và giải quyết ở phase tiếp theo:

### Về Giả Lập Dữ Liệu MQTT
*   **Q1:** Thiết kế định dạng kịch bản (Scenario Schema) cho công cụ giả lập bằng JSON hay YAML để dễ dàng khai báo các tham số đầu vào (ví dụ: vận tốc vật thể, quỹ đạo)?
*   **Q2:** Làm sao để đồng bộ hóa thời gian giả lập khi có nhiều cảm biến/publisher cùng kích hoạt một lúc trong kịch bản cắt ngang?

### Về UI Simulator trên PC
*   **Q3:** Framework UI hiện tại đang dùng (ví dụ: LVGL, TouchGFX, hay custom?) có hỗ trợ porting sang PC qua SDL/OpenGL không? Cần setup CMake/Makefile như thế nào cho môi trường PC?

### Về Kiểm Thử Tự Động
*   **Q4:** Làm thế nào để Mocking/Faking các module phần cứng (I2C, SPI, LCD) để có thể test hàm "Mô hình phân loại tín hiệu" trên PC mà không bị lỗi biên dịch?
*   **Q5:** CI/CD pipeline (VD: GitHub Actions) nào phù hợp để chạy Unit Test tự động mỗi khi có Push code firmware mới?

### Về Ghi và Phát Lại Dữ Liệu (Record & Replay)
*   **Q6:** Dữ liệu cảm biến cần lưu dưới định dạng gì (CSV, JSON, hay Binary) để tối ưu dung lượng và tốc độ đọc/ghi khi tần số lấy mẫu (Sampling rate) cao?
*   **Q7:** Khi phát lại dữ liệu vào hệ thống, nên phát lại qua cổng vật lý ảo (Virtual Serial Port) hay bơm trực tiếp qua MQTT/Internal Queue để đảm bảo tính thời gian thực (Real-time)?

### Về MQTT Nâng Cao
*   **Q8:** Cách cấu hình bảo mật TLS/SSL cho kết nối MQTT giữa thiết bị phần cứng và Broker để chống nghe lén?
*   **Q9:** Cơ chế Retained Messages và Last Will and Testament (LWT) trong MQTT có thể áp dụng thế nào để phát hiện thiết bị mất kết nối đột ngột trong hệ thống hiện tại?
