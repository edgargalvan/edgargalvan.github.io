---
layout: page
title: Backtest Results
permalink: /projects/3fund/strategy/
project: 3fund
audience: "Investor / allocator"
---

# Faber's 200dma Strategy Still Works. Here's What Happens When You Don't Exit to Cash.

## The Punchline

Mebane Faber's 2007 paper "A Quantitative Approach to Tactical Asset Allocation" is the most downloaded paper on SSRN. The idea: check each asset against its 200-day moving average once a month. Below? Exit to cash. Simple, and it works.

I implemented it on SPY (equities), TLT (bonds), and GLD (gold). Faber's 5-asset version is included as a benchmark; the 3-fund outperforms on both Sharpe and CAGR.

Faber's paper treats exit to cash as a given. What if you redistribute freed capital to surviving assets instead? The choice is continuous: 0% (pure cash, Faber's version) to 100% (full renormalization). Risk-adjusted returns stay roughly constant. You're not picking a better strategy, just a different place on the risk curve.

| | Sharpe | CAGR | Max DD | $200K becomes |
|---|---|---|---|---|
| **Cash exit** | 1.04 | 7.9% | -14% | $918K |
| **Renormalize** | 0.96 | 11.2% | -23% | $1.61M |

![Redistribution sweep: Sharpe stays flat while CAGR and MaxDD both rise](figures/redistribution_sweep.png)

## The Base Strategy

Equal weight (33/33/33) across SPY, TLT, and GLD. On the last trading day of each month, check each fund against its 200-day moving average. Below? That fund's allocation is freed. What you do with it is the parameter.

Faber published the core idea in 2007 ([SSRN](https://ssrn.com/abstract=962461)). His version uses 5 asset classes and always exits to cash. The filter, monthly evaluation, and per-asset logic are all his.

### Does the filter help?

| Strategy | Sharpe | CAGR | Max Drawdown |
|---|---|---|---|
| SPY buy-and-hold | 0.60 | 10.4% | -55.2% |
| 60/40 | 0.70 | 7.9% | -35.0% |
| Faber 5-asset with 200dma | 0.66 | 5.1% | -15.8% |
| 60/40 with 200dma | 0.92 | 6.9% | -19.6% |
| EW SPY/TLT/GLD buy-and-hold | 0.95 | 9.2% | -23.1% |
| **EW SPY/TLT/GLD with 200dma** | **1.04** | **7.9%** | **-13.8%** |

The filter cuts max drawdown by 40% and improves Sharpe from 0.95 to 1.04. It costs ~1.3% CAGR. That's the price of trend following.

Adding 200dma to 60/40 gets to 0.92 Sharpe; replacing AGG with GLD gets to 1.04. Faber's 5-asset has the shallowest drawdown (-15.8%) but DBC and VNQ drag CAGR to 5.1%. Caveat: comparison only runs from ~2007 due to ETF availability; Faber tested from 1972 with index data.

![Equity curves: all strategies and benchmarks over 18 years](figures/full_period_equity_curves.png)

![Drawdowns: the filter dodges the GFC and limits 2022](figures/full_period_drawdowns.png)

## The Redistribution Dial

When a fund drops below its 200dma, its weight is freed:

- **0% (cash exit):** Freed weight goes to cash (SHY). Conservative.
- **100% (renormalize):** Freed weight splits among survivors. Stays ~100% invested. Aggressive.
- **In between:** X% to cash, (100-X)% to survivors.

| Redistribution % | Sharpe | CAGR | Max Drawdown |
|---|---|---|---|
| 0% (cash exit) | 1.04 | 7.9% | -13.8% |
| 25% | 1.03 | 8.7% | -15.2% |
| 50% | 1.01 | 9.6% | -17.4% |
| 75% | 0.99 | 10.4% | -19.6% |
| 100% (renormalize) | 0.96 | 11.2% | -22.6% |

![Redistribution scatter — risk-return tradeoff with benchmarks](figures/redistribution_scatter.png)

Sharpe stays within a ~0.08 band. CAGR climbs from 7.9% to 11.2% and max drawdown worsens from -13.8% to -22.6%, both monotonically. This pattern holds across every window tested (2007 full period, 2009 post-GFC, 2011 all-tickers, 2018 pre-COVID, 2021 post-COVID, 2009-2019 GFC-to-COVID).

The cash-vs-renormalize choice is a risk appetite dial, not an optimization. Sharpe stays flat. Pick a drawdown you can sit through.

Important caveat: this holds for SPY/TLT/GLD, which are meaningfully uncorrelated. With more correlated assets, redistribution would degrade Sharpe because you're concentrating into similar risks. At 100% redistribution, when 2 of 3 funds are below trend, you're 100% in a single asset. In 2022, that meant riding the last survivor down.

## Momentum Tilt

Instead of equal-weighting, rank survivors by trailing 12-month return (ex last month) and allocate 70/20/10.

| | EW-cash | Momentum-cash | Delta |
|---|---|---|---|
| CAGR | 7.9% | 9.4% | +1.5% |
| Max Drawdown | -13.8% | -16.9% | -3.1% |
| Sharpe | 1.04 | 0.98 | -0.06 |
| $200K becomes | $918K | $1.06M | +$142K |

Full-period, momentum trades 0.06 Sharpe for +1.5% CAGR. The cost is real; so is the pickup.

But look at the windows:

| Start Date | EW-cash Sharpe | Momentum Sharpe | Winner |
|---|---|---|---|
| Full Period (2007) | 1.04 | 0.98 | EW |
| Post-GFC (2009) | 1.011 | 1.014 | Momentum |
| All Tickers (2011) | 0.97 | 1.05 | Momentum |
| Pre-COVID (2018) | 0.98 | 1.09 | Momentum |
| Post-COVID (2021) | 0.92 | 1.28 | Momentum |

EW wins only the full period; momentum wins every window from 2009 forward, and the gap widens.

The cynic's read: GLD recency bias. Gold has gone on a historic tear, and momentum is just a way of being overweight gold. Mean-revert gold and momentum underperforms.

The bull's read: asset class divergence is increasing. When stocks, bonds, and gold take turns leading, momentum captures whichever one is working. That's the feature, not a bug.

![EW vs momentum on one scatter](figures/overlay_full_ew_vs_mom.png)

## Crisis Behavior

This is where the 200dma filter pays off. All four strategies were largely out of the market before the worst hits:

| Crisis | EW-cash | Momentum-cash | SPY B&H |
|---|---|---|---|
| Oct 2008 (GFC) | -0.6% | -0.6% | -16.5% |
| Mar 2020 (COVID) | +2.0% | +3.3% | -12.5% |
| 2022 Bear | -10.6% | -10.3% | -18.2% |
| 2025 Tariffs | +2.6% | +7.9% | -7.6% |

For the GFC, both strategies were already in cash before the crash. COVID was the cleanest: both were positive while SPY lost 12.5%. In the 2025 tariff shock, momentum was up 7.9% (concentrated in gold) while SPY dropped 7.6%.

The 2022 bear is the hardest test: stocks, bonds, and gold all declined together. Usually at least one rallies. Not in 2022. Even so, the filter exited TLT early as the bond bear took hold, then exited SPY mid-year. Filtered strategies lost about half what SPY did. The filter worked; cross-asset diversification temporarily failed.

![Crisis zoom — GFC and 2022](figures/crisis_scatter.png)

## When It Doesn't Work

Between the GFC and COVID (April 2009 to December 2019), the filter barely fires. All three assets spend most of the decade above their 200dma, and the strategy reduces to equal-weight buy-and-hold with occasional false exits that create cash drag.

| Portfolio | CAGR | Max Drawdown | Sharpe |
|---|---|---|---|
| SPY buy-and-hold | 16.0% | -19.4% | 1.06 |
| 60/40 (SPY/AGG) | 11.3% | -11.0% | 1.27 |
| All-Weather-Lite | 8.7% | -9.1% | 1.26 |
| **EW-cash (0%)** | **6.7%** | **-8.2%** | **1.00** |
| EW-renorm (100%) | 9.2% | -22.6% | 0.90 |

The strategy underperforms every benchmark on CAGR. At 0%, you're earning 6.7% while SPY does 16% and 60/40 does 11.3%. At 100%, you close the CAGR gap but take on -22.6% drawdown, worse than SPY.

![Calm-market cost: strategy underperforms benchmarks during 2009-2019](figures/redistribution_scatter_gfc_to_covid.png)

What 0% buys you: -8.2% max drawdown, shallower than even All-Weather-Lite. At 100%, you're closer to benchmark returns but at -22.6% drawdown.

This is the fundamental tradeoff. The filter protects in crises and drags in calm markets. Over 18 years that include both crises, risk-adjusted returns are strong. Exclude those crises, and they aren't.

### The 2022-2023 specifics

[TODO: The TLT story in detail.
- TLT fell below 200dma early 2022 during the rate hike cycle
- The filter correctly exited, avoiding the worst of the bond drawdown
- But TLT stayed below its 200dma for most of 2023 even as it partially recovered
- The filter missed the recovery
- Show numbers: what the filter saved in 2022 vs what it missed in 2023
- Net result was still positive but this is the price of trend following: you give up recovery upside to avoid drawdown.]

### Tax considerations

Monthly filter generates short-term capital gains. Turnover is low (a few round-trips per year) but exits within 12 months are taxed at ordinary income rate. Best in tax-advantaged accounts.

## What the Numbers Really Mean

The GFC and 2022 are the only two sustained multi-asset drawdowns in the 18-year sample. Everything else is a short crash (COVID) or a calm bull market where the filter never fires. Two out-of-sample crises of different types (credit-driven GFC, rate-driven 2022), and the strategy handled both. But two data points is two data points. In a calm market, this is just equal-weight buy-and-hold with a monthly check that never triggers.

![EW redistribution sweep across all 6 time windows — the pattern is stable](figures/overlay_ew_all_windows.png)

## Current Positioning (as of Feb 2026)

All three funds are above their 200-day moving averages. 12-1 momentum readings:

- GLD: +64.2%
- SPY: +15.6%
- TLT: +5.5%

| Strategy | SPY | TLT | GLD | Cash |
|---|---|---|---|---|
| EW-cash | 33% | 33% | 33% | 0% |
| EW-renorm | 33% | 33% | 33% | 0% |
| Momentum-cash | 20% | 10% | 70% | 0% |
| Momentum-renorm | 20% | 10% | 70% | 0% |

With all three above trend, cash exit vs renormalize disappears. Both are fully invested. The only difference is equal weight vs momentum-tilted toward gold.

## How to Choose

**EW-cash**: One rule, one action, equal weights. Nothing to overfit or second-guess. ~14% max drawdown.

**EW-renorm**: Same mechanics, but freed weight goes to survivors instead of cash. $200K becomes $1.61M instead of $918K over 18 years, but you'll sit through 22% drawdowns.

**Momentum-cash** if you believe asset class divergence will persist. Rank survivors by trailing momentum and tilt 70/20/10. Wins every window from 2009 forward, but that edge may not last.

**Momentum-renorm**: Maximum growth, maximum risk. 13.1% CAGR, $1.98M terminal value on $200K. You're running concentrated bets, fully invested, with -21% drawdowns.

## Try It Yourself

The full framework is open source: [trend-allocator](https://github.com/edgargalvan/trend-allocator)

Config-driven: you edit a YAML file, not Python code. You can:
- Set the redistribution parameter anywhere from 0% to 100%
- Test different fund universes
- Try different filters (only the 200dma is validated)
- Add momentum tilts or custom weighting
- Compare against benchmarks

The repo includes an AI governance document for honest testing, designed to work with or without a coding agent.

## The Rules

All four strategies share the same mechanics:

1. **Universe**: SPY, TLT, GLD
2. **Signal**: 200-day SMA, evaluated on the last trading day of each month
3. **Filter**: If a fund closes below its 200dma on month-end, it's "out" for the next month
4. **Weights**: Equal (33/33/33) or momentum-ranked (70/20/10 by 12-1 momentum)
5. **Exit mode**: Cash (reduce exposure) or renormalize (redistribute to survivors)
6. **Rebalance**: Monthly, on the last trading day

No machine learning, no regime detection. Just a moving average and a monthly check.

---

*Fixed-rule backtest on SPY/TLT/GLD daily data from January 2007 to February 2026. No in-sample optimization, no parameter fitting. All rules (200dma, equal weight, momentum lookback) are standard choices set before running. 2bps slippage per trade (negligible for monthly rebalancing of liquid ETFs at zero-commission brokers). $200,000 initial capital. Past performance does not guarantee future results. This is a personal research project, not investment advice.*

*Citation: Faber, Mebane T. "A Quantitative Approach to Tactical Asset Allocation." The Journal of Wealth Management, Vol. 9, No. 4 (2007). SSRN: https://ssrn.com/abstract=962461*

