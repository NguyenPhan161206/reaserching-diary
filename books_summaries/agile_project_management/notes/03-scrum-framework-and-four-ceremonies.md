# Phiên 03 — Scrum Framework & 4 Ceremony Chi Tiết

**Ngày:** 2026-09-12
**Nguồn kiến thức:** [[../knowledge/part-2-being-agile.md]], [[../knowledge/part-3-planning-and-execution.md]]
**Tags:** `#Scrum`, `#DailyStandup`, `#SprintPlanning`, `#SprintReview`, `#SprintRetrospective`, `#StatusReport`

> 📌 Đây là phiên Q&A đã diễn ra trong chat và bị mất do Ctrl+C — nội dung được khôi phục lại từ database opencode.

---

## ❓ Câu hỏi 1: Scrum là gì?

### 📚 Trả lời chi tiết

**Scrum** là một **framework** (khuôn khổ) Agile phổ biến nhất, dùng để quản lý công việc tri thức phức tạp. Không phải quy trình cứng nhắc — là "bộ khung" bạn đổ đầy bằng team của mình.

Scrum gồm 3 trụ cột (**3 pillars**):
- **Transparency**: mọi thứ nhìn thấy được (backlog, bảng Kanban, burndown)
- **Inspection**: kiểm tra thường xuyên (Standup mỗi ngày, Review cuối sprint)
- **Adaptation**: điều chỉnh ngay khi phát hiện lệch (Retro mỗi sprint)

**Cấu trúc 1 Sprint:**
```
Sprint (1-4 tuần, timeboxed)
├── Sprint Planning      → chọn User Stories, chốt mục tiêu
├── Daily Standup (mỗi ngày) → đồng bộ 24h
├── Sprint Review        → demo cho stakeholder, nhận feedback
└── Sprint Retrospective → team cải tiến process
```

**3 vai trò** (roles):
| Role | Sở hữu | Trách nhiệm |
|---|---|---|
| Product Owner (PO) | "What" | Tối ưu giá trị backlog, quyết ưu tiên, nói "không" với stakeholder |
| Scrum Master (SM) | "Process" | Servant-leader: dọn impediment, coach, không quản lý người |
| Development Team | "How" | Tự tổ chức, tự phân việc, cross-functional |

**Artifacts** (sản phẩm quản lý): Product Backlog (danh sách ưu tiên), Sprint Backlog (việc của sprint này), Increment (phần sản phẩm hoàn thiện "Done").

📌 Nguồn trong thư mục: `knowledge/part-2-being-agile.md` (Chapter 5 mục Scrum) và `knowledge/part-3-planning-and-execution.md` (ceremonies).

---

## ❓ Câu hỏi 2: "Báo cáo tis" / status report là gì?

### 📚 Trả lời chi tiết

**Status report** = báo cáo trạng thái kiểu truyền thống: mỗi người/tuần liệt kê *"Tôi đã làm gì, đến đâu rồi, theo % đúng kế hoạch"* để cấp trên kiểm soát.

**Vấn đề của nó trong Agile:**
- Standup KHÔNG phải status report lên sếp. Nếu team đứng họp mà nói chuyện với trạng thái "sếp đang nghe" → thành "báo cáo tí tách" — lãng phí 15 phút và tạo văn hóa phòng thủ (ai cũng kể công, giấu khó khăn).
- Standup đúng nghĩa là **team nói chuyện với nhau**: "tôi kẹt chỗ này, ai giúp được?" → impediment được tháo ngay, không cần chờ báo cáo cuối tuần.

> Công thức phân biệt: *Status report = thông tin đi LÊN để ai đó kiểm soát. Standup = thông tin đi NGANG để team tự phối hợp.*

---

## ❓ Câu hỏi 3: Giải thích kỹ 4 ceremony của Scrum

### 📚 Trả lời chi tiết

Một sprint chuẩn có **4 event** (Scrum Guide gọi Sprint là "container" chứa cả 4):

### 🔷 A. Sprint Planning (đầu sprint)
- **Ai:** PO + Development Team (+ SM hỗ trợ).
- **Timebox:** ~2 giờ cho mỗi tuần sprint (sprint 2 tuần → họp ~4 giờ).
- **Trả lời 2 câu:** *"Chúng ta sẽ làm gì (Which)?"* và *"Làm thế nào (How)?"*
- **Diễn biến:**
  1. PO trình bày các User Stories ưu tiên cao nhất từ Product Backlog kèm rõ **mục tiêu sprint (Sprint Goal)**.
  2. Team chọn stories đủ để cam kết trong thời gian có (based on **Velocity**).
  3. Team tự chia task (đừng để manager giao!) và ước lượng.
- **Tại sao quan trọng:** là nơi duy nhất team "cam kết" scope của sprint — nếu bỏ qua, sprint không có điểm xuất phát rõ.

> **Ví dụ:** Sprint Goal: *"Người dùng mới có thể đăng ký bằng Google trong 1 phút."* → Stories: "Đăng nhập OAuth", "Tạo profile mặc định", "Email chào mừng". Team nhận thấy đã ước lượng xong 2 story, story 3 chưa rõ → đưa lên bàn, thống nhất bỏ đợt này cho chắc "Done".

### 🔷 B. Daily Standup / Daily Scrum (MỖI ngày trong sprint)
- **Ai:** Chỉ Development Team (PO/SM tham gia tùy, KHÔNG bắt buộc). Có thể không cần SM.
- **Timebox:** **15 phút. Đứng.** Đúng giờ, kể cả trễ.
- **3 câu hỏi:**
  1. Hôm qua tôi làm gì (đóng góp cho Sprint Goal)?
  2. Hôm nay tôi sẽ làm gì?
  3. Có impediment gì đang cản tôi?
- **Không phải**: báo cáo sếp, không giải quyết vấn đề ngay tại họp (việc sâu → để breakout sau họp).
- **Tại sao quan trọng:** là "nhịp tim" 24h — cho phép Inspection micro: nếu ai đó lệch hướng, team phát hiện trong ngày thay vì cuối tuần.

> **Ví dụ:** "Hôm qua tôi xong API login. Hôm nay làm trang profile. Cản trở: chưa có test credentials từ team backend — cần ai đó cấp giúp." → Một dev khác: "Để tôi cấp sau 10 phút." Problem chết ngay trong họp.

### 🔷 C. Sprint Review (cuối sprint — focus SẢN PHẨM)
- **Ai:** Team + **stakeholders/khách hàng** (bắt buộc có!) + PO.
- **Timebox:** ~1 giờ/tuần sprint (sprint 2 tuần → ~2 giờ).
- **Nội dung:**
  1. Team demo **working software** (không phải slide!) — phần nào "Done".
  2. Stakeholder xem, thử, cho feedback.
  3. PO cập nhật Product Backlog dựa trên feedback (thêm/đổi/ưu tiên lại).
- **Tại sao quan trọng:** là cơ chế **thu feedback sớm nhất có thể** — "building the wrong thing" là rủi ro chết người nhất; Review là vaccine chống nó. Chỗ này khách "ngồi chung bàn" ở Value #3 của Manifesto.

> **Ví dụ:** Demo xong tính năng "đăng ký Google", khách: "Ồ hay, nhưng tụi mình cần cả đăng nhập bằng số điện thoại nữa." → PO ghi story mới, ưu tiên cao vào backlog → sprint sau team làm. Nếu không có Review, team làm xong mới biết khách cần thêm cái khác → tốn 1 sprint.

### 🔷 D. Sprint Retrospective (cuối sprint — focus QUY TRÌNH)
- **Ai:** Chỉ team (KHÔNG stakeholder). SM facilitate.
- **Timebox:** ~45-60 phút.
- **Nội dung:** nhìn lại CÁCH team làm việc trong sprint vừa rồi. Framework kinh điển: **Start / Stop / Continue**:
  - *Start:* điều mới nên làm (vd: thêm code review)
  - *Stop:* thói quen hại (vd: commit thẳng vào main)
  - *Continue:* giữ nguyên (vd: pair programming hiệu quả)
- **Quan trọng nhất:** chọn **1-3 action item CỤ THỂ, có chủ sở hữu** để sprint sau thực hiện ngay. Retro mà không có action item = họp suông.
- **Tại sao quan trọng:** là vòng **Adaptation** của vòng lặp — chỗ duy nhất team nâng cấp cách làm việc. "Work smarter" xảy ra ở đây, không phải ở chỗ code.

> **Ví dụ:** Team nhận ra sprint vừa rồi 3 lần deadline trễ vì story "xong" nhưng lại vỡ ở production. → Action item: *"Lần sau, story chỉ Done khi pass CI + chạy được trên staging — assign: Minh, deadline: trước Sprint Planning tới."*

---

## 🔄 Tóm tắt 4 ceremony bằng 4 câu

| Ceremony | Câu hỏi trung tâm | Focus |
|---|---|---|
| Sprint Planning | *"Chúng ta làm gì, và làm gì tiếp?"* | Quyết định |
| Daily Standup | *"24h tới, ai cần giúp gì?"* | Đồng bộ |
| Sprint Review | *"Sản phẩm này có đúng ý khách không?"* | Sản phẩm + feedback |
| Retrospective | *"Cách chúng ta làm việc có tốt hơn chưa?"* | Quy trình |

> 💡 Mẹo nhớ: **Planning = nhìn tới, Review = nhìn sản phẩm, Retro = nhìn chính team. Standup giữ cho cả 3 chạy đúng hướng mỗi ngày.**

---

## 💡 Tổng kết phiên

- [x] Hiểu Scrum = framework với 3 pillars (Transparency, Inspection, Adaptation)
- [x] Nắm 3 roles (PO/SM/Dev Team) + artifacts (Product/Sprint Backlog, Increment)
- [x] Phân biệt Standup (đi NGANG, phối hợp) vs Status report (đi LÊN, kiểm soát)
- [x] Giải thích chi tiết 4 ceremony: ai, timebox, diễn biến, tại sao quan trọng
- [x] Retro phải có action item cụ thể kèm chủ sở hữu — nếu không là họp suông

## 🔄 Câu hỏi tiếp theo đề xuất

1. [SUGGESTED] **Velocity & Story Points** — vì Sprint Planning "chọn bao nhiêu story" dựa vào nó; không hiểu velocity thì planning thành đoán mò.
2. [SUGGESTED] **Definition of Done (DoD)** — thứ quyết định "story này có tính 'Done' không" trong Review.
3. [SUGGESTED] **Burndown chart** — information radiator đo mỗi ngày xem team có kịp sprint goal không.

---

## 🔗 Related Topics

- [[01-agility-mindset-and-manifesto.md]] — Phiên 01: mindset, Manifesto
- [[02-ceremony-gantt-core-concepts.md]] — Phiên 02: ceremony tổng quan, Gantt, hypothesis
- [[../knowledge/part-2-being-agile.md]]
- [[../knowledge/part-3-planning-and-execution.md]]
- [[README.md]] — index các phiên Q&A

---
*Học bằng cách hỏi đúng câu hỏi.*