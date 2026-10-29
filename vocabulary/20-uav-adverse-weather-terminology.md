# UAV Adverse-Weather Terminology — Control, Routing, Perception and Onboard Decision-Making

**Created Date:** 2026-10-30
**Last Updated:** 2026-10-30
**Tags:** `#vocabulary`, `#technical-english`, `#uav`, `#adverse-weather`, `#flight-control`, `#robotics`, `#sensor-fusion`, `#edge-ai`
**Status:** Active
**Template:** Template V — Advanced Vocabulary & Collocation Note
**Source context:** seventeen terms requested as one Vietnamese–English keyword list for UAV operation in bad weather. The Vietnamese glosses in the tables are the user-supplied translations from the source list, kept verbatim for vocabulary tracking. English definitions and engineering precision notes are canonical here.
**Code repository:** The UAV code lives in the sibling project repositories; concept anchors are in [`iot_aiot/05`](../iot_aiot/05-uav-iot-stack-and-c2-links.md), [`06`](../iot_aiot/06-uav-ai-perception-and-data-pipeline.md) and [`07`](../iot_aiot/07-uav-logistics-terminology-and-optimization.md).

## 📌 Term List & Quick Reference

IPA uses American English forms; pronunciation varies by accent.

| # | Term / Phrase | IPA | Vietnamese gloss (source) | Core technical meaning | Layer |
|---|---|---|---|---|---|
| 1 | wind gust rejection / turbulence mitigation | /wɪnd ɡʌst rɪˈdʒekʃən/ · /ˈtɜːrbjələns ˌmɪtɪˈɡeɪʃən/ | Triệt tiêu gió giật / giảm thiểu rung lắc do nhiễu động khí quyển | Keep the vehicle on the commanded state when the air mass moves; *reject* the effect, not the wind | Control |
| 2 | disturbance observer-based control (DOBC) | /dɪˈstɜːrbəns əbˈzɜːrvər beɪst kənˈtroʊl/ | Điều khiển dựa trên bộ quan sát nhiễu | Estimate the lumped disturbance online, then cancel it in the control law | Control |
| 3 | model predictive control (MPC) / adaptive robust control | /ˈmɑːdəl prɪˈdɪktɪv kənˈtroʊl/ · /əˈdæptɪv roʊˈbʌst kənˈtroʊl/ | Điều khiển dự báo theo mô hình / điều khiển bền vững thích nghi | Optimise a finite-horizon plan under constraints / guarantee performance under bounded uncertainty while updating parameters | Control |
| 4 | deep reinforcement learning (DRL) for attitude control | /diːp ˌriːɪnˈfɔːrsmənt ˈlɜːrnɪŋ/ · /ˈætɪtuːd/ | Học tăng cường sâu cho kiểm soát góc nghiêng và tư thế bay | Learn a policy for roll–pitch–yaw tracking instead of hand-tuning a controller | Control |
| 5 | physics-informed neural network (PINN) for aerodynamics | /ˈfɪzɪks ɪnˈfɔːrmd ˈnʊrəl ˈnetwɜːrk/ · /ˌeroʊdaɪˈnæmɪks/ | Mạng neural kết hợp tri thức vật lý mô phỏng khí động học | A network trained with PDE residuals as a loss term, used as a physics-consistent surrogate | Control / modelling |
| 6 | weather-aware path planning / trajectory replanning | /ˈweðər əˈwer pæθ ˈplænɪŋ/ · /trəˈdʒektəri ˌriːˈplænɪŋ/ | Lập kế hoạch đường bay nhận thức thời tiết / tái lập quỹ đạo tức thời | Plan through a time-varying cost field; re-solve online when the field changes | Routing |
| 7 | autonomous weather adaptation / severe weather avoidance | /ɔːˈtɑːnəməs ˈweðər ˌædæpˈteɪʃən/ · /sɪˈvɪr ˈweðər əˈvɔɪdəns/ | Thích ứng thời tiết tự động / chủ động né tránh vùng thời tiết cực đoan | Change mission behaviour, or refuse the region, without a human in the loop | Routing / safety |
| 8 | energy-aware routing in adverse wind | /ˈenərdʒi əˈwer ˈruːtɪŋ/ · /ædˈvɜːrs wɪnd/ | Tối ưu hóa lộ trình tiết kiệm năng lượng pin khi gặp sức cản của gió | Optimise for joules, not metres, when ground speed depends on the wind field | Routing |
| 9 | dynamic geofencing and rerouting | /daɪˈnæmɪk ˈdʒiːoʊˌfensɪŋ/ · /ˌriːˈruːtɪŋ/ | Khoanh vùng địa lý ảo động theo thời gian thực | Airspace constraints with a validity interval; the route must be feasible at the time of flight | Routing / airspace |
| 10 | adverse-weather visual perception / degraded visual environments (DVE) | /ædˈvɜːrs ˈweðər ˈvɪʒuəl pərˈsepʃən/ · /dɪˈɡreɪdɪd/ | Nhận thức thị giác trong môi trường tầm nhìn suy giảm | Detect and localise when scattering, attenuation and low contrast destroy normal image statistics | Perception |
| 11 | sensor fusion (thermal infrared + radar / LiDAR + RGB) | /ˈsensər ˈfjuːʒən/ | Hợp nhất cảm biến (camera nhiệt hồng ngoại + radar/LiDAR + camera màu) | Combine modalities whose failure modes are different, not merely redundant | Perception |
| 12 | image dehazing / deraining / desnowing | /ˈɪmɪdʒ diːˈheɪzɪŋ/ · /diːˈreɪnɪŋ/ · /diːˈsnoʊɪŋ/ | Khử sương mù / loại bỏ hạt mưa / khử nhiễu tuyết | Restore an image degraded by atmospheric scattering or occluding particles | Perception |
| 13 | low-visibility SLAM | /loʊ ˌvɪzəˈbɪləti slæm/ | Định vị và lập bản đồ thời gian thực trong điều kiện thiếu sáng hoặc sương dày | Simultaneous localisation and mapping when the primary sensor is unreliable | Perception / state estimation |
| 14 | microclimate / localized weather forecasting | /ˈmaɪkroʊˌklaɪmət/ · /ˈloʊkəlaɪzd ˈweðər ˈfɔːrkæstɪŋ/ | Dự báo vi khí hậu siêu cục bộ ở độ cao thấp | Predict the wind, humidity and precipitation at the mission site, below NWP grid resolution | Environment model |
| 15 | onboard meteorological sensing | /ˌɑːnbɔːrd ˌmiːtiərəˈlɑːdʒɪkəl ˈsensɪŋ/ | Đo lường khí tượng trực tiếp trên drone | Measure pressure, airspeed, humidity and temperature on the vehicle itself | Sensing |
| 16 | edge AI for in-flight diagnostics (propeller icing detection) | /edʒ eɪˈaɪ/ · /ˌdaɪəɡˈnɑːstɪks/ · /ˈprɑːpələr ˈaɪsɪŋ/ | Trí tuệ nhân tạo biên xử lý chẩn đoán an toàn chuyến bay | Detect a developing fault onboard, within the control loop's time budget | Onboard compute |
| 17 | fail-safe autonomous landing / emergency return-to-home (RTH) | /ˈfeɪl seɪf ɔːˈtɑːnəməs ˈlændɪŋ/ · /rɪˈtɜːrn tə hoʊm/ | Hạ cánh khẩn cấp an toàn tự động / kích hoạt cơ chế quay về căn cứ | Bring the vehicle to a defined safe state when an operating limit is exceeded | Safety |

## 🧠 Detailed Vocabulary Analysis

### 1. Flight Control and Disturbance Rejection

#### 1.1 Wind gust rejection / turbulence mitigation

- **Definition & domain context.** A **gust** is a transient change in wind speed or direction; **turbulence** is a stochastic field of velocity fluctuations. **Rejection** means the controller attenuates the disturbance's effect on the output; **mitigation** means reducing the severity of the consequence. Neither word means the wind is removed.
- **Why it matters.** A multirotor has no wing to smooth the air; attitude disturbances propagate directly into position error, motor saturation and battery drain. The measurable claim is a bound: *position deviation under a gust of X m/s at frequency band Y*, not "flies well in wind".
- **Collocations.** *gust rejection bandwidth*, *turbulence mitigation strategy*, *reject a lateral gust*, *mitigate wind-induced oscillation*, *gust load alleviation* (fixed-wing).
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Triệt tiêu gió giật" does **not** become *eliminate the gust* — you cannot delete weather. Write *"the controller rejects the gust disturbance"* or *"attenuates the gust response"*. Likewise *mitigate* ≠ *minimize*: mitigate reduces harm, minimize reduces quantity.
- **Cross-domain bridge.** The same word appears in `dev_foundation/security/` as *mitigation* for risk, and in `economics_business/` as *risk mitigation*. Control theory and risk management both mean "reduce the impact of an event you cannot prevent" — a useful shared mental model.

#### 1.2 Disturbance observer-based control (DOBC)

- **Definition & domain context.** A **disturbance observer** estimates the *lumped* disturbance — wind, model error, unmodelled dynamics, payload change — from the difference between the measured output and the model prediction. The estimate $\hat{d}$ is then cancelled in the control law, typically $u = u_0 - \hat{d}/b$ for a input-affine model. DOBC is closely related to, but not identical with, **active disturbance rejection control (ADRC)**: ADRC uses an extended state observer plus a nonlinear feedback law; DOBC is the broader family.
- **Why it matters.** Wind is not measurable at the rotor disc in real time. DOBC turns an unmeasurable disturbance into an estimated signal, which is cheaper and faster than adding a wind sensor. The cost is a **bandwidth trade-off**: a fast observer rejects more disturbance but amplifies sensor noise and can destabilise the loop.
- **Collocations.** *disturbance observer*, *lumped disturbance*, *observer bandwidth*, *disturbance rejection performance*, *estimate and compensate*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Bộ quan sát nhiễu" is *disturbance observer*, not *noise observer* and not *interference observer*. In control English, *noise* is measurement noise; *disturbance* is the external input you want to reject.
- **Cross-domain bridge.** This is the control-theory twin of **anomaly detection** in ML: both estimate "what the model did not explain". In `ai_ml/08-fraud-detection-foundations.md` the residual is the fraud signal; in DOBC the residual is the wind.

#### 1.3 Model predictive control (MPC) / adaptive robust control

- **Definition & domain context.** **MPC** solves an optimisation over a finite horizon at every sampling instant, applies the first control move, and repeats — the "receding horizon" principle. It handles actuator limits, tilt limits and obstacle constraints *explicitly*. **Adaptive control** updates parameters online; **robust control** guarantees a performance bound for a *set* of possible plants (H∞, μ-synthesis, sliding mode); "adaptive robust" combines both.
- **Why it matters.** MPC is the natural language for constrained flight: "maximise progress subject to tilt ≤ 35°, thrust ≤ T_max, and stay inside the geofence". The price is computation and a model that must be good enough; nonlinear MPC at 50–100 Hz on embedded hardware is a real engineering constraint, not a footnote.
- **Collocations.** *receding-horizon control*, *constraint tightening*, *prediction horizon*, *control horizon*, *recursive feasibility*, *robust to bounded uncertainty*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Điều khiển dự báo" is *model predictive control* — not *forecast control*. *Forecast* belongs to weather; *predictive* belongs to control. "Bền vững" here is **robust**, not *sustainable*: this is the same false friend flagged in `vocabulary/18` (ESG) and it is worth memorising as a cross-domain trap.
- **Cross-domain bridge.** MPC is **optimisation under constraints**, the same object as the MILP in `iot_aiot/07`. The difference is time: MILP plans a route once, MPC re-solves every 20 ms.

#### 1.4 Deep reinforcement learning (DRL) for attitude control

- **Definition & domain context.** A neural policy $\pi_\theta(a \mid s)$ maps the state (attitude, rates, wind estimate) to motor commands, trained to maximise a reward (tracking error, smoothness, energy). Common algorithms are PPO and SAC; the hard part is not the algorithm but **reward design, safety shielding and the sim-to-real gap**.
- **Why it matters.** Classical PID/INDI controllers are strong and interpretable; DRL is justified when the plant is hard to model (damaged rotor, strong nonlinearity) or when many objectives must be traded. It is not justified by novelty. The engineering question is always: *what does the learned policy do that a tuned controller cannot, and how do I bound its failure?*
- **Collocations.** *policy gradient*, *reward shaping*, *sim-to-real transfer*, *domain randomisation*, *residual control*, *safety shield*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Tư thế bay" is **attitude**, not *posture*. In aeronautics *attitude* = orientation (roll, pitch, yaw); *posture* is for humans/animals. Note also that *attitude* has a second everyday sense ("thái độ") — a classic polysemy.
- **Cross-domain bridge.** DRL is the intersection of control and ML: the reward is the objective function, the policy is the controller, and the replay buffer is the dataset. The evaluation discipline from `books_summaries/07` (seeds, ablation, held-out conditions) applies exactly.

#### 1.5 Physics-informed neural network (PINN) for aerodynamics

- **Definition & domain context.** A PINN adds PDE residuals to the training loss:
  $$\mathcal{L} = \mathcal{L}_{\text{data}} + \lambda\,\big\|\mathcal{N}[u_\theta]\big\|^2$$
  where $\mathcal{N}$ is the governing operator (e.g., Navier–Stokes). The network is pushed to satisfy both observations and physics.
- **Why it matters.** A learned aerodynamic surrogate can be queried in microseconds, which matters for onboard MPC and for design sweeps. **Honest limitation:** PINNs are data-efficient and mesh-free, but they are not yet a replacement for RANS/LES at high Reynolds number; training can be unstable, and the physics residual is only as good as the assumed model. Use them as a **surrogate or hybrid component**, not as a universal CFD replacement.
- **Collocations.** *physics-informed loss*, *PDE residual*, *collocation points*, *surrogate model*, *hybrid physics–data model*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Mạng neural kết hợp tri thức vật lý" = *physics-informed neural network*. Do not translate as *physical neural network* (that means hardware) or *physics-aware network* (too vague to be a standard term).
- **Cross-domain bridge.** Same idea as **regularisation** in statistics: add prior knowledge to constrain a flexible model. The PDE is a very strong prior; that is why PINNs generalise outside the training distribution — and why a wrong PDE silently poisons the model.

### 2. Dynamic Routing and Weather Adaptation

#### 2.1 Weather-aware path planning / trajectory replanning

- **Definition & domain context.** **Path** is the geometric curve; **trajectory** adds a time parameterisation; **replanning** re-solves online when the environment or the vehicle state changes. A *weather-aware* planner optimises over a cost field that includes wind, precipitation, icing risk and visibility, not just distance.
- **Why it matters.** In wind, the shortest path is often not the cheapest: a headwind leg can cost more energy than a longer detour. Replanning frequency is a design choice tied to forecast update rate and onboard compute.
- **Collocations.** *cost map*, *time-varying cost field*, *online replanning*, *trajectory optimisation*, *feasible corridor*, *replanning trigger*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Quỹ đạo" is *trajectory*, not *orbit* (orbit is celestial/gravitational). "Tái lập quỹ đạo" is *replan the trajectory*, not *re-establish the orbit*.
- **Cross-domain bridge.** This is **dynamic vehicle routing** in operations research: the same re-optimisation logic as `iot_aiot/07`, with weather replacing demand as the stochastic input.

#### 2.2 Autonomous weather adaptation / severe weather avoidance

- **Definition & domain context.** **Adaptation** changes behaviour (altitude, speed, route); **avoidance** refuses the region. Both can be rule-based, model-based or learned. *Severe weather* is a defined threshold, not a feeling: convective cells, lightning, icing conditions, wind above the vehicle's operating limit.
- **Why it matters.** The safety case needs a **quantified operating envelope**: maximum sustained wind, maximum gust, precipitation rate, visibility minimum, temperature/icing range. Without those numbers, "weather adaptation" is a demo.
- **Collocations.** *operating envelope*, *weather threshold*, *abort criteria*, *contingency planning*, *no-go condition*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Chủ động né tránh" = *proactively avoid*; the standard technical noun is *avoidance*. Do not write *evasion* — in aviation that suggests hostile intent.
- **Cross-domain bridge.** Same structure as **risk management** in `economics_business/`: define exposure, set a limit, define the action at the limit. The control analogue is a **safety envelope** with a barrier function.

#### 2.3 Energy-aware routing in adverse wind

- **Definition & domain context.** Energy depends on airspeed and wind, not ground speed. The wind triangle gives
  $$\vec{V}_g = \vec{V}_a + \vec{V}_w,$$
  and aerodynamic power grows roughly with the cube of airspeed in forward flight, so energy per unit distance grows roughly with the square. A headwind forces a higher airspeed for the same ground speed, and a crosswind increases the power needed to hold track.
- **Why it matters.** Battery is the binding constraint in wind. The planner should minimise **joules**, subject to a reserve for go-around and landing. "Shortest route" is the wrong objective in bad weather.
- **Collocations.** *energy budget*, *specific energy consumption*, *headwind penalty*, *battery reserve*, *range reduction*, *optimal cruise speed*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Sức cản của gió" is *wind drag* / *wind resistance*; but the routing problem is about **headwind** and **crosswind**, not generic drag. Write *"headwind increases energy per kilometre"*, not *"wind makes the battery tired"*.
- **Cross-domain bridge.** This is the UAV version of **fuel-aware routing** in logistics; it is also an optimisation under uncertainty, because the wind forecast is not exact. The `iot_aiot/07` cost model is the right place to attach the weather term.

#### 2.4 Dynamic geofencing and rerouting

- **Definition & domain context.** A **geofence** is a virtual boundary. A **static** fence is fixed geometry; a **dynamic** fence has a validity interval and can be activated by weather, NOTAM, emergency or traffic. **Rerouting** computes a new feasible path around it.
- **Why it matters.** Airspace constraints are temporal: a storm cell closes a corridor for 40 minutes, then reopens. A planner that treats the fence as permanent either violates it or over-avoids.
- **Collocations.** *geofence breach*, *temporal validity*, *no-fly zone*, *corridor*, *hysteresis*, *reroute on breach*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Khoanh vùng địa lý ảo" = *virtual geographic boundary*; the standard term is **geofence**. Do not write *geo-fence* as two words in technical prose.
- **Cross-domain bridge.** In `dev_foundation/security/` a *policy boundary* is enforced by code; here it is enforced by geometry and time. Both are **access control**, one for data, one for airspace.

### 3. Adverse-Weather Vision and Sensing

#### 3.1 Adverse-weather visual perception / degraded visual environments (DVE)

- **Definition & domain context.** **DVE** is the aviation term for conditions where the outside scene is obscured — fog, rain, snow, dust, smoke, glare, whiteout. In computer vision, the mechanism is **scattering and attenuation**: contrast falls, colours shift, and high-frequency detail disappears.
- **Why it matters.** Detection models trained on clear images fail silently: confidence can stay high while localisation drifts. The correct evaluation is **detection performance under a measured degradation level**, not PSNR on a restored image.
- **Collocations.** *degraded visual environment*, *low-contrast scene*, *visibility degradation*, *adverse-weather benchmark*, *domain shift*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Nhận thức thị giác" = *visual perception*. Do not use *cognition* — that is the human interpretative process, not the sensor-to-detection pipeline.
- **Cross-domain bridge.** This is a **distribution shift** problem, the same one described in `iot_aiot/06`. Weather is a domain, and the test set must contain it.

#### 3.2 Sensor fusion (thermal infrared + radar / LiDAR + RGB)

- **Definition & domain context.** **Fusion** combines sensors at the data, feature or decision level. The point is **complementary failure modes**: radar (77 GHz) penetrates fog and rain better than LiDAR/RGB; thermal (LWIR) works in darkness and sees warm bodies, but rain and fog reduce thermal contrast; LiDAR (905/1550 nm) suffers backscatter from fog and rain.
- **Why it matters.** No single sensor is reliable in all weather. The engineering question is *which sensor fails when*, and whether the fusion logic can detect its own degradation.
- **Collocations.** *early / late / hybrid fusion*, *complementary modality*, *fusion architecture*, *modality dropout*, *sensor degradation*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Hợp nhất cảm biến" = *sensor fusion*. Do not write *sensor combination* or *sensor merge* — those are not standard terms.
- **Cross-domain bridge.** The same idea as **ensemble methods** in ML: combine models whose errors are uncorrelated. Fusion is an ensemble over physical sensors.

#### 3.3 Image dehazing / deraining / desnowing

- **Definition & domain context.** These are **image restoration** tasks. The classic haze model is
  $$I(x) = J(x)\,t(x) + A\,(1 - t(x)),$$
  where $J$ is the clean scene, $A$ the atmospheric light and $t$ the transmission. Deraining and desnowing remove occluding particles rather than a global scattering term.
- **Why it matters.** Restoration can **hallucinate**: a network trained to remove rain may also remove thin wires, cables or distant pedestrians. The metric that matters is downstream detection/avoidance, not visual pleasantness.
- **Collocations.** *image restoration*, *atmospheric scattering*, *transmission map*, *clean image*, *synthetic degradation*, *real-world degradation*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Khử sương mù" is *dehazing*; *defogging* also exists but is less standard in the vision literature. Do not write *de-fog* with a hyphen.
- **Cross-domain bridge.** Restoration is **inverse problems** — the same class as denoising in `ai_ml/`. Both need a forward model and a prior; both can overfit the synthetic degradation.

#### 3.4 Low-visibility SLAM

- **Definition & domain context.** SLAM estimates the vehicle's pose and builds a map simultaneously. Visual SLAM (ORB-SLAM3 is the reference) relies on feature matching and photometric consistency, both of which collapse in fog, rain, snow or darkness. Alternatives: thermal SLAM, radar SLAM, LiDAR SLAM, and multi-sensor fusion with **degeneracy detection**.
- **Why it matters.** Localisation failure is a safety event, not a UX bug. The system must know when it is lost — an **uncertainty/health signal** is part of the design.
- **Collocations.** *pose estimation*, *loop closure*, *degeneracy*, *relocalisation*, *sensor dropout*, *robust state estimation*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Định vị và lập bản đồ" = *localisation and mapping* (British spelling) / *localization* (American). Choose one convention per document and keep it; do not write *localisation and mapping* in one paragraph and *localization* in the next.
- **Cross-domain bridge.** Same problem as **dead reckoning** in `iot_aiot/05`: when the absolute reference disappears, error accumulates. The fix is always a second, independent source.

### 4. Microclimate and Onboard Computing

#### 4.1 Microclimate / localized weather forecasting

- **Definition & domain context.** Numerical weather prediction (NWP) operates on grids of roughly 1–10 km; the air 50 m above a street, a valley or a rooftop is **sub-grid**. **Microclimate** is the local modification of the atmosphere by terrain and buildings; **nowcasting** is the 0–6 hour forecast regime.
- **Why it matters.** A drone flies inside the boundary layer, where wind shear, thermals and urban canyon effects dominate. A forecast for the city is not a forecast for the corridor.
- **Collocations.** *boundary layer*, *urban canopy*, *downscaling*, *nowcasting*, *localised forecast*, *assimilation*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Vi khí hậu" = *microclimate* (one word). "Dự báo siêu cục bộ" = *hyperlocal forecasting* — a real term, but in research prose *localised* or *site-specific* is safer.
- **Cross-domain bridge.** This is **spatial data at multiple resolutions**, the same issue as map tiling and GSD in `iot_aiot/06`: the scale of the model must match the scale of the decision.

#### 4.2 Onboard meteorological sensing

- **Definition & domain context.** Sensors on the vehicle measure **static pressure**, **differential pressure** (airspeed via pitot), **temperature**, **humidity**, and sometimes **wind** via a multi-hole probe or an ultrasonic anemometer. Wind can also be *estimated* from the thrust/attitude model rather than measured directly.
- **Why it matters.** Onboard measurement is the only way to know the conditions *at the vehicle*, and it feeds both control (gust rejection) and safety (icing, envelope breach).
- **Collocations.** *air data*, *pitot-static system*, *air data boom*, *wind estimation*, *sensor calibration*, *measurement lag*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Đo lường khí tượng" = *meteorological measurement*; *meteorological* is the adjective, *meteorology* the field. Do not write *meteorological sensing* as *weather sensing* in a technical paper, though it is acceptable in speech.
- **Cross-domain bridge.** This is **instrumentation** in the experimental-design sense from `books_summaries/07`: a sensor with unknown calibration error is a threat to construct validity.

#### 4.3 Edge AI for in-flight diagnostics (propeller icing detection)

- **Definition & domain context.** **Edge AI** runs inference on the vehicle, within latency and power budgets. **Diagnostics** is the process of detecting and classifying a fault; **prognostics** predicts its evolution. **Propeller icing** occurs when supercooled liquid water freezes on the blade; the signature is thrust loss, vibration change and motor current anomaly.
- **Why it matters.** Icing changes aerodynamics and can cause asymmetric thrust and loss of control. A cloud round-trip is too slow and too unreliable; the detector must be onboard and must fail safe.
- **Collocations.** *fault detection and diagnosis (FDD)*, *anomaly detection*, *residual-based diagnosis*, *TinyML*, *latency budget*, *false-alarm rate*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Chẩn đoán" = *diagnosis* (the conclusion) or *diagnostics* (the process/system). "Chẩn đoán an toàn chuyến bay" is better rendered *in-flight safety diagnostics*, not *flight safety diagnosis*.
- **Cross-domain bridge.** Same structure as **fraud detection** in `ai_ml/08`: a rare positive class, asymmetric costs, a threshold that trades false alarms against missed events. Icing detection is an imbalanced classification problem with a physical prior.

#### 4.4 Fail-safe autonomous landing / emergency return-to-home (RTH)

- **Definition & domain context.** **Fail-safe** means the system transitions to a defined safe state on failure; **fail-operational** means it keeps performing the mission. **RTH** is a specific automation mode: climb to a safe altitude, fly back to the home point, descend and land. It depends on GNSS and a link; when either is lost, the safe state may be *land in place* or *descend under visual/inertial control*.
- **Why it matters.** The safety case is a **state machine with quantified triggers**: link loss timeout, battery reserve, geofence breach, wind above limit, icing detected. Each trigger needs a defined action and a hysteresis rule, or the vehicle will oscillate between modes.
- **Collocations.** *safe state*, *contingency landing*, *loss-of-link procedure*, *battery reserve*, *state machine*, *hysteresis*, *operator override*.
- **Viet-Glish pitfall vs. Band 7.0+ alternative.** "Hạ cánh khẩn cấp" = *emergency landing*; *crash landing* is uncontrolled and is **not** a synonym. "Quay về căn cứ" = *return-to-home (RTH)* as a mode, or *return to base (RTB)* in military/aviation usage.
- **Cross-domain bridge.** This is the same **fail-safe vs. fail-operational** distinction used in `software_architecture/` for distributed systems. The question "what is the safe state?" is identical whether the system is a drone or a database.

## 🔍 Cross-Cutting Distinctions Worth Memorising

| Pair | The distinction that matters |
|---|---|
| **gust vs. turbulence** | A gust is a discrete transient; turbulence is a continuous random field. Gust rejection is a transient response; turbulence mitigation is a spectral/statistical one |
| **rejection vs. mitigation vs. elimination** | Reject = attenuate effect on output; mitigate = reduce harm; eliminate = remove the cause (usually impossible for weather) |
| **disturbance vs. noise** | Disturbance is an external input (wind); noise is a measurement error (sensor). Different filters, different bandwidths |
| **predictive vs. forecast** | *Predictive control* predicts the plant; *weather forecast* predicts the atmosphere. Do not mix the collocations |
| **robust vs. sustainable** | *Robust control* = guaranteed under uncertainty; *sustainable* = ESG/environmental. A false friend from `vocabulary/18` |
| **adaptive vs. adaptable** | Adaptive control changes its own parameters; an adaptable system can be reconfigured. Not interchangeable |
| **attitude vs. posture vs. altitude** | Attitude = orientation; posture = body position; altitude = height. Three different words |
| **path vs. trajectory vs. route** | Path = geometry; trajectory = geometry + time; route = ordered waypoints/legs (network sense) |
| **fog vs. mist vs. haze vs. smog** | Fog: water droplets, visibility < 1 km; mist: 1–2 km; haze: dry aerosol, not necessarily humid; smog: smoke + fog |
| **icing vs. frost vs. condensation** | Icing = supercooled water freezing on a surface; frost = vapour depositing as ice; condensation = liquid water. Only icing has the aerodynamic severity |
| **fail-safe vs. fail-operational** | Fail-safe goes to a safe state; fail-operational continues the mission. Different certification claims |
| **fusion levels** | Data-level (raw), feature-level (embeddings), decision-level (votes). The level determines what can be recovered when one sensor degrades |

## 📐 Formulas You Should Be Able to State

**Wind triangle (ground speed from airspeed and wind):**
$$\vec{V}_g = \vec{V}_a + \vec{V}_w$$

**Aerodynamic power scaling (forward flight, idealised):** $P \propto V_a^3$, so energy per unit distance $E/d \propto V_a^2$. This is why a headwind is disproportionately expensive.

**Disturbance observer (input-affine plant $\dot{x} = f(x) + b u + d$):**
$$\hat{d} = \text{observer}(x, u), \qquad u = \frac{1}{b}\big(u_0 - \hat{d}\big)$$
The estimate is only as good as the observer bandwidth; too fast and sensor noise enters the loop.

**MPC (receding horizon):**
$$\min_{u_{0:N-1}} \sum_{k=0}^{N-1} \ell(x_k,u_k) + V_f(x_N) \quad \text{s.t. } x_{k+1}=f(x_k,u_k),\ x_k \in \mathcal{X},\ u_k \in \mathcal{U}$$

**Atmospheric scattering (haze):** $I = Jt + A(1-t)$.

**GSD (from `iot_aiot/06`):** $\text{GSD} = pH/f$ — the reason a person is only ~16 px tall at 100 m, and therefore why weather degradation is catastrophic for small-object detection.

## 📚 Resource & Literature Mapping

These are starting points, not a complete bibliography. Verify the edition before citing.

| Topic | Starting reference |
|---|---|
| DOBC overview | Chen, W.-H. et al., "Disturbance-Observer-Based Control and Related Methods—An Overview," *IEEE Trans. Ind. Electron.* 63(2), 2016 |
| MPC theory | Mayne, D. Q. et al., "Constrained model predictive control: Stability and optimality," *Automatica* 36(6), 2000 |
| DRL for aggressive flight | Kaufmann, E. et al., "Champion-level drone racing using deep reinforcement learning," *Nature* 620, 2023 |
| PINN | Raissi, M., Perdikaris, P., Karniadakis, G. E., "Physics-informed neural networks," *J. Comput. Phys.* 378, 2019 |
| Dehazing | He, K., Sun, J., Tang, X., "Single Image Haze Removal Using Dark Channel Prior," *IEEE TPAMI* 33(12), 2011 |
| Visual SLAM | Campos, C. et al., "ORB-SLAM3," *IEEE Trans. Robotics* 37(6), 2021 |
| Airframe icing | Bragg, M. B., Broeren, A. P., Blumenthal, L. A., "Iced-airfoil aerodynamics," *Progress in Aerospace Sciences* 41(5), 2005 |
| UAS airspace | EASA U-space regulation (EU) 2021/664; FAA Part 107 / Part 108 rulemaking |
| Reproducible evaluation | Demšar, J., "Statistical Comparisons of Classifiers over Multiple Data Sets," *JMLR* 7, 2006 |

Search terms for the gaps: *UAV adverse weather review*, *radar–camera fusion fog*, *thermal SLAM*, *nowcasting boundary layer*, *propeller icing detection*, *UTM dynamic geofence*.

## 📝 Research Journey

- **Why:** The keyword list arrived as preparation for UAV operation in wind, rain and low visibility — the weather layer that sits on top of the already-documented vehicle, perception and routing layers (`iot_aiot/05–07`).
- **Struggle:** The Vietnamese list compresses several distinct engineering objects into single glosses: "triệt tiêu" covers *reject/eliminate/cancel*, "bền vững" covers both *robust* and *sustainable*, and "tư thế" covers both *attitude* and *posture*. Each had to be split.
- **Aha moment:** The same three-layer structure from `iot_aiot/06` (fly / see / decide) organises the whole weather list: control is *fly*, perception is *see*, routing and microclimate are *decide*. The weather terms are not a new domain — they are a stress test applied to the existing one.
- **Career link:** An AI Engineer working on aerial or field robotics must be able to say *which* layer a weather claim belongs to, *what* the measurable bound is, and *what* the failure mode is. This vocabulary is the interface between the ML work and the safety case.

## 🔗 Related Topics

- [UAV IoT stack and C2 links](../iot_aiot/05-uav-iot-stack-and-c2-links.md) — link loss, failover state machine, safety case; the place where RTH and DAA live.
- [UAV AI perception and data pipeline](../iot_aiot/06-uav-ai-perception-and-data-pipeline.md) — GSD, domain shift, edge inference budgets, sensor-fusion framing.
- [UAV logistics terminology and optimisation](../iot_aiot/07-uav-logistics-terminology-and-optimization.md) — routing, MILP/VRP-D, energy and GHG models; weather-aware routing attaches here.
- [Sustainable fashion and the conscious consumer](18-sustainable-fashion-consumer-vocabulary.md) — *robust* vs. *sustainable*, the false friend that matters in control papers.
- [Fraud detection reading vocabulary](19-fraud-detection-reading-vocabulary.md) — the same imbalanced-classification structure as icing detection.
- [Scientific research methodology](../books_summaries/07-scientific-research-methodology.md) — validity, evaluation discipline and the difference between a demo and a claim.
- [Web security](../dev_foundation/security/README.md) — *mitigation* and *fail-safe* in a second domain, useful for the shared mental model.
- [SUGGESTED] `iot_aiot/08-uav-weather-operations.md` — a full Template A note on the operating envelope, weather thresholds and the safe-state state machine. *Why:* the vocabulary note defines terms; the safety case needs numbers, and there is no home for them yet.
- [SUGGESTED] `vocabulary/21-aviation-attitude-and-altitude-confusables.md` — *attitude/posture/altitude/heading/bearing* and the rest of the aviation polysemy cluster. *Why:* note 20 flags the trap but does not own the full family.
- [SUGGESTED] `books_summaries/08-statistical-methods-for-machine-learning.md` — already suggested in note 07; the seeds/CI discipline is exactly what weather-robustness claims need.

## 🤔 Open Questions

- [ ] What is the minimum weather-sensor suite that makes a DOBC claim testable outdoors, rather than only in simulation?
- [ ] Can a learned weather-aware planner be certified, or does the safety case force a rule-based envelope around it?
- [ ] How should "weather robustness" be reported so that it is falsifiable — a bound, a distribution, or a table of failure cases?

---

*"When the wind is the disturbance, the controller's job is not to stop the wind. It is to make the wind irrelevant."*
