---
layout: post
title: Energy Management and Distributed Control of AC/DC Microgrids
description: Designed and simulated hierarchical energy management systems for both DC and AC microgrids using MATLAB/Simulink. Covered bandwidth-separated power flow control for DC hybrid systems (fuel cell, supercapacitor, battery, PV), droop-based distributed control for AC microgrid clusters, and cascaded inverter control in grid-forming and grid-following modes — validated through multiple Simulink lab sessions.
skills:
  - MATLAB/Simulink
  - Hierarchical Control Design
  - Droop Control
  - DC/DC Converter Modeling
  - Microgrid Energy Management
  - Power Electronics
main-image: /mainvisual.png
---

**Institution**: Université de Lorraine – DENSYS Master's Programme  
**Supervisors**: Prof. S. Pierefederici, Prof. M. Bahrami  
**Role**: Co-author & Simulation Engineer  
**Tools**: MATLAB/Simulink  

---

# Objective

To design, model, and validate hierarchical energy management and distributed control strategies for both islanded DC microgrids and AC microgrid clusters. The work addresses two core challenges:

1. **DC Microgrids** — How to coordinate multiple energy sources (fuel cell, supercapacitor, battery, PV) with different power/energy densities to maintain DC bus stability and manage state of charge across all timescales.
2. **AC Microgrids** — How to achieve equal active and reactive power sharing among distributed generators (DGs) using droop control, restore frequency and voltage to nominal values without centralized communication, and design cascaded inverter controllers for both grid-forming and grid-following operation.

---

# Methodology

## Part 1 — DC Microgrid Energy Management

The DC microgrid control strategy is based on **bandwidth separation** (hierarchical control), where each energy source is assigned a control loop operating at a different timescale, matched to its power/energy density characteristics as visualized on the Ragone diagram.

**Control hierarchy (fastest to slowest):**
1. **Inner loop** — Supercapacitor (SC) regulates DC bus voltage. Responds to fast transients.
2. **Middle loop** — Battery regulates SC state of charge. Operates at intermediate timescale.
3. **Outer loop** — Fuel cell regulates battery SOC. Responds to slow, sustained energy demands.
4. **PV** — Treated as an uncontrolled intermittent source (MPPT tracked, not used for control).

Each control loop implements a **second-order error dynamics** law, where the error ε_k for each source k must satisfy:

> ε̇_k(t) + 2·ξ_k·ω_k·ε_k(t) + ω_k² · ∫ε_k(τ)dτ = 0

with the condition **ω_k >> ω_(k+1)** ensuring bandwidth separation between layers. ξ_k is the damping ratio (0.7 for good transient behavior) and ω_k is the natural frequency of loop k.

**Three lab configurations were simulated:**
- **Lab 1**: Fuel Cell + Supercapacitor hybrid with current load
- **Lab 2**: PV + Fuel Cell + Supercapacitor + Battery hybrid with constant power load (CPL)
- **Lab 3**: DC/DC Boost Converter with PWM switching model, average model, and cascaded current/voltage controllers

**Load types modeled:**
- Resistive load
- Current load
- Constant Power Load (CPL) — nonlinear, destabilizing behavior used to stress-test controllers

## Part 2 — AC Microgrid Distributed Control

The AC microgrid work focuses on a T-shaped cluster with two grid-forming DGs, progressing from classical droop control to enhanced reactive power sharing and secondary frequency/voltage restoration.

**Classical Droop Control:**

Active power sharing is achieved through frequency droop:

> ω_i = ω_0 − m_i · (P_i − P_in)

Reactive power sharing is achieved through voltage droop:

> V_i = V_0 − n_i · (Q_i − Q_in)

**Limitation:** classical droop achieves equal active power sharing but **not** equal reactive power sharing due to mismatched line impedances between DGs and the PCC.

**Four enhanced reactive power sharing strategies were analyzed and simulated:**
1. **Virtual Impedance Method** — Introduces artificial inductive impedance in the control loop to equalize effective output impedances across DGs.
2. **Virtual Power Droop Control** — Redefines P and Q using a rotation matrix based on line impedance angle, compensating for network effects.
3. **Non-linear Droop Coefficient Method** — Adds a time-varying decoupling term J_i to the voltage droop equation to eliminate P-Q coupling.
4. **Q-Vest Droop with PCC Voltage Estimation** — Estimates PCC RMS voltage from DG output measurements and feeds the error into an integrator to correct the voltage reference. Steady-state condition:

> K_v · (V_n − V_0_Est) = n_i · (Q_i − Q_in)

**Secondary Control — Frequency Restoration (without communication network):**
- A secondary control variable X_ωi is added to the P-F droop law: **ω_i = ω_n − m_i·(P_i − P_in) − X_ωi**
- Its dynamics ensure all DG frequencies converge to nominal: **dX_ω/dt = −L·K1·X_ω + K2·(ω − ω_n·I2)**
- DG1 acts as frequency leader; DG2 synchronizes via angular error comparison.
- Result: all DG frequencies converge to nominal (~20s settling time) while preserving active power sharing.

**Secondary Control — Voltage Restoration at PCC:**
- A distributed consensus algorithm updates a voltage correction variable X_v for each DG: **dX_v/dt = −K_Pv·(E_n − X) + K_Iv·Y_v**
- An auxiliary consensus loop synchronizes all DGs: **dY_v/dt = −L_w·X_v − f·Y_v**
- Result: PCC voltage restored to 110V nominal; equal reactive power sharing preserved.

**Inverter Modeling and Controller Design:**
- Three-phase inverters modeled in the dq-frame using Clarke and Park transformations.
- **Grid-Forming (GFM)**: Cascaded outer voltage loop + inner current loop. Establishes grid voltage and frequency autonomously. Supports droop control and Virtual Synchronous Machine (VSM) strategies.
- **Grid-Following (GFL)**: Outer power control loop (P & Q references) + inner current loop + Phase-Locked Loop (PLL) for grid synchronization. Acts as a controlled current source.

---

# Assumptions

- Converter efficiencies assumed close to 1 for system-level analysis in Labs 1 & 2 (controlled source models).
- Lab 3 uses a full average model and switching model for the boost converter.
- AC microgrid network assumed inductive (typical for microgrids), enabling linearized P-Q decoupling.
- Perfect knowledge of line impedance angles assumed for virtual power droop implementation.
- PV generation treated as intermittent feedforward input — not used as a control source.
- Damping ratio ξ = 0.7 used for better transient behavior across all second-order controllers.
- Bandwidth condition: ω_0 < ω_s / 10, where ω_s is the switching pulsation.

---

# Results

## DC Microgrid — Lab 1 (FC + SC)

- DC bus voltage maintained stable throughout, with minimal overshoot during step load changes at t = 10s.
- SC responded immediately to load transients (fast inner loop), then returned to nominal as FC gradually took over.
- FC output increased slowly and steadily, confirming its role as the long-term energy source (outer loop).
- Power references and actual outputs closely matched, validating the second-order control law implementation.

## DC Microgrid — Lab 2 (PV + FC + SC + Battery, CPL)

Load events applied at: [0, 6, 6.01, 15, 15.01, 20, 20.01, 25, 25.01, 30] seconds.

| Layer | Source | Behavior |
|-------|--------|----------|
| Fast inner loop | Supercapacitor | Immediate voltage stabilization, power spike then recovery |
| Medium loop | Battery | Gradual energy rebalancing after SC response |
| Slow outer loop | Fuel Cell | Slow ramp-up for long-term SOC compensation |
| Uncontrolled | PV | Step-down at t ≈ 20s simulating solar drop — system adapted correctly |

- Battery SOC dropped from t ≈ 20–25s, then recovered as FC recharged it — validating correct energy flow sequencing.
- Vdc remained stable throughout all CPL disturbances — no overshoot or oscillation.

## DC Microgrid — Lab 3 (Boost Converter)

| Feature | PWM Switching Model | Average Model |
|---------|---------------------|---------------|
| Output waveform | High-frequency ripple, sharp transients | Smooth, no switching ripple |
| Startup | Sharp current/voltage peaks | Smooth rise |
| Load step response | Fast with small oscillations | Immediate, smooth settling |
| Simulation time | Long (small timesteps) | Fast (large timesteps possible) |

- Duty cycle settled at ~40% steady state; responded rapidly to load step at 0.2s.
- Output voltage regulated to 65V with small transient at load disturbance (0.35s), quickly recovered.
- Switching, average, and mathematical models showed close alignment — validating the average model approach.

## AC Microgrid — Reactive Power Sharing

- Classical droop: equal active power sharing achieved, reactive power sharing unequal.
- Q-Vest droop: reactive power converged equally between DG1 and DG2 after ~12s simulation time.

## AC Microgrid — Frequency & Voltage Restoration

- Frequency restoration: DG1 and DG2 frequencies restored to nominal values after ~20s.
- Voltage restoration: PCC RMS voltage successfully restored to 110V nominal after consensus algorithm activated at t = 25s. DG voltages stabilized near (but not exactly at) nominal due to different line impedances.
- Equal reactive power sharing confirmed to be preserved throughout voltage restoration.

---

# Significance & Impact

- Demonstrates mastery of **multi-timescale hierarchical control** — a core technique in modern microgrid and energy storage system design.
- The bandwidth separation approach is directly applicable to real-world hybrid energy systems (EV charging stations, island microgrids, industrial DC buses).
- The AC droop control work covers the full progression from classical to enhanced strategies, including communication-free secondary control — relevant to resilient and decentralized grid architectures.
- Cascaded inverter controller design (GFM/GFL) reflects industry-standard practice for grid-connected and islanded power electronics systems.
- Validated across multiple Simulink models including switching-level, average-model, and controlled-source representations — demonstrating multi-fidelity modeling competence.

---

# Key Takeaway

- Hierarchical control with bandwidth separation is a powerful and systematic framework for coordinating hybrid energy sources — each source handles disturbances at its natural timescale.
- Constant Power Loads (CPLs) introduce nonlinear destabilizing dynamics that stress-test controller robustness — a critical consideration in aircraft and EV applications.
- Classical droop control achieves active power sharing but fails for reactive power — enhanced methods (virtual impedance, Q-Vest, virtual power droop) each offer different trade-offs between complexity, parameter knowledge, and performance.
- Frequency and voltage can both be restored to nominal values using distributed consensus algorithms without a central controller, while preserving power sharing — a key enabler for resilient microgrids.
- Grid-forming inverters establish the grid; grid-following inverters inject power into it — both are essential and complementary in modern power systems.
- Average models enable fast, accurate simulation for control design without switching-level complexity, consistent with the energetic-based control design methodology.

---

# Figures and Visualization

## DC Microgrid — Lab 1 Simulation Results (FC + SC)
{% include image-gallery.html images="/mainvisual.png" height="450" %}
<br>
Simulation scope output showing power flows (P_load, P_FC, P_SC), reference currents (iFCRef, iSCRef), power references vs actuals, and DC/SC/FC voltages. SC handles fast transients at t = 10s; FC ramps up slowly to restore SC energy.
<br>
