# Electricity Demand Forecasting for Bangladesh 🔌

## Overview
A machine learning pipeline to forecast electricity demand (MW) 
for Bangladesh using socioeconomic and time series indicators (2014–2025).

## Results
| Model | MAPE |
|---|---|
| XGBoost | 2.54% ✅ |
| Random Forest | 4.64% |

## 🗂️ Dataset

| Source | Description | Period |
|--------|-------------|--------|
| PGCB (`PGCB_date_power_demand.xlsx`) | Hourly demand, generation by fuel type, load shedding | 2015–2025 |
| Weather (`weather_data.xlsx`) | Hourly apparent temperature, humidity, precipitation, cloud cover, sunshine | 2015–2025 |
| World Bank (`economic_full.csv`) | Macroeconomic & demographic indicators for Bangladesh | 2014–2025 |

## Features Used
- GDP per capita (log transformed)
- Urban population %
- Industry value added % GDP
- Employment in industry %
- T&D losses %
- Net migration (log transformed)
- Labor force total
- Lag features (1h, 3h, 1w)


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
| `hour_lag_1h` | Demand 1 hour ago | Short-term momentum |
| `hour_lag_3h` | Demand 3 hours ago | Pre-peak patterns |
| `hour_lag_1w` | Demand 168 hours ago | Same hour last week — weekly seasonality |
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

| Rank | Feature | Insight |
|------|---------|---------|
| 1 | `relative_demand_h` | Daily load curve shape is the strongest repeating pattern |
| 2 | `apparent_temperature` | AC/fan load dominates Bangladesh summers |
| 3 | `hour_lag_1w` | Weekly rhythm — same hour last week is highly predictive |
| 4 | `hour_sin/cos` | Time-of-day cycle |
| 5 | `GDP_log` | Captures long-term demand growth trend |
| 6 | `hour_lag_1h` | Short-term momentum |
| 7 | `month_sin/cos` | Seasonal demand pattern |
| 8 | `Urban_population_%` | Urbanization-driven demand growth |
| 9 | `T&D losses %` | Grid efficiency changes over time |
| 10 | `Employment in industry %` | Industrial load proxy |

> Short-term temporal features dominate hourly prediction.
> Economic indicators capture the long-term upward trend
> from ~8,000 MW to ~14,000+ MW over 2015–2025.

---


## Tech Stack
![Python](https://img.shields.io/badge/Python-3.x-blue)
![XGBoost](https://img.shields.io/badge/XGBoost-green)
![Sklearn](https://img.shields.io/badge/Scikit--learn-orange)

## Project Structure
├── data/
│   ├── economic_full.csv
│   └── df_merged.csv
├── notebooks/
│   └── demand_forecasting.ipynb
├── models/
│   └── xgboost_demand_mw.pkl
└── README.md

## How to Run
    # Install dependencies
    pip install pandas numpy scikit-learn xgboost matplotlib seaborn scipy joblib

    # Run notebook
    jupyter notebook notebooks/demand_forecasting.ipynb

## Key Findings
- XGBoost achieves 2.54% MAPE, beating industry standard of 5%
- Electricity demand strongly driven by GDP growth and urbanization
- Lag features critical for capturing short-term demand patterns

## Author
Jitesh Mundhra
