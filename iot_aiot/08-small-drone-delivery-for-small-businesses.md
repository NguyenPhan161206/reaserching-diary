# Small Drone Delivery for Small Businesses — Feasibility, Unit Economics and the Density Gap

**Created Date:** 2026-11-09
**Last Updated:** 2026-11-09
**Tags:** `#UAV`, `#last-mile`, `#unit-economics`, `#logistics`, `#BVLOS`, `#SME`
**References:** See §7 — every figure is tagged with a source tier (A = peer-reviewed, B = official/government, C = industry report, D = trade press, E = unverified)
**Status:** Draft — unit-economics inputs are secondary benchmarks, not a measured operation. Regulation section verified 2026-11-09 and will rot.
**Difficulty:** Intermediate → Advanced
**Prerequisite notes:** [`05`](05-uav-iot-stack-and-c2-links.md) (C2 link, BVLOS, failsafe) · [`07`](07-uav-logistics-terminology-and-optimization.md) (TSP-D/VRP-D, cost per parcel)

---

## 📌 Key Takeaways (TL;DR)

- **"Small business buys a drone" is the wrong question and the answer to it is no.** Fixed costs (hub, remote supervision, airworthiness compliance, maintenance reserve) are larger than a small retailer's entire delivery revenue. The only viable shape is **drone-delivery-as-a-service (DaaS)**: the SME buys *drops*, not aircraft.
- **The dominant cost term is not the aircraft — it is supervision labour divided by utilisation.** In the model below, at 8 deliveries/day the labo(u)r share is **89.9 %** at 1 drone per observer and **30.7 %** at 20 drones per observer, with total cost falling from **\$34.77 to \$5.08** per delivery *using the identical aircraft*.
- **Drone economics are gated by a regulatory parameter, not an engineering one.** The number of aircraft one remote pilot may supervise is a rule, not a battery. That is why this is an economics note wearing an IoT note's clothes.
- **The research gap is demand density, not technology.** Every published model assumes dense e-commerce demand (Amazon/Walmart/Zipline). Nobody models the sparse, lumpy, weekday/Saturday-unstable demand of a *cluster* of small businesses — which is exactly the case where the utilisation assumption collapses.
- **Payload reality caps the use case set.** "Small drone" in practice = **1–4 kg, 10–16 km** (Amazon MK30 5 lb/7.5 mi, Zipline P2 8 lb/10 mi, Wing ~2.5 lb). Food, pharmacy, urgent spares and documents are in scope; groceries and general parcels are out.

---

## 🧠 Detailed Notes

### 1. The reframe — why the naive question fails

The intuitive framing is: *a small shop wants faster delivery → buy a delivery drone.* Run the fixed costs and it dies immediately:

| Fixed or quasi-fixed cost | Order of magnitude | Source tier |
|---|---|---|
| Non-recurring engineering / integration (fleet, telemetry, dispatch) | not representable at SME scale | — |
| Small launch/recovery pad, build | \$200k–400k | D |
| Small pad, annual operation | \$600k–900k/yr | D |
| Full hub | \$6–7M capex + \$15–17M/yr opex | E (needs verification) |
| Aircraft, 2–4 unit pilot | \$40k–120k | D |
| Aircraft, 8–20 unit hub + spares | \$180k–600k | D |

A neighbourhood retailer doing 40 deliveries a day at a \$2 margin does not have a \$600k/yr cost line. The arithmetic does not need to be precise to be decisive — it is off by **two orders of magnitude**, not 20 %.

So the unit of decision is not *ownership* but *procurement*:

> **Research question (the one worth answering):** at what order density does a shared-hub DaaS drop become cheaper than a motorbike courier for a small business, and can a *cluster* of small businesses (a market street, a specialised trade street, an industrial estate, a hospital campus) reach that density?

This reframe is what makes the topic a contribution rather than a business-plan exercise, and it is what note [`07`](07-uav-logistics-terminology-and-optimization.md) does **not** already cover — 07 owns the routing model, this note owns the demand side that feeds it.

### 2. What "small drone" actually means

| Platform | Payload | Range | Note |
|---|---|---|---|
| Amazon MK30 | **5 lb ≈ 2.3 kg** | 7.5 mi ≈ 12 km | FAA type-certified; ~16,000 deliveries total by Feb 2026 (tier D) |
| Zipline P2 | **8 lb ≈ 3.6 kg** | 10 mi ≈ 16 km radius | P2 targets restaurant/retail; Zipline reports >2M deliveries, 130M autonomous miles by Jan 2026 (tier C/D) |
| Wing (some craft) | **~2.5 lb ≈ 1.1 kg** | — | Alphabet subsidiary |
| DJI FlyCart 30 (reference, heavy-lift) | 30 kg | 16 km | Not a last-mile parcel aircraft; useful as the payload ceiling |

Consequences for the SME use case:

- ❌ groceries, bottled water, white goods, bulk e-commerce — payload kills them
- ✅ **food and beverage** (a pizza box clears every current aircraft), **pharmacy**, **urgent spare parts**, **documents**, **samples**
- ✅ the real selection criterion is not weight but **value of time**: what does 20 minutes instead of 60 minutes buy the customer?

> **Design rule:** the viable SME segment is *high value-of-time, low mass, short radius*. Any note or pitch that omits the payload class is not describing this technology.

### 3. Mechanism — where the money actually goes

Cost per completed delivery:

$$
C = \underbrace{\frac{K}{d \cdot D \cdot L}}_{\text{capex amortisation}} + \underbrace{\frac{K \, m}{d \cdot D}}_{\text{maintenance}} + \underbrace{\frac{K_b}{N_b}}_{\text{battery cycles}} + \underbrace{\frac{E_b}{n_c} p_e}_{\text{energy}} + \underbrace{\frac{w \cdot H}{N_o \cdot d}}_{\text{supervision}} + \underbrace{\frac{F}{Q}}_{\text{hub}}
$$

with $K$ aircraft capex, $d$ deliveries per aircraft per day, $D$ operating days per year, $L$ life in years, $m$ maintenance rate, $K_b$ pack cost, $N_b$ cycles to end-of-life, $E_b$ usable pack energy, $n_c$ deliveries per charge, $p_e$ electricity price, $w$ observer wage, $H$ supervised hours per day, $N_o$ **aircraft per observer**, and $F/Q$ hub opex over total hub throughput.

Two structural facts fall out of that formula:

1. **$C$ scales as $1/d$ and as $1/N_o$.** Everything else is second-order. A better airframe buys you a 30 % capex improvement; a 20× supervision ratio buys you a **7×** total-cost improvement.
2. **$F/Q$ is not an aircraft property at all.** It is an *aggregation* property — which is precisely why a single SME cannot host a hub and a cluster might.

#### 3.1 Executed model

Runnable, dependency-free. Inputs are deliberately conservative and all are listed at the top so the reader can disagree with one number instead of the whole output.

```python
CAPEX = 20_000.0            # USD per aircraft, small multirotor class
LIFE_YEARS = 4.0
DAYS_PER_YEAR = 300.0       # weather + maintenance downtime
MAINT_RATE = 0.05           # fraction of capex per year (props, motors, airframe)
BATTERY_COST = 300.0        # USD per pack
BATTERY_CYCLES = 300.0      # cycles before 80% capacity
BATTERY_KWH = 0.6           # usable pack energy
DELIVERIES_PER_CHARGE = 4.0
KWH_PRICE = 0.15            # USD/kWh
OBSERVER_HOURLY = 25.0      # USD/h, one remote supervisor
HOURS_PER_DAY = 10.0


def cost_per_delivery(deliveries_per_day, drones_per_observer, capex=CAPEX):
    annual = deliveries_per_day * DAYS_PER_YEAR
    life_deliveries = annual * LIFE_YEARS

    capex_term = capex / life_deliveries
    maint_term = capex * MAINT_RATE / annual
    battery_term = BATTERY_COST / BATTERY_CYCLES
    energy_term = (BATTERY_KWH / DELIVERIES_PER_CHARGE) * KWH_PRICE
    labour_term = OBSERVER_HOURLY * HOURS_PER_DAY / (drones_per_observer * deliveries_per_day)

    return {
        "capex": capex_term, "maintenance": maint_term, "battery": battery_term,
        "energy": energy_term, "labour": labour_term,
        "total": capex_term + maint_term + battery_term + energy_term + labour_term,
    }
```

Executed output (`python3 drone_model.py`, 2026-11-09):

```
 del/day  1 observer:N   capex   maint   batt  energy   labour    TOTAL
-----------------------------------------------------------------------
       2             1    8.33    1.67   1.00   0.022   125.00   136.02
       2             5    8.33    1.67   1.00   0.022    25.00    36.02
       2            20    8.33    1.67   1.00   0.022     6.25    17.27
       4             1    4.17    0.83   1.00   0.022    62.50    68.52
       4             5    4.17    0.83   1.00   0.022    12.50    18.52
       4            20    4.17    0.83   1.00   0.022     3.12     9.15
       8             1    2.08    0.42   1.00   0.022    31.25    34.77
       8             5    2.08    0.42   1.00   0.022     6.25     9.77
       8            20    2.08    0.42   1.00   0.022     1.56     5.08
      20             1    0.83    0.17   1.00   0.022    12.50    14.52
      20             5    0.83    0.17   1.00   0.022     2.50     4.52
      20            20    0.83    0.17   1.00   0.022     0.62     2.65
      50             1    0.33    0.07   1.00   0.022     5.00     6.42
      50             5    0.33    0.07   1.00   0.022     1.00     2.42
      50            20    0.33    0.07   1.00   0.022     0.25     1.67

Break-even: deliveries/day needed to reach a target cost per delivery
  target $2.00/delivery at 1:5  observer ->   71.6 deliveries/day/aircraft
  target $2.00/delivery at 1:20 observer ->   33.2 deliveries/day/aircraft
  target $3.00/delivery at 1:5  observer ->   35.4 deliveries/day/aircraft
  target $3.00/delivery at 1:20 observer ->   16.4 deliveries/day/aircraft
  target $5.00/delivery at 1:5  observer ->   17.6 deliveries/day/aircraft
  target $5.00/delivery at 1:20 observer ->    8.2 deliveries/day/aircraft

Sensitivity: labour share of total cost at 8 deliveries/day
  1:1  observer -> labour =  89.9% of $34.77
  1:2  observer -> labour =  81.6% of $19.15
  1:5  observer -> labour =  64.0% of $9.77
  1:10 observer -> labour =  47.0% of $6.65
  1:20 observer -> labour =  30.7% of $5.08
```

**Why these outputs are credible:** the 1:1 figure (\$34.77 at 8 deliveries/day) lands on top of DroneUp's reported **~\$30 per delivery** (tier D), and the 1:20 figure (\$5.08) lands inside PwC's **\$6–25** current-industry band (tier C) and near Amazon's claimed **\$63 → target reduction** trajectory (tier D, and the \$63 is *wildly* off the model — treat it as a programme-cost artefact, not a unit cost). Three independent estimates from different methods and different companies agreeing within a factor of ~1.5 is the strongest signal available in this literature.

**Why these outputs are still not a forecast:** the model has no weather-cancellation term, no failed-delivery term, no hub term (`F/Q` is excluded — set it explicitly for a specific site), no insurance, no airworthiness-certification amortisation, and no pilot-training cost. It is a *sensitivity instrument*, not a price.

### 4. The density gap — the actual research contribution

Substituting the model into the three demand regimes:

| Regime | deliveries/day/aircraft | model \$/drop (1:20) | who this describes |
|---|---|---|---|
| Dense e-commerce (published literature) | 50+ | **\$1.67** | Amazon, Walmart, Wing — 5–10 mi radius, purpose-built hubs |
| Small-business cluster (unmodelled) | 8–20 | **\$2.65–5.08** | a market street, a trade estate, a hospital campus |
| Single small business (unmodelled) | 2–4 | **\$9.15–17.27** | one shop, its own pad, its own supervision |

The literature occupies the top row and *asserts* it as the general case. Walmart's **1 million drone deliveries across 66 stores in 4 states / 5 metros by May 2026** (tier D) is evidence for **economies of density**, not for small-business delivery — 1M drops over 66 hubs is ~15k drops/hub, which is the top row, not the bottom.

Three things break when you move down the table, and none of them are modelled anywhere I could find:

1. **Utilisation is not a design choice, it is a demand property.** At $d=8$ with 1:20 supervision, \$5.08/drop only beats a motorbike courier if the motorbike courier is expensive — and in most of Asia it is *very* cheap. The competitive question is therefore **local wage arbitrage**, not drone cost.
2. **Demand is lumpy, not just small.** A market street has lunch peaks and dead afternoons. Variance, not the mean, decides how many aircraft you buy — and capex is paid on the peak.
3. **The hub is a commons, and commons need a governance model.** A shared hub must answer: who pays the \$600–900k/yr, who owns the landing rights, who eats the cost of an idle Tuesday. This is a **platform/coordination problem** (see SUGGESTED note below), not an aerospace one.

### 5. Regulation as an economic parameter

The supervision ratio $N_o$ is not a technology limit — it is a **licence condition**. That makes the regulator a first-order determinant of unit economics, which is an unusual and under-appreciated claim.

**United States — Part 108 (BVLOS), verified 2026-11-09:**

| Event | Date |
|---|---|
| NPRM published, docket FAA-2025-1908, 90 FR 38212 | 2025-08-07 |
| Comment period closed (then reopened on conspicuity/right-of-way, closed again) | 2025-10-06 / 2026-02-11 |
| Statutory deadline (FAA Reauthorization Act 2024 §930) — **missed** | 2026-01-16 |
| EO 14307 "Unleashing American Drone Dominance" 240-day deadline — **missed** | 2026-02-01 |
| Final rule sent to OIRA | 2026-07-10 |
| **Status at time of writing: still pending review, not published** | 2026-11-09 |
| FAA stated hope (Commercial UAV Expo) | publication by end-2026 |

Until the final rule is effective, US delivery flies under **Part 135 certificates plus per-operation exemptions** — i.e. exactly the case-by-case regime that makes $N_o$ small and cost high. The FAA's July 2026 programmatic environmental assessment (PEA/FONSI/ROD) supports Part 135 package delivery *and* estimates **Part 108 operations may begin 1–2 years after rule issuance** once consensus standards exist. Fifteen states sued the FAA over that PEA in October 2026.

**Europe:** open category stays **<25 kg, VLOS, 120 m**; SORA 2.5 folded into the June 2026 Easy Access Rules revision; NPA 2026-103 would drop the registration threshold to 100 g, but its own timeline puts the complete framework **no earlier than 2031**.

**Vietnam:** I could not establish an equivalent BVLOS framework in this session — recorded as an open question rather than asserted. If a dedicated Vietnamese regulatory note is written later, this is the anchor to check.

> **Why this matters for the note's thesis:** any SME business case built on 1:20 supervision is a *bet on a rule that has not been published*. That is a different kind of risk from "the battery is not good enough yet", and it should be underwritten differently.

### 6. Comparison — for an SME choosing between modes

| Criterion | Drone (DaaS, shared hub) | Motorbike courier | Electric cargo bike | Van (own) |
|---|---|---|---|---|
| Cost per drop, small volume | \$5–17 | **\$0.5–2** (low-wage market) | \$1–3 | \$9–11 |
| Cost per drop, high density | **\$1.67** | \$1–2 | \$1–2 | \$5–9 |
| Payload | **1–4 kg** | 10–20 kg | 40–80 kg | 500+ kg |
| Radius | 10–16 km | city-wide | 5–10 km | city-wide |
| Time for 5 km | **~8 min** | 15–25 min | 20–30 min | 20–35 min |
| Weather sensitivity | **high** | low | medium | low |
| Regulatory drag | **high and moving** | low | low | low |
| Fixed cost to the SME | **zero** (buys drops) | zero | low | high |

Read the table honestly: **the drone loses on cost at every volume an individual small business can generate, and wins only on time and on payloads it cannot carry anyway.** The niche is narrow and it is defined by value-of-time, not by cost.

### 7. Evidence inventory — with tiers

Tier **A** = peer-reviewed · **B** = government/official · **C** = industry/consultancy report · **D** = trade press citing a company · **E** = unverified, treat as a hypothesis.

| Claim | Figure | Source | Tier | Caveat |
|---|---|---|---|---|
| Current cost/drop | ~\$13.50 | Enayati, Univ. of Missouri–St. Louis, 2025 | A | single-market estimate |
| Conventional vehicle comparison | ~\$2/drop | same | A | low-wage market |
| Industry current band | \$6–25 → ~\$2 at scale | PwC | C | global projection, not a measurement |
| 1:1 vs 1:20 supervision | \$13.50 vs \$1.80; labour up to 95 % | Net Zero Insights, 2026-08 | C | techno-economic benchmark |
| Amazon internal, 2025 | ~\$63/drop | Business Insider via eMarketer | D | programme cost, not unit cost |
| DroneUp | ~\$30 → target <\$7 | eMarketer | D | |
| \$20k aircraft, 8 drops/day → \$3 for 10 % IRR | \$3/drop | Thunder Said Energy, 2026-06 | C | paid data-file |
| Hub capex/opex | \$200–400k / \$600–900k/yr (small) | uasfeed aggregation | D/E | **verify before citing** |
| Walmart scale | 1M drops, 66 stores, 4 states, 5 metros by May 2026 | Walmart announcement via trade press | D | |
| Zipline scale | >2M deliveries, 130M autonomous miles by Jan 2026 | company claims | D | |
| Systematic review of the field | payload/range/weather constrain viability; truck–drone hybrids are the scalable path | *Drones in last-mile delivery: a systematic literature review*, 2025 | A | ⚠️ not read in full — **do not cite until read** |
| India unit economics | drone ~₹100/km vs ~10× cheaper e-2-wheeler | IJSDR 2026 | E | low-quality venue; use only to illustrate the *direction* of the gap |

**Repo discipline:** tier D/E rows stay in the note as *navigation aids*, not as evidence. If this note is ever promoted from Draft to Review, every D/E row must either be upgraded to A/B/C or deleted.

### 8. Research design that would actually work

- **Unit of analysis:** the *completed delivery*, not the firm. Firm-level data will not exist and proxies will be circular.
- **Design:** cost model + break-even sensitivity (what this note does), followed by a **discrete-choice / stated-preference survey** of SMEs to estimate willingness to pay, then a **cluster-level density simulation** on real geodata (how many drops/day can a 1 km² trade street generate?).
- **Densities you can obtain without a partner:** OpenStreetMap POIs per km² + published order volumes per sector. This is the honest path for a student project — it produces a *feasibility envelope*, clearly labelled as a model.
- **Threats to validity to state up front:** (i) imported benchmark costs vs local wages — the single largest error source; (ii) demand assumed stationary when it is peaked; (iii) regulation assumed final when it is pending; (iv) no failed-delivery cost.
- **The result that would be publishable as a student paper:** a *density threshold curve* — minimum drops/day/km² required for DaaS to undercut the local courier, parameterised by local wage. Nobody has drawn that curve for a low-wage market.

## 🔗 Tech Cross-References

- **[`07` UAV logistics — TSP-D/VRP-D](07-uav-logistics-terminology-and-optimization.md)** — this note supplies the *demand* that 07's routing model consumes. 07's "cost per parcel" parameter is exactly the $C$ computed here; wire the two together and the model becomes a route-level one.
- **[`05` UAV IoT stack, C2 and BVLOS](05-uav-iot-stack-and-c2-links.md)** — the C2 link budget sets the *feasible arc set*, which is why a 16 km radius is a link-and-regulation claim, not a straight-line one. The offboard ≥2 Hz liveness rule is also why 1:20 supervision is hard: one operator watching 20 video feeds is a human-factors problem before it is a software one.
- **[`06` UAV perception & data pipeline](06-uav-ai-perception-and-data-pipeline.md)** — the GSD argument explains why the aircraft cannot be made much smaller *and* autonomous at the same time: at 100 m a person is ~16 px, so sense-and-avoid needs resolution, and resolution needs mass and power.
- **[`../economics_business/05-urban-car-restriction-strategies.md`](../economics_business/05-urban-car-restriction-strategies.md)** — the same instrument-choice problem in a different medium: quantity rationing (licence plates, quotas) vs price rationing (tolls). COE is a *quantity* instrument on ownership; congestion charging is a *price* instrument on use. DaaS pricing faces the identical design choice (per-drop vs subscription vs surge).
- **[`../dev_foundation/databases/README.md`](../dev_foundation/databases/README.md)** — a hub is a dispatch system: order intake, aircraft state, battery cycles and delivery proof are a time-series + transactional schema problem, and idempotency on retry is what stops a customer being charged twice for one drop.

## 🧪 Practice Tasks

> No solutions provided. Do these, then ask for a review.

- [ ] **Task 1 — invert the model.** Add a weather-cancellation term (assume 15 % of scheduled days are lost) and a failed-delivery term (3 % of attempts, cost = one full sortie). Re-run the 8/day × 1:20 case. State the new \$/drop and which term grew fastest.
- [ ] **Task 2 — find your local threshold.** Swap `OBSERVER_HOURLY` and add a `COURIER_COST_PER_DROP` parameter for a market you can source (e.g. Grab/Lalamove/Be in a Vietnamese city). At what deliveries/day does DaaS break even *against that local courier*, and is that density plausible for one ward?
- [ ] **Task 3 — build the density envelope.** Pick a 1 km² area you know. Query OpenStreetMap for restaurant + pharmacy + clinic POIs. Assume 2 drops/POI/day. Does the area clear 20 drops/day/aircraft? What radius must the hub serve to reach 33 drops/day (the \$2 break-even at 1:20)?
- [ ] **Task 4 — audit the regulation claim.** Read the actual Part 108 NPRM section on supervision ratios (docket FAA-2025-1908). What $N_o$ does it propose, and is it performance-based or prescriptive? This task exists because the entire economic argument above hinges on a rule.
- [ ] **Task 5 — kill the note.** Find one published source that models SME-scale (5–20 drops/day) drone delivery and report the \$/drop. If it exists, this note's "gap" claim is wrong and must be retracted.

## 📝 Research Journey

- **Why:** the question arrived as "what if we researched small drones delivering for small shops?" — and the interesting part turned out to be that the naive version of the question has a *decisive* answer (no, they cannot afford it), which forces the real question (density) into view.
- **Struggle:** separating cost-per-delivery from programme-cost. Amazon's \$63/drop and PwC's \$6–25/drop are not measurements of the same object, and trade press reports them in the same sentence. The only defence is to build the cost stack yourself and see which term moves.
- **Aha moment:** the aircraft is not the expensive part. In my own model the airframe is **\$2.08 of a \$34.77 delivery** at 8 drops/day — 6 %. The expensive part is a human watching a screen divided by how many screens they are allowed to watch. That reframes the whole field from *aerospace* to *regulatory economics*.
- **Career link:** this is the shape of AI-engineering work I actually want — the model is a decision instrument, the binding constraint is a policy parameter, and the honest output is a break-even curve with the uncertainty stated, not a demo with a highlight reel.

## 🔗 Related Topics

- [`07`](07-uav-logistics-terminology-and-optimization.md) — routing and cost-per-parcel: the supply side of this note's demand side
- [`05`](05-uav-iot-stack-and-c2-links.md) — BVLOS, C2, failsafe: why the range number is a regulation number
- [`../economics_business/05-urban-car-restriction-strategies.md`](../economics_business/05-urban-car-restriction-strategies.md) — price vs quantity instruments; the same choice appears in DaaS pricing
- [`../economics_business/04-network-effects.md`](../economics_business/04-network-effects.md) — a shared hub is a two-sided market; without density there is no network to speak of
- [SUGGESTED] **UAV energy & endurance model** — already the top item in this folder's backlog, and note 08 cannot be validated without it: $d$ (drops/day) is bounded by hover power, pack energy and charge turnaround. Write it *before* promoting this note past Draft.
- [SUGGESTED] **DaaS platform economics** (`economics_business/`, Template B) — per-drop vs subscription vs surge pricing; who bears idle-Tuesday cost; hub as a commons. This is the business half of the same question.
- [SUGGESTED] **Drone acceptance and airspace NIMBY** (`psychology_society/`, Template C) — 15 US states sued the FAA over the delivery PEA in Oct 2026 and Amazon hubs drew local opposition in Massachusetts and North Myrtle Beach. Adoption is a social variable, not only a technical one.

## 🤔 Open Questions

- [ ] What $N_o$ does Part 108 actually propose, and does it bind the operator or the aircraft? (Blocks the entire economic argument.)
- [ ] Is there **any** published model of drone delivery at 5–20 drops/day/firm? If yes, this note's gap claim is false.
- [ ] What is the Vietnamese regulatory baseline for BVLOS commercial delivery, and is there a licensing path at all? Not established in this session.
- [ ] Does a shared hub change the *legal* operator? If an SME buys drops from a hub, who holds the Part 135/108 certificate, and does that change the SME's liability?
- [ ] Do the \$200–400k pad and \$600–900k/yr opex figures survive contact with a primary source? They are tier E and they carry a lot of the "SME cannot afford it" argument.

---
*The aircraft is cheap. The licence to stop watching it is not.*
