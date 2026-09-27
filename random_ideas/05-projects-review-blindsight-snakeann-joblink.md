# Projects Review — UMT-Truck-Blind-Sight · Snake_ANN · Joblink

**Created Date:** 2026-09-14
**Last Updated:** 2026-09-28
**Tags:** `#Projects`, `#Portfolio`, `#IoT`, `#ML`, `#Fullstack`, `#CareerPath`
**Status:** Published

---

## 🚀 Overview of the 3 projects

| Project | Type | Short description | Stack | Biggest lesson |
|---|---|---|---|---|
| [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) | IoT (personal / ACLAB) | A truck blind-spot warning system: JSN-SR04T ultrasonic sensor → ESP32-S3 → CoreIoT cloud (ThingsBoard) → real-time warning on a 7" LVGL LCD | C/C++, ESP32-S3, JSN-SR04T, MQTT, CoreIoT Rule-Chain, ESP-IDF, LVGL v9, FreeRTOS, PlatformIO | The whole IoT chain end to end: sensor → firmware (FreeRTOS task) → cloud processing → embedded UI |
| [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) | AI/ML (personal) | An ANN that learns to replace the human-defined Manhattan heuristic in A* pathfinding for Snake, learning the real obstacle-aware distance from BFS data | Python, TensorFlow, Pygame, NumPy, Matplotlib | A data → train → integrate → evaluation pipeline; the ability to compare "guessing with a formula" against "learning from data" |
| [Joblink](https://github.com/qminh77/joblink) | Full-stack (team — SE005) | A job and recruitment social network: posts, connections, real-time messaging, recruitment, CV, admin panel | Next.js 16, React 19, TypeScript, Tailwind, Supabase (PostgreSQL + RLS), TanStack Query, Zod | Layered security + feature-based code organisation in a larger team |

---

## 🧠 Lessons extracted from the projects

### 1. UMT-Truck-Blind-Sight — embedded systems + IoT lessons

- **Clear layering:** `sensor-node` (reads/filters data, drives the local buzzer) is separated from `waveshare-screen` (renders the LVGL UI) — each board is independent and they talk through the cloud. This is **separation of concerns** applied at the hardware level.
- **Pick the framework by driver constraint, not by preference:** the display board had to use plain ESP-IDF because the LVGL 9.1 / GT911 drivers require ESP-IDF ≥ 5.5, while PlatformIO's `arduino,espidf` combination is locked at 4.4.7. → A technical decision has to start from a **real constraint**, be recorded as an [ADR](docs/adr/), and the reasoning logged.
- **Secure by default:** the token / core key live in `config/keys.json` (gitignored); only the template is committed — never push a secret (rule B4 of this diary).
- **Low latency ⇒ no cloud round-trip:** the warning buzzer runs locally against a threshold (<50 cm WARNING, <20 cm DANGER) instead of going up to the cloud and back — a latency-sensitive task must be handled at the edge.

### 2. Snake_ANN — ML + heuristic lessons

- **Core idea:** instead of a human *writing* the distance metric (Manhattan), let the network *learn* the real obstacle-aware distance from BFS data. This is exactly the "learned heuristic" spirit that modern search/planning systems use.
- **A clean pipeline is the key:** `data_generation → training → integration → evaluation → visualization` — separating each **stage** makes it debuggable and reusable.
- **Any comparison needs a benchmark:** there is a real-time A/B toggle (keys 1/2) plus a benchmark of win rate / number of steps / time → the claim "the ANN is better" must come with numbers, never with intuition.

### 3. Joblink — full-stack + security lessons

- **Layered defence:** Zod validates input → an action wrapper (requireRole) → ActionError that leaks neither SQL nor stack traces → **Row Level Security (RLS)** in the database. Security is spread across layers instead of being "checked in the UI".
- **RSC-first + feature-based:** Server Components by default, `use client` only when needed; code split by feature module (posts, jobs, network…) rather than by layer → easier to scale as the team grows.
- **Zero ORM + RPC:** complex logic runs as an RPC function directly in the database — fewer round-trips, **leveraging** PostgreSQL.
- **Unauthorized access:** reaching into user data outside your scope is *unauthorized access* — which is exactly what RLS blocks. Authorisation lives in the database, not in the UI.
- **Teamwork:** 164 commits with clear messages and shared conventions — version control plus commit convention is the team's communication standard (see the CI/CD note for the direct connection).

---

## 🔗 Connection to the AI Engineer goal

- **Blind-Sight** ≈ AIoT going forward: every real AI system runs sensor → edge → cloud; understanding the hardware underneath makes the data pipeline design correct.
- **Snake_ANN** ≈ the basis of ML engineering: data pipeline, training, evaluation, deploying a heuristic. It extends naturally into MLOps (CI for models, drift testing — see the [SUGGESTED] item in the version-control note).
- **Joblink** ≈ the backend foundation for an AI product: understanding auth, RLS and APIs is the precondition for getting a model into production safely. Economics/business direction: Joblink is a job social graph — somewhere ML-based recommendation or matchmaking can be attached later.

## 📝 Research Journey

- **Why:** the 3 projects sit exactly in the 3 areas I am pursuing (IoT, ML, full-stack) — reviewing them shows which pieces of the AI Engineer picture I already have.
- **Struggle:** while building them I did not see the connection — I thought they were 3 separate projects (hardware, game AI, web).
- **Aha moment:** all three obey the same timeout principle: **break it down + pipeline + early feedback** — precisely the Agile/DevOps mindset I had just been studying. Doing and theorising finally met.
- **Career link:** this portfolio is the practical evidence for the goal of an AI Engineer in Economics/Business — above all Joblink (social/job graph) plus Snake_ANN (learned heuristic), combined with Blind-Sight (AIoT edge).

## 🔗 Related Topics

- [[../books_summaries/05-agile_project_management/knowledge/version-control-devops-ci-cd.md]] — Joblink's 164 commits and its pipeline quality gate are Agile + CI/CD in practice
- [[../ai_ml/04-xp-engineering-practices-for-ml.md]] — Snake_ANN should grow TDD/CI for ML
- [[../vocabulary/12-one-vietnamese-word-many-english-words.md]] — why this note uses *leverage* and not "tận dụng", and why "unauthorised access" is not "interfering"
- [[../iot_aiot/]] — Blind-Sight belongs to the AIoT edge
- [SUGGESTED] **Bridge note: "From a snake heuristic to learned recommenders"** — connecting Snake_ANN (learned heuristic A*) with recommendation/matchmaking in Joblink — this belongs in `random_ideas/`

## ✅ Improvement Todo

- [ ] Blind-Sight: add TLS for MQTT + token rotation (removes the security note in SECURITY.md)
- [ ] Snake_ANN: write a CI pipeline for data validation + a smoke train; log standard metrics (MLflow/DVC)
- [ ] Joblink: try a simple content-based feature recommendation to experience the learn → label → retrain loop

---

*Building projects is the price of turning theory into skill.*
