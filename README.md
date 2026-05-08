# Market Volatility Detector

## Overview
A quantitative market intelligence tool that ingests hourly OHLC data across major forex pairs, Gold and NAS100, calculates true range volatility, labels trading sessions and applies an ensemble anomaly detection model to identify institutional liquidity sweep windows with precision.

## Problem Statement
Retail traders consistently get stopped out before price moves in their predicted direction. The root cause is institutional algorithms executing scheduled liquidity sweeps at specific time windows — raiding stop losses to fill large orders before reversing to the intended direction.

This tool identifies exactly when and where those sweeps occur so traders can protect their positions, wait for the sweep to complete and enter at a higher probability window.

## Assets Covered
- EURUSD, GBPUSD, USDJPY, USDCHF, AUDUSD, USDCAD
- DXY (US Dollar Index)
- Gold (GC=F)
- NAS100 (NQ=F)

## Methodology

### 1. Data Ingestion
Hourly OHLC data pulled via yfinance across all assets for a defined date range. Sunday candles are filtered out to remove partial open candles that distort volatility calculations.

### 2. True Range Calculation
True Range is used instead of simple High minus Low to capture the full extent of price movement including gaps between candles — the most accurate measure of volatility for stop hunt detection.

```
TR = max(High - Low, |High - Previous Close|, |Low - Previous Close|)
```

### 3. Session Labelling
Every candle is tagged with its trading session based on UTC time:

| Session | UTC Hours |
|---|---|
| Asian | 00:00 - 08:00 |
| London | 08:00 - 13:00 |
| London-NY Overlap | 13:00 - 17:00 |
| London Close | 17:00 |
| New York | 18:00 - 21:00 |
| Off Hours | 21:00 - 00:00 |

### 4. Ensemble Anomaly Detection
Three independent methods vote on whether a candle is anomalous. A candle requires at least 2 out of 3 votes to be flagged as a confirmed anomaly — reducing false positives from any single method being oversensitive.

| Method | Approach |
|---|---|
| Z-score | Statistical — flags values beyond 3 standard deviations |
| IQR | Robust statistical — flags values beyond 1.5x interquartile range |
| Isolation Forest | ML based — learns normal volatility pattern and flags outliers |

QuantileTransformer is used for scaling before Isolation Forest to handle the non-normal distribution of price data.

### 5. Cross Asset Macro Event Detection
Anomaly timestamps are compared across all assets simultaneously. When 2 or more assets flag an anomaly at the same timestamp it is classified as a macro liquidity event driven by institutional algorithms rather than an asset specific move.

## Key Findings
- London Close at 17:00 UTC is the highest risk window for stop hunts across all asset classes
- Tuesday and Thursday 17:00 UTC showed coordinated sweeps across 4 to 5 assets simultaneously confirming macro institutional execution
- Gold showed isolated anomalies during London session on Monday independent of forex pairs pointing to asset specific drivers
- Volatility clustering at specific hours confirms institutional algorithms operate on scheduled execution windows regardless of news calendar

## Tech Stack
Python, Pandas, NumPy, Matplotlib, Scikit-learn, SciPy, yfinance

## Project Structure
```
market-volatility-detector/
│
├── notebooks/
│   └── time_analysis.ipynb    # Complete pipeline in one notebook —
│                              # data ingestion, feature engineering,
│                              # session labelling, volatility calculation,
│                              # anomaly detection and visualisation
│
├── src/                       # To be populated after project completion
├── data/                      # Raw OHLC data
└── README.md
```

## How To Run
1. Clone the repository
2. Install dependencies:
```bash
pip install pandas numpy matplotlib seaborn yfinance scikit-learn scipy
```
3. Open `notebooks/time_analysis.ipynb`
4. Update `start` and `end` dates in Cell 2 to your desired date range
5. Run all cells in order

## Roadmap
- Correlation breakdown monitor with rolling correlation across pairs
- Telegram notification system for real time anomaly alerts
- Fibonacci retracement analysis for identifying institutional offload zones
- Extended backtesting across multiple weeks and market regimes

## Author
Victor — Backend and AI/ML Engineer with domain expertise in capital markets and retail forex trading.
