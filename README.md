# 📈 Enhancing Momentum Investing with Skewness Filtering

> **Financial Markets Analytics Project** | Master's in Data Science  
> Testing whether a skewness screen improves momentum investing strategies on the S&P 500 (1990–2018)

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📌 Project Overview

Momentum investing — buying recent winners and avoiding recent losers — is one of the most well-documented anomalies in financial markets. This project investigates whether adding a **skewness filter** on top of momentum strategies improves risk-adjusted performance on S&P 500 stocks.

The key idea: stocks with **low rolling skewness** (negatively skewed return distributions) tend to have more predictable momentum. By keeping only stocks in the **lowest skewness quintile** before applying momentum rules, we test whether this simple screen materially enhances performance.

**Research question:** Does filtering by skewness before applying momentum strategies improve Sharpe ratio and terminal wealth compared to plain momentum?

---

## 👥 Team & Roles

| Name | Role |
|------|------|
| **Stephen Adu Poku Yeboah** | Strategy design for all momentum variants (TS, CS, Dual) and skewness overlay; cap-bucket analysis (idea & implementation); contributed to momentum & skewness coding |
| Gianluca Licciardello | Lead coder — full pipeline implementation, performance metrics, results analysis |

---

## 🗂️ Dataset

- **Universe:** S&P 500 constituent stocks
- **Period:** December 1990 – November 2018 (monthly prices)
- **Source:** Monthly adjusted closing prices for all S&P 500 constituents
- **Note:** Significant NaN values present for stocks that entered the index later — handled during preprocessing

---

## 📐 Strategies

### 1. Plain Momentum (my contribution)

All three momentum strategies use a **12–1 lookback** (12-month return, skipping the most recent month) with **lagged weights** to avoid look-ahead bias.

**Time-Series (TS) Momentum**
- Go **long** a stock if its own past 12–1 return is ≥ 0
- Otherwise **exclude** it from the portfolio
- Each eligible stock is equally weighted

**Cross-Sectional (CS) Momentum**
- Rank all investable stocks by their 12–1 momentum score each month
- Hold only the **top 20%** (best performers relative to peers)
- Rebalance monthly

**Dual Momentum**
- Hold only stocks that **pass both** TS and CS criteria simultaneously
- Most selective strategy — requires positive absolute momentum AND top relative ranking
- Results in a more concentrated but higher-conviction portfolio

### 2. Skewness-Enhanced Momentum

Before applying any of the three momentum rules above:
- Compute **36-month rolling skewness** for each stock (shifted by 1 month to avoid look-ahead)
- Keep only stocks in the **lowest skewness quintile (Q1)**
- Then apply TS, CS, or Dual momentum within that filtered universe

### 3. Cap-Bucket Analysis

At each date, split the universe by index weight into:
- **Small cap** — bottom 20% by weight
- **Mid cap** — next 30%
- **Large cap** — top 50%

Run all 6 strategies (3 plain + 3 skewness-enhanced) within each bucket separately.

---

## 🔍 My Contribution — Strategy Design, Cap-Bucket Analysis & Coding

I was responsible for designing all three momentum strategies and the skewness overlay logic, proposed and implemented the cap-bucket analysis (splitting the universe into Small, Mid, and Large cap groups), and contributed to the coding implementation alongside Gianluca.

**Key implementation details:**

- **12–1 lookback:** computed as the cumulative return from month t−12 to t−2 (skipping t−1) using `pct_change()` and rolling products on monthly price data
- **Lagged weights:** portfolio weights at month t are determined using only information available at t−1, preventing any forward-looking bias
- **TS signal:** binary mask — 1 if 12–1 return ≥ 0, 0 otherwise — applied before equal weighting
- **CS ranking:** `rank(pct=True)` across all stocks each month, selecting top quintile (rank > 0.8)
- **Dual filter:** logical AND of TS and CS masks, keeping only stocks satisfying both conditions simultaneously
- **Missing value handling:** stocks with insufficient history (< 13 months) excluded from signal computation using `min_periods` parameter
- **Equal weighting:** within each eligible set, portfolio weights normalized to sum to 1 each month

---

## 📊 Key Results

### Overall (Full Universe)

| Strategy | Avg Monthly Return | Monthly Sharpe | Max Drawdown |
|----------|--------------------|----------------|--------------|
| S&P 500 Benchmark | ~0.72% | 0.179 | -59.77% |
| Plain TS | ~0.98% | 0.193 | ~-53% |
| Plain CS | ~0.99% | 0.190 | ~-51% |
| Plain Dual | ~1.00% | 0.195 | ~-50% |
| **Skew-Enhanced TS** | **~1.01%** | **0.200** | ~-50% |
| **Skew-Enhanced CS** | **~1.00%** | **0.205** | ~-50% |
| **Skew-Enhanced Dual** | **~1.02%** | **0.203** | **-49.93%** |

> Skewness-enhanced portfolios generally deliver **higher terminal wealth** and **higher Sharpe ratios** than plain momentum across the full sample.

### By Cap Bucket

One of the key contributions of this project was splitting the S&P 500 universe into **Small, Mid, and Large cap** buckets to evaluate whether the skewness enhancement behaves differently across firm sizes. The results reveal striking differences.

**Small Cap (bottom 20% by index weight)**

| Strategy | Monthly Sharpe |
|----------|---------------|
| Benchmark | 0.179 |
| Plain TS | 0.186 |
| Plain CS | 0.190 |
| Plain Dual | 0.188 |
| **Skew-Enhanced TS** | **0.225** |
| **Skew-Enhanced CS** | **0.210** |
| **Skew-Enhanced Dual** | **0.215** |

→ The skewness filter adds the **most value** in small caps. The Sharpe ratio for TS jumps from 0.186 to 0.225 — a +20% improvement. Small cap stocks are more volatile and less efficiently priced, making the skewness screen particularly effective at filtering out lottery-like, crash-prone names.

---

**Mid Cap (next 30% by index weight)**

| Strategy | Monthly Sharpe |
|----------|---------------|
| Benchmark | 0.179 |
| Plain TS | 0.193 |
| Plain CS | 0.190 |
| Plain Dual | 0.195 |
| **Skew-Enhanced TS** | **0.205** |
| **Skew-Enhanced CS** | **0.203** |
| **Skew-Enhanced Dual** | **0.210** |

→ The **strongest overall effect** is observed here — all three skewness-enhanced strategies dominate their plain counterparts with the widest terminal wealth gaps across all buckets. Mid cap stocks combine enough liquidity to trade efficiently with enough inefficiency to reward the skewness screen.

---

**Large Cap (top 50% by index weight)**

| Strategy | Monthly Sharpe |
|----------|---------------|
| Benchmark | 0.179 |
| Plain TS | 0.190 |
| Plain CS | 0.193 |
| Plain Dual | 0.200 |
| **Skew-Enhanced TS** | **0.202** |
| Skew-Enhanced CS | 0.195 |
| Skew-Enhanced Dual | 0.201 |

→ The skewness filter is **less decisive** in large caps — likely because mega-cap stocks are more efficiently priced and exhibit lower return skewness variation. However, skewness-enhanced TS still outperforms its plain version, and both Dual strategies comfortably beat the benchmark.

---

**💡 Key Takeaway from Cap-Bucket Analysis**

> The smaller the firm, the more the skewness filter helps. This makes economic sense: small and mid cap stocks are more exposed to idiosyncratic risk and lottery-ticket behaviour — exactly the kind of noise the skewness screen is designed to remove. For large caps, momentum alone is already more effective, but skewness still provides a marginal boost in most cases.

---

## 💡 Conclusions

- Adding a simple skewness screen to momentum strategies **consistently improves** both total return and Sharpe ratio vs plain momentum and the S&P 500 benchmark
- The effect is **strongest in Small and Mid caps** and weaker (but still present) in Large caps
- **Trade-offs to consider:** skewness portfolios hold fewer stocks → higher concentration and tracking error; higher monthly turnover → transaction costs matter in practice
- **Future work:** incorporate transaction costs, test on international markets, explore LSTM-based momentum signals

---

## 📁 Repository Structure

```
momentum-skewness-sp500/
├── README.md
└── Financial_Markets_Analytics_project.ipynb   # Full analysis: data loading, strategies, results
```

---

## 🚀 How to Run

### Prerequisites
```bash
pip install pandas numpy matplotlib plotly
```

### Run the notebook
```bash
jupyter notebook Financial_Markets_Analytics_project.ipynb
```

> ⚠️ The notebook was originally run on **Google Colab** with data loaded from Google Drive. To run locally, update the data loading cell to point to your local file path instead of `/content/gdrive/...`.

---

## 🛠️ Technologies

| Tool | Purpose |
|------|---------|
| Python | Core language |
| Pandas | Data manipulation, rolling calculations, monthly rebalancing |
| NumPy | Return computations, signal construction |
| Matplotlib | Static performance charts |
| Plotly | Interactive cumulative return and drawdown charts |

---

## 📄 License

Academic project — Università degli Studi di Milano-Bicocca, MSc Data Science.
