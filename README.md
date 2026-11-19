# TQT

1. Macro Regime Detection (Gaussian HMM vs. Jump Model) + EWMA Vol ScalingOut-of-sample walk-forward test (2000–2019 initial train, 2020-01-02 to 2026-11-18 OOS evaluation, after 5bps transaction costs) comparing a Gaussian HMM vs. the Statistical Jump Model ($\lambda = 30.0$) paired with EWMA volatility scaling ($\lambda = 0.94$, 16% target vol) on SPY:Plaintext=== Phase 2 Prototype: Regime Detection + EWMA Vol Scaling (2020-01-02 to 2026-11-18) ===
                S&P 500 (SPY)   Gaussian HMM + EWMA   Jump Model + EWMA
Total Return          149.51%               103.49%              61.83%
CAGR                   14.25%                10.90%               7.26%
Sharpe Ratio             0.77                  0.92                0.86
Max Drawdown          -33.72%               -15.90%             -11.37%
Regime Switches             —                    93                  52
Risk-Adjusted Outperformance: Both regime-overlay models beat SPY on Sharpe Ratio (0.92 / 0.86 vs. 0.77), while the Jump Model cuts noisy regime switches by ~44% (52 vs. 93) and compresses Max Drawdown from -33.72% down to -11.37%.2. Composite Quality Z-Score PrototypeCross-sectional composite Z-score combining GPA, ROE, ROA, CFOA, Gross Margin, and inverted Accruals (sample universe check before scaling to QuantConnect's full fundamental universe):Plaintext=== Composite Quality Z-Score Prototype (Sample Universe) ===
          GPA    ROE   CFOA  GrossMargin  Accruals  Quality_Z
Ticker                                                       
NVDA    0.742  0.763  0.497        0.711     0.084      1.226
AAPL    0.577  1.179  0.359        0.488    -0.032      1.107
MSFT    0.297  0.302  0.241        0.679    -0.065      0.476
HD      0.522  1.105  0.155        0.333    -0.021      0.423
PG      0.345  0.297  0.155        0.502    -0.028      0.086
COST    0.437  0.258  0.178        0.128    -0.074      0.058
JNJ     0.321  0.329  0.123        0.679     0.011      0.025
T       0.178  0.199  0.096        0.596    -0.044     -0.113
DIS     0.189  0.089  0.093        0.380    -0.045     -0.301
XOM     0.159  0.111  0.116        0.220    -0.052     -0.359
UNH     0.268  0.128  0.064        0.185    -0.025     -0.485
INTC    0.087 -0.002  0.046        0.348    -0.047     -0.546
F       0.006 -0.228  0.074        0.009    -0.102     -0.727
BA      0.025  0.410  0.006        0.048     0.007     -0.870
Notice how the composite score cleanly ranks high-margin compounders (NVDA, AAPL, MSFT) at the top while penalizing single-metric distortions at the bottom (e.g., BA has an artificially high ROE due to leverage, but low GPA/CFOA pushes it to the bottom).
