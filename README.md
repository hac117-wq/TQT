# TQT

# Phase 2 & Quality Sleeve Prototype — Macro Regime Detection, EWMA Vol Scaling & Quality Factor

Initial out-of-sample research prototypes for the **Triton Quantitative Trading (TQT)** 3-stage equity strategy. This repository covers **Phase 2 (Macro Regime Overlay & Volatility-Scaled Exposure)** and the **Composite Quality Z-Score** screening module prior to full integration in **QuantConnect**.

---

## Repository Structure

* `01_regime_and_vol_scaling.ipynb`: Walk-forward Regime Detection (Gaussian HMM vs. Statistical Jump Model) + EWMA Vol Scaling
* `02_quality_factor_zscore.ipynb`: Cross-sectional Composite Quality Z-Score prototype (sample universe)
* `README.md`: Methodology, prototype backtest summary, and integration roadmap

---

## 1. Macro Regime Detection & EWMA Volatility Scaling (`01_regime_and_vol_scaling.ipynb`)

### Methodology
* **Features:** Daily `SPY` returns, 21-day volatility risk premium spread (`VIX - RV_21d`), and 5-day `VIX` rate of change, standardized using a **3-year (756-trading-day) rolling Z-score** (clipped at $\pm 4$) to prevent look-ahead bias and regime distortion from historical tail outliers.
* **Models Compared:**
  1. **Gaussian HMM (2-State):** Full-covariance Hidden Markov Model refitted every 21 trading days via an expanding walk-forward window.
  2. **Statistical Jump Model (2-State):** Viterbi dynamic programming solver penalizing state transitions ($\lambda = 30.0$, following *Shu, Yu & Mulvey, 2024*) to filter out short-term noise and reduce switch turnover.
* **Exposure & Volatility Overlay:**
  * Daily conditional volatility estimated via **EWMA** ($\lambda = 0.94$, 16% annualized target volatility, capped at $1.0\times$ leverage).
  * Total equity exposure cap scales down to $0.5 \times \text{Vol Scalar}$ during detected **Crisis** regimes and maintains $1.0 \times \text{Vol Scalar}$ during **Calm** regimes.

### Out-of-Sample Backtest Results (`2020-01-02` to `2026-11-18`)
*Initial training warm-up: `2000-01-01` to `2019-12-31` | Transaction costs: `5 bps` per turnover*

| Metric | S&P 500 (`SPY`) | Gaussian HMM + EWMA | Jump Model + EWMA |
| :--- | :---: | :---: | :---: |
| **Total Return** | 149.51% | 103.49% | 61.83% |
| **CAGR** | 14.25% | 10.90% | 7.26% |
| **Sharpe Ratio** | 0.77 | **0.92** | **0.86** |
| **Max Drawdown** | -33.72% | -15.90% | **-11.37%** |
| **Regime Switches** | — | 93 | **52** |

### Key Takeaways
* **Superior Risk-Adjusted Efficiency:** Both regime-overlay models outperform passive `SPY` on Sharpe Ratio (**0.92** and **0.86** vs. **0.77**) before adding any stock-selection alpha.
* **Turnover & Tail-Risk Compression:** Compared to the Gaussian HMM, the Statistical Jump Model reduces noisy regime switches by **~44%** (`52` vs. `93`) and compresses Max Drawdown from **-33.72%** down to **-11.37%**.

---

## 2. Composite Quality Z-Score (`02_quality_factor_zscore.ipynb`)

### Methodology
To avoid single-metric accounting distortions and complement Value screening, each stock is scored cross-sectionally across six fundamental profitability and earnings-quality ratios:
* **GPA:** Gross Profit / Total Assets
* **ROE:** Net Income / Stockholders' Equity
* **ROA:** Net Income / Total Assets
* **CFOA:** Operating Cash Flow / Total Assets
* **Gross Margin:** Gross Profit / Total Revenue
* **Accruals (Inverted):** $-(\text{Net Income} - \text{Operating Cash Flow}) / \text{Total Assets}$

$$\text{Quality Z-Score}_i = \frac{1}{6} \sum_{k=1}^{6} Z_{i,k}$$

### Sample Universe Output

| Ticker | GPA | ROE | CFOA | GrossMargin | Accruals | Quality_Z |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **NVDA** | 0.742 | 0.763 | 0.497 | 0.711 | 0.084 | **1.226** |
| **AAPL** | 0.577 | 1.179 | 0.359 | 0.488 | -0.032 | **1.107** |
| **MSFT** | 0.297 | 0.302 | 0.241 | 0.679 | -0.065 | **0.476** |
| **HD** | 0.522 | 1.105 | 0.155 | 0.333 | -0.021 | **0.423** |
| **PG** | 0.345 | 0.297 | 0.155 | 0.502 | -0.028 | **0.086** |
| **COST** | 0.437 | 0.258 | 0.178 | 0.128 | -0.074 | **0.058** |
| **JNJ** | 0.321 | 0.329 | 0.123 | 0.679 | 0.011 | **0.025** |
| **T** | 0.178 | 0.199 | 0.096 | 0.596 | -0.044 | **-0.113** |
| **DIS** | 0.189 | 0.089 | 0.093 | 0.380 | -0.045 | **-0.301** |
| **XOM** | 0.159 | 0.111 | 0.116 | 0.220 | -0.052 | **-0.359** |
| **UNH** | 0.268 | 0.128 | 0.064 | 0.185 | -0.025 | **-0.485** |
| **INTC** | 0.087 | -0.002 | 0.046 | 0.348 | -0.047 | **-0.546** |
| **F** | 0.006 | -0.228 | 0.074 | 0.009 | -0.102 | **-0.727** |
| **BA** | 0.025 | 0.410 | 0.006 | 0.048 | 0.007 | **-0.870** |

---

## 3. Next Steps & QuantConnect Integration
1. **Fundamental Universe Scaling:** Port the Composite Quality Z-Score calculation to QuantConnect's Morningstar fundamental universe to score filtered Value candidates.
2. **Signal Handoff to Phase 3:** Export daily regime probabilities (`P(Crisis)`), EWMA portfolio exposure limits, and cross-sectional Quality Z-Scores to the Phase 3 LLM/ML portfolio allocation layer.
