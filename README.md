# Researching Diary 📖

<table>
  <tr>
    <td align="center">
      <img src="b0a2877b09cb8895d1da.jpg" alt="Phan Hữu Bình Nguyên" width="200" style="border-radius: 50%;">
    </td>
    <td>
      <h3>Phan Hữu Bình Nguyên</h3>
      <ul>
        <li><b>Occupation:</b> IT student</li>
        <li><b>Dream:</b> Master</li>
        <li><b>Career Goal:</b> AI Engineer (Economics & Business fields)</li>
        <li><b>Currently Researching:</b> AI, IoT, AIoT, Economics, Politics, Psychology</li>
        <li><b>Hobbies:</b>
          <ul>
            <li>Exploring and applying knowledge</li>
            <li>Traveling and exploring places</li>
            <li>Networking</li>
          </ul>
        </li>
        <li><b>Contact:</b> <a href="mailto:phanhuubinhnguyen30@gmail.com">phanhuubinhnguyen30@gmail.com</a></li>
      </ul>
    </td>
  </tr>
</table>

---

Welcome to **Researching Diary**! This is a personal repository where I document my journey of exploring and researching **everything in the world**—from technology, science, and life to the fascinating little things of everyday existence.

## 🔄 Active Learning Methodology
More than just a storage space, this repository acts as the core of my active learning workflow. Because I am pressed for time, I use a seamless loop of technology and AI to study efficiently:

- 💻 **Terminal & Git Version Control:** Leveraging the Ubuntu Linux terminal and GitHub to quickly push and consolidate the technical and business concepts I skim through.
- 🤖 **AI-Driven Interleaved Practice:** Actively interacting with AI models (like Antigravity/agy) to analyze materials and generate targeted Q&A tests based on my notes.
- 📈 **Continuous Evaluation & Feedback:** Using real-time AI feedback to push my understanding deeper, systematically upgrade my English proficiency (IELTS standard), and evaluate my professional competence for my future career.

## 🎯 Core Objectives
This repository is not merely a collection of simple notes, but a tightly organized knowledge system designed to support long-term development:
- **Building a "Second Brain":** Digitizing and connecting all acquired knowledge, experiences, and collected materials. This offloads short-term memory, makes information retrieval effortless, connects ideas, and sparks creativity.
- **Developing Interdisciplinary Thinking:** Combining the logical and systematic mindset of Engineering/Technology (AI, IoT) with the analytical and empathetic perspective of the Social Sciences (Economics, Politics, Psychology) to work toward becoming an AI Engineer capable of solving practical business challenges.
- **Tracking Personal Growth:** Serving as clear evidence of "Exploring & Applying", allowing me to reflect on changes in cognition and the broadening of my worldview across each stage.

## 📂 Repository Structure (Knowledge Base)
The knowledge system is designed and organized into specialized directories, reflecting the exact domains I am pursuing:

- 🤖 [`/ai_ml`](ai_ml/README.md) - Artificial Intelligence, Machine Learning, Deep Learning, and applications in economics/business. 8 notes; the professional track — concepts here, code in [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) and [Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection).
- 📡 `/iot_aiot` - Internet of Things, AIoT architecture, embedded hardware, UAV/UAS systems (C2 links, onboard AI, drone logistics), and automation solutions.
- 📊 `/economics_business` - Macro/Microeconomics, business strategy, management, and market analysis.
- 🧠 `/psychology_society` - Behavioral psychology, politics, and how society functions.
- 📚 `/books_summaries` - Summaries, reviews, and key takeaways distilled from books. Sub-folders per book: `05-agile_project_management/`, `06-building-a-second-brain/`.
- 📈 `/personal_evaluations` - Personal competence tracking, English/technical skill evaluations, and feedback logs.
- 🔤 `/vocabulary` - Advanced vocabulary & collocation notes (Template V), focused on register, confusables, and Viet-Glish pitfalls.
- 🎧 `/ielts_training` - IELTS preparation practice (listening, reading, writing, speaking) using real-world content.
- 🔧 `/git_github` - Git version control & GitHub collaboration — practical mastery from first commit to team workflows.
- 🐧 `/dev_foundation` - Engineering fundamentals: Linux commands, Docker, YAML/config, web basics (HTML/CSS/JS/DOM), the Express SSR backend (`web_server/`) with its security chapter (`security/`), databases, the DSA anchor, and the Native Android track (`mobile_android/`). See [`dev_foundation/README.md`](dev_foundation/README.md) for the learning path.
- 🏢 `/software_architecture` - System architecture & design: event-driven back-ends, modularity vs microservices, application architecture models, and web performance/caching.
- 📄 `/cv` - LaTeX CV in English + Vietnamese (source + build script + rendered PDFs).
- 💡 `/random_ideas` - Personal perspectives, random ideas, or breakthrough project concepts that come to mind.
- 📥 `/inbox` - Fleeting notes, drafts, and unclassified ideas (staging area).

## 🗺️ Learning Path (dev_foundation)

Technical foundations are studied in a deliberate order, because each layer assumes the previous one:

```
git_github/ (done: 11 notes)
      ↓
dev_foundation/linux/     ← start here: filesystem, permissions, navigation, pipes
      ↓  (runs in parallel)
dev_foundation/dsa/ ──────────► [DSA-reserching-diary]  ← code lives in its own repo
      roadmap → complexity toolkit → structure-selection guide
      → Java (course language) + C++ (port target)
      JavaScript is the web track, below — not a DSA language
      ↓
dev_foundation/docker/  ·  dev_foundation/yaml_config/
      ↓
dev_foundation/web_basics/  ·  dev_foundation/web_server/  ·  dev_foundation/security/  ·  dev_foundation/databases/
      (Web Application Development course track — completed 2026-10-10)
      ↓
dev_foundation/mobile_android/   (Mobile Application Development course track — written 2026-10-11)
      ↓
Real projects: Joblink · UMT-Truck-Blind-Sight · Snake_ANN
```

Every gap found while building goes back into `dev_foundation/` as a note.

## 🚀 Featured Projects

Hands-on projects where I apply and deepen what I research in this diary:

| Project | Description | Stack |
|---------|-------------|-------|
| [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) | IoT blind-spot warning system: ESP32-S3 + waterproof ultrasonic sensor detects vehicles near trucks, streams distance/state to CoreIoT (ThingsBoard) cloud, and renders real-time warnings on a 7" LVGL touch LCD. | ESP32-S3, JSN-SR04T, CoreIoT/MQTT, ESP-IDF, LVGL v9, FreeRTOS |
| [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) | AI research project: an Artificial Neural Network learns to replace the handcrafted Manhattan heuristic inside A* pathfinding for the Snake game, accounting for obstacles & the snake's own body. Multi-stage pipeline (data generation → training → integration → evaluation). | Python, TensorFlow, Pygame, NumPy |
| [Joblink](https://github.com/qminh77/joblink) | Full-stack job-seeking social network: posts, connections, real-time messaging, job recruiting, CVs, notifications, and a full admin panel. Layered security (Zod → Server Action guards → RLS). Team project for SE005. | Next.js 16, React 19, TypeScript, Tailwind, Supabase (PostgreSQL/Realtime), TanStack Query |

### 🔗 Companion repositories

Some knowledge **needs code to make sense**. Those codebases live in their own repositories, and this diary keeps the concepts, roadmaps, and pointers:

| Repository | Holds | This diary holds |
|------------|-------|------------------|
| [Credit_Card_Fraud_Detection](https://github.com/NguyenPhan161206/Credit_Card_Fraud_Detection) | Source inspection, executable work and experimental evidence | [Fraud detection foundations](ai_ml/08-fraud-detection-foundations.md), concepts and learning reflections |
| [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) | Algorithm solutions, per-problem write-ups, practice log, error journal | [Roadmap & anchor](dev_foundation/dsa/README.md), complexity/tooling notes, reflections |

*Rule: no solution code is duplicated into this repo — notes point instead (see `AGENTS.md` Section L).*


## 📋 Quick Links
| Document | Purpose |
|----------|---------|
| [AGENTS.md](AGENTS.md) | AI agent configuration & rules |
| [NOTE_TEMPLATE.md](NOTE_TEMPLATE.md) | 7 template variants per domain |
| [RESEARCH_AREAS.md](RESEARCH_AREAS.md) | Research tracking & domain coverage |
| [CONTRIBUTION_CALENDAR.md](CONTRIBUTION_CALENDAR.md) | Visual activity calendar |
| [dev_foundation/README.md](dev_foundation/README.md) | Engineering fundamentals — learning path & stack map |
| [dev_foundation/web_server/README.md](dev_foundation/web_server/README.md) | Express SSR track — MVC, routing, templates, sessions, auth |
| [dev_foundation/security/README.md](dev_foundation/security/README.md) | Web security track — injection, XSS, CSRF, authZ, DoS |
| [books_summaries/06-building-a-second-brain/](books_summaries/06-building-a-second-brain/README.md) | The framework behind this diary — PARA, progressive summarization, Zettelkasten |
| [RESEARCH_AREAS.md](RESEARCH_AREAS.md#language-compliance-audit-agentsmd-section-a6) | Language compliance audit (A6) — files pending English translation |

## 🚀 Status & Roadmap
- **Current Stage:** `Active` — *Mobile Application Development course track* — Native Android (Kotlin): lifecycle, UI, storage, coroutines, Firebase (written 2026-10-11).
- **Current track:**
  1. ✅ Git & GitHub fundamentals — [`git_github/`](git_github/README.md) (11 notes)
  2. 🔄 Linux commands — [`dev_foundation/linux/`](dev_foundation/linux/README.md) (3 / 9 notes)
  3. 🔄 DSA method notes — [`dev_foundation/dsa/`](dev_foundation/dsa/README.md) (5 notes: roadmap, complexity, structure selection, Java, C++); practice code in [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary)
  4. ✅ Web basics — [`dev_foundation/web_basics/`](dev_foundation/web_basics/README.md) (8 notes)
  5. ✅ Express SSR backend — [`dev_foundation/web_server/`](dev_foundation/web_server/README.md) (8 notes)
  6. ✅ Web security — [`dev_foundation/security/`](dev_foundation/security/README.md) (2 notes)
  7. ✅ Databases — [`dev_foundation/databases/`](dev_foundation/databases/README.md) (5 notes)
  8. ⬜ Docker & YAML — [`dev_foundation/`](dev_foundation/README.md)
  9. ✅ Mobile (Native Android) — [`dev_foundation/mobile_android/`](dev_foundation/mobile_android/README.md) (6 notes, written 2026-10-11)
- **Upcoming Plans:**
  - [x] Set up directory structure and note-taking standards.
  - [x] Push initial notes on AI and Economics to the repository.
  - [x] Configure AI Agent system (OpenCode + AGY).
- [x] Complete the Web Application Development track (26 notes across web_basics / web_server / security / databases / docker / software_architecture).
   - [x] Complete the Mobile Application Development track (6 notes in dev_foundation/mobile_android/).
   - [ ] Complete the Linux command track (notes 04–09).
  - [ ] Docker core notes (01–04) + YAML notes (01–03).
  - [ ] Start the DSA practice loop (5–8 problems/week).
  - [ ] Maintain a habit of periodically updating new knowledge.
  - [ ] Build contribution streak 🔥

## 🤖 AI Agent System
This repo uses AI agents to manage content. Configuration files:

```
AGENTS.md                    ← Shared rules (both agents)
├── .opencode/               ← OpenCode config
│   ├── opencode.json        ← Project settings
│   ├── agents/              ← Agent definitions
│   ├── commands/            ← Slash commands (/new-note, /suggest, etc.)
│   └── skills/              ← Content management skill
└── .gemini/
    └── GEMINI.md            ← AGY (Antigravity) overrides
```

**Commands available (OpenCode):**
- `/new-note` — Create a new research note
- `/suggest` — Suggest improvements & missing topics
- `/review` — Review template compliance
- `/log` — Log today's activity
- `/propose-folder` — Propose new folder for unclassified content
- `/push-ready` — Check push readiness

---
*"Live as if you were to die tomorrow. Learn as if you were to live forever."* - Mahatma Gandhi
