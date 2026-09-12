# Custom Multi-TUI Project Idea (AGY & Opencode)

**Created Date:** 2026-09-12
**Tags:** `#ProjectIdea`, `#Python`, `#Textual`, `#TerminalUI`, `#AI_Agents`

## 💡 Ý tưởng cốt lõi (Core Idea)
Xây dựng một hệ thống Terminal User Interface (TUI) tùy chỉnh để tương tác với các AI Agent, thay vì dùng CLI mặc định. Dự án sẽ hỗ trợ nhiều engine khác nhau, cụ thể là:
1. **AGY TUI**: Giao diện tương tác với Antigravity Agent (sử dụng `google-antigravity` Python SDK).
2. **Opencode TUI**: Giao diện tương tác với Opencode (tích hợp API/SDK tương ứng).

## 🛠️ Tech Stack dự kiến
- **Ngôn ngữ:** Python (>= 3.10)
- **Thư viện UI:** `textual` (xây dựng giao diện terminal hiện đại, hỗ trợ async tốt).
- **SDK:** `google-antigravity` và SDK của Opencode.

## 🗂️ Kiến trúc đề xuất (Mono-repo)
Sử dụng chung một repository và một môi trường ảo (`venv`) để dễ quản lý và tái sử dụng code giao diện (khung chat, ô input).

```text
custom_terminal_uis/
├── requirements.txt
├── shared/
│   ├── __init__.py
│   └── ui_components.py    # (Các widget UI dùng chung: RichLog chat, Input bar)
├── agy_tui/
│   ├── __init__.py
│   └── main.py             # App tích hợp google-antigravity SDK
└── opencode_tui/
    ├── __init__.py
    └── main.py             # App tích hợp Opencode SDK/API
```

## 📝 Action Items (Next Steps)
- [ ] Khởi tạo thư mục dự án nằm NGOÀI repository `reaserching-diary` để tránh rác log.
- [ ] Setup `venv` và cài đặt `textual`.
- [ ] Xây dựng khung UI cơ bản (Header, RichLog, Input) trong thư mục `shared/`.
- [ ] Tích hợp thử AGY SDK vào khung UI và xử lý luồng stream (thoughts, tool calls, text).
