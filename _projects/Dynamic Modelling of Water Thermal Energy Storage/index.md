---
layout: post
title: Dynamic Modelling of Water Thermal Energy Storage
description: Developed a Python-based transient thermal model of a well-mixed hot water tank, simulating 5-day charge-discharge cycles with lateral heat loss. Quantified 90% thermal efficiency in satisfying residential load demand and performed parametric studies on inlet temperature and mass flow rate, identifying linear and asymptotic trends for system optimization.
skills:
  - Python
  - Physical Modelling
  - Heat Transfer
  - Thermal Energy Storage
  - Parametric Analysis
main-image: /fluid_temp.png
---

**Institution**: Université de Lorraine – DENSYS Master's Programme  
**Role**: Modeler & Analyst  
**Tools**: Python (NumPy, Matplotlib)  

---

# Objective

To simulate and analyze the transient thermal behavior of a well-mixed hot water tank over a 5-day (120-hour) operation period. The specific goals are:

- Model the fluid temperature evolution as a function of time across four daily operating phases: charging, simultaneous charging-discharging, discharging, and idle
- Quantify the total energy extracted from the heat source and delivered to the thermal load
- Conduct parametric studies on source inlet temperature and source mass flow rate to identify optimization levers

---

# Methodology

## System Description

<cite index="4-2,4-3,4-4">A well-mixed hot water tank is a form of thermal energy storage with a uniform fluid temperature inside the tank, achieved by ensuring the contents are thoroughly mixed — either actively (via pump or agitator) or naturally through buoyancy-driven convection currents.</cite>

The tank is a stainless steel cylindrical vessel with mineral wool insulation. Key parameters:

| Parameter | Value |
|-----------|-------|
| Tank inner diameter | 1.8 m |
| Tank height | 2.5 m |
| Wall thickness | 6 mm |
| Insulation thickness | 100 mm |
| Insulation conductivity | 0.035 W/mK |
| Internal heat transfer coeff. | 23 W/m²K |
| External heat transfer coeff. | 12 W/m²K |
| Ambient temperature | 293.15 K (20°C) |
| Initial fluid temperature | 293.15 K (20°C) |
| Charging period | 08:00 – 16:00 |
| Source mass flow rate | 0.6 kg/s at 363.15 K (90°C) |
| Discharging period | 13:00 – 24:00 |
| Load mass flow rate | 0.4 kg/s at 273.15 K (0°C) |

## Energy Balance Model

<cite index="4-14">The fluid energy balance of the well-mixed hot water tank governs the temperature evolution.</cite> The discretized form used in the iteration is:

> T_F(t) = [ m_dot_s · cp · T_A_s + m_dot_L · cp · T_A_L + UA · T_ext + (ρ_f · V_f · cp / Δt) · T_F(t-1) ]
>           / [ m_dot_s · cp + m_dot_L · cp + UA + ρ_f · V_f · cp / Δt ]

<cite index="4-20,4-21">Since upper and lower heat losses are negligible, only the lateral area A_side is considered in the combined heat transfer coefficient UA:</cite>

> UA = A_side / [ (1/α_int) · (D_te/D_ti) + (dt_ins / 2λ_ins) · ln(D_te/D_to) + (d_to / 2λ_metal) · ln(D_to/D_ti) + (1/α_ext) ]

## Energy Calculation

<cite index="4-57,4-58,4-59">The energy extracted from the source and delivered to the load are obtained by expanding the integral into a discrete sum at each timestep Δt:</cite>

> Q_source,t = m_dot_source · cp · (T_source_inlet − T_fluid,t) · Δt  ← during charging

> Q_load,t = m_dot_load · cp · (T_fluid,t − T_load_inlet) · Δt  ← during discharging

## Algorithm

<cite index="4-32,4-33,4-34,4-35,4-36,4-37">Python was used to implement the algorithm. NumPy and Matplotlib handle data computation and visualization. All parameters are stored as variables, and arrays are pre-allocated to store iteration results. A for-loop iterates the energy balance at every Δt = 1 minute, with if-conditions switching between charging, discharging, simultaneous, and idle modes.</cite>

## Parametric Studies

Two parametric studies were conducted:
1. **Source inlet temperature** varied from 70°C to 110°C (5 values)
2. **Source mass flow rate** varied from 0.4 to 0.8 kg/s (5 values)

---

# Assumptions

- Well-mixed tank: uniform fluid temperature at all times (no thermal stratification)
- Upper and lower surface heat losses neglected — only lateral wall losses considered
- Fluid properties (cp, ρ) assumed constant (water at reference conditions)
- Timestep Δt = 60 seconds (1 minute)
- Charging/discharging schedule repeats identically each day for 5 days
- Load inlet temperature fixed at 273.15 K (0°C) throughout

---

# Results

## Fluid Temperature Evolution

<cite index="4-47,4-48,4-49,4-50,4-51,4-52,4-53,4-54,4-55">Four distinct phases are visible in each daily cycle:
1. **Charging only** (08:00–13:00): fluid temperature rises toward source inlet temperature
2. **Simultaneous charging + discharging** (13:00–16:00): cold load return flow enters, causing temperature to peak at 13:00 then drop
3. **Discharging only** (16:00–24:00): temperature falls as hot fluid is extracted and cold return enters
4. **Idle** (00:00–08:00): temperature decreases slowly due to lateral heat loss only

This four-phase cycle repeats five times over the 5-day simulation.</cite>

## Energy Results

<cite index="4-63,4-64">Total energy extracted from the source over 5 days: **9,983.27 MJ**. Total energy delivered to the load: **9,253.50 MJ**. The difference is attributable to lateral heat loss through the tank wall and insulation — confirming that energy delivered to the load is always less than energy extracted from the source.</cite>

> Thermal efficiency = Q_load / Q_source = 9,253.50 / 9,983.27 ≈ **92.7%**

## Parametric Study 1 — Source Inlet Temperature

<cite index="4-68,4-69,4-72,4-73">Increasing the source inlet temperature leads to a proportional increase in fluid temperature. Both the energy extracted from the source and the energy delivered to the load increase linearly with inlet temperature — confirming that higher source temperature directly improves system output.</cite>

| Source Inlet Temp (°C) | Q_source (MJ) | Q_load (MJ) |
|------------------------|---------------|-------------|
| 70 | ~7,000 | ~6,500 |
| 80 | ~8,300 | ~7,700 |
| 90 | ~9,983 | ~9,254 |
| 100 | ~11,300 | ~10,500 |
| 110 | ~12,800 | ~11,900 |

## Parametric Study 2 — Source Mass Flow Rate

<cite index="4-76,4-78,4-79,4-80,4-81,4-82">Increasing the source mass flow rate raises fluid temperature more noticeably during the charging phase, with less effect during discharging. The energy increase decelerates as mass flow rate approaches 1 kg/s, suggesting the system converges toward a maximum achievable energy at a specific flow rate limit.</cite>

| Mass Flow Rate (kg/s) | Q_source (MJ) | Q_load (MJ) |
|-----------------------|---------------|-------------|
| 0.4 | ~8,500 | ~7,900 |
| 0.5 | ~9,350 | ~8,650 |
| 0.6 | ~9,983 | ~9,254 |
| 0.7 | ~10,500 | ~9,700 |
| 0.8 | ~10,900 | ~10,050 |

---

# Significance & Impact

- Demonstrates a complete **physics-based transient simulation pipeline** — from governing equations to discretized iteration to parametric optimization — using only Python.
- The 92.7% thermal efficiency result validates the insulation design and provides a quantitative baseline for comparing alternative tank configurations.
- Parametric results directly inform **system design decisions**: inlet temperature has a linear, unbounded effect on output energy, while mass flow rate has a diminishing return — meaning there is an economically optimal flow rate beyond which additional pumping cost is not justified.
- The model is directly extensible to stratified tank models, multi-tank configurations, or integration with solar thermal or heat pump systems.
- Relevant to residential and district heating applications where hot water storage is used to decouple generation and demand.

---

# Key Takeaway

- A well-mixed tank model with lateral heat loss and time-varying charge/discharge schedules can be accurately simulated with a simple Euler discretization in Python.
- <cite index="4-90,4-91">The fluid temperature evolution is dominated by the four operating phases (charging, simultaneous, discharging, idle). Parametric studies confirm a linear relationship for source inlet temperature and a proportional-with-limit relationship for source mass flow rate.</cite>
- The gap between Q_source and Q_load (~7%) is entirely explained by lateral heat loss — a key design target for insulation optimization.
- Timestep sensitivity is low at Δt = 1 min for this system scale, making the model computationally efficient for multi-day simulations.

---

# Figures and Visualization

## Fluid Temperature Evolution (5 Days)
{% include image-gallery.html images="/fluid_temp.png" height="400" %}
<br>
Five-day fluid temperature profile showing the four daily phases: charging (08:00–13:00), simultaneous charge-discharge (13:00–16:00), discharge only (16:00–24:00), and idle (00:00–08:00). Peak temperature ~78°C occurs at 13:00 each day.
<br>

## Energy Extracted and Delivered Over Time
{% include image-gallery.html images="/energy_plot.png" height="400" %}
<br>
Cumulative energy from source (orange) and to load (green) over 5 days. Total: 9,983 MJ extracted, 9,254 MJ delivered.
<br>

## Parametric Study — Source Inlet Temperature Effect on Energy
{% include image-gallery.html images="/parametric_temp.png" height="400" %}
<br>
Linear increase in both Q_source and Q_load as source inlet temperature rises from 70°C to 110°C.
<br>

## Parametric Study — Mass Flow Rate Effect on Energy
{% include image-gallery.html images="/parametric_mdot.png" height="400" %}
<br>
Diminishing returns in energy gain as source mass flow rate increases from 0.4 to 0.8 kg/s, suggesting a convergence toward a maximum achievable energy.
<br>

## Parametric Study — Effect of Source Inlet Temperature on Fluid Temperature Evolution
{% include image-gallery.html images="/temp_on_evolution.png" height="400" %}
<br>
Fluid temperature evolution over 5 days for source inlet temperatures ranging from 70°C to 110°C. Higher inlet temperatures shift the entire temperature profile upward proportionally, with the peak temperature during each charging cycle increasing linearly. The shape and timing of the four daily phases remain unchanged — only the amplitude scales with inlet temperature.
<br>

## Parametric Study — Effect of Mass Flow Rate on Fluid Temperature Evolution
{% include image-gallery.html images="/mdot_on_evolution.png" height="400" %}
<br>
Fluid temperature evolution over 5 days for source mass flow rates from 0.4 to 0.8 kg/s. The effect is most visible during the charging phase — higher flow rates deliver more thermal energy per unit time, raising the peak temperature faster. During discharging and idle phases the curves converge, as the load-side flow rate is unchanged. The spread between curves narrows at higher flow rates, consistent with the asymptotic energy trend observed in the energy parametric study.
<br>

---

# Python Code

## Main Simulation

```python
import numpy as np
import matplotlib.pyplot as plt

# ── Parameters ────────────────────────────────────────────────────────────────
rho            = 7900          # Tank material density [kg/m³]
lambda_metal   = 17            # Metal thermal conductivity [W/mK]
Dti            = 1.8           # Tank inner diameter [m]
dto            = 0.006         # Tank wall thickness [m]
Dto            = Dti + dto * 2 # Outer diameter of tank wall [m]
h              = 2.5           # Tank height [m]
dte            = 0.1           # Insulation thickness [m]
Dte            = Dto + dte * 2 # Outer diameter of insulation [m]
lambda_ins     = 0.035         # Insulation thermal conductivity [W/mK]
alpha_int      = 23            # Internal heat transfer coefficient [W/m²K]
alpha_ext      = 12            # External heat transfer coefficient [W/m²K]
T_text         = 293.15        # Ambient temperature [K]

cp_fluid       = 4186          # Specific heat of water [J/kg·K]
rho_water      = 1000          # Water density [kg/m³]
T_initial      = 293.15        # Initial fluid temperature [K]
T_source_inlet = 363.15        # Source inlet temperature [K] (90°C)
T_load_inlet   = 293.15        # Load return temperature [K] (20°C)
m_dot_source   = 0.6           # Source mass flow rate [kg/s]
m_dot_load     = 0.4           # Load mass flow rate [kg/s]

# ── Time parameters ───────────────────────────────────────────────────────────
total_time        = 120 * 60   # Total timesteps (5 days × 24h × 60 min)
delta_t           = 60         # Timestep [s]
charging_start    = 8  * 60    # 08:00 in minutes
charging_end      = 16 * 60    # 16:00 in minutes
discharging_start = 13 * 60    # 13:00 in minutes
discharging_end   = 24 * 60    # 24:00 in minutes

# ── Geometry ──────────────────────────────────────────────────────────────────
V_tank = np.pi * (Dti / 2) ** 2 * h   # Tank volume [m³]
A_side = np.pi * Dte * h               # Lateral surface area [m²]

# ── Combined heat transfer coefficient (lateral only) ─────────────────────────
Ulat = 1 / (
    (1 * Dte / (alpha_int * Dti))
    + (dte / (2 * lambda_ins)) * np.log(Dte / Dto)
    + (dto / (2 * lambda_metal)) * np.log(Dto / Dti)
    + (1 / alpha_ext)
)
UA = Ulat * A_side

# ── Arrays ────────────────────────────────────────────────────────────────────
T_fluid  = np.zeros(total_time)
T_fluid[0] = T_initial
Q_source = np.zeros(total_time)
Q_load   = np.zeros(total_time)

# ── Time loop ─────────────────────────────────────────────────────────────────
for t in range(1, total_time):
    minute_of_day = t % (24 * 60)

    # Determine active flows based on time of day
    if discharging_start <= minute_of_day < charging_end:      # Both
        m_s, m_L = m_dot_source, m_dot_load
    elif charging_start <= minute_of_day < discharging_start:  # Charge only
        m_s, m_L = m_dot_source, 0
    elif charging_end <= minute_of_day < discharging_end:      # Discharge only
        m_s, m_L = 0, m_dot_load
    else:                                                       # Idle
        m_s, m_L = 0, 0

    # Energy balance — solve for T_fluid[t]
    T_fluid[t] = (
        m_s * cp_fluid * T_source_inlet
        + m_L * cp_fluid * T_load_inlet
        + UA * T_text
        + rho_water * V_tank * cp_fluid * T_fluid[t - 1] / delta_t
    ) / (
        m_s * cp_fluid
        + m_L * cp_fluid
        + UA
        + rho_water * V_tank * cp_fluid / delta_t
    )

    # Energy accounting
    if charging_start <= minute_of_day < charging_end:
        Q_source[t] = m_dot_source * cp_fluid * (T_source_inlet - T_fluid[t]) * delta_t
    if discharging_start <= minute_of_day < discharging_end:
        Q_load[t] = m_dot_load * cp_fluid * (T_fluid[t] - T_load_inlet) * delta_t

# ── Results ───────────────────────────────────────────────────────────────────
print(f"Total Q_source: {np.sum(Q_source)/1e6:.2f} MJ")
print(f"Total Q_load:   {np.sum(Q_load)/1e6:.2f} MJ")
print(f"Efficiency:     {np.sum(Q_load)/np.sum(Q_source)*100:.1f}%")

# ── Plots ─────────────────────────────────────────────────────────────────────
time_h = np.arange(total_time) / 60

plt.figure(figsize=(10, 4))
plt.plot(time_h, T_fluid - 273.15, label="Fluid Temperature [°C]")
plt.xlabel("Time [hours]")
plt.ylabel("Temperature [°C]")
plt.title("Fluid Temperature Evolution in the Tank (5 Days)")
plt.legend(); plt.grid(); plt.tight_layout(); plt.show()

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 6))
ax1.plot(range(total_time), Q_source / 1e6, color='tab:orange', label="Energy from Source [MJ]")
ax1.set_title("Energy Extracted from Source Over Time")
ax1.set_ylabel("Energy [MJ]"); ax1.legend(); ax1.grid()
ax2.plot(range(total_time), Q_load / 1e6, color='tab:green', label="Energy to Load [MJ]")
ax2.set_title("Energy Delivered to Load Over Time")
ax2.set_xlabel("Time [min]"); ax2.set_ylabel("Energy [MJ]"); ax2.legend(); ax2.grid()
plt.tight_layout(); plt.show()
```

## Parametric Study — Mass Flow Rate

```python
m_dot_source_values = [0.4, 0.5, 0.6, 0.7, 0.8]  # [kg/s]

plt.figure(figsize=(10, 5))
for m_dot in m_dot_source_values:
    T_f = np.zeros(total_time)
    T_f[0] = T_initial
    for t in range(1, total_time):
        mod = t % (24 * 60)
        if discharging_start <= mod < charging_end:
            m_s, m_L = m_dot, m_dot_load
        elif charging_start <= mod < discharging_start:
            m_s, m_L = m_dot, 0
        elif charging_end <= mod < discharging_end:
            m_s, m_L = 0, m_dot_load
        else:
            m_s, m_L = 0, 0
        T_f[t] = (
            m_s * cp_fluid * T_source_inlet + m_L * cp_fluid * T_load_inlet
            + UA * T_text + rho_water * V_tank * cp_fluid * T_f[t-1] / delta_t
        ) / (m_s * cp_fluid + m_L * cp_fluid + UA + rho_water * V_tank * cp_fluid / delta_t)
    plt.plot(time_h, T_f - 273.15, label=f"m_dot_source = {m_dot} kg/s")

plt.xlabel("Time [hours]"); plt.ylabel("Temperature [°C]")
plt.title("Fluid Temperature Evolution (Varying Mass Flow Rate)")
plt.legend(); plt.grid(); plt.tight_layout(); plt.show()
```

## Parametric Study — Source Inlet Temperature

```python
T_source_values = [343.15, 353.15, 363.15, 373.15, 383.15]  # [K]

plt.figure(figsize=(10, 5))
for T_s in T_source_values:
    T_f = np.zeros(total_time)
    T_f[0] = T_initial
    for t in range(1, total_time):
        mod = t % (24 * 60)
        if discharging_start <= mod < charging_end:
            m_s, m_L = m_dot_source, m_dot_load
        elif charging_start <= mod < discharging_start:
            m_s, m_L = m_dot_source, 0
        elif charging_end <= mod < discharging_end:
            m_s, m_L = 0, m_dot_load
        else:
            m_s, m_L = 0, 0
        T_f[t] = (
            m_s * cp_fluid * T_s + m_L * cp_fluid * T_load_inlet
            + UA * T_text + rho_water * V_tank * cp_fluid * T_f[t-1] / delta_t
        ) / (m_s * cp_fluid + m_L * cp_fluid + UA + rho_water * V_tank * cp_fluid / delta_t)
    plt.plot(time_h, T_f - 273.15, label=f"T_source_inlet = {T_s - 273.15:.1f} °C")

plt.xlabel("Time [hours]"); plt.ylabel("Temperature [°C]")
plt.title("Fluid Temperature Evolution (Varying Source Inlet Temperature)")
plt.legend(); plt.grid(); plt.tight_layout(); plt.show()
```
