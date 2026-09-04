# Kiến trúc Backend: Rate Limit và Server-Sent Events (SSE)

**Ngày tạo:** 2026-09-04
**Tags:** `#SystemDesign`, `#Backend`, `#Realtime`, `#API`
**Tài liệu tham khảo/Nguồn:** [Gemini Share](https://share.gemini.google/zEAVGArmeM7e) & [https://share.gemini.google/C3HUVEIvYfGX]

---

## 📌 Tóm tắt ý chính (TL;DR)
- **Rate Limit Constraints:** Cơ chế kiểm soát lưu lượng request đến server nhằm bảo vệ tài nguyên, chống DDoS, đảm bảo công bằng và hỗ trợ phân cấp dịch vụ (monetization).
- **Server-Sent Events (SSE):** Giao thức kết nối một chiều (Server -> Client) trên nền HTTP tiêu chuẩn, tối ưu cho các tính năng real-time chỉ cần luồng dữ liệu đọc (ví dụ: Chatbot AI streaming, log system).

## 🧠 Nội dung chi tiết

### 1. Rate Limit Constraints
Rate limit là các ràng buộc định tuyến trên tầng Server, API Gateway hoặc Firewall nhằm giới hạn tần suất gửi request từ một định danh (IP, User ID, API Key) trong một khung thời gian (Time window).
- **Cơ chế phản hồi:** Vượt quá ngưỡng -> `429 Too Many Requests`.
- **Mục đích cốt lõi:**
  1. **Prevent Overload (Bảo vệ tài nguyên):** Ngăn cản crash server do peak traffic.
  2. **Security (Chống tấn công):** Block các luồng request rác từ botnet/hacker muốn DDoS hệ thống.
  3. **Fair Use (Chia sẻ công bằng):** Tránh hiện tượng Noisy Neighbor (một user chiếm dụng toàn bộ pool tài nguyên, làm chậm hệ thống của người khác).
  4. **Monetization (Mô hình kinh doanh):** Phân tier dịch vụ API của bên thứ ba (Ví dụ: Free cho 1.000 reqs/tháng, Pro cho 10.000 reqs/tháng).

### 2. Server-Sent Events (SSE)
SSE là kỹ thuật đẩy dữ liệu (Data Push) từ Server về Client qua một kết nối HTTP duy nhất được giữ mở (Persistent Connection). Nó sinh ra để khắc phục sự lãng phí băng thông của kỹ thuật Polling truyền thống (Client liên tục ping hỏi server xem có data mới không).

- **Đặc trưng kỹ thuật:**
  - **Unidirectional (Một chiều):** Data chỉ chảy từ Server -> Client.
  - **HTTP Native:** Chạy trên giao thức HTTP thông thường, không cần cơ chế handshake phức tạp, bypass qua firewall/proxy dễ dàng.
  - **Auto-reconnect:** Trình duyệt tự động cố gắng kết nối lại với server nếu đường truyền bị ngắt.
- **Use-cases (Ứng dụng) tiêu biểu:**
  - Streaming output của các LLM Chatbot (như ChatGPT, Gemini) để hiển thị từng từ (typing effect) thay vì đợi load toàn bộ.
  - Ticker giá chứng khoán/Crypto trực tiếp.
  - Notification feeds (Thông báo hệ thống).
  - Live system logs monitor.

#### Thiết kế cơ sở (Implementation cơ bản)
Nguyên tắc: Client mở request `GET`, Server phản hồi với các header đặc thù và giữ nguyên trạng thái kết nối.

**Backend (Node.js/Express):**
```javascript
const express = require('express');
const app = express();

app.get('/stream', (req, res) => {
    // 1. Headers bắt buộc để định nghĩa stream SSE
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    // 2. Bơm dữ liệu định kỳ cho client
    const intervalId = setInterval(() => {
        const time = new Date().toLocaleTimeString();
        // Cú pháp chuẩn SSE bắt buộc: data: [payload]\n\n
        res.write(`data: Thời gian hiện tại là ${time}\n\n`); 
    }, 2000);

    // 3. Dọn dẹp tài nguyên (memory leak) khi client ngắt kết nối (tắt tab)
    req.on('close', () => clearInterval(intervalId));
});

app.listen(3000, () => console.log('SSE Server running on port 3000'));
```

**Frontend (HTML/JS - EventSource API):**
```html
<script>
    // Mở kết nối
    const eventSource = new EventSource('http://localhost:3000/stream');
    
    // Lắng nghe sự kiện đẩy về
    eventSource.onmessage = (event) => {
        console.log("Server pushed:", event.data);
    };
</script>
```

## 🤔 Kiến thức cần đào sâu
- [ ] **Các thuật toán Rate Limiting:** Token Bucket, Leaky Bucket, Fixed Window Counter, Sliding Window Log hoạt động ra sao và ưu nhược điểm?
- [ ] **Thực hành Cấu hình:** Cách thiết lập Rate Limit trên Nginx, Kong Gateway hoặc dùng Redis để store counter phân tán.
- [ ] **So sánh Giao thức:** Khi nào SSE là đủ, và khi nào bắt buộc phải chuyển sang WebSockets (Bi-directional) hoặc gRPC streaming?
- [ ] **Hạ tầng mạng:** Cấu hình Load Balancer (HAProxy, AWS ALB) như thế nào để không bị timeout (ngắt sớm) các kết nối SSE đang mở dài hạn?

## 📚 Tài liệu nên tham khảo
- Nguồn từ Gemini: [https://share.gemini.google/zEAVGArmeM7e](https://share.gemini.google/zEAVGArmeM7e)
- Nguồn tham khảo thêm: [https://share.gemini.google/C3HUVEIvYfGX](https://share.gemini.google/C3HUVEIvYfGX)
