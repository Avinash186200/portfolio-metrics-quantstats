# Portfolio Metrics with QuantStats

[![CI](https://github.com/Avinash186200/portfolio-metrics-quantstats/actions/workflows/ci.yml/badge.svg)](https://github.com/Avinash186200/portfolio-metrics-quantstats/actions/workflows/ci.yml) ![Python](https://img.shields.io/badge/python-3.11-blue) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

When I had a few dozen trading strategies to choose from, I needed two things for each: how it performed on its own, and how much it overlapped with the others. This notebook does both from a trade log, with QuantStats doing the metric work.

## What it does

1. Rolls trade-level P&L up to daily P&L per strategy.
2. Computes pairwise correlation only on days both strategies traded, and reports the number of common days next to each figure.
3. Turns P&L into returns on a fixed capital base and computes Sharpe, Sortino, max drawdown, VaR, average return and win rate.
4. Writes one HTML tear sheet per strategy and an Excel summary.

The trade log is synthetic (strategies with a tunable shared factor), so it runs without any data.

## Structure

```
notebooks/
  strategy_correlation_and_metrics.ipynb
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab notebooks/
```

Outputs go to `data/outputs/` (git-ignored); set `DATA_DIR` to change that. To use your own trades, load a table with `strategy, opened, profit` columns in place of the generator.

## What I learned

- Correlation between sparse strategies depends on how you treat days without trades. Filling them with zero pulls every correlation toward zero; using only common days is more honest, but then the number of common days has to be shown next to it.
- The capital base you divide P&L by changes drawdown and VaR in percentage terms, but not Sharpe. Fixing it up front avoids comparing numbers that were scaled differently.
- QuantStats saves a lot of time, but I still check one or two metrics by hand so I know what definition it is using.

## Status

Complete as a small utility notebook. The numbers it prints come from synthetic data and describe nothing real.

## Original research code

[`original/`](original/README.md) holds the code as I first wrote it for this work, sanitised and nothing else:
client, company and person names replaced, credentials and local paths removed, data files and notebook outputs
dropped. It is kept so the real work can be read alongside the maintained version. Some of it contains the issues
described above; comparing the two shows each fix.

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [CHANGELOG](CHANGELOG.md).
