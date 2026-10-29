# IoT / AIoT & UAV Systems — Notes

**Purpose:** How a device that talks to the physical world is actually built — sensor → firmware →
protocol → vehicle → data → operations. The folder started with embedded/web glue for a
telemetry product and now also holds the **UAV track**, where the same IoT questions become
safety-critical: a C2 link that drops, a model that must run at 10 W, a route that must be
feasible in four dimensions.

**Scope boundary:** this folder holds *concepts, architecture and engineering judgement*. Code
lives in the sibling repositories. No note here duplicates a code walkthrough or a build log.

**Sibling code:** [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) ·
`git@github.com:NguyenPhan161206/UMT-Truck-Blind-Sight.git` — the ESP32/CAN work behind the
telemetry notes. Notes **05–07** are concept anchors: the UAV XML/MAVLink/ROS 2 code lives in the
sibling UAV project repositories, not here (AGENTS.md Section L).

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | [Rate Limit & Server-Sent Events](01-rate-limit-and-sse.md) | A (Tech) | How do you stop one client from starving a streaming endpoint — and why is SSE enough when you only need server → client? |
| 02 | [MQTT Simulation & Testing System](02-mqtt-simulation-testing-system.md) | A (Tech) | How do you test a telemetry pipeline before the hardware exists — and what does MQTT QoS actually promise? |
| 03 | [Hardware Abstraction Layer (HAL)](03-hardware-abstraction-layer-hal.md) | A (Tech) | How do you keep application logic unchanged when the MCU changes, and unit-test firmware without a board? |
| 04 | [PSRAM vs Internal SRAM on ESP32](04-esp32-psram-vs-internal-sram.md) | A (Tech) | When does an ESP32 need external PSRAM, and why can that memory hold data but not run code? |
| 05 | [UAV as an IoT System: Stack, C2 Links, Airspace](05-uav-iot-stack-and-c2-links.md) | A (Tech) | What does a UAV share with any IoT device — and where does the analogy break (hard real-time, C2 link, airspace)? |
| 06 | [UAV AI & Data Pipeline](06-uav-ai-perception-and-data-pipeline.md) | A (Tech) | A person is ~16 px at 100 m — what does that one number force you to change in the whole pipeline? |
| 07 | [UAV Logistics: Terminology & Optimisation](07-uav-logistics-terminology-and-optimization.md) | A (Tech) | Why is TSP-D not "TSP plus a drone" — and which layer of the stack does "GHG per parcel" belong to? |

Status of 05–07 is **Review**, not Published: the concepts are verified (05's link budget and 07's
TSP-D solver were executed, not estimated), but the regulatory and literature sections track a
moving target and should be re-read before being cited.

---

## 🗺️ Reading order

Numbering is chronological; the *conceptual* order is two separate arcs that meet at note 05.

**Arc A — embedded & telemetry (02 → 03 → 04).**
1. **03 (HAL)** first: it is the idea that makes the rest testable. Without it, 02 and 04 are
   hardware trivia.
2. **04 (PSRAM/SRAM)** next: a capability budget. It answers "what can this chip actually hold",
   which decides whether on-device buffering or a store-and-forward design is even possible.
3. **02 (MQTT simulation)** last in this arc: with a mockable HAL (03) and a known memory budget
   (04), the emulator and Record & Replay system become a design rather than a wish.

**Arc B — UAV (05 → 06 → 07).**
1. **05** defines the vehicle, the five planes, the link and the airspace. Every later note borrows
   its vocabulary (`offboard`, `BVLOS`, `C2`, `failsafe`).
2. **06** assumes 05: perception only matters because there is a companion computer allowed to die,
   and GSD only matters because the aircraft flies at a regulated altitude.
3. **07** assumes both: the routing model's drone range parameter comes from 05's link budget, and
   its "feasible arc" set is a C2 constraint, not a distance constraint.

**Note 01 is the outlier.** Rate limiting and SSE are web-server concerns that happen to have been
filed here early on. It is a correct note in the wrong folder; see the backlog.

**If you only read three:** 03, 05, 06 — abstraction that makes firmware testable, the system frame,
and the number (GSD) that decides whether the AI is possible at all.

---

## 🧩 How the notes fit together

```
Physical world
   │
   ├─ 04 memory budget ──► 03 abstraction ──► 02 protocol + test rig
   │                                              │
   │                                        (telemetry product)
   │
   └─ 05 vehicle + link + airspace
            │
            ├─ 06 perception & data pipeline  (see)
            └─ 07 routing & sustainability    (decide)

01 (rate limit / SSE) is the control plane of the *server* side of either branch.
```

The two arcs share one question: **where does the computation live, and what happens when that
place fails?** In arc A the answer is a HAL and a simulator; in arc B it is a flight controller
that keeps flying when the companion computer reboots.

---

## 🔗 Tech Cross-References (Section K)

- **Web server (SSR)** — note 01's rate limit and SSE are the same primitives an Express app uses
  for streaming and abuse control; the security half (429, connection exhaustion) is a DoS topic
  → [`../dev_foundation/web_server/README.md`](../dev_foundation/web_server/README.md),
  [`../dev_foundation/security/README.md`](../dev_foundation/security/README.md)
- **Linux & Docker** — the broker, the telemetry lake and the SITL stack all run as containers on a
  Linux host; a Mosquitto ACL mistake is a deployment problem before it is an IoT problem
  → [`../dev_foundation/linux/README.md`](../dev_foundation/linux/README.md),
  [`../dev_foundation/docker/README.md`](../dev_foundation/docker/README.md)
- **Databases** — fleet telemetry is a time-series schema question: write amplification, retention,
  and whether the hot path is a table or a stream
  → [`../dev_foundation/databases/README.md`](../dev_foundation/databases/README.md)
- **AI/ML & MLOps** — note 06's fleet learning loop is note `ai_ml/05` applied to a vehicle: model
  registry, drift, and the cost of retraining from the edge
  → [`../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md)
- **Software architecture** — the cloud plane (ingest → stream → lake → API) is an event-driven
  system with at-least-once semantics; idempotency is what makes retries safe
  → [`../software_architecture/01-event-driven-architecture-task-orchestration.md`](../software_architecture/01-event-driven-architecture-task-orchestration.md)

---

## 🔗 Related Topics

- [`../dev_foundation/README.md`](../dev_foundation/README.md) — the engineering foundation these
  notes sit on (Linux, Docker, databases, DSA)
- [`../ai_ml/README.md`](../ai_ml/README.md) — the AI track; note 06 is where it meets hardware
- [`../vocabulary/20-uav-adverse-weather-terminology.md`](../vocabulary/20-uav-adverse-weather-terminology.md)
  — the seventeen-term vocabulary note, sorted onto note 06's fly/see/decide map
- [`../RESEARCH_AREAS.md`](../RESEARCH_AREAS.md) — folder registry and note history
- 🔗 [UMT-Truck-Blind-Sight](https://github.com/NguyenPhan161206/UMT-Truck-Blind-Sight) — device code
  & explanations live there; this diary holds concept anchors and lessons learned

---

## 📐 Template

All notes use **Template A: Tech Note** and must pass the Section H depth checklist: mechanism before
command, an executed example (not a plausible one), a "why it matters" angle, and the Research
Journey. Notes 05–07 additionally carry the Section K cross-technology block.

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | **UAV energy & endurance model** (hover power, payload trade-off, battery C-rate, reserve) | Referenced by 05 (range), 06 (edge power budget) and 07 (cost per parcel) but owned by none — currently scattered across three notes |
| High | **SITL lab** (PX4 SITL + MAVSDK, offboard at 2 Hz, watchdog and failsafe drills) | 05 explains the rule in prose; nobody has run it. A Template G note would make the failure visible |
| Medium | **ROS 2 + uXRCE-DDS in practice** (QoS mismatch, frame conventions, timestamping) | 06 lists the failure modes; a hands-on note would turn the list into reflexes |
| Medium | **Remote ID & UTM integration** (broadcast vs network RID, what an operator must expose) | 05 covers the regulation; the implementation surface is missing |
| Medium | **MQTT 5 + TLS/ACL for a fleet** (auth, topic design, backpressure) | 02 covers the protocol and the simulator; the production hardening is absent |
| Low | **Refile note 01** into `dev_foundation/web_server/` (or add a bridge note) | Rate limiting/SSE is a server-side concern; leaving it here makes the folder index misleading |
| Low | **CAN bus & vehicle telemetry** (frames, arbitration, error states) | The UMT project's other half; currently only mentioned, never explained |

---

*Last updated: 2026-10-30*
