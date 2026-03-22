# ETF & Mutual Fund Portfolio Optimization

Final project for **MATH 40570 — Mathematical Methods in Financial Economics** at the **University of Notre Dame**. The project applies mean-variance portfolio theory to real market data, constructing optimal portfolios from a universe of sector ETFs and mutual funds and validating the results with Monte Carlo simulation. The full written report is [`MMFE_Project.pdf`](MMFE_Project.pdf).

---

## Project Overview

The central questions are:
1. Given 15 years of historical returns across sector ETFs and mutual funds, where does the efficient frontier lie?
2. Which portfolios maximize risk-adjusted return (max Sharpe ratio) or minimize variance?

The analysis is split across two notebooks: a main analytical pipeline built for the course, and a companion OOP framework developed to handle real API data and more flexible portfolio construction.

---

## Notebooks

### [`portfolio_optimization.ipynb`](portfolio_optimization.ipynb)

End-to-end analytical pipeline for a universe of 11 sector ETFs and 11 mutual funds.

**Data** — 15-year monthly price history (Jan 2010 – Jan 2025) pulled via `yfinance`:

| Sector | ETF | Mutual Fund |
|---|---|---|
| Technology | VGT | VITAX |
| Financial Services | XLF | VFAIX |
| Consumer Cyclical | XLY | BPTRX |
| Healthcare | XLV | VGHCX |
| Communication Services | VOX | VTCAX |
| Industrials | XLI | VINAX |
| Consumer Defensive | XLP | VCSAX |
| Energy | XLE | VENAX |
| Real Estate | VNQ | VGSLX |
| Basic Materials | XLB | VMIAX |
| Utilities | XLU | VUIAX |

**Summary statistics** — annualized return and volatility per asset over the 15-year period. Highlights:
- **VGT** (Technology ETF): 18.6% annualized return, 1,198% total return — highest in the universe
- **XLV** (Healthcare ETF): 12.8% annualized return, lowest volatility at 13.6% — best risk-adjusted performance among defensive names

**Mean-variance optimization** — solves the long-only quadratic program (via `cvxpy`):

$$\min_w \; w^\top \Sigma w \quad \text{s.t.} \quad \mu^\top w = \bar{\mu}, \quad \mathbf{1}^\top w = 1, \quad w \geq 0$$

Traces the full efficient frontier from minimum-variance to maximum-return portfolios. Key results at a 4.32% risk-free rate:

| Portfolio | Annual Return | Weights |
|---|---|---|
| **Max Sharpe** | 16.1% | ~62% VGT, ~25% XLV |
| **Min Volatility** | 11.1% | ~30% XLV, ~6% VOX |

**Monte Carlo validation** — two approaches:
- *Random portfolios*: 10,000 randomly weighted portfolios plotted by return vs. volatility, colored by Sharpe ratio — confirms the analytical frontier as the outer boundary
- *Path simulation*: 1,000 GBM paths over a 10-year horizon for a $100,000 equal-weight portfolio, using Cholesky-decomposed correlated shocks to model co-movement between assets

---

### [`portfolio_classes.ipynb`](portfolio_classes.ipynb)

An OOP framework for portfolio construction, developed alongside the main analysis. Demonstrates a more flexible, data-driven approach using real price data from the Alpha Vantage API.

**Classes:**
- `Security` (base) → `Equity`, `Fixedincome`, `Option`, `Future` subclasses — each tracks returns, CVaR, max drawdown, and Sharpe ratio
- `Portfolio` — builds an efficient frontier by exhaustively simulating all valid weight combinations, then runs Monte Carlo path simulations with Plotly visualizations

**Applied to an 8-asset RRSP portfolio:**

| Asset | Type |
|---|---|
| SPY — SPDR S&P 500 | Equity |
| IEFA — iShares MSCI EAFE | Equity |
| EEMV — iShares MSCI Min Vol Emerging Markets | Equity |
| TSE:QCN — Mackenzie Canadian Equity Index | Equity |
| TSE:ZFL — BMO Long Federal Bond Index | Fixed Income |
| TSE:ZAG — BMO Aggregate Bond Index | Fixed Income |
| TSE:XSH — iShares Core Canadian Short Term Corp Bond | Fixed Income |
| GLD — Gold ETF | Equity |

**Efficient frontier** — 6,728 weight combinations evaluated (0–30% per asset, 10% increments). Optimal portfolio from the frontier:

| Asset | Weight |
|---|---|
| SPY | 30% |
| TSE:QCN | 30% |
| GLD | 30% |
| TSE:ZFL | 10% |

**Monte Carlo results** (500 simulations, 10-year horizon, $100,000 initial value):
- Annualized return: **8.85%** | Volatility: **11.64%** | Sharpe: **0.42**
- CVaR (95%): **−6.12%** | Max Drawdown: **−18.97%**
- Expected ending value: **$153,000 – $343,000** (68% confidence interval)

---

## Setup

```bash
pip install -r requirements.txt
```

`portfolio_classes.ipynb` requires a free [Alpha Vantage API key](https://www.alphavantage.co/support/#api-key). Save it to `keys/alphavantage.txt` — this path is excluded from version control by `.gitignore`.

---

## Key Methods

**Efficient Frontier (analytical, `portfolio_optimization.ipynb`)**

Long-only mean-variance optimization solved as a quadratic program. The efficient frontier is traced by sweeping target return $\bar{\mu}$ from the minimum to maximum achievable return across the asset universe.

**Monte Carlo — GBM (`portfolio_optimization.ipynb`)**

Asset log-returns follow:
$$\Delta \ln S_t = \left(\mu - \tfrac{1}{2}\,\text{diag}\,\Sigma\right)\Delta t + \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0,\,\Sigma\,\Delta t)$$

Correlated shocks are generated via Cholesky decomposition: $\Sigma \Delta t = LL^\top$, so $ZL^\top \sim \mathcal{N}(0, \Sigma \Delta t)$ for $Z$ i.i.d. standard normal.

**Efficient Frontier (brute-force, `portfolio_classes.ipynb`)**

Enumerates all valid discrete weight combinations and runs Monte Carlo simulations for each, then selects the Pareto-optimal set by return/risk. Slower than the analytical approach but does not require convexity assumptions.