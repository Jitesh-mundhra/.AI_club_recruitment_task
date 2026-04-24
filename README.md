# Electricity Demand Forecasting for Bangladesh 🔌

## Overview
A machine learning pipeline to forecast electricity demand (MW) 
for Bangladesh using socioeconomic and time series indicators (2014–2025).

## Results
| Model | MAPE |
|---|---|
| XGBoost | 4.68% ✅ |
| Random Forest | 7.05% |

## 🗂️ Dataset

| Source | Description | Period |
|--------|-------------|--------|
| PGCB (`PGCB_date_power_demand.xlsx`) | Hourly demand, generation by fuel type, load shedding | 2015–2025 |
| Weather (`weather_data.xlsx`) | Hourly apparent temperature, humidity, precipitation, cloud cover, sunshine | 2015–2025 |
| World Bank (`economic_full.csv`) | Macroeconomic & demographic indicators for Bangladesh | 2014–2025 |

## Features Used
- previous weekyly,daily,last hour demand data
- other Energy generating sources data
- Lag features (1h,2h,3h,1w)
- GDP per capita
- Urban population %
- relative demand of every hour
- Employment in industry %
- Apparent Temp(C)
- Net migration (log transformed)
- population of agglomeration of more than 1 million


---

## 🧹 Missing Data & Anomaly Handling

### Power Data
- **Duplicate timestamps** → resolved by taking the **mean** (demand_mw is a rate, not a sum)
- **Zero values** in `demand_mw`, `gas`, `solar`, `wind` etc. treated as invalid →
  replaced with `NaN` → filled using **time-based interpolation** to respect temporal order
- **Outliers** handled with two strategies:
  - `liquid_fuel`, `coal`, `hydro`, `solar` → **percentile clipping** (1st–99th), replaced with column mean
  - `gas`, `wind` → **rolling window method** (window=11, centered), values outside
    0.5×–1.5× of neighboring mean were clipped
- Dropped `india_adani`, `nepal` (sparse/unreliable), `remarks` (no relation to load shedding found)

### Economic Data
- Missing 2025 values filled using **polynomial regression (degree=2)**
  fitted on 2014–2024 — appropriate for accelerating GDP/population trends
- `Access to electricity` dropped — near 100% in recent years, near-zero variance

---

## ⚙️ Feature Engineering

### Temporal Features
| Feature | Description | Why |
|---------|-------------|-----|
| `hour_sin`, `hour_cos` | Circular encoding of hour (24h cycle) | Prevents discontinuity between hour 23 and 0 |
| `month_sin`, `month_cos` | Circular encoding of month (12-month cycle) | Smooth seasonal transitions |
| `demandr_lag_1h` | Demand 1 hour ago | Short-term momentum |
| `demand_lag_24h` | Demand 24 hours ago | Pre-peak patterns |
| `demand_lag_1w` | Demand 168 hours ago | Same hour last week — weekly seasonality |
| 'generation_lag_2h' | Generation 2 hour ago | To support the momentum pattern |
| `relative_demand_h` | Normalized avg demand by hour (0–1) | Encodes typical daily load curve shape |

### Weather Features
| Feature | Reason Selected |
|---------|----------------|
| `apparent_temperature` | Covers both temp + humidity effect on AC/heating load |
| `relative_humidity_2m` | Direct cooling/comfort driver |
| `precipitation` | Affects outdoor activity and industrial operations |
| `cloud_cover` | Proxy for solar generation and lighting demand |
| `sunshine_duration` | Solar irradiance proxy |

> Dropped: `temperature_2m` (redundant with apparent temp), `dew_point_2m`
> (absorbed into apparent temp), `soil_temperature` (high correlation),
> `wind_direction` (irrelevant for demand)

### Economic Features
| Feature | Transformation | Signal |
|---------|---------------|--------|
| `GDP per capita` | Log transform | Income → appliance ownership → demand |
| `Urban_population_%` | Derived (urban/total × 100) | Urbanization rate |
| `Net migration` | Signed log transform | Population redistribution |
| `Population in urban agglomerations > 1M %` | — | Metro demand concentration |
| `Industry value added % GDP` | — | Industrial electricity load |
| `T&D losses %` | — | Grid efficiency over time |
| `Employment in industry %` | — | Industrial demand proxy |

---

## 🔍 Key Insights from Feature Importance

| Rank | Feature | Importance | Insight |
|------|---------|------------|---------|
| 1 | `demand_lag_24h` | 0.28 | Yesterday same hour — strongest repeating daily pattern |
| 2 | `demand_lag_1h` | 0.22 | Short-term momentum drives hourly prediction |
| 3 | `demand_lag_1w` | 0.11 | Weekly rhythm — same hour last week is highly predictive |
| 4 | `hour_sin` | 0.065 | Time-of-day cycle (sine component) |
| 5 | `hour_cos` | 0.065 | Time-of-day cycle (cosine component) |
| 6 | `india_bheramara_hvdc` | 0.058 | Cross-border power import signal |
| 7 | `gas` | 0.035 | Fuel mix affects generation availability |
| 8 | `generation_lag_2h` | 0.028 | Recent generation output as demand proxy |
| 9 | `apparent_temperature` | 0.020 | AC/fan load during hot seasons |
| 10 | `coal` | 0.015 | Base load generation source |
| 11 | `load_shedding` | 0.013 | Suppressed demand correction |
| 12 | `Net_migration_log` | 0.012 | Population shift drives long-term demand growth |
| 13 | `liquid_fuel` | 0.011 | Peaking plant usage indicator |
| 14 | `india_tripura` | 0.010 | Regional cross-border import signal |
| 15 | `solar` | 0.010 | Renewable generation offset on demand |


> **Lag features dominate** — `demand_lag_24h`, `demand_lag_1h`, and `demand_lag_1w`
> together account for ~61% of total feature importance, confirming that
> electricity demand is highly autocorrelated.

> **Engineered features** (`monthly_avg`, `weekday_avg`, `peak_season_avg`, `weekend_avg`)
> have low individual importance but provide useful seasonal baselines
> that complement the lag-based signals.

> **Cross-border imports** (`india_bheramara_hvdc`, `india_tripura`) rank
> surprisingly high, reflecting Assam/Northeast India's dependence on
> grid interconnections for supply balancing.
---


## Tech Stack
![Python](https://img.shields.io/badge/Python-3.x-blue)
![XGBoost](https://img.shields.io/badge/XGBoost-green)
![Sklearn](https://img.shields.io/badge/Scikit--learn-orange)

## Key Findings
- XGBoost achieves 4.68% MAPE, beating industry standard of 5%
- Electricity demand strongly driven by lag features and urbanization
- Lag features critical for capturing short-term demand patterns

## Author
Jitesh Mundhra
