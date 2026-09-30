# 04. Solar Harvesting & REM Consolidation Cycle

## 1. Ecological Energetics: Beyond Continuous Operation

Animal biology does not expend physical locomotion continuously; organisms oscillate between **foraging/work**, **rest/sunning (ectothermic regulation)**, and **REM sleep (memory consolidation)**.

Conventional robots attempt to maintain constant compute and actuation until the battery is critically drained, forcing a frantic retreat to a wall charger. SOMA-Robot adopts an **ecological cycle governed by solar rhythms**:

```
                       ┌────────────────────────────────┐
                       │   DIURNAL ACTIVE WORK PHASE    │
                       │ SoC: 40% - 90% | Motors: Active│
                       │ Executive: Event-Driven Sleep  │
                       └───────────────┬────────────────┘
                                       │
                      Battery drops    │ Solar Irradiance High &
                      below 35%        │ Battery reaches >80%
                                       ▼
┌───────────────────────────────┐              ┌───────────────────────────────┐
│     PASTURE MODE ("PASTAR")   │              │     SOLAR REM SLEEP ("SOÑAR") │
│ Find Sun | Park | Motors OFF  │              │ Motors OFF | Compute: 100%    │
│ Sub-5W Autonomic Core only    │              │ Offline Memory Distillation   │
└───────────────────────────────┘              └───────────────────────────────┘
```

---

## 2. The Operational Modes

### 2.1 Active Transit & Survey Mode
* **Status:** Motors active, Navigation controller running at 50 Hz.
* **Compute Profile:**
  * Autonomic Core (Domain A): 3.5W
  * Thalamus Vision Gating: 4W
  * Neocortex SLM: Suspended in RAM (0.5W sleep mode)
  * Actuation: 30W – 80W (terrain-dependent)
* **Goal:** Progress toward human-assigned mission coordinates while monitoring solar flux gradients.

### 2.2 Pasture Mode ("Pastar")
When the Energy Drive flags moderate depletion ($D_{\text{energy}} > 0.65$, corresponding to $\text{SoC} < 35\%$), the robot transitions into **Pasturing**:
1. **Solar Gradient Tracking:** The robot queries the Hippocampal resource map for known high-irradiance clearings or tracks ambient light sensors to locate the nearest sunny patch.
2. **Optimal Solar Parking:** Once in the sunlight, the robot aligns its horizontal dorsal array directly perpendicular to the sun's azimuth/elevation (using chassis rotation or articulated solar panel tilt).
3. **Deep System Power-Down:**
   * Motor inverters: Disconnected via hardware contactors.
   * Executive NPU/GPU: Powered down into ultra-low-power standby.
   * Total system draw: Drops to **$< 2.5\text{W}$**.
4. **Energy Accumulation:** With solar input at $100\text{–}140\text{W}$ and system consumption at $2.5\text{W}$, almost 98% of harvested photons flow directly into replenishing the $\text{LiFePO}_4$ cells.

---

## 3. The Solar REM Sleep Cycle ("Soñar")

The most biologically inspired feature of SOMA-Robot is the **Solar REM Phase**. 

In mammals, REM sleep is metabolically expensive: the brain consumes high glucose to replay, prune, and consolidate synaptic connections. In SOMA-Robot, running a local 8B LLM, fine-tuning neural navigation policies, or pruning dense SLAM graphs consumes $20\text{–}30\text{W}$ of power.

**The Economic Opportunity:** When the battery reaches $>80\%$ SoC and the robot is stationary under clear midday sunlight, the solar panels generate an **unusable surplus** of power (the battery cannot absorb charge faster without overheating). SOMA-Robot utilizes this exact surplus to execute cognitive sleep consolidation:

```
Solar Generation:   +135 W  ═══════════════════════════════════════════════════════
Battery Trickle:     -25 W  ──┐
Chassis Motors:        0 W    │── Surplus Solar Energy (~100W) powers 
Cognitive Core:      -25 W  ──┘   full NPU/GPU memory consolidation for FREE
```

### 3.1 What Happens During the "Dream Phase"?

#### A. Episodic Memory Distillation (L1/L2 $\to$ L4)
Throughout the morning, hundreds of routine sensory snapshots were logged. During REM sleep:
* The SLM inspects the raw event stream.
* Discards uninformative sensory redundancies (e.g., 40 minutes of driving over identical gravel).
* Extracts causal heuristics: *"On slope 14B, moisture caused 22% track slip; avoid this path if humidity $>85\%$"*.
* Writes compact semantic summaries to the permanent L4 archive.

#### B. Topological Map Graph Compression
* Merges redundant geometric nodes in the SLAM graph.
* Updates edge transition costs with empirical energy expenditures measured during the morning trek.

#### C. Policy & Value Fine-Tuning
* Replays critical failure incidents (e.g., near-tip events or sudden obstacle encounters).
* Adjusts Cerebellar MPC parameters or small RL policy weights using synthetic counterfactual perturbations.

#### D. Subsequent Diurnal Planning
* Synthesizes weather forecasts, ephemeris calculations (sun elevation for the afternoon), and remaining mission goals to compute an energy-optimal route for the second half of the day.

---

## 4. State Transition Logic

```text
Transition Rules:
IF (SoC < 0.25) AND (P_solar > 40W):
    ACTION -> TRANSITION_TO_PASTURE
    REASON -> "Survival drive overrides mission; seeking immediate charge."

IF (SoC > 0.80) AND (P_solar > 80W) AND (Chassis == STATIONARY):
    ACTION -> TRANSITION_TO_REM_SLEEP
    REASON -> "Solar energy surplus detected; initiating cognitive memory consolidation."

IF (REM_Sleep_Complete == TRUE) AND (Mission_Queue_NotEmpty == TRUE):
    ACTION -> WAKE_AND_RESUME_MISSION
    REASON -> "Cognitive consolidation finished; resuming operational survey."
```
