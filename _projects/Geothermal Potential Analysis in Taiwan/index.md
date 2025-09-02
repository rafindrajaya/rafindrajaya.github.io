---
layout: post
title: Geothermal Potential Analysis in Taiwan
description: Developed and modeled Python-based Single Flash and Binary Rankine Cycle Geothermal Power Plants (CoolProp) in the Datun region, Taiwan.
skills: 
- Python
- CoolProp
- Thermodynamics
main-image: /Schematic Single Flash.png
---

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
<br>
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
### Thermodynamic Points Topology of the Single Flash System
{% include image-gallery.html images="/Schematic Single Flash.png" height="400" %}
<br>
### Thermodynamic Points of the Single Flash System
{% include image-gallery.html images="/Schematic Single Flash.png" height="400" %}
