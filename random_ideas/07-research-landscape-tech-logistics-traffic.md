# Research Landscape — Technology × Logistics × Traffic

**Created Date:** 2026-11-09
**Last Updated:** 2026-11-09
**Tags:** `#research-topic`, `#logistics`, `#traffic`, `#operations-research`, `#its`, `#career`
**Status:** Working document — the landscape is stable; the choice of world is not yet made.
**Related:** [[../personal_evaluations/06-what-i-want-3-year-self-definition]] · [[../personal_evaluations/07-standards-and-review-system]]

---

## 📌 Summary (TL;DR)

This note maps the territory where **technology, logistics and traffic** overlap, so that a research topic can be chosen by evidence rather than by mood. It contains four things:

1. A formula for what makes a topic viable for an undergraduate.
2. Eight "worlds" with a concrete topic, dataset, compute cost and risk for each.
3. A feasibility matrix and three recommended entry points.
4. A protocol for **reading the news** to generate topics — because news supplies the *pain and the numbers*, while papers supply the *mechanism and the method*.

The core claim: **a research topic is a mechanism behind a symptom.** News reports the symptom; the researcher supplies the mechanism and the measurement.

---

## 1. The formula

```
Topic = (World) × (Open question) × (Data you can actually get) × (Method you can actually run)
```

Most student projects die in columns 3 and 4, not column 1. Every world below is scored on five criteria: novelty, data access, compute, portfolio fit, and rebuttal risk.

---

## 2. Eight worlds

### W1 — City logistics & last-mile routing (Operations Research)

**Open question.** Route planning when demand arrives *dynamically* — cancellations, rain, congestion. Static VRP is a planning toy.

| | |
|---|---|
| Sample topic | **Learn-to-optimize for TSP-D**: a GNN/attention constructor learns to pair van and drone, benchmarked against OR-Tools and a GA on synthetic Hanoi-like instances |
| Data | Synthetic (gravity model) + OpenStreetMap — **no company data required** |
| Compute | One small GPU, or CPU for n ≤ 50 |
| Portfolio fit | Very high — `Snake_ANN` is already a "learned heuristic"; `iot_aiot/07` already has MILP + GA + a brute-force solver |
| Risk | L2O is hyped; without a strong OR-Tools baseline the rebuttal is easy |
| Score | Feasibility ★★★★★ · Novelty ★★★ |

### W2 — Urban traffic flow & signal control (ITS)

**Open question.** Adaptive signal control for traffic that is **majority motorbike and lane-less** — almost the entire RL traffic-signal literature assumes cars and lanes.

| | |
|---|---|
| Sample topic | RL adaptive signal control in SUMO, importing one Hanoi intersection from OSM; reward = average waiting time; compared against fixed-time and actuated |
| Data | SUMO + OSM fully open; METR-LA / PEMS-BAY for forecasting comparison |
| Compute | Medium — SUMO runs on a laptop; RL training is slow |
| Portfolio fit | Medium (no RL yet), but `dev_foundation/dsa` and `software_architecture` provide the base |
| Risk | The **sim-to-real gap** is the fatal weakness — but if it is turned into the contribution (calibrating motorbike behaviour in SUMO), it becomes real novelty |
| Score | Feasibility ★★★★ · Novelty ★★★★ (if the Vietnam specificity is kept) |

### W3 — Fleet telematics & visibility (IoT → data)

**Open question.** ETA and anomaly detection from noisy GPS traces; cold-chain compliance.

| | |
|---|---|
| Sample topic | **ETA prediction** on the Porto Taxi dataset (1.7M trajectories) — GBDT baseline + graph features, MAPE sliced by peak hour |
| Data | Porto Taxi, GeoLife, OSM — downloadable |
| Compute | Low |
| Portfolio fit | High — directly extends `UMT-Truck-Blind-Sight` |
| Risk | Very crowded; needs a new angle (e.g. ETA under rain/flooding, a Vietnam-specific condition) |
| Score | Feasibility ★★★★★ |

### W4 — Warehouse & fulfilment automation

**Open question.** Multi-agent path finding (MAPF), order batching, layout optimisation.

- Sample topic: CBS-based MAPF + RL for batching on a grid simulator.
- Data: synthetic. Compute: low–medium. Portfolio fit: lower (no real traffic). Risk: limited local applicability.
- Score: ★★★

### W5 — Freight networks, ports & multimodal (network science + economics)

**Open question.** Resilience of Vietnam's port/logistics network when one node (Cat Lai, Cai Mep) congests.

| | |
|---|---|
| Sample topic | Graph analysis + cascade-failure simulation on Vietnam's container network; AIS data + World Bank LSCI |
| Data | Open AIS (Global Fishing Watch, MarineTraffic samples), LSCI — **harder, needs lead time** |
| Compute | Low (NetworkX) |
| Portfolio fit | High for the **career goal** (AI Engineer in Economics/Business) — the clearest "economics with AI" topic |
| Risk | Data access is the real bottleneck; easy to drift into pure economics without a model |
| Score | Feasibility ★★★ · Career fit ★★★★ |

### W6 — Autonomous driving / V2X / ADAS

**Open question.** Perception under occlusion; cooperative perception.

- Sample topic: extend Blind-Sight into **cooperative perception** (vehicle A sees for vehicle B) on OPV2V.
- Compute: high (real GPU). Risk: too heavy for one semester.
- Score: ★★

### W7 — AI agents / LLM for logistics operations

**Open question.** Exception handling, documents (bill of lading, customs), dispatch triage.

| | |
|---|---|
| Sample topic | LLM agent + MCP tools reading documents → detecting discrepancies → proposing resolution; RAG over tariffs/docs |
| Data | Synthetic + public documents; **the cheapest of the eight** |
| Compute | Low (API) |
| Portfolio fit | Very high — `ai_ml/06` (MCP/function calling) + `ai_ml/03` (RAG) + Joblink backend |
| Risk | **The most crowded** — needs a strict evaluation protocol (RAGAS, golden set) to have value |
| Score | Feasibility ★★★★★ · Novelty ★★ |

### W8 — Sustainability / carbon accounting for freight

**Open question.** Does "green routing" actually reduce emissions once battery manufacturing and the Vietnamese grid mix are included?

- Sample topic: Green VRP + LCA sensitivity; a rebuttal of the GHG claims in `iot_aiot/07 §5`.
- Portfolio fit: high with note 07. Risk: can become an arithmetic exercise with no method.
- Score: ★★★★ if folded into W1.

---

## 3. Feasibility matrix

| World | Data | Compute | Portfolio fit | VN novelty | Total |
|---|---|---|---|---|---|
| W1 Routing / L2O | 5 | 4 | 5 | 4 | **18** |
| W2 Traffic signal RL | 5 | 3 | 3 | 5 | **16** |
| W3 ETA / telematics | 5 | 5 | 4 | 3 | **17** |
| W4 Warehouse | 4 | 3 | 2 | 2 | 11 |
| W5 Port network | 2 | 5 | 4 | 4 | 15 |
| W6 Cooperative perception | 3 | 1 | 4 | 3 | 11 |
| W7 LLM agent ops | 5 | 5 | 5 | 2 | 17 |
| W8 Carbon routing | 4 | 4 | 4 | 3 | 15 |

---

## 4. Recommended entry points

1. **W1 — Learn-to-optimize for TSP-D.** It joins `Snake_ANN` (learning to replace a heuristic) with `iot_aiot/07` (the MILP/GA/brute-force baselines already exist). Both halves of the project are already owned.
2. **W2 — RL signals for motorbike traffic.** The strongest **local novelty**: the literature almost universally assumes lane-based, car-only traffic. If chosen, the real contribution is the *behaviour calibration*, not the RL.
3. **W5 — Vietnam freight network.** The best fit for the "AI Engineer in Economics/Business" goal, but accept 2–3 weeks of data hunting first.
4. **W3 is the fastest first lab**, not necessarily the final topic: one evening to a working baseline.

**Common property of the three:** open data + a clear baseline + a measurable metric. That is the condition for a project that survives week six.

---

## 5. Reading the news to generate a topic

### 5.1 The principle

A news article gives you a **symptom** ("congestion", "port backup", "freight rates up"). A research topic is the **mechanism** behind the symptom, measured by a number.

From every article, extract three things:

1. **The number** (%, hours, containers).
2. **Who bears the pain** (driver, shipper, port, state).
3. **A hypothesised mechanism** — usually *not* stated in the article. This is the part you add, and the part that turns news into research.

### 5.2 Sources, in three tiers

**Tier 1 — Vietnamese press (to find the local problem)**

| Source | What to take |
|---|---|
| VnExpress (Business, Transport) | Events + fast numbers |
| CafeF, Vietstock | Logistics company figures (GHTK, GHN, Viettel Post, HAH, Gemadept) |
| TheLEADER, Nhip Cau Dau Tu | Infrastructure, ports, expressways |
| Vietnam Logistics Business Association (VLA) | Industry reports, logistics cost |
| Ministry of Transport, General Department of Customs, GSO | **Primary data** — always trace back here |
| Annual reports of GHTK/GHN/Viettel Post/HAH | Real operational problems, with numbers |

**Tier 2 — International press (to learn the world standard and what is currently salient)**

- FreightWaves, The Loadstar, Lloyd's List, JOC — sea and road freight
- Supply Chain Dive — operational case studies
- Reuters / Bloomberg — macro numbers, supply-chain disruption
- World Bank LPI, UNCTAD LSCI, ADB — rankings and regional comparison

**Tier 3 — Technical sources (to check whether the technology is ready)**

- IEEE Spectrum, NVIDIA/Google Research blogs
- arXiv (titles + abstracts only)

> Rule: **Tier 1 to choose the problem, Tier 3 to check whether it is solvable.** Never read Tier 3 first.

### 5.3 Cadence

**Daily (15 minutes) — log only, no analysis**

| Date | Source | Headline | Number | Who bears the pain | Hypothesised mechanism | Open data? |
|---|---|---|---|---|---|---|

One row per article. No paragraphs.

**Saturday (20 minutes) — cluster**

Three articles from different sources pointing at the same mechanism is a strong signal. A single article is noise — news reports events, not distributions.

**Sunday (45 minutes) — trace the primary number**

Pick the most interesting number and find the original report behind it. This is the step 90% of people skip, and it is where topics come from. Example: "logistics costs are 16–17% of GDP" — defined how? Does it include what? Is it comparable with Thailand? **The ambiguity of the number is itself a topic** (a measurement problem).

### 5.4 From article to research question

```
In the context of [X in Vietnam],
  how does [mechanism Y] affect [metric Z],
  and can [method M] improve Z?
```

| News item | Mechanism (inferred) | Research question | World |
|---|---|---|---|
| Cat Lai congestion, 4–6 h waits | No truck appointment system → arrival spikes at peak | "How many waiting hours can truck appointment scheduling with AIS + GPS remove?" | W5 |
| Hanoi pilots smart signals | Fixed-time control cannot adapt to motorbikes | "How many seconds of waiting does RL signal control remove in mixed, lane-less traffic?" | W2 |
| 2-hour delivery promises | Bad ETA → failed delivery → cost | "How much does ETA prediction under rain/flooding improve MAPE?" | W3 |
| Freight rates up after Red Sea disruption | Loss of route diversity → one node propagates | "How resilient is Vietnam's freight network when one route is cut?" | W5 |
| Camera-based traffic fines | Edge ANPR under bandwidth limits | "What accuracy is traded for quantisation on an edge device?" | W6 |

Note: the **mechanism column does not exist in the article**. It is the researcher's contribution.

### 5.5 Pitfalls

- **No number** → an opinion, skip it.
- **Number without a source** → usually a vendor press release ("AI will revolutionise logistics").
- **Number without a definition** → the most dangerous; trace the original report.
- **Single-event article** → cannot infer a distribution; use it only to *suggest* a hypothesis.
- **Technology-praise article** → read the deployment-cost and who-pays paragraphs, usually at the end.

### 5.6 Four-week plan

| Week | Work | Output |
|---|---|---|
| 1 | One article/day, log it | 5–7 rows, 2–3 clusters |
| 2 | Trace the primary number for the two strongest clusters | Two half-page problem sheets |
| 3 | For each cluster, find three existing papers/solutions | What is solved, what is not |
| 4 | Choose one, write a one-page proposal | The topic |

Week 3 is where journalism meets academia: **news says the problem exists; papers say whether anyone has solved it.** Solved elsewhere but not applied in Vietnam → an *applied* topic (valid). Not solved under Vietnamese conditions → a *research* topic (stronger).

---

## 6. Discovery protocol — how to start on your own

### 6.1 Four mini-labs, one evening each

The goal is to choose a world **by hand**, not by feeling.

| Lab | World | Data (no login) | Dumb baseline | Metric |
|---|---|---|---|---|
| A | W1 | None — reuse the brute-force solver in `iot_aiot/07 §6` | Nearest-neighbour + 2-opt on random n = 10..20 | gap % vs brute force (n ≤ 10) |
| B | W2 | SUMO + `osmWebWizard.py` (one Hanoi intersection) | Fixed-time signal, 1 simulated hour | average waiting time |
| C | W3 | NYC TLC parquet | LightGBM on `distance` + `hour` | MAE (minutes) |
| D | W7 | One public tariff/customs PDF | Naive RAG + 20 self-written questions | % correct |

```bash
# Lab C — NYC TLC, ~50 MB, no account
curl -O https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet

# Lab B — SUMO
pip install eclipse-sumo
python $SUMO_HOME/tools/osmWebWizard.py

# Lab A — Vietnam OSM extract if a real map is needed
# https://download.geofabrik.de/asia/vietnam-latest.osm.pbf
```

**Decision gate after four evenings:** which lab made you *want to keep fixing bugs at 11 p.m.*? That is the predictor, better than any scoring matrix.

### 6.2 Reading ladder — exactly three documents per world

Read in reverse order of difficulty:

1. **Dataset card / README** (15 min) — schema, size, who built it, what they did *not* measure.
2. **Canonical paper** — Introduction + Problem Formulation; skip proofs.
3. **Survey, taxonomy table only.**
4. **One SOTA paper from 2024–2026 — Limitations / Future Work only.**

| World | Canonical | Survey | SOTA to read for limitations |
|---|---|---|---|
| W1 | Murray & Chu (2015), *The flying sidekick TSP* | Bengio, Lodi & Prouvost (2021), *ML for Combinatorial Optimization* | Any "neural TSP-D / VRP" paper, 2024–2026 |
| W2 | Wei et al. (2019), *DRL for Traffic Signal Control: A Survey* | Li et al. (2018), DCRNN | Any RL signal-control paper containing "mixed traffic" or "heterogeneous" |
| W3 | Uber Engineering, *DeepETA* | Any "travel time estimation" survey | Any ETA paper, 2025–2026, "adverse conditions" section |

### 6.3 Gap table

Pick five recent papers; read only three parts: the last paragraph of the Introduction, Limitations, Future Work.

| Paper | Year | Problem | What they did | What they did **not** do (verbatim) | Feasible with my data? |
|---|---|---|---|---|---|

If three papers say "X is not considered" → **X is the topic**. If five papers point in five directions → the world is too broad; narrow it.

### 6.4 One-page proposal, before any note is written

```
Problem:  ...
Gap:      ... (quoted from the gap table)
Method:   ... (baseline + what I add)
Data:     ... (dataset, sample size, downloaded yet?)
Metric:   ... (one primary, one secondary)
Risk:     ... (what would kill the project)
Timeline: ... (which week produces which artifact)
```

If the `Data` line cannot answer "is it downloaded yet?", the project has not started.

---

## 7. Research Journey

- **Why:** the goal was a research topic at the intersection of technology, logistics and traffic, tied to the AI Engineer career path — but there was no map of the territory, only a list of interests.
- **Struggle:** "explore the world" was ambiguous. It first meant *domains* and then meant *reading the news* — and the two turned out to be the same protocol seen from two ends: news generates the problem, papers check whether it is solved.
- **Aha moment:** news reports **symptoms**, research needs **mechanisms**. The mechanism column is empty in every article, which means it is exactly where a student can still contribute — not by reading more, but by inferring what the article did not say.
- **Career link:** the choice of world is a six-month bet on the three-year self. That is why this note lives beside `personal_evaluations/06` and `07` — a topic should be chosen for *ownership potential*, not only for academic interest.

---

## 8. Open questions

- [ ] In the target city, is demand density high enough for the continuous approximation to be a valid screening model, or is demand too clustered?
- [ ] What is the real price of C2 unavailability in a routing model — a hard arc deletion or a probabilistic penalty?
- [ ] For n = 100, what is the integrality gap of the formulation in `iot_aiot/07 §2.2` with HiGHS, and does a warm start from a GA close it?
- [ ] Can a learned dispatch policy beat ALNS on re-optimisation latency without violating the safety case?
- [ ] Does the "logistics cost = 16–17% of GDP" figure survive a definition check against the original report?

---

## 9. 🔗 Related Topics

- [[../iot_aiot/07-uav-logistics-terminology-and-optimization]] — TSP-D, VRP-D, MILP, GA, GHG; the technical base for W1 and W8
- [[../iot_aiot/05-uav-iot-stack-and-c2-links]] — the physical system that must fly the route
- [[../iot_aiot/06-uav-ai-perception-and-data-pipeline]] — data quality and fleet learning
- [[../ai_ml/05-mlops-lifecycle-and-deployment-architecture]] — how a model enters a dispatch loop safely
- [[../dev_foundation/dsa/04-complexity-analysis-toolkit]] — why "NP-hard" is a statement about worst case, not about your instance
- [[../personal_evaluations/06-what-i-want-3-year-self-definition]] — the identity layer behind the topic choice
- [[../personal_evaluations/07-standards-and-review-system]] — how to measure whether the chosen topic is progressing

### SUGGESTED

- **[SUGGESTED] Column generation and branch-and-price for VRP-D** — new `operations_research/` folder. The technique behind state-of-the-art fleet routing; the MILP in `iot_aiot/07` stops scaling around n ≈ 50.
- **[SUGGESTED] Stochastic and dynamic VRP** — demand uncertainty, cancellations, weather; identified as the open frontier in recent reviews.
- **[SUGGESTED] Queueing theory for fleet sizing** — utilisation, waiting time, the M/M/c intuition behind "how many vehicles per operator". Fleet sizing, not routing, is where most of the money is.
- **[SUGGESTED] Truck appointment scheduling at Vietnamese ports** — a W5 topic that is both researchable and sellable.
