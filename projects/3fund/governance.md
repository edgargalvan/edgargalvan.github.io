---
layout: page
title: AI Governance
permalink: /projects/3fund/governance/
project: 3fund
audience: "Risk / compliance / reviewer"
---

# I Used AI to Test 12 Trading Strategies Over a Weekend. The Best One Was From 2007.

## Opening

Over a couple of weekends I used Claude to build and test 12+ variations of a trend-following strategy. I'm not a quant — my background is in [your field]. I wanted to see how far I could get using AI as a research tool in a domain I was learning.

The short version: the strategy that survived all my testing was Mebane Faber's Tactical Asset Allocation framework, published in 2007. Three funds, a 200-day moving average, check once a month. I couldn't beat it. But I did find something Faber's paper doesn't cover — how the choice of what to do with freed capital lets you slide along the risk/return frontier in a way that matters.

The longer version is about the process, because I think the process is more transferable than the strategy.

## The setup

I wrote a governance document before writing any code. It had rules like: decide your parameters before running a test. Compare every variant against the simplest version that works. Run across multiple time windows, not just the one that looks best. If two reasonable parameter choices give meaningfully different results, the approach is fragile — kill it.

I'm not a quant, so I didn't have the intuition to know when I was overfitting. The governance doc was my substitute for that intuition. I put it where Claude could read it, and Claude applied the rules literally every time. That turned out to be useful in a specific way I'll get to.

I set up a phased plan. Phase 1: build a simple baseline (three ETFs, moving average filter, monthly check). Phase 2: test structural variations (how to filter, how to weight, what to do when an asset exits). Phase 3: try more sophisticated signals and see if they improve on whatever survived Phase 2.

Claude made this practical. Implementing a variant, running a backtest, generating comparison tables across five time windows — I could do a full test cycle in about an hour. That speed is why I could test 12+ variants in a couple of weekends.

## What the governance doc caught

I tested a gradual scaling approach — instead of binary in/out at the moving average, partial weight when price is near the threshold. The governance rules said: test two reasonable parameter values, if they diverge by more than a set amount, it's fragile. They diverged. Done. Without that rule I would have picked whichever width looked better and kept iterating.

I tested adding a managed futures ETF. The governance rules flagged that it was below its moving average 43% of all trading days — the binary filter is just incompatible with how that asset behaves. Killed.

Momentum ranking among survivors won 6 of 7 time windows. Looks great in a table. But the full-period improvement was below the threshold I'd set for "meaningful." This was a case where the governance doc stopped me specifically — I wanted to keep the momentum version because the recent windows looked so good. Having a pre-committed threshold to point at made the decision easier.

Each of these was a place where, without pre-committed rules, I'd have kept going. The rules made the decisions boring, which is the point.

## What the governance doc didn't catch

Claude recommended I ship a version with inverse-volatility weighting — adjusting each fund's allocation based on recent volatility. It passed every governance criterion. Robust across windows, good Sharpe, good drawdowns.

But I wondered: is the volatility weighting actually doing anything? It was the most complex component. I ran the dumb test — just use equal weights (33/33/33) with the same filter.

Negligible difference. The moving average filter was doing all the work. The weighting scheme was dead code that happened to produce numbers.

The governance doc didn't catch this because it checks whether a strategy is robust, not whether its components matter. That was just me being curious.

There was also a numerical bug in the backtesting code. Claude mixed two return conventions — computed one way in one file, compounded with a formula that expects the other. Every metric was systematically off. The governance checks all passed because the error was consistent — relative comparisons still held, thresholds were still met. I eventually got fed up with the accumulation of small numerical issues in the custom backtesting code and replaced it with quantstats, a standard library. That's when the numbers shifted and I traced the root cause.

The governance doc is good at one thing: preventing overfitting and over-optimization. It doesn't catch bugs. It doesn't catch unnecessary complexity. It doesn't replace code review or testing.

## What I actually found

The base strategy is Faber's: hold funds above their 200-day moving average, exit to cash when below, check monthly. I confirmed it works on SPY/TLT/GLD over 18 years of data.

The part Faber's paper doesn't explore in depth: what happens to the freed capital when an asset exits.

Exit to cash (Faber's approach): ~7% CAGR, ~12% max drawdown. Your portfolio shrinks during crises. Safe.

Redistribute to survivors: ~10% CAGR, ~22% max drawdown. The capital stays invested in whatever's still above trend. Nearly doubles terminal wealth over 18 years. But when two of three funds are below trend, you're 100% in a single asset. During 2022, that meant riding the last fund down before it broke too.

Momentum tilt on top of either: rank survivors by recent performance, overweight the strongest. Another ~1-2% CAGR, modestly deeper drawdowns.

The Sharpe ratios across all variants are nearly identical. You're not getting better risk-adjusted returns by redistributing or tilting — you're choosing where to sit on the CAGR vs drawdown frontier. The governance framework helped me see this by forcing multi-metric evaluation instead of fixating on one number.

## What I learned about using AI for research

Claude is good at: implementing things fast, generating comparison tables, running the same evaluation framework on the twentieth variant without getting sloppy. The speed is the real value — I tested more ideas in two weekends than I could have in months manually.

Claude is bad at: telling you when to stop. Flagging that a component isn't earning its keep. Catching its own numerical errors. It produces confident output regardless of whether the result is a 0.15 Sharpe improvement or a 0.015 one. If you ask it to build a custom Sharpe ratio function, it will, and it'll look correct, and it might be subtly wrong. Use a library.

The governance doc worked because Claude follows written instructions literally. It reads the doc, applies the rules, doesn't get creative about when to bend them. That's useful when the rules are "kill this if the threshold isn't met" — because I might not have killed it myself.

One thing I'd do differently: I'd build in a separate review step using a different Claude conversation. The instance that wrote the code never caught the return convention bug. A fresh instance reviewing the finished code is where issues surface. The tool is better at reviewing things it didn't write than at catching its own mistakes.

## Practical advice if you're trying something similar

Write your evaluation criteria before you see any results. You will talk yourself into bad ideas without pre-committed criteria.

Start with the simplest version. Test it before adding anything. I spent time on volatility weighting and momentum ranking that turned out to be noise on top of a moving average.

Use libraries for core math. The custom backtesting code had bugs that hid for weeks because the output looked plausible. quantstats computes Sharpe ratios correctly because thousands of people have already found the edge cases.

Don't let the tool that built something evaluate that same thing. Separate review, different instance, fresh eyes.

The governance doc is worth writing even if you never share it. The act of deciding "what counts as meaningful improvement" before testing forces you to think about what you're actually looking for. And if you're using AI, putting those rules where it can read them means they get applied consistently.

## Closing

The strategy: three funds, equal weight, 200-day moving average, monthly check. Exit to cash for safety, redistribute for growth, your choice. Two minutes per month. Faber published the core idea in 2007.

What I added: a systematic comparison of cash vs redistribution showing they're the same strategy at different points on the risk/return frontier, and a governance framework that kept me from overcomplicating it.

What I actually got out of it: a better understanding of how to use AI for research without fooling myself. The tool is fast and confident. Those are the same properties that make it dangerous.

Repo link. Strategy writeup link.

---

Footnote: I also tested DMD/Koopman decomposition on correlation eigenvalue dynamics (Phase 3 of the plan). Every predictive mode I extracted was concurrent with well-known market indicators, not leading. Operators rearrange information — they don't create it. Probably obvious to practitioners, but I had to learn it firsthand.

