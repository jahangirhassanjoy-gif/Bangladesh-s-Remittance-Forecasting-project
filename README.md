# 🇧🇩 Bangladesh Remittance Forecasting

**A time-series predictive analytics project forecasting Bangladesh's monthly remittance inflow — and a case study in knowing when a model can (and can't) be trusted.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3%2B-orange)
![statsmodels](https://img.shields.io/badge/statsmodels-0.14%2B-green)
![License](https://img.shields.io/badge/license-Educational-lightgrey)

---

## 📌 Overview

Bangladesh Bank, commercial banks, and money-transfer operators need to plan foreign
exchange liquidity and staffing around monthly remittance inflows — the country's most
important source of foreign currency. This project builds a model to forecast
next-month remittance inflow (US$ million) using 187 months of central bank data
(Jan 2011 – Jul 2026), spanning exchange rates, foreign reserves, migrant worker
deployment, Islamic holiday timing, government incentive policy, and Gulf oil-market
indicators.

**Problem type:** Time-series regression, evaluated with a strictly chronological
train/test split (never random).

## 🏆 Headline Finding

This project's most important result isn't a leaderboard win — it's a demonstrated,
evidence-based example of **when sophisticated models fail, and why**.

On the most recent 24 months of data, a naive "predict last month's value" baseline
beat every machine-learning model *and* a classical SARIMAX model. Investigation traced
this to Bangladesh's **August 2024 political transition**, which triggered a crackdown
on informal "hundi" remittance channels and drove official remittances to unprecedented
record highs — a structural break no model trained on prior history could have
anticipated.

A robustness check on an earlier, stable-regime window confirms the modeling approach
itself is sound: **Gradient Boosting achieves R² = +12.6%** there (vs. the naive
baseline's −5.7%), proving the engineered features genuinely add value once you're not
asking a model to predict something completely unprecedented.

> 📖 Full narrative, evidence, and sourcing: [`reports/Remittance_Forecasting_Report.docx`](reports/Remittance_Forecasting_Report.docx), Section 9.

## 📁 Repository Structure

```
├── data/
│   ├── Data.xlsx                     # raw source data (Bangladesh Bank compilation)
│   ├── Sources.xlsx                  # full data-source citations & modeling caveats
│   └── remittance_cleaned.csv        # cleaned data after Step 1
├── notebooks/
│   └── Remittance_Forecasting.ipynb  # full analysis — runs end-to-end in Google Colab
├── src/
│   ├── 01_preprocessing_eda.py       # cleaning, EDA, domain-flag script
│   ├── 02_feature_modeling.py        # feature engineering, modeling, tuning, evaluation
│   └── 03_robustness_check.py        # stable-regime robustness check
├── figures/                          # all 15 EDA & evaluation charts (PNG)
├── reports/
│   └── Remittance_Forecasting_Report.docx   # full written report
├── docs/
│   └── Remittance_Forecasting_StudyGuide.pdf # 33-page beginner's guide to the whole project
├── requirements.txt
└── README.md
```

## 🚀 Quickstart

**Run in Google Colab (recommended, zero setup):**
1. Open [Google Colab](https://colab.research.google.com/)
2. Upload `data/Data.xlsx` to the file panel
3. Upload and open `notebooks/Remittance_Forecasting.ipynb`
4. `Runtime → Run all`

**Run locally:**
```bash
git clone <this-repo-url>
cd bangladesh-remittance-forecasting
pip install -r requirements.txt
python src/01_preprocessing_eda.py
python src/02_feature_modeling.py
python src/03_robustness_check.py
```

## 🔬 Methodology

| Stage | What Was Done |
|---|---|
| **Preprocessing** | Forward-fill for the single incomplete trailing month; COVID-era collapse encoded as a regime dummy (not capped as an outlier); IQR outlier detection shown to be *inappropriate* for this trending series and deliberately not applied |
| **EDA** | Full time series, seasonal decomposition, Eid-effect quantification, exchange-rate/incentive overlay, oil/GCC relationships, ACF/PACF |
| **Feature Engineering** | Autoregressive lags (1, 2, 3, 12), a leakage-safe rolling mean, FX reserves lagged one month (per a documented endogeneity caveat), cyclical month encoding, a trend index, a COVID regime dummy |
| **Feature Selection** | Correlation + Random Forest importance, computed both with and without own-lag features, to separate autocorrelation from genuine exogenous drivers |
| **Model Building** | 2 mandatory time-series baselines (naive, seasonal naive) + 5 models (Linear Regression, Decision Tree, Random Forest, SVR, Gradient Boosting) + SARIMAX, on a chronological (never random) split |
| **Optimization** | `GridSearchCV` with `TimeSeriesSplit` (not plain K-Fold) |
| **Evaluation** | Full metrics suite (MAE, MAPE, MSE, RMSE, R²), plus a critical robustness check across two different test windows |

## 📊 Results

**Full test window (Aug 2024 – Jul 2026, spans the structural break):**

| Model | MAE | MAPE | RMSE | R² |
|---|---|---|---|---|
| 🥇 **Naive (persistence)** | 316.4 | 11.19% | 382.1 | **9.76%** |
| Linear Regression | 396.5 | 13.26% | 503.8 | −56.86% |
| Seasonal Naive (lag-12) | 505.3 | 18.03% | 589.4 | −114.68% |
| SARIMAX(1,1,1)(1,1,1,12) | 701.9 | 24.07% | 789.2 | −284.94% |
| Gradient Boosting (Optimized) | 704.8 | 23.83% | 808.1 | −303.52% |
| Random Forest (Optimized) | 773.3 | 26.21% | 878.6 | −377.02% |
| Support Vector Machine | 1,044.9 | 36.21% | 1,135.1 | −696.28% |

**Robustness check — stable-regime window (ends Jun 2024, no structural break):**

| Model | RMSE | R² |
|---|---|---|
| 🥇 **Gradient Boosting (Optimized)** | 261.8 | **12.61%** |
| Random Forest (Optimized) | 286.3 | −4.51% |
| Naive (persistence) | 287.9 | −5.70% |

## 💡 Key Business Insights

- **Eid holidays reliably lift remittance ~9%** — a recurring, plannable seasonal effect for bank liquidity management
- **Exchange rate and cash-incentive policy both track formal-channel remittance volume** — consistent with reduced use of informal "hundi" transfer channels
- **Model forecasts should be trusted in stable periods and heavily discounted during political/economic upheaval** — the single most actionable operating rule this project produced

## ⚠️ Limitations

- Structural breaks (political, policy) are fundamentally unpredictable from historical data alone
- Only 187 monthly observations — short for capturing multiple full business cycles
- Tree-based models cannot extrapolate beyond their training range — a real weakness during periods of rapid, unprecedented growth
- The exchange rate used is the interbank rate, not the effective rate migrants actually received

## 📖 New to Predictive Analytics?

[`docs/Remittance_Forecasting_StudyGuide.pdf`](docs/Remittance_Forecasting_StudyGuide.pdf)
is a 33-page companion guide that explains **every chart, every metric, and every line
of code in this repository in plain English** — written for someone who has never taken
a stats class or opened Python before. It includes a full jargon glossary, a
block-by-block code walkthrough, and a script for explaining the project to someone
else.

## 📄 Data Sources & License

Dataset compiled from public Bangladesh Bank sources — see [`data/Sources.xlsx`](data/Sources.xlsx)
for the complete citation trail and documented modeling caveats. This project (code,
notebook, report, and study guide) is provided for educational purposes.
