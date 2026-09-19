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
| Agile — Knowledge | books_summaries/agile_project_management/knowledge/ | Template D (Book) |
| Agile — Q&A Notes | books_summaries/agile_project_management/notes/ | Q&A custom (xem notes/README.md) |
| Agile — Practice | books_summaries/agile_project_management/practice/ | Practice custom (xem practice/README.md) |
| IELTS — Listening | ielts_training/listening/ | Template F: IELTS Listening Practice |
| Git & GitHub | git_github/ | Template A: Tech Note (adapted) |
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

### Push Conditions (tất cả phải thỏa):
- [ ] 3+ notes mới đã format đúng template
- [ ] CONTRIBUTION_CALENDAR.md đã cập nhật
- [ ] Files đã move từ inbox/ (nếu có)
- [ ] Cross-references đã thêm (nếu có)

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

## ═══════════════════════════════════════════
## SECTION I: RESEARCH AREA MAPPING
## ═══════════════════════════════════════════

Maintained in: RESEARCH_AREAS.md

| Folder | Domain | Template | Status | Notes Count | Last Updated |
|--------|--------|----------|--------|-------------|-------------|
| ai_ml/ | AI/ML/DL | A (Tech) | Active | 3 | 2026-09-07 |
| iot_aiot/ | IoT/AIoT | A (Tech) | Active | 2 | 2026-09-07 |
| economics_business/ | Economics | B (Business) | Active | 1 | 2026-09-04 |
| psychology_society/ | Psychology | C (Social) | Reserved | 0 | — |
| books_summaries/ | Books | D (Book) | Active | 1 | 2026-09-09 |
| personal_evaluations/ | Self-eval | E (Eval) | Active | 1 | 2026-09-09 |
| ielts_training/listening/ | IELTS Listening | F (IELTS) | Active | 1 | 2026-09-14 |
| git_github/ | Git/GitHub | A (Tech) | Active | 10 | 2026-09-15 |
| cv/ | CV / Resume | LaTeX CV | Active | 2 | 2026-09-15 |
| random_ideas/ | Ideas | Flexible | Reserved | 0 | — |
| inbox/ | Staging | Flexible | Active | 1 | 2026-09-07 |

## ═══════════════════════════════════════════
## SECTION J: FOLDER→FILE NAMING CONVENTION
## ═══════════════════════════════════════════

- Files: `XX-descriptive-name.md` (XX = 01, 02, 03... chronological)
- Snake_case cho tên file và folder
- Prefix number thể hiện thứ tự học/đọc
- Connection notes: `bridge-topic-a-and-topic-b.md`

---
*Config version: 3.0 — Last updated: 2026-09-14*
