# 05. Development & Falsification Roadmap

## 1. Engineering Philosophy: Avoid Hardware Prematurely

Building physical custom robotics hardware before stabilizing the underlying cognitive and regulatory software guarantees wasted capital and sluggish iteration cycles. 

SOMA-Robot follows a **rigorous three-phase progression**:

```
PHASE 0: THE SYNTHETIC MIND (Pure Python Discrete Simulation)
- Validate homeostatic veto, SOMA JSON schemas, and SLM tool-calling locally.
- Zero hardware, zero physics engine overhead. Fast iteration on desktop CPU/GPU.
                          │
                          ▼
PHASE 1: THE VIRTUAL BODY (3D Physics & Photometric Simulation)
- Integrate ROS 2 / Zenoh into Webots or Isaac Sim.
- Sun angle tracking, shadow casting, realistic battery drain, dynamic slip.
                          │
                          ▼
PHASE 2: PHYSICAL ROVER 1:10 SCALE PROTOTYPE
- Low-cost off-the-shelf RC chassis, 30W solar panel, dual MCU/NPU compute.
- Field endurance validation under real sunlight and weather fluctuations.
```

---

## 2. Phase 0: The Synthetic Mind (Current Focus)

### Objectives
1. Implement the **Autonomic Core (Domain A)** as a deterministic Python state machine.
2. Implement mock **Interoception & Solar Telemetry generators** (simulating diurnal sun intensity, battery discharge curves, and random obstacle encounters).
3. Connect the SOMA Orchestrator to a local quantized language model (e.g., Llama-3.2-3B or Mistral-7B running via Ollama / vLLM / llama.cpp).
4. Verify that the **Homeostatic Veto Mechanism** successfully interrupts the SLM when simulated battery drops below critical thresholds or when simulated motor stall occurs.

### Falsification Milestones
* **Milestone 0.1 (Veto Invariance):** Run 1,000 synthetic simulation runs where battery depletion or catastrophic tilt is injected mid-task. Verify 100% interception by the autonomic veto with zero motor runaway.
* **Milestone 0.2 (SOMA Contract Adherence):** The local SLM achieves $\ge 99.5\%$ valid JSON schema tool emission without conversational hallucinations over 200 consecutive decision steps.
* **Milestone 0.3 (REM Distillation):** Demonstrate that an offline sleep session successfully summarizes a day's synthetic log into $< 10\%$ of original token volume while retaining 100% of recorded failure waypoints.

---

## 3. Phase 1: The Virtual Body (3D Simulation)

### Objectives
1. Build a faithful digital twin of the rover chassis in **Webots** or **NVIDIA Isaac Sim**.
2. Implement dynamic solar irradiance modeling:
   * Sun position follows real-world ephemeris equations based on virtual latitude, longitude, and time of day.
   * Virtual trees and structures cast dynamic shadows; the rover's solar panel receives power proportional to direct unoccluded beam irradiance.
3. Thalamus visual perception:
   * Synthetic RGB-D cameras generate point clouds.
   * A lightweight YOLO model segments traversable ground vs. obstacles.
4. Cerebellar kinematic execution:
   * A Model Predictive Controller (MPC) drives simulated wheel motors to follow symbolic paths provided by the Neocortex.

### Falsification Milestones
* **Milestone 1.1 (Autonomous Solar Pasturing):** When placed in a partially shaded environment with 25% battery, the rover autonomously navigates to a sunny clearing, parks, and waits until battery exceeds 75% before resuming work.
* **Milestone 1.2 (Continuous Diurnal Survival):** The rover operates for 7 continuous simulated virtual days (168 hours of simulated time) without human intervention, maintaining its battery above survival minimums through day/night cycles.

---

## 4. Phase 2: Physical Rover Scale 1:10 Prototype

### Hardware Bill of Materials (Approximate Target Budget: < $1,500)
* **Chassis:** 4WD aluminum off-road rover platform with independent shock suspension (e.g., DFRobot Devastator or custom heavy-duty RC crawler).
* **Power Harvesting:**
  * 30W – 50W rigid monocrystalline solar panel mounted horizontally.
  * Genasun or custom MPPT charge controller with I2C digital telemetry.
  * 12.8V / 24V $15\text{ Ah}$ $\text{LiFePO}_4$ battery pack with active smart BMS.
* **Autonomic Compute (Domain A):**
  * STM32H7 dual-core or Raspberry Pi RP2350 microcontroller ($< 1\text{W}$).
  * Custom relay board for hardware motor disconnect.
* **Cognitive Compute (Domain B):**
  * NVIDIA Jetson Orin Nano Developer Kit (8GB) or Raspberry Pi 5 + Hailo-8 M.2 NPU accelerator.
* **Sensory Stack:**
  * Intel RealSense D435i depth camera or dual stereo cameras.
  * RPLIDAR A1/A2 2D LiDAR.
  * 9-DOF BNO085 IMU with hardware sensor fusion.

### Field Validation Goals
* Deploy the prototype in a real outdoor agricultural / park setting.
* Validate that real-world cloud cover, thermal throttling under summer heat, and dust accumulation on panels trigger appropriate regulatory responses (pasturing, sleeping, shading).
