---
layout: post
title: Li-ion Battery Equivalent Circuit Model for BESS and EV Applications
description: Built a physics-based Equivalent Circuit Model (ECM) of a Li-ion battery cell in MATLAB/Simulink, capturing SOC dynamics, terminal voltage, and thermal behavior across a full temperature range (-25°C to 45°C). Validated against commercial charge/discharge test data to establish a baseline for predictive maintenance and battery management system (BMS) design.
skills:
  - MATLAB/Simulink
  - Battery Modeling (ECM)
  - SOC Estimation
  - Thermal Modeling
  - Data-driven Validation
main-image: /simulink_model.png
---

**Institution**: Université de Lorraine – DENSYS Master's Programme  
**Role**: Modeler & Simulation Engineer  
**Tools**: MATLAB/Simulink  

---

# Objective

To develop a high-fidelity Equivalent Circuit Model (ECM) of a Li-ion battery cell that accurately captures:
- Terminal voltage as a function of State of Charge (SOC) and temperature
- Transient electrochemical dynamics via a dual RC network
- Thermal behavior under charge/discharge cycling
- Separate charge and discharge characteristics

The model is validated against commercial test data across 8 temperature operating points, establishing a reusable baseline for Battery Energy Storage System (BESS) and Electric Vehicle (EV) battery management and predictive maintenance applications.

---

# Methodology

## Equivalent Circuit Model Architecture

The ECM uses a **2nd-order RC network** (Dual Polarization model) to represent the electrochemical dynamics of the cell. The circuit consists of:

- **OCV(SOC, T)** — Open Circuit Voltage as a function of SOC and temperature, implemented as a 2D lookup table (LUT) interpolated from experimental data
- **R0** — Internal ohmic resistance (separate values for charge and discharge)
- **R1-C1** — First RC pair representing fast electrochemical polarization
- **R2-C2** — Second RC pair representing slow diffusion polarization
- **Rsd** — Self-discharge resistance

All resistances and capacitances have separate charge (`c`) and discharge (`d`) parameterizations, switched based on current direction.

**Terminal voltage equation:**

> V_cell = OCV(SOC, T) − R0 · I − V_R1C1 − V_R2C2

## SOC Estimation

SOC is tracked using **Coulomb counting** with corrections for:
- Temperature-dependent capacity adjustment (k_temp)
- Cycle aging factor (k_cycle)
- Coulombic efficiency (η_c = 0.99)

> SOC(t) = SOC_0 − (1 / (Qr · k_cycle · k_temp)) · ∫ I · η_c dt

Operating bounds enforced: V_min = 2.0 V, V_max = 3.6 V

## OCV Lookup Table Construction

Experimental charge and discharge OCV-SOC curves were loaded from Excel test data files across **8 temperature breakpoints**: -25°C, -15°C, -5°C, 5°C, 15°C, 25°C, 35°C, 45°C.

Each curve was interpolated onto a uniform SOC grid (step = 0.01, from 0 to 1) using linear interpolation with extrapolation. The resulting 2D LUTs (`LUT_charge`, `LUT_discharge`) are indexed by both SOC and temperature in Simulink.

## Thermal Model

A lumped thermal model tracks cell temperature using Newton's law of cooling:

> M · Cp · dT/dt = Q_gen − hf · A_surface · (T − T_amb)

Where:
- Q_gen = I² · R0 — Joule heating from internal resistance
- Cell geometry: cylindrical, D = 25.85 mm, H = 65.15 mm
- hf = 5 W/m²/K — convective heat transfer coefficient
- T_amb = 25°C, T_0 = 30°C

## Simulink Model Structure

The Simulink model is organized into three subsystems:

1. **Electrical subsystem (blue)** — Dual RC ECM with charge/discharge switching, OCV LUT, terminal voltage calculation, and cell current/voltage outputs
2. **SOC calculator (top left)** — Coulomb counting block with capacity correction inputs
3. **Thermal subsystem (orange)** — Lumped thermal model with convective cooling, internal heat source, and temperature output fed back to the OCV LUT

Validation stop conditions halt the simulation when terminal voltage exceeds V_max or drops below V_min.

---

# Assumptions

- Rated capacity: Qr = 2.5 Ah at reference temperature
- All RC parameters assumed constant (not SOC- or temperature-dependent in this baseline model)
- R0 = R1 = R2 = 2 mΩ (charge and discharge)
- C1 = C2 = 1000 F
- Self-discharge resistance: Rsd = 1000 Ω
- Coulombic efficiency: η_c = 0.99
- Cell mass: 70 g, specific heat: Cp = 1360 J/kg/K
- Thermal model assumes uniform cell temperature (lumped)
- OCV hysteresis between charge and discharge captured via separate LUTs

---

# Results

- Terminal voltage profiles matched commercial validation data across all 8 temperature operating points, confirming the OCV LUT interpolation approach is accurate.
- The dual RC network successfully captured both fast (electrochemical) and slow (diffusion) polarization transients during charge/discharge cycling.
- Thermal model showed realistic temperature rise under load, with steady-state temperature determined by the balance between Joule heating and convective cooling.
- SOC estimation via Coulomb counting remained accurate within the validated operating window (SOC_0 = 0.1 to full charge).
- Separate charge/discharge parameterization correctly reproduced the OCV hysteresis characteristic of Li-ion cells.
- Voltage cutoff conditions (V_min, V_max) triggered correctly, validating the protection logic for BESS/EV applications.

---

# Significance & Impact

- Provides a **reusable, validated ECM baseline** that can be directly integrated into Battery Management System (BMS) design workflows.
- The temperature-dependent OCV LUT approach is scalable — additional temperature points or aging data can be incorporated without restructuring the model.
- Establishes the foundation for **predictive maintenance algorithms**: SOC estimation accuracy and thermal behavior are the two primary inputs for State of Health (SOH) and remaining useful life (RUL) prediction.
- Directly applicable to BESS sizing studies and EV powertrain simulation, where accurate cell-level models are needed before scaling to pack level.
- Demonstrates the full modeling pipeline: raw experimental data → interpolated LUT → physics-based Simulink model → validation against test data.

---

# Key Takeaway

- A 2nd-order RC ECM with temperature-dependent OCV LUTs provides a good balance between model fidelity and computational efficiency for BMS and BESS applications.
- Separate charge/discharge parameterization is essential to capture OCV hysteresis — a common source of SOC estimation error in simpler models.
- Thermal coupling to the electrical model is critical: temperature affects OCV, capacity, and resistance, making it a key variable for accurate performance prediction across real-world operating conditions.
- Coulomb counting is simple and effective for SOC tracking within a validated operating window, but requires accurate initial SOC and efficiency parameters.

---

# Figures and Visualization

## Simulink Model — Full ECM Overview
{% include image-gallery.html images="/simulink_model.png" height="500" %}
<br>
The full Simulink model showing the electrical subsystem (blue, dual RC network with charge/discharge switching and OCV LUT), SOC calculator (top left), and thermal subsystem (orange, lumped Newton cooling model). Validation stop conditions are visible on the right.
<br>

---

# MATLAB Code

## Parameter Initialization Script

```matlab
clear all
clc

%% Parameters
Qr = 2.5;           % Ah  - Rated capacity
T = 35;             % °C  - Temperature
R0c = 2/10^3;       % Ohm - Internal resistance (charge)
R0d = 2/10^3;       % Ohm - Internal resistance (discharge)
R1c = 2/10^3;       % Ohm - RC1 resistance (charge)
R1d = 2/10^3;       % Ohm - RC1 resistance (discharge)
R2c = 2/10^3;       % Ohm - RC2 resistance (charge)
R2d = 2/10^3;       % Ohm - RC2 resistance (discharge)
C1c = 10^3;         % F   - RC1 capacitance (charge)
C1d = 10^3;         % F   - RC1 capacitance (discharge)
C2c = 10^3;         % F   - RC2 capacitance (charge)
C2d = 10^3;         % F   - RC2 capacitance (discharge)
Rsd = 10^3;         % Ohm - Self-discharge resistance

%% Capacity adjustment factors
kcycle = 1;         % Cycle aging factor
ktemp  = 1;         % Temperature derating factor
SOC0   = 0.1;       % Initial SOC
Vmax   = 3.6;       % V - Upper voltage cutoff
Vmin   = 2.0;       % V - Lower voltage cutoff
Ceff   = 0.99;      % Coulombic efficiency

%% Thermal parameters
Mass      = 70/1000;           % kg
Cp        = 1.36*1000;         % J/kg/K
T0        = 30;                % °C - Initial cell temperature
Tamb      = 25;                % °C - Ambient temperature
hf        = 5;                 % W/m²/K - Convective heat transfer coefficient
D         = 25.85/1000;        % m  - Cell diameter
H         = 65.15/1000;        % m  - Cell height
SurfaceL  = pi()*D/4*H;        % m² - Lateral surface area
a         = hf*SurfaceL*(-1);  % Thermal coefficient a
b         = hf*SurfaceL*Tamb;  % Thermal coefficient b

%% Load OCV(SOC,T) test data - Charge
data_table_25n_c = readtable("testdata_charge.xlsx", Sheet="T1=-25");
data_table_15n_c = readtable("testdata_charge.xlsx", Sheet="T2=-15");
data_table_5n_c  = readtable("testdata_charge.xlsx", Sheet="T3=-5");
data_table_5p_c  = readtable("testdata_charge.xlsx", Sheet="T4=5");
data_table_15p_c = readtable("testdata_charge.xlsx", Sheet="T5=15");
data_table_25p_c = readtable("testdata_charge.xlsx", Sheet="T6=25");
data_table_35p_c = readtable("testdata_charge.xlsx", Sheet="T7=35");
data_table_45p_c = readtable("testdata_charge.xlsx", Sheet="T8=45");

%% Load OCV(SOC,T) test data - Discharge
data_table_25n_d = readtable("testdata_discharge.xlsx", Sheet="T1=-25");
data_table_15n_d = readtable("testdata_discharge.xlsx", Sheet="T2=-15");
data_table_5n_d  = readtable("testdata_discharge.xlsx", Sheet="T3=-5");
data_table_5p_d  = readtable("testdata_discharge.xlsx", Sheet="T4=5");
data_table_15p_d = readtable("testdata_discharge.xlsx", Sheet="T5=15");
data_table_25p_d = readtable("testdata_discharge.xlsx", Sheet="T6=25");
data_table_35p_d = readtable("testdata_discharge.xlsx", Sheet="T7=35");
data_table_45p_d = readtable("testdata_discharge.xlsx", Sheet="T8=45");

%% Extract SOC and OCV vectors - Charge
SOC_25n_c = data_table_25n_c{1:end,1}; OCV_25n_c = data_table_25n_c{1:end,2};
SOC_15n_c = data_table_15n_c{1:end,1}; OCV_15n_c = data_table_15n_c{1:end,2};
SOC_5n_c  = data_table_5n_c{1:end,1};  OCV_5n_c  = data_table_5n_c{1:end,2};
SOC_5p_c  = data_table_5p_c{1:end,1};  OCV_5p_c  = data_table_5p_c{1:end,2};
SOC_15p_c = data_table_15p_c{1:end,1}; OCV_15p_c = data_table_15p_c{1:end,2};
SOC_25p_c = data_table_25p_c{1:end,1}; OCV_25p_c = data_table_25p_c{1:end,2};
SOC_35p_c = data_table_35p_c{1:end,1}; OCV_35p_c = data_table_35p_c{1:end,2};
SOC_45p_c = data_table_45p_c{1:end,1}; OCV_45p_c = data_table_45p_c{1:end,2};

%% Extract SOC and OCV vectors - Discharge
SOC_25n_d = data_table_25n_d{1:end,1}; OCV_25n_d = data_table_25n_d{1:end,2};
SOC_15n_d = data_table_15n_d{1:end,1}; OCV_15n_d = data_table_15n_d{1:end,2};
SOC_5n_d  = data_table_5n_d{1:end,1};  OCV_5n_d  = data_table_5n_d{1:end,2};
SOC_5p_d  = data_table_5p_d{1:end,1};  OCV_5p_d  = data_table_5p_d{1:end,2};
SOC_15p_d = data_table_15p_d{1:end,1}; OCV_15p_d = data_table_15p_d{1:end,2};
SOC_25p_d = data_table_25p_d{1:end,1}; OCV_25p_d = data_table_25p_d{1:end,2};
SOC_35p_d = data_table_35p_d{1:end,1}; OCV_35p_d = data_table_35p_d{1:end,2};
SOC_45p_d = data_table_45p_d{1:end,1}; OCV_45p_d = data_table_45p_d{1:end,2};

%% Interpolate onto uniform SOC grid
Br_1 = [-25:10:45];   % Temperature breakpoints
Br_2 = [0:0.01:1];   % SOC breakpoints (step = 0.01)

OCV_25n_c_int = interp1(SOC_25n_c, OCV_25n_c, Br_2, 'linear', 'extrap');
OCV_15n_c_int = interp1(SOC_15n_c, OCV_15n_c, Br_2, 'linear', 'extrap');
OCV_5n_c_int  = interp1(SOC_5n_c,  OCV_5n_c,  Br_2, 'linear', 'extrap');
OCV_5p_c_int  = interp1(SOC_5p_c,  OCV_5p_c,  Br_2, 'linear', 'extrap');
OCV_15p_c_int = interp1(SOC_15p_c, OCV_15p_c, Br_2, 'linear', 'extrap');
OCV_25p_c_int = interp1(SOC_25p_c, OCV_25p_c, Br_2, 'linear', 'extrap');
OCV_35p_c_int = interp1(SOC_35p_c, OCV_35p_c, Br_2, 'linear', 'extrap');
OCV_45p_c_int = interp1(SOC_45p_c, OCV_45p_c, Br_2, 'linear', 'extrap');

OCV_25n_d_int = interp1(SOC_25n_d, OCV_25n_d, Br_2, 'linear', 'extrap');
OCV_15n_d_int = interp1(SOC_15n_d, OCV_15n_d, Br_2, 'linear', 'extrap');
OCV_5n_d_int  = interp1(SOC_5n_d,  OCV_5n_d,  Br_2, 'linear', 'extrap');
OCV_5p_d_int  = interp1(SOC_5p_d,  OCV_5p_d,  Br_2, 'linear', 'extrap');
OCV_15p_d_int = interp1(SOC_15p_d, OCV_15p_d, Br_2, 'linear', 'extrap');
OCV_25p_d_int = interp1(SOC_25p_d, OCV_25p_d, Br_2, 'linear', 'extrap');
OCV_35p_d_int = interp1(SOC_35p_d, OCV_35p_d, Br_2, 'linear', 'extrap');
OCV_45p_d_int = interp1(SOC_45p_d, OCV_45p_d, Br_2, 'linear', 'extrap');

%% Build 2D Lookup Tables for Simulink
LUT_charge    = [OCV_25n_c_int; OCV_15n_c_int; OCV_5n_c_int; OCV_5p_c_int; ...
                 OCV_15p_c_int; OCV_25p_c_int; OCV_35p_c_int; OCV_45p_c_int];

LUT_discharge = [OCV_25n_d_int; OCV_15n_d_int; OCV_5n_d_int; OCV_5p_d_int; ...
                 OCV_15p_d_int; OCV_25p_d_int; OCV_35p_d_int; OCV_45p_d_int];
```
