# Vocabulary & Collocation — Agile, IoT & Project Management Terminology

**Created Date:** 2026-09-19
**Last Updated:** 2026-09-19
**Tags:** `#vocabulary`, `#it`, `#iot_aiot`
**Status:** Published

---

## 📌 Term List & Quick Reference

| Term / Phrase | IPA | Core Meaning | Domain |
|---------------|-----|--------------|--------|
| **Spike** | `/spaɪk/` | A time-boxed research task to investigate uncertainty | Agile / R&D |
| **Epic** | `/ˈepɪk/` | A large body of work spanning multiple sprints | Agile PM |
| **User Story** | `/ˈjuːzər ˈstɔːri/` | A requirement expressed from the user's perspective | Agile Development |
| **Vertical Slicing** | `/ˈvɜːrtɪkl ˈslaɪsɪŋ/` | Cutting features end-to-end across layers | Software Architecture |
| **Telemetry** | `/təˈlemətri/` | Automated measurement and transmission of data | IoT / Telecommunications |
| **Time-box** | `/ˈtaɪm bɑːks/` | A strict, pre-scheduled time limit allocated to an activity | Agile Project Management |
| **SAFe** | `/seɪf/` | Scaled Agile Framework for enterprise-level multi-team coordination | Enterprise Agile |
| **LeSS** | `/les/` | Large-Scale Scrum framework sharing a single Product Backlog | Scaling Agile |
| **HAL** | `/hɑːl/` | Hardware Abstraction Layer separating hardware from application logic | Embedded Systems / IoT |
| **Evolutionary Architecture** | `/ˌevəˈluːʃəneri ˈɑːrkɪtektʃər/` | Architecture that grows incrementally rather than being fully designed upfront | Software Architecture |
| **Architectural Debt** | `/ˌɑːrkɪˈtektʃərəl det/` | Compromises in design that make future changes more difficult | System Design |
| **Refactoring** | `/riˈfæktərɪŋ/` | Restructuring existing code without changing its external behavior | Software Engineering |
| **Faux Agile** | `/foʊ ˈædʒaɪl/` | Scrum in name only while retaining traditional top-down control | Agile Management |

---

## 🧠 Detailed Vocabulary Analysis

### 1. Spike
- **IPA:** `/spaɪk/`
- **Definition & Domain Context:** In Agile development, a Spike is a dedicated research task allocated with a strict time limit to investigate a technical unknown, evaluate feasibility, or prototype a solution before committing to backlog estimation.
- **Word Family:**
  - Noun: Spike, Spiker
  - Verb: Spike
  - Adjective: Spiked
- **Collocations & Native Pairings:**
  - `verb + term`: *run a spike, allocate time for a spike, time-box a spike, resolve uncertainty via a spike*
  - `adjective + term`: *technical spike, research spike, 2-day spike*
  - `term + noun`: *spike task, spike investigation*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "do research task" or "test blindly"
  - ✅ *Native standard:* "run a time-boxed technical spike to mitigate R&D risk"
- **Resource & Literature Mapping:**
  - **Technical Architecture / Problem:** Handling high uncertainty in R&D or greenfield IoT/AI projects.
  - **Academic Paper / Documentation:** Scrum Guide & *Cohn, M. (2004). User Stories Applied for Agile Software Development.*
- **Cross-Domain Bridge (Tech ↔ Economics):** In investment and economics, a spike is equivalent to a pilot study or feasibility study conducted to calculate Expected Value (EV) before large capital expenditure.

### 2. Epic
- **IPA:** `/ˈepɪk/`
- **Definition & Domain Context:** A large body of work that cannot be completed within a single sprint and must be decomposed into smaller, manageable User Stories.
- **Word Family:**
  - Noun: Epic, Epic decomposition
  - Adjective: Epical
- **Collocations & Native Pairings:**
  - `verb + term`: *break down an epic, decompose an epic, manage epics in the backlog*
  - `adjective + term`: *high-level epic, major epic*
  - `term + noun`: *epic backlog, epic roadmap*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "big task" or "huge function"
  - ✅ *Native standard:* "decompose a high-level epic into granular User Stories"
- **Cross-Domain Bridge (Tech ↔ Economics):** Comparable to a macro-project divided into sub-projects or operational milestones.

### 3. Vertical Slicing
- **IPA:** `/ˈvɜːrtɪkl ˈslaɪsɪŋ/`
- **Definition & Domain Context:** The practice of breaking down features so each increment delivers end-to-end value across all system layers (UI, business logic, database/hardware) rather than building layer-by-layer (horizontal slicing).
- **Word Family:**
  - Noun: Vertical slice, Vertical slicing
  - Verb: Vertically slice
  - Adjective: Vertically sliced
- **Collocations & Native Pairings:**
  - `verb + term`: *perform vertical slicing, slice features vertically, deliver vertical slices*
  - `adjective + term`: *end-to-end vertical slice*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "cut by layer" or "split horizontally"
  - ✅ *Native standard:* "slice requirements vertically to ensure independent, deployable increments"

### 4. Telemetry
- **IPA:** `/təˈlemətri/`
- **Definition & Domain Context:** The process of automatically collecting measurements and other data from remote embedded devices/sensors and transmitting them to receiving equipment for monitoring.
- **Word Family:**
  - Noun: Telemetry, Telemetrist
  - Verb: Telemeter
  - Adjective: Telemetric
- **Collocations & Native Pairings:**
  - `verb + term`: *transmit telemetry, push telemetry data, ingest telemetry streams, collect sensor telemetry*
  - `adjective + term`: *real-time telemetry, device telemetry, raw telemetry*
  - `term + noun`: *telemetry payload, telemetry pipeline, telemetry broker*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "send data of device" or "sensor information"
  - ✅ *Native standard:* "ingest real-time device telemetry via MQTT payloads"

### 5. SAFe (Scaled Agile Framework)
- **IPA:** `/seɪf/` (Scaled Agile Framework)
- **Definition & Domain Context:** A heavy, enterprise-level framework designed to scale Agile practices across large organizations, aligning multiple teams around Agile Release Trains (ART) and Program Increment (PI) planning.
- **Word Family:**
  - Noun: SAFe, Scaled Agile, SAFe adoption
  - Adjective: Scaled
- **Collocations & Native Pairings:**
  - `verb + term`: *adopt SAFe, run PI planning, manage Agile Release Trains, implement enterprise SAFe*
  - `adjective + term`: *enterprise SAFe framework, scaled agile transformation*
  - `term + noun`: *SAFe implementation, SAFe portfolio*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "big agile method for big company"
  - ✅ *Native standard:* "Adopt SAFe to coordinate multiple Agile Release Trains across large enterprise departments."
- **Resource & Literature Mapping:**
  - **Technical Architecture / Problem:** Scaling Agile synchronization across 100+ developers in traditional corporations.
  - **Academic Paper / Documentation:** Scaled Agile Framework Official Documentation (`scaledagileframework.com`).

### 6. LeSS (Large-Scale Scrum)
- **IPA:** `/les/` (Large-Scale Scrum)
- **Definition & Domain Context:** A scaling framework that extends Scrum rules to multiple teams working on a single product with one shared Product Backlog, minimizing added bureaucracy.
- **Word Family:**
  - Noun: LeSS, Large-Scale Scrum
- **Collocations & Native Pairings:**
  - `verb + term`: *scale Scrum with LeSS, maintain a single Product Backlog in LeSS, coordinate multiple LeSS teams*
  - `adjective + term`: *LeSS framework, LeSS Huge*
  - `term + noun`: *LeSS adoption, LeSS sprint planning*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "many scrum teams working together"
  - ✅ *Native standard:* "Apply LeSS to scale Scrum across multiple development teams sharing a unified Product Backlog."
- **Resource & Literature Mapping:**
  - **Technical Architecture / Problem:** De-scaling complex multi-team products without heavy management overhead.
  - **Academic Paper / Documentation:** *Larman, C., & Vodde, B. (2010). Practices for Scaling Lean and Agile Development: Large, Multisite, and Offshore Product Development with Large-Scale Scrum.*

### 7. HAL (Hardware Abstraction Layer)
- **IPA:** `/hɑːl/`
- **Definition & Domain Context:** A software layer that insulates application logic from physical hardware differences, allowing software to interact with devices through standardized APIs.
- **Word Family:**
  - Noun: HAL, Abstraction layer
  - Verb: Abstract, Wrap
  - Adjective: Abstracted, Hardware-agnostic
- **Collocations & Native Pairings:**
  - `verb + term`: *implement a HAL, decouple hardware via HAL, write hardware abstraction layers*
  - `adjective + term`: *robust HAL, hardware-independent HAL*
  - `term + noun`: *HAL driver, HAL architecture*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "connect code directly to hardware"
  - ✅ *Native standard:* "Isolate hardware dependencies by implementing a robust Hardware Abstraction Layer (HAL)."

### 8. Evolutionary Architecture
- **IPA:** `/ˌevəˈluːʃəneri ˈɑːrkɪtektʃər/`
- **Definition & Domain Context:** An architectural style that supports guided, incremental change across multiple dimensions as a primary objective, avoiding upfront over-engineering.
- **Word Family:**
  - Noun: Evolution, Architecture
  - Verb: Evolve
  - Adjective: Evolutionary, Extensible
- **Collocations & Native Pairings:**
  - `verb + term`: *design an evolutionary architecture, guide architectural evolution*
  - `adjective + term`: *incremental architecture, flexible system design*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "change design randomly"
  - ✅ *Native standard:* "Adopt an evolutionary architecture to accommodate shifting requirements incrementally across sprints."

### 9. Architectural Debt & Refactoring
- **IPA:** `/ˌɑːrkɪˈtektʃərəl det/`, `/riˈfæktərɪŋ/`
- **Definition & Domain Context:** Architectural debt refers to design shortcuts that incur maintenance friction later, while refactoring is the disciplined process of restructuring existing code without altering external behavior.
- **Word Family:**
  - Noun: Debt, Refactoring, Refactor
  - Verb: Refactor, Incur debt
  - Adjective: Refactored, Tech-debt ridden
- **Collocations & Native Pairings:**
  - `verb + term`: *incur architectural debt, pay off tech debt, refactor legacy code, perform continuous refactoring*
  - `adjective + term`: *code refactoring, structural refactoring*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "rewrite everything from scratch when broken"
  - ✅ *Native standard:* "Allocate sprint capacity to refactor modular components and pay off architectural debt."

### 10. Faux Agile
- **IPA:** `/foʊ ˈædʒaɪl/`
- **Definition & Domain Context:** An organizational anti-pattern where companies adopt agile ceremonies and terminology while maintaining rigid, top-down command-and-control management.
- **Word Family:**
  - Noun: Faux Agile, Agile theater
  - Adjective: Agile-washed
- **Collocations & Native Pairings:**
  - `verb + term`: *fall into faux agile, spot agile theater, transition away from faux agile*
  - `adjective + term`: *superficial agile, cargo cult agile*
- **Viet-Glish Pitfall vs. Band 7.0+ Alternative:**
  - ❌ *Viet-Glish mistake:* "fake agile company"
  - ✅ *Native standard:* "Recognize the anti-patterns of faux agile where daily standups become status reports for micromanagement."

---

## 📝 Research Journey
- **Why:** Mastering agile and IoT terminology in English ensures professional communication with mentors, instructors, and international developer communities.
- **Struggle:** Distinguishing between horizontal and vertical slicing, and understanding how to articulate R&D unknowns as "Spikes".
- **Aha moment:** Realizing that a "Spike" isn't wasted time, but a systematic risk-mitigation tool essential for AI and IoT engineering.
- **Career link:** Precise technical English is vital for leading teams as a Tech Lead/PM on projects like Truck Blind Sight (TBS).

## 🔗 Related Topics
- [[01-core-technical-collocations-and-viet-glish-fixes.md]] — core engineering collocations
- [[../books_summaries/05-agile_project_management/notes/05-agile-planning-invest-epic-spike-iot.md]] — agile project management session
