# 01. System Architecture Blueprint

## 1. Executive Summary

**SOMA-Robot** is an embodied autonomous platform engineered for **indefinite outdoor operational persistence** across unstructured terrains (precision agriculture, long-range environmental surveying, perimeter patrol, and planetary exploration analogues). 

The platform departs from modern robotics dogma—which attempts to brute-force continuous, end-to-end multimodal perception on power-hungry accelerators—by implementing an **asymmetric, biomimetic cognitive architecture**. Low-level autonomic survival loops operate continuously at micro-watt levels, while executive reasoning (driven by an on-device Small Language Model / SLM) is invoked on-demand in an event-driven loop.

---

## 2. Physical & Energy Platform (The Body)

### 2.1 Mechanical Morphology
* **Locomotion:** 4-wheel independent skid-steer or dual rubber continuous tracks with rocker-bogie passive suspension. Bipedal and quad-legged designs are explicitly rejected due to baseline energy consumption when static: a wheeled/tracked rover expends **zero mechanical energy** while stationary under sunlight.
* **Payload Bay:** Sealed IP67 central avionics hull with passive conductive heat-pipe dissipation coupled to the aluminum chassis base.
* **Manipulation (Optional Subsystem):** 2-DOF or 3-DOF folding under-chassis robotic arm with magnetic or mechanical tool exchange, designed to fold completely into an aerodynamic/dust-protected recess during high-speed transit or pasturing.

### 2.2 Power Harvesting & Interoceptive Storage
* **Solar Array:** High-efficiency monocrystalline PV cells laminated directly onto the top horizontal dorsal surface ($\approx 0.8\text{ m}^2$, generating $120\text{–}160\text{W}$ peak at 1000 $\text{W/m}^2$ standard solar irradiance).
* **Charge Topology:** Dual Maximum Power Point Tracking (MPPT) regulators feeding into a central Power Distribution Board (PDB).
* **Battery Chemistry:** Lithium Iron Phosphate ($\text{LiFePO}_4$, 24V nominal, $20\text{–}40\text{ Ah}$). Selected for deep thermal stability ($-20^\circ\text{C}$ to $+65^\circ\text{C}$), high cycle life ($>3000$ cycles at 80% DoD), and resistance to thermal runaway.
* **Interoceptive Sensor Array:**
  * High-precision shunt current monitors measuring solar input current and instantaneous system draw.
  * Real-time State of Charge (SoC), State of Health (SoH), and individual cell differential voltage monitoring via an isolated I2C/CAN Battery Management System (BMS).
  * Multi-point thermal thermistors on battery cells, motor drive MOSFETs, and main compute SoCs.
  * 3-axis high-g shock/vibration accelerometer mounted directly on the structural bulkhead to detect physical hull impacts or tip-overs.

---

## 3. Heterogeneous Dual-Compute Architecture

To prevent energy depletion during idle or monitoring phases, computation is strictly divided across two hardware domains:

```
┌────────────────────────────────────────────────────────────────────────┐
│ DOMAIN A: AUTONOMIC SURVIVAL CORE (Sub-5W, Always-On)                  │
│ Hardware: Dual-Core ARM Cortex-M7/M4 MCU (e.g., STM32H7) or RP2350     │
│ OS: Real-Time OS (FreeRTOS / Zephyr)                                   │
│ Responsibilities:                                                      │
│  - 1 kHz motor PID/odometry loop & wheel encoder integration           │
│  - Interoception sampling (voltage, temperature, current, tilt)        │
│  - Hardwired spinal collision/cliff reflexes                           │
│  - Hardware Watchdog & Power gating relay for Domain B                 │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ SPI / High-Speed UART / CAN Bus
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ DOMAIN B: EXECUTIVE COGNITIVE CORTEX (15–30W, Event-Driven / Sleep)   │
│ Hardware: Ultra-low-power Edge NPU/GPU (NVIDIA Jetson Orin Nano,       │
│           Hailo-8 accelerator, or AMD Embedded Ryzen / NPU)            │
│ OS: Minimal Linux with real-time kernel patches (PREEMPT_RT)           │
│ Responsibilities:                                                      │
│  - Thalamic sensory compression (YOLO-Nano, depth filters)             │
│  - Hippocampal Topological SLAM & Vector Memory                        │
│  - SOMA Executive SLM (3B to 8B 4-bit quantized local model)           │
│  - High-level mission planning and anomaly resolution                  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. The 5 Biomimetic Cognitive Layers

```
                                 [ HUMAN MISSION ]
                                         │
                                         ▼
  LEVEL 4: NEOCORTEX ──────────► [ SOMA Sovereign SLM ]
  (Executive Reasoning)                 │
                                        │ High-level tool calls (JSON)
                                        ▼
  LEVEL 3: CEREBELLUM ─────────► [ Motion Planner (MPC/PID) ]
  (Motor Execution)                     │
                                        │ Motor Commands (PWM/Velocities)
                                        ▼
  LEVEL 0: BRAINSTEM ──────────► [ SAFETY VETO GATE ] ──► [ MOTOR INVERTERS ]
  (Survival Reflexes)                   ▲
                                        │ Interoceptive Override (Battery/Tilt/Heat)
                                 [ BMS & SENSORS ]
```

### Layer 0: Brainstem & Hypothalamus (Survival & Homeostasis)
* **Frequency:** 100 Hz to 1,000 Hz.
* **Substrate:** Domain A (Cortex-M7 MCU).
* **Role:** Evaluates internal bodily state against physiological boundaries. Calculates the **Homeostatic Drive Vector**:
  $$\mathbf{D} = \langle D_{\text{energy}}, D_{\text{thermal}}, D_{\text{mechanical}}, D_{\text{tilt}} \rangle$$
* **Veto Mechanism:** Hardwired circuit and firmware gate. If $D_{\text{mechanical}} = 1$ (impact detected) or $D_{\text{energy}} > 0.95$ (critical battery depletion), the Brainstem hardware relay directly severs actuator drive signals or overrides the motor controller, forcing the robot into an emergency braked or charging state. **The Neocortex cannot prevent or cancel this veto.**

### Layer 1: Thalamus (Sensory Filtering & Attentional Gating)
* **Frequency:** 10 Hz to 30 Hz (camera/LiDAR rate).
* **Substrate:** Edge NPU or low-power DSP.
* **Role:** High-bandwidth exteroception (megabytes of point clouds and camera frames) is compressed into compact semantic tokens.
* **Attentional Gate:** If the environment is static or matches the current expected trajectory, the Thalamus suppresses updates. If an anomaly occurs (unmapped boulder, animal crossing, loss of traction), the Thalamus emits a hardware interrupt waking the Neocortex.

### Layer 2: Hippocampus (Spatial Mapping & Episodic Recall)
* **Frequency:** Asynchronous event updates (0.1 Hz – 5 Hz).
* **Substrate:** Fast on-device vector DB (e.g., Qdrant / SQLite-VSS) + Topological Graph SLAM.
* **Role:**
  * **Topological Resource Map & Tour Optimization:** Graphs representing waypoints annotated with physical properties (terrain cost, solar exposure history, obstacle density).
    * **Multi-Waypoint Survey Planning (`k-Alternatives`):** Sequences scientific sampling waypoints and solar pasturing tours via bounded $k$-deviations over greedy distance/irradiance metrics, utilizing learned heuristic list ordering across diurnal cycles.
    * **Dynamic Waypoint Insertion (`Ripple Insertion`):** Handles real-time adjustments (e.g., sudden discovery of unmapped solar patches or terrain blockages) by dynamically inserting/removing waypoints in $O(N \log N)$ ($<0.1\text{ ms}$), relaxing trajectory tension locally without recalculating the global route from scratch.
  * **Episodic RAG (SOMA L3/L4 Memory):** Stores key operational incidents formatted as semantic episodes: `Episode(ID, Context, Action, Outcome, EnergyDelta)`.

### Layer 3: Neocortex (Executive Planning & SOMA Sovereign Agent)
* **Frequency:** Event-driven (invoked only upon task changes, waypoint arrivals, or unexpected alerts).
* **Substrate:** Quantized Local Language Model (e.g., Llama-3-3B-Instruct, Phi-3.5-mini, or Mistral-7B at 4-bit quantization).
* **Role:** High-level problem solving, goal decomposition, and causal hypothesis generation. Communicates solely via strictly typed JSON tool calls.

### Layer 4: Cerebellum & Motor Cortex (Kinematic Coordination)
* **Frequency:** 50 Hz – 100 Hz.
* **Substrate:** Domain A (or auxiliary motor coprocessor).
* **Role:** Receives symbolic waypoints from the Neocortex (`NAVIGATE_TO(x, y, max_velocity, clearance)`) and executes smooth, dynamically feasible trajectories using Model Predictive Control (MPC). Handles local reactive micro-steering around small stones and bumps without burdening the Neocortex.

---

## 5. Asynchronous Subsumption Execution Loop

```
                        ┌───────────────────────┐
                        │      SENSE (1kHz)     │
                        │ Interoception/Extero  │
                        └───────────┬───────────┘
                                    │
                                    ▼
                        ┌───────────────────────┐
                        │  SURVIVAL CHECK (L0)  │
                        │  Is Drive Critical?   │
                        └─────┬───────────┬─────┘
                     YES      │           │ NO
       ┌──────────────────────┘           └──────────────────────┐
       ▼                                                         ▼
┌───────────────┐                                         ┌───────────────┐
│ EXECUTE VETO  │                                         │ THALAMUS (L1) │
│ Park & Charge │                                         │ Filter & Gate │
│ Cut Motors    │                                         └───────┬───────┘
└───────────────┘                                                 │
                                                      Anomaly or  │ Routine
                                                      New Goal    │ Navigation
                                                                  ▼
                                                          ┌───────────────┐
                                                          │ WAKE CORTEX   │
                                                          │ SOMA SLM Plan │
                                                          └───────┬───────┘
                                                                  │
                                                                  ▼
                                                          ┌───────────────┐
                                                          │  CEREBELLUM   │
                                                          │ Track & Drive │
                                                          └───────────────┘
```

1. **Interoception & Reflexes:** Brainstem reads currents and accelerometers at 1 kHz. If any threshold is breached, the robot halts or takes evasive action instantly.
2. **Sensory Ingestion:** Thalamus captures exteroception, strips uninformative background, and creates semantic state descriptors.
3. **Executive Triggering:** If no reflex is active, the Executive Core (SLM) is provided with an integrated sensory payload only when an action boundary is reached.
4. **Execution:** The SLM emits kinematic goals to the Cerebellum; the Cerebellum executes the smooth physical trajectory.
