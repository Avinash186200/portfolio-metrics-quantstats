# Methodology

Short notes on the choices in the notebook. The README has the walkthrough; this file says why.

## Data

The trade log is synthetic and seeded (42): 6 strategies over 750 business days. Each strategy trades on a random
share of days (20% to 60%), 1 to 3 trades per day. A trade's P&L is

    300 x (load x common_factor + sqrt(1 - load^2) x own_noise) + drift

where `common_factor` is the same for all strategies on a given day, `load` is drawn per strategy between 0 and
0.7, and `drift` is about 20 per trade. Two strategies with high loads end up correlated on the days they both
trade. The numbers mean nothing about real strategies.

## Daily P&L

Trades are grouped by calendar day of the open time and summed per strategy. A day with no trade stays NaN in
this table on purpose, so the correlation step can tell "did not trade" from "made zero".

## Correlation on common days

For each pair of strategies, keep the days where both have a value, then take the Pearson correlation. The
number of those days is stored next to the figure. Two choices I rejected:

- Filling no-trade days with zero. Most days are then zero for at least one side, and every correlation is
  pulled toward zero.
- `DataFrame.corr()` on the whole table. It also uses pairwise common days, but it hides how many days each
  number rests on.

Treat any correlation over a few dozen days as rough. With 41 common days, the 95% range around a sample
correlation of 0.4 is about 0.10 to 0.63 (Fisher z interval).

## Returns and metrics

- Returns are daily P&L divided by a fixed capital base of 100,000, with no-trade days as zero return. The base
  sets the scale of drawdown and VaR in percent. Sharpe and Sortino do not change with it, because each divides a
  mean by a spread of the same returns.
- Metrics come from `quantstats.stats` with its defaults: 252 periods a year, risk-free rate 0, compounded
  returns for drawdown, parametric 95% VaR, and win rate over non-zero days.
- I check one or two numbers by hand when I use a library like this, so I know which definition it uses.

## Reports

`quantstats.reports.html` writes a tear sheet per strategy with no benchmark. The Excel file holds the pairwise
table, the correlation matrix and the metrics.

## Known limitations

- Synthetic trades; no costs or slippage.
- One fixed capital base for all strategies, whatever their real size.
- Days when no strategy traded at all are missing from the daily table rather than counted as zero. With six
  strategies this rarely happens, but with fewer it would shorten the return series.
