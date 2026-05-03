---
layout: post
title: Japan Gas Supplier Network Map for SOFC Market Entry
description: Architected an ETL pipeline in Python and Excel to aggregate and clean data from 200+ regional gas suppliers across Japan, then built an interactive choropleth network map using Folium to visualize supplier density by municipality. The analysis successfully identified 8 strategic biogas partners for Bosch's SOFC market entry strategy.
skills:
  - Python (Folium, Pandas)
  - ETL Pipeline Design
  - Geospatial Data Visualization
  - Market Analysis
  - Data Engineering
main-image: /map_cover.png
---

**Company**: Bosch | Solid Oxide Fuel Cell (SOFC) Project  
**Role**: Business Analyst Intern  
**Location**: Tokyo, Japan  
**Tools**: Python (Folium, Pandas), Excel, GeoJSON  

---

# Objective

To support Bosch's SOFC market entry in Japan by identifying strategic biogas supply partners. The goal was to build a data pipeline that aggregates regional gas supplier information across Japan, geocodes it by municipality, and visualizes it as an interactive map — enabling the business team to identify high-density supplier regions and shortlist strategic partners for biogas-fed SOFC deployment.

---

# Methodology

## Data Pipeline (ETL)

The pipeline was built in Python and Excel across three stages:

1. **Extract** — Collected gas supplier data from 200+ regional providers across Japan, compiled into a structured Excel database with administrative area codes, supplier names, and geographic coordinates.
2. **Transform** — Cleaned and standardized supplier records using Pandas. Aggregated supplier counts per municipality. Merged with Japan's official administrative boundary GeoJSON (N03 dataset) by matching administrative area codes.
3. **Load** — Fed the cleaned, merged dataset into a Folium choropleth map rendered as a self-contained interactive HTML file.

## Interactive Map Features

The map was built using Folium with the following layers:

- **Choropleth layer** — Municipality polygons colored by gas supplier count (BuGn color scale), with dark gray fill for municipalities with no data
- **Marker layer** — Green pin markers at each municipality centroid, with a popup listing all available gas providers on click
- **Search functionality** — Municipality-level search using the Folium Search plugin
- **Layer control** — Toggle markers on/off independently from the choropleth

The map is centered on Tokyo and exported as a standalone HTML file (`InteractiveJapanDigitalGasMap.html`) that can be shared without any server or dependencies.

---

# Assumptions

- Administrative area codes in the CSV match the N03_007 field in the official GeoJSON boundary dataset
- Supplier count per municipality is treated as a proxy for biogas availability and market accessibility
- Geographic coordinates (lat/lng) are pre-assigned per municipality centroid in the source data
- Only municipalities present in the CSV are rendered — unmatched GeoJSON features are filtered out

---

# Results

- Built a fully interactive map covering 200+ gas supplier records across Japanese municipalities
- Choropleth visualization revealed clear geographic clustering of biogas supply density in specific prefectures
- Marker popups provided the business team with direct supplier name lookup per region
- The analysis successfully identified **8 strategic partners** for SOFC market entry based on supplier density, geographic proximity to target deployment sites, and biogas availability
- The HTML output was shared directly with the Bosch SOFC project team as a decision-support tool

---

# Significance & Impact

- Transformed a fragmented Excel database into a spatial, interactive decision tool — reducing manual analysis time significantly.
- Provided a replicable geospatial analysis framework applicable to other market entry studies (e.g., hydrogen infrastructure, EV charging networks).
- Directly contributed to Bosch's SOFC commercial strategy in Japan by grounding partner selection in data rather than intuition.
- Demonstrates end-to-end data engineering competence: from raw supplier data to a production-ready interactive visualization.

---

# Key Takeaway

- ETL pipelines don't need to be complex to be impactful — a well-structured Pandas + GeoJSON merge can turn a spreadsheet into a strategic asset.
- Geospatial visualization makes patterns in regional data immediately actionable for non-technical stakeholders.
- Matching administrative codes between datasets is the critical join step — data quality here determines map accuracy.
- Folium's self-contained HTML output is ideal for sharing interactive maps in corporate environments without requiring a web server.

---

# Figures and Visualization

## Interactive Network Map — Gas Supplier Density by Municipality

![](/_projects/Japan Gas Supplier Network Map for SOFC Market Entry/map_cover.png){:height="450px"}

<br>
Choropleth map of Japan showing gas supplier count per municipality (BuGn color scale). Darker green indicates higher supplier density. Gray municipalities have no data. Green markers indicate supplier locations with clickable popups listing available providers.
<br>

## Supplier Density Detail — Kanto, Chubu, Kansai Region

![](/_projects/Japan Gas Supplier Network Map for SOFC Market Entry/map_kanto_detail.png){:height="450px"}

<br>
Zoomed view of the Kanto, Chubu, Kansai region (Tokyo and surrounding prefectures) showing the concentration of gas suppliers. High-density clusters in this region were among the primary candidates for SOFC partner shortlisting.
<br>


# Python Code

```python
import folium
from folium import plugins
import json
import pandas as pd

# ── Load municipality boundary data (Japan N03 GeoJSON) ───────────────────────
geojson_path = 'N03-19_190101.geojson'
with open(geojson_path, 'r', encoding='utf-8_sig') as f:
    geojson = json.load(f)

# ── Load gas supplier data from Excel-exported CSV ────────────────────────────
csv_path = 'Japan Digital Map Database.csv'
prod_df = pd.read_csv(csv_path, dtype='object')
prod_df['Gas provider count'] = prod_df['Gas provider count'].astype('float')

# ── Filter GeoJSON to only municipalities present in the CSV ──────────────────
filtered_geojson = {
    "type": "FeatureCollection",
    "features": [
        feature for feature in geojson["features"]
        if feature["properties"]["N03_007"] in prod_df["Administrative area code"].values
    ]
}

# ── Create base map centered on Tokyo ─────────────────────────────────────────
map_center = [35.66388889, 139.6980556]
m2 = folium.Map(location=map_center, tiles='cartodbpositron', zoom_start=7)

# ── Choropleth layer — supplier count by municipality ─────────────────────────
choropleth_gas_provider = folium.Choropleth(
    geo_data=filtered_geojson,
    data=prod_df,
    columns=['Administrative area code', 'Gas provider count'],
    key_on='feature.properties.N03_007',
    fill_color='BuGn',
    nan_fill_color='darkgray',
    fill_opacity=0.8,
    nan_fill_opacity=0.8,
    line_opacity=0.2,
    legend_name='Number of Available Gas Suppliers'
).add_to(m2)

# ── Marker layer — one pin per municipality with supplier popup ───────────────
markers_fg = folium.FeatureGroup(name='Markers').add_to(m2)
for index, item in prod_df.iterrows():
    popup_text = "Available Gas Providers: " + item['Gas providers combined'] + "<br/>"
    folium.Marker(
        location=[item['lat'], item['lng']],
        popup=popup_text,
        icon=folium.Icon(color='green')
    ).add_to(markers_fg)

# ── Search plugin — search by municipality name ───────────────────────────────
search = plugins.Search(
    layer=choropleth_gas_provider,
    search_label='N03_004',
    geom_type='Point'
).add_to(m2)

# ── Layer control — toggle markers independently ──────────────────────────────
folium.LayerControl().add_to(m2)

# ── Export as standalone HTML ─────────────────────────────────────────────────
m2.save('InteractiveJapanDigitalGasMap.html')
print("Map saved.")
```
