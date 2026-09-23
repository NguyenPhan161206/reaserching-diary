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

- 🤖 `/ai_ml` - Artificial Intelligence, Machine Learning, Deep Learning, and applications in economics/business.
- 📡 `/iot_aiot` - Internet of Things, AIoT architecture, hardware, and automation solutions.
- 📊 `/economics_business` - Macro/Microeconomics, business strategy, management, and market analysis.
- 🧠 `/psychology_society` - Behavioral psychology, politics, and how society functions.
- 📚 `/books_summaries` - Summaries, reviews, and key takeaways distilled from books.
- 📈 `/personal_evaluations` - Personal competence tracking, English/technical skill evaluations, and feedback logs.
- 🎧 `/ielts_training` - IELTS preparation practice (listening, reading, writing, speaking) using real-world content.
- 🔧 `/git_github` - Git version control & GitHub collaboration — practical mastery from first commit to team workflows.
- 📄 `/cv` - LaTeX CV in English + Vietnamese (source + build script + rendered PDFs).
- 💡 `/random_ideas` - Personal perspectives, random ideas, or breakthrough project concepts that come to mind.
- 🏗️ `/software_architecture` - System architecture & design: event-driven back-end, modularity vs microservices, and agent interfaces — the engineering layer for building real-world systems.
- 📥 `/inbox` - Fleeting notes, drafts, and unclassified ideas (staging area).

## 🚀 Featured Projects

Hands-on projects where I apply and deepen what I research in this diary:

| Project | Description | Stack |
|---------|-------------|-------|
| [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) | IoT blind-spot warning system: ESP32-S3 + waterproof ultrasonic sensor detects vehicles near trucks, streams distance/state to CoreIoT (ThingsBoard) cloud, and renders real-time warnings on a 7" LVGL touch LCD. | ESP32-S3, JSN-SR04T, CoreIoT/MQTT, ESP-IDF, LVGL v9, FreeRTOS |
| [Snake_ANN](https://github.com/NguyenPhan161206/Snake_ANN) | AI research project: an Artificial Neural Network learns to replace the handcrafted Manhattan heuristic inside A* pathfinding for the Snake game, accounting for obstacles & the snake's own body. Multi-stage pipeline (data generation → training → integration → evaluation). | Python, TensorFlow, Pygame, NumPy |
| [Joblink](https://github.com/qminh77/joblink) | Full-stack job-seeking social network: posts, connections, real-time messaging, job recruiting, CVs, notifications, and a full admin panel. Layered security (Zod → Server Action guards → RLS). Team project for SE005. | Next.js 16, React 19, TypeScript, Tailwind, Supabase (PostgreSQL/Realtime), TanStack Query |

## 📋 Quick Links
| Document | Purpose |
|----------|---------|
| [AGENTS.md](AGENTS.md) | AI agent configuration & rules |
| [NOTE_TEMPLATE.md](NOTE_TEMPLATE.md) | 5 template variants per domain |
| [RESEARCH_AREAS.md](RESEARCH_AREAS.md) | Research tracking & domain coverage |
| [CONTRIBUTION_CALENDAR.md](CONTRIBUTION_CALENDAR.md) | Visual activity calendar |

## 🚀 Status & Roadmap
- **Current Stage:** `Active` (Agent system established).
- **Upcoming Plans:**
  - [x] Set up directory structure and note-taking standards.
  - [x] Push initial notes on AI and Economics to the repository.
  - [x] Configure AI Agent system (OpenCode + AGY).
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
