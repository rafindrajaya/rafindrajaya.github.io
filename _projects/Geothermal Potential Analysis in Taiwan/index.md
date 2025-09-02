---
layout: post
title: Geothermal Potential Analysis in Taiwan
description: Developed and modeled Python-based Single Flash and Binary Rankine Cycle Geothermal Power Plants (CoolProp) in the Datun region, Taiwan.
skills: 
- Python
- CoolProp
- Thermodynamics
main-image: /ssf.png
---

[Access Full Report](https://drive.google.com/file/d/1_0MxYZgvV-AR0ko0zGHC9BETNRGnZ1Nb/view)
# Objective
- Assess the impact of geothermal power development in Datun on Taiwan's transition to net-zero by 2050
- Model two types of geothermal plants to estimate net power output and CO2 emissions reduction.

# Methodology 
- Developed and analyzed single-flash and binary Rankine cycle geothermal power plant models using Python (CoolProp) and MS Excel for thermodynamic point-based numerical analysis.
- Used real well data (E-201 and GT-19) from Datun high-enthalpy reservoirs.
- Compared geothermal output against coal-fired generation to evaluate CO₂ offsets.

# Key Assumptions
- Continuous annual operation (~8,000 h/year).
- Coal replacement as baseline for CO₂ reduction.
- Binary cycle fluid: isopentane.
- Grid connection assumed feasible due to proximity to Taipei.

# Results
## Well E-201 (Single-flash, >260 degree celcius):
- Net output: **197,088 MWh/year**
- CO2 avoided is equivalent to **194,000 t/year** (assuming coal displacement)

## Well GT-19 (Binary Rankine with Isopentane):
- Net output: **74.61 GWh/year**
- CO2 avoided is equivalent to **61,180 t/year**

# Insights & Significance
- Datun geothermal resources could replace coal, oil, and gas plants with a stable renewable baseload.
- Even a small number of wells provide substantial CO₂ reductions.
- Taiwan’s volcanic geology makes geothermal a strategic, untapped clean energy source.
- Main barriers: upfront drilling costs, technical risk, environmental safeguards.
- Success depends on investment in advanced geothermal technology and supportive energy policy.

# Figures and Visualization
### Schematic of the Single Flash Geothermal Power Plant System
{% include image-gallery.html images="_projects/Geothermal Potential Analysis in Taiwan/schematicsingleflash.jpeg" height="400"%}
<br>
### Schematic of the Binary Cycle Geothermal Power Plant System
{% include image-gallery.html images="/schematicbinary.jpeg" height="400" %}
<br>
# Python Code Single Flash Geothermal Plant Model
### Thermodynamic Points Modelling
```python
import CoolProp.CoolProp as CP

# Given Data
T1 = 293 + 273.15  # Convert to Kelvin (293°C)  
P3 = 0.05e5  # 0.05 bar (condenser pressure)
eta_turbine = 0.80  # 80% Turbine isentropic efficiency
eta_pump = 0.70  # 70% Pump isentropic efficiency
eta_oel_pump = 0.93 
x1 = 0.29 #Based on research in Taiwan
eta_organic = 0.995
eta_el_ST = 0.985
x4 = 0
P4 = P3
T4 = CP.PropsSI('S', 'P', P4, 'Q', x4, 'Water')
Tsep_opt = (T1 + T4) / 2 #Optimized seperator temperature for max turbine output based on literature
P2 = CP.PropsSI('P', 'T', Tsep_opt, 'Q', 0, 'Water')  # Get corresponding pressure

# --- Point 1: Geothermal Fluid from Reservoir ---
P1 = CP.PropsSI('P', 'T', T1, 'Q', x1,  'Water')  # Pressure and I need to add quality
T1 = T1
hg1 = CP.PropsSI('H', 'T', T1, 'Q', 1, 'Water')  # Enthalpy
hf1 = CP.PropsSI('H', 'T', T1, 'Q', 0, 'Water')  # Enthalpy
sg1 = CP.PropsSI('S', 'T', T1, 'Q', 1, 'Water')  # Entropy
sf1 = CP.PropsSI('S', 'T', T1, 'Q', 0, 'Water')  # Entropy
h1 = x1*(hg1-hf1)+hf1
s1 = x1*(sg1-sf1)+sf1  # Entropy
x1=x1


# --- Enthaly and entropy after seperation process ---
h_f2 = CP.PropsSI('H', 'P', P2, 'Q', 0, 'Water')  # Sat. Liquid Enthalpy at P2
h_g2 = CP.PropsSI('H', 'P', P2, 'Q', 1, 'Water')  # Sat. Vapor Enthalpy at P2
s_f2 = CP.PropsSI('S', 'P', P2, 'Q', 0, 'Water')  # Sat. Liquid Entropy at P2
s_g2 = CP.PropsSI('S', 'P', P2, 'Q', 1, 'Water')  # Sat. Vapor Entropy at P2

# Thermodynamics properties at point 2
h2 = h1  # Isenthalpic process
x2 = (h2 - h_f2) / (h_g2 - h_f2)
Tsep_opt = (T1 + T4) / 2 #Optimized seperator temperature for max turbine output
P2 = CP.PropsSI('P', 'T', Tsep_opt, 'Q', x2, 'Water')  # Get corresponding pressure
T2 = CP.PropsSI('T', 'H', h2, 'P', P2, 'Water')  # Temperature at point 2
s2 = s_f2 + x2 * (s_g2 - s_f2)
print(f'T2={T2}, Tsep={Tsep_opt}')

# --- Point 2L (Liquid separated in the separator) ---
x2L = 0
P2L = P2
T2L = CP.PropsSI('T', 'P', P2L, 'Q', x2L, 'Water')  # Temperature at 2L
h2L = CP.PropsSI('H', 'T', T2L, 'Q', x2L, 'Water') 
s2L = CP.PropsSI('S', 'T', T2L, 'Q', x2L, 'Water') 


# --- Point 2V (Vapor separated in the separator) ---
P2V = P2
x2V = 1
T2V = CP.PropsSI('T', 'P', P2V, 'Q', x2V, 'Water')  # Temperature at 2L
h2V = CP.PropsSI('H', 'T', T2V, 'Q', x2V, 'Water')
s2V = CP.PropsSI('S', 'T', T2V, 'Q', x2V, 'Water')

# --- Point 3s: Ideal expansion in Turbine ---
s3s = s2V  # Isentropic entropy at turbine exit
P3s = P3
hf3 = CP.PropsSI('H', 'P', P3, 'Q', 0, 'Water')
hg3 = CP.PropsSI('H', 'P', P3, 'Q', 1, 'Water')
sf3 = CP.PropsSI('S', 'P', P3, 'Q', 0, 'Water')
sg3 = CP.PropsSI('S', 'P', P3, 'Q', 1, 'Water')
x3s = (s3s - sf3) / (sg3 - sf3) #ideal quality
T3s = CP.PropsSI('T', 'P', P3, 'S', s3s, 'Water')
h3s = hf3+x3s*(hg3-hf3)  # Isentropic enthalpy
h3s_cp = CP.PropsSI('H', 'P', P3, 'S', s3s, 'Water')
x3s_cp = CP.PropsSI('Q', 'H', h3s, 'P', P3s, 'Water')  
print(f'point 3 w/ cp h3s={h3s_cp}, x3s={x3s_cp}, wo/ cp h3s={h3s}, x3s={x3s}')

# Point 3: Real enthalpy at turbine exit (considering efficiency)
P3 = P3
h3 = h2V - eta_turbine * (h2V - h3s)
x3 = (h3 - hf3) / (hg3 - hf3) #real quality
s3 = sf3 + x3 * (sg3 - sf3) #to calculate the entropy by x
s3_cp = CP.PropsSI('S', 'H', h3, 'P', P3, 'Water')  # Real entropy at P3
T3= CP.PropsSI('T', 'P', P3, 'Q', x3, 'Water')
h3_check = CP.PropsSI('H', 'S', s3, 'P', P3, 'Water')  # Direct entropy lookup
print(f"Calculated h3: {h3:.2f}, CoolProp h3: {h3_check:.2f}")

# --- Point 4: Condenser Exit (Saturated Liquid) ---
x4 = 0
P4 = P3
T4 = CP.PropsSI('T', 'P', P4, 'Q', x4, 'Water')
h4 = CP.PropsSI('H', 'P', P4, 'Q', x4, 'Water')  
s4 = CP.PropsSI('S', 'T', T4, 'Q', x4, 'Water') 



# --- Point 5: Pump (Isentropic) ---
s5s = s4  # Isentropic compression
x5s = 0
P5 = 1e5
T5s = CP.PropsSI('T', 'P', P5, 'S', s5s, 'Water')
h5s = CP.PropsSI('H', 'P', P5, 'T', T5s, 'Water')


# --- Point 5: Pump (real) Real enthalpy at pump exit (considering efficiency)
x5 = 0
h5 = h4 + ((h5s - h4) / eta_pump)
T5 = CP.PropsSI('T', 'P', P5, 'H', h5, 'Water')  # Real temperature
s5 = CP.PropsSI('S', 'P', P5, 'T', T5, 'Water')  # Real entropy at P2

# --- Mass Flow Analysis ---
m_total = 100  # Assumption [kg/s]
m_vapor = x2 * m_total  # Vapor mass flow
m_liquid = (1 - x2) * m_total  # Liquid mass flow
m1 = 100
m2 = m1
m2v = m_vapor
m2l = m_liquid
m3 = m_vapor
m4 = m_vapor
m5 = m_vapor
m6 = m_vapor + m_liquid

# --- Point 6: Re-injection Pump Inlet(to Reservoir) ---
P6 = 1e5 #1 atm
x6 = 0
h6 = (m5*h5+m2l*h2L)/m6
s6 = CP.PropsSI('S', 'H', h6, 'P', P6, 'Water')  
T6 = CP.PropsSI('T', 'H', h6, 'P', P6, 'Water')  


# --- Point 7: Re-injection Pump Outlet (Isentropic) ---
P7 = 10e5
x7 = 0
s7s = s6
T7s = CP.PropsSI('T', 'P', P7, 'S', s7s, 'Water') 
h7s = CP.PropsSI('H', 'P', P7, 'T', T7s, 'Water')

# --- Point 7: Re-injection Pump Outlet (Real) ---
h7 = h6 + ((h7s-h6)/eta_pump)
T7 = CP.PropsSI('T', 'P', P7, 'H', h7, 'Water')
s7 = CP.PropsSI('S', 'P', P7, 'H', h7, 'Water')

# --- Power and Efficiency Calculations ---
W_turbine = m2v * (h2V - h3)  # Turbine work
W_pump1 = (m4 * (h5 - h4)) # Pump 1 work
W_pump2 = (m6 * (h7 - h6)) # Pump 2 work

# Net Power Output
W_net = W_turbine - (W_pump1 + W_pump2)

# Heat Input (from geothermal well)
Q_in = m_total * (h1 - h7)

# Efficiency of the Cycle
eta_cycle = (W_net / Q_in) * 100  # Convert to %

# Energy balance verification
Q_out = m3 * (h3 - h4)  # Heat rejected at condenser
print(f"h3: {h3:.2f}, h4: {h4:.2f}, Δh: {h3 - h4:.2f}")
h4_check = CP.PropsSI('H', 'P', P3, 'Q', 0, 'Water')
print(f"Calculated h4: {h4:.2f}, CoolProp h4: {h4_check:.2f}")
energy_balance = Q_in - (W_net + Q_out)

from tabulate import tabulate

# Create a table for thermodynamic properties
data = [
    ["1", P1, T1, h1, s1, m_total, x1],
    ["2", P2, T2, h2, s2, m2, x2],
    ["2L", P2L, T2L, h2L, s2L, m2l, x2L],
    ["2V", P2V, T2V, h2V, s2V, m2v, x2V],
    ["3s", P3s, T3s, h3s, s3s, m3, x3s],
    ["3", P3, T3, h3, s3, m3, x3],
    ["4", P4, T4, h4, s4, m4, x4],
    ["5s", P5, T5s, h5s, s5s, m5, x5s],
    ["5", P5, T5, h5, s5, m5, x5],
    ["6", P6, T6, h6, s6, m6, x6],
    ["7s", P7, T7s, h7s, s7s, m6, x7],
    ["7", P7, T7, h7, s7, m6, x7],
]

headers = ["State", "P (Pa)", "T (K)", "h (J/kg)", "s (J/kg-K)", "m (kg/s)", "x"]
print("\n--- Thermodynamic Properties Table ---")
print(tabulate(data, headers=headers, tablefmt="plain", floatfmt=".2f", numalign="right", stralign="center"))

# Create a table for power and efficiency calculations
power_data = [
    ["Turbine Power Output", W_turbine],
    ["Pump Power (Pump 1 + Pump 2)", W_pump1 + W_pump2],
    ["Net Power Output", W_net],
    ["Cycle Efficiency (%)", eta_cycle],
]
print("\n--- Power and Efficiency Calculations ---")
print(tabulate(power_data, headers=["Description", "Value (W)"], tablefmt="plain", floatfmt=".2f"))

# Create a table for energy balance check
energy_data = [
    ["Heat Input (Q_in)", Q_in],
    ["Heat Rejection (Q_out)", Q_out],
    ["Net Power Output (W_net)", W_net],
    ["Energy Balance Error", energy_balance],
    ["Error (%)", (energy_balance/Q_in) * 100],
]
print("\n--- Energy Balance Check ---")
print(tabulate(energy_data, headers=["Description", "Value (W)"], tablefmt="plain", floatfmt=".2f"))

# Check energy balance tolerance
tolerance = 0.01 * Q_in  # 1% of input heat
if abs(energy_balance) < tolerance:
    print("\n✅ Energy balance is within acceptable tolerance.")
else:
    print("\n⚠️ Energy balance error is significant. Check calculations.")


# import matplotlib.pyplot as plt

# # Given parameters
# m_parametric = [50, 100, 150, 200, 250]
# W_net_parametric = []  # List to store net power output

# # Assuming h2V, h3, h4, h5, h6, h7, and x2 are defined
# for m in m_parametric:
#     m_total = m  # Assumption [kg/s]
#     m_vapor = x2 * m_total  # Vapor mass flow
#     m_liquid = (1 - x2) * m_total  # Liquid mass flow

#     m1 = m
#     m2 = m1
#     m2v = m_vapor
#     m2l = m_liquid
#     m3 = m_vapor
#     m4 = m_vapor
#     m5 = m_vapor
#     m6 = m_vapor + m_liquid

#     # --- Power and Efficiency Calculations ---
#     W_turbine = m2v * (h2V - h3)  # Turbine work
#     W_pump1 = m4 * (h5 - h4)  # Pump 1 work
#     W_pump2 = m6 * (h7 - h6)  # Pump 2 work

#     # Net Power Output
#     W_net = (W_turbine - (W_pump1 + W_pump2))/10**6
#     W_net_parametric.append(W_net)  # Store W_net for plotting

# # Plot results
# plt.figure(figsize=(8, 5))
# plt.plot(m_parametric, W_net_parametric, marker='o', linestyle='-')
# plt.xlabel('Mass Flow Rate (kg/s)')
# plt.ylabel('Net Power Output (MW)')
# plt.title('Net Power Output vs. Mass Flow Rate')
# plt.grid(True)
# plt.show()
'''
```
### Creating the T-S Diagram
```python
import CoolProp.CoolProp as CP

# Given Data
T1 = 293 + 273.15  # Convert to Kelvin (293°C)  
P3 = 0.05e5  # 0.05 bar (condenser pressure)
eta_turbine = 0.80  # 80% Turbine isentropic efficiency
eta_pump = 0.70  # 70% Pump isentropic efficiency
eta_oel_pump = 0.93 
x1 = 0.3 #Based on research in Taiwan
eta_organic = 0.995
eta_el_ST = 0.985
x4 = 0
P4 = P3
T4 = CP.PropsSI('S', 'P', P4, 'Q', x4, 'Water')
Tsep_opt = (T1 + T4) / 2 #Optimized seperator temperature for max turbine output based on literature
P2 = CP.PropsSI('P', 'T', Tsep_opt, 'Q', 0, 'Water')  # Get corresponding pressure

# --- Point 1: Geothermal Fluid from Reservoir ---
P1 = CP.PropsSI('P', 'T', T1, 'Q', x1,  'Water')  # Pressure and I need to add quality
T1 = T1
hg1 = CP.PropsSI('H', 'T', T1, 'Q', 1, 'Water')  # Enthalpy
hf1 = CP.PropsSI('H', 'T', T1, 'Q', 0, 'Water')  # Enthalpy
sg1 = CP.PropsSI('S', 'T', T1, 'Q', 1, 'Water')  # Entropy
sf1 = CP.PropsSI('S', 'T', T1, 'Q', 0, 'Water')  # Entropy
h1 = x1*(hg1-hf1)+hf1
s1 = x1*(sg1-sf1)+sf1  # Entropy
x1=x1


# --- Enthaly and entropy after seperation process ---
h_f2 = CP.PropsSI('H', 'P', P2, 'Q', 0, 'Water')  # Sat. Liquid Enthalpy at P2
h_g2 = CP.PropsSI('H', 'P', P2, 'Q', 1, 'Water')  # Sat. Vapor Enthalpy at P2
s_f2 = CP.PropsSI('S', 'P', P2, 'Q', 0, 'Water')  # Sat. Liquid Entropy at P2
s_g2 = CP.PropsSI('S', 'P', P2, 'Q', 1, 'Water')  # Sat. Vapor Entropy at P2

# Thermodynamics properties at point 2
h2 = h1  # Isenthalpic process
x2 = (h2 - h_f2) / (h_g2 - h_f2)
Tsep_opt = (T1 + T4) / 2 #Optimized seperator temperature for max turbine output
P2 = CP.PropsSI('P', 'T', Tsep_opt, 'Q', x2, 'Water')  # Get corresponding pressure
T2 = CP.PropsSI('T', 'H', h2, 'P', P2, 'Water')  # Temperature at point 2
s2 = s_f2 + x2 * (s_g2 - s_f2)
print(f'T2={T2}, Tsep={Tsep_opt}')

# --- Point 2L (Liquid separated in the separator) ---
x2L = 0
P2L = P2
T2L = CP.PropsSI('T', 'P', P2L, 'Q', x2L, 'Water')  # Temperature at 2L
h2L = CP.PropsSI('H', 'T', T2L, 'Q', x2L, 'Water') 
s2L = CP.PropsSI('S', 'T', T2L, 'Q', x2L, 'Water') 


# --- Point 2V (Vapor separated in the separator) ---
P2V = P2
x2V = 1
T2V = CP.PropsSI('T', 'P', P2V, 'Q', x2V, 'Water')  # Temperature at 2L
h2V = CP.PropsSI('H', 'T', T2V, 'Q', x2V, 'Water')
s2V = CP.PropsSI('S', 'T', T2V, 'Q', x2V, 'Water')

# --- Point 3s: Ideal expansion in Turbine ---
s3s = s2V  # Isentropic entropy at turbine exit
P3s = P3
hf3 = CP.PropsSI('H', 'P', P3, 'Q', 0, 'Water')
hg3 = CP.PropsSI('H', 'P', P3, 'Q', 1, 'Water')
sf3 = CP.PropsSI('S', 'P', P3, 'Q', 0, 'Water')
sg3 = CP.PropsSI('S', 'P', P3, 'Q', 1, 'Water')
x3s = (s3s - sf3) / (sg3 - sf3) #ideal quality
T3s = CP.PropsSI('T', 'P', P3, 'S', s3s, 'Water')
h3s = hf3+x3s*(hg3-hf3)  # Isentropic enthalpy
h3s_cp = CP.PropsSI('H', 'P', P3, 'S', s3s, 'Water')
x3s_cp = CP.PropsSI('Q', 'H', h3s, 'P', P3s, 'Water')  
print(f'point 3 w/ cp h3s={h3s_cp}, x3s={x3s_cp}, wo/ cp h3s={h3s}, x3s={x3s}')

# Point 3: Real enthalpy at turbine exit (considering efficiency)
P3 = P3
h3 = h2V - eta_turbine * (h2V - h3s)
x3 = (h3 - hf3) / (hg3 - hf3) #real quality
s3 = sf3 + x3 * (sg3 - sf3) #to calculate the entropy by x
s3_cp = CP.PropsSI('S', 'H', h3, 'P', P3, 'Water')  # Real entropy at P3
T3= CP.PropsSI('T', 'P', P3, 'Q', x3, 'Water')
h3_check = CP.PropsSI('H', 'S', s3, 'P', P3, 'Water')  # Direct entropy lookup
print(f"Calculated h3: {h3:.2f}, CoolProp h3: {h3_check:.2f}")

# --- Point 4: Condenser Exit (Saturated Liquid) ---
x4 = 0
P4 = P3
T4 = CP.PropsSI('T', 'P', P4, 'Q', x4, 'Water')
h4 = CP.PropsSI('H', 'P', P4, 'Q', x4, 'Water')  
s4 = CP.PropsSI('S', 'T', T4, 'Q', x4, 'Water') 



# --- Point 5: Pump (Isentropic) ---
s5s = s4  # Isentropic compression
x5s = 0
P5 = 1e5
T5s = CP.PropsSI('T', 'P', P5, 'S', s5s, 'Water')
h5s = CP.PropsSI('H', 'P', P5, 'T', T5s, 'Water')


# --- Point 5: Pump (real) Real enthalpy at pump exit (considering efficiency)
x5 = 0
h5 = h4 + ((h5s - h4) / eta_pump)
T5 = CP.PropsSI('T', 'P', P5, 'H', h5, 'Water')  # Real temperature
s5 = CP.PropsSI('S', 'P', P5, 'T', T5, 'Water')  # Real entropy at P2

# --- Mass Flow Analysis ---
m_total = 100  # Assumption [kg/s]
m_vapor = x2 * m_total  # Vapor mass flow
m_liquid = (1 - x2) * m_total  # Liquid mass flow
m1 = 100
m2 = m1
m2v = m_vapor
m2l = m_liquid
m3 = m_vapor
m4 = m_vapor
m5 = m_vapor
m6 = m_vapor + m_liquid

# --- Point 6: Re-injection Pump Inlet(to Reservoir) ---
P6 = 1e5 #1 atm
x6 = 0
h6 = (m5*h5+m2l*h2L)/m6
s6 = CP.PropsSI('S', 'H', h6, 'P', P6, 'Water')  
T6 = CP.PropsSI('T', 'H', h6, 'P', P6, 'Water')  


# --- Point 7: Re-injection Pump Outlet (Isentropic) ---
P7 = 10e5
x7 = 0
s7s = s6
T7s = CP.PropsSI('T', 'P', P7, 'S', s7s, 'Water') 
h7s = CP.PropsSI('H', 'P', P7, 'T', T7s, 'Water')

# --- Point 7: Re-injection Pump Outlet (Real) ---
h7 = h6 + ((h7s-h6)/eta_pump)
T7 = CP.PropsSI('T', 'P', P7, 'H', h7, 'Water')
s7 = CP.PropsSI('S', 'P', P7, 'T', T7, 'Water')

import CoolProp.CoolProp as CP
import numpy as np
import matplotlib.pyplot as plt

# Define valid temperature range for the saturation curve
T_sat = np.linspace(273.15, 647.096, 100)  # Limited to the critical point of water

# Get entropy values for saturated liquid and vapor
s_f = [CP.PropsSI('S', 'T', T, 'Q', 0, 'Water') for T in T_sat]
s_g = [CP.PropsSI('S', 'T', T, 'Q', 1, 'Water') for T in T_sat]

# Handle superheated region (beyond 647.096 K)
T_superheated = np.linspace(647.096, 800, 50)  # Extend beyond the critical point
s_superheated = [CP.PropsSI('S', 'T', T, 'P', CP.PropsSI('Pcrit', 'Water'), 'Water') for T in T_superheated]

# Plot saturation curve
plt.plot(s_f, T_sat, 'k', label="Saturation Curve")
plt.plot(s_g, T_sat, 'k')

# Plot superheated extension
plt.plot(s_superheated, T_superheated, 'k--', label="Superheated Region")

# Define process points (excluding 6 and 7)
points = {
    "1": (s1, T1),
    "2": (s2, T2),
    "2L": (s2L, T2L),
    "2V": (s2V, T2V),
    "3s": (s3s, T3s),
    "3": (s3, T3),
    "4": (s4, T4), 
    "5s": (s5s, T5s),
    "5": (s5, T5),
}

# Offsets for overlapping points
offsets = {
    "4": (-40, -40),   # Shift label 4 slightly down-right
    "5": (10, 10),  # Shift label 5 slightly down-left
    "5s": (-300, 0),   # Shift label 5s upwards
}

# Plot each point with adjusted labels
for label, (s, T) in points.items():
    plt.plot(s, T, 'ko')  # Black dots for points
    dx, dy = offsets.get(label, (0, 0))  # Get offset, default is (0,0)
    plt.text(s + dx, T + dy, label, fontsize=12, ha='center', va='bottom')

# Process paths
plt.plot([s1, s2], [T1, T2], 'b--')  # Expansion process
plt.plot([s2L, s2V], [T2L, T2V], 'b--')  # Separation process
plt.plot([s2V, s3], [T2V, T3], 'b-')  # Turbine expansion
plt.plot([s3, s4], [T3, T4], 'b-')  # Condensation
plt.plot([s4, s5], [T4, T5], 'b-')  # Pump

# Labels and display
plt.xlabel("Entropy (J/kg·K)")
plt.ylabel("Temperature (K)")
plt.legend()
plt.grid()
plt.title("T-s Diagram of the Single Flash Geothermal Power Cycle")
plt.show()
```

