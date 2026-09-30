# SOMA-Robot Phase 0 Simulation Environment

## 1. Overview

The **Phase 0 Mental Simulator** is a lightweight, pure-Python event-driven simulation harness designed to validate the cognitive and cybernetic loop before committing to 3D robotics physics engines (Webots/Isaac Sim) or physical hardware.

It evaluates:
1. **The Conant-Ashby Loop:** Verifying that the homeostatic state machine correctly models and preserves the simulated battery, thermal, and kinematic state variables.
2. **The Autonomic Veto:** Demonstrating that physical survival interrupts always preempt language model tool calls when simulated physiological drives exceed safety thresholds.
3. **SOMA Schema Compliance:** Testing that a locally hosted language model (via Ollama, vLLM, or llama.cpp) adheres to strictly typed JSON contracts under varying operational stress.

---

## 2. Core Python Components to Implement

* `autonomic_core.py`: Pure Python implementation of the 1 kHz Homeostatic Drive computation ($D_{\text{energy}}, D_{\text{thermal}}, D_{\text{mech}}, D_{\text{tilt}}$) and the hardware veto latch.
* `mock_plant.py`: Simulated battery discharge curve, simple photovoltaic solar irradiance model (sine curve day/night cycle), and terrain friction perturbations.
* `soma_orchestrator.py`: The deterministic boundary that packages `sensory_telemetry.json`, sends it to the SLM API, validates the returned `executive_command.json`, and records the step into an append-only JSONL ledger (`audit_event.json`).
* `test_veto_invariance.py`: Automated test runner executing 1,000 synthetic failure injections (sudden shadow, boulder obstacle, motor stall) to verify zero unhandled runaways.

---

## 3. Quick Start (Concept)

```bash
# Ensure local Ollama or vLLM server is running with Llama-3.2-3B or similar
ollama run llama3.2:3b

# Run the Phase 0 simulation test suite (once scripts are placed in this directory)
python -m simulation.test_veto_invariance --model "llama3.2:3b" --steps 1000
```
