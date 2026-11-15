# Portfolio Metrics with QuantStats

[![CI](https://github.com/Avinash186200/portfolio-metrics-quantstats/actions/workflows/ci.yml/badge.svg)](https://github.com/Avinash186200/portfolio-metrics-quantstats/actions/workflows/ci.yml) ![Python](https://img.shields.io/badge/python-3.11-blue) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

When I had a few dozen trading strategies to choose from, I needed two things for each: how it performed on its own, and how much it overlapped with the others. This notebook does both from a trade log, with QuantStats doing the metric work.

## What problem this solves

When you run many strategies, two questions matter for each one: is it any good on its own, and does it just repeat what another strategy already does? This notebook answers both from a plain trade log (strategy, time, profit): standard risk and return metrics per strategy, and pairwise correlation computed fairly for strategies that do not trade every day.

## What you will learn

- How to roll trade-level P&L up to daily P&L per strategy.
- Why correlation between strategies that trade on different days needs care, and how to compute it on common days only.
- How to turn P&L into returns on a fixed capital base.
- What Sharpe, Sortino, max drawdown, VaR, average return and win rate measure, and how to get them from QuantStats.
- How to produce an HTML tear sheet per strategy and an Excel summary.

## How it works

There is one notebook, [`strategy_correlation_and_metrics.ipynb`](notebooks/strategy_correlation_and_metrics.ipynb). Read its cells in order:

1. **Setup and synthetic trades.** Six strategies over 750 business days from 2021-01-04. Each strategy trades on a random 20% to 60% of days, 1 to 3 trades a day. Every trade's P&L mixes a common market factor (with a different loading per strategy) and its own noise, plus a small positive drift. Teaches: what a trade log looks like and how shared exposure creates correlation.
2. **Daily P&L and pairwise correlation.** Trades are summed per day and strategy; a day without trades stays empty (NaN), not zero. For each pair, correlation uses only the days both traded, and the number of common days is reported next to it. On the default seed the top pair has a correlation of 0.44 over 67 common days, and some pairs share as few as 41 days. Teaches: correlation on common days, and why the day count must be shown.
3. **Metrics.** Daily P&L divided by 100,000 of capital gives daily returns (no trade = 0 return). QuantStats computes Sharpe, Sortino, max drawdown, VaR, average return and win rate per strategy. On the default seed the annualised Sharpe ratios range from 0.06 to 0.58. Teaches: what each metric means and how the capital base enters.
4. **Reports.** One QuantStats HTML tear sheet per strategy and an Excel file with the pairwise table, the correlation matrix and the metrics, all in `data/outputs/`.

## Key concepts

- **Trade log:** one row per trade with the strategy name, the time it opened and its profit.
- **Daily P&L:** the sum of a strategy's trade profits on each day.
- **Correlation on common days:** for two strategies, use only the days both traded. Filling no-trade days with zero pulls every figure toward zero.
- **Common days:** how many days a correlation is based on. A correlation over 40 days is far less certain than one over 400.
- **Capital base:** the fixed amount P&L is divided by to get returns. It sets the scale of drawdown and VaR in percent.
- **Sharpe ratio:** mean daily return over its standard deviation, times the square root of 252 (risk-free rate 0 here). Return per unit of risk.
- **Sortino ratio:** like Sharpe, but only downside moves count as risk.
- **Max drawdown:** the largest fall from a peak of the compounded equity curve.
- **Value at risk (VaR):** a daily loss that should only be exceeded on 5% of days. QuantStats estimates it from the mean and standard deviation of returns (the normal, variance-covariance method).
- **Win rate:** the share of non-zero days that were positive.
- **Tear sheet:** a one-page report of returns, drawdowns and metrics for one strategy.

## Repo layout

```
notebooks/
  strategy_correlation_and_metrics.ipynb   trade log -> daily P&L -> correlation and metrics -> reports
tests/
  test_notebooks.py   every notebook is valid, output-free, with no credentials or local paths
docs/
  README.md           index of the docs
  methodology.md      method notes and the reasons behind each choice
original/             the original research notebook (read-only record)
.github/workflows/ci.yml   runs the tests on every push
.env.example · requirements.txt · CHANGELOG.md · CONTRIBUTING.md · LICENSE
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m pytest -q
jupyter lab notebooks/
```

Outputs go to `data/outputs/` at the repo root (git-ignored), whether you start Jupyter from the repo root or from `notebooks/`; set `DATA_DIR` to change that. To use your own trades, load a table with `strategy, opened, profit` columns in place of the generator. The notebook is committed without outputs; run it to see the tables.

## What I learned

- Correlation between sparse strategies depends on how you treat days without trades. Filling them with zero pulls every correlation toward zero; using only common days is more honest, but then the number of common days has to be shown next to it.
- The capital base you divide P&L by changes drawdown and VaR in percentage terms, but not Sharpe. Fixing it up front avoids comparing numbers that were scaled differently.
- QuantStats saves a lot of time, but I still check one or two metrics by hand so I know what definition it is using.

## Status

Complete as a small utility notebook. The numbers it prints come from synthetic data and describe nothing real.

## Original research code

[`original/`](original/README.md) holds the code as I first wrote it for this project. It is kept so you can
read the first version alongside the maintained one; some of it contains the issues in the audit table above,
and comparing the two shows each fix.

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [CHANGELOG](CHANGELOG.md).
