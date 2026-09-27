# Vocabulary & Collocation — **FRAGILE** & **CRITICAL PROPERTY** (robustness, severity & failure)

**Created Date:** 2026-09-27
**Last Updated:** 2026-09-27
**Tags:** `#vocabulary`, `#collocations`, `#word-family`, `#reliability-engineering`, `#materials-science`, `#safety-engineering`, `#risk`, `#viet-glish`
**Status:** Active | Reviewed
**Template:** Template V — Advanced Vocabulary & Collocation
**Source note:** Vietnamese *mỏng manh / dễ vỡ / mong manh* and *tính chất quyết định*

---

## 📌 Term List & Quick Reference

| Term | IPA | Core meaning | Domain |
|---|---|---|---|
| **fragile** (adj) | `/ˈfrædʒ.əl/` | Easily broken, damaged or harmed; not able to tolerate disturbance | General / Systems / Materials |
| **critical** (adj) | `/ˈkrɪt.ɪ.kəl/` | Deciding the outcome; so important that failure is unacceptable | General / Engineering / Medical |
| **critical property** (n phrase) | — | A property whose violation causes failure; a decisive material or thermodynamic property | Materials / Systems / Safety |
| **brittle** (adj) | `/ˈbrɪt.əl/` | Hard but liable to snap suddenly, without warning | Materials / Systems |
| **robust** (adj) | `/rəʊˈbʌst/` | Strong; able to withstand stress without failing | All |
| **resilient** (adj) | `/rɪˈzɪl.i.ənt/` | Able to recover quickly after difficulty | Systems / Organisations / Health |
| **antifragile** (adj) | `/ˌæn.tiˈfrædʒ.əl/` | Improved by stress and volatility | Taleb / Systems |

---

## 🧠 Detailed Vocabulary Analysis

### 1. FRAGILE — "low tolerance to disturbance"

The single idea behind every sense: **something breaks under a small load**.

| Sense | Vietnamese | Example |
|---|---|---|
| **Physical object** | dễ vỡ, mỏng manh | *The parcel is marked **fragile**.* |
| **Physiology / health** | yếu, mỏng manh về thể chất | *He has **fragile** bones / a **fragile** immune system.* |
| **Emotion** | mong manh, nhạy cảm | *a **fragile** sense of self* |
| **Abstract relations** | mong manh, dễ vỡ | *a **fragile** peace / **fragile** truce* |
| **Institutions & states** | mong manh, chưa vững | *a **fragile** democracy / a **fragile** state* |
| **Systems & software** | dễ sập, không chịu được thay đổi nhỏ | ***fragile** integration tests* |
| **ML & data** | nhạy cảm, dễ vỡ với nhiễu | *a **fragile** model / **fragile** features / a **fragile** prompt* |
| **Business & infrastructure** | dễ đứt gãy | *a **fragile** supply chain / **fragile** infrastructure* |
| **Logistics label** | "dễ vỡ" (marking) | *Handling marking: **FRAGILE*** |

#### The systems meaning deserves special attention
In engineering and software, *fragile* has a precise definition: **a component that fails when a small, unrelated change is made somewhere else.**
- **Fragile code** — works until you change an unrelated module.
- **Fragile tests** — a test that fails when you rename a variable three files away. The cure is testing *behaviour* through a stable interface, not internal structure.
- **Fragile abstraction** — a leaky abstraction (Ousterhout): the abstraction hides some things well and others badly, so users depend on the wrong details.
- **Fragile coupling** — modules that must change together, so every change propagates.

#### ⚠️ In materials science, *brittle* is the technical term, not *fragile*
| Property | Vietnamese | Meaning |
|---|---|---|
| **brittle** | giòn, dễ gãy đột ngột | High hardness, fractures suddenly, little warning |
| **ductile** | dẻo | Deforms plastically before breaking |
| **tough** | dai, bền | Absorbs a lot of energy before fracture |
| **malleable** | dễ bị bóp dẹt | (Metals, under compression) |
| **fragile** | dễ vỡ | Everyday / figurative only; not a materials term |

- **Classic illustration:** glass is **strong in tension** yet **brittle** — it resists a pull but shatters without warning. Steel is **ductile**: it deforms and warns you first. This is exactly the same trade-off as **speed vs safety in software**.
- **Toughness** = strength + ductility combined; engineers quote toughness as energy absorbed before fracture.

#### Fragile vs brittle vs delicate vs flimsy vs frail vs robust vs resilient

| Word | Core nuance | Example |
|---|---|---|
| **fragile** | Easily broken or harmed (general, both senses) | *a **fragile** ecosystem* |
| **brittle** | Hard, but snaps **suddenly without warning** | ***brittle** fracture* / *a **brittle** system* |
| **delicate** | Finely made, needs care — ⚠️ **also positive**: subtle, precise | *a **delicate** mechanism* · *a **delicate** distinction* · ***delicate** negotiations* |
| **flimsy** | Thin **and** unconvincing | *a **flimsy** excuse / **flimsy** evidence* |
| **frail** | Physically weak (mainly people) | *the **frail** elderly* |
| **sturdy** | Solidly built | *a **sturdy** chair / **sturdy** construction* |
| **robust** | Withstands stress; not affected much | *a **robust** model / **robust** method* |
| **resilient** | Recovers **after** being damaged | *a **resilient** economy / **resilient** team* |

> **The triad worth memorising:** **robust** = does not break · **resilient** = breaks but recovers · **antifragile** = **improves because of** the stress (Taleb, *Antifragile*, 2012). Fragile is the member of the family with none of these protections.

### 2. CRITICAL — "deciding; failure is unacceptable"

| Sense | Vietnamese | Example |
|---|---|---|
| **Deciding the outcome** | quyết định, then chốt | *Today is a **critical** moment in the project.* |
| **Of a system: failure is unacceptable** | then chốt, không được phép hỏng | *We need **critical** reliability on the payment path.* |
| **Serious / grave** | nghiêm trọng, nguy kịch | *The patient is in a **critical** condition.* |
| **Important** (general) | quan trọng | *Maintaining backward compatibility is **critical**.* |
| **Disapproving** *(be critical of)* | chỉ trích | *Don't be **critical** of your own early drafts.* |
| **Disapproving** *(a critical remark)* | gay gắt | *She made a **critical** remark about the estimate.* |

- **Pronunciation:** `/ˈkrɪt.ɪ.kəl/` — a well-known trap. ❌ */ˈkrɪ.tɪ.kal/*, ❌ */ˈkrɪ.ti.kəl/*.
- ⚠️ **Misspelling:** ~~crucical~~ → **crucial** (extremely common learner error).

### 3. CRITICAL PROPERTY — a technical phrase, not everyday English

⚠️ First, the important register note: **`critical property` is a technical collocation, not a general way of saying "important characteristic".** In ordinary writing, say **key property**, **essential attribute** or **must-have**. Use *critical property* when you mean *this property, if violated, is unacceptable*.

| Domain | Meaning of "critical property" | Example |
|---|---|---|
| **Materials & manufacturing** | A decisive physical/mechanical/thermal/chemical property used to select or qualify a material | *The **critical properties** of the alloy are tensile strength and corrosion resistance.* |
| **CAE / FEA** | A **critical property data** entry — the material inputs the simulation needs | *Convergence failed: missing **critical property** data.* |
| **Thermodynamics** | The properties defining a **critical point**: $T_c$, $P_c$, $V_c$, $\rho_c$ | *The **critical property** of interest is the critical temperature.* |
| **Systems & software** | A property whose violation causes failure, hazard or data loss | *For a payment service, idempotency is a **critical property**.* |
| **Safety engineering** | A **safety-critical** property, governed by standards | *IEC 61508 classifies functions by how critical those properties are.* |
| **Business continuity** | A **facility/asset** whose loss stops operations | *We classified three sites as **critical properties**.* |
| **ML** | A property a model must preserve to remain acceptable | *Calibration is a **critical property** for a credit-scoring model.* |

#### ⚠️ `critical property` ≠ `critical value`
| Term | Domain | Meaning |
|---|---|---|
| **critical property** | Engineering / systems | A decisive or safety-critical characteristic |
| **critical value** | **Statistics** | The threshold beyond which a result is significant at level $\alpha$ |
| **critical point** | Physics / thermodynamics | The endpoint of a liquid–gas boundary |
| **critical region** | Statistics | The set of outcomes that reject the null hypothesis |
| **critical exponent** | Physics | The exponent governing behaviour near a phase transition |

> Writing *"the critical value of the tensile strength"* in a statistics context would be read as a statistical threshold — a genuine ambiguity worth avoiding by saying *"the threshold tensile strength"*.

#### The four-step sequence
**identify → measure → verify → monitor** — every critical property in engineering follows it: name the property, define how it is measured, prove it holds (test or proof), then track it in production. In databases, atomicity and durability are the critical properties, verified by failure-injection tests and monitored by replication lag.

#### `critical` vs `crucial` vs `vital` vs `essential`
- **critical** = deciding the outcome; there is no tolerance for failure
- **crucial** = of decisive importance (very close synonym, more common in speech)
- **vital** = necessary for life or existence (*vital signs*, *vital to survival*)
- **essential** = strictly necessary, cannot be done without
- ⚠️ **None of them take a comparative:** ❌ *"more critical"* is acceptable, ❌ *"most crucial"* is fine, but ❌ *"very critical"* and ❌ *"the most criticalness"* are not.

### 4. Viet-Glish Pitfalls

| ❌ Viet-Glish / common error | ✅ Native standard | Why |
|---|---|---|
| "This is **very critical**." | "This is **critical**." / "This is **crucial**." | Redundant intensifier — a famous error |
| "This is **most importance**." | "This is **of utmost importance**." / "This is **very important**." | Wrong part of speech |
| "**crucical**" | **crucial** | Spelling |
| "**kritical**" / "**criticial**" | **critical** | Spelling |
| "**the critical**" for a decisive point | "**a critical** turning point" | `critical` as a noun is countable: *a critical* |
| "Tính chất **very critical** của hệ thống" | "The system has **critical** properties" / "**safety-critical** properties" | *Very critical* again |
| "Tính chất quan trọng nhất" | "the **key property**" (everyday) vs "the **critical property**" (technical) | Register |
| "Tôi **không quan trọng**" | "**I don't mind**." / "It's **not important**." | `I don't care` is much stronger in English |
| "Bệnh nhân **very critical**" | "The patient is in a **critical condition** / **critically ill**" | Fixed collocations |
| "**delicate** = mỏng manh (always)" | *delicate* also = tinh tế, nhạy cảm | Polysemy, see matrix |
| "**Fragile** glass = giòn" (materials) | "**Brittle** glass" | Wrong technical term |
| "My laptop is **flimsy**." | "My laptop is **flimsy-built** / **poorly constructed**." | *Flimsy* criticises flimsy design, not durability of use |
| "**robust** = bền vững (only)" | *robust* also = chịu được nhiễu, ổn định về mặt thống kê | Technical sense dominates |

### 5. Cross-Domain Bridge (Tech ↔ Business ↔ Social Science)

**Reliability engineering**
- The standard vocabulary is a **spectrum, not a binary**: *fragile* → *brittle* → *robust* → *resilient* → *antifragile*.
- **Brittle failure** vs **fail-safe** vs **fail-stop** vs **graceful degradation** — the design goal for a payment or IoT gateway is usually *fail-safe / degrade gracefully*, never *fail silently*.

**Software & architecture**
- **Fragile tests** are a code-smell with a known cure: assert on public behaviour, not on internals. A test that must change when a private field is renamed is testing implementation, not behaviour.
- **Leaky abstractions** (Ousterhout) are fragile abstractions; **blast radius** is the quantitative version of fragility. Together they belong in the same design review.
- **Backward compatibility** is a *critical property* of any public API — see [[vocabulary/11-position-sequence-and-direction-vocabulary.md]], where *backward* is analysed in full.
- **Safety-critical** systems (IEC 61508, ISO 26262) assign severity by which properties, if lost, cause harm. For a bank ledger the critical properties are durability, atomicity, auditability and idempotency.

**AI / ML**
- **Adversarial robustness** is the formal version of *not fragile*: the model must not change its prediction under small, bounded perturbations. Robust accuracy is reported alongside clean accuracy, and a large gap is a fragility signal.
- **Fragile features** are inputs whose usefulness disappears under a small distribution shift; **brittle prompts** behave the same way. If your pipeline only works on the exact data you trained on, it is fragile — the practical warning sign is a *high* train–serve skew.
- **Critical model properties** for regulated use: calibration, fairness across slices, monotonicity (for credit scoring), and auditability. Each is a property that can be tested, so each is a *critical property* in the engineering sense.

**Economics & finance**
- **Systemic fragility**: Minsky's financial instability hypothesis argues that stability breeds fragility, so calm periods are when risk accumulates. **Too big to fail** is the policy response to fragility, and it is itself a fragility amplifier.
- **Critical infrastructure** and **business continuity**: organisations classify assets and locations as *critical* precisely because their loss halts operations — the same phrase used for buildings.
- **Supply-chain fragility** became a board-level topic after 2020; the vocabulary is *fragile*, *critical*, *single point of failure*.

**Psychology & health**
- **Resilience** in developmental psychology (Masten's "ordinary magic") is the science of why some children and systems recover from adversity. Its companions are **stress inoculation** and **post-traumatic growth** (Tedeschi & Calhoun).
- **Social fragility**: fragile institutions, fragile trust, a *fragile peace* — the vocabulary of peace studies, where the goal of peacebuilding is resilience rather than the absence of disturbance.
- **Fragile X syndrome** (FMR1) is the one place in biology where the word is a proper clinical term; "fragile site" describes a chromosome region prone to breakage.

**Neuroscience**
- **Fragility** of synapses: the contrast between **stable** synapses (LTP-consolidated) and **labile** ones (still plastic) determines whether a memory survives consolidation. Same word, opposite outcomes depending on timing.

---

## 🧪 Practice Section

### A. Choose the best word
1. The test suite is ______ — it fails every time we rename a private field. *(fragile / delicate / flimsy)*
2. The pension is a ______ commitment; the bank cannot absorb a default. *(critical / crucial / cruciality)*
3. At the ______ point of the liquid–gas curve, the two phases become indistinguishable. *(critical / fragile / delicate)*
4. The blank it makes on failure is ______, so the service should degrade gracefully. *(fragile / resilient / antifragile)*
5. The patient is in a ______ condition. *(critical / fragile / delicate)*

### B. Fix the Viet-Glish
6. "This is a very critical property for the payment system." → ______
7. "The glass is fragile, so it breaks suddenly without warning." → ______
8. "Delicate arguments" nghĩa là gì? → ______

### Answer Key
1. **fragile** · 2. **critical** · 3. **critical** · 4. **fragile** · 5. **critical**
6. *"This is a **critical** property of the payment system."* (drop `very`)
7. *"The glass is **brittle**: strong in tension but it fractures suddenly, without warning."*
8. **Lập luận tinh tế / vi mô** — *delicate* here is **positive**, meaning finely balanced and precise, not *mỏng manh*.

---

## 🔬 Research Journey

- **Why I'm researching this:** *fragile* and *critical* are the two adjectives I reach for constantly in project writing, and both are misused in the same direction — *fragile* where *brittle* is the technical term, and *"very critical"*, which is a redundancy a native speaker notices immediately. Both words are also the core of how risk gets discussed, so getting them right changes how a specification reads.
- **What I struggled with:** (a) not knowing that *delicate* can be a compliment; (b) using *fragile* for materials instead of *brittle*; (c) never realising *critical property* is a real engineering collocation with a precise meaning — the property whose violation is unacceptable — which is stronger than "important".
- **Aha moments:** (1) the spectrum **fragile → brittle → robust → resilient → antifragile** is a genuine decision aid, not a list of synonyms; (2) *critical value* is a statistics term and *critical property* is an engineering term, so writing both in one document without care creates real ambiguity; (3) Minsky's insight that stability breeds fragility explains why "everything is fine" is the most dangerous engineering state.
- **Career connection:** robustness of a model, critical properties of a banking API, and business continuity for critical infrastructure are three views of the same question — *what breaks, how badly, and what do I do then?* That is the question an interviewer is actually asking when they ask "how would you handle failure?"

---

## 🔗 Related Topics

- [[vocabulary/08-property-multi-sense-lexicon.md]] — the noun *property* behind *critical property*
- [[vocabulary/11-position-sequence-and-direction-vocabulary.md]] — *backward compatibility* as a critical property
- [[vocabulary/09-commit-and-working-state-terminology.md]] — atomicity and durability as critical properties of a transaction
- [[software_architecture/01-event-driven-architecture-task-orchestration.md]] — graceful degradation and idempotency
- [[ai_ml/05-mlops-lifecycle-and-deployment-architecture.md]] — model robustness and monitoring
- [SUGGESTED] **Failure-mode vocabulary** (`failure mode`, `fault`, `defect`, `tolerance`, `fault tolerance`, `graceful degradation`, `fail-safe`, `redundancy`, `MTBF`, `retry`, `circuit breaker`) — *why:* one note that turns "fragile" into a design discipline, and it is exactly the vocabulary used in reliability interviews and in architecture reviews. → fits `software_architecture/`
- [SUGGESTED] **Risk and severity vocabulary** (`risk`, `hazard`, `exposure`, `likelihood`, `impact`, `severity`, `mitigation`, `residual risk`) — *why:* pairs with *critical property* to complete the language of risk registers, which the idea notes in `random_ideas/` already use. → fits `software_architecture/` or `economics_business/`
- [SUGGESTED] **Materials & mechanics vocabulary** (`brittle`, `ductile`, `tough`, `yield strength`, `fatigue`, `fracture`, `hardness`) — *why:* surprising depth for a CS student, but it explains hardware and IoT reliability physically, and it sharpens the software speed-vs-safety analogy. → fits `iot_aiot/`

---
*Precision in vocabulary reflects precision in engineering.*
