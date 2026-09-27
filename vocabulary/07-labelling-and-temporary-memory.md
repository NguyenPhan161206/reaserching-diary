# Vocabulary & Collocation — **LABEL** (nhãn) & **SCRATCHPAD** (bộ nhớ tạm)

**Created Date:** 2026-09-27
**Last Updated:** 2026-09-27
**Tags:** `#vocabulary`, `#collocations`, `#mlops`, `#data-labelling`, `#memory-hierarchy`, `#ui-ux`, `#viet-glish`
**Status:** Active | Reviewed
**Template:** Template V — Advanced Vocabulary & Collocation
**Source note:** Vietnamese *nhãn / dán nhãn / gán nhãn* and *bản nháp / vùng nhớ tạm*

---

## 📌 Term List & Quick Reference

| Term | IPA | Core meaning | Domain |
|---|---|---|---|
| **label** (n) | `/ˈleɪ.bəl/` | A tag attached or assigned for identification, categorisation, or attribution | ML / UI / Business / Law |
| **label** (v) | `/ˈleɪ.bəl/` | To attach such a tag; to designate something as a category; (pejorative) to stereotype a person | All |
| **scratchpad** (n) | `/ˈskrætʃ.pæd/` | A temporary writing surface or working buffer, not intended for permanent storage | Computing / Hardware |
| **scratchpad memory** | — | On-chip SRAM used as a short-lived working buffer for computation | Hardware / AI accelerators |
| **tmpfs** | — | An OS mechanism exposing a RAM-backed directory as a filesystem | Linux / DevOps |

> ⚠️ **Spelling:** *scratchpad* — one word, **no `h` after `c`**. Common learner errors: ~~schratchpad~~, ~~scrachpad~~, ~~scratpad~~. The two-word form *scratch pad* is also attested but less common in technical prose.

---

## 🧠 Detailed Vocabulary Analysis

### 1. LABEL — the multi-domain term

**Critical distinction:** a **label is not a brand**. A brand is reputation and identity; a label is the assigned or attached tag.

#### 1.1 Sense map

| Sense | Domain | Vietnamese | Example |
|---|---|---|---|
| **A. Physical tag** | Consumer / Legal | nhãn dán, nhãn hiệu trên bao bì | *Always read the label before taking medicine.* |
| **B. Assigned category** | **ML / Data** | **nhãn (đáp án)** | *The model outputs 10 possible labels.* |
| **C. Retailer brand** | Business | nhãn riêng / nhãn của chuỗi | *It's a store **private label**, not Coca-Cola.* |
| **D. Warning / rating** | Law / Policy | nhãn cảnh báo, nhãn năng lượng | *The EU energy label is mandatory.* |
| **E. Credit / affiliation** | Media | hãng đĩa, nhãn thu âm | *signed to a major record label* |
| **F. Chemical designation** | Chemistry | tên hóa học được dán nhãn | *The substance is **labelled** ethanol.* |
| **G. Verb: designate** | All | gán nhãn, xếp vào loại | *Don't **label** the data as 0 and 1.* |
| **H. Verb: stereotype** ⚠️ | Social | dán nhãn người (tiêu cực) | *Don't **label** him as lazy — he has ADHD.* |

**⚠️ Verb senses G and H are the ones learners confuse.**
- **G (technical, neutral):** *label a record as fraudulent* = ghi nhãn "gian lận" vào bản ghi.
- **H (social, negative):** *label him as lazy* = **gán nhãn/xếp nhãn một con người** — this carries stigma, bias and social harm. Natives choose it deliberately, usually critically.

#### 1.2 Word family

| POS | Form | Example |
|---|---|---|
| Noun | **label** | *a warning label* |
| Verb | **label / labels / labelled (BrE) / labeled (US)** | *Please **label** your files.* |
| Noun (process) | **labelling / labeling** | *Manual **labelling** takes 4 hours per batch.* |
| Adjective | **labelled / labeled** | *a **labelled** dataset* |
| Noun (human role) | **labeller / annotator** | *Contractors act as **labellers**.* |

> **Spelling note:** **labeled** (US) vs **labelled** (UK, Canada, Australia). Both are correct in their regions; the double-`l` rule matches British spelling.

#### 1.3 AI / ML — the sense that matters most for your career

```
Input X (features)  →  Model  →  Predicted output ŷ   ≈   Label y (ground truth)
```

| Term | Vietnamese | Meaning / why it matters |
|---|---|---|
| **label** | nhãn, đáp án | The target value the model must predict |
| **labeled data** | dữ liệu có nhãn | Supervised learning input; expensive |
| **unlabeled data** | dữ liệu chưa gán nhãn | Fuel for unsupervised / semi-supervised learning |
| **ground-truth label** | nhãn chuẩn, nhãn vàng | Human-verified reference value |
| **label noise** | **nhiễu nhãn** | Wrong labels → the model learns the error |
| **label leakage** | **rò rỉ nhãn** | Target-derived feature → inflated accuracy, unusable model |
| **weak labels / weak supervision** | nhãn yếu | Heuristic labels replacing expensive human annotation |
| **pseudo-labels** | nhãn giả | Model-generated labels used for self-training |
| **multi-label classification** | phân loại đa nhãn | One sample can carry several labels (an image is both *cat* and *indoor*) |
| **label shift** | dịch chuyển nhãn | Change in **P(y)** |
| **label distribution** | phân phối nhãn | Class imbalance check before training |
| **data labelling / annotation** | gán nhãn / chú thích dữ liệu | Human pipeline: CVAT, Label Studio, Amazon Mechanical Turk |
| **annotator bias** | thiên lệch người gán nhãn | A measurable source of model bias and unfairness |

**The three distribution shifts — do not confuse them**

| Shift | Changes | Typical cause |
|---|---|---|
| **Covariate shift** | P(x) | New camera, new sensor, new population |
| **Label shift** | P(y) | Class prevalence changes in production |
| **Concept shift** | P(y \| x) | The relationship itself changed (an ML problem) |

**Why this matters:** a model can degrade in production purely because of **label shift** even with zero data change in x. Detect it by comparing production P(y) against training P(y).

#### 1.4 Software / UI register

| Context | Term | Note |
|---|---|---|
| Form control | **label** | `<label for="email">Email</label>` — clickable label bound to the control |
| Chart | **axis label** / **data label** | Trục và nhãn giá trị |
| Map / dashboard | **label** | Nhãn điểm dữ liệu |
| Accessibility | **`aria-label`** | Screen-reader text; often the only accessible name |
| Code / data | **named labels** vs **zero-based indices** | `['cat','dog']` vs `[0,1]` — a real bug source |
| Git | **tag** (❌ not label) | `git tag v1.0` |

#### 1.5 Collocations

- `attach / apply / assign / put on` a label
- `label sth as + noun / gerund` — *label data as sensitive*
- `manually label` vs `automatically label`
- `a mislabeled record` — dán nhãn sai
- `the label reads "…"`, `printed on the label`
- `a warning label` · `a nutrition label` · `a price label` · `a hazard label`
- `a private label / own-brand product` · `a white-label solution`
- `label set` · `label mapping` · `label distribution` · `label schema`
- `sensitive label` / `PII label` — data governance
- `annotated and labeled` — humans and tools combined

#### 1.6 Confusion set

| Term | Core focus | Difference |
|---|---|---|
| **label** | A tag **assigned or attached** for identification | Systematically assigned, implies categorisation |
| **tag** | A looser marker, usually digital | Metadata; less official (blog tags, `git tag`) |
| **class** | A set within a taxonomy | The **group**, not the individual tag |
| **category** | A bucket of items | Usually broader, flatter |
| **name** | What something is called | Identifies, does not classify |
| **sticker** | Adhesive physical object | Material, not semantic |
| **brand** | Reputation and identity | ❌ not a label |
| **annotation** | Human explanation added to data | The *action*; may be richer than a label |

> In ML: **label** = the answer for one sample · **class** = the set of all possible answers.

#### 1.7 Viet-Glish pitfalls

| ❌ Viet-Glish | ✅ Native standard | Why |
|---|---|---|
| *"Gán label 0/1 cho dữ liệu."* | *"**Encode** the data as 0/1"* or *"**assign** the label 0/1"* | **Gán giá trị = encode**, not *label*. *Label* = gán **nghĩa/nhãn** |
| *"Dán label cho file."* | *"**Label your files**"* / "hãn (dán) nhãn cho file" | English verb, no noun-verb mixing |
| *"The label of the model output"* | *"The **label** the model outputs"* | Redundant `of` |
| *"private label" = "thương hiệu"* | *"private label* / *own-brand* vs *brand* are three things" | Retail lexicon has distinct terms |
| *"I don't want to label him as stupid."* | *"I don't want to **stereotype** him / **put a label on** him."* | `label a person` = stigmatising, so avoid casually |
| *"label" for HTML element | *`label`* = display text; *`tag`* = HTML element; *`name` / `id`* = form attributes | Four different concepts |
| *"Vietnamese NLP labeling is very difficult"* ✅ fine | — | — |
| ~~`labelled` / `labeled`~~ | Pick your region's convention and stay consistent | Consistency matters in a repo |

---

### 2. SCRATCHPAD — the temporary working surface

#### 2.1 Sense map across the stack

| Layer | Vietnamese | Example |
|---|---|---|
| **Physical / everyday** | tờ giấy nháp, gạch nháp | *I jotted the number down on a scratchpad.* |
| **Mental** | nhớ trong đầu, ghi tạm trong đầu | *mental scratchpad* |
| **Programming** | file nháp, file tạm xử lý trung gian | *Write to a scratchpad file, not to the CSV.* |
| **OS / DevOps** | ổ đĩa tạm trong RAM (`tmpfs`) | *Mount `/tmp` as tmpfs → an in-memory scratchpad.* |
| **Office software** | vùng tạm của ứng dụng | *Excel application scratch area* |
| **Hardware / AI accelerator** | vùng SRAM tạm trên chip để tính toán | *Tensors must fit in the on-chip scratchpad.* |
| **Neuroscience** | neuron nhớ tạm | *Scratchpad neurons temporarily store information.* |

#### 2.2 The technical meaning: the memory hierarchy

This is the sense with lasting engineering value, and it sits **between cache and DRAM**:

```
CPU Registers            →  fastest, tiniest (bytes)
Scratchpad / Local SRAM   →  "working area on chip", measured in CPU cycles
    ↓
Cache (L1 / L2 / L3)     →  reuse-oriented
    ↓
DRAM / PSRAM             →  large, slow
    ↓
SSD / Storage            →  permanent
```

- **Key property: ephemeral.** Short-lived by design, no need to write to disk.
- **Design principle:** *keep the working set on-chip.* Every operation that must spill to DRAM pays a bandwidth penalty — on modern accelerators, **DRAM bandwidth, not FLOPs, is usually the bottleneck.**

**ESP32 cross-reference** (links to existing notes in this repository):

| ESP32 memory | Scratchpad role? |
|---|---|
| **Internal SRAM** (320 KB total on-class) | ✅ **Yes** — fast on-chip working memory, the scratchpad proper |
| **PSRAM** (external) | ⚠️ Large capacity, higher latency — the "extended desk", not the desk |
| **Flash** | ❌ Permanent storage, not a scratchpad |

> The one-line summary: **scratchpad = fast, small, temporary; PSRAM = slower, large, still temporary; Flash = permanent.** Choosing wrongly is a performance bug, not a style choice.

#### 2.3 Collocations

- `throwaway scratchpad` · `mental scratchpad` · `a scratchpad file / note / script`
- `on-chip scratchpad memory` · `scratchpad capacity` · `fit in the scratchpad` · `scratchpad allocation`
- `mount tmpfs` · `in-memory scratchpad` · `tmpfs-backed /tmp`
- `application scratch area` (Excel)
- `a scratchpad neuron` (Lundgren, Miller & Gershman, 2018)
- `build from scratch` — **từ con số 0**, unrelated to the noun (a very useful idiom)

#### 2.4 Confusion set

| Term | Core focus | Difference |
|---|---|---|
| **scratchpad** | Temporary, **disposable** thinking/working area | Emphasises "throw away after use" |
| **draft** | Version in progress, may be shared | Emphasises "not final but serious" |
| **temp file** | Machine-created intermediate artefact | Technical purpose, not human thinking |
| **notepad** | A note-taking application | A durable tool, not temporary |
| **scratch** (verb) | To think/scribble quickly | *scratch an idea, build from scratch* |
| **RAM disk** | A virtual block device in RAM | The **tool**; `tmpfs` is the **mechanism** |
| **cache** | Stores results for **reuse** | Reuse-oriented, not temporary |
| **buffer** | Holds data in transit | Flow control, not scratch space |

#### 2.5 Viet-Glish pitfalls

| ❌ Viet-Glish | ✅ Native standard | Why |
|---|---|---|
| *"Tôi viết draft trên scratchpad."* | *"I keep a **scratchpad note** for ideas."* | `draft` ≠ `scratchpad` |
| *"Dùng scratchpad để lưu data lâu dài."* | *"Use a **database** for permanent storage; the scratchpad is temporary."* | Ignores the ephemeral meaning |
| *"mount scratchpad /tmp"* | *"**mount tmpfs** on /tmp"* | `tmpfs` is the mechanism name |
| *"the scratch memory of CPU"* | *"**on-chip scratchpad memory** / **SRAM**"* | `scratch memory` is not a standard term |
| *"Ghi chú trong scratchpad cả ngày"* | *"I **brain-dump** into a scratchpad."* | Needs the verb, not the noun alone |
| *schratchpad / scrachpad* | **scratchpad** | Spelling |

---

### 3. ⚖️ Cross-Term Comparison Matrix: label vs scratchpad

| Dimension | **label** | **scratchpad** |
|---|---|---|
| **Lifetime** | Often permanent and versioned | Deliberately ephemeral |
| **Core purpose** | Identify, classify, attribute | Hold working data / thinking, then discard |
| **Machine role** | Target variable in supervised learning | SRAM / tmpfs / temp file |
| **Human role** | Categorise, annotate, warn, brand | Brain-dump, prototype, test |
| **Failure mode** | Label noise, label leakage, annotator bias | Lost work, bottleneck if spilled to DRAM |
| **Verb usage** | *label the data* / *label a person* (H = bad) | No common verb form — noun only |
| **Typical artifact** | `y = [0,1]`, taxonomy, warning sticker | `scratchpad.md`, `/dev/shm`, on-chip SRAM |

---

### 4. Cross-Domain Bridge (Tech ↔ Business ↔ Neuroscience)

**MLOps & data engineering**
- **Annotation cost is a real business line item.** Large-scale labelling is priced per unit and is a major cost in speech and vision datasets. Understanding this makes you a better engineer *and* a better person to talk to finance about.
- **Label leakage is the classic silent failure.** Validation accuracy looks superb, production collapses. Detecting it requires knowing which features are target-derived.
- **Label drift monitoring:** compare production vs training label distributions; alert on divergence. This is the practical form of `label shift`.
- **MLOps vocabulary bridge:** `annotation queue` · `label schema versioning` · `golden dataset` (curated, human-verified) · `agreement rate` (inter-annotator, e.g. Cohen's κ) · `sampling for labelling`.

**Software & system design**
- **Vague labels in UIs become architectural debt.** A dropdown labelled "Status" that mixes *workflow state* and *error severity* will eventually be queried by both and break.
- **`aria-label` is an accessibility contract,** not decoration.
- **Named labels vs positional indices** are a classic source of silent data bugs; agree on the convention and document it.
- **Zero-based vs one-based indexing** — the same class of mistake, different language, and it belongs in the same note as `scratchpad` and `label`: *conventions that silently corrupt data when broken.*

**IoT / embedded (links to `iot_aiot/`)**
- On an MCU, "scratchpad" ≈ **Internal SRAM**: stack, heap and DMA buffers live here. Exceeding it causes a hard fault, not an exception.
- **PSRAM** relieves the pressure but is not free — DMA cannot always reach it, so buffer placement becomes a design constraint. This is exactly the *precision* argument in hardware: cheap capacity in the wrong place is useless capacity.
- **Labelling** at the device layer matters too: sensor calibration tables, firmware version labels, and MQTT topic labelling determine whether an incident can be reconstructed.

**Business & economics**
- **Private label / own-brand** is a margin strategy: a retailer captures manufacturer margin. Understanding the term lets you read a company's annual report segment note correctly.
- **Warning and nutrition labels** are regulation-driven (EU Regulation 1169/2011 for food labelling; CLP/GHS for chemicals) — compliance is a cost line and a barrier to market entry.
- **Historical accounting sense:** *extraordinary* items (see [[vocabulary/05-extraordinary-deep-dive.md]]) were the old category for gains **outside normal operations** — the same Latin logic as *label* marking something **outside** the default category.

**Neuroscience & psychology (links to `psychology_society/`)**
- **Scratchpad neuron** (Lundgren, Miller & Gershman, 2018): a hypothesis that some neurons temporarily store information for later use, supporting working memory. A memorable, human-level intuition for why temporary buffers are computationally necessary.
- **Labelling and stereotype:** *labelling theory* in sociology shows how applying labels to people produces real behavioural consequences — a direct bridge to AI fairness, where models trained on human labels reproduce human prejudice.

---

### 5. Resource & Literature Mapping

| Resource | What to check |
|---|---|
| Lundgren, Miller & Gershman (2018), *Cogent Psychology* | "Scratchpad neurons" |
| O'Neil & Rudd, *Weapons of Math Destruction* | Labeling systems, bias, scale |
| Zhang et al. (2015), *ICML* | "Deep Learning with Label Noise" |
| Kaufman et al. (2012), *Big Data* | "Leakage in Data Mining" |
| Hand, (2009), *Pattern Recognition* | Classifier evaluation: precision, recall, accuracy |
| Linux `man tmpfs(5)` | Authoritative mechanism definition |
| JEDEC / hardware vendor docs | On-chip scratchpad vs cache terminology |
| W3C WAI-ARIA spec | `aria-label` normative definition |
| EU Reg. 1169/2011 · CLP/GHS | Food & chemical labelling regulation |
| Label Studio / CVAT documentation | Real annotation-pipeline vocabulary |

---

## 🧪 Practice Section

### A. Choose the correct term
1. The annotator marked the transaction as ________, but the customer confirmed it was legitimate. *(labelled / scratch-padded)*
2. Tensors too large for the on-chip ________ must spill to DRAM, which bottlenecks throughput. *(label / scratchpad)*
3. Metrics only make sense if the ________ and metric name appear on the same axis. *(label / scratchpad)*
4. Mount a RAM-backed ________ on `/tmp` to avoid writing temporary files to the SSD. *(label / tmpfs)*

### B. Fill in the collocations
5. Please ________ every file before uploading it to the dataset. → *(complete)*
6. The IR team keeps a ________ Markdown file with raw ideas before promoting anything to a real ADR. → *(complete)*

### C. Rewrite the Viet-Glish
7. "Gán label 0 và 1 cho dữ liệu." → ______
8. "Dùng scratchpad để lưu dữ liệu vĩnh viễn." → ______

### Answer Key
1. **labelled** (or: *mislabeled*) · 2. **scratchpad** · 3. **labels** · 4. **tmpfs**
5. **label** · 6. **scratchpad**
7. *"**Encode** the data as 0 and 1"* / *"**Assign** the labels 0 and 1."*
8. *"Use a **database** (or object storage) for permanent data; a **scratchpad** is temporary."*

---

## 🔬 Research Journey

- **Why I'm researching this:** two words that look unrelated but share one idea — *things that are marked*, and *things that are temporary*. In an ML system, labels are the marked thing and the scratchpad is the temporary thing; together they describe where data lives and what it means. That is a useful mental model for the whole data lifecycle.
- **What I struggled with:** (a) *label* as a verb applied to people is a stigma machine, and I did not know how offensive the neutral translation felt; (b) *encode* vs *label* in Vietnamese collapses into one phrase, so my first instinct was always wrong; (c) I had assumed "scratchpad" only meant a piece of paper, and completely missed the on-chip memory meaning — which is the one that actually matters for hardware reasoning.
- **Aha moments:** (1) label leakage is the reason a perfect validation score is a red flag — a lesson about measurement, not machine learning; (2) on an ESP32, "scratchpad = internal SRAM" makes the PSRAM-vs-SRAM note suddenly intuitive; (3) *labelling theory* in sociology and *label bias* in ML are the same problem at two scales, which is a strong bridge between my AI and psychology folders.
- **Career connection:** annotation cost, label quality and label drift are three things every ML engineer eventually owns, and each one is a conversation with a non-technical stakeholder. Meanwhile the scratchpad concept ties my software architecture work to the IoT notes I already have, and to accelerator performance reasoning.

---

## 🔗 Related Topics

- [[vocabulary/05-extraordinary-deep-dive.md]] — the historical accounting sense of *extraordinary* shares the "outside the ordinary" logic of labelling
- [[vocabulary/06-document-and-communication-verb-family.md]] — *label* is half of documentation language; this note covers the verbs
- [[iot_aiot/]] — internal SRAM vs PSRAM: the concrete embedded case of "scratchpad"
- [[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]] — where monitoring meets label shift
- [[software_architecture/01-event-driven-architecture-task-orchestration.md]] — the bank task system's data and labelling contracts
- [SUGGESTED] **Distribution shift family** (`covariate shift`, `label shift`, `concept drift`, `data drift`, `out-of-distribution`) — *why:* one note, four terms that are constantly confused in MLOps and interviews; this is the single highest-value next vocabulary note for an MLOps-oriented AI Engineer. → fits `vocabulary/`, with a bridge to `ai_ml/`
- [SUGGESTED] **Evaluation metrics precision** (`precision`, `recall`, `F1`, `accuracy`, `confusion matrix`, `Cohen's κ`, `golden dataset`) — *why:* a bank or IoT project always needs a defensible metric, and precision/accuracy confusion is the most common technical interview error. → fits `vocabulary/`
- [SUGGESTED] **Storage tiers & non-volatile memory** (`SRAM`, `DRAM`, `PSRAM`, `flash`, `EEPROM`, `cache`, `buffer`) — *why:* completes the memory hierarchy line begun here and bridges software architecture to IoT hardware. → fits `software_architecture/` or `iot_aiot/`

---
*Precision in vocabulary reflects precision in engineering.*
