# Phan Hữu Bình Nguyên — Master Profile (CV / Portfolio Source)

**Created:** 2026-09-18
**Tags:** `#CV`, `#Portfolio`, `#CareerPath`, `#AIEngineer`
**Status:** Active — single source of truth
**Language policy:** English (repo rule A6). Vietnamese proper nouns / organization names are kept verbatim.

> **Purpose:** This is the consolidated master record of my education, experience, projects,
> and skills. It merges (1) what already exists in this diary repo, (2) my latest verified
> personal information, and (3) details pulled from my GitHub repositories
> `UMT-TBS-official` and `Snake_ANN`. Use it as the raw material for role-specific CVs
> (AI, Software/Web/Data, Finance-Banking, Operations) and for portfolio pages.

---

## 1. Contact & Links

| Channel | Detail |
|---|---|
| **Full name** | Phan Hữu Bình Nguyên |
| **Email (primary)** | phanhuubinhnguyen30@gmail.com |
| **Email (alternate)** | phbn303030@gmail.com *(used in earlier CV drafts)* |
| **Phone** | 0386 186 434 *(from existing CV records — confirm before publishing)* |
| **Location** | Ho Chi Minh City, Vietnam |
| **GitHub** | [github.com/NguyenPhan161206](https://github.com/NguyenPhan161206) |
| **LinkedIn** | [linkedin.com/in/nguyen-phan-85aa92436](https://www.linkedin.com/in/nguyen-phan-85aa92436/?locale=vi) |
| **Repository (this diary)** | Researching Diary — personal knowledge base |

---

## 2. Professional Summary / Objective

Third-year **Information Technology** student specializing in **Artificial Intelligence (AI)**
at **UMT (University of Management and Technology, Ho Chi Minh City)** with a **GPA of 3.39**.
I build end-to-end AI and data systems — from embedded IoT sensor pipelines and machine-learning
training loops to full-stack products — and I am working toward becoming an **AI Engineer
applying AI to Economics and Business**.

I combine a **technical mindset (code + data)** with the ability to **apply AI deeply to
business operations** (finance & banking, agriculture, retail). My strengths are practical
machine learning, clean system design, leadership of student organizations, and a bridge
mentality between technology and business domains.

**Target roles:** AI / Machine Learning Intern · Data / Backend Engineer Intern · AI Engineer
(Economics & Business).

---

## 3. Education

### University of Management and Technology, HCMC (UMT) — 3rd year student
- **Major:** Information Technology (Công nghệ Thông tin)
- **Specialization:** Artificial Intelligence (AI)
- **GPA:** 3.39 / 4.0 (Very Good / Giỏi)
- **Expected graduation:** 2028

### Asian Institute of Technology (AIT), Thailand — International Study Program
- **Period:** August 2026
- **Description:** International learning-experience program. Practiced adapting to a
  multicultural environment and communicating/presenting entirely in English.

---

## 4. Experience — Leadership & Extracurricular Activities

### President (Chủ nhiệm) — E-NG Club, UMT — *2025 – present*
- Planned club operations and built the club charter from the ground up.
- Directed content strategy and managed communications.
- Organized large-scale events such as **Clubs Day 2026** and **Orientation Day**:
  event planning, budget estimation, logistics, and equipment coordination.

### Vice President (Phó Chủ nhiệm) — YEC (Young Economists Club), UMT — *2024 – present*
- Co-led operations and organized student-facing activities and academic workshops.
- Acted as a bridge between technical analysis thinking and economic management models.

### Active Member — STAC@UMT (School of Technology Academic Club)
- Participates in the technology-focused academic community at UMT.

### Samsung Innovation Campus (SIC) — *2026*
- Intensive technology training program from Samsung, including competency assessments.

### Google Gemini Student Ambassador Challenge — *January 2026*
- Developed a **systems-thinking research framework** and designed the infographic for the
  **Gemini Explorer** challenge.

### Community Tech Event — AI Riser Vietnam — *2026*
- Attended the AI Riser Vietnam 2026 community technology event.

### Workshop Organization & Coordination — *2025 – 2026*
- Coordinated logistics, prepared content, and managed operations for student skill-development
  workshops, in both online and offline formats.

---

## 5. Projects

### 5.1 UMT-TBS-official — Truck Blind-spot Warning System (IoT / AIoT)
**Repo:** [github.com/NguyenPhan161206/UMT-TBS-official](https://github.com/NguyenPhan161206/UMT-TBS-official)
**Also shipped as:** UMT-Truck-Blind-Sight · **Period:** 2026 · **Scope:** Team project (ACLAB) / official repo

- **What it is:** A vehicle blind-spot collision warning system (V2, greenfield) for trucks,
  inheriting and fixing architectural flaws of the V1 `supersonic-warning-system` repo
  (copy-pasted contracts, scattered thresholds, hard-coded secrets, no CI).
- **Hardware:** 2× **ESP32-S3** boards, **6× JSN-SR04T waterproof ultrasonic sensors**
  mounted around the vehicle, a **Waveshare 7" RGB touch LCD (GT911)**, and a buzzer.
- **Hybrid architecture:**
  - **ESP-NOW (primary / critical path):** sensor-node → screen, target latency **< 50 ms**,
    no access point required.
  - **CoreIoT / MQTT (secondary):** remote monitoring, ThingsBoard **Rule-Chain**, and
    dashboard; latency 100–500 ms on `v1/devices/me/telemetry`.
- **Shared contract:** a single source of truth in `firmware/shared/` —
  `espnow_protocol.h` (30-byte packed payload, channel/MAC/slot with `static_assert`) and
  `thresholds.h` (CAUTION ≤ 100 cm, DANGER ≤ 30 cm; buzzer WARNING < 50 cm for 3 s,
  DANGER < 20 cm for 1 s; 6 sensor pin mappings).
- **Firmware:** `sensor-node` on Arduino framework (measure/filter, local buzzer on GPIO11);
  `waveshare-screen` on **pure ESP-IDF** because LVGL v9 + GT911 drivers require ESP-IDF ≥ 5.5.
  Real-time tasks implemented with **FreeRTOS**.
- **Cloud:** snapshot CoreIoT Rule-Chain (7 nodes: TS → Attr → TypeSwitch → JS Transform →
  ChangeOriginator → Shared Attr / Save TS) producing a `warning_status` value.
- **Security engineering:** all secrets live only in `config/keys.json` (git-ignored, generated
  via `tools/guard/gen_credentials.py`); `scan_secrets.py` guard; **Gitleaks** in CI.
- **CI/CD:** GitHub Actions — build 2 environments × 2 firmwares, host tests, pytest,
  secret scanning, size gate; documented CD release flow.
- **Testing:** 3-level protocol (host / flash-and-observe / cloud), 16 pytest guard tests,
  10/10 firmware host tests, plus a Python MQTT acceptance tool (`test_mqtt_coreiot.py`).
- **Docs:** LaTeX technical report (4 chapters) and a separate code-detail report (8 chapters
  across 4 IoT layers), architecture diagrams, ADRs, and per-task implementation logs.
- **Key lesson:** separation of concerns at the hardware level (edge handles latency-sensitive
  alerts locally; cloud handles monitoring), and choosing a framework from real driver
  constraints rather than preference.

### 5.2 Snake_ANN — Learned Heuristic replacing Manhattan in A* (AI / ML)
**Repo:** [github.com/NguyenPhan161206/Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN)
**Period:** 2026 · **Scope:** Personal AI research project

- **What it is:** An **Artificial Neural Network** that learns to replace the hand-crafted
  **Manhattan distance heuristic** inside **A\*** pathfinding for the game Snake, so the
  learned distance accounts for **obstacles and the snake's own body**.
- **Input representation:** a **7×7 observation window** plus relative `(dx, dy)` to food =
  **51 input features**.
- **Data pipeline (BFS-grounded):** `data_generation` (BFS reachable/food placement) →
  `training` → `integration` → `evaluation` → `visualization`, orchestrated by `main.py`.
- **Datasets / grids:** 10×10 (10,000 samples), 20×20 (20,000), 50×50 (50,000).
- **Model & training:** TensorFlow/Keras ANN, **Adam** optimizer (`lr = 0.001`), batch size 32,
  up to 100 epochs with **EarlyStopping** (patience 10), 80/20 validation split.
- **Game environment:** random obstacles at 15% density per reset, fixed snake start `(1,1)`,
  food only spawns on BFS-reachable cells, snake grows over multiple feeds.
- **Evaluation:** real-time toggle between A\*+Manhattan and A\*+ANN (keys 1/2); benchmark of
  **win rate, average steps, and runtime** (50 games per run); loss-vs-epoch plots to inspect
  overfitting; per-grid Markdown reports and CSV/`.npy` logs.
- **Stack:** Python, TensorFlow, Pygame, NumPy, Matplotlib, tqdm.
- **Key lesson:** a clear stage-by-stage pipeline makes ML debuggable and reusable, and any
  claim that "the model is better" must be backed by a quantitative benchmark.

### 5.3 Joblink — Student–Employer Job Platform (Full-stack, team)
**Repo:** [github.com/qminh77/joblink](https://github.com/qminh77/joblink) · **Period:** 2026 · **Course:** SE005 (team)

- Full-stack job-seeking social network: posts, connections, real-time messaging, job
  recruitment, CVs, notifications, and a full admin panel.
- **Stack:** Next.js 16, React 19, TypeScript, Tailwind, Supabase (PostgreSQL + Realtime +
  **Row Level Security**), TanStack Query, Zod.
- **Layered security:** Zod input validation → server-action role guards → sanitized
  `ActionError` (no SQL/stack leakage) → **RLS** at the database.
- RSC-first, feature-based code organization; zero-ORM with PostgreSQL RPC; 164 commits with a
  clear commit convention (team communication standard).

### 5.4 Fraud Detection — Credit Card Fraud (Machine Learning)
**Period:** 2025 – 2026 · **Repo:** `intro_to_AI_project_fraud_detection`

- Processed **284,807 transactions** (492 fraud cases), handled severe class imbalance, and
  compared the performance of **27 ML models** (XGBoost, LightGBM, Logistic Regression, …).
- Designed a normalization pipeline and selected evaluation metrics appropriate for imbalanced
  data.

### 5.5 Agentic RAG — Financial & Banking Legal Advisory (AI)
**Period:** 2026

- Researched and built an advisory system using an **Agentic RAG** architecture — retrieval,
  verification, and orchestration.

### 5.6 Additional Projects

| Project | Period | Description | Stack / Repo |
|---|---|---|---|
| Project Management Software | Mar 2026 | Relational DB design, **MySQL Workbench** SQL scripts, **ER diagram**, and **REST API** endpoints tested with **Postman** | MySQL, REST, Postman · `Self_Task_Management` |
| Zero Trust & Network Security Talk | Apr 2026 | Technical presentation on **Zero Trust Network Access** and **Microsoft Entra Private Access** to secure database deployments | Security / Networking |
| Tứ Vị Brand Project | Nov – Dec 2025 | Brand strategy, content direction, and video scripts for a Gia Lai agricultural brand (tea, macadamia, honey) | Brand / Content |
| AI Self-Project Suite | 2026 | Mask detection, face recognition, hand-gesture recognition | `AI_self_project` |
| Big Data Course | 2026 | Big-data practice coursework | `BigData_Course` |
| Telegram / UMT Chatbots | 2026 | Chatbot projects | `TelebotProject`, `ChatBotUMT` |
| Trading Agent | 2026 | AI-assisted financial analysis agent | `vibe_trading` |
| Retail Analytics — "Heading for the Future: Retail" | 2025 – 2026 | Applied quantitative analysis to retail-sector data | Data analysis |

### 5.7 Internship Application — Nam A Bank
- Prepared an application for the **Personal Customer** position at **Nam A Bank
  (An Dong Branch)**.

---

## 6. Skills (Hard Skills)

### Programming & Algorithms
- **C++** and **Python** — solving algorithmic problems and writing system utilities.

### Database Management
- Relational DBMS: **MySQL, MariaDB, PostgreSQL**.
- Cloud platform: **Supabase**.
- Database design, normalization, complex queries, ER modeling (MySQL Workbench).

### Operating Systems & Systems
- **Ubuntu Linux** for system administration, CLI operations, and software installation.

### Artificial Intelligence & Data
- Research/application of **Large Language Models (LLMs)**, **Agentic RAG**, and
  **Computer Vision**.
- Machine learning with **XGBoost, LightGBM, scikit-learn**; imbalance handling and model
  evaluation.
- Data visualization & analysis with **Power BI** and **Tableau**.

### Computer Networking
- Switch configuration and routing study; **Cisco CCNA** fundamentals review.

### Presentation & Reporting
- Academic and technical report writing in **LaTeX**.
- Developer tooling: **Git/GitHub**, **Postman**, **MySQL Workbench**, Jira, Trello, Asana.

---

## 7. Certifications & Training

| Item | Provider | Period | Status |
|---|---|---|---|
| Samsung Innovation Campus (SIC) | Samsung | 2026 | In progress |
| International Study Program | AIT, Thailand | Aug 2026 | Completed |
| IELTS Preparation | Nhà IELTS | 2026 | In progress |
| Cisco CCNA Fundamentals | Self-study / review | 2025 – 2026 | Foundations |

---

## 8. Languages

- **Vietnamese** — Native.
- **English** — Reading technical documents and basic professional communication; currently
  preparing for IELTS.

---

## 9. Profile Highlights (for quick CV bullets)

- 3rd-year IT student, AI specialization, **GPA 3.39/4.0**, expected graduation 2028.
- Built a full **end-to-end IoT/AIoT system** (6 ultrasonic sensors → ESP32-S3 → ESP-NOW /
  CoreIoT MQTT → LVGL touchscreen) with CI, secret scanning, and a 3-level test protocol.
- Trained an **ANN to replace the Manhattan heuristic in A\*** with a reproducible
  data→train→integrate→evaluate pipeline and quantitative benchmarks.
- Shipped a **production-grade full-stack product (Joblink)** with layered security and RLS.
- **President** of E-NG Club and **Vice President** of YEC (UMT) — event planning, budgeting,
  logistics, and communications.
- International experience at **AIT Thailand**; active in the tech community
  (**AI Riser Vietnam 2026**, Google Gemini Student Ambassador challenge).

---

## 10. Cross-References (this knowledge base)

- [`random_ideas/05-projects-review-blindsight-snakeann-joblink.md`](../random_ideas/05-projects-review-blindsight-snakeann-joblink.md)
  — deep review of the three flagship projects (IoT, ML, full-stack).
- [`iot_aiot/`](../iot_aiot/) — IoT / AIoT theory behind the blind-spot system (MQTT,
  edge processing).
- [`ai_ml/`](../ai_ml/) — ML, RAG, and engineering-practice notes feeding Snake_ANN and
  Agentic RAG.
- [`books_summaries/agile_project_management/knowledge/version-control-devops-ci-cd.md`](../books_summaries/agile_project_management/knowledge/version-control-devops-ci-cd.md)
  — CI/CD & DevOps practices applied in UMT-TBS-official and Joblink.
- [`economics_business/`](../economics_business/) — business/economics foundation for the
  AI-Engineer-in-business goal.
- [`personal_evaluations/`](../personal_evaluations/) — English/technical skill evaluations
  and IELTS progress.

---

## 11. Research Journey

- **Why I'm building this profile:** I am aiming for AI/ML internships and, long-term, an
  AI Engineer role applied to Economics and Business. A single, accurate, English master record
  makes tailoring each application fast and keeps my story consistent.
- **What I struggled with:** My information was scattered across old Vietnamese LaTeX CVs,
  GitHub repositories, and memory. Contact details and project facts had drifted, and the major
  needed correcting to **Information Technology — AI specialization**.
- **Aha moments:** Reviewing the GitHub repos showed my three flagship projects are actually one
  system-thinking story — **sensor → edge → cloud** (blind-spot), **data → model → integration**
  (Snake_ANN), and **product + security + team process** (Joblink). They map directly onto how
  real AI systems are built and shipped.
- **Career connection:** This file is the portfolio backbone. Next step is generating
  role-specific CVs (AI / Data / Software) and a public portfolio page from this single source.

---

*Maintained as the single source of truth. Role-specific CVs should be derived from — never
become — this file.*
