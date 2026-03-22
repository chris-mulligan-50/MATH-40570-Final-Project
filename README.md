# Portfolio Optimization — MATH 40570 Final Project

Final project for MATH 40570 (Mathematical Methods in Financial Economics). The project applies mean-variance optimization theory to construct efficient portfolios from real ETF and mutual fund data, and validates the results with Monte Carlo simulation.

The full write-up is included as `MMFE_Project.pdf`.

---

## Notebooks

### `portfolio_optimization.ipynb`
The main project notebook. Covers the end-to-end analysis pipeline:

1. **Data ingestion** — pulls 15 years of monthly price data (2010–2025) for 12 ETFs and 11 mutual funds across 11 financial sectors (Technology, Financials, Energy, Healthcare, etc.) using `yfinance`
2. **Summary statistics** — computes total return, annualized return, volatility, and annualized volatility for each asset
3. **Mean-variance optimization** — constructs minimum-variance portfolios at target expected returns, both with and without short-selling constraints (long-only via `cvxpy`)
4. **Efficient frontier** — traces the set of optimal risk/return tradeoffs and plots the Capital Market Line to identify the maximum Sharpe ratio portfolio
5. **Monte Carlo simulation** — simulates thousands of random portfolios and portfolio growth paths using geometric Brownian motion with Cholesky-decomposed correlated shocks

### `portfolio_classes.ipynb`
A reusable OOP framework (`Security`, `Equity`, `Fixedincome`, `Portfolio` classes) for portfolio analysis built alongside the project. Uses the Alpha Vantage API for price data. Features:
- Per-security metrics: CVaR, max drawdown, Sharpe ratio
- Brute-force efficient frontier over discrete weight grids
- Monte Carlo portfolio path simulation with box plots and expected value plots

---

## Assets Covered

| Sector | ETF | Mutual Fund |
|---|---|---|
| Technology | VGT | VITAX |
| Financial Services | XLF | VFAIX |
| Consumer Cyclical | XLY | BPTRX |
| Consumer Defensive | XLP | VCSAX |
| Communication Services | VOX | VTCAX |
| Industrials | XLI | VINAX |
| Basic Materials | XLB | VMIAX |
| Energy | XLE | VENAX |
| Healthcare | XLV | VGHCX |
| Utilities | XLU | VUIAX |
| Real Estate | VNQ | VGSLX |

---

## Setup

```bash
pip install -r requirements.txt
```

To use `portfolio_classes.ipynb`, an [Alpha Vantage API key](https://www.alphavantage.co/support/#api-key) is required. Save it to `keys/alphavantage.txt` (this path is excluded from version control).

---

## Key Methods

**Efficient Frontier (analytical)**
Solves the quadratic program:

$$\min_w \; w^\top \Sigma w \quad \text{s.t.} \quad \mu^\top w = \bar{\mu}, \; \mathbf{1}^\top w = 1, \; w \geq 0$$

**Monte Carlo (GBM)**
Asset prices follow geometric Brownian motion. Correlated shocks are generated via Cholesky decomposition of the covariance matrix:

$$\Delta \ln S_t = \left(\mu - \tfrac{1}{2}\,\text{diag}\,\Sigma\right)\Delta t + \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0,\,\Sigma\,\Delta t)$$