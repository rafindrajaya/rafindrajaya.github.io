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

main-image: _projects/demo-project/diesel-pv-bat-topology-medium.jpeg
---

---
# Logic of the Code 
<br>
## 1. Input Processing
-	Load, irradiance, and temperature are read from CSV.
-	Converted into consistent units (W, Wh, °C).
## 2. System Parameters
-	Define PV, battery, diesel, and cost parameters.
-	Establish GA decision vector: [Npv, Ncell, Ndiesel, SOC_ini].
## 3.	Optimization Setup
-	Objective = CAPEX + aggregated O&M + lifetime fuel cost + penalty for unmet load.
-	Constraints = SOC bounds, no excessive unmet demand, diesel not exceeding nameplate.
## 4.	Dispatch Simulation (per timestep)
-	Surplus case: PV → charge battery (respecting SOC & efficiency) → curtail excess.
-	Deficit case: Battery discharges → if still short, diesel supplies → if still short, unmet load recorded.
## 5.	Genetic Algorithm
-	GA searches for the cost-optimal mix of PV panels, battery cells, and diesel gensets.
-	Integer constraints ensure physical realism (whole panels, whole gensets).
## 6.	Post-Optimization Simulation
-	Runs the system with the optimal configuration.
-	Generates time-series of SOC, power flows, diesel use, and unmet load.
## 7.	Visualization
-	Multi-panel plots show:
-	Load vs PV vs Diesel.
-	Battery charge/discharge cycles and curtailment.
-	SOC trajectory.
-	Diesel share of demand.
-	(Optional) unmet load profile and percentages.

# Key Assumptions
## Load data: 
Simplified one year of demand, temperature, and irradiance (288 timesteps = 12 months × 24 hours). One month is represented by load profile of one day of that particular month for simplicity.
## PV system: 
- Standard test condition parameters, irradiance and temperature corrections, conversion efficiency losses.
-	Battery: Defined by capacity per cell, SOC operating window (20%–90%).
-	Charging/discharging efficiencies split for realism.
-	Simple cycle aging approximation.
-	Diesel generators:
1.	500 kW per unit.
2.	Includes efficiency, fuel cost per liter, and O&M.
- Economic assumptions:
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

### Embeed images
<img src="_projects/demo-project/loadprofileoptimal-medium.jpeg" alt="Load profile of the PV-Battery-Diesel topology" height="400">


## Embedding youtube video
The second video has the autoplay on. copy and paste the 11-digit id found in the url link. <br>
*Example* : https://www.youtube.com/watch?v={**MhVw-MHGv4s**}&ab_channel=engineerguy
{% include youtube-video.html id="MhVw-MHGv4s" autoplay= "false"%}
{% include youtube-video.html id="XGC31lmdS6s" autoplay = "true" %}

you can also set up custom size by specifying the width (the aspect ratio has been set to 16/9). The default size is 560 pixels x 315 pixels.  

The width of the video below. Regardless of initial width, all the videos is responsive and will fit within the smaller screen.
{% include youtube-video.html id="tGCdLEQzde0" autoplay = "false" width= "900px" %}  
-->

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


