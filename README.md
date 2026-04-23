# Electricity Demand Forecasting for Bangladesh 🔌

## Overview
A machine learning pipeline to forecast electricity demand (MW) 
for Bangladesh using socioeconomic and time series indicators (2014–2025).

## Results
| Model | MAPE |
|---|---|
| XGBoost | 2.54% ✅ |
| Random Forest | 4.64% |
| Linear Regression | 89.71% |

## Dataset
- World Bank socioeconomic indicators (Bangladesh)
- Historical electricity demand data (hourly/monthly)
- Period: 2014–2025

## Features Used
- GDP per capita (log transformed)
- Urban population %
- Industry value added % GDP
- Employment in industry %
- T&D losses %
- Net migration (log transformed)
- Labor force total
- Lag features (1h, 3h, 1w)

# Observation and optimisation
-

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
Your Name
