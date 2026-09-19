# Session 05 — Agile Planning, INVEST, Epic, Spike & IoT Project Management

**Date:** 2026-09-19
**Source:** [[../knowledge/part-3-planning-and-execution.md]] | [[../knowledge/part-4-agility-management.md]]

## ❓ Question 1: What is the difference between horizontal and vertical slicing, and why is independence important in INVEST?
### 📚 Detailed answer
- **Explanation:** In Agile, **Vertical Slicing** means breaking down features so each User Story delivers end-to-end value across all system layers (UI, business logic, DB/hardware). **Horizontal Slicing** splits tasks by technical layers (e.g., all DB design first, then all backend, then frontend), which creates heavy bottlenecks. The **'I' in INVEST** stands for *Independent*, meaning stories should be developed and delivered without blocking each other.
- **Example:** For a User Profile feature:
  - *Horizontal (Bad):* Story 1 = Database schema, Story 2 = Backend API, Story 3 = Frontend UI (teams wait for each other).
  - *Vertical (Good):* Story 1 = View profile (mock data), Story 2 = Update display name (real DB), Story 3 = Upload avatar (independent increments).
- **Why it matters:** Vertical slicing enables parallel development and continuous integration, avoiding the "integration hell" at the end of a sprint.

---

## ❓ Question 2: How does the Fibonacci scale work in Agile estimation, and does it rely on intuition and experience?
### 📚 Detailed answer
- **Explanation:** Yes! Fibonacci numbers (1, 2, 3, 5, 8, 13, 21) are used for relative sizing (Story Points) rather than hours. They rely on a combination of **technical analysis, team experience, and gut feeling/intuition**. Humans are notoriously bad at estimating exact hours, but much better at relative complexity.
- **Example / Scale:** 
  - 1-2 points: Simple bug fix or minor UI tweak.
  - 5 points: Standard feature with known architecture.
  - 8-13 points: Complex feature with integration risks or unfamiliar tech stack.
  - 21+ points: Too large; must be treated as an Epic and broken down.
- **Why it matters:** It acknowledges the uncertainty in software development and prevents anchoring bias during Planning Poker.

---

## ❓ Question 3: How should a team handle a User Story or task involving unfamiliar technology (e.g., in AI/IoT)?
### 📚 Detailed answer
- **Explanation:** When a team encounters an unknown problem with high uncertainty (common in AI/IoT research), they use a **Spike**. A Spike is a time-boxed research task dedicated to prototyping, investigating, or answering a specific technical question before committing to backlog estimation.
- **Example in AI/IoT:** *"Spike (2 days): Test if LVGL v9 rendering on ESP32-S3 Waveshare screen causes memory leaks with FreeRTOS multitasking."*
- **Why it matters:** It prevents teams from getting paralyzed by uncertainty and ensures research time is strictly bounded and productive.

---

## ❓ Question 4: How to lead an IoT/AIoT greenfield project (like Truck Blind Sight - TBS) from scratch as a Tech Lead / PM?
### 📚 Detailed answer
- **Explanation:** When starting a project with initial checklists and high uncertainty (IoT moving to AI), the Tech Lead/PM must structure the work into a macro roadmap and lightweight agile processes.
- **Framework for TBS:**
  1. **Phase 1 (Edge & Sensor):** ESP-NOW communication, JSN-SR04T ultrasonic sensors (< 50ms latency).
  2. **Phase 2 (Cloud Telemetry):** MQTT broker, CoreIoT / ThingsBoard rule-chain integration.
  3. **Phase 3 (AI Expansion):** Computer Vision / AI model integration for blind-spot detection.
  4. **Phase 4 (Integration & Field Testing):** Physical truck mounting and stress testing.
- **Management Tools:** Use Spikes for R&D unknowns, GitHub Actions for CI/CD automation, and MQTT CLI tools (`mosquitto_pub/sub` or REST APIs) for telemetry testing.
- **Why it matters:** Gives stakeholders (instructors/mentors) clear visibility through milestones and demos rather than vague progress reports.

---

## 💡 Session Summary
- **What was learned:** Mastered vertical slicing (INVEST), Fibonacci relative sizing, Spikes for handling R&D uncertainty, and end-to-end Agile leadership for IoT/AIoT projects like TBS.
- **Next steps:** Apply these frameworks to structure TBS Sprint backlogs and prepare the Phase 1 demonstration for the instructor.
