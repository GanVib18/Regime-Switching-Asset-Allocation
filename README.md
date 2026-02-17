# Regime-Switching Asset Allocation

A machine learning portfolio strategy designed for Canadian investors. This project utilizes a **Hidden Markov Model (HMM)** to detect market regimes and dynamically adjust allocations across TSX-listed ETFs.

## Strategy Overview

This strategy performs a 10-year backtest (2015–2024) using five liquid TSX ETFs, validated through a rigorous **walk-forward out-of-sample** methodology.

### Key Results

*Average across 4 out-of-sample periods*

| Strategy | Sharpe Ratio | Ann. Return | Max Drawdown |
| --- | --- | --- | --- |
| 60/40 Benchmark | 0.88 | 9.5% | -7.3% |
| 80/20 Benchmark | 1.04 | 13.1% | -8.2% |
| Vol Threshold | 1.08 | 11.1% | -7.3% |
| 3-Regime HMM | 1.06 | 11.0% | -6.8% |
| **3-Regime HMM Optimized** | **1.27** | **15.0%** | **-8.3%** |

---

## How It Works

The pipeline follows a four-stage process to move from raw market data to tradeable signals:

1. **Feature Engineering**: Computes 10 daily market features, including realized volatility, momentum, credit spread proxies, yield curve slope, and equity-bond correlation.
2. **Regime Classification**: A **Gaussian HMM** classifies market states into three distinct regimes:
* **Low-Vol Bull** (~52% of days): High equity exposure.
* **Mid-Vol Normal** (~46% of days): Balanced, "all-weather" allocation.
* **High-Vol Crisis** (~2% of days): Defensive tilt—heavy on Bonds and Gold.


3. **Optimization**: Weights are derived using **Mean-Variance Optimization** with **Ledoit-Wolf shrinkage**. Note: The Crisis regime uses a defensive hard-coded split ( Bonds /  Gold /  Equities) due to the rarity of training samples.
4. **Signal Smoothing**: Uses a **5-day dwell filter** and posterior probability blending to prevent "whipsawing" (excessive trading from noisy regime flips).

---

## ETF Universe

All assets are TSX-listed, requiring no currency conversion for CAD-funded accounts.

| Ticker | Fund Name | Asset Class |
| --- | --- | --- |
| **XIU.TO** | iShares S&P/TSX 60 | Canadian Large Cap |
| **VFV.TO** | Vanguard S&P 500 (CAD-hedged) | US Equities |
| **XEF.TO** | iShares MSCI EAFE | Intl Developed |
| **XBB.TO** | iShares Canadian Bond Index | Fixed Income |
| **CGL-C.TO** | iShares Gold Bullion (CAD-hedged) | Commodities |

---

## Installation & Usage

### Requirements

```bash
pip install yfinance hmmlearn scikit-learn scipy numpy pandas matplotlib seaborn joblib

```

### Quick Start

Run the full pipeline with optimized weights and walk-forward validation:

```python
results = quick_start_with_walk_forward()

```

### Granular Control

```python
# Step-by-step execution
data              = prepare_data_only()
data_with_regimes = train_model_only()
portfolio         = run_backtest_only(use_optimization=True)

# Custom risk aversion parameters
run_pipeline(use_optimization=True, risk_aversion={0: 2.0, 1: 3.0, 2: 5.0})

```

---

## Project Structure

Results are exported to `/outputs/`, including:

* `data`: Contains the raw and processed time-series data used for model training and backtesting.
* `model`: The persistent ML model required to reproduce or extend the analysis.
* `weights`: The specific allocation logic used across different market environments.
* `charts`: High-resolution visualizations (PNG) for performance analysis and presentation.

---

## Limitations & Disclaimer

* **Sample Size**: The Crisis regime only contains 52 days of data; hence the manual defensive weighting.
* **Gold Bias**: The 60/40 benchmark lacks gold, while this strategy maintains 10–25%—some performance is derived from this structural tilt.
* **Costs**: Transaction costs are modeled at a flat 0.08% spread.

> This project is for **educational purposes only**. It is not investment advice. Past performance is never a guarantee of future results.
