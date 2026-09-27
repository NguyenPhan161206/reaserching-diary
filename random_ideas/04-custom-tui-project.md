# Custom Multi-TUI Project Idea (AGY & Opencode)

**Created Date:** 2026-09-12
**Tags:** `#ProjectIdea`, `#Python`, `#Textual`, `#TerminalUI`, `#AI_Agents`

## 💡 Core Idea
Build a custom Terminal User Interface (TUI) for interacting with AI agents, instead of using the default CLI. The project will support several engines, specifically:

1. **AGY TUI:** an interface for interacting with the Antigravity agent (using the `google-antigravity` Python SDK).
2. **Opencode TUI:** an interface for interacting with Opencode (via the corresponding API/SDK).

## 🛠️ Planned Tech Stack
- **Language:** Python (>= 3.10)
- **UI library:** `textual` (builds modern terminal interfaces and handles async well).
- **SDKs:** `google-antigravity` and Opencode's SDK.

## 🗂️ Proposed Architecture (multi-repo)
For safety, dependency isolation and the open/closed nature of the projects (private vs open source), the project will be split into 3 separate repositories:

1. **`reaserching-diary`**: the current repository, containing Markdown notes only.
2. **`agy-tui`**: an independent Python repository with the interface and the `google-antigravity` SDK integration.
3. **`opencode-tui`**: an independent Python repository with the interface and Opencode's API integration.

*(Note: this accepts code duplication of basic UI components such as Header, Footer and ChatBox across the 2 code repositories, in exchange for absolute independence.)*

## 📝 Action Items (Next Steps)
- [ ] Initialise the project directory OUTSIDE the `reaserching-diary` repository to avoid log clutter.
- [ ] Set up `venv` and install `textual`.
- [ ] Build the basic UI scaffold (Header, RichLog, Input) in a `shared/` directory.
- [ ] Integrate the AGY SDK into the UI scaffold and handle the stream flow (thoughts, tool calls, text).
