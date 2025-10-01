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
1. Achieved cost minimization through priority dispatch (PV > BSS > grid import > generator).
2. Reduced grid dependence by incentivizing battery discharging during PV deficit and penalizing EV discharging.
3. Ensured EVs reached required SOC targets at departure while maintaining grid stability.
- System Sizing:
1. Identified optimal PV and BSS capacities within defined upper bounds.
2. Balanced capital expenditure with long-term operational savings, providing insights into cost-effective system design.
3. Ensured robust operation under varying daily demand and generation scenarios.

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



# Key Figures: Power System Performance
## January (Winter)
![](/_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.26.46.png){:height="400px"}

<br>

- Moderate PV availability leads to partial reliance on battery discharge and some grid import. The objective function prioritizes battery usage, resulting in reduced operational costs.

<br>

## June (Summer)
![](/_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.28.34.png){:height="400px"}

<br>

- High solar irradiance enables complete self-sufficiency with zero imports and significant grid export. The battery and EV are charged efficiently, achieving the lowest operational cost
<br>

## October (Autumn)
![](/_projects/Microgrid Optimization and Sizing using Pyomo & Gurobi
/Screenshot 2025-10-01 at 10.29.08.png){:height="400px"}

<br>

- Transitional weather results in lower PV output than summer, causing the system to rely on both battery discharge and grid import. The optimizer balances cost and availability.
<br>
# Python Code
## Operational Planning Python Code
```python
from param import delta_t
from param import SOC_i_bss, SOC_min_bss, SOC_max_bss, eff_bss
from param import C_ev, P_nom_ev, eff_ev, SOC_min_ev, SOC_max_ev, SOC_target_ev
from param import PI_gen, PI_imp, PI_exp
from pyomo.environ import ConcreteModel, Param, Var, Objective, Constraint, NonNegativeReals, minimize
from datetime import datetime
import utils as utils

C_pv = 70                            # PV system size [kWp]
C_bss = 40                           # Battery capacity [kWh]	
P_nom_bss = 10                       # Battery inverter nominal power [kW]
P_nom_pv = 10                        # PV inverter nominal power [kW]
P_max_gen = 10                       # Maximum generator power [kW]

    # Access all parameters present in the res object.
    # These exists for each t in model.t:
    #  - res.P_load[t] = Load power at time t
    #  - res.P_pv_max[t] = Max available PV power at time t [kW/kWp]
    #  - res.EV_connected[t] = EV connected at time t


    # These exists for each c in model.connections: ####WHAT IS THIS FOR?
    #  - res.t_arr[c] = Time at which the EV is connected for the c-th connection
    #  - res.t_dep[c] = Time at which the EV is disconnected for the c-th connection
    #  - res.SOC_i_ev[c] = Initial SOC of the EV connected at time t_arr[c]



def create_model(res):
    # Create a concrete model
    model = ConcreteModel()
    
    model.periods = range(res.t_s)
    model.connections = range(len(res.t_arr)) # A set is created with length equal to the number of EV connections

    # Operation planning asset sizes
    model.P_nom_pv = Param(initialize=P_nom_pv)                            # Nominal power for PV inverter [kW]
    model.C_bss = Param(initialize=C_bss)                                  # Battery capacity [kWh]
    model.C_pv = Param(initialize=C_pv)                                    # PV system size [kWp]
    model.P_nom_bss = Param(initialize=P_nom_bss)                          # Battery inverter nominal power [kW]
    model.P_nom_ev = Param(initialize=P_nom_ev)                            # EV charger nominal power [kW]
    model.C_ev = Param(initialize=C_ev)                                    # EV capacity [kWh]
    model.P_max_gen = Param(initialize=P_max_gen)                          # Maximum generator power [kW]

    # Variables
    model.P_imp = Var(model.periods, within=NonNegativeReals)              # Imported power [kW]
    model.P_exp = Var(model.periods, within=NonNegativeReals)              # Exported power [kW]
    model.P_pv = Var(model.periods, within=NonNegativeReals)               # PV power output  [kW]
    model.P_gen = Var(model.periods, within=NonNegativeReals)              # Generator power output [kW]
    model.P_charge_bss = Var(model.periods, within=NonNegativeReals)       # Battery charging power [kW]
    model.P_discharge_bss = Var(model.periods, within=NonNegativeReals)    # Battery discharging power [kW]
    model.P_charge_ev = Var(model.periods, within=NonNegativeReals)        # EV charging power [kW]
    model.P_discharge_ev = Var(model.periods, within=NonNegativeReals)     # EV discharging power [kW] 

    model.SOC_bss = Var(model.periods, within=NonNegativeReals)            # Bss state of charge [kWh]
    model.SOC_ev = Var(model.periods, within=NonNegativeReals)             # EV state of charge [kWh]
    

    # Define the objective function ----------------------------------------------------------------------------
    model.objective = Objective(
    sense=minimize,
    expr=sum(
        PI_gen * model.P_gen[t]+
        PI_imp * model.P_imp[t]* (1 + 5 * (model.SOC_bss[t] - SOC_min_bss * C_bss) / (C_bss))+ #this makes sure bss is used first before import
        0.1 * (model.P_charge_bss[t])+
        0.1 * (model.P_discharge_bss[t])- #battery degradation cost
        0.1 * model.P_pv[t]+ #prioritizes using PV
        3*model.P_discharge_ev[t]- #penalizes using EV 
        model.P_charge_ev[t]- #prioritizes charging EV
        PI_exp * model.P_exp[t]- #gives reward to export energy in excess condition
        0.01*model.P_discharge_bss[t] #prioritizes discharge battery when PV is insufficient


        # (model.P_charge_bss[t] - model.P_discharge_bss[t])
        # 0.02 * (model.P_charge_ev[t] - )than
        
        
        for t in model.periods)
    )

    #Constraints ---------------------------------------------------------------------------------------------------------------------------
    # Power balance constraint:
    # Total Power In = Total Power Out
    # (PV + Gen + Grid Import + Discharges) = (Load + Grid Export + Charges)

    model.P_bal_cstr = Constraint(model.periods, expr=lambda model, t:
    model.P_pv[t]
    + model.P_gen[t]
    + model.P_imp[t]
    + model.P_discharge_bss[t]
    + model.P_discharge_ev[t]
    ==
    res.P_load[t]
    + model.P_exp[t]
    + model.P_charge_bss[t]
    + model.P_charge_ev[t])

    # Battery Energy constraints:
    model.bss_SOC_i = Constraint(model.periods, expr=lambda model, t: model.SOC_bss[0]==SOC_i_bss*C_bss)
    model.bss_SOC_min = Constraint(model.periods, expr=lambda model, t: 
        SOC_min_bss*C_bss <= model.SOC_bss[t] + (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss) * delta_t 
    )
    model.bss_SOC_max = Constraint(model.periods, expr=lambda model, t: 
        model.SOC_bss[t] + (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss) * delta_t <= SOC_max_bss*C_bss
    )
    #Battery SOC update
    model.bss_SOC_dyn = Constraint(model.periods[:-1], expr=lambda model, t:
    model.SOC_bss[t+1] == model.SOC_bss[t] +
        delta_t * (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss)
    )

    # PV power constraints:
    model.pv_limit = Constraint(model.periods, expr=lambda model, t: model.P_pv[t] <= res.P_pv_max[t]*C_pv)
    #Add nominal PV limit
    model.pv_inv_limit = Constraint(model.periods, expr=lambda model, t:
        model.P_pv[t] <= model.P_nom_pv)

    # Battery charging and discharging power constraints:
    model.bss_ch_limit = Constraint(model.periods, expr=lambda model, t: model.P_charge_bss[t] <= P_nom_bss)
    model.bss_disch_limit = Constraint(model.periods, expr=lambda model, t: model.P_discharge_bss[t] <= P_nom_bss)

    # EV charging and discharging power constraints:
    model.ev_charge_limit = Constraint(model.periods, expr=lambda model, t: model.P_charge_ev[t] <= res.EV_connected[t] * P_nom_ev)
    model.ev_discharge_limit = Constraint(model.periods, expr=lambda model, t: model.P_discharge_ev[t] <= res.EV_connected[t] * P_nom_ev)

    # Prevent simultaneous charging and discharging for BSS
    model.bss_charge_discharge_exclusion = Constraint(model.periods, rule=lambda model, t:model.P_charge_bss[t] + model.P_discharge_bss[t] <= P_nom_bss)

    # Prevent simultaneous charging and discharging for EV
    model.ev_charge_discharge_exclusion = Constraint(model.periods, rule=lambda model, t:model.P_charge_ev[t] + model.P_discharge_ev[t] <= P_nom_ev * res.EV_connected[t])

    # EV target SOC constraint:
    model.ev_SOC_min = Constraint(model.periods, expr=lambda model, t: model.SOC_ev[t] >= SOC_min_ev * C_ev)
    model.ev_SOC_max = Constraint(model.periods, expr=lambda model, t: model.SOC_ev[t] <= SOC_max_ev * C_ev)

    #SOC update for EV during EV connection
    def ev_soc_update_rule(model, t):
        if res.EV_connected[t] == 1:
            return model.SOC_ev[t+1] == model.SOC_ev[t] + delta_t * (
                model.P_charge_ev[t] * eff_ev - model.P_discharge_ev[t] / eff_ev
            )
        else:
            return Constraint.Skip

    model.ev_SOC_dyn = Constraint(model.periods[:-1], rule=ev_soc_update_rule)

    # This will be ensured for all connections c. You should access variables etc with model.variable_name[t_dep[c]]
    #EV SOC Initialization
    model.EV_init = Constraint(model.connections, rule=lambda model, c:model.SOC_ev[res.t_arr[c]] == res.SOC_ev_i[c] )

    #EV reaching target 
    model.EV_target = Constraint(model.connections, rule=lambda model, c:model.SOC_ev[res.t_dep[c]]==SOC_target_ev * C_ev)
  
    # Generator power and Import constraints :
    model.gen_constraint = Constraint(model.periods, expr=lambda model, t: model.P_gen[t]<=P_max_gen)

    #Flickering Constraints
    model.ramp_up_gen = Constraint(model.periods[1:], rule=lambda model, t:model.P_gen[t] - model.P_gen[t-1] <= 4)
    model.ramp_down_gen = Constraint(model.periods[1:], rule=lambda model, t:model.P_gen[t-1] - model.P_gen[t] <= 4)
    model.ramp_up_grid = Constraint(model.periods[1:], rule=lambda model, t:model.P_imp[t] - model.P_imp[t-1] <= 4)
    model.ramp_down_grid = Constraint(model.periods[1:], rule=lambda model, t:model.P_imp[t-1] - model.P_imp[t] <= 4)

    
    return model

def run(model, results):
    model, results = utils.solve_model(model, results)
    if model and results:
        utils.check_res(results)
        utils.print_res(results)
        #utils.plot_res(results)
        #utils.plot_res2(results) # /!\ Do not use this plotting function to create plots for your report. It is only for interactive visualisation purposes.
    return results

start_time = datetime(2021, 6, 1, 0, 0, 0)                                  # Start time of the simulation [YYYY, MM, DD, HH, MM, SS]
n_days = 10                                                             # Number of days to simulate
results = utils.Results(start_time, n_days, yearly_kwh=10000, yearly_km=10000)      # Initialize results object with start time and number of days, yearly consumption and km driven
model = create_model(results)
run(model, results)

```
<br>

## Optimal Sizing Python Code

```python
from param import delta_t, inv_hor, C_pv_max, C_bss_max
from param import SOC_i_bss, SOC_min_bss, SOC_max_bss, eff_bss
from param import C_ev, P_nom_ev, eff_ev, SOC_min_ev, SOC_max_ev, SOC_target_ev
from param import PI_gen, PI_imp, PI_exp, PI_c_inv, PI_c_bss, PI_c_pv, PI_c_gen
from pyomo.environ import ConcreteModel, Param, Var, Objective, Constraint, NonNegativeReals, minimize
from datetime import datetime
import utils
import numpy as np

def create_model(res):
    # Create a concrete model
    model = ConcreteModel()
    
    model.periods = range(res.t_s)                          # A set is created with length equal to the number of time steps in the simulation
    model.connections = range(len(res.t_arr))               # A set is created with length equal to the number of EV connections

    # Access all parameters present in the res object.
    # These exists for each t in model.t:
    #  - res.P_load[t] = Load power at time t
    #  - res.P_pv_max[t] = Max available PV power at time t
    #  - res.EV_connected[t] = EV connected at time t

    # These exists for each c in model.connections:
    #  - res.t_arr[c] = Time at which the EV is connected for the c-th connection
    #  - res.t_dep[c] = Time at which the EV is disconnected for the c-th connection
    #  - res.SOC_i_ev[c] = Initial SOC of the EV connected at time t_arr[c]
    
    # Asset sizes
    model.P_nom_pv = Var(within=NonNegativeReals)                           # Nominal power for PV inverter [kW]
    model.C_bss = Var(within=[0,C_bss_max])                                 # Battery capacity [kWh]
    model.C_pv = Var(within=[0,C_pv_max])                                   # PV system size [kWp]
    model.P_nom_bss = Var(within=NonNegativeReals)                          # Battery inverter nominal power [kW]
    model.P_max_gen = Var(within=NonNegativeReals)                          # Maximum generator power [kW]
    model.P_nom_ev = Param(initialize=P_nom_ev)                             # EV charger nominal power [kW]
    model.C_ev = Param(initialize=C_ev)                                     # EV capacity [kWh]

    # Variables
    model.P_imp = Var(model.periods, within=NonNegativeReals)               # Imported power
    model.P_exp = Var(model.periods, within=NonNegativeReals)               # Exported power 
    model.P_pv = Var(model.periods, within=NonNegativeReals)                # PV power output 
    model.P_gen = Var(model.periods, within=NonNegativeReals)               # Generator power output 
    model.P_charge_bss = Var(model.periods, within=NonNegativeReals)        # Battery charging power 
    model.P_discharge_bss = Var(model.periods, within=NonNegativeReals)     # Battery discharging power 
    model.P_charge_ev = Var(model.periods, within=NonNegativeReals)         # EV charging power 
    model.P_discharge_ev = Var(model.periods, within=NonNegativeReals)      # EV discharging power 

    # Energy storage variables for battery and EV
    model.SOC_bss = Var(model.periods, within=NonNegativeReals)             # Bss state of charge [kWh]
    model.SOC_ev = Var(model.periods, within=NonNegativeReals)              # EV state of charge [kWh]

    # Objective function: CAPEX + OPEX
    model.objective = Objective(sense=minimize, expr=
        (PI_c_pv * model.C_pv +
        PI_c_inv*model.P_nom_pv+
        PI_c_bss * model.C_bss +
        PI_c_inv * model.P_nom_bss +
        PI_c_gen * model.P_max_gen)*n_days/(inv_hor*365) + 

        
        delta_t * sum( 
        PI_imp * model.P_imp[t] +
        PI_gen * model.P_gen[t] -
        PI_exp * model.P_exp[t] 
        # model.P_pv[t]
            for t in model.periods
        )
    )


    #Investment Constraints


    # Power balance
    model.P_bal_cstr = Constraint(model.periods, expr=lambda model, t:
        model.P_pv[t] + model.P_gen[t] + model.P_imp[t] + model.P_discharge_bss[t] + model.P_discharge_ev[t] ==
        res.P_load[t] + model.P_exp[t] + model.P_charge_bss[t] + model.P_charge_ev[t])

    # Battery constraints
    model.bss_SOC_i = Constraint(expr=model.SOC_bss[0] == SOC_i_bss * model.C_bss)
    model.bss_SOC_min = Constraint(model.periods, expr=lambda model, t:
        model.SOC_bss[t] + (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss) * delta_t >= SOC_min_bss * model.C_bss)
    model.bss_SOC_max = Constraint(model.periods, expr=lambda model, t:
        model.SOC_bss[t] + (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss) * delta_t <= SOC_max_bss * model.C_bss)
    model.bss_SOC_dyn = Constraint(model.periods[:-1], expr=lambda model, t:
        model.SOC_bss[t+1] == model.SOC_bss[t] + delta_t * (model.P_charge_bss[t] * eff_bss - model.P_discharge_bss[t] / eff_bss))

    # PV constraint
    model.pv_limit = Constraint(model.periods, expr=lambda model, t:
        model.P_pv[t] <= res.P_pv_max[t] * model.C_pv)
    #Add nominal PV limit
    model.pv_inv_limit = Constraint(model.periods, expr=lambda model, t:
        model.P_pv[t] <= model.P_nom_pv)

    # Battery and EV energy limits
    model.bss_ch_limit = Constraint(model.periods, expr=lambda model, t: model.P_charge_bss[t] <= model.P_nom_bss)
    model.bss_disch_limit = Constraint(model.periods, expr=lambda model, t: model.P_discharge_bss[t] <= model.P_nom_bss)
    model.ev_charge_limit = Constraint(model.periods, expr=lambda model, t: model.P_charge_ev[t] <= res.EV_connected[t] * model.P_nom_ev)
    model.ev_discharge_limit = Constraint(model.periods, expr=lambda model, t: model.P_discharge_ev[t] <= res.EV_connected[t] * model.P_nom_ev)

    # No simultaneous charge/discharge
    model.bss_charge_discharge_exclusion = Constraint(model.periods, rule=lambda model, t:
        model.P_charge_bss[t] + model.P_discharge_bss[t] <= model.P_nom_bss)
    model.ev_charge_discharge_exclusion = Constraint(model.periods, rule=lambda model, t:
        model.P_charge_ev[t] + model.P_discharge_ev[t] <= model.P_nom_ev * res.EV_connected[t])

    # EV SoC constraints
    model.ev_SOC_min = Constraint(model.periods, expr=lambda model, t: model.SOC_ev[t] >= SOC_min_ev * model.C_ev)
    model.ev_SOC_max = Constraint(model.periods, expr=lambda model, t: model.SOC_ev[t] <= SOC_max_ev * model.C_ev)
    def ev_soc_update_rule(model, t):
        if res.EV_connected[t] == 1:
            return model.SOC_ev[t+1] == model.SOC_ev[t] + delta_t * (model.P_charge_ev[t] * eff_ev - model.P_discharge_ev[t] / eff_ev)
        else:
            return Constraint.Skip
    model.ev_SOC_dyn = Constraint(model.periods[:-1], rule=ev_soc_update_rule)
    model.EV_init = Constraint(model.connections, rule=lambda model, c:
        model.SOC_ev[res.t_arr[c]] == res.SOC_ev_i[c])
    model.EV_target = Constraint(model.connections, rule=lambda model, c:
        model.SOC_ev[res.t_dep[c]] == SOC_target_ev * model.C_ev)

    # Generator & grid constraints
    model.gen_constraint = Constraint(model.periods, expr=lambda model, t: model.P_gen[t] <= model.P_max_gen)
    model.ramp_up_gen = Constraint(model.periods[1:], rule=lambda model, t:
        model.P_gen[t] - model.P_gen[t-1] <= 4)
    model.ramp_down_gen = Constraint(model.periods[1:], rule=lambda model, t:
        model.P_gen[t-1] - model.P_gen[t] <= 4)
    model.ramp_up_grid = Constraint(model.periods[1:], rule=lambda model, t:
        model.P_imp[t] - model.P_imp[t-1] <= 4)
    model.ramp_down_grid = Constraint(model.periods[1:], rule=lambda model, t:
        model.P_imp[t-1] - model.P_imp[t] <= 4)

    return model

def run(model, results):
    model, results = utils.solve_model(model, results)
    if model and results:
        utils.check_res(results)
        #utils.print_res(results)
        #utils.plot_res(results)
        utils.print_sizing_results(results)
        utils.plot_res2(results)
    return results

start_time = datetime(2021, 10, 5, 0, 0, 0)                                  # Start time of the simulation [YYYY, MM, DD, HH, MM, SS]
n_days = 2                                                             # Number of days to simulate
results = utils.Results(start_time, n_days, yearly_kwh=7000, yearly_km=13000)      # Initialize results object with start time and number of days, yearly consumption and km driven
model = create_model(results)
run(model, results)
```

<br>

## Parameters

```python
import pandas as pd
from scipy.interpolate import interp1d
import numpy as np
import matplotlib.pyplot as plt

# Simulation Parameters
delta_t = 1 / 4                                    # Duration of each time step [hour]
inv_hor = 20                                       # Investment horizon [year]

# =============================================================================
#  Costs
# ==============  Operation  ==================================================
PI_gen = 0.6                          # Fuel costs for generator [EUR/kWh]
PI_imp = 0.2  #0.4 to 0.2                        # Cost of importing energy [EUR/kWh]
PI_exp = 0.03 #0 to 1 by 20% increments 0, 0.03, ..., 1                          # Revenue from exporting energy [EUR/kWh]
# ==============  Investment  =================================================
PI_c_pv = 1800                        # Cost of PV capacity [EUR/kWp]
PI_c_bss = 280                        # Cost of battery capacity [EUR/kWh]
PI_c_inv = 150                        # Cost of inverter capacity [EUR/kW]
PI_c_gen = 150                        # Cost of generator capacity [EUR/kW]
# =============================================================================
C_pv_max = 50                      # Maximum PV system size [kWp]
C_bss_max = 100                    # Maximum Battery capacity [kWh]

# Battery Parameters
SOC_i_bss = 0.5                 # Initial SOC for battery as a fraction of C_bss [0, 1]
SOC_min_bss = 0.2               # Minimum SOC for battery as a fraction of C_bss [0, 1]
SOC_max_bss = 0.85              # Maximum SOC for battery as a fraction of C_bss [0, 1]
eff_bss = 0.95                  # Efficiency for battery charging process [0, 1]

# EV Parameters
SOC_target_ev = 0.8             # Target SOC for the EV [0, 1]
C_ev = 60                       # Total storage capacity of the EV battery [kWh]
P_nom_ev = 10                   # Nominal power of the EV charger [kW]
eff_ev = 0.98                   # Efficiency for EV dis.charging process [0, 1]
SOC_min_ev = 0.2                # Minimum SOC for the EV battery [0, 1]
SOC_max_ev = 0.95               # Maximum SOC for the EV battery [0, 1]
```

<br>

## Utils Python Code

```python
import matplotlib.pyplot as plt
from pyomo.environ import SolverFactory, SolverStatus
from datetime import timedelta
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from param import *
from plotly.subplots import make_subplots

def check_res(res):
    print("\n=== Constraint Check ===")

    for t in range(res.t_s):
        # Split charging/discharging
        P_ch_bss = max(res.P_bss[t], 0)
        P_dis_bss = -min(res.P_bss[t], 0)
        P_ch_ev = max(res.P_ev[t], 0)
        P_dis_ev = -min(res.P_ev[t], 0)

        # 🔧 Fixed Power balance: IN = OUT
        power_in = res.P_pv[t] + res.P_gen[t] + res.P_imp[t] + P_dis_bss + P_dis_ev
        power_out = res.P_load[t] + res.P_exp[t] + P_ch_bss + P_ch_ev

        if abs(power_in - power_out) > 0.01:
            print(f"[Time {t}] ⚠️ Power balance mismatch: IN = {power_in:.2f} kW, OUT = {power_out:.2f} kW")

        # ⚠️ Battery simultaneous charge & discharge
        if P_ch_bss > 0 and P_dis_bss > 0:
            print(f"[Time {t}] ❌ Battery charging and discharging simultaneously: Charge = {P_ch_bss:.2f}, Discharge = {P_dis_bss:.2f}")

        # ⚠️ EV simultaneous charge & discharge
        if P_ch_ev > 0 and P_dis_ev > 0:
            print(f"[Time {t}] ❌ EV charging and discharging simultaneously: Charge = {P_ch_ev:.2f}, Discharge = {P_dis_ev:.2f}")

        # ✅ Battery SoC range check
        soc_bss_pct = res.SOC_bss[t] / res.C_bss
        if not SOC_min_bss <= soc_bss_pct <= SOC_max_bss:
            print(f"[Time {t}] ⚠️ Battery SoC out of bounds: {soc_bss_pct:.2%}")

        # ✅ EV SoC range check
        soc_ev_pct = res.SOC_ev[t] / res.C_ev
        if not SOC_min_ev <= soc_ev_pct <= SOC_max_ev:
            print(f"[Time {t}] ⚠️ EV SoC out of bounds: {soc_ev_pct:.2%}")

def print_res(res):
    print("\n=== Optimization Summary ===")
    print(f"Objective value: {res.objective:.2f} EUR")

    # Charge/discharge decomposition
    P_ch_bss = np.maximum(res.P_bss, 0)
    P_dis_bss = -np.minimum(res.P_bss, 0)
    P_ch_ev = np.maximum(res.P_ev, 0)
    P_dis_ev = -np.minimum(res.P_ev, 0)

    # Power balance totals
    power_generated = (np.sum(res.P_pv) + np.sum(res.P_gen) + np.sum(res.P_imp)
                       + np.sum(P_dis_bss) + np.sum(P_dis_ev)) * delta_t
    power_used = (np.sum(res.P_load) + np.sum(res.P_exp)
                  + np.sum(P_ch_bss) + np.sum(P_ch_ev)) * delta_t

    print(f"Power Generated: {power_generated:.2f} kWh")
    print(f"Power Used:      {power_used:.2f} kWh")

    print("\n=== Generation ===")
    print(f"Total PV generation:      {np.sum(res.P_pv) * delta_t:.2f} kWh")
    print(f"Total generator output:   {np.sum(res.P_gen) * delta_t:.2f} kWh")
    print(f"Total grid import:        {np.sum(res.P_imp) * delta_t:.2f} kWh")
    print(f"Battery discharge:        {np.sum(P_dis_bss) * delta_t:.2f} kWh")
    print(f"EV discharge:             {np.sum(P_dis_ev) * delta_t:.2f} kWh")
    

    print("\n=== Consumption ===")
    print(f"Load:                     {np.sum(res.P_load) * delta_t:.2f} kWh")
    print(f"Total grid export:        {np.sum(res.P_exp) * delta_t:.2f} kWh")
    print(f"Battery charge:           {np.sum(P_ch_bss) * delta_t:.2f} kWh")
    print(f"EV charge:                {np.sum(P_ch_ev) * delta_t:.2f} kWh")


    print("\n=== State of Charge ===")
    print(f"Final Battery SOC:        {(res.SOC_bss[-1] / res.C_bss):.2f}")
    print(f"Final EV SOC:             {(res.SOC_ev[-1] / res.C_ev):.2f}")

def plot_res(res):
    import matplotlib.dates as mdates

    fig, axs = plt.subplots(2, 1, figsize=(14, 8), sharex=True)

    # First subplot: Power flows
    axs[0].plot(res.datetime, -res.P_load, label="Load", color="black", linewidth=1.2)
    axs[0].plot(res.datetime, res.P_pv, label="PV", color="orange")
    axs[0].plot(res.datetime, res.P_gen, label="Generator", color="red")
    axs[0].plot(res.datetime, res.P_imp, label="Imports", color="blue")
    axs[0].plot(res.datetime, -res.P_exp, label="Exports", color="cyan")
    axs[0].plot(res.datetime, -np.maximum(res.P_bss, 0), label="BSS Charge", linestyle='--', color="green")
    axs[0].plot(res.datetime, -np.minimum(res.P_bss, 0), label="BSS Discharge", linestyle='--', color="purple")
    axs[0].plot(res.datetime, -np.maximum(res.P_ev, 0), label="EV Charge", linestyle='--', color="yellow")
    axs[0].plot(res.datetime, -np.minimum(res.P_ev, 0), label="EV Discharge", linestyle='--', color="darkblue")
    axs[0].set_ylabel("Power [kW]")
    axs[0].set_title("Power Flows Over Time")
    axs[0].legend(loc="upper right")
    axs[0].grid(True)

    # Second subplot: SoC
    axs[1].plot(res.datetime, res.SOC_bss / res.C_bss * 100, label="Battery SOC [%]", color="blue")
    axs[1].plot(res.datetime, res.SOC_ev / res.C_ev * 100, label="EV SOC [%]", color="orange")
    axs[1].set_ylabel("State of Charge [%]")
    axs[1].set_xlabel("Time")
    axs[1].set_title("SoC Over Time")
    axs[1].legend(loc="upper right")
    axs[1].grid(True)

    # Add EV arrival (green) and departure (red) vertical lines
    for t in res.t_arr:
        for ax in axs:
            ax.axvline(x=res.datetime[t], color='green', linestyle='--', linewidth=0.8, label='EV Arrival' if t == res.t_arr[0] else "")
    for t in res.t_dep:
        for ax in axs:
            ax.axvline(x=res.datetime[t], color='red', linestyle='--', linewidth=0.8, label='EV Departure' if t == res.t_dep[0] else "")

    # Avoid duplicate legend entries
    handles, labels = axs[0].get_legend_handles_labels()
    unique = dict(zip(labels, handles))
    axs[0].legend(unique.values(), unique.keys(), loc="upper right")

    # X-axis formatting
    axs[1].xaxis.set_major_locator(mdates.AutoDateLocator())
    axs[1].xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d %H:%M'))
    fig.autofmt_xdate(rotation=45)

    plt.tight_layout()
    plt.show()
    plt.close()

def print_sizing_results(res):
    print("\n=== Objective Result and Parameter ===")

    # Objective value
    print(f"Total Objective value                    :      {res.objective:.2f} EUR") #CAPEX+OPEX
    print(f"Number of days (Simulation period)       :      {res.n_days} days")

    # Daily factor for CAPEX
    ann_factor = res.n_days / (inv_hor * 365)

    # CAPEX breakdown (Cost per n days)
    capex_pv = PI_c_pv * res.C_pv * ann_factor
    capex_pv_inverter = PI_c_inv * res.P_nom_pv * ann_factor
    capex_bss = PI_c_bss * res.C_bss * ann_factor
    capex_inv = PI_c_inv * res.P_nom_bss * ann_factor
    capex_gen = PI_c_gen * res.P_max_gen * ann_factor
    total_capex = capex_pv + capex_bss + capex_inv + capex_gen

    # Sizing values
    print("\n--- Sizing Results ---")
    print(f"PV size:                    {res.C_pv:.2f} kWp")
    print(f"PV inverter size:           {res.P_nom_pv:.2f} kW")
    print(f"Battery capacity:           {res.C_bss:.2f} kWh")
    print(f"Battery inverter size:      {res.P_nom_bss:.2f} kW")
    print(f"Generator size:             {res.P_max_gen:.2f} kW")

    # CAPEX summary
    print("\n--- CAPEX over simulation period---")
    print(f"PV CAPEX:                   {capex_pv:.2f} EUR")
    print(f"PV Inverter CAPEX:          {capex_pv_inverter:.2f} EUR")
    print(f"Battery CAPEX:              {capex_bss:.2f} EUR")
    print(f"Inverter CAPEX:             {capex_inv:.2f} EUR")
    print(f"Generator CAPEX:            {capex_gen:.2f} EUR")
    print(f"Total Annualized CAPEX:     {total_capex:.2f} EUR")

    # OPEX breakdown
    energy_imported = np.sum(res.P_imp) * delta_t
    energy_generated = np.sum(res.P_gen) * delta_t
    energy_exported = np.sum(res.P_exp) * delta_t

    cost_import = PI_imp * energy_imported
    cost_gen = PI_gen * energy_generated
    reward_export = PI_exp * energy_exported

    total_opex = cost_import + cost_gen - reward_export

    print("\n--- OPEX (Operating Cost over simulation period) ---")
    print(f"Grid Import Cost:           {cost_import:.2f} EUR")
    print(f"Generator Fuel Cost:        {cost_gen:.2f} EUR")
    print(f"Export Revenue:            -{reward_export:.2f} EUR")
    print(f"Total OPEX:                 {total_opex:.2f} EUR")

def solve_model(m, res):
    # Solve the optimization problem
    solver = SolverFactory('gurobi')
    output = solver.solve(m)#, tee=True)  # Parameter 'tee=True' prints the solver output

    # Print elapsed time
    status = output.solver.status

    # Check the solution status
    if status == SolverStatus.ok:
        print("Simulation completed")
        res = save_results(res, m)
        res.save_sizing_results(m)
        return m, res
    elif status == SolverStatus.warning:
        print("Solver finished with a warning.")
    elif status == SolverStatus.error:
        print("Solver encountered an error and did not converge.")
    elif status == SolverStatus.aborted:
        print("Solver was aborted before completing the optimization.")
    else:
        print("Solver status unknown.")
    return None, None

class Results:
    def __init__(self, start_time, n_days, yearly_kwh, yearly_km):
        self.start_time = start_time 
        self.t_s = int(n_days*24/delta_t)                # Total number of discrete time steps in the simulation
        self.n_days = n_days
        self.yearly_kwh = yearly_kwh
        self.yearly_km = yearly_km
        self.t = np.arange(0,self.t_s)

        self.P_pv = np.zeros(self.t_s)
        self.P_bss = np.zeros(self.t_s)
        self.P_ev = np.zeros(self.t_s)
        self.P_gen = np.zeros(self.t_s)
        self.P_imp = np.zeros(self.t_s)
        self.P_exp = np.zeros(self.t_s)

        self.SOC_ev = np.zeros(self.t_s)
        self.SOC_bss = np.zeros(self.t_s)

        # Initialize SOCs
        self.SOC_bss_i = 0.5          

        # Load data from CSV files into pandas DataFrames
        self.df = pd.read_csv('DENSYS/DENSYS/Semester 2/Liege Part 2 Real-time Optimization of a Microgrid/Densys_OPSizing/HW2.csv', delimiter=';', index_col="DateTime", parse_dates=True, date_format='%Y-%m-%d %H:%M:%S')#, date_parser=lambda x: datetime.strptime(x, '%Y-%m-%d %H:%M:%S'))
        self.P_load = np.array([self.df.loc[self.start_time + timedelta(hours=t*delta_t)]["Load"].clip(min=0) * self.yearly_kwh for t in self.t])
        self.P_pv_max = np.array([self.df.loc[self.start_time + timedelta(hours=t*delta_t)]["PV"].clip(min=0) for t in self.t])
        self.EV_connected = np.array([self.df.loc[self.start_time + timedelta(hours=t*delta_t)]["EV"] for t in self.t])
        self.datetime = [self.start_time + timedelta(hours=t*delta_t) for t in self.t]


        self.SOC_ev_i = [SOC_target_ev*C_ev - (self.EV_connected[t]*self.yearly_km) / (5e6) for t in range(self.t_s) if self.EV_connected[t] > 0 and (t == 0 or self.EV_connected[t-1] == 0)]
        self.t_arr = [t for t in range(self.t_s) if self.EV_connected[t] > 0 and (t == 0 or self.EV_connected[t-1] == 0)]
        self.t_dep = [t for t in range(self.t_s) if self.EV_connected[t] == 0 and (t > 0 and self.EV_connected[t-1] > 0)] + ([self.t_s-1] if self.EV_connected[-1] > 0 else [])
        self.EV_connected = [1 if self.EV_connected[t] > 0 else 0 for t in range(self.t_s)]

    def save_sizing_results(self, m):
        self.C_bss = m.C_bss.value
        self.P_nom_bss = m.P_nom_bss.value
        self.C_pv = m.C_pv.value
        self.P_nom_pv = m.P_nom_pv.value
        self.C_ev = m.C_ev.value
        self.P_nom_ev = m.P_nom_ev.value
        self.P_max_gen = m.P_max_gen.value

def save_results(res, m):
    res.P_imp = np.array([m.P_imp[t].value for t in m.periods])
    res.P_exp = np.array([m.P_exp[t].value for t in m.periods])
    res.P_pv = np.array([m.P_pv[t].value for t in m.periods])
    res.P_bss = np.array([m.P_charge_bss[t].value - m.P_discharge_bss[t].value for t in m.periods])
    res.P_ev = np.array([m.P_charge_ev[t].value - m.P_discharge_ev[t].value for t in m.periods])
    res.P_gen = np.array([m.P_gen[t].value for t in m.periods])
    res.SOC_ev = np.array([m.SOC_ev[t].value for t in m.periods])
    res.SOC_bss = np.array([m.SOC_bss[t].value for t in m.periods])
    res.objective = m.objective()
    return res


def plot_res2(res, vert_lines=True):
    # Create an interactive plotly figure
    fig = make_subplots(
        rows=2, cols=1,
        shared_xaxes=True,
        vertical_spacing=0.1,
        subplot_titles=("Power Values Over Time", "State of Charge Over Time"),
        specs=[[{"type": "scatter"}], [{"type": "scatter"}]]
    )
    # Reconstruct P_charge and P_discharge
    P_dis_bss = -np.minimum(0, res.P_bss)
    P_ch_bss = -np.maximum(0, res.P_bss)
    P_dis_ev = -np.minimum(0, res.P_ev)
    P_ch_ev = -np.maximum(0, res.P_ev)

    # Add traces for positive power values (generation)
    fig.add_trace(go.Scatter(x=res.datetime, y=res.P_imp, mode='lines', name='Imports', stackgroup='positive'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=res.P_pv, mode='lines', name='PV', stackgroup='positive'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=res.P_gen, mode='lines', name='Generator', stackgroup='positive'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=P_dis_bss, mode='lines', name='BSS (Discharge)', stackgroup='positive'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=P_dis_ev, mode='lines', name='EV (Discharge)', stackgroup='positive'), row=1, col=1)

    # Add traces for negative power values (consumption)
    fig.add_trace(go.Scatter(x=res.datetime, y=-res.P_load, mode='lines', name='Load', stackgroup='negative'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=P_ch_bss, mode='lines', name='BSS (Charge)', stackgroup='negative'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=P_ch_ev, mode='lines', name='EV (Charge)', stackgroup='negative'), row=1, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=-res.P_exp, mode='lines', name='Exports', stackgroup='negative'), row=1, col=1)

    # Second subplot for SOC values
    fig.add_trace(go.Scatter(x=res.datetime, y=res.SOC_ev / res.C_ev * 100, mode='lines', name='EV SOC [%]'), row=2, col=1)
    fig.add_trace(go.Scatter(x=res.datetime, y=res.SOC_bss / res.C_bss * 100, mode='lines', name='Battery SOC [%]'), row=2, col=1)

    if vert_lines: # Add vertical lines for EV arrival and departure times
        for t in res.t_arr:
            fig.add_vline(x=res.datetime[t], line=dict(color="green", width=1, dash="dash"), row=2, col=1)
        for t in res.t_dep:
            fig.add_vline(x=res.datetime[t], line=dict(color="red", width=1, dash="dash"), row=2, col=1)

    # Update layout for the figure
    fig.update_layout(
        title="Simulation Results",
        xaxis_title="Time",
        yaxis_title="Power [kW]",
        yaxis2_title="State of Charge [%]",
        legend_title="Legend",
        template="plotly_white",
        hovermode="x unified"
    )

    # Show the interactive plot
    fig.show()
    return

```


