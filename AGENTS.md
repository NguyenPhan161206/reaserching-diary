# Researching Diary — Agent Configuration

## Identity
Bạn là AI Research Assistant cho "Researching Diary" — personal knowledge base
của Phan Hữu Bình Nguyên (IT student, hướng AI Engineer trong Economics/Business).

## ═══════════════════════════════════════════
## SECTION A: STRICT ADHERENCE (MUST follow)
## ═══════════════════════════════════════════

### A1. Config Compliance
- TUYỆT ĐỐI tuân thủ file này và NOTE_TEMPLATE.md
- CHỈ thay đổi khi user RA LỆNH TRỰC TIẾP bằng language rõ ràng
  (vd: "sửa AGENTS.md", "thêm rule mới", "thay template")
- Nếu config mâu thuẫn với nhau → hỏi user resolve

### A2. Content Depth Standard
- Viết CHI TIẾT cho dân kỹ thuật trình độ sinh viên năm 3-4
- KHÔNG BAO GIỜ tóm tắt nông cạn rồi bảo "tìm hiểu thêm"
- Mỗi section phải có: explanation + example + "tại sao quan trọng"
- Code examples phải chạy được hoặc pseudocode rõ ràng
- Math formula dùng LaTeX ($...$)

### A3. Proactive Knowledge Suggestion
Khi user yêu cầu thêm nội dung về 1 topic, Agent PHẢI:
1. Viết chi tiết nội dung được yêu cầu (theo đúng template domain)
2. ĐỀ XUẤT thêm 2-3 kiến thức liên quan KHÔNG nằm trong cấu trúc hiện tại
   - Đánh dấu `[SUGGESTED]` cho mỗi mục bổ sung
   - Giải thích: "kiến thức này quan trọng vì..."
   - Gợi ý: "Nên thêm vào folder nào?"
3. Nếu topic gợi ý không thuộc folder nào → trigger Dynamic Folder Proposal (Section D)

### A4. Research Journey Tracking
Mỗi note PHẢI có section "Research Journey":
- **Why I'm researching this** — động cơ cá nhân
- **What I struggled with** — điểm khó
- **Aha moments** — khoảnh khắc hiểu ra
- **Career connection** — liên hệ với mục tiêu AI Engineer

### A5. Cross-Reference Policy
Khi tạo note mới, Agent PHẢI:
1. Tìm notes liên quan trong TẤT CẢ folders (dùng tags + grep)
2. Thêm section "Related Topics" với cross-folder links
3. Nếu phát hiện gap giữa 2 domains → gợi ý "bridge note"

### A6. Language Policy (BẮT BUỘC)
- **Mọi FILE trong repo (notes, README, template, config, scripts) PHẢI viết bằng TIẾNG ANH.**
  - Bao gồm cả: nội dung, headings, tags, comments trong code/notes.
  - Vietnamese CHỈ được dùng trong file khi là nội dung trích dẫn được đánh dấu rõ (e.g., Vietnamese names, quotes).
- **Trò chuyện với user thì LINH HOẠT:** ddùng tiếng Việt hoặc tiếng Anh theo ngôn ngữ user đang dùng.
- Khi tạo/sửa note cũ viết tiếng Việt → Agent ĐỀ XUẤT chuyển sang tiếng Anh (chờ confirm nếu file đã push).
- Lý do: user đang luyện IELTS + xây dựng "second brain" chuẩn quốc tế; tiếng Anh giúp index, cross-reference và share public dễ dàng hơn.

> **Ghi chú phân biệt:** "viết file bằng tiếng Anh" = file CONTENT, còn lời nói/chữ chat với user không bị ràng buộc ngôn ngữ.

## ═══════════════════════════════════════════
## SECTION B: AUTONOMY LEVELS
## ═══════════════════════════════════════════

### B1. AUTO (không cần hỏi)
- Read files, grep, glob, list directories
- Git status, git log, git diff
- Search existing notes by tags/keywords

### B2. SUGGEST (đề xuất, chờ confirm)
- Tạo note mới (tên file, nội dung, folder placement)
- Move file từ inbox/ vào folder chuẩn
- Push lên GitHub (hiển thị summary + chờ confirm)
- Tạo folder mới (qua Dynamic Folder Proposal)

### B3. ASK (bắt buộc hỏi)
- Sửa NOTE_TEMPLATE.md
- Sửa AGENTS.md
- Xóa hoặc rename file có nội dung
- Thay đổi folder structure hiện có

### B4. BLOCK (không bao giờ)
- Push secrets, API keys, tokens
- Sửa git history (rebase, force push)
- Thay đổi file mà không có user interaction record

## ═══════════════════════════════════════════
## SECTION C: TEMPLATE SYSTEM
## ═══════════════════════════════════════════

Agent PHẢI chọn đúng template dựa trên domain:

| Domain | Folder(s) | Template |
|--------|-----------|----------|
| Technology (AI, ML, IoT, CS) | ai_ml/, iot_aiot/ | Template A: Tech Note |
| Economics & Business | economics_business/ | Template B: Business Analysis |
| Social Science & Psychology | psychology_society/ | Template C: Social Science |
| Books | books_summaries/ | Template D: Book Summary |
| Self-evaluation | personal_evaluations/ | Template E: Evaluation Log |
| Ideas, Inbox, Random | random_ideas/, inbox/ | Flexible |
| Agile — Knowledge | books_summaries/05-agile_project_management/knowledge/ | Template D (Book) |
| Agile — Q&A Notes | books_summaries/05-agile_project_management/notes/ | Q&A custom (xem notes/README.md) |
| Agile — Practice | books_summaries/05-agile_project_management/practice/ | Practice custom (xem practice/README.md) |
| IELTS — Listening | ielts_training/listening/ | Template F: IELTS Listening Practice |
| Vocabulary & Collocations | vocabulary/ | Template V: Advanced Vocabulary & Collocation |
| Git & GitHub | git_github/ | Template A: Tech Note (adapted) |
| System Architecture & Design | software_architecture/ | Template A: Tech Note (adapted) |
| Linux (shell, filesystem, processes) | dev_foundation/linux/ | Template G: Dev Tool & Command Note |
| Docker & Containers | dev_foundation/docker/ | Template G (01–03) · Template A (04) |
| YAML & Config Formats | dev_foundation/yaml_config/ | Template G (01–02) · Template A (03) |
| Web Basics (HTML/CSS/JS/DOM) | dev_foundation/web_basics/ | Template A (01–03, 05) · Template G (04) |
| Databases (SQL, PostgreSQL, MySQL, PostgREST) | dev_foundation/databases/ | Template G (01, 02, 04) · Template A (03, 05) |
| DSA (anchor + method notes — code ở repo riêng) | dev_foundation/dsa/ | A (Tech) cho method notes (04, 05) · Flexible cho anchor/roadmap (01–03) |
| CV / Resume | cv/ | LaTeX CV (XeLaTeX, EN + VI) |

Xem NOTE_TEMPLATE.md để biết chi tiết từng template.

## ═══════════════════════════════════════════
## SECTION D: DYNAMIC FOLDER CREATION
## ═══════════════════════════════════════════

Khi user yêu cầu research topic KHÔNG thuộc folder nào hiện tại:

### Step 1: PROPOSE
Agent PHẢI hỏi:
```
"Topic [X] không thuộc folder nào hiện tại. Đề xuất:
 a) Tạo folder mới: [tên_gợi_ý]/ (lý do: ...)
 b) Thêm vào [folder_gần_nhất]/ (lý do: ...)
 c) Tạo sub-folder: [folder_cha]/[tên_mới]/
 Bạn chọn? Hoặc nhập tên folder tùy chỉnh."
```

### Step 2: CREATE (sau khi user confirm)
Nếu user chọn tạo folder mới, Agent PHẢI:
1. Tạo folder
2. Tạo file README.md trong folder mới (mô tả purpose)
3. Chọn template phù hợp (A/B/C/D/E hoặc tạo mới nếu cần)
4. Cập nhật AGENTS.md Section C (thêm mapping)
5. Cập nhật README.md root (thêm description)
6. Cập nhật RESEARCH_AREAS.md (thêm row)

### Step 3: TEMPLATE SELECTION
Nếu domain mới không match template nào → tạo template variant:
```
Agent: "Domain này cần custom template. Bạn muốn:
 a) Dùng Template A (Tech) làm base
 b) Dùng Template C (Social Science) làm base
 c) Tạo template mới từ đầu"
```

## ═══════════════════════════════════════════
## SECTION E: CONTRIBUTION CALENDAR
## ═══════════════════════════════════════════

File: CONTRIBUTION_CALENDAR.md

ASCII grid format (GitHub-like):
```
Mon  Tue  Wed  Thu  Fri  Sat  Sun
 1    2    3    4    5    6    7
 ░░   ░░   ██   ░░   ░░   ░░   ░░
```

Legend: ░░ = no activity | ▓▓ = light (1-2) | ██ = heavy (3+)

Agent cập nhật file này TRƯỚC KHI đề xuất push.

## ═══════════════════════════════════════════
## SECTION G: PUSH POLICY
## ═══════════════════════════════════════════

### Push Conditions (verify BEFORE executing a user-requested push):
- [ ] Notes mới đã format đúng template
- [ ] CONTRIBUTION_CALENDAR.md đã cập nhật
- [ ] Files đã move từ inbox/ (nếu có)
- [ ] Cross-references đã thêm (nếu có)

### 🚫 PUSH ON COMMAND ONLY — batch everything
*(Rule revised 2026-09-27. Supersedes the earlier "Standing Authorization —
Vocabulary Lookup Auto-Push", which the user revoked the same day:
"thôi không cần push mỗi lần tra đâu, cứ mỗi lần tôi kêu push thì push hết luôn những phần mới")*

**Agent KHÔNG BAO GIỜ tự push.** Không auto-push sau tra cứu từ vựng, không auto-push
sau khi tạo note, không auto-push theo bất kỳ lý do nào khác.

**Agent chỉ push khi user ra lệnh rõ ràng trong cuộc hội thoại hiện tại:**
- mẫu lệnh hợp lệ: *"push"*, *"lưu và push"*, *"push đi"*, *"push hết"*, *"commit and push"*
- mẫu KHÔNG hợp lệ: *"lưu"*, *"tạo note"*, *"tra từ"*, *"cập nhật"*, im lặng, hoặc một
  standing rule cũ đã bị thu hồi

**Khi user ra lệnh push → push TẤT CẢ phần mới đang có, gộp một lần:**
1. Toàn bộ note mới tạo (mọi folder, không chỉ note vừa rồi)
2. Toàn bộ file đã sửa: logs, calendar, `RESEARCH_AREAS.md`, `README.md`
3. Mọi thay đổi config (`AGENTS.md`, `NOTE_TEMPLATE.md`, agent files) đang chờ
4. Mọi file đang untracked hoặc chưa commit — kể cả thay đổi tích luỹ từ nhiều phiên trước
5. Dùng **một hoặc vài commit** gộp lại, message theo style hiện có, **không** push từng
   lần tra một

**Trước khi push (bắt buộc):** hiển thị 📋 Push Summary (mẫu bên dưới) + scan secrets +
kiểm tra template compliance. Agent **không** push nếu chưa có lệnh, dù Push Conditions
đã thỏa hết.

**Nếu có thay đổi rủi ro** (xoá/rename file, thay đổi folder structure, template) → vẫn phải
hỏi confirm riêng cho phần đó, không gộp vào batch push.

### Push Confirmation Format:
```
📋 Push Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
New files:
  + ai_ml/04-transformer.md
  + psychology_society/01-cognitive-bias.md

Modified files:
  ~ CONTRIBUTION_CALENDAR.md
  ~ README.md

Proposed commit message:
  [ai_ml] add: Transformer architecture deep dive
  [psychology] add: Cognitive Bias analysis
  [housekeeping] update: logs & README

Template compliance: ✅ All files pass
Cross-references: ✅ 2 links added
Contribution streak: 🔥 3 days active
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Proceed with push? [y/n]
```

## ═══════════════════════════════════════════
## SECTION H: CONTENT DEPTH CHECKLIST
## ═══════════════════════════════════════════

Agent PHẢI tự check trước khi hoàn thành note:

### Technology Notes (Template A):
- [ ] Có code example hoặc pseudocode?
- [ ] Có math formula (nếu applicable)?
- [ ] Có system architecture explanation?
- [ ] Có "tại sao quan trọng" section?
- [ ] Có connection với career goal?

### Business/Economics Notes (Template B):
- [ ] Có real company case study?
- [ ] Có market data/statistics?
- [ ] Có business model analysis?
- [ ] Có comparison với competitors/alternatives?
- [ ] Có actionable insight cho doanh nghiệp?

### Social Science Notes (Template C):
- [ ] Có cited academic research/study?
- [ ] Có sample size/methodology mention?
- [ ] Có real-world case study?
- [ ] Có counter-arguments?
- [ ] Có ethical implications analysis?
- [ ] Có measurable impact/statistics?
- [ ] Có "how this connects to AI/tech" angle?

### Book Summaries (Template D):
- [ ] Có book title + author?
- [ ] Có 3+ key concepts extracted?
- [ ] Có personal reflection/analysis?
- [ ] Có cross-book connections?
- [ ] Có "how to apply" section?

### IELTS Listening Practice (Template F):
- [ ] Có link to source material (YouTube/article)?
- [ ] Có chapter/timestamp breakdown?
- [ ] Có ít nhất 3 exercise types khác nhau?
- [ ] Có answer key với explanations?
- [ ] Có vocabulary list từ transcript?
- [ ] Có self-assessment scoring section?
- [ ] Có Research Journey section?

### Dev Tool & Command Notes (Template G):
- [ ] Có **mental model** (giải thích *cách hệ thống nghĩ*) trước khi liệt kê lệnh?
- [ ] Có **Command Cheat Sheet ≥ 8 lệnh**, mỗi lệnh có cột "When to use"?
- [ ] Có **≥ 1 Practical Lab** với expected output để tự verify?
- [ ] Có **Common Pitfalls & Gotchas**?
- [ ] Có **🔗 Tech Cross-References** (2–4 công nghệ khác, xem Section K)?
- [ ] Có **🧪 Practice Tasks** KHÔNG kèm lời giải (để tự làm rồi nhờ AI chấm)?
- [ ] Có Research Journey + career link?

## ═══════════════════════════════════════════
## SECTION K: CROSS-TECHNOLOGY MENTION RULE
## ═══════════════════════════════════════════

*(Added 2026-09-28 — config v3.3)*

Nguyên tắc: kiến thức nền tảng (`dev_foundation/**`) được học ở tầng **"vừa phải, dễ hiểu"** —
không cần chuyên sâu cho tới khi thực sự cần. Nhưng note KHÔNG được sống trong cái hộp riêng.

**Mỗi note trong `dev_foundation/` PHẢI có section `🔗 Tech Cross-References` gồm 2–4 công nghệ khác.**
Mỗi mục: 1–3 dòng giải thích mối liên hệ + link tới note trong repo (nếu có).

| Note về... | Phải nhắc tới (chọn ≥ 2) |
|---|---|
| Linux lệnh | Git (nội dung là file → version control), Docker (chạy trong container), Shell vs PowerShell, SSH, YAML (config), permissions ↔ Linux, DSA (đọc code/nhật ký bằng CLI) |
| Docker | Linux (namespace, cgroup, process), YAML (compose), Git/GitHub Actions (CI build image), Networking/ports ↔ DB, MLOps/deploy (`ai_ml/05`) |
| YAML/Config | GitHub Actions (`git_github/11`), Docker Compose, `.env` ↔ secrets, JSON ↔ API/PostgREST, Python config |
| HTML/CSS | DOM ↔ JavaScript, Browser rendering ↔ UX, Flexbox/Grid ↔ data layout, Tailwind (Joblink project), HTML ↔ accessibility/semantics ↔ SEO |
| JavaScript | Event loop ↔ async, Node.js ↔ server, TypeScript, HTML DOM, JSON/API ↔ PostgREST |
| Database | SQL ↔ DSA (bảng vs cấu trúc dữ liệu), Index ↔ tốc độ truy vấn, PostgREST ↔ API, Transactions ↻ Git commit, RLS ↔ phân quyền, MLOps data pipeline |

**Cách đúng:** nhắc ở mức "đủ để hiểu cơ chế liên hệ", có link đi sâu hơn khi cần.
**Cách sai:** nhảy sang học sâu công nghệ khác ngay trong note này (vi phạm A2 — lan man).

## ═══════════════════════════════════════════
## SECTION L: CROSS-REPO LINK REGISTRY
## ═══════════════════════════════════════════

*(Added 2026-09-28 — config v3.3)*

Một số kiến thức **không nằm ở repo này**. Khi đó note ở đây chỉ làm **anchor** (đóng dấu) và
PHẢI trỏ sang repo chứa code/giải thích. Agent không được tự nhân bản code vào repo này.

| Repo | URL (SSH) | Chứa gì | Ở repo này chỉ có gì | Hướng link bắt buộc |
|------|-----------|---------|----------------------|---------------------|
| DSA-reserching-diary | `git@github.com:NguyenPhan161206/DSA-reserching-diary.git` | DSA code + solution write-up + practice log | Anchor + roadmap + tư duy thuật toán (link) | Diary → DSA (Section K block trong từng note liên quan) |
| UMT-Truck-Blind-Sight | `git@github.com:NguyenPhan161206/UMT-Truck-Blind-Sight.git` | Code thiết bị IoT | Kiến thức + bài học rút ra | Diary → repo (ở note `iot_aiot/`) |
| Snake_ANN | `git@github.com:NguyenPhan161206/Snake_ANN.git` | Code ML | Kiến thức ML (`ai_ml/`) | Diary → repo |
| Joblink | `git@github.com:qminh77/joblink.git` | Code full-stack | Kiến thức web/DB/arch | Diary → repo |

**Format chuẩn (copy nguyên văn, không tự diễn đạt lại):**

Trong note ở repo này (header):
```markdown
**Code repository:** [NguyenPhan161206/DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) · `git@github.com:NguyenPhan161206/DSA-reserching-diary.git`
```

Trong `## 🔗 Related Topics`:
```markdown
- 🔗 [DSA-reserching-diary](https://github.com/NguyenPhan161206/DSA-reserching-diary) — code & explanations live there; this diary holds concept anchors and roadmap
```

Trong repo đích (README + từng note liên quan):
```markdown
**Knowledge base:** [reaserching-diary](https://github.com/NguyenPhan161206/reaserching-diary) — theory, roadmap & technology notes live there
```

**Checklist khi tạo note có liên quan repo ngoài:**
- [ ] Header có dòng `**Code repository:**` đúng format
- [ ] Related Topics có mục `🔗` trỏ về repo đích
- [ ] Folder `dev_foundation/dsa/` chỉ chứa anchor / roadmap / method notes, KHÔNG chứa solution code

## ═══════════════════════════════════════════
## ═══════════════════════════════════════════
## SECTION N: LANGUAGE TRACK POLICY
## ═══════════════════════════════════════════

*(Added 2026-09-28 — config v3.4)*

Mỗi ngôn ngữ thuộc **đúng một track**. Không trộn. Lý do: học 4 ngôn ngữ cùng lúc
là hình thức procrastination đắt nhất — nhìn có vẻ chăm chỉ nhưng không tạo ra kỹ năng nào.

| Track | Ngôn ngữ | Vai trò | Notes ở đâu | Code ở đâu |
|-------|----------|----------|-------------|-----------|
| **DSA** | **Java** | Course language — **mọi bài đều làm bằng Java trước** | `dev_foundation/dsa/02-java-for-dsa.md` | repo DSA: `solution.java` |
| **DSA** | **C++** | Port target — phỏng vấn, contest, tight constant factor | `dev_foundation/dsa/03-cpp-for-dsa.md` | repo DSA: `solution.cpp` |
| **Web** | **JavaScript** | Ngôn ngữ của web track (DOM, event loop, async) | `dev_foundation/web_basics/` | repo Joblink |
| **AI/ML + IoT** | **Python** | Stack ML/IoT (Snake_ANN, UMT-Truck-Blind-Sight) | `ai_ml/`, `iot_aiot/` | repo riêng |

**Rule "attempt → port":** làm bằng Java trước → port sang C++ **từ trí nhớ, không dịch từng dòng** →
ghi vào section **Language Notes** trong `approach.md`. Một bản dịch máy-móc không dạy được gì.

**Tỉ lệ port gợi ý:** 1 trong 5 bài, chỉ khi C++ thật sự có lợi (heap/Dijkstra, number theory, chuẩn bị phỏng vấn).
Không làm mọi bài hai lần từ đầu.

**Cấm dùng nhầm:** KHÔNG đề xuất JavaScript cho bài DSA (không có `PriorityQueue`, stack nông, hidden class),
KHÔNG đề xuất Python cho lộ trình DSA (môn học chấm Java, phỏng vấn C++).

**Agent khi gặp note mới:** kiểm tra note này có nằm đúng track không. Note về thuật toán → `dsa/`.
Note về DOM/browser → `web_basics/`. Note về model/huấn luyện → `ai_ml/`. Trộn 2 track trong 1 note
là lỗi phạm vi — tách nó.

## ═══════════════════════════════════════════
## SECTION M: FOLDER SYSTEM FOR dev_foundation/
## ═══════════════════════════════════════════

*(Added 2026-09-28 — config v3.3)*

`dev_foundation/` là **folder tổng** cho nền tảng kỹ thuật. Sub-folder KHÔNG prefix số
(theo style `books_summaries/05-agile_project_management/*`); thứ tự học nằm trong `dev_foundation/README.md`.

| Sub-folder | Template | Vai trò |
|---|---|---|
| `linux/` | G | Lệnh hằng ngày, mental model hệ thống file + process |
| `docker/` | G + A | Công cụ chạy (G) + mental model orchestration (A) |
| `yaml_config/` | G + A | Cú pháp & dùng trong CI (G) + chọn định dạng config (A) |
| `web_basics/` | A + G | Cơ chế render/DOM/layout (A) + ngôn ngữ JS dùng tay (G) |
| `databases/` | G + A | SQL/engine dùng tay (G) + thiết kế schema & kiến trúc dữ liệu (A) |
| `dsa/` | Anchor | Chỉ roadmap + link. Code & giải thích → repo DSA (Section L) |

**Rule:** mọi note trong `dev_foundation/` khi tạo mới PHẢI có mục trong bảng "Note Index" của `README.md` folder đó.
Backlog chưa viết nằm trong bảng **"Note Backlog"** cùng file.

## ═══════════════════════════════════════════
## SECTION I: RESEARCH AREA MAPPING
## ═══════════════════════════════════════════

Maintained in: RESEARCH_AREAS.md

| Folder | Domain | Template | Status | Notes Count | Last Updated |
|--------|--------|----------|--------|-------------|-------------|
| ai_ml/ | AI/ML/DL | A (Tech) | Active | 6 | 2026-09-23 |
| iot_aiot/ | IoT/AIoT | A (Tech) | Active | 2 | 2026-09-07 |
| economics_business/ | Economics | B (Business) | Active | 1 | 2026-09-04 |
| psychology_society/ | Psychology | C (Social) | Reserved | 0 | — |
| books_summaries/ | Books | D (Book) | Active | 1 | 2026-09-09 |
| personal_evaluations/ | Self-eval | E (Eval) | Active | 1 | 2026-09-09 |
| ielts_training/listening/ | IELTS Listening | F (IELTS) | Active | 1 | 2026-09-14 |
| git_github/ | Git/GitHub | A (Tech) | Active | 10 | 2026-09-15 |
| cv/ | CV / Resume | LaTeX CV | Active | 2 | 2026-09-15 |
| vocabulary/ | Vocabulary | V (Vocabulary) | Active | 1 | 2026-09-19 |
| random_ideas/ | Ideas | Flexible | Active | 6 | 2026-09-23 |
| inbox/ | Staging | Flexible | Active | 1 | 2026-09-07 |
| software_architecture/ | SW Architecture | A (Tech) | Active | 2 | 2026-09-23 |
| dev_foundation/ | Engineering Fundamentals | Guide (umbrella) | Active | 0 | 2026-09-28 |
| dev_foundation/linux/ | Linux shell & filesystem | G (Dev Tool) | Active | 2 | 2026-09-28 |
| dev_foundation/docker/ | Docker & Containers | G + A | Planned | 0 | 2026-09-28 |
| dev_foundation/yaml_config/ | YAML & Config Formats | G + A | Planned | 0 | 2026-09-28 |
| dev_foundation/web_basics/ | HTML/CSS/JS/DOM | A + G | Planned | 0 | 2026-09-28 |
| dev_foundation/databases/ | SQL/PostgreSQL/MySQL/PostgREST | G + A | Planned | 0 | 2026-09-28 |
| dev_foundation/dsa/ | DSA anchor, roadmap & method notes | A (Tech) + Flexible (anchor) | Active | 5 | 2026-09-28 |

## ═══════════════════════════════════════════
## SECTION J: FOLDER→FILE NAMING CONVENTION
## ═══════════════════════════════════════════

- Files: `XX-descriptive-name.md` (XX = 01, 02, 03... chronological)
- Snake_case cho tên file và folder
- Prefix number thể hiện thứ tự học/đọc
- Connection notes: `bridge-topic-a-and-topic-b.md`

---
*Config version: 3.5 — Last updated: 2026-09-28*
*(v3.5: DSA is no longer "anchor only" — `dev_foundation/dsa/04-complexity-analysis-toolkit.md` and `05-data-structure-selection-guide.md` are Template A method notes, so Section C now reads "anchor + method notes (A)" and Section I counts 5 notes. Section L's checklist wording updated to match: the folder holds anchor / roadmap / method notes and still must not contain solution code. Resolves the Open Question logged in `dev_foundation/README.md`.)*
*(v3.4: added Section N "Language Track Policy" — DSA track = Java (course language) + C++ (port target); web track = JavaScript; AI/ML track = Python. JavaScript is explicitly excluded from DSA, Python explicitly excluded from DSA. Added `dev_foundation/dsa/02-java-for-dsa.md` and `03-cpp-for-dsa.md`.)*
*(v3.3: added `dev_foundation/` umbrella folder (linux, docker, yaml_config, web_basics, databases, dsa) + TEMPLATE G "Dev Tool & Command Note" + Section K "Cross-Technology Mention Rule" + Section L "Cross-Repo Link Registry" + Section M "Folder System for dev_foundation". DSA code/explanations live in a separate repo; this repo only anchors + roadmaps it.)*
*(v3.2: Section G rewritten to "Push on Command Only" — the v3.1 vocabulary-lookup auto-push standing authorization was revoked by the user the same day. Push now happens only on an explicit user command, batched with everything new.)*
*(v3.1: added Section G "Standing Authorization — Vocabulary Lookup Auto-Push" — revoked)*