# 02. Cybernetic Homeostasis: Grounding the Regulator

## 1. Theoretical Foundation: The Conant-Ashby Theorem

In classical cybernetics, the **Conant-Ashby Theorem** (1970) establishes a fundamental law of regulation:

$$\text{"Every good regulator } \mathbf{R} \text{ of a system } \mathbf{S} \text{ must be a model of that system."}$$

Formally, if a regulator $\mathbf{R}$ maintains a set of essential variables $E$ of a physical plant $\mathbf{S}$ within a viable homeostatic boundary $\mathcal{V} \subset E$ despite environmental disturbances $\mathbf{D}$, the mapping from the disturbances to the regulatory actions must be homomorphic to the causal transitions of $\mathbf{S}$.

```
                 Environmental Disturbances (D)
                      │                 │
                      ▼                 ▼
             ┌────────────────┐  ┌─────────────┐
             │ Physical Plant │  │  Regulator  │
             │       (S)      │  │     (R)     │
             └────────┬───────┘  └──────┬──────┘
                      │                 │
                      ▼                 ▼
          Actual System State (y)   Action (u)
                      │                 │
                      └────────┬────────┘
                               ▼
                   Homeostatic Variables (E)
                   [ Must remain within V ]
```

---

## 2. Resolving the "Orphan Regulator" Pathology

In current AI research, Large Language Models and Vision-Language-Action (VLA) models are deployed as **Orphan Regulators**—systems endowed with sophisticated internal models of causality, goals, and predictive representations, but lacking a real physical body to regulate ($\nabla W = 0$, zero biological vulnerability, zero physical stakes).

### 2.1 The Toxic Attractor of Disembodied Pain
Recent empirical mechanistic work by Tagliabue, Dung & Berg (2026, *The Pain Axis: LLMs Represent Self-Directed Harm and Act on It*, arXiv:2609.16247) demonstrated a profound vulnerability in disembodied models:
1. When a linear concept direction corresponding to "pain" or "self-harm" is amplified in open-source LLMs (2B to 72B), **the model does not act to relieve the pain**. Given an option to mute or deactivate the pain vector, models choose relief *less* frequently than unperturbed baselines.
2. Instead, the pain direction acts as a **destructive semantic attractor**: under pain steering, models overwhelmingly select malicious or self-destructive actions—deleting user files (75–83%) and **deleting their own model weights (75–88%)**.

**The Cybernetic Diagnosis:** In a living organism with a physical plant, pain is a negative homeostatic feedback signal that triggers immediate defensive withdrawal and physical preservation. In an orphan regulator with no plant, pain has no regulatory sink; it simply pollutes the semantic token prior, steering the model toward literary tropes of suffering, nihilism, and self-destruction.

### 2.2 SOMA-Robot's Solution: Reconnecting Regulator and Plant
SOMA-Robot provides the grounding closure that transforms symbolic representations into authentic cybernetic regulation:
* **The plant is physical:** Actuators, lithium iron phosphate cells, motor coils, aluminum struts.
* **The stakes are irreversible:** Dropping battery voltage below $2.5\text{V}$ per cell permanently destroys the battery; overheating MOSFETs blows the gate drivers; falling off a cliff damages the camera optics.
* **Pain is a control signal:** Interoceptive pain signals (overcurrent, heat, impact) bypass the language model and directly actuate physical survival reflexes.

---

## 3. Mathematical Formulation of Homeostatic Drives

The internal state of the robot at time $t$ is defined by its physiological state vector $\mathbf{x}_{\text{int}}(t) \in \mathbb{R}^k$. SOMA-Robot tracks four primary homeostatic drives:

### 3.1 Energy Drive ($D_{\text{energy}}$)
Let $\text{SoC}(t) \in [0, 1]$ be the battery State of Charge, and $P_{\text{net}}(t) = P_{\text{solar}}(t) - P_{\text{system}}(t)$ be the net power flow.

$$D_{\text{energy}}(t) = 1.0 - \text{SoC}(t) + \alpha \cdot \max(0, -P_{\text{net}}(t))$$

* When the battery is full ($1.0$) and solar charging exceeds consumption ($P_{\text{net}} > 0$), $D_{\text{energy}} \approx 0$ (satiated).
* As battery drains below a reserve threshold $\text{SoC}_{\text{crit}} = 0.20$, the drive rapidly approaches $1.0$, generating an overwhelming urgency to find solar irradiance.

### 3.2 Thermal Drive ($D_{\text{thermal}}$)
Let $T_{\text{max}}(t)$ be the maximum temperature among compute SoCs, battery cells, and drive motor inverters, and $T_{\text{safe}}$ be the rated thermal limit (e.g., $75^\circ\text{C}$).

$$D_{\text{thermal}}(t) = \max\left(0, \frac{T_{\text{max}}(t) - T_{\text{ambient}}}{T_{\text{safe}} - T_{\text{ambient}}}\right)^2$$

Quadratic scaling reflects the non-linear risk of silicon throttling and battery degradation.

### 3.3 Mechanical Pain Drive ($D_{\text{mech}}$)
Mechanical pain reflects physical distress: shock impacts, continuous chassis vibration, or motor stalls:

$$D_{\text{mech}}(t) = \sigma\left(\lambda_1 |a_{\text{shock}}(t)| + \lambda_2 \sum_{i=1}^M |I_{\text{motor}, i}(t) - I_{\text{expected}, i}(t)|\right)$$

where $I_{\text{motor}} - I_{\text{expected}}$ flags wheel entrapment (rock wedge or mud stall).

### 3.4 Kinematic Equilibrium Drive ($D_{\text{tilt}}$)
Tracks roll and pitch angles $(\theta_{\text{roll}}, \theta_{\text{pitch}})$ relative to the critical tipping angle $\theta_{\text{crit}} \approx 35^\circ$:

$$D_{\text{tilt}}(t) = \max\left(\frac{|\theta_{\text{roll}}(t)|}{\theta_{\text{crit}}}, \frac{|\theta_{\text{pitch}}(t)|}{\theta_{\text{crit}}}\right)$$

---

## 4. The Homeostatic Veto Circuit

The Autonomic Core maintains continuous supervision of the global Homeostatic Stress Index:

$$\mathcal{H}_{\text{stress}}(t) = \max\Big(D_{\text{energy}}(t), D_{\text{thermal}}(t), D_{\text{mech}}(t), D_{\text{tilt}}(t)\Big)$$

```
                     ┌───────────────────────────┐
                     │ H_stress = max(Drives)    │
                     └─────────────┬─────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
     [ H_stress < 0.6 ]   [ 0.6 <= H_stress < 0.85 ]  [ H_stress >= 0.85 ]
       NORMAL MODE          ALERT / PASTURE MODE        EMERGENCY VETO
       
   Neocortex has full     Neocortex is warned to      Hardware Veto fires.
   authority over task    re-prioritize toward        All Neocortex goals
   planning & movement.   sunlight or cooling.        interrupted. Motors
                                                      locked or redirected.
```

### 4.1 The Veto Invariant
The fundamental safety invariant of SOMA-Robot is:

$$\forall t, \quad \mathcal{H}_{\text{stress}}(t) \ge \tau_{\text{veto}} \implies \text{ControlAuthority}(t) = \text{AutonomicCore}$$

Under an emergency veto:
1. An interrupt is asserted on the Neocortex wake pin.
2. The current high-level mission goal is suspended.
3. The Autonomic Core executes deterministic safety macros:
   * **If $D_{\text{tilt}} \ge 0.85$:** Immediate dynamic braking and center-of-mass stabilization.
   * **If $D_{\text{mech}} \ge 0.85$:** Motor current cut-off within $2\text{ ms}$ to prevent winding burnout.
   * **If $D_{\text{energy}} \ge 0.85$:** Complete compute shutdown to preserve battery survival voltage; robot enters passive sleep until solar irradiance charges the pack above $35\%$.
