# Vocabulary & Collocation — **PROPERTY** (7 senses across maths, code, law & everyday speech)

**Created Date:** 2026-09-27
**Last Updated:** 2026-09-27
**Tags:** `#vocabulary`, `#collocations`, `#word-family`, `#etymology`, `#maths`, `#object-oriented`, `#css`, `#intellectual-property`, `#viet-glish`
**Status:** Active | Reviewed
**Template:** Template V — Advanced Vocabulary & Collocation
**Source note:** Vietnamese *thuộc tính / đặc tính / bất động sản / tài sản trí tuệ*

---

## 📌 Term List & Quick Reference

| Item | Content |
|---|---|
| **Headword** | property |
| **IPA** | UK `/ˈprɒp.ə.ti/` · US `/ˈprɑː.pɚ.ti/` |
| **Part of speech** | Noun (countable & uncountable) · plural *properties* / `ˈprɒp.ətiz/ |
| **Level** | C1 (the mathematical and legal senses are advanced; the everyday sense is B1) |
| **Etymology** | Latin **proprietās** ← **proprius** ("one's own") ← **pro-** ("forward/before") → *proper, property, proprietary, propriety, appropriate* |
| **Senses** | A. characteristic of a thing · B. abstract formal property (philosophy/maths) · C. algebraic property · D. code property / attribute · E. CSS & DOM property · F. real estate & ownership · G. intellectual property |
| **Star collocations** | `a defining property` · `an intrinsic property` · `have the property that…` · `property of X is Y` · `intellectual property` · `a custom property` |
| **Main confusables** | attribute, feature, characteristic, trait, quality, possession, asset, propriety |

---

## 🧠 Detailed Vocabulary Analysis

### 1. Etymology — one Latin root explains five English words

- **proprietās** ("ownership") ← **proprius** ("one's own, proper") ← **pro-** ("forward, before").
- Therefore these words are **literally the same idea** — *what belongs to you*:
  - **proper** = one's own → by extension *correct, suitable*; hence **proper noun**, and the mathematical **proper subset / proper class / proper value**
  - **property** = what is owned → what belongs to a thing by its nature (its characteristics)
  - **proprietary** = owned by a private owner → *proprietary software*, *a proprietary format*
  - **propriety** = what is appropriate → *sense of propriety*
  - **appropriate** = fitted to purpose ← *pro- + proprius*
- **Practical payoff:** when you see *proprietary*, *proper* or *property*, think **"own"**. So *"a proprietary database engine"* = an engine owned by someone, and *"data is a property of the bank"* = it belongs to the bank. The legal and technical senses are cognates, not separate words.

### 2. The seven senses

#### Sense A — Characteristic of a thing (default, most common)
> *Friction is a **property** of the material, not of the design.*
> *The system has the **property** of recovering automatically after a crash.*

- **VI gloss:** thuộc tính, đặc điểm, đặc tính.
- **Register:** neutral to formal. Everyday natives prefer *feature* or *characteristic*: *"What are its properties?"* sounds technical or medical.
- **Grammar:** countable — *one property*, *several properties*, *a property of X*.

#### Sense B — Abstract formal property (philosophy, logic, linguistics)
> *Colour is an external property; shape is an internal property.*
> *Being a verb is a **property** of the word "run".*

- **VI gloss:** đặc tính (thuộc tính khách quan, mang tính định nghĩa).
- Used in philosophy, linguistics and linguistics of grammar: *genetic properties of a word*.
- Note the phrasal form: **have / possess the property of being + adjective** — *"the property of being idempotent"*.

#### Sense C — Algebraic / structural property of a mathematical object
> *Which **properties** must a cryptographic hash function satisfy?*
> *Closure is a **property** of the set of integers under addition.*

| Property | Meaning |
|---|---|
| **commutativity** | $a + b = b + a$ |
| **associativity** | $(a + b) + c = a + (b + c)$ |
| **closure** | the result stays inside the set |
| **idempotence** | $f(f(x)) = f(x)$ |
| **invariance** | unchanged under a transformation |
| **monotonicity** | never reverses direction |
| **determinism** | same input → same output, always |
| **pre-image resistance** | infeasible to invert a hash |
| **collision resistance** | infeasible to find two inputs with the same hash |

- **Proof-writing pattern:** *"Property P holds for all $x \in S$"* · *"a function is said to have the **property** P if …"*
- **Verification pattern (interview-grade):** *"the function must satisfy the **properties** of correctness, termination and complexity bound."*

#### Sense D — Object property in programming
> *The object's **properties** are accessible only through its accessors.*
> *Java reads system **properties** from the command line via `-Dkey=value`.*

| Concept | Meaning |
|---|---|
| **property** | A data member of an object; often accessed through a getter/setter so validation is enforced |
| `@property` (Python) | A **decorator** that turns a method into an attribute-like accessor |
| **system property** (Java) | JVM-level key/value config, e.g. `System.getProperty("user.dir")` |
| **.properties file** | Flat `key=value` config file |
| **stored vs derived property** | Stored = persisted; derived = computed on the fly (database design) |
| **multivalued property** | May hold several values (ER modelling) |

- ⚠️ **`property` vs `attribute` — the distinction is real but blurry.** Strictly: *property* = the stored value; *attribute* = the descriptive characteristic. In practice, languages use the words loosely. **Python** technically has *attributes* only; `@property` is a descriptor that gives attribute-style access to a computed value. **JavaScript** objects have *properties*; HTML has *attributes*. **Databases** have *attributes* (columns) and *properties* (metadata such as nullability or key-ness).

#### Sense E — CSS & the DOM
- **CSS property:** a declaration slot in the cascade — `color`, `display`, `grid-template-columns`. The trio is always **selector → property → value**: `p { color: red; }`.
- **Custom property:** `--brand-color` (note the `--` prefix; it is a variable, not a normal property).
- **`@property` at-rule:** lets a custom property be *registered* with a type and an initial value, so the browser can animate or validate it.
- **Inheritance:** some properties inherit (`color`, `font-size`), others do not (`margin`, `border`).
- **HTML attribute vs DOM property:** the classic trap — `getAttribute("value")` returns a **string**; the IDL `.value` returns a **typed** value. `className` ↔ `class`, `innerHTML` ↔ `textContent`. This is a frequent source of bugs.

#### Sense F — Real estate & ownership
> *The bank repossessed the commercial **property**.*
> *She owns two **properties** in Đà Nẵng.*

- **VI gloss:** bất động sản, tài sản.
- **BrE vs AmE:** *property* is normal British usage; American English more often says **real estate** or **real property**.
- **Key collocations:** `property tax` · `property valuation` · `property management` · `property developer` · `property agent` · `commercial / residential property` · `property insurance` · `on the property`

#### Sense G — Intellectual property & legal ownership
> *The company holds intellectual **property** over the training pipeline.*

- **VI gloss:** tài sản trí tuệ.
- **The IP family:** *intellectual property (IP)* · *patents* · *trademarks* · *copyright* · *trade secrets* · *licensing* · *assignment* · *the public domain*
- **Extended legal senses:** *property rights* · *real property* vs *personal property* · *a deed of property* · *property law* · *to dispose of property*

### 3. Word family

| POS | Form | Example |
|---|---|---|
| Noun | **property** | *an intrinsic property* |
| Noun (plural) | **properties** | *the algebraic properties of the operation* |
| Noun (legal) | **property** | *ownership of intellectual property* |
| Adjective | **proper** | *a proper noun* · *a proper subset* |
| Adjective | **proprietary** | *proprietary technology* |
| Noun | **propriety** | *a sense of propriety* |
| Adjective | **appropriate** | *an appropriate response* |
| Noun | **proprietor** | *the proprietor of the shop* |
| Adjective | **properly** (adverb) | *handle the exception properly* |

⚠️ **Propriety ≠ property.** *Propriety* = hợp pháp, đứng đắn, thích hợp (*a breach of propriety*). *Property* = tài sản/thuộc tính. A Vietnamese learner confusing these produces sentences like *"It violates propriety of the data"*, where the intended word is **privacy** or **propriety**. Three near-homophones, three different meanings.

### 4. ⚖️ Confusables Comparison Matrix

| Term | Core focus | Typical domain | Example |
|---|---|---|---|
| **property** | A characteristic or ownership interest | Maths, code, law | *The function has the **property** of being monotone.* |
| **attribute** | A descriptive characteristic; a data column | OOP, databases, linguistics | *Every employee record has a **name attribute**.* |
| **feature** | A notable, usually user-facing characteristic | Products, ML | *The main **feature** is offline support.* |
| **characteristic** | The general, neutral word | Any | *Colour is a **characteristic** of the species.* |
| **trait** | Of a person or organism | Psychology, biology | *She has a **trait** of caution.* |
| **quality** | A subjective evaluation | Any | *The build **quality** is excellent.* |
| **possession** | The act/state of having | Legal, everyday | ***Possession** is nine-tenths of the law.* |
| **asset** | Economic value that can be owned | Finance | *The building is a **fixed asset**.* |
| **propriety** | Appropriateness / respectability | Ethics, law | *a breach of **propriety*** |

> **Memory anchor:** *feature* is what a product **offers**; *property* is what a thing **is**; *asset* is what you **own**; *propriety* is what is **decent**.

### 5. Native sentence patterns

- **`X is a property of Y`** — the most productive frame in technical writing.
  *Throughput is a property of the architecture, not of the code.*
- **`the property of being + adj/gerund`** — *the property of being idempotent*
- **`have / possess the property that + clause`** — *the operator has the property that it is associative*
- **`which properties must … satisfy?`** — the interview question about correctness, safety and complexity
- **`one of its key properties is …`** — natural in papers and specifications
- **`for any property P of X, …`** — mathematical quantifier style
- **`by its very nature, it has the property that …`** — argumentative prose

### 6. Register notes

- The **mathematical and philosophical senses are academic register**. Never use *property* in casual speech to describe a phone: *"This phone has great properties"* sounds wrong in everyday English. Say **features**.
- In **legal and business English**, *property* is a high-frequency technical noun with no casual equivalent — do not soften it.
- `custom property` and `system property` are **developer register**; avoid in non-technical writing.

### 7. Viet-Glish pitfalls

| ❌ Viet-Glish / common error | ✅ Native standard | Why |
|---|---|---|
| "Dùng property để mô tả tính năng sản phẩm trong marketing" | "Use **features** for product marketing; **properties** for technical characteristics" | Register: *properties* sounds technical/medical in casual speech |
| "Độc quyền sở hữu trí tuệ" | "**intellectual property**" / "**IP rights**" | Fixed legal term |
| "Property của HTML element" | "**attribute** of an HTML element" | Markup = attribute; the DOM value = property |
| "set property color = red" | "`color` is the property, `red` is the **value**" | CSS trio: selector → property → value |
| "This function is a property." | "This function **has** the property of being **monotone**." | A function *has* properties; it is not one |
| "The property is belongs to me." | "The **property belongs** to me." | No article; *belong* takes no `to` after subject |
| "a proper" (math) vs "a property" | **proper subset / proper class / proper value** vs **property** | Same Latin root, different words |
| "This property describes the state." | "**Propriety** describes what is appropriate." | Near-homophone trap |
| "property" for what a person possesses | "**possession** / **belonging**" | *Property* is the thing, *possession* is the state |
| "Tôi muốn hỏi về properties của hàm này." | ✅ fine in a technical setting; in casual speech say "…about how this function behaves" | Over-formal outside technical contexts |

### 8. Cross-Domain Bridge (Tech ↔ Business ↔ Social Science)

**Mathematics & algorithms**
- Properties are **testable claims** about an object, and proofs establish them. Software engineers operationalised this: **property-based testing** (Hypothesis, fast-check) asserts *properties* instead of fixed examples — *"the property under test is associativity"*. This is the single most productive idea in this note for a career in engineering.

**Software & system design**
- **Design by Contract** (Meyer) formalises the vocabulary: *preconditions*, *postconditions*, *invariants* — all **properties** that must hold at program points.
- **Non-functional properties** are how requirements are written: performance, reliability, security, scalability, maintainability. A specification that lists properties instead of features is reviewable; one that lists features is not.
- **CSS** is a cascade of properties with inheritance — a clean, teachable model of specificity resolution.

**Security**
- **Security properties** are named, testable claims: confidentiality, integrity, availability, authenticity, non-repudiation. *"Verifying a signature establishes the authenticity property, not the identity of a person."*

**AI / ML**
- **Scaling laws** express a **property of the function**, not of the model: loss follows a power law in parameters, data and compute. This is the direct link to [[vocabulary/05-extraordinary-deep-dive.md]] — *extraordinary* performance and **out-of-distribution generalisation** are both properties of the learned function.
- Model **properties** used in evaluation: robustness, calibration, monotonicity, fairness, consistency across slices. A model is a function; a *property* is a claim about the function over its whole domain.
- **Data properties**: distribution, support, sparsity, cardinality, missingness, label schema version — the metadata an MLOps team must declare before a model is approved.

**Economics & business**
- **Property rights theory** (Coase, Demsetz, North — the 1990 Nobel to North) argues that the **definition and enforcement of property rights** shapes allocation and growth. If you cannot exclude someone from an asset, it is not a property; it is open access. This links directly to [[economics_business/04-network-effects.md]] and [[economics_business/03-sunk-cost-fallacy.md]].
- **Intellectual property as a business asset**: patent portfolios, trade-secret governance, licensing revenue, and the platform problem — a standards war is a fight over which specification is the *de facto* property right.
- **S&OP vocabulary:** *firm / committed / forecast* buckets — see [[vocabulary/09-commit-and-working-state-terminology.md]].

**Psychology & society**
- **Attitude theory** (Rosenberg): an attitude is an **affective evaluation of an object through its properties**. This is the social-science form of the same word and explains why naming properties triggers evaluation.
- **Stereotyping** is the cognitive abuse of the same operation: attribute a few properties to a person and treat them as constitutive. Compare with the labelling discussion in [[vocabulary/07-labelling-and-temporary-memory.md]] — labelling people and labelling data are the same act at two scales.

**Neuroscience**
- **Intrinsic vs extrinsic properties** of a neuron: intrinsic (membrane conductances, firing threshold) versus extrinsic (synaptic input from outside). The word *intrinsic* here means "generated internally", not "essential" — a useful second sense of *intrinsic* to watch.

---

## 🧪 Practice Section

### A. Choose the correct sense
1. Which ______ must a hash function satisfy to be usable in a blockchain? *(properties / features / possessions)*
2. The company sued for theft of intellectual ______. *(property / propriety / propriety's)*
3. The retrieval of `.value` returns a typed DOM ______. *(attribute / property / trait)*
4. Nested transactions must be flattened before the outer transaction can ______. *(commit / property / label)*

### B. Fill in the collocations
5. Closure is a ______ of integers under addition. → *(complete)*
6. The bank valued the commercial ______ at 4.2 billion VND. → *(complete)*
7. In the cascade, `display` is the ______ and `none` is its ______. → *(complete)*

### C. Rewrite the Viet-Glish
8. "Properties của cái máy này rất tốt." → ______
9. "set attribute color = red" → ______

### Answer Key
1. **properties** · 2. **property** · 3. **property** · 4. **commit**
5. **property** · 6. **property** · 7. **property / value**
8. *"This phone has **excellent features**."*
9. *"`color` is the CSS **property**; `red` is the **value**."*

---

## 🔬 Research Journey

- **Why I'm researching this:** one word standing in for five different domains is a comprehension hazard, not just a vocabulary item. The moment I stopped treating *property* as "thuộc tính" and started asking *whose property is this, in which sense*, technical English became much easier.
- **What I struggled with:** (a) the DOM attribute/property split, which contradicts most tutorials; (b) *propriety* vs *privacy* vs *property*; (c) knowing when a Vietnamese sentence wants *feature* instead of *property*.
- **Aha moments:** (1) the Latin root **proprius = "one's own"** links *property*, *proper*, *proprietary* and *appropriate* — one etymology, five words, all about ownership; (2) **property-based testing** reframes "properties" as things you *assert*, which is how I now write requirements; (3) North's property-rights theory explains why "can I exclude others?" is the real test of whether something is a property.
- **Career connection:** properties are the unit in which I specify systems (non-functional requirements), verify algorithms (correctness, termination, complexity), reason about AI models (scaling laws, robustness, calibration) and argue about economics (property rights). One word, four job interviews.

---

## 🔗 Related Topics

- [[vocabulary/05-extraordinary-deep-dive.md]] — scaling laws and OOD generalisation as *properties of a function*
- [[vocabulary/07-labelling-and-temporary-memory.md]] — attributing properties is the cognitive operation behind labelling, in ML and in society
- [[vocabulary/09-commit-and-working-state-terminology.md]] — *commit* carries its own five senses in the same layered way
- [[economics_business/04-network-effects.md]] — property rights and the economics of open access
- [[software_architecture/]] — properties as an alternative to features in a specification
- [SUGGESTED] **Requirement vocabulary** (`requirement`, `constraint`, `invariant`, `assumption`, `precondition`, `postcondition`, `trade-off`) — *why:* this is the family that lets you write a spec that survives review, and it is what technical interviews actually test. → fits `software_architecture/` or `vocabulary/`
- [SUGGESTED] **Correctness vocabulary for algorithms** (`correctness proof`, `loop invariant`, `termination`, `complexity bound`, `worst case`, `precondition`) — *why:* pairs directly with the mathematical-property sense and is high-frequency in algorithms coursework. → fits `ai_ml/` or `vocabulary/`
- [SUGGESTED] **Intellectual property & IP strategy** (`patent`, `trademark`, `copyright`, `trade secret`, `licensing`, `assignment`, `fair use`) — *why:* if you build a bank system or an AI product, the IP question is a real commercial question, not trivia. → fits `economics_business/`

---
*Precision in vocabulary reflects precision in engineering.*
