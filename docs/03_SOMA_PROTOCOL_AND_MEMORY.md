# 03. SOMA Protocol & Memory Architecture

## 1. The SOMA Philosophy in Embodied Robotics

In the software agent domain, **SOMA** (*Structured Operative Memory and Autonomic Architecture*) is founded on a core tenet:

> **"Dumb Orchestrator, Sovereign LLM."**

In traditional agent systems, brittle external Python scripts attempt to micromanage the language model through heuristic prompting, leading to context pollution and objective drift. SOMA reverses this:
* The **Orchestrator** is strictly deterministic, unopinionated, and enforcing: it manages hardware timers, verifies typed schemas, enforces safety boundaries, and logs telemetry.
* The **LLM** is given sovereign visibility into its own operative state—including an explicit dashboard of its token budget, battery reserves, sensory confidence, and memory tiers.

When applied to physical robotics, SOMA elevates this principle: **energy and compute are treated as formal context parameters**, and motor actions are governed by strictly typed contracts.

---

## 2. The 4-Tier Memory Hierarchy (L1 to L4)

The memory system prevents cognitive saturation by separating real-time sensory bandwidth from long-term conceptual distillation:

```
┌────────────────────────────────────────────────────────────────────────┐
│ L1: THALAMIC PERCEPTUAL SCRATCHPAD                                      │
│ Lifespan: 100 ms – 5 seconds (Volatile RAM)                            │
│ Content: Immediate exteroceptive symbols (detected obstacles,          │
│          current pitch/roll, instantaneous solar Watts, camera tokens).│
│ Governance: Overwritten continuously by Thalamus filtering pipeline.   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Promoted on anomaly or goal change
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ L2: WORKING EXECUTIVE CONTEXT (Neocortex Context Window)               │
│ Lifespan: Duration of current sub-mission (minutes to hours)           │
│ Content: Current human goal, active kinematic plan, active tool-call   │
│          results, remaining energy budget, and reasoning scratchpad.   │
│ Governance: Governed directly by the Sovereign SLM.                    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Key state checkpoints & waypoints
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ L3: TOPOLOGICAL & SPATIAL RESOURCE MAP (Hippocampus Graph)             │
│ Lifespan: Persistent across operational sessions (Local SQLite / Graph)│
│ Content: Metric/Topological graph annotated with solar irradiance      │
│          history, soil compaction, slope hazards, and safe haven zones.│
│ Governance: Updated asynchronously by SLAM and navigation controllers. │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Consolidated during Solar REM sleep
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ L4: EPISODIC FLIGHT RECORDER & LONG-TERM DISTILLED ARCHIVE             │
│ Lifespan: Permanent / Append-Only (Flash storage / NVMe)               │
│ Content: Cryptographically hashed immutable audit ledger of every      │
│          high-level command, sensory trigger, and failure incident.    │
│ Governance: Read-only for the LLM; write-only for the Orchestrator.    │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Strict Typed Contracts: The Inter-Layer Bus

The Neocortex (SLM) is never permitted to emit raw unstructured conversational text to the hardware. All communications pass through **Strict JSON Schema Contracts**:

### 3.1 Inbound Telemetry Payload (Orchestrator $\to$ SLM)
Before the SLM is invoked to reason or plan, the SOMA Orchestrator constructs an aggregated state snapshot:

```json
{
  "timestamp_utc": "2026-09-30T10:14:02Z",
  "wake_trigger": "WAYPOINT_REACHED",
  "homeostasis": {
    "battery_soc": 0.38,
    "solar_input_watts": 85.4,
    "net_power_watts": 42.1,
    "max_core_temp_c": 52.0,
    "stress_index": 0.42
  },
  "thalamus_percepts": [
    {"class": "unpaved_track", "heading_deg": 12.0, "clearance_m": 8.5},
    {"class": "dense_foliage", "heading_deg": 45.0, "clearance_m": 1.8}
  ],
  "hippocampus_context": {
    "nearest_solar_haven": "Waypoint_42_SouthRidge",
    "distance_to_haven_m": 45.0,
    "active_mission": "Perimeter Survey Sector C"
  }
}
```

### 3.2 Outbound Executive Directive (SLM $\to$ Cerebellum / Hippocampus)
The SLM must respond with a valid schema-compliant tool call:

```json
{
  "reasoning_trace": "Battery is at 38% but net power is positive (+42W). We have sufficient margin to complete Sector C survey before pasturing at SouthRidge.",
  "executive_action": {
    "command": "NAVIGATE_TRAJECTORY",
    "parameters": {
      "target_waypoint": "Sector_C_Gate",
      "speed_mps": 0.6,
      "energy_contingency": "HALT_IF_SOC_BELOW_0.25",
      "obstacle_clearance_margin_m": 0.5
    }
  },
  "memory_update": {
    "annotate_waypoint": "Dense foliage encroaching at heading 45; recommend reduced velocity on return."
  },
  "next_wake_condition": "ON_DESTINATION_OR_TRACTION_LOSS"
}
```

---

## 4. The Immutable Flight Recorder (Black Box Audit Ledger)

A foundational tenet of SOMA safety is **Causal Attribution**:

> **"An autonomous agent must never possess the privilege to edit, truncate, or rewrite its own audit trail."**

### 4.1 Ledger Mechanics
1. **Append-Only Hardware Bus:** The telemetry logger runs in the Autonomic Core (Domain A). Every state transition, sensor trigger, and SLM tool call is written to a ring-buffered flash memory.
2. **Cryptographic Hashing:** Each record includes the SHA-256 hash of the preceding record ($H_t = \text{Hash}(H_{t-1} \parallel \text{Event}_t)$), rendering retroactive tampering impossible.
3. **Forensic Replay:** If the robot suffers a hardware fault, rolls over, or enters an unrecoverable state, engineers can perform a deterministic contrafactual replay:
   $$\text{Trace} = \Big\{\langle L1_t, \text{Prompt}_t, \text{SLM\_Output}_t, \text{ActuatorResponse}_t \rangle\Big\}_{t=0}^T$$

### 4.2 Pre-Execution Safety Gates
Even when the SLM produces a syntactically valid JSON action, the SOMA Orchestrator executes a **fail-closed pre-execution sanity check** before passing commands to the Cerebellum:
* **Blast Radius Check:** Does the target velocity exceed the physical deceleration capacity given the measured obstacle distance?
* **Energy Envelope Check:** Does the path length require more energy than $\text{SoC} - \text{SoC}_{\text{reserve}}$?
* **Permission Gate:** Does the action attempt to command an actuator that is currently locked out by a thermal or mechanical safety alert?
