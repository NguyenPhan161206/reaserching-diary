# PSRAM vs Internal SRAM in ESP32 Architecture

**Created Date:** 2026-09-21
**Last Updated:** 2026-09-28
**Tags:** `#iot_aiot`, `#esp32`, `#hardware-architecture`, `#memory-management`
**References:** Espressif ESP-IDF Documentation
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)
- **PSRAM (Pseudo Static RAM):** external memory, reached over an SPI bus. Large capacity (2 MB – 8 MB) but slow to access.
- **Internal SRAM:** on-chip memory of the ESP32 microcontroller. Tiny (~520 KB) but extremely fast, with near-zero latency.
- **Roles:** the ESP32 is the brain (CPU), internal SRAM is a small ultra-fast workbench, PSRAM is a large but distant storage room. PSRAM cannot compute or compile code at all.
- **Hardware cache:** in the ESP32 architecture PSRAM acts as *main memory*, while a portion of internal SRAM is set aside as a **cache** that hides PSRAM's slowness.

## 🧠 Detailed Notes

### 1. Core Concepts: Why is PSRAM needed?
By default the ESP32 has only about 520 KB of internal SRAM. That is enough for traditional embedded tasks (reading a sensor, toggling a relay, publishing MQTT). But once an IoT project moves up a level of complexity, 520 KB is not enough:

- **Camera (ESP32-CAM):** it needs a huge frame buffer to hold JPEG frames before transmitting them over WiFi.
- **Edge AI/ML (TinyML):** loading the weights of face or speech recognition models.
- **Graphical UI (LVGL):** drawing an interface on a large LCD requires allocating a display buffer.

The solution is to solder an extra PSRAM chip (2 MB / 4 MB / 8 MB) next to the ESP32.

### 2. How It Works (Mechanism): architecture and mechanics
Many people assume PSRAM is a chip that can compile code or process data by itself. In reality:

- **No computation:** PSRAM is only a storage room. It has no CPU. 100% of all processing (math, compilation, running logic) is done by the ESP32's CPU.
- **How data circulates:** to process a large image, the ESP32's CPU traverses the SPI bus, fetches a small block from PSRAM into internal SRAM, processes it, and writes the result back to PSRAM.

### 3. Configuration in ESP-IDF (Implementation)
Even when the board already has PSRAM soldered on, ESP-IDF will **not use it by default**. The reason is that PSRAM is much slower than internal RAM. If ESP-IDF merged the two into one pool, real-time tasks (interrupts, ISRs) that happened to land in PSRAM would hang the system.

To use it you must enable it in the configuration:
1. Run `menuconfig`
2. Navigate: `Component config` ➔ `ESP32-specific` ➔ `Support for external, SPI-connected RAM`

**Example code — manual allocation into PSRAM:**
```c
#include "esp_heap_caps.h"

// Allocate a large buffer, forcing it into PSRAM instead of internal RAM
void* allocate_large_buffer(size_t size) {
    // MALLOC_CAP_SPIRAM restricts the allocation to the external PSRAM chip
    void* buffer = heap_caps_malloc(size, MALLOC_CAP_SPIRAM);

    if (buffer == NULL) {
        printf("Error: not enough PSRAM!\n");
    } else {
        printf("Successfully allocated %zu bytes on PSRAM.\n", size);
    }

    return buffer;
}
```

### 4. Practical Application: Software Cache vs Hardware Cache
This is a classic misconception:

- **By purpose (software cache):** you are perfectly free to use PSRAM as a "temporary store" (cache) for application data — for example, caching images downloaded from the internet so they do not have to be fetched again.
- **By architecture (hardware cache):** at the chip-design level, **PSRAM IS NOT A CACHE**. PSRAM is the main memory plugged in externally (like the DDR4 stick in a laptop). Because PSRAM is so slow, the ESP32 hardware automatically carves out part of its internal SRAM (which runs extremely fast) to act as a **cache** for PSRAM. When the CPU reads from PSRAM it pulls a chunk of data into the cache in the ESP32, so subsequent reads happen instantly.

### 5. Comparison Table
| Criterion | Internal SRAM | PSRAM (external RAM) |
|----------|---------------|-----------------------|
| **Location** | Integrated into the ESP32 die | Separate chip soldered on the board, connected over SPI |
| **Capacity** | Very small (~520 KB) | Very large (2 MB – 8 MB) |
| **Speed (latency)** | Absolutely fast, near-zero latency | Slower, dependent on SPI bus speed |
| **Best suited for** | Core logic, interrupts, global variables | Storing images, AI models, audio buffers, graphics |

## 📝 Research Journey
- **Why:** analysing memory-allocation requirements while building advanced features (AI, camera) on the ESP-IDF ecosystem.
- **Struggle:** I initially thought PSRAM was a chip that could process on its own (like a co-processor), and I confused the software-level cache mechanism with the hardware-level cache architecture.
- **Aha moment:** the ESP32 is like a craftsman (CPU), internal SRAM is the workbench right in front of them (fast but cramped), and PSRAM is the warehouse shelf further away (slow but spacious). And the ESP32 hardware has to use SRAM as the cache *for* PSRAM, not the other way round.
- **Career link:** understanding the memory hierarchy is what prevents "Out of Memory" and "Watchdog Reset" errors — the two fatal bugs a system engineer hits on IoT projects such as TBS.

## 🔗 Related Topics
- [[03-hardware-abstraction-layer-hal.md]] — PSRAM is a hardware-architecture concern, while HAL is a software-architecture concern: two independent factors that complement each other in system design
- [[vocabulary/07-labelling-and-temporary-memory.md]] — the same memory-hierarchy reasoning applied to software memory levels (cache → main → scratchpad)
- [SUGGESTED] esp32-memory-layout-and-linker-scripts — how the linker places `.bss` and `.data` variables into the different RAM regions
- [SUGGESTED] DMA to PSRAM on ESP32 — related to the open question below

## 🤔 Open Questions
- [ ] When PSRAM runs out, can Flash memory (also connected over SPI) be used as a swap space (virtual memory), the way Linux does it?
- [ ] How is DMA (Direct Memory Access) implemented on the ESP32 to move data from a sensor straight into PSRAM without passing through the CPU?

---
*A good system architecture means putting the right data in the right memory.*
