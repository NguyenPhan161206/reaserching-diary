# Phiên 02 — Ceremony, Gantt & Core Concepts

**Ngày:** 2026-09-12
**Nguồn kiến thức:** [[../knowledge/part-1-understanding-agility.md]], [[../knowledge/part-3-planning-and-execution.md]]
**Tags:** `#Ceremony`, `#GanttChart`, `#Standup`, `#Retrospective`, `#AgileManifesto`

> 📌 Đây là phiên Q&A đã diễn ra trong chat và bị mất do Ctrl+C — nội dung được khôi phục lại từ database opencode.

---

## ❓ Câu hỏi 1: Ceremony là gì?

**Ceremony** = các cuộc họp/ghi nhận *có cấu trúc + có thời gian giới hạn (timeboxed)* mà team Agile lặp lại theo nhịp Sprint. Từ "ceremony" (nghi thức) nghe trang trọng nhưng chỉ có nghĩa: **những thời điểm cố định mà cả team quy tụ để đồng bộ và cải tiến**.

### 📚 Trả lời chi tiết

4 ceremony chính của Scrum:
| Ceremony | Mục đích | Timebox |
|---|---|---|
| Sprint Planning | Chọn User Stories cho sprint tới | ~ vài giờ |
| Daily Standup | Đồng bộ 24h tới | **15 phút** |
| Sprint Review | Demo sản phẩm cho stakeholder | ~ 1-2 giờ |
| Sprint Retrospective | Cải tiến process | ~ 1 giờ |

Quan trọng: ceremony **không phải rào cản hành chính** — chúng là cơ chế "inspect & adapt" định kỳ. Nếu team họp mà không cải tiến được gì → đó là ceremony rỗng (dấu hiệu Fake Agile #10).

---

## ❓ Câu hỏi 2: Báo cáo bằng Gantt là gì? Tại sao phải dùng? Lợi thế & khuyết điểm?

### 📚 Trả lời chi tiết

#### Là gì
**Gantt chart** = biểu đồ ngang (bar chart) biểu diễn **lịch trình dự án**: mỗi task là 1 thanh nằm ngang, độ dài thanh = thời gian dự kiến, trục ngang = thời gian, các thanh nối nhau thể hiện dependency (task B chỉ bắt đầu khi task A xong).

```
Task A ████████████░░░░░░
Task B ░░░░░████████████░  ← phụ thuộc A
Task C ░░░░░░░██████░░░░░
        Jan   Feb   Mar
```

#### Làm như thế nào (cách dựng)
1. Liệt kê tất cả tasks (Work Breakdown Structure)
2. Ước lượng thời gian mỗi task (thường theo *giờ tuyệt đối*)
3. Xác định thứ tự + dependencies
4. Vẽ thanh trên trục thời gian
5. Theo dõi bằng % hoàn thành (task "90% xong" mà 3 tuần vẫn 90% 😅)

#### Tại sao người ta PHẢI dùng nó? (gốc rễ lịch sử)
Gantt ra đời ~1910s, phổ biến rộng trong **chiến tranh thế giới & xây dựng** (đập Hoover, tòa nhà) — nơi:
- Yêu cầu **biết trước và cố định** (bản vẽ có sẵn)
- Chi phí làm lại cực đắt
- Đội ngũ lớn, làm theo lệnh cấp trên

→ Trong thế giới đó, "lên kế hoạch chuẩn từ đầu rồi thi công đúng kế hoạch" là tối ưu. Gantt sinh ra để phục vụ tư duy đó. Người ta vẫn dùng nó vì: **ai cũng hiểu, trực quan, giúp cam kết deadline với khách hàng.** Nhưng nó giả định rằng **thế giới không đổi trong suốt dự án** — đúng với bê-tông, sai với phần mềm.

#### Lợi thế
- Trực quan, dễ đọc, dễ trình bày cho ban lãnh đạo
- Thấy dependency rõ ràng → phát hiện task chặn
- Tốt cho kiểu công việc vật lý/lặp lại

#### Khuyết điểm (với phần mềm)
1. **% hoàn thành là ảo** — "task 90% xong" không cho biết thật sự còn bao nhiêu việc (phần mềm thì 90% hiển thị nhưng 90% logic ẩn chưa đụng)
2. **Giả định thay đổi = thảm họa** — nhưng trong dev, thay đổi là chuyện thường
3. **Khuyến khích báo cáo sai** — ai cũng sợ báo chậm nên "hợp thức hóa" % cho đẹp
4. **Không đo được giá trị** — chỉ đo "đã làm bao nhiêu việc", không đo "user có hài lòng không"
5. **Chi phí bảo trì cao** — mỗi lần đổi lịch là vẽ lại cả bảng

> **Tóm lại:** Gantt đo **output** (đã làm xong bao nhiêu task), còn Agile đo **outcome** (task đó có tạo giá trị không). Đó là lý do Agile thay thế "báo cáo Gantt" bằng *Information Radiators* (bảng Kanban, burndown chart) — cập nhật bởi chính team, không cần họp báo cáo.

---

## ❓ Câu hỏi 3: Standup và Retro là gì?

### 📚 Trả lời chi tiết

#### Daily Standup (Daily Scrum)
- **Họp 15 phút mỗi sáng, đứng họp** (đứng để... nhanh chóng hết giờ!). Chỉ dành cho Development Team.
- **3 câu hỏi cố định:** Hôm qua làm gì? Hôm nay sẽ làm gì? Có gì cản trở (impediment)?
- **Bản chất:** không phải báo cáo cho manager — là **micro-planning 24h tới**. Tìm ra "có ai đang bị kẹt không, team cần giúp gì".
- ⚠️ Nếu thành "báo cáo tis tá cho sếp" → ceremony rỗng.

#### Retrospective (Sprint Retro)
- **Họp cuối Sprint (~1 giờ)**, cả team nhìn lại **cách làm việc** (không phải sản phẩm — đó là việc của Sprint Review).
- Framework quen thuộc: **Start / Stop / Continue** — "bắt đầu làm gì mới / ngừng làm gì / tiếp tục làm gì".
- Là **ceremony quan trọng nhất** vì là nơi duy nhất team trực tiếp cải tiến process của chính mình, lặp lại mỗi sprint → vòng lặp cải tiến liên tục.
- 📌 Học sâu hơn: `knowledge/part-3-planning-and-execution.md` Chapter 12.

---

## ❓ Câu hỏi 4: "Sự khác biệt Project vs Product có nói lên sự khác biệt của manager 2 kiểu đó không?"

### 📚 Trả lời chi tiết

**Đúng, và đó là điểm mấu chốt** — nhưng cần tinh chỉnh: không phải "manager khác kiểu người", mà là **vai trò & văn hóa quản lý khác nhau**:

| | Project Manager (Waterfall) | Product-minded leader (Agile) |
|---|---|---|
| Câu hỏi hằng ngày | "Còn đúng deadline không? % xong?" | "Chúng ta đang giải quyết đúng vấn đề không?" |
| Với team | Command & control: giao việc, kiểm tra, đôn đốc | Servant-leader: dọn chướng ngại, coach, bảo vệ team khỏi xao nhãng |
| Với thay đổi | Cầu xin đừng đổi (đổi = phá kế hoạch) | Chào đón, đưa vào backlog ưu tiên lại |
| Đo lường | Output: task, giờ, Gantt % | Outcome: value user nhận, velocity, working software |
| Quan hệ khách | "Hợp đồng là thánh chỉ, đổi = kiện" | "Hợp tác, cùng học, cùng thích nghi" |
| Team | Tạm thời, mượn người từ phòng ban | Đội cố định lâu dài, tự tổ chức |

> Ví dụ: Project Manager kiểu Waterfall nhìn "user đổi requirement" = tin xấu cần chặn. Servant-leader nhìn đó = **tín hiệu thị trường miễn phí**, càng sớm phát hiện càng rẻ. Nói gọn: *Project manager quản lý kế hoạch; Agile leader quản lý vấn đề của user.*

---

## ❓ Câu hỏi 5: Giải thích kỹ 4 Values (Phần 3)

### 📚 Trả lời chi tiết

Ý đồ của manifesto: **khi xung đột, ưu tiên vế trái, KHÔNG vứt vế phải.**

**① Individuals & interactions > Processes & tools**
- *Nghĩa:* Con người và trao đổi trực tiếp quan trọng hơn việc áp giáo trình quy trình & công cụ.
- *Ví dụ:* Một team có Jira tối tân + 200 trang SOP nhưng không nói chuyện với nhau → chết. Team không có tool nhưng đồng nghiệp ngồi cạnh nhau trao đổi mỗi giờ → sống.
- *Tại sao:* Công cụ là phương tiện; "tool triệu đô" không sửa được văn hóa.
- *Liên hệ AI:* Mua GPU đắt tiền/autoML không thay được việc 2 kỹ sư trao đổi về dữ liệu.

**② Working software > Comprehensive documentation**
- *Nghĩa:* Sản phẩm chạy được phải được ưu tiên hơn tài liệu "đầy đủ".
- *Ví dụ:* Khách hàng vui vì thấy bản demo chạy thật sau 2 tuần, còn hơn nhận 300 trang spec sau 6 tháng.
- *Tại sao:* Docs lỗi thời ngay khi viết xong (vì code đổi); working software là "sự thật duy nhất" (single source of truth).
- *Lưu ý:* Vẫn cần docs tối thiểu (README, API doc) — value 2 không cấm docs, chỉ cấm docs làm *lý do* trì hoãn giao hàng.

**③ Customer collaboration > Contract negotiation**
- *Nghĩa:* Thay vì ngồi "chốt điều khoản" trong hợp đồng để sau này bám vào mà đòi nhau, hãy ngồi chung bàn với khách liên tục.
- *Ví dụ:* Hợp đồng truyền thống: "Xong đúng scope = OK, đổi scope = phạt". Hợp tác Agile: khách ngồi trong Sprint Review, thấy sản phẩm, góp ý trực tiếp → cả 2 bên cùng thắng.
- *Tại sao:* Trong sản phẩm phần mềm, khách **không thể biết họ cần gì trước khi thấy bản chạy**. Bám hợp đồng = đảm bảo cả 2 cùng thua.

**④ Responding to change > Following a plan**
- *Nghĩa:* Kế hoạch có giá trị nhưng khả năng phản ứng với thay đổi có giá trị hơn.
- *Ví dụ:* Kế hoạch 6 tháng xây model X. Tháng 2 phát hiện dữ liệu không đủ cho X nhưng cực tốt cho Y (nhỏ hơn, rẻ hơn, user cần hơn). Agile: pivot sang Y. Waterfall: "kế hoạch ghi X, làm X".
- *Tại sao:* Kế hoạch dựa trên giả định; giả định sai → bám kế hoạch = đi sai có hệ thống. "Plans are worthless, but planning is invaluable." — Eisenhower.

---

## ❓ Câu hỏi 6: Hypothesis là gì?

### 📚 Trả lời chi tiết

**Hypothesis (giả thuyết)** = một **khẳng định có thể kiểm chứng** về mối quan hệ giữa hành động và kết quả, viết dưới dạng "Nếu... thì... bởi vì...". Trong Agile, nó biến product development thành khoa học:

```
If   [ta xây tính năng X cho nhóm user Y]
Then [chỉ số Z sẽ thay đổi (tăng/giảm)]
Because [lý do ta tin điều đó]
```

**Ví dụ AI cụ thể:**
> "IF chúng ta thêm nút 'Tóm tắt tự động' cho sinh viên đang ôn thi, THEN tỉ lệ quay lại dùng app tăng 20%, BECAUSE họ đang tốn nhiều thời gian đọc tài liệu dài."

- Điểm chí mạng: hypothesis **phải đo được** (chỉ số Z định lượng) và **phải có ngày hết hạn** ("sau 1 sprint, đo…"). Không đo được = không phải hypothesis, là mong ước.
- Kết quả thử nghiệm: **xác nhận** (thật, scale lên) hoặc **bác bỏ** (học được điều quý, pivot). Cả 2 đều là thắng — vì bạn đã tránh được việc đổ thêm $ vào thứ vô dụng.
- Đây chính là MVP (Minimum Viable Product): build đủ nhỏ để *test giả thuyết*, không phải build đủ lớn để "gây ấn tượng".

---

## ❓ Câu hỏi 7: 12 Principles ở đâu trong nội dung sách / thư mục?

### 📚 Trả lời chi tiết

**Có trong sách** — cuốn *Agile Project Management For Dummies* trình bày 12 Principles trong **Chapter 2 của Part 1**.

Trong thư mục của bạn:
- **`knowledge/part-1-understanding-agility.md`** → Chapter 2: đoạn tóm tắt ngắn: *"The 12 Principles: Emphasize early delivery, welcoming changing requirements, daily business-developer cooperation, and self-organizing teams."* (Chỉ là bản tóm lược 1 dòng).
- **`notes/01-agility-mindset-and-manifesto.md`** → gom 12 nguyên tắc thành 4 cụm để dễ nhớ (số nguyên tắc ghi kèm từng cụm).

**Nguyên văn đầy đủ 12 principles** (để bạn đối chiếu — nằm trong Agile Manifesto 2001):

1. Ưu tiên cao nhất: thỏa mãn khách hàng qua giao phần mềm **sớm và liên tục**.
2. **Chào đón yêu cầu thay đổi**, kể cả trễ trong dự án — thay đổi là lợi thế cạnh tranh.
3. Giao phần mềm chạy được **thường xuyên**, từ 2 tuần đến 2 tháng.
4. **Business và developers làm việc cùng nhau hằng ngày** suốt dự án.
5. Xây dựng dự án quanh những **cá nhân được động viên** — tin họ, tạo môi trường và hỗ trợ họ.
6. **Face-to-face** là phương thức truyền đạt hiệu quả nhất.
7. **Working software** là thước đo tiến độ chính.
8. Agile thúc đẩy **phát triển bền vững** — duy trì nhịp độ đều đặn vô hạn.
9. Quan tâm liên tục đến **kỹ thuật xuất sắc và thiết kế tốt** giúp tăng agility.
10. **Simplicity** — nghệ thuật tối đa hóa lượng việc *không làm* — là thiết yếu.
11. Kiến trúc, yêu cầu và thiết kế tốt nhất đến từ **team tự tổ chức**.
12. Team phản ánh **cách làm việc của mình đều đặn** và điều chỉnh hành vi cho hiệu quả hơn.

> Mẹo học: đừng học thuộc 12 câu; hãy nhớ 4 nhóm + nhớ rằng *nguyên tắc 7 và 8 đối trực tiếp với 2 tội ác Waterfall: đo progress bằng "%, giờ" và ép team chạy nước rút tới kiệt sức.*

---

## 💡 Tổng kết phiên

- [x] Phân biệt được ceremony thật vs ceremony rỗng ("inspect & adapt")
- [x] Hiểu Gantt chart: lợi thế (trực quan, dependency), khuyết điểm với software (%, giả định tĩnh, báo cáo sai)
- [x] Standup = micro-planning 24h, Retro = nơi team cải tiến process (Start/Stop/Continue)
- [x] Project Manager = quản lý kế hoạch; Agile leader = quản lý vấn đề user
- [x] Giải thích sâu 4 values của Manifesto
- [x] Nắm framework "If-Then-Because" của Hypothesis + liên hệ MVP
- [x] Nhận diện vị trí 12 Principles trong sách + thư mục

## 🔄 Câu hỏi tiếp theo đề xuất

1. [SUGGESTED] **Burndown chart vs Burnup chart** — 2 information radiator thay thế Gantt, cách đọc để biết team có "đang đi đúng hướng" không.
2. [SUGGESTED] **Definition of Done (DoD)** — vì phần 3 nhắc "Done means Done" nhưng chưa giải thích vì sao thiếu DoD là nguồn gốc technical debt.
3. [SUGGESTED] **Waterfall vs Agile đấu trực diện trên cùng 1 ví dụ** — để thấy đúng lúc nào nên dùng cái nào (Agile không phải luôn đúng!).

---

## 🔗 Related Topics

- [[01-agility-mindset-and-manifesto.md]] — Phiên 01: mindset, 4 values, 12 principles (4 cụm)
- [[03-scrum-framework-and-four-ceremonies.md]] — Phiên 03: Scrum, status report, 4 ceremonies chi tiết
- [[../knowledge/part-1-understanding-agility.md]]
- [[../knowledge/part-3-planning-and-execution.md]]
- [[README.md]] — index các phiên Q&A

---
*Học bằng cách hỏi đúng câu hỏi.*