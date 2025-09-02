---
layout: post
title: Optimal Design of a Hybrid PV-Battery-Diesel System for Reliable Energy Supply on Sipora Island
description:  This MATLAB project implements a hybrid energy system optimizer that sizes solar PV panels, battery storage, and diesel backup generators to meet a given load profile at minimum cost. The code integrates time-series simulation, realistic dispatch logic, and genetic algorithm optimization, producing both numerical and graphical outputs.

skills: 
  - MATLAB Programming
  - Modular Code Design
  - Data Handling
  - Data Visualization
  - Energy System Modelling
  - Genetic Algorithm
  - Techno-economic Analysis
  - Life-cycle Analysis

main-image: /diesel-pv-bat-topology Large.jpeg
---

---
# Logic of the Code 
## Input Processing
-	Load, irradiance, and temperature are read from CSV.
-	Converted into consistent units (W, Wh, °C).

## System Parameters
-	Define PV, battery, diesel, and cost parameters.
-	Establish GA decision vector: [Npv, Ncell, Ndiesel, SOC_ini].
  
## Optimization Setup
-	Objective = CAPEX + aggregated O&M + lifetime fuel cost + penalty for unmet load.
-	Constraints = SOC bounds, no excessive unmet demand, diesel not exceeding nameplate.
  
## Dispatch Simulation (per timestep)
-	Surplus case: PV → charge battery (respecting SOC & efficiency) → curtail excess.
-	Deficit case: Battery discharges → if still short, diesel supplies → if still short, unmet load recorded.
  
## Genetic Algorithm
-	GA searches for the cost-optimal mix of PV panels, battery cells, and diesel gensets.
-	Integer constraints ensure physical realism (whole panels, whole gensets).
  
## Post-Optimization Simulation
-	Runs the system with the optimal configuration.
-	Generates time-series of SOC, power flows, diesel use, and unmet load.
  
## Visualization
-	Multi-panel plots show:
-	Load vs PV vs Diesel.
-	Battery charge/discharge cycles and curtailment.
-	SOC trajectory.

# Key Assumptions
## Load data: 
Simplified one year of demand, temperature, and irradiance (288 timesteps = 12 months × 24 hours). One month is represented by load profile of one day of that particular month for simplicity.

## PV system: 
- Standard test condition parameters, irradiance and temperature corrections, conversion efficiency losses.
-	Battery: Defined by capacity per cell, SOC operating window (20%–90%).
-	Charging/discharging efficiencies split for realism.
-	Simple cycle aging approximation.
  
## Diesel generators:
1.	500 kW per unit.
2.	Includes efficiency, fuel cost per liter, and O&M.
   
## Economic assumptions:
1.	CAPEX per kW PV, per kWh battery, per genset.
2.	O&M cost fractions over 25-year lifetime.
3.	Fuel costs accumulated over system lifetime.
4.	Penalty: Unmet load is penalized in the objective (instead of strictly forbidden) to balance cost vs. reliability.
 
# Results & Insights 
## The optimizer identified a cost-optimal hybrid system with the following configuration:
-	**6,270 PV panels** supplying the bulk of annual energy demand.
-	**1 battery cell (minimal storage)**, indicating that in this scenario additional storage was not cost-effective compared to diesel backup.
-	**5 diesel units (≈2.5 MW total capacity)** serving as reliability support during deficits.
-	Initial SOC set at 24%, close to the lower operational limit.
-	Objective cost ≈ **$13.8 million (simplified lifetime CAPEX + O&M + fuel)**.

## System performance:
-	Annual demand coverage: **>99.9%**.
-	Unmet load: 270 MWh, only 0.05% of annual consumption—demonstrating that the optimizer balanced cost and reliability effectively.
- Role of components:
1. PV provided the lowest-cost energy.
2. Diesel units were favored over large battery banks for backup due to lower cost per unit of reliability in this setup.
3. Minimal battery sizing shows that storage value depends strongly on relative costs, penalty weighting for unmet load, and system constraints.

## Insights:
-	The optimizer tends to minimize expensive storage investment if diesel fuel and O&M costs remain competitive relative to battery CAPEX.
-	A very low unmet load percentage confirms the system can operate with high reliability while keeping costs down.
-	The results highlight the trade-offs between renewables, storage, and fossil backup, a central challenge in designing hybrid microgrids.

<!--
## Embedding images
### External images
{% include image-gallery.html images="https://live.staticflickr.com/..." height="400" %}
-->

<!--
## Embedding images 
### External images
{% include image-gallery.html images="https://live.staticflickr.com/65535/52821641477_d397e56bc4_k.jpg, https://live.staticflickr.com/65535/52822650673_f074b20d90_k.jpg" height="400"%}
<span style="font-size: 10px">"Starship Test Flight Mission" from https://www.flickr.com/photos/spacex/52821641477/</span>  
You can put in multiple entries. All images will be at a fixed height in the same row. With smaller window, they will switch to columns.  
-->

## Visualization of Results
<br>
### Topology of the system
{% include image-gallery.html images="/diesel-pv-bat-topology Large.jpeg" height="400"%}
<br>
### Load Profile of The Simulated System
{% include image-gallery.html images="/loadprofileoptimal.jpg" height="400"%}
<br> 
### System Performance
{% include image-gallery.html images="/resultgraphoptimal.jpg" height="400"%}

# MATLAB Code

```MATLAB
% Hybrid_Solar_Battery_Diesel_Optimizer.m
% Simplified, documented and corrected implementation of the PV - Battery - Diesel
% hybrid system optimization. This file contains a runnable top-level script and
% the helper functions used by the optimizer and simulator.
%
% Key features / changes from original:
% - Dispatch logic corrected: PV -> Battery -> Diesel (battery dispatched before
%   diesel when there is a deficit; battery charged before curtailment when
%   there is a surplus).
% - PV output guarded against negative temperature correction.
% - Efficiencies applied consistently when converting between DC and AC sides.
% - Diesel capacity consistency: 500 kW per genset (500e3 W).
% - Objective and constraint functions simplified and corrected for units.
% - Code simplified to improve readability and maintainability; documentation
%   included inline so you can understand each block.
%
% Usage:
% 1) Put a CSV named 'Year Data.csv' in the same folder with at least 288 rows
%    and columns such that:
%      - column 5: Temperature (degC)
%      - column 6: Irradiance (W/m^2)
%      - column 7: Load (kW)
% 2) Run the script. It will run a GA to search for 4 integer/continuous
%    decision variables: [Npv, Ncell, Ndiesel, SOC_ini_percent]
% 3) Results (optimal system sizing) and diagnostic plots will be shown.
%
% Notes about units and variables used consistently across the file:
% - Power is expressed in W (watts)
% - Energy is expressed in Wh (watt-hours)
% - Load read from CSV is expected in kW; it is converted to W immediately
% - Battery capacity per 'cell' is in Wh
% - Diesel genset capacity is 500 kW per unit (500e3 W)
% - deltaT is in hours
%
% -------------------------------------------------------------------------
clear; close all; clc;

%% --------------------- 1. INPUT DATA -----------------------------------
% Read a representative year (288 rows = 12 months x 24 hours) or any
% time-series of length n. If you want to move to 8760 hourly data later,
% keep structure the same and use longer vectors.
if ~isfile('Year Data.csv')
    error("File 'Year Data.csv' not found. Please place it in the working folder.");
end
B = readmatrix('Year Data.csv');
Temp_data = B(1:288,5);   % degC
Pirr_data = B(1:288,6);   % W/m^2
P_load_data = B(1:288,7); % kW

% Convert load to W for internal calculations
P_load = P_load_data(:)' * 1000; % row vector, W
Pirr = Pirr_data(:)';
Tenv = Temp_data(:)';

n = length(P_load);            % number of timesteps
deltaT = 1;                    % hours per timestep

time = 1:n;
months = {"Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"};
xticks_idx = 1:24:n;

%% --------------------- 2. SYSTEM PARAMETERS ----------------------------
% PV parameters
P_PV_STC = 665;    % W per panel
G_STC = 1000;      % W/m^2 (reference)
CT = -0.34e-2;     % temperature coeff (1/degC)
T_STC = 25;        % degC
eta_PV = 0.95;     % system-level PV DC efficiency (incl. mismatch)

% Battery parameters
C_cell = 4.22e6;   % Wh per cell (as you had)
eta_bat = 0.90;     % round-trip efficiency factor used split as needed
eta_charge = sqrt(eta_bat);   % approximate charge eff
eta_discharge = sqrt(eta_bat);% approximate discharge eff
SOC_min = 0.20;    % fraction
SOC_max = 0.90;    % fraction

% Grid-conversion efficiencies
eta_DCDC = 0.98;   % PV DC->bus (or converter)
eta_ACDC = 0.95;   % AC->DC (inverter or rectifier)

% Diesel parameters
Diesel_unit_capacity_W = 500e3; % 500 kW per genset [W]
Cinv_diesel_per_unit = 78 * 500; % $ per 500 kW (your original)
Diesel_efficiency = 0.45;        % electrical efficiency
Fuel_cost_per_liter = 0.67;      % $/liter
Energy_content_kWh_per_liter = 10; % kWh per liter
% fuel cost in $/Wh = $/liter / (kWh/liter * 1000 Wh/kWh)
Fuel_cost_per_Wh = Fuel_cost_per_liter / (Energy_content_kWh_per_liter * 1000);

% Investment costs
Cinv_bat_per_kWh = 500;    % $/kWh
Cinv_PV_per_kW = 876;      % $/kW

% O&M (annual fraction of CAPEX)
OandM_PV_annual_fraction = 0.02;
OandM_bat_annual_fraction = 0.01;
OandM_diesel_annual_fraction = 0.03;

lifetime_years = 25;       % years for levelized costs (simplified)

%% --------------------- 3. AUX VARIABLES --------------------------------
% Precompute cost-per-unit items consistent units
Cinv_panel_per_panel = Cinv_PV_per_kW * (P_PV_STC/1000); % $ per panel
Cinv_cell = Cinv_bat_per_kWh * (C_cell/1000);           % $ per cell (Wh->kWh)
Cinv_diesel_unit = Cinv_diesel_per_unit;                 % $ per genset

% Lower / upper bounds for GA decision vector X = [Npv; Ncell; Ndiesel; SOC_ini_percent]
E_load_Wh = sum(P_load) * deltaT; % total energy demand in Wh for simulation horizon
% Rough guess for min PV to supply energy (avoid dividing by zero if E_panel small)
E_panel_Wh = sum((eta_PV * P_PV_STC / G_STC) .* Pirr .* deltaT);
if E_panel_Wh <= 0
    E_panel_Wh = 1;
end
Npv_min = max(1, round(E_load_Wh / E_panel_Wh * 0.2)); % small lower bound
Npv_max = 60000;
Ncell_min = 1;
Ncell_max = 20000;
Ndiesel_min = 0;  % allow zero genset (maybe PV+battery suffice)
Ndiesel_max = ceil(max(P_load) / (Diesel_unit_capacity_W));
SOC_ini_min = SOC_min*100;
SOC_ini_max = SOC_max*100;

Xl = [Npv_min; Ncell_min; Ndiesel_min; SOC_ini_min];
Xu = [Npv_max; Ncell_max; Ndiesel_max; SOC_ini_max];

%% --------------------- 4. OBJECTIVE & CONSTRAINT HANDLES ---------------
% Objective (only uses X and global params via anonymous handle)
obj = @(X) obj_cost(X, Cinv_panel_per_panel, Cinv_cell, Cinv_diesel_unit, ...
    OandM_PV_annual_fraction, OandM_bat_annual_fraction, OandM_diesel_annual_fraction, ...
    Fuel_cost_per_Wh, lifetime_years, deltaT, P_load, eta_DCDC, eta_ACDC, eta_charge, eta_discharge, ...
    C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, Pirr, Tenv, Diesel_efficiency, Diesel_unit_capacity_W);

% Nonlinear constraints
nlcon = @(X) nl_constraints(X, deltaT, P_load, eta_DCDC, eta_ACDC, eta_charge, eta_discharge, ...
    C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, Pirr, Tenv, Diesel_efficiency, Diesel_unit_capacity_W);

%% --------------------- 5. RUN GENETIC ALGORITHM ------------------------
% Use GA; for stability set integer constraints on Npv, Ncell, Ndiesel; SOC_ini treat continuous
intCon = [1,2,3];
opts = optimoptions('ga','Display','iter','PopulationSize',60,'MaxGenerations',80);

rng('default');
[Xopt, cost_opt] = ga(obj, 4, [], [], [], [], Xl, Xu, nlcon, intCon, opts);

Npv_opt = round(Xopt(1));
Ncell_opt = round(Xopt(2));
Ndiesel_opt = round(Xopt(3));
SOC_ini_opt = Xopt(4)/100;

fprintf('\nOptimized system sizing:\n');
fprintf('  PV panels      : %d panels\n', Npv_opt);
fprintf('  Battery cells  : %d cells\n', Ncell_opt);
fprintf('  Diesel units   : %d units\n', Ndiesel_opt);
fprintf('  Initial SOC    : %.2f (fraction)\n', SOC_ini_opt);
fprintf('  Objective cost : $%.2f (simplified, lifetime aggregated)\n', cost_opt);

%% --------------------- 6. RUN SIMULATION WITH OPTIMAL DESIGN ----------
% Run comp_Power to get time-series with the optimal X
[P_pv, P_bat, SOC, P_lost, P_unsup, Diesel_power, N_cycles, Bat_capacity_end] = comp_Power( ...
    Npv_opt, Ncell_opt, Ndiesel_opt, SOC_ini_opt, deltaT, P_load, eta_DCDC, eta_ACDC, ...
    eta_charge, eta_discharge, C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, Pirr, Tenv, 0, 0, 0, Diesel_efficiency, Diesel_unit_capacity_W);

%% --------------------- 7. PLOTTING ------------------------------------
figure('Name','Time Series Overview','NumberTitle','off');
subplot(3,1,1);
plot(time, P_load/1000,'k','LineWidth',1.5); hold on;
plot(time, (eta_DCDC.*P_pv)/1000,'-','LineWidth',1); % DC->bus scaled for view
plot(time, Diesel_power/1000,'r--','LineWidth',1);
xticks(xticks_idx); xticklabels(months);
ylabel('Power (kW)'); legend('Load','PV (DC to bus)','Diesel (AC)'); grid on;

subplot(3,1,2);
plot(time, P_bat/1000,'b','LineWidth',1.2); hold on;
plot(time, P_lost/1000,'m','LineWidth',1.2);
xticks(xticks_idx); xticklabels(months);
ylabel('Power (kW)'); legend('Battery (positive=discharge)','Lost PV (curtail)','Location','best'); grid on;

subplot(3,1,3);
plot(0:n, SOC,'LineWidth',1.5); xticks(0:24:n); xticklabels(["0" months]); ylabel('SOC (fraction)'); xlabel('Time step'); grid on;

figure('Name','Load vs Supply Area','NumberTitle','off');
area(time, max(0,(eta_DCDC.*P_pv))/1000,'FaceAlpha',0.6); hold on;
area(time, max(0,P_bat)/1000,'FaceAlpha',0.6);
area(time, Diesel_power/1000,'FaceAlpha',0.6);
plot(time, P_load/1000,'k','LineWidth',1.6);
xticks(xticks_idx); xticklabels(months);
legend('PV','Battery','Diesel','Load','Location','bestoutside'); ylabel('Power (kW)'); grid on;

diesel_fraction = Diesel_power ./ P_load;
diesel_fraction(P_load == 0) = 0; % avoid NaN
figure('Name','Diesel Share','NumberTitle','off');
plot(time, diesel_fraction,'b','LineWidth',1.6);
xticks(xticks_idx); xticklabels(months);
ylabel('Diesel Share of Load'); xlabel('Time');
ylim([0 1]); grid on;
title('Fraction of Load Supplied by Diesel');

%% --------------------- 8. SUPPORTING FUNCTIONS ------------------------
% The rest of the script contains all helper functions used above.
% 1) comp_P_PV
% 2) comp_Power (dispatch PV -> Battery -> Diesel)
% 3) obj_cost (objective function)
% 4) nl_constraints (nonlinear constraints)

%% ----------------- helper: comp_P_PV ---------------------------------
function P_PV = comp_P_PV(Npv, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, T_PV)
    % Returns DC power per timestep from Npv panels (W)
    % Apply temperature correction but avoid negative multiplier
    temp_factor = max(0, 1 + CT.*(T_PV - T_STC));
    P_single = eta_PV .* (P_PV_STC./G_STC) .* temp_factor .* GSR; % W per panel
    P_PV = Npv .* max(0, P_single);
end

%% ----------------- helper: comp_Power --------------------------------
function [P_pv, P_bat, SOC, P_lost, P_unsup, Diesel_power, N_cycles, Bat_capacity] = comp_Power( ...
        Npv, Ncell, Ndiesel, SOC_ini, deltaT, P_load, eta_DCDC, eta_ACDC, eta_charge, eta_discharge, ...
        C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, T_PV, years, k_cyc, k_cal, Diesel_efficiency, Diesel_unit_capacity_W)

    % Get PV power (DC)
    P_pv = comp_P_PV(Npv, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, T_PV); % W

    n = length(P_pv);
    P_unsup = zeros(1,n);
    P_lost = zeros(1,n);
    Diesel_power = zeros(1,n);
    P_bat = zeros(1,n);
    SOC = zeros(1,n+1);
    SOC(1) = min(max(SOC_ini, SOC_min), SOC_max);

    Bat_capacity_Wh = Ncell * C_cell; % Wh
    throughput_Wh = 0;
    N_cycles = 0;

    diesel_cap_ac_W = Ndiesel * Diesel_unit_capacity_W;

    for k=1:n
        % DC bus: PV after DC conversion
        Ppv_dc = eta_DCDC * P_pv(k);
        Pload_dc = P_load(k) / eta_ACDC; % represent AC load as DC-equivalent
        net = Ppv_dc - Pload_dc; % positive => surplus, negative => deficit

        if net >= 0
            % surplus: charge battery (respect efficiency) then curtail
            % available headroom Wh
            headroom_Wh = (SOC_max - SOC(k)) * Bat_capacity_Wh;
            % max charge power (W) limited by headroom and charge eff
            max_charge_W = headroom_Wh / (eta_charge * deltaT);
            Pcharge = min(net, max(0, max_charge_W));
            if Pcharge > 0
                P_bat(k) = -Pcharge; % negative indicates charging
                SOC(k+1) = SOC(k) + (eta_charge * Pcharge * deltaT) / Bat_capacity_Wh;
                throughput_Wh = throughput_Wh + eta_charge * Pcharge * deltaT;
            else
                SOC(k+1) = SOC(k);
            end
            P_lost(k) = max(0, net - Pcharge);

        else
            % deficit: discharge battery first, then diesel
            deficit = -net; % W needed on DC bus
            avail_Wh = (SOC(k) - SOC_min) * Bat_capacity_Wh;
            max_discharge_W = (avail_Wh * eta_discharge) / deltaT; % deliverable to DC bus
            Pdis = min(deficit, max(0, max_discharge_W));
            if Pdis > 0
                P_bat(k) = Pdis; % positive discharge
                SOC(k+1) = max(SOC_min, SOC(k) - (Pdis * deltaT) / (eta_discharge * Bat_capacity_Wh));
                throughput_Wh = throughput_Wh + (Pdis * deltaT) / eta_discharge;
            else
                SOC(k+1) = SOC(k);
            end
            deficit = max(0, deficit - Pdis);

            if deficit > 0 && Ndiesel > 0
                % Diesel produces AC; convert to DC support using eta_ACDC
                needed_ac_W = deficit / eta_ACDC;
                produced_ac_W = min(needed_ac_W, diesel_cap_ac_W);
                Diesel_power(k) = produced_ac_W; % AC side
                provided_dc = produced_ac_W * eta_ACDC;
                deficit = max(0, deficit - provided_dc);
            end

            P_unsup(k) = deficit; % remaining unmet energy on DC side (W)
        end
    end

    % estimate cycle count roughly (throughput relative to full capacity)
    N_cycles = throughput_Wh / (2 * Bat_capacity_Wh);

    % simple calendar+cycle capacity fade
    Bat_capacity = Bat_capacity_Wh * (1 - k_cyc * N_cycles - k_cal * years);
end

%% ----------------- helper: obj_cost ---------------------------------
function cost = obj_cost(X, Cinv_panel_per_panel, Cinv_cell, Cinv_diesel_unit, ...
    OandM_PV_annual_fraction, OandM_bat_annual_fraction, OandM_diesel_annual_fraction, ...
    Fuel_cost_per_Wh, lifetime_years, deltaT, P_load, eta_DCDC, eta_ACDC, eta_charge, eta_discharge, ...
    C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, Tenv, Diesel_efficiency, Diesel_unit_capacity_W)

    % Unpack X
    Npv = round(X(1));
    Ncell = round(X(2));
    Ndiesel = round(X(3));
    SOC_ini = X(4)/100;

    % Simulate one-year horizon to compute Diesel energy used
    [~, ~, ~, ~, P_unsup, Diesel_power, ~, ~] = comp_Power(Npv, Ncell, Ndiesel, SOC_ini, deltaT, P_load, ...
        eta_DCDC, eta_ACDC, eta_charge, eta_discharge, C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, Tenv, 0, 0, 0, Diesel_efficiency, Diesel_unit_capacity_W);

    % Lifetime economic terms (simplified aggregation over lifetime_years)
    % CAPEX
    CAPEX = Npv * Cinv_panel_per_panel + Ncell * Cinv_cell + Ndiesel * Cinv_diesel_unit;

    % Annual O&M approximated as fraction of CAPEX; aggregated as fraction*lifetime
    OPEX_agg = (Npv * Cinv_panel_per_panel * OandM_PV_annual_fraction + ...
        Ncell * Cinv_cell * OandM_bat_annual_fraction + Ndiesel * Cinv_diesel_unit * OandM_diesel_annual_fraction) * lifetime_years;

     % Fuel cost based on Diesel_power time-series (Diesel_power in W AC); energy Wh = sum(P*dt)
    Diesel_elec_Wh = sum(Diesel_power) * deltaT;             % Wh electrical (AC)
    Diesel_fuel_Wh = Diesel_elec_Wh / Diesel_efficiency;     % Wh fuel energy consumed
    Fuel_cost_agg = Diesel_fuel_Wh * Fuel_cost_per_Wh * lifetime_years; % $ over lifetime
    Penalty_unmet = 15 * sum(P_unsup) * deltaT; % $10 per Wh unmet
    cost = CAPEX + OPEX_agg + Fuel_cost_agg + Penalty_unmet;
end

%% ----------------- helper: nl_constraints ------------------------------
function [c, ceq] = nl_constraints(X, deltaT, P_load, eta_DCDC, eta_ACDC, eta_charge, eta_discharge, ...
    C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, Tenv, Diesel_efficiency, Diesel_unit_capacity_W)

    Npv = round(X(1));
    Ncell = round(X(2));
    Ndiesel = round(X(3));
    SOC_ini = X(4)/100;

    [~, ~, SOC, ~, P_unsup, Diesel_power, ~, ~] = comp_Power(Npv, Ncell, Ndiesel, SOC_ini, deltaT, P_load, ...
        eta_DCDC, eta_ACDC, eta_charge, eta_discharge, C_cell, SOC_min, SOC_max, P_PV_STC, G_STC, CT, T_STC, eta_PV, GSR, Tenv, 0, 0, 0, Diesel_efficiency, Diesel_unit_capacity_W);

    % Constraints:
    % c <= 0
    % 1. SOC never below SOC_min
    c1 = SOC_min - min(SOC);
    % 2. SOC never above SOC_max
    c2 = max(SOC) - SOC_max;
    % 3. No unmet demand allowed (P_unsup must be zero)
    %c3 = max(P_unsup);
    % 4. Diesel not exceeding nameplate (AC)
    c3 = max(Diesel_power) - Ndiesel * Diesel_unit_capacity_W;
    % 5. Cyclic SOC optional (we enforce start ~ end) small tolerance
    c4 = abs(SOC(1) - SOC(end)) - 0.05; % allow 5% drift

    

    c = [c1, c2, c3, c4];
    ceq = [];
end

%%
% Energy [Wh]
E_unsup = sum(P_unsup * deltaT);   % unmet load energy over the year
E_load  = sum(P_load * deltaT);    % total demand energy over the year

% Percentage unmet
percent_unmet = 100 * E_unsup / E_load;

fprintf('Total unmet load = %.2f MWh (%.2f%% of annual load)\n', ...
    E_unsup/1000, percent_unmet);
```

<!--
## Embedding youtube video
The second video has the autoplay on. copy and paste the 11-digit id found in the url link. <br>
*Example* : https://www.youtube.com/watch?v={**MhVw-MHGv4s**}&ab_channel=engineerguy
{% include youtube-video.html id="MhVw-MHGv4s" autoplay= "false"%}
{% include youtube-video.html id="XGC31lmdS6s" autoplay = "true" %}

you can also set up custom size by specifying the width (the aspect ratio has been set to 16/9). The default size is 560 pixels x 315 pixels.  

The width of the video below. Regardless of initial width, all the videos is responsive and will fit within the smaller screen.
{% include youtube-video.html id="tGCdLEQzde0" autoplay = "false" width= "900px" %}  


<br>

## Adding a hozontal line
---

## Starting a new line
leave two spaces "  " at the end or enter <br>

## Adding bold text
this is how you input **bold text**

## Adding italic text
Italicized text is the *cat's meow*.

## Adding ordered list
1. First item
2. Second item
3. Third item
4. Fourth item

## Adding unordered list
- First item
- Second item
- Third item
- Fourth item

## Adding code block
```ruby
def hello_world
  puts "Hello, World!"
end
```

```python
def start()
  print("time to start!")
```

```javascript
let x = 1;
if (x === 1) {
  let x = 2;
  console.log(x);
}
console.log(x);

```

## Adding external links
[Wikipedia](https://en.wikipedia.org)


## Adding block quote
> A blockquote would look great if you need to highlight something


## Adding table 

| Header 1 | Header 2 |
|----------|----------|
| Row 1, Col 1 | Row 1, Col 2 |
| Row 2, Col 1 | Row 2, Col 2 |

make sure to leave aline betwen the table and the header
-->

