# Session 07 — Agile Architecture, HAL, Refactoring, Scaling & Faux Agile

**Date:** 2026-09-19
**Source:** [[../knowledge/part-5-ensuring-success.md]] | [[../knowledge/part-6-the-part-of-tens.md]]

## ❓ Question 1: How does Agile interact with System Architecture (Evolutionary vs. Upfront Design)?
### 📚 Detailed answer
- **Explanation:** Instead of "Big Design Up Front" (BDUF), Agile relies on **Evolutionary Architecture**. The team builds a Minimum Viable Architecture in early sprints and lets the system design evolve incrementally based on real-world feedback and technical spikes.
- **Example:** In the Truck Blind Sight (TBS) project, rather than designing every database schema and sensor driver upfront, the team starts with an ESP-NOW sensor-node prototype, validates it via a spike, and refines the architecture iteratively.
- **Why it matters:** Prevents over-engineering and eliminates wasted effort when initial assumptions prove incorrect.

---

## ❓ Question 2: What is HAL (Hardware Abstraction Layer) and why is it crucial for IoT/AIoT?
### 📚 Detailed answer
- **Explanation:** A **Hardware Abstraction Layer (HAL)** is a software wrapper that separates physical hardware components from application business logic, exposing standardized APIs.
- **Example:** In TBS, instead of hardcoding `digitalRead(pin)` inside the warning decision logic, a HAL function like `get_front_distance()` is used. If the hardware changes from JSN-SR04T ultrasonic sensors to LiDAR, only the HAL needs updating, leaving application logic untouched.
- **Why it matters:** Enforces decoupling, reduces architectural debt, and protects code from breaking when hardware components change.

---

## ❓ Question 3: How do teams manage Architectural Debt and Refactoring in Agile?
### 📚 Detailed answer
- **Explanation:** Architectural debt arises from design shortcuts taken to meet early deadlines. In Agile, **Refactoring** (restructuring existing code without altering external behavior) is treated as a first-class citizen rather than a taboo.
- **Example:** Allocating capacity in a sprint to migrate a rigid 6-column database table into a flexible JSON schema to support incoming AI camera bounding boxes.
- **Why it matters:** Keeps the system extensible and prevents "refactoring hell" where technical debt slows feature delivery to a crawl.

---

## ❓ Question 4: What is "Faux Agile" and how does it manifest?
### 📚 Detailed answer
- **Explanation:** **Faux Agile** (Agile theater) occurs when organizations adopt agile terminology and ceremonies (like daily standups) while retaining rigid, top-down command-and-control management.
- **Example:** Daily standups turning into status reports for management, or managers imposing fixed scopes with inflexible deadlines.
- **Why it matters:** Recognizing faux agile helps engineers and tech leads advocate for true psychological safety, self-organization, and iterative value delivery.

---

## 💡 Session Summary
- **What was learned:** Mastered evolutionary architecture, HAL design patterns for IoT, architectural debt management, scaling frameworks (SAFe/LeSS), and spotting faux agile anti-patterns.
- **Agile Syllabus Status Report:** We have now covered core Agile mindset (Part 1), Scrum framework & roles (Part 2), planning, user stories & INVEST (Part 3), metrics & crisis management (Part 4), scaling agile (Part 5), and architectural/faux agile patterns (Part 6)!
