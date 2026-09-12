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

## 🗂️ Kiến trúc đề xuất (Multi-repo)
Vì lý do an toàn, độc lập dependencies và tính chất đóng/mở (Private vs Open Source), dự án sẽ được chia thành 3 repositories riêng biệt:

1. **`reaserching-diary`**: Repo hiện tại, thuần túy chứa Markdown để ghi chép.
2. **`agy-tui`**: Repo Python độc lập, chứa giao diện và tích hợp SDK `google-antigravity`.
3. **`opencode-tui`**: Repo Python độc lập, chứa giao diện và tích hợp API của Opencode.

*(Lưu ý: Chấp nhận việc lặp lại code (Code Duplication) ở các thành phần UI cơ bản như Header, Footer, ChatBox giữa 2 repo code để đổi lấy sự độc lập tuyệt đối).*

## 📝 Action Items (Next Steps)
- [ ] Khởi tạo thư mục dự án nằm NGOÀI repository `reaserching-diary` để tránh rác log.
- [ ] Setup `venv` và cài đặt `textual`.
- [ ] Xây dựng khung UI cơ bản (Header, RichLog, Input) trong thư mục `shared/`.
- [ ] Tích hợp thử AGY SDK vào khung UI và xử lý luồng stream (thoughts, tool calls, text).
