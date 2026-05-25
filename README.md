# 🏥 Irish Public Healthcare Demand Analysis & Forecasting

> **MSc Business Analytics Dissertation** · Trinity College Dublin · Industry Project with Capventis  
> Group project with 2 collaborators
---

## 👤 My Contributions

- ✅ End-to-end **HSE employment data cleaning** — PDF extraction, hierarchy restructuring in R
- ✅ **Correlation analysis** — county-level data aggregation, matrix generation, visualisation
- ✅ All **time series forecasting** — EDA, model selection, ETS/SARIMA/Prophet implementation, cross-validation, final forecast generation

---

## 📌 Project Overview

This project was developed in collaboration with **Capventis**, a Dublin-based tech consulting firm specialising in data analytics and digital transformation. Capventis serves clients across the Irish public healthcare sector and commissioned this research to better understand healthcare **availability, demand, and capacity** across Ireland's 26 counties — with the goal of informing targeted, location-based investment decisions.

The analysis centres on **public hospitals in Ireland** (2014–2023) and covers three core strands:

| Strand | Description |
|---|---|
| **Exploratory Analysis** | Population, age, income, health ratings, GP distribution, hospital capacity by county |
| **Correlation Analysis** | Interdependencies between 7 socioeconomic and healthcare factors across counties |
| **Time Series Forecasting** | Predicting national outpatient wait-count using ETS, SARIMA, and Facebook's Prophet |

---

## 📂 Repository Structure

```
healthcare-forecasting-ireland/
│
├── data/
│   ├── raw/                        # Source data (CSO, HSE, NTPF — not included due to size)
│   └── processed/                  # Cleaned and aggregated datasets
│
├── R/
│   ├── 01_employment_data_cleaning.R   # HSE workforce data restructuring
│   ├── 02_correlation_data_prep.R      # County-level aggregation for correlation matrix
│   ├── 03_correlation_analysis.R       # Correlation matrix generation and visualisation
│   └── 04_forecasting.R                # ETS, SARIMA, and Prophet forecasting models
│
├── outputs/
│   ├── figures/                    # All charts and visualisations
│   └── forecasts/                  # 12-month Prophet forecast output (May 2023 – Apr 2024)
│
└── README.md
```

---

## 🗂️ Data Sources

| Source | Data Provided | Format |
|---|---|---|
| [Central Statistics Office (CSO)](https://www.cso.ie) | Population, age, gender, disposable income | XLSX |
| [Health Service Executive (HSE)](https://www.hse.ie) | Hospital workforce (WTE) | PDF (per hospital) |
| [National Treatment Purchase Fund (NTPF)](https://www.ntpf.ie) | Outpatient wait counts | XLSX |

**Data period:** January 2014 – April 2023 (111 monthly observations for forecasting)

---

## 🔧 Data Cleaning

### HSE Employment Dataset
The HSE workforce data presented a significant structural challenge: each hospital's data was in a **separate PDF file**, with job descriptions, post levels, and departments all combined in a single column using a three-level hierarchy (L0 → Specialty, L1 → Post, L2 → Department).

**Cleaning pipeline:**
1. Extracted relevant attributes from 50+ individual PDF files and consolidated into a single Excel file
2. Used a custom Excel `isBold()` function to programmatically identify and label L1 and L2 hierarchy levels
3. Wrote R code to iterate through the hierarchy and populate three separate columns — `Specialty`, `Post`, and `Department` — preserving hierarchical relationships
4. Resolved gaps in the dataset using **linear interpolation**

The result: a clean, analysis-ready dataset with WTE (Whole Time Equivalent) values correctly attributed by hospital, county, specialty, and department.

---

## 📊 Correlation Analysis

Conducted at **county level** using 2022 data (2020 for disposable income and GP referrals, the most recent available).

### Factors Analysed
- Total population
- Average age
- Gender distribution
- Disposable income per person
- Health ratings
- Average outpatient wait count
- Workforce strength (WTE)

### Key Findings

| Correlation | Value | Significance |
|---|---|---|
| Population ↔ Wait Count | **+0.95** | Strong positive — high population drives demand |
| Outpatient Count ↔ GP Referrals | **+0.82** | GPs are a reliable proxy for hospital demand |
| Population ↔ Disposable Income | **+0.59** | Denser counties attract higher-income residents |
| Average Age ↔ Health Rating | **−0.43** | Older populations report lower health ratings |

> All correlations were statistically significant at the **0.05 significance level**.

### Investment Implication
Regions with higher population correlate strongly with both higher wait counts (demand) and higher disposable income — making them attractive targets for private healthcare investment where willingness to pay is also high.

---

## 📈 Time Series Forecasting

### Data
- **Series:** National monthly outpatient wait count, Jan 2014 – Apr 2023
- **Observations:** 111 monthly data points
- **Train/Test split:** 80/20 → 88 training observations (Jan 2014 – Apr 2021), 23 test observations (May 2021 – Apr 2023)

### EDA Findings
- Series exhibits a **clear upward trend** with a notable spike in 2020 (COVID-19 backlog)
- **Seasonality confirmed** via decomposition: wait counts rise and fall alternately each year
- **Not white noise** (Ljung-Box test: p < 0.05)
- **Not a random walk** (ADF test: p ≈ 0)

### Models Applied

| Model | Description |
|---|---|
| **ETS(A,Ad,N)** | Holt-Winters exponential smoothing — additive errors, additive-damped trend, no seasonal component. Benchmark model. |
| **SARIMA(1,1,0) with drift** | Seasonal ARIMA — one autoregressive term, one differencing, no MA, with drift for systematic trend |
| **Prophet** | Facebook's Prophet — piecewise linear trend, Fourier-series seasonality, automatic changepoint detection |

### Model Performance

**Train set errors:**
| Model | RMSE | MAPE |
|---|---|---|
| ETS(A,Ad,N) | 6,496.57 | 0.88% |
| SARIMA(1,1,0) with drift | 6,010.50 | 0.81% |
| **Prophet** | **5,562.97** | **0.79%** |

**5-Fold Cross-Validation (decisive comparison):**
| Model | RMSE | MAPE |
|---|---|---|
| ETS(A,Ad,N) | 124,648.8 | 24.45% |
| SARIMA(1,1,0) with drift | 124,585.0 | 24.43% |
| **Prophet** | **48,404.76** | **6.00%** |

> Cross-validation was used as the decisive criterion because train/test results were inconclusive (ETS performed best on the test set, Prophet on the training set). Cross-validation removes overfitting bias by averaging performance across all data partitions.

**Prophet was selected as the final model** — it outperformed ETS and SARIMA by over 4× on MAPE in cross-validation.

### 12-Month Forecast (May 2023 – April 2024)

| Month | Forecast (Wait Count) |
|---|---|
| May 2023 | 688,187 |
| Jun 2023 | 683,253 |
| Jul 2023 | 684,042 |
| Aug 2023 | 685,137 |
| Sep 2023 | 684,274 |
| Oct 2023 | 679,873 |
| Nov 2023 | 671,246 |
| Dec 2023 | 660,154 |
| Jan 2024 | 661,017 |
| Feb 2024 | 661,640 |
| Mar 2024 | 663,268 |
| Apr 2024 | 666,256 |

The forecast shows a **continuing decrease through late 2023**, followed by an **anticipated uptick in early 2024** — consistent with seasonal patterns in the historical data.

---

## 💡 Key Insights

- 🏙️ **Dublin** accounts for **32.7% of all public hospitals** in Ireland, followed by Cork. County Meath and Kildare had the worst hospital-to-population ratio.
- 👶 Irish population grew **8.5% in 5 years** — Longford had the highest county-level growth at 13%.
- 🩺 The **top 3 counties by GP count** (Dublin, Cork, Galway) account for **49% of all GPs** in Ireland. County Meath has a GP-to-population ratio of 1:2,400.
- 📉 Health ratings **dropped in all counties** between 2016 and 2022 — likely COVID-19 impact. Largest drop in County Longford.
- 💉 GP referrals **peaked in 2021** and were predominantly for **cancer treatments**.
- 📊 Outpatient wait count **more than doubled** over 7 years, despite no significant growth in healthcare workforce.

---

## ⚠️ Limitations

1. **Private hospital data excluded** — HSE data covers public hospitals only, which may underestimate true national demand
2. **County-level forecasting not performed** — methodology is extensible to individual counties but was out of scope for this study
3. **External factors not modelled** — healthcare policy changes, economic shocks, and demographic shifts were not incorporated as exogenous variables

---

## 🛠️ Tech Stack

![R](https://img.shields.io/badge/R-276DC3?style=flat&logo=r&logoColor=white)
![Excel](https://img.shields.io/badge/Microsoft_Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)

**R packages used:** `forecast`, `prophet`, `ggplot2`, `dplyr`, `tseries`, `corrplot`



