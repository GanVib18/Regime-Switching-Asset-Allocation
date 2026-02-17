# Regime-Switching Asset Allocation
==================================

A machine learning portfolio strategy for Canadian investors that detects market
regimes using a Hidden Markov Model and adjusts allocations accordingly.

10-year backtest (2015–2024) using five TSX-listed ETFs, with walk-forward
out-of-sample validation.


Results (walk-forward average across 4 out-of-sample periods)
--------------------------------------------------------------
Strategy                  Sharpe    Ann. Return    Max DD
---------------------------------------------------------
60/40 Benchmark            0.88        9.5%        -7.3%
80/20 Benchmark            1.04       13.1%        -8.2%
Vol Threshold              1.08       11.1%        -7.3%
3-Regime HMM               1.06       11.0%        -6.8%
3-Regime HMM Optimized     1.27       15.0%        -8.3%


How It Works
------------
1. Ten market features are computed daily (realised volatility, momentum,
   credit spread proxy, yield curve slope, equity-bond correlation).

2. A Gaussian HMM classifies each day into one of three regimes:
   - Low-Vol Bull   (~52% of days): high equity allocation
   - Medium-Vol Normal (~46%): balanced allocation
   - High-Vol Crisis (~2%): defensive — bonds and gold heavy

3. Portfolio weights are derived per regime using mean-variance optimisation
   with Ledoit-Wolf covariance shrinkage. The crisis allocation is hard-coded
   (55% bonds / 25% gold / 20% equities) due to insufficient training data.

4. Daily allocations blend regime weights by posterior probability rather than
   hard-switching. A 5-day dwell filter suppresses noisy regime flips.


ETF Universe (TSX-listed, no currency conversion required)
----------------------------------------------------------
XIU.TO   iShares S&P/TSX 60
VFV.TO   Vanguard S&P 500 Index (CAD-hedged)
XEF.TO   iShares MSCI EAFE
XBB.TO   iShares Canadian Bond Index
CGL-C.TO iShares Gold Bullion (CAD-hedged)


Requirements
------------
Python 3.8+
yfinance
hmmlearn
scikit-learn
scipy
numpy
pandas
matplotlib
seaborn
joblib

Install: pip install yfinance hmmlearn scikit-learn scipy numpy pandas matplotlib seaborn joblib


Usage
--------------------
Run the full pipeline with optimised weights and walk-forward validation:

    results = quick_start_with_walk_forward()

Step-by-step:

    data              = prepare_data_only()
    data_with_regimes = train_model_only()
    portfolio         = run_backtest_only(use_optimization=True)
    comparison        = compare_strategies(use_optimization=True)
    walk_forward      = run_walk_forward_only()

Generate charts and tables for the blog post:

    create_presentation_materials()

Custom risk aversion parameters:

    run_pipeline(use_optimization=True, risk_aversion={0: 2.0, 1: 3.0, 2: 5.0})


Output Files
------------
/regime_switching_output/
    cleaned_data.csv              — feature-engineered daily data
    data_with_regimes.csv         — data with HMM regime labels
    model_hmm.pkl                 — saved HMM model bundle
    portfolio_optimized.csv       — daily portfolio values
    walk_forward_results.csv      — per-split out-of-sample metrics
    model_comparison_summary.csv  — full-sample strategy comparison

/regime_switching_output/presentation_charts/
    01_regime_timeline.png
    02_cumulative_returns.png
    03_drawdown_analysis.png
    04_rolling_metrics.png
    05_metrics_comparison.png
    06_risk_return_scatter.png
    07_regime_allocations.png
    08_walk_forward_validation.png


Limitations
-----------
- Only 52 crisis-regime days in the full dataset. Crisis allocation is
  hard-coded rather than optimised for this reason.
- The 60/40 benchmark holds zero gold; this strategy holds 10–25% at all
  times. Some outperformance reflects a structural gold tilt, not purely
  regime-switching skill.
- Walk-forward validation covers 4 one-year splits — enough to span
  different macro environments, but not a large statistical sample.
- Sharpe ratios in the walk-forward table use a constant 2% risk-free rate.
  Full-sample Sharpe uses the actual time-varying T-bill rate (^IRX).
- Transaction costs modelled as a flat 0.08% bid-ask spread per trade.


Disclaimer
----------
For educational purposes only. Not investment advice.
Past performance is not indicative of future results.
