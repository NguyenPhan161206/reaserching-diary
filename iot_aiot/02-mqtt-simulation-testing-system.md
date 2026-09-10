# Technical Analysis: Simulation, Testing System & MQTT Protocol

*Created: 2026-09-09*

This document synthesizes technical knowledge, analyzes the current system status, and outlines areas requiring deep-dive investigation regarding network data emulation (MQTT), PC-based UI simulation, automated testing, and the Record & Replay system.

---

## 1. Technical Breakdown

### 1.1. MQTT Data Emulator
*   **Core Idea:** Software running on a PC acting as an *MQTT Publisher* (virtual device). It connects to an *MQTT Broker* (such as EMQX, Mosquitto) and continuously publishes payloads containing simulated sensor measurements and zone information to the corresponding Topics.
*   **Current Status:** Basic framework is complete. TCP/IP communication to the Broker is seamless. Payloads follow the correct schema.
*   **Technical Gap:** Lacks a state machine or scenario engine to automatically generate data based on spatial/temporal event sequences. For example:
    *   *Approaching object scenario:* Distance values decrease linearly or non-linearly over time.
    *   *Crossing scenario:* Sensor 1 triggers, followed by Sensor 2 triggering after a specific delay.

### 1.2. PC-based UI Simulator
*   **Core Idea:** Decouple UI/UX logic from embedded hardware (such as STM32/ESP32 + LCD). Utilize frameworks like LVGL (Light and Versatile Graphics Library) with a PC Simulator (SDL2) or Qt to compile and run the UI layer directly on a host OS (Windows/Linux) as a desktop GUI application.
*   **Current Status:** Not implemented yet (0%).
*   **Impact:** The build-flash-test cycle on physical microcontrollers is too slow (taking minutes per iteration). Having a PC simulator reduces this cycle to seconds, enabling hot-reloading or rapid testing of animations and layouts.

### 1.3. Automated Unit Testing
*   **Core Idea:** Apply CI/CD and TDD (Test-Driven Development) to firmware. Write test cases isolating business logic from the hardware layer (Hardware Abstraction Layer - HAL).
*   **Current Status:**
    *   *Distance filtering algorithm:* 100% test coverage, runs smoothly in a PC environment.
    *   *Sensor signal classification model (Display/Hardware-bound):* No tests yet. The classification logic is currently coupled with hardware/display code, preventing isolated testing.
*   **Solution:** Refactoring is required using Design Patterns (e.g., MVC or Observer) to decouple the signal processing layer (Business Logic) from the presentation layer (View/Hardware).

### 1.4. Record & Replay Tool
*   **Core Idea:**
    *   *Record:* A background MQTT Subscriber or Serial logger that listens to all raw sensor data and persists it into log files or a time-series database (such as InfluxDB, CSV, or rosbag in ROS).
    *   *Replay:* A tool that reads log files and reinjects data into the processing pipeline with accurate timestamps (supporting speed up / slow down) to reliably reproduce issues (deterministic debugging).
*   **Current Status:** Not yet available. Hard to troubleshoot corner cases or data-race conditions (which are rare in real-world testing).

### 1.5. MQTT Architecture
*   **Pub/Sub Model:** Decoupled sender and receiver via a **Broker**. Hardware devices (Clients) do not need to know which server processes the data; they only need to know the Topic.
*   **Advantages:** Extremely low bandwidth overhead (compact header of ~2 bytes), maintains connection well over unstable networks (mechanisms like Keep-Alive, LWT - Last Will and Testament).
*   **Routing & Wildcards:** Tree-structured topic hierarchy (`/`). Supports `+` (single-level) and `#` (multi-level) wildcards. Example: `sensor/+/temp` receives data from both living room and bedroom.
*   **QoS (Quality of Service):**
    *   `QoS 0` (At most once): Fire and forget.
    *   `QoS 1` (At least once): Guaranteed delivery, but duplicates may occur (requires ACK).
    *   `QoS 2` (Exactly once): Guaranteed delivery without duplicates (4-step handshake, highest overhead/slowest).

---

## 2. Backlog & Deep-dive Questions

Below is the list of topics and questions to research and resolve in the next phase:

### MQTT Data Simulation
*   **Q1:** Should the Scenario Schema for the emulator be designed using JSON or YAML to facilitate configuring input parameters (e.g., object velocity, trajectory)?
*   **Q2:** How can simulation timestamps be synchronized when multiple sensors/publishers trigger simultaneously in a crossing scenario?

### PC-based UI Simulator
*   **Q3:** Does the current UI framework (e.g., LVGL, TouchGFX, or custom) support porting to PC via SDL/OpenGL? How should the CMake/Makefile build system be configured for PC environments?

### Automated Testing
*   **Q4:** How should hardware modules (I2C, SPI, LCD) be mocked/faked so that the "Sensor signal classification model" function can be tested on PC without compilation errors?
*   **Q5:** Which CI/CD pipeline (e.g., GitHub Actions) is best suited to automatically run Unit Tests on every new firmware push?

### Record & Replay
*   **Q6:** What format should sensor data be recorded in (CSV, JSON, or Binary) to optimize storage size and I/O throughput at high sampling rates?
*   **Q7:** When replaying data into the system, should it be replayed through a Virtual Serial Port or injected directly via MQTT / an internal queue to ensure real-time accuracy?

### Advanced MQTT
*   **Q8:** How should TLS/SSL security be configured for MQTT connections between hardware devices and the Broker to prevent eavesdropping?
*   **Q9:** How can Retained Messages and Last Will and Testament (LWT) in MQTT be utilized to detect sudden device disconnections in the current system?
