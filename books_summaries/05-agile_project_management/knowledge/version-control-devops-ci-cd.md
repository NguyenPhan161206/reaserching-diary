# Version Control, CI/CD & DevOps in Agile

**Created Date:** 2026-09-14
**Source:** External reference — Agile source code management combined with DevOps/CI-CD practices (bổ sung ngoài cuốn *Agile Project Management For Dummies*)
**Tags:** `#VersionControl`, `#Git`, `#BranchingStrategy`, `#ContinuousIntegration`, `#ContinuousDelivery`, `#AutomatedTesting`, `#DevOps`
**Status:** Published
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- **Version control trong Agile ≠ chỉ lưu phiên bản.** Nó gắn liền với chu trình phát triển **lặp và tăng dần** (iterative & incremental), thường hợp nhất với mô hình **CI/CD + DevOps**.
- **Work nhỏ → branch ngắn hạn:** feature lớn bị bẻ nhỏ thành User Story / task độc lập, mỗi cái tương ứng 1 branch ngắn; `main` luôn giữ trạng thái "chạy được".
- **Tích hợp mỗi ngày thay vì mỗi tuần:** PR/MR nhỏ, thường xuyên giúp phát hiện **xung đột sớm**, tránh "integration hell".
- **Automated testing như quality gate:** mỗi commit/merge kích hoạt pipeline (unit test + lint + security check) đảm bảo code là *working software*.
- **Continuous Delivery + Feedback loop:** code vượt kiểm thử → sẵn sàng lên staging/production → feedback từ user/PO → hiệu chỉnh chu kỳ sau.

---

## 🧠 Detailed Notes

### 0. Bối cảnh: tại sao version control phải gắn với DevOps/CI-CD?

Trong Agile, "done" nghĩa là **chạy được, được kiểm thử, sẵn sàng bàn giao** (Definition of Done). Vì Sprint thường 1–4 tuần, team không thể chờ tới cuối Sprint mới tích hợp code — nếu vậy, 2 tuần code cô lập sẽ sản sinh ra hàng trăm conflict. Vì thế quản lý mã nguồn phải được thiết kế theo **vòng lặp nhỏ** giống chính vòng lặp Sprint:

```
Product Backlog ──► Sprint ──► User Story ──► short-lived branch
                                              │
                                              ▼
                                    commit / git push (thường xuyên, nhỏ)
                                              │
                                              ▼
                              PR/MR ──► CI pipeline (build + test + lint + security)
                                              │ pass
                                              ▼
                                     CD: staging ▼ production
                                              │
                                              ▼
                                 Feedback (user / Product Owner)
                                              │
                                              └────► backlog điều chỉnh (next Sprint)
```

**Tại sao quan trọng:** Git và CI/CD là "bộ khung hạ tầng" hiện thực hóa các giá trị Agile — *working software over comprehensive documentation* và *responding to change*. Đọc quy trình này giúp bạn nhìn thấy Agile **chạy thật trên repo**, không chỉ trên slide lý thuyết.

---

### 1. Backlog & Sprints → Phân rã work nhỏ thành branch ngắn hạn

**Nguyên lý:** Tính năng lớn (Epic) bị chia nhỏ thành **User Story / task độc lập**; mỗi story được nhận một **branch ngắn hạn** trong repository. Branch này có vòng đời tính bằng **giờ/ngày**, không phải tuần.

```bash
# 1 story = 1 branch, tên gói đủ ngữ cảnh (JIRA/issue id + mô tả)
git checkout -b feature/USERSTORY-123-login-refactor
# ... code, commit nhỏ nhiều lần trong ngày
git commit -m "add password validation for login form"
git commit -m "extract auth helper to reduce duplication"
# sau khi PR merge → xóa branch, main luôn sạch
git checkout main && git pull && git branch -d feature/USERSTORY-123-login-refactor
```

| Đặc điểm | Branch dài hạn (Waterfall/docs cũ) | Branch ngắn hạn (Agile/Trunk-based) |
|---|---|---|
| Vòng đời | Tuần → tháng | Giờ → vài ngày |
| Độ lớn | Nhiều feature lộn xộn | Đúng 1 story |
| Tương quan backlog | Mơ hồ | 1 branch ↔ 1 backlog item |
| Merge | Lũy tiến, dễ conflict | Thường xuyên, small diff |

**Tại sao quan trọng:** branch ngắn + nhỏ = **code review nhanh, conflict hiếm, rollback dễ** (revert 1 PR là xong). Đồng thời nó ép team phải *thực sự* chia nhỏ công việc (phân rã backlog), đúng tinh thần iterative & incremental.

---

### 2. Continuous Integration (CI) → PR/MR thường xuyên

**Nguyên lý:** Thay vì giữ code trên nhánh riêng nhiều tuần rồi mới merge (nhưng nhiều dev thích làm vậy vì "chưa xong đã merge có gì hớ"), lập trình viên **thường xuyên tạo Pull Request / Merge Request** đưa code về nhánh chung **mỗi ngày** để phát hiện xung đột sớm.

**Vấn đề CI giải quyết (integration hell):**
```
Dev A code 3 tuần trên file_a.py
Dev B code 3 tuần trên file_b.py
→ Ngày merge: 500 conflict, test vỡ loạn, không ai biết ai làm hỏng
```

**Cách Agile làm:**
```
Mỗi commit nhỏ → CI server auto: build + test → báo ngay
→ lỗi được phát hiện TRONG VÀI GIỜ, không phải sau 3 tuần
```

**PR tốt trong Agile:**
- Diff nhỏ (1 story, vài trăm dòng), review nhanh ≤ 1 ngày
- Tự động chạy pipeline ngay trên PR (không chỉ sau khi merge)
- Ghi rõ link backlog item + screenshot/mô tả hành vi
- Author tự verify pipeline xanh trước khi gọi reviewer

**Tại sao quan trọng:** tích hợp mỗi ngày = **phát hiện xung đột/code hỏng sớm nhất có thể** (càng phát hiện muộn chi phí sửa càng cao — Lean/defect cost curve). Đây cũng là tiền đề để team "luôn có bản chạy được", phục vụ Sprint Demo mỗi cuối Sprint.

---

### 3. Automated Testing → pipeline như quality gate

**Nguyên lý:** Mỗi lần commit/merge đều kích hoạt **pipeline kiểm tra chất lượng tự động**: unit test, linting, security check… Mục tiêu không phải "test giỏi" mà là giữ code **luôn ở trạng thái sẵn sàng chạy được** (working software) — yêu cầu của Definition of Done.

**Pipeline điển hình (GitHub Actions):**
```yaml
name: ci
on:
  pull_request:            # chạy ngay trên PR + mỗi lần push lên nhánh
  push:
    branches: [main]
jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pip install -r requirements-dev.txt
      - run: ruff check .            # lint (static analysis)
      - run: pytest tests/ --cov     # unit test + coverage
      - run: bandit -r src/          # security scan
      - run: docker build .          # build image = kiểm tra "dựng được"
```

**Chú ý theo thứ tự gate:** nếu bất kỳ bước nào **đỏ**, PR **không được merge**. Đó là *quality gate* — không phải "khuyến nghị".

**Tại sao quan trọng:** testing thủ công không scale; tự động hóa biến việc "code luôn ổn" thành **quy trình cơ học**, giúp team:
- Tự tin merge cả ngày, không sợ "vỡ main".
- Bắt lỗi hồi quy ngay khi nó xuất hiện (regression).
- Bước đệm bắt buộc trước khi CD — **CI xanh thì mới dám deploy**.

---

### 4. Continuous Delivery & Feedback Loop → bàn giao nhanh để điều chỉnh

**Nguyên lý:** Code sau khi vượt qua kiểm thử sẽ **sẵn sàng triển khai** lên môi trường **staging/production** để nhận phản hồi từ người dùng hoặc **Product Owner**, từ đó điều chỉnh kế hoạch cho chu kỳ tiếp theo.

| Khái niệm | Ý nghĩa |
|---|---|
| **Continuous Delivery (CD)** | Mã luôn ở trạng thái *sẵn sàng deploy*; việc bấm nút deploy là lựa chọn của team/business |
| **Continuous Deployment** | Cấp độ mạnh hơn: PR hợp vào main → tự động deploy production, không cần bấm nút |
| **Feedback loop** | Phản hồi (metrics, user, PO) quay về Product Backlog → điều chỉnh ưu tiên Sprint sau |

**Mô hình triển khai layer:**
```
local → CI (build+test) → staging (demo, PO, user test) → production (canary / blue-green) → metrics feedback
```

**Ví dụ concreet:** team phát hành feature "dark mode" lên **staging**, PO xem và nói "màu đối lập chưa đủ tương phản" → team tạo backlog item mới (điều chỉnh contrast) điều chỉnh trong Sprint kế tiếp, *trước khi* ảnh hưởng tới production user. Nếu không có staging/CD, lỗi UX này chỉ lộ ra cho hàng triệu user.

**Tại sao quan trọng:** CD là cơ chế hiện thực hóa **"respond to change" + inspect & adapt**. Chu kỳ feedback càng ngắn, team càng sớm biết mình "xây đúng thứ cần xây" — giảm rủi ro xây nhầm (lớn nhất trong dự án phần mềm, xem Part 4 / Chương 17: risk mitigation bằng small increments).

---

### 5. Tóm tắt: 4 phần gắn với vòng lặp Agile

| # | Bước Agile | Công cụ | Mục đích cốt lõi |
|---|---|---|---|
| 1 | Backlog & Sprint | User Story → branch ngắn | Phân rã work, 1 nhánh = 1 đơn vị có thể bàn giao |
| 2 | CI | PR/MR hàng ngày | Phát hiện xung đột sớm, main luôn xanh |
| 3 | Automated Testing | Pipeline (test+lint+security) | Giữ code luôn *working software* |
| 4 | CD & Feedback | Staging → Production → backlog | Học từ user/PO nhanh, điều chỉnh chu kỳ sau |

---

## 📝 Research Journey

- **Why:** Học Agile lý thuyết dễ bị "khô" — tôi cần thấy nó vận hành trên repo thật. Học version control + DevOps giúp nối cầu giữa project management và kỹ thuật code mà sau này phải dùng hằng ngày khi làm ML Engineer.
- **Struggle:** Ban đầu tôi nhầm "branch lâu đời + merge cuối kỳ" là bình thường. Chỉ khi hiểu defect cost curve + integration hell mới thấy tại sao phải tích hợp mỗi ngày dù rất mất công quản lý conflict nhỏ vặt.
- **Aha moment:** CI/CD không phải "tool DevOps", mà là **hiện thân kỹ thuật của Definition of Done** — "chạy được, được kiểm thử, sẵn sàng bàn giao" nếu không có pipeline thì chỉ là lời nói suông.
- **Career link:** AI/ML cũng dùng đúng mô hình này — data/model pipeline trong CI (smoke test, drift check) chính là gate trước khi deploy model. Nắm vững Git+CI/CD là điều kiện tiên quyết của MLOps.

## 🔗 Related Topics

- [[../notes/04-permanent-team-tuckman-decision-timing.md]] — permanent team + last responsible moment: lý do cần branch ngắn (đổi hướng nhanh, không bị cam kết sớm)
- [[../../../ai_ml/04-xp-engineering-practices-for-ml.md]] — TDD + CI cho ML: test data/model thay vì chỉ unit test
- [[part-4-agility-management.md]] — Chương 17: Quality & Risk — Automated Testing + CI như built-in quality
- [[part-2-being-agile.md]] — XP (engineering practices) trong Big Three frameworks
- [SUGGESTED] **Branching strategy deep-dive (Git Flow vs Trunk-based vs GitHub Flow)** — so sánh chi tiết, chọn cho team size khác nhau — nên nằm trong `knowledge/`
- [SUGGESTED] **MLOps end-to-end note** — data versioning (DVC), model registry, deploy model — nên nằm trong `ai_ml/`
- [SUGGESTED] **Definition of Done cấp repository** — checklist mà pipeline nên bắt buộc (test pass, coverage %, no-secrets scan, doc) — nên nằm trong `knowledge/`

## 🤔 Open Questions

- [ ] Trunk-based hay feature-branch (Git Flow) phù hợp team ML nhỏ? Cân bằng "review quality" và "speed of delivery"?
- [ ] Làm sao giữ CI pipeline nhanh khi test tăng (test split, caching, remote execution)?
- [ ] Với model ML, "test hình thức" như test unit code có đủ chưa — cần thêm gates gì cho data drift / fairness?
- [ ] Feedback loop từ production (metrics, logs, user behavior) đo lường thế nào để cải thiện backlog Sprint sau một cách có dữ liệu?

---

*Backlog là gốc. Pipeline là sợi dây chuyền. Feedback là vòng lặp.*