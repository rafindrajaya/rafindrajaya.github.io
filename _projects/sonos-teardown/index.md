---
layout: post
title: Implementation of Renewables-based Power-to-Hydrogen-to-Power System for
Tropical Remote Island Stand-Alone Microgrids
description: Designed and evaluated a renewables-based Power-to-Hydrogen-to-Power (P2H2P) microgrid for Sipora Island, Indonesia, using Homer Pro to compare diesel, PV-battery, and PV+Battery+Hydrogen systems. The study found that the PV+Battery+P2H2P configuration achieved the lowest cost of electricity (0.236 USD/kWh), zero emissions, and 4.5 days of energy autonomy, outperforming the diesel and battery-only systems both economically and environmentally. 
skills: 
  - techno-economic modeling
  - renewable integration
  - hybrid storage optimization
  - Homer Pro

main-image: /mainvisual.png
---

**Institution**: Sophia University – TESLab (Prof. Masafumi Miyatake) <br>
**Role**: Lead Researcher & Author <br>
**Tools**: Homer Pro <br>

# Objective
To design and evaluate a zero-emission power system for Sipora Island (Indonesia) by comparing diesel-based, PV-battery, and PV+Battery+Power-to-Hydrogen-to-Power (P2H2P) configurations. The goal was to identify the most cost-effective, reliable, and sustainable system for remote, diesel-dependent communities.

# Methodology
- Case Study: Sipora Island, ~22,500 residents, 40 MWh/day load, currently supplied by diesel generators.
- Scenarios Analyzed:
1.	Original Case (OC): Existing diesel generators.
2.	Case 1 (C1): Solar PV + Lithium-ion battery.
3.	Case 2 (C2): Solar PV + Battery + P2H2P (PEM electrolyzer, compressed H₂ storage, PEM fuel cell).
- Simulation Tool: Homer Pro for 25-year techno-economic and environmental optimization.
- Metrics: Net Present Cost (NPC), Cost of Electricity (COE), emissions, autonomy, reliability.

# Assumptions
- Diesel price: subsidized (0.67 USD/L) and unsubsidized (1.34 USD/L).
- PV efficiency: 18.7% (25-year lifetime).
- Lithium-ion batteries: 90% round-trip efficiency, 4-hour discharge rate.
- PEM electrolyzer efficiency: 80%.
- H₂ storage: compressed at 30 MPa.
- Project horizon: 25 years, discount rate 6.25%.

# Results
1. **Diesel System (OC)**: COE = 0.242–0.419 USD/kWh, ~10,000 tCO₂/year emissions, highly subsidy-dependent.
2. **PV+Battery (C1)**: Zero emissions, COE = 0.366 USD/kWh, but 70.7% excess energy wasted. Only competitive if diesel subsidies are removed.
3. **PV+Battery+P2H2P (C2)**:
- Best performer: COE = 0.236 USD/kWh (lowest across cases).
- NPC = 57.9M USD (cheaper than diesel and C1).
- Zero emissions, 100% renewable fraction.
- Hybrid storage (Li-ion + H₂) provided 4.5 days autonomy, ensuring energy security during low-sunlight periods.
- Reduced wasted energy from 70.7% (C1) to 45.9%.
- Positioned Sipora Island for potential hydrogen economy applications (domestic, transport, export).

| Properties | Original Case (DG Subsidized) | Original Case (DG Unsubsidized) | Case 1 (PV+LIBs) | Case 2 (PV+LIBs+P2H2P) |
|------------|-------------------------------|---------------------------------|------------------|------------------------|
| COE [USD/kWh] | 0.242 | 0.419 | 0.366 | 0.236 |
| NPC [USD] | 59.5M | 103M | 89.9M | 57.9M |
| CAPEX [USD] | 313,860 | 313,860 | 67.2M | 42.0M |
| OPEX [USD/year] | 863,388 | 887,575 | 617,663 | 579,834 |
| Fuel Cost [USD/year] | 2.6M | 5.16M | 0 | 0 |
| Present worth [USD] | – | – | -30.4M or 13.1M | 1.6M or 45.1M |
| Annual Worth [USD/year] | – | – | -1.8M or 780K | 95,192 or 2.7M |
| Total electric production [kWh/year] | 14,622,630 | 14,622,630 | 55,493,756 | 33,488,798 |
| RE fraction [%] | 0 | 0 | 100 | 100 |
| Unmet load [%] | 0 | 0 | 0.0642 | 0.0566 |
| Excess Electricity [kWh/year] | 0 | 0 | 39,228,299 | 15,370,860 |
| Clipped Energy [%] | 0 | 0 | 70.7 | 45.9 |
| Carbon Dioxide [kg/year] | 10,089,465 | 10,089,465 | 0 | 0 |
| Carbon Monoxide [kg/year] | 53,725 | 53,725 | 0 | 0 |
| Unburned Hydrocarbon [kg/year] | 2,795 | 2,795 | 0 | 0 |
| Particulate Matter [kg/year] | 416 | 416 | 0 | 0 |
| Sulfur Dioxide [kg/year] | 24,881 | 24,881 | 0 | 0 |
| Nitrogen Oxides [kg/year] | 9,191 | 9,191 | 0 | 0 |

# Significance & Impact
- Technical Expertise: Demonstrated advanced skills in energy system modeling, optimization, and techno-economic analysis using Homer Pro and hybrid storage design.
- Strategic Insight: Showed how hydrogen integration can make renewables economically competitive with subsidized diesel in tropical developing regions.
- Scalability: Provides a replicable framework for remote islands worldwide, accelerating energy transition and improving resilience.
- Industry Relevance: Addresses real-world challenges of subsidy dependency, diesel price volatility, and emissions regulation, with actionable pathways for utilities and policymakers.

# Key Takeaway
By integrating PV, batteries, and hydrogen storage, I demonstrated that remote tropical islands can achieve 100% renewable, zero-emission power at lower cost than diesel, paving the way for sustainable microgrids and hydrogen society development.

# Key Figures
### 

