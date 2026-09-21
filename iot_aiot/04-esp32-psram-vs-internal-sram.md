# PSRAM vs Internal SRAM in ESP32 Architecture

**Created Date:** 2026-09-21
**Last Updated:** 2026-09-21
**Tags:** `#iot_aiot`, `#esp32`, `#hardware-architecture`, `#memory-management`
**References:** Espressif ESP-IDF Documentation
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)
- **PSRAM (Pseudo Static RAM):** Là bộ nhớ ngoài, giao tiếp qua bus SPI. Dung lượng lớn (2MB - 8MB) nhưng truy xuất chậm.
- **Internal SRAM:** Là bộ nhớ trong của vi điều khiển ESP32. Dung lượng nhỏ (~520KB) nhưng tốc độ cực cao, độ trễ gần như bằng không.
- **Vai trò:** ESP32 là não (CPU), Internal SRAM là bàn làm việc nhỏ siêu nhanh, PSRAM là kho lưu trữ lớn nhưng xa. PSRAM hoàn toàn không có khả năng tính toán hay biên dịch code.
- **Hardware Cache:** Trong kiến trúc ESP32, PSRAM đóng vai trò là Main Memory, còn Internal SRAM được trích một phần ra để làm "Cache" che lấp đi sự chậm chạp của PSRAM.

## 🧠 Detailed Notes

### 1. Core Concepts: Tại sao lại cần PSRAM?
Mặc định, ESP32 chỉ có khoảng 520KB Internal SRAM. Dung lượng này đủ cho các tác vụ nhúng truyền thống (đọc cảm biến, bật tắt Relay, gửi MQTT). Tuy nhiên, khi các dự án IoT tiến lên một cấp độ phức tạp hơn, 520KB là không đủ:
- **Camera (ESP32-CAM):** Cần bộ đệm (Frame Buffer) khổng lồ để lưu trữ ảnh JPEG trước khi truyền qua WiFi.
- **AI/ML trên Edge (TinyML):** Load trọng số (weights) của các mô hình nhận diện khuôn mặt hay giọng nói.
- **UI/UX Đồ họa (LVGL):** Vẽ giao diện trên màn hình LCD lớn yêu cầu cấp phát bộ nhớ đệm hiển thị.

Để giải quyết, người ta gắn thêm con chip PSRAM (2MB/4MB/8MB) bên cạnh ESP32.

### 2. How It Works (Mechanism): Kiến trúc và Cơ chế
Nhiều người lầm tưởng PSRAM là một con chip có khả năng tự biên dịch code hoặc xử lý dữ liệu. Thực chất:
- **Không có tính toán:** PSRAM chỉ là cái kho chứa đồ. Nó không có CPU. 100% mọi quá trình xử lý (tính toán toán học, biên dịch, chạy logic) đều do CPU của ESP32 thực hiện.
- **Cách dữ liệu luân chuyển:** Khi cần xử lý một bức ảnh lớn, CPU của ESP32 sẽ chạy qua Bus SPI, bốc một mảng nhỏ từ PSRAM mang về Internal SRAM, xử lý xong lại đẩy kết quả ra PSRAM cất đi.

### 3. Cấu hình trong ESP-IDF (Implementation)
Dù trên mạch có hàn sẵn PSRAM, ESP-IDF mặc định sẽ **KHÔNG sử dụng nó**. Lý do là vì PSRAM chậm hơn RAM nội rất nhiều. Nếu ESP-IDF tự động gộp chung vào làm một, các tác vụ thời gian thực (Interrupt, ISR) lỡ rơi vào PSRAM sẽ làm treo hệ thống.

Để sử dụng, bạn phải bật trong cấu hình:
1. Mở `menuconfig`
2. Truy cập: `Component config` ➔ `ESP32-specific` ➔ `Support for external, SPI-connected RAM`

**Code ví dụ - Cấp phát bộ nhớ thủ công vào PSRAM:**
```c
#include "esp_heap_caps.h"

// Hàm cấp phát bộ nhớ lớn, ép buộc phải nằm ở PSRAM thay vì Internal RAM
void* allocate_large_buffer(size_t size) {
    // MALLOC_CAP_SPIRAM chỉ định bộ nhớ nằm trên con chip ngoại vi PSRAM
    void* buffer = heap_caps_malloc(size, MALLOC_CAP_SPIRAM);
    
    if (buffer == NULL) {
        printf("Lỗi: Không đủ bộ nhớ trên PSRAM!\n");
    } else {
        printf("Cấp phát thành công %zu bytes trên PSRAM.\n", size);
    }
    
    return buffer;
}
```

### 4. Practical Application: Software Cache vs Hardware Cache
Đây là một sự nhầm lẫn kinh điển:
- **Xét theo mục đích (Software Cache):** Bạn hoàn toàn có thể dùng PSRAM làm nơi "lưu tạm" (cache) dữ liệu của ứng dụng, ví dụ cache ảnh tải từ internet để khỏi tải lại.
- **Xét theo kiến trúc (Hardware Cache):** Về mặt thiết kế chip, **PSRAM KHÔNG PHẢI LÀ CACHE**. PSRAM là bộ nhớ chính (Main Memory) cắm ngoài (giống thanh RAM DDR4 trên laptop). Do tốc độ PSRAM quá chậm, phần cứng ESP32 sẽ tự động lấy một phần Internal SRAM (chạy cực nhanh) của nó ra để làm **Cache** (bộ nhớ đệm phần cứng) cho PSRAM. Khi CPU đọc PSRAM, nó sẽ kéo một cụm dữ liệu vào Cache trong ESP32 để các lần đọc tiếp theo diễn ra tức thời.

### 5. Comparison Table
| Tiêu chí | Internal SRAM | PSRAM (RAM ngoài) |
|----------|---------------|-------------------|
| **Vị trí** | Tích hợp trong nhân vi điều khiển ESP32 | Chip hàn rời trên board, nối qua SPI |
| **Dung lượng** | Rất nhỏ (~520 KB) | Rất lớn (2 MB - 8 MB) |
| **Tốc độ (Latency)** | Nhanh tuyệt đối, độ trễ gần bằng 0 | Chậm hơn, phụ thuộc vào tốc độ Bus SPI |
| **Phù hợp cho** | Chạy logic cốt lõi, ngắt (Interrupts), biến toàn cục | Lưu trữ ảnh, model AI, buffer âm thanh, đồ họa |

## 📝 Research Journey
- **Why:** Phân tích nhu cầu cấp phát bộ nhớ khi làm các tính năng nâng cao (như AI/Camera) trên hệ sinh thái ESP-IDF.
- **Struggle:** Ban đầu nhầm lẫn vai trò của PSRAM là một chip có thể tự xử lý (như Co-processor) và nhầm lẫn cơ chế Cache của phần mềm với kiến trúc Cache phần cứng.
- **Aha moment:** Nhận ra ESP32 giống như người thợ (CPU), Internal SRAM là cái bàn ngay trước mặt (nhanh nhưng chật), còn PSRAM là cái kệ kho ở xa (chậm nhưng rộng). Và phần cứng ESP32 phải tự dùng SRAM làm Cache cho PSRAM chứ không phải ngược lại.
- **Career link:** Nắm chắc kiến trúc bộ nhớ (Memory Hierarchy) là yếu tố quyết định để tránh lỗi "Out of Memory" và "Watchdog Reset" - những lỗi chí tử khi làm System Engineer cho các dự án IoT (như TBS).

## 🔗 Related Topics
- [[03-hardware-abstraction-layer-hal.md]] — PSRAM là cấp độ kiến trúc phần cứng, còn HAL là kiến trúc phần mềm (hai yếu tố độc lập nhưng bổ trợ nhau khi thiết kế hệ thống).
- [SUGGESTED] esp32-memory-layout-and-linker-scripts — Tìm hiểu cách Linker sắp xếp biến `.bss` và `.data` vào các phân vùng RAM khác nhau.

## 🤔 Open Questions
- [ ] Khi PSRAM cạn kiệt, liệu có thể dùng Flash Memory (cũng nối qua SPI) để làm nơi Swap Memory (bộ nhớ ảo) như Linux không?
- [ ] Việc DMA (Direct Memory Access) chuyển dữ liệu từ Cảm biến thẳng vào PSRAM mà không đi qua CPU được thực hiện như thế nào trên ESP32?

---
*Kiến trúc hệ thống tốt là biết đặt đúng dữ liệu vào đúng bộ nhớ.*
