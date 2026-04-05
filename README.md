## Groundwater trends in France (2005–2025)

👉 **What does 20 years of groundwater evolution look like?**

![Groundwater trends map](assets/images/map_preview.png)

👉 **See the interactive map**  
https://wrdvz.github.io/parallaxe-groundwater-france-trends/


This project explores long-term groundwater level trends across France using 1,080 monitoring stations from the national network.

The objective is simple in appearance, but methodologically challenging:

How to transform point-based groundwater observations into a spatial representation that remains physically meaningful?

## Key output 

In the docs file, you will find:
- National map of 20-year groundwater trends: parallaxe_groundwater_france_trends_2005_2025.png
- Interactive HTML version for exploration: index.html
- Aggregated dataset by hydrogeological entity: network70_affleurant_nv1_agg_2005_2025.csv


## The challenge

Groundwater data is inherently sparse and unevenly distributed.

Each station captures a local signal, but:
- stations are irregularly spaced
- aquifers are heterogeneous
- measurements are noisy and seasonal

Two core questions therefore emerged:
1.	How to measure groundwater trends robustly for each station?
2.	How to aggregate them spatially without introducing artefacts?


## Methodological framework

This project is structured around three key methodological decisions.

⸻

1. Station selection

A significant part of the work focused on selecting a reliable observational base.

Multiple datasets were evaluated, with different trade-offs:
- some offered many stations, but included inactive or unreliable sites
- others had good spatial coverage, but strong temporal gaps
- some were technically clean, but not used in operational contexts

The selected dataset is the network 070, which provides:
- consistent long-term time series
- broad national coverage
- alignment with operational groundwater monitoring (e.g. Météo des nappes)

This choice prioritizes robustness and comparability over exhaustiveness.

⸻

2. Trend estimation

Before addressing spatial aggregation, several approaches were tested to define a meaningful groundwater trend metric.

Tested approaches
- Simple difference (2005 → 2025): Too sensitive to start/end conditions, ignores variability
- Raw regression on monthly observations: Strongly affected by seasonality and short-term variability
- Winter recharge metrics: Strongly influenced by interannual climate variability

Selected approach
Groundwater level measurements are first aggregated into annual means for each station. A linear regression is then fitted over the 2005–2025 period to estimate long-term trends.

This method:
- uses all available observations
- reduces sensitivity to short-term variability
- provides stable and comparable estimates across stations

The slope is converted into a 20-year variation (cm) for interpretability.

This metric captures a long-term structural signal, rather than short-term dynamics.

⸻

3. Spatial aggregation

The final challenge was to aggregate station-level signals into a coherent spatial representation.

Several approaches were considered:
- interpolation (IDW): visually smooth, but physically artificial
- regular grids: simple, but geologically arbitrary
- watersheds: hydrologically relevant, but not aligned with groundwater systems
- multi-layer hydrogeological units: rich, but ambiguous due to overlapping units

Selected approach
Exclusive hydrogeological entities from BDLISA (NV1 outcropping units)

This approach combines geometry and geology:

- The territory is first divided into non-overlapping polygons (POLYG_ELEMENTAIRES), each representing a vertical slice of the subsurface.
- For each polygon, BDLISA provides a vertical stack of hydrogeological entities (TABLE_PILE_ENTITES_NIV1), ordered from surface to depth.
- By keeping only OrdRelatif = 1, we retain the uppermost unit, i.e. the aquifer that is in direct contact with the surface (outcropping unit).

Each polygon is assigned to a single hydrogeological entity (CodeEH), ensuring:
- full spatial coverage
- no overlaps
- consistency with geological structure

Aggregation
For each CodeEH:
- stations are spatially joined to hydrogeological entities
- trends are aggregated using the median to lower sensitivity to outliers
- entities with fewer than 5 stations are flagged as low support

Coverage
- Out of 181 outcropping hydrogeological entities (NIV1), 115 are covered by at least one monitoring station.
- Although this represents only 63.5% of entities, these account for 82.9% of the total mapped surface area.
- The uncovered entities are therefore likely to be smaller and more localized on average, which limits their effect on the macro-scale interpretation of the map.


## Interpretation

The resulting map represents a macro-level signal of groundwater evolution.

It should be interpreted as:
- a structural tendency over 20 years
- a proxy for pressure on groundwater systems
- an indicator of territorial robustness to hydrological variability

It does not aim to describe local conditions or short-term dynamics.


## Limitations

- heterogeneous station density across exclusive hydrogeological entities
- some hydrogeological entities are spatially fragmented
- aggregation may smooth internal variability
- no explicit uncertainty modelling beyond station count

This work is therefore best suited for macro-scale interpretation, not local prediction.


## Outputs

## Pipeline overview

1. Data acquisition: 00_fetch_hubeau_to_duckdb.py  

Output (path: data/raw/water/):
- Local DuckDB database containing raw station time series

This database is used as the input for subsequent processing steps.

2. Trend estimation & spatial aggregation: 01_compute_stations_trends.py  

Outputs (path: data/processed/mapping/):
- Station-level trend dataset: network70_station_to_polyg_affleurant_nv1_2005_2025.csv
- Polygon to aquifer mapping: bdlisa_polyg_affleurant_nv1_2005_2025.csv
- GeoJSON dissolved hydrogeological entities (full geometry): bdlisa_eh_nv1_dissolved_2005_2025.geojson
- GeoJSON dissolved hydrogeological entities (simplified web version): bdlisa_eh_nv1_dissolved_light_2005_2025.geojson

3. Publication layer: 02_generate_maps.py 

Outputs (path: docs/)
- Interactive HTML map: index.html
- Static PNG map: parallaxe_groundwater_france_trends_2005_2025.png
- Aggregated dataset by hydrogeological entity: network70_affleurant_nv1_agg_2005_2025.csv

## Repository structure

```bash
groundwater-france-trends/
├── assets/        # images used in documentation
├── data/          # raw and processed data (partially versioned)
├── docs/          # methodological notes
├── notebooks/     # exploratory analysis
├── outputs/       # maps and exports
├── scripts/       # execution pipeline
└── src/           # reusable modules
```


## Data sources

Groundwater level time series

Groundwater level observations are sourced from the national ADES database (BRGM), which centralizes groundwater monitoring data for France.

Data are accessed programmatically via the Hubeau API (EauFrance), which provides structured access to ADES time series.
- Data owner: BRGM
- Database: ADES
- Access channel: Hubeau API (hubeau.eaufrance.fr)
- Network used: Réseau 070

⸻

Hydrogeological reference framework

Spatial hydrogeological entities are derived from BDLISA (BRGM), the national hydrogeological reference database.

The following layers are used:
- POLYG_ELEMENTAIRES
- ENTITES_NIVEAU1
- TABLE_PILE_ENTITES_NIV1
- Data owner: BRGM
- Dataset: BDLISA

## Author

Edward Vizard  
Parallaxe processing


