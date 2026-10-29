# UAV Logistics: Terminology, Routing Models and Sustainability Metrics

**Created Date:** 2026-10-30
**Last Updated:** 2026-10-30
**Tags:** `#uav`, `#drone-delivery`, `#tsp-d`, `#vrp-d`, `#milp`, `#genetic-algorithm`, `#continuous-approximation`, `#ghg`, `#operations-research`, `#last-mile`
**References:** Murray & Chu, *Transp. Res. C* 54:86–109 (2015) · Ha et al., *Transp. Res. C* 86:597–621 (2017) · Bouman et al., *Networks* 72(4):528–542 (2018) · Otto et al., *Networks* 72(4):411–458 (2018) · Daganzo, *Transp. Res. B* (1984) · Wang, Poikonen & Golden, *Optim. Lett.* 11:679–697 (2017) · Han & Shen, *Appl. Comput. Intell.* (2025) · Almodather & Mohamed, *Drones* 9(6):413 (2025) · Bao et al., *Drones* 9(1):54 (2024) · Serve Robotics investor material (2026)
**Status:** Review
**Difficulty:** Advanced

---

## 📌 Key Takeaways (TL;DR)

- **TSP-D and VRP-D are not "TSP with a drone bolted on".** The coupling constraint — the truck must be at the recovery node when the drone lands — makes the problem a *synchronisation* problem, and that is where the combinatorial difficulty lives.
- **MILP is the language of exact formulations; it is not an algorithm.** The formulation is a model; branch-and-bound (plus cutting planes and lazy constraints) is the algorithm. Writing a correct MILP is 20% of the work; making it solvable at realistic size is the other 80%.
- **Continuous approximation (CA) is the cheapest useful model in logistics.** It replaces individual customers with a density and gives closed-form cost estimates in seconds — perfect for *screening* before you spend a week solving a MILP.
- **Genetic algorithms are a last resort, not a default.** They shine on large, non-linear, multi-objective, badly-behaved instances; they give no optimality proof and their performance is dominated by encoding and repair design.
- **GHG numbers for drone delivery span three orders of magnitude** (roughly 0.6–1,786 g CO₂e per parcel in the reviewed literature). The spread is not noise — it is *system boundary, grid intensity, payload, and demand density*. Any claim without those four parameters is marketing.
- **The C2 link is a constraint, not a footnote.** Link availability defines which arcs are feasible (corridor geofences), and latency defines whether teleoperation is a fallback at all. A routing model that ignores this produces routes the fleet cannot fly.
- The academic frontier in 2025–2026 is explicit: **data-driven prediction + learning-based optimisation inside dynamic VRP**, because static models with weak stochasticity do not survive real operations.

---

## 🧠 Detailed Notes

### 1. The Terminology, Properly Defined

> The Vietnamese glosses in the table are the user-supplied translations from the source list, kept verbatim for vocabulary tracking.

| Acronym | Expansion | Vietnamese gloss | Precision note |
|---------|-----------|------------------|----------------|
| **UAV** | Unmanned Aerial Vehicle | Phương tiện bay không người lái / Drone | The *vehicle*. The system (vehicle + link + operator) is a **UAS** |
| **SUGV** | Sidewalk Unmanned Ground Vehicle | Robot giao hàng tự hành trên vỉa hè | Domain-specific usage. In defence robotics SUGV = *Small* Unmanned Ground Vehicle (the US Army programme built on iRobot's PackBot) — different machine, different risk profile |
| **TSP-D** | Traveling Salesperson Problem with Drone | Bài toán người giao hàng kết hợp drone | *One* truck + drone(s); the truck route and drone sorties are coupled |
| **VRP-D** | Vehicle Routing Problem with Drone | Bài toán định tuyến phương tiện kết hợp drone | Fleet of trucks + drones, capacities and time windows; TSP-D is the single-vehicle special case |
| **MILP** | Mixed-Integer Linear Programming | Quy hoạch tuyến tính nguyên hỗn hợp | Some variables integer (usually binary), the rest continuous; objective and constraints linear |
| **CA** | Continuous Approximation | Phương pháp tiếp cận xấp xỉ liên tục | Replace discrete stops with a spatial density; get closed-form cost estimates |
| **GA** | Genetic Algorithm | Giải thuật di truyền | Population-based metaheuristic inspired by selection/crossover/mutation |
| **GHG** | Greenhouse Gas | Khí thải nhà kính / CO₂ tương đương | Always specify the **functional unit** (per parcel, per tonne-km, per kg·km) |
| **C2 Link** | Command and Control Link | Kênh liên lạc chỉ huy và điều khiển | Availability, continuity, integrity, latency — see `05-uav-iot-stack-and-c2-links.md` |

#### 1.1 The actors and the decision timeline

```
Strategic (months)   fleet size, depot location, service radius, vehicle mix
        │              → continuous approximation, market analysis
Tactical (weeks)     base routes, drone-vs-truck assignment policy, shift design
        │              → MILP / large-neighbourhood search on forecast demand
Operational (minutes) which customer, which truck, which sortie, when to recharge
        │              → dynamic VRP, re-optimisation, dispatch heuristics
Real-time (seconds)  hold, reroute, abort, failover to C2 backup
                       → policy / rule engine / RL, bounded by safety case
```

The most common research mistake is to solve a *tactical* MILP and claim it is an *operational* policy. The most common industry mistake is to build a *real-time* heuristic with no *strategic* model behind it, so the fleet size is wrong before a single route is computed.

---

### 2. TSP-D: The Core Model

#### 2.1 Problem statement

Given a depot $0$, customers $C=\{1,\dots,n\}$, a truck (speed $v_t$) and one drone (speed $v_d$, endurance $E$), find a truck route and a set of drone sorties minimising completion time (or cost), such that:

1. every customer is served exactly once, by truck or by drone;
2. a drone sortie is a triple $(i,j,k)$: launched at $i$ (where the truck is), serves $j$, recovered at $k$ (where the truck will be);
3. the truck arrives at $k$ **no earlier than** the drone (the drone may hover/loiter, which costs energy);
4. drone flight time for the sortie respects endurance;
5. the drone is available for its next sortie (it is a single resource).

Assumption 3 is a modelling choice. The alternative — the truck *waits* at $k$ — is also common; the difference changes the objective value and which instances are feasible. Always state it.

#### 2.2 MILP formulation (arc-based, min completion time)

**Sets and parameters**

- $V = \{0\} \cup C$, depot $0$; arcs $(i,j)$, $i \ne j$
- $d_{ij}$ = travel distance; $v_t, v_d$ speeds; $E$ = drone endurance (time); $s$ = service/launch-recovery time
- $\mathcal{S} = \{(i,j,k) : i,j,k \in V \text{ distinct},\ d_{ij}+d_{jk} \le v_d E\}$ = feasible sorties

**Variables**

- $x_{ij} \in \{0,1\}$: truck traverses arc $(i,j)$
- $y_{ijk} \in \{0,1\}$: drone sortie launched at $i$, serves $j$, recovered at $k$
- $u_i \in \{0,1\}$: truck visits node $i$ (for customers), $u_0 = 1$
- $t_i \ge 0$: truck arrival time at $i$
- $T \ge 0$: makespan

**Objective**

$$
\min\ T
$$

**Constraints**

$$
\sum_{j \ne i} x_{ij} = u_i, \quad \sum_{j \ne i} x_{ji} = u_i \quad \forall i \in V
\tag{flow}
$$

$$
u_j + \sum_{(i,j,k) \in \mathcal{S}} y_{ijk} = 1 \quad \forall j \in C
\tag{coverage: truck XOR drone}
$$

$$
\sum_{j} \sum_{k} y_{ijk} \le u_i, \quad
\sum_{i} \sum_{j} y_{ijk} \le u_k \quad \forall i,k
\tag{sortie endpoints must be truck-visited}
$$

$$
t_j \ge t_i + \frac{d_{ij}}{v_t} - M(1 - x_{ij}) \quad \forall (i,j)
\tag{truck time propagation}
$$

$$
t_k \ge t_i + \frac{d_{ij}+d_{jk}}{v_d} + 2s - M(1 - y_{ijk}) \quad \forall (i,j,k) \in \mathcal{S}
\tag{drone arrives before truck (or hovers)}
$$

$$
T \ge t_i \quad \forall i \in V
\tag{makespan}
$$

$$
\text{subtour elimination on } x \quad \text{(MTZ or flow-based)}
$$

$$
\text{drone reuse: at most one active sortie per truck gap}
\tag{optional, single-drone}
$$

**Why this is hard.**

| Source of hardness | Explanation |
|--------------------|-------------|
| TSP inside | Finding the truck tour alone is NP-hard |
| Assignment | Each customer is truck-or-drone: $2^n$ combinations |
| Synchronisation | The coupling constraint links two vehicles' clocks |
| Big-$M$ weakness | Large $M$ produces a weak LP relaxation → huge branch-and-bound trees |
| Symmetry | Many equivalent truck tours/sortie labels |
| Endurance | Couples geometry (distance) with time and payload (energy) |

**Practical formulation advice.**

1. Use **arc-based + MTZ** for teaching, **flow-based or cutting-plane** formulations for solving.
2. Replace big-$M$ with the tightest valid bound (e.g. $M = T^{UB} - \underline{t}$) — this is often the single biggest speedup.
3. Add **valid inequalities**: every drone-served customer needs a truck-visited launch and recovery node; prune sorties with $\frac{d_{ij}+d_{jk}}{v_d} + 2s > E$.
4. Solve the **LP relaxation** first and look at the integrality gap — if it is 40%, your formulation needs work before you buy more CPU.
5. Use a MIP start from a GA/heuristic solution. Free upper bound, often 10–100× faster proof.

**Solver landscape (2026).**

| Solver | Licence | Notes |
|--------|---------|-------|
| Gurobi | Commercial | Fastest general MIP; free academic licence |
| CPLEX | Commercial | Strong, mature; free academic |
| HiGHS | Open source (MIT) | Now genuinely competitive for LP/MIP; good default for reproducibility |
| SCIP | Open source (academic licence) | Strong for constraint programming hybrids |
| CBC | Open source | Slow but ubiquitous; fine for small instances |
| OR-Tools CP-SAT | Open source | Excellent for routing/scheduling with integer structure; not an LP solver |

#### 2.3 VRP-D: what changes when the fleet grows

| Aspect | TSP-D | VRP-D |
|--------|-------|-------|
| Vehicles | 1 truck, 1..m drones | $K$ trucks, $m$ drones (often per truck) |
| Capacity | Usually ignored | Payload per drone, per truck |
| Time windows | Optional | Usually required (food, medicine) |
| Objective | Makespan / cost | Cost, tardiness, emissions |
| Depot structure | Single | Single or multi-depot; drone stations possible |
| New decisions | Sortie design | Truck-drone pairing, drone handover between trucks, recharging |
| Typical methods | DP, MILP, branch-and-price | MILP + ALNS, column generation, decomposition |

Known theoretical results worth remembering (Wang, Poikonen & Golden 2017): the **price of drone assistance can be unbounded** in worst case relative to a pure-truck solution when the drone must return to the same truck — the benefit is not automatic. This is the formal reason "add drones and save money" is an empirical claim, not a theorem.

---

### 3. Continuous Approximation (CA): Think in Densities, Not Points

#### 3.1 The core idea

Instead of enumerating $n$ customers, assume they are spread over an area $A$ with density $\delta = n/A$. Then expected route length has closed forms:

**Euclidean TSP** (Beardwood–Halton–Hammersley / Daganzo):

$$
L_{\text{TSP}} \approx \beta\sqrt{nA}, \qquad \beta \approx 0.712
$$

**Capacitated VRP** with $q$ stops per route and service radius $r$:

$$
L_{\text{VRP}} \approx \frac{2rn}{q} + k\sqrt{nA}
$$

The first term is the **radial** cost of going out and back; the second is the **local** tour cost. This single equation already explains a lot of logistics intuition:

- More stops per route ($q \uparrow$) → fewer radial trips → lower cost per stop.
- Larger area ($A \uparrow$) → local term grows as $\sqrt{A}$ (sub-linear — density helps).
- A drone with small $q$ and high speed changes the *coefficients*, not the structure.

#### 3.2 A UAV-specific CA sketch

For a truck launching drones along a corridor, a first-order model is:

$$
\text{Cost per parcel} \approx \underbrace{\frac{c_t}{\delta A}}_{\text{truck fixed}} + \underbrace{\frac{c_d}{\rho}}_{\text{drone ops}} + \underbrace{\frac{c_b \cdot e}{Q}}_{\text{energy per parcel}}
$$

where $\rho$ = drone sorties per truck stop, $e$ = energy per sortie, $Q$ = payload per sortie, $c_b$ = battery/energy cost. The point is not precision — it is **which terms dominate**, before you write a single constraint.

#### 3.3 When CA is valid — and when it lies

| Valid when | Lies when |
|------------|-----------|
| Many customers, dense, roughly uniform | Few customers or clustered demand |
| Strategic/tactical decisions | Turn-by-turn operational routing |
| Cost structure smooth | Hard time windows, strict precedence |
| Geometry roughly Euclidean | Real road network with barriers (rivers, ring roads) |
| You need an order-of-magnitude answer in minutes | You need a certificate of optimality |

Use CA as a **screening model**: it tells you whether drones can plausibly help on this geography, and roughly how many you need. Then spend the MILP budget only on promising configurations.

---

### 4. Genetic Algorithms (GA): When and How

#### 4.1 The canonical loop

```
P ← random_population(size)
repeat:
    evaluate fitness(P)                 # cost of decoded solution
    P' ← elitism(P, k)                  # keep best k unchanged
    while |P'| < size:
        p1, p2 ← tournament_select(P)
        c1, c2 ← crossover(p1, p2)      # e.g. order crossover (OX)
        mutate(c1); mutate(c2)          # swap / inversion / 2-opt move
        repair(c1); repair(c2)          # enforce endurance, capacity
        P' ← P' ∪ {c1, c2}
    P ← P'
until termination (generations, time, stagnation)
```

#### 4.2 Encoding matters more than the operators

For TSP-D/VRP-D, a workable chromosome is:

$$
\text{chromosome} = \big(\underbrace{\pi}_{\text{customer permutation}},\ \underbrace{\alpha}_{\text{mode: truck/drone}},\ \underbrace{\gamma}_{\text{sortie grouping}}\big)
$$

Decoding walks $\pi$, assigns truck or drone according to $\alpha$, and groups drone customers into sorties whose launch/recovery nodes are on the truck route. **The decoder is where the domain knowledge lives** — a bad decoder produces infeasible children and the GA spends its budget repairing instead of searching.

Standard operators:

| Operator | Purpose | Note |
|----------|---------|------|
| Order crossover (OX) | Preserves relative order in permutations | Good default for routing |
| PMX | Preserves absolute positions | Useful when position encodes assignment |
| Swap mutation | Local perturbation | Weak alone |
| Inversion (2-opt) | Removes crossings | Strong for Euclidean instances |
| Segment reversal + repair | Fixes endurance violations | Necessary in TSP-D |

#### 4.3 Parameter guidance (starting points, not laws)

| Parameter | Typical range | Effect |
|-----------|---------------|--------|
| Population | 50–200 | Larger = more diversity, slower per generation |
| Generations | 200–2,000 | Stop on stagnation, not on a fixed count |
| Crossover rate | 0.8–0.95 | Routing problems tolerate high rates |
| Mutation rate | 0.01–0.1 | Too high destroys structure, too low stagnates |
| Elitism | 1–5% | Guarantees monotone improvement |
| Selection | Tournament (size 3–5) | Better than roulette for cost scales |
| Local search | Optional memetic hybrid | Biggest single quality gain |

#### 4.4 Honest comparison of solution methods

| Method | Solution quality | Runtime | Optimality proof | Best for |
|--------|------------------|---------|------------------|----------|
| Exact MILP / branch-and-price | Optimal | Exponential in worst case | ✅ | $n \lesssim 50$–100 with good formulation |
| Dynamic programming (TSP-D) | Optimal | $O(n^2 2^n)$-ish | ✅ | Small $n$, benchmarking heuristics |
| Continuous approximation | Order of magnitude | Milliseconds | ❌ | Strategic sizing, intuition |
| GA | Good, variable | Minutes–hours | ❌ | Large $n$, non-linear, multi-objective |
| ALNS / LNS | Very good | Minutes | ❌ (bounds possible) | Large VRP-D; current industrial favourite |
| Column generation | Optimal/near | Hours | ✅ (with gap) | Fleet-size + routing jointly |
| RL policy | Fast inference | Training expensive | ❌ | Dynamic dispatch, repeated instances |

**Rule of thumb:** if $n \le 30$ and you can formulate it, solve it exactly and use the optimum as the benchmark. Only then does a heuristic's "3% gap" mean anything.

---

### 5. GHG Accounting: The Numbers and the Traps

#### 5.1 Functional units — the source of most disagreement

| Functional unit | Typical value from the literature | What it hides |
|-----------------|-----------------------------------|---------------|
| g CO₂e / vehicle-km | ≈ 1.9–79 | Payload utilisation |
| g CO₂e / parcel | ≈ 0.6–1,786 | Distance and consolidation |
| g CO₂e / kg·km | Proposed standardised unit | Still sensitive to boundary choices |

A review of the last-mile UAV literature reports that drones can cut emissions by up to ~71% vs diesel trucks and ~69% vs electric trucks, while truck-assisted UAV operations cut truck count by up to ~33% (≈28.6% lower emissions vs diesel). But the same review documents the variance drivers:

| Driver | Effect |
|--------|--------|
| Demand density −50% (e.g. 500 → 250 recipients/mile) | Reported as up to **−84%** GHG in one study — but the sign is model-dependent and should never be assumed |
| Grid carbon intensity 399 → 900 g CO₂e/kWh | Up to **+93%** |
| Strict vs lean flight regulation (buffers, altitude) | Up to **+400%** |
| Payload 2.25 kg vs 8 kg | Large increase per parcel |
| System boundary: operation only vs full LCA | Can flip the conclusion |

The boundary problem is stark: one study attributes **99.2%** of delivery emissions to production/disposal (all UAV components), while others report **87.5–97.5%** in the utilisation phase (with battery production at 2.5–12.5%). These are not contradictory measurements — they are different system boundaries. **State the boundary before the number.**

#### 5.2 The LCA skeleton

```
1. Goal & scope      functional unit, boundary, lifetime (truck 10 y, drone ~5 y)
2. Inventory         manufacturing, battery, electricity, maintenance, end-of-life
3. Impact            GWP-100 (kg CO2e), plus noise and local air quality
4. Interpretation    hotspots, sensitivity, scenario comparison
```

Comparison result from a parallel drone-truck study (PDSTSP, *Drones* 2025): ≈**20% lower carbon** than an electric-vehicle fleet and **20–30% lower cost** than a fuel-truck fleet, with drones contributing only ~2% of operational emissions — but manufacturing emissions relatively higher. Read that as: *drones are a small share of operational emissions and a non-trivial share of manufacturing emissions*, so the greener the grid, the more the manufacturing phase matters.

#### 5.3 What a defensible sustainability claim looks like

- [ ] Functional unit stated (per parcel **and** per kg·km)
- [ ] Boundary stated (tank-to-wheel vs well-to-wheel vs full LCA)
- [ ] Grid carbon intensity stated, with source and year
- [ ] Payload and load factor stated
- [ ] Demand density and service radius stated
- [ ] Battery replacement schedule stated
- [ ] Sensitivity analysis on the top three parameters
- [ ] Noise and non-GHG impacts at least acknowledged

---

### 6. Worked Example: Solving a 5-Customer TSP-D by Enumeration

The code below is deliberately brute force: for $n=5$ it is fast, and it makes the *structure* of the problem visible (truck route × sortie assignment). Use it to generate ground truth for testing a GA or a MILP.

```python
import itertools, math, random

def dist(a, b):
    return math.hypot(a[0] - b[0], a[1] - b[1])

def tspd(points, vt=10.0, vd=18.0, endurance=600.0, hover_margin=20.0):
    """Single truck + single drone. Assumptions:
       - one sortie per truck gap (single drone, no parallel sorties)
       - the drone must arrive at the recovery node no later than the truck
         (it may hover; hover_margin is charged against endurance)
       - Euclidean distances, no time windows, no capacity
    """
    n = len(points) - 1
    D = [[dist(points[i], points[j]) for j in range(n + 1)] for i in range(n + 1)]
    best = (float("inf"), None)
    customers = list(range(1, n + 1))

    for k in range(n + 1):
        for truck in itertools.permutations(customers, k):
            route = (0,) + truck + (0,)
            t = [0.0]                                   # truck arrival times
            for a, b in zip(route, route[1:]):
                t.append(t[-1] + D[a][b] / vt)

            remaining = [c for c in customers if c not in truck]
            gaps = list(range(len(route) - 1))
            if not remaining:
                if t[-1] < best[0]:
                    best = (t[-1], (route, []))
                continue
            if len(remaining) > len(gaps):
                continue                                # not enough gaps

            for assign in itertools.product(gaps, repeat=len(remaining)):
                if len(set(assign)) != len(assign):
                    continue                            # one sortie per gap
                ok, sorties, last_ret = True, [], 0.0
                for j, g in zip(remaining, assign):
                    a, b = route[g], route[g + 1]
                    td = (D[a][j] + D[j][b]) / vd
                    if td + hover_margin > endurance or td > t[g + 1] - t[g] + 1e-9:
                        ok = False
                        break
                    sorties.append((a, j, b, round(td, 1)))
                    last_ret = max(last_ret, t[g] + td)
                if not ok:
                    continue
                makespan = max(t[-1], last_ret)
                if makespan < best[0]:
                    best = (makespan, (route, sorties))
    return best

def tsp(points, vt=10.0):
    n = len(points) - 1
    best = (float("inf"), None)
    for perm in itertools.permutations(range(1, n + 1)):
        route = (0,) + perm + (0,)
        travel = sum(dist(points[a], points[b]) for a, b in zip(route, route[1:])) / vt
        if travel < best[0]:
            best = (travel, route)
    return best

if __name__ == "__main__":
    random.seed(7)
    pts = [(0.0, 0.0)] + [(random.uniform(-1500, 1500), random.uniform(-1500, 1500))
                          for _ in range(5)]
    t_tsp, t_tspd = tsp(pts), tspd(pts)
    print("points :", [(round(x), round(y)) for x, y in pts])
    print(f"TSP only: {t_tsp[0] / 60:6.2f} min  route={t_tsp[1]}")
    print(f"TSP-D   : {t_tspd[0] / 60:6.2f} min  route={t_tspd[1][0]}")
    print("sorties :", t_tspd[1][1])
    print(f"saving  : {100 * (1 - t_tspd[0] / t_tsp[0]):5.1f}%")
```

Actual output on this machine (seed 7, so your run should match):

```
points : [(0, 0), (-529, -1047), (453, -1283), (108, -403), (-1326, 22), (-1388, -199)]
TSP only:   8.56 min  route=(0, 3, 2, 1, 5, 4, 0)
TSP-D   :   8.15 min  route=(0, 4, 2, 0)
sorties : [(4, 1, 2, 130.2), (2, 3, 0, 75.7), (0, 5, 4, 90.6)]
saving  :   4.7%
```

**How to read this result honestly.**

- The saving is ~4.7%, not 50%. With one drone, a tight endurance budget, and a requirement that the drone never arrives after the truck, the drone can only shave the *last* mile of a short tour. This is the realistic regime.
- The optimal truck route visits only `{4, 2}` — three of five customers are served by drone. The model has effectively turned the truck into a **mobile launch platform**, which is exactly the TSP-D insight.
- Change one assumption and the number moves a lot: allow the truck to wait (instead of the drone hovering), increase `endurance`, or add a second drone, and re-run. That sensitivity *is* the research contribution.

---

### 7. Practical Application

#### 7.1 Which model should I use for which question?

| Question | Model | Why |
|----------|-------|-----|
| "Can drones help at all in this city?" | CA + simple cost model | Minutes, not weeks |
| "What is the optimal route for these 40 orders today?" | MILP (with warm start) or ALNS | Exactness vs scale |
| "How many drones do I need for 2030 demand?" | CA + queueing/utilisation model | Strategic, smooth |
| "Design a dispatch policy that reacts to cancellations" | Dynamic VRP + RL/heuristic | Re-optimisation speed matters |
| "Prove my heuristic is good" | Exact solution on small instances | Benchmark |
| "Is this service greener than vans?" | LCA with stated boundary | Avoids boundary games |
| "Can I actually fly this route?" | C2 feasibility overlay | Link corridors, no-fly zones, geofences |

#### 7.2 The C2-feasibility overlay (the bridge to note 05)

A route is only valid if the aircraft can be *commanded* along it. Formalise as a **feasible-arc subset** $A' \subseteq A$:

$$
A' = \{(i,j) : \text{link availability}(i,j) \ge \alpha,\ \text{latency}_{p95}(i,j) \le \lambda,\ (i,j) \notin \text{NoFly}\}
$$

Then solve the routing problem on $G' = (V, A')$. Two consequences:

1. Some customers become **drone-infeasible** and must be truck-served — which changes the economics, not just the route.
2. The optimal route may deliberately **detour** to stay inside a C2 corridor. A model without $A'$ will systematically overestimate savings.

This is the concrete place where the IoT note and the OR note meet: the link budget in `05` produces the numbers that populate $A'$.

#### 7.3 Economics sanity check (order of magnitude)

| Cost component | Aerial UAV | Sidewalk robot (company-reported) |
|----------------|------------|-----------------------------------|
| Energy per delivery | Small but non-trivial (payload × distance) | Very small |
| Hardware amortisation | High per unit, short life | High per unit, longer life |
| Operator/teleop ratio | Regulated; 1 operator : many aircraft | 1 remote operator : several robots |
| Insurance/regulatory | High | Medium |
| Reported all-in cost | Depends heavily on utilisation | `< $1` per delivery at scale vs `$8–10` human courier status quo (Serve Robotics, 2026; company figure, not audited) |

The lesson: **utilisation is the dominant term.** A drone that flies 6 deliveries a day cannot amortise its hardware and crew; the routing model's real job is to maximise sorties per hour subject to energy and C2 constraints.

---

### 8. Comparison Tables

#### 8.1 TSP-D vs VRP-D vs PDSTSP

| Feature | TSP-D | VRP-D | PDSTSP (parallel drone scheduling) |
|---------|-------|-------|-----------------------------------|
| Ground vehicles | 1 | Many | 1+ (truck route fixed/optimised separately) |
| Drone–truck coupling | Tight (launch/recover) | Tight | Loose (drones fly from depot in parallel) |
| Drone serves | 1 customer per sortie | 1+ (varies) | 1 per sortie, many sorties |
| Main difficulty | Synchronisation | Scale + synchronisation | Scheduling/sequencing |
| Typical method | DP, MILP | MILP, ALNS, column generation | Scheduling heuristics |

#### 8.2 When each sustainability claim is credible

| Claim | Credible if… | Suspicious if… |
|-------|--------------|----------------|
| "Drones cut emissions 70%" | Grid intensity, payload, distance, boundary all stated | Compares a full drone with a half-empty diesel van |
| "Drones are worse than vans" | Includes manufacturing + battery replacement | Ignores that vans also have manufacturing |
| "Our fleet is carbon neutral" | Offsets separated from reductions | Renewable certificates counted as zero without additionality |
| "Per-parcel emissions fell" | Load factor held constant | Achieved by dropping low-density customers |

---

## 🔗 Tech Cross-References (Section K)

- **C2 link and airspace** — the feasible-arc set $A'$ comes directly from link budgets, latency, and geofences → [`05-uav-iot-stack-and-c2-links.md`](05-uav-iot-stack-and-c2-links.md)
- **AI perception and data** — demand forecasting, travel-time prediction and obstacle maps are inputs to the routing model; the model is only as good as the data pipeline → [`06-uav-ai-perception-and-data-pipeline.md`](06-uav-ai-perception-and-data-pipeline.md)
- **DSA / complexity** — branch-and-bound, DP over subsets, and heap-based Dijkstra are the machinery under every solver claim → [`../dev_foundation/dsa/04-complexity-analysis-toolkit.md`](../dev_foundation/dsa/04-complexity-analysis-toolkit.md), [`../dev_foundation/dsa/05-data-structure-selection-guide.md`](../dev_foundation/dsa/05-data-structure-selection-guide.md)
- **Databases** — orders, time windows, and fleet state are a transactional schema problem; a routing query is a spatial join → [`../dev_foundation/databases/README.md`](../dev_foundation/databases/README.md)
- **Economics** — opportunity cost and network effects explain why fleet utilisation beats route optimality in the P&L → [`../economics_business/02-opportunity-cost.md`](../economics_business/02-opportunity-cost.md), [`../economics_business/04-network-effects.md`](../economics_business/04-network-effects.md)
- **Software architecture** — a dispatch system is an event-driven service with idempotent commands; re-optimisation is a background job → [`../software_architecture/01-event-driven-architecture-task-orchestration.md`](../software_architecture/01-event-driven-architecture-task-orchestration.md)

## 🔗 Related Topics

- [`05-uav-iot-stack-and-c2-links.md`](05-uav-iot-stack-and-c2-links.md) — the physical system that must actually fly the route
- [`06-uav-ai-perception-and-data-pipeline.md`](06-uav-ai-perception-and-data-pipeline.md) — data quality, georeferencing, and fleet learning
- [`../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md) — how a demand-forecasting model gets into the dispatch loop safely
- [`../dev_foundation/dsa/04-complexity-analysis-toolkit.md`](../dev_foundation/dsa/04-complexity-analysis-toolkit.md) — why "NP-hard" is a statement about worst case, not about your instance
- [SUGGESTED] **Column generation and branch-and-price for VRP-D** — `dev_foundation/dsa/` or a new `operations_research/` folder: the technique behind state-of-the-art fleet routing. Important because the MILP in §2.2 stops scaling around $n \approx 50$.
- [SUGGESTED] **Stochastic and dynamic VRP (demand uncertainty, cancellations, weather)** — the 2025 reviews identify this as the open frontier; a static model is a planning toy.
- [SUGGESTED] **Queueing theory for fleet sizing** — utilisation, waiting time, and the $M/M/c$ intuition behind "how many drones per operator". Important because fleet sizing, not routing, is where most of the money is.
- [SUGGESTED] **Noise externality and curfew regulation for urban drones** — `economics_business/` or `psychology_society/`: the constraint that decides night operations.

## 🤔 Open Questions

- [ ] In my target city, is the demand density high enough for CA to be a valid screening model, or is demand too clustered?
- [ ] What is the real price of C2 unavailability in the routing model — a hard arc deletion, or a probabilistic penalty?
- [ ] For $n = 100$, what is the actual integrality gap of the formulation in §2.2 with HiGHS, and does a warm start from a GA close it?
- [ ] Do published GHG savings survive when battery replacement and manufacturing are included with a realistic grid mix for Vietnam?
- [ ] Can a learned dispatch policy beat ALNS on re-optimisation latency without violating the safety case?

---

*Feynman check: "A delivery drone is a fast scooter that can only carry one parcel and must meet the van at a corner before its battery dies. The maths is: choose the van's loop and the corners where the drone takes off and lands, so the last parcel arrives soonest. If you pretend the meeting never has to be timed, the model is easy and wrong."*
