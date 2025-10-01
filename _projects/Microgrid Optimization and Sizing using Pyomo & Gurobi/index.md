---
layout: post
title: Microgrid Optimization and Sizing using Python Pyomo Environment & Gurobi Solver
description: Designed and implemented a comprehensive microgrid optimization model using Pyomo, covering both operational planning and component sizing. The framework minimizes cost while prioritizing PV usage, incorporates EV charging strategies, battery and generator constraints, and extends to optimal sizing of system components to ensure reliable, efficient, and economically viable microgrid performance.
skills: 
  - Optimization modeling (pyomo)
  - Energy systems analysis
  - Data handling
  - Techno-economic evaluation

main-image: 
---

# Objective
To develop an optimization framework for the operation and sizing of a microgrid that integrates photovoltaic (PV) generation, battery storage systems (BSS), electric vehicle (EV) charging, and backup generators. The goal is to minimize total cost while ensuring reliable energy supply, efficient asset utilization, and compliance with operational constraints.

# Methodology
- Implemented two stages of optimization using Python, Pyomo, and the Gurobi solver:
	1.	Operational Planning – optimized dispatch of PV, battery, EV, and generator to minimize daily operating costs.
	2.	System Sizing – optimized capacity of PV, BSS, and generator to minimize lifecycle cost (CAPEX + OPEX) under realistic demand and generation profiles.
- Defined key variables (power import/export, SOC dynamics, charge/discharge rates) and formulated constraints covering:
  1. Power balance
  2. PV generation limits
	3. Battery & EV state-of-charge dynamics
	4. Generator & grid ramping limits (flicker reduction)
	5. EV availability and target SOC requirements
- Objective functions included both economic terms (import/export price, generator cost, degradation cost) and system priorities (PV utilization, BSS prioritization, EV target SOC).

# Assumptions
Perfect foresight of load demand, PV availability, and EV connection schedules.
- Fixed energy prices for grid import, export, and generator fuel.
- Ideal converter efficiency and no network losses (except explicitly modeled round-trip battery and EV charging efficiencies).
- Static degradation cost coefficients for storage assets.
- Simulation horizons: 365 days or 1 year

# Results
- Operational Planning:
	•	Achieved cost minimization through priority dispatch (PV > BSS > grid import > generator).
	•	Reduced grid dependence by incentivizing battery discharging during PV deficit and penalizing EV discharging.
	•	Ensured EVs reached required SOC targets at departure while maintaining grid stability.
- System Sizing:
	•	Identified optimal PV and BSS capacities within defined upper bounds.
	•	Balanced capital expenditure with long-term operational savings, providing insights into cost-effective system design.
	•	Ensured robust operation under varying daily demand and generation scenarios.

## Optimal Sizing Results
<br>

| PV [kWp] | PV Inverter [kW] | Battery [kWh] | Battery Inverter [kW] | Generator [kW] | Annualized CAPEX [EUR] | OPEX [EUR] | Total Cost [EUR] |
|----------|------------------|---------------|-----------------------|----------------|-------------------------|------------|------------------|
| 5.84     | 4.51             | 12.66         | 2.12                  | 0.00           | 718.70                  | 813.77     | 1,566.33         |

# Significance & Impact
- Demonstrates how mathematical optimization can inform both short-term operation and long-term planning of decentralized energy systems.
- Supports the transition to renewable-based microgrids by reducing cost and improving reliability.
- Highlights the importance of integrating EVs not just as loads but as flexible storage assets.
- Provides a replicable framework for techno-economic assessment of microgrid projects.

# Key Takeaway
- Two-level optimization (operation + sizing) provides a holistic strategy for cost minimization in microgrids.
- Battery prioritization is crucial for reducing expensive grid imports and generator reliance.
- EV charging strategies must balance user requirements with system efficiency.
- Optimization frameworks using Pyomo + Gurobi are powerful tools for real-world energy system planning and can be scaled to different contexts (islands, rural electrification, smart grids).

<!--
# Key Figures
## Sipora Island Parameters and Proposed Topology
### Sipora Island Load Profile
![](/_projects/sonos-teardown/loadprofile.png){:height="400px"}
<br>
### Solar Global Horizontal Irradiance & Temperature Data
![](/_projects/sonos-teardown/ghi.png){:height="400px"}
<br>
![](/_projects/sonos-teardown/dailytemp.png){:height="400px"}
<br>
### Original Case Sizing
![](/_projects/sonos-teardown/ocsizing.png){:height="400px"}
<br>
### Proposed Topology
![](/_projects/sonos-teardown/mainvisual.png){:height="400px"}
<br>

## PV+Battery (C1)
### C1 Power System Performance 
![](/_projects/sonos-teardown/c1loadprofile.png){:height="400px"}
<br>
### C1 SOC
![](/_projects/sonos-teardown/c1soc.png){:height="400px"}
<br>

## PV+Battery+P2H2P (C2)
### C2 Power System Performance
![](/_projects/sonos-teardown/c2loadprofile.png){:height="400px"}
<br>
### C2 SOC
![](/_projects/sonos-teardown/c2SOC.png){:height="400px"}
<br>
### C2 FC Performance
![](/_projects/sonos-teardown/c2fcperformance.png){:height="400px"}
<br>

-->


































Microgrid Optimization and Sizing using Pyomo & Gurobi

Objective

To develop an optimization framework for the operation and sizing of a microgrid that integrates photovoltaic (PV) generation, battery storage systems (BSS), electric vehicle (EV) charging, and backup generators. The goal is to minimize total cost while ensuring reliable energy supply, efficient asset utilization, and compliance with operational constraints.

⸻

Methodology
	•	Implemented two stages of optimization using Python, Pyomo, and the Gurobi solver:
	1.	Operational Planning – optimized dispatch of PV, battery, EV, and generator to minimize daily operating costs.
	2.	System Sizing – optimized capacity of PV, BSS, and generator to minimize lifecycle cost (CAPEX + OPEX) under realistic demand and generation profiles.
	•	Defined key variables (power import/export, SOC dynamics, charge/discharge rates) and formulated constraints covering:
	•	Power balance
	•	PV generation limits
	•	Battery & EV state-of-charge dynamics
	•	Generator & grid ramping limits (flicker reduction)
	•	EV availability and target SOC requirements
	•	Objective functions included both economic terms (import/export price, generator cost, degradation cost) and system priorities (PV utilization, BSS prioritization, EV target SOC).

⸻

Assumptions
	•	Perfect foresight of load demand, PV availability, and EV connection schedules.
	•	Fixed energy prices for grid import, export, and generator fuel.
	•	Ideal converter efficiency and no network losses (except explicitly modeled round-trip battery and EV charging efficiencies).
	•	Static degradation cost coefficients for storage assets.
	•	Simulation horizons: 10 days for operation planning, 30 days for sizing.

⸻

Results
	•	Operational Planning:
	•	Achieved cost minimization through priority dispatch (PV > BSS > grid import > generator).
	•	Reduced grid dependence by incentivizing battery discharging during PV deficit and penalizing EV discharging.
	•	Ensured EVs reached required SOC targets at departure while maintaining grid stability.
	•	System Sizing:
	•	Identified optimal PV and BSS capacities within defined upper bounds.
	•	Balanced capital expenditure with long-term operational savings, providing insights into cost-effective system design.
	•	Ensured robust operation under varying daily demand and generation scenarios.

⸻

Significance & Impact
	•	Demonstrates how mathematical optimization can inform both short-term operation and long-term planning of decentralized energy systems.
	•	Supports the transition to renewable-based microgrids by reducing cost and improving reliability.
	•	Highlights the importance of integrating EVs not just as loads but as flexible storage assets.
	•	Provides a replicable framework for techno-economic assessment of microgrid projects.

⸻

Key Takeaways
	•	Two-level optimization (operation + sizing) provides a holistic strategy for cost minimization in microgrids.
	•	Battery prioritization is crucial for reducing expensive grid imports and generator reliance.
	•	EV charging strategies must balance user requirements with system efficiency.
	•	Optimization frameworks using Pyomo + Gurobi are powerful tools for real-world energy system planning and can be scaled to different contexts (islands, rural electrification, smart grids).
