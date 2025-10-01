---
layout: post
title: Microgrid Optimization and Sizing using Python Pyomo Environment & Gurobi Solver
description: Designed and implemented a comprehensive microgrid optimization model using Pyomo, covering both operational planning and component sizing. The framework minimizes cost while prioritizing PV usage, incorporates EV charging strategies, battery and generator constraints, and extends to optimal sizing of system components to ensure reliable, efficient, and economically viable microgrid performance.
skills: 
  - Optimization modeling (pyomo)
  - Energy systems analysis
  - Data handling
  - Techno-economic evaluation

main-image: /Flexgen-EV-charging.png
---

# Objective
To develop an optimization framework for the operation and sizing of a microgrid that integrates photovoltaic (PV) generation, battery storage systems (BSS), electric vehicle (EV) charging, and backup generators. The goal is to minimize total cost while ensuring reliable energy supply, efficient asset utilization, and compliance with operational constraints.

# Methodology
- Implemented two stages of optimization using Python, Pyomo, and the Gurobi solver:
1. Operational Planning – optimized dispatch of PV, battery, EV, and generator to minimize daily operating costs.
2. System Sizing – optimized capacity of PV, BSS, and generator to minimize lifecycle cost (CAPEX + OPEX) under realistic demand and generation profiles.
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


# Key Figures
## Power System Performance
### January (Winter)
![](/_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.26.46.png){:height="400px"}
<br>
- Moderate PV availability leads to partial reliance on battery discharge and some grid import. The objective function prioritizes battery usage, resulting in reduced operational costs.
<br>
### June (Summer)
![](/_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.28.34.png){:height="400px"}
<br>
- High solar irradiance enables complete self-sufficiency with zero imports and significant grid export. The battery and EV are charged efficiently, achieving the lowest operational cost
<br>
### October (Autumn)
![](_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.29.08.png){:height="400px"}
<br>
- Transitional weather results in lower PV output than summer, causing the system to rely on both battery discharge and grid import. The optimizer balances cost and availability.
<br>


































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
