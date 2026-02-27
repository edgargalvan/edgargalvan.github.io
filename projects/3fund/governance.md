---
layout: page
title: Governance Framework
permalink: /projects/3fund/governance/
project: 3fund
audience: "Risk / compliance / reviewer"
---

# Pre-Committed Rules for the Researcher and the AI

Before writing any code, I wrote a governance document. Rules like: decide your parameters before running a test. Compare every variant against the simplest version that works. Run across multiple time windows, not just the one that looks best. If two reasonable parameter choices give meaningfully different results, the approach is fragile, kill it.

Over a couple of weekends I used an AI coding assistant to build and test 12+ variations of a trend-following strategy. I'm not a quant by training, so I wanted a process that couldn't be bent by whichever result looked good that afternoon.

The AI follows written rules literally. When a threshold isn't met, it kills the strategy without attachment to the work it took to get there.

The constraints on me worked differently. When momentum ranking won 6 of 7 time windows and looked compelling in a table, the governance doc said the full-period improvement was below my pre-committed threshold. Having written that number before seeing the results, there was nowhere to go.

Pre-committed rules work differently on humans than on AIs. For the AI, they're a stop condition. For me, they removed the option to rationalize.

## The setup

I structured the research in three phases. Phase 1: build a simple baseline (three ETFs, moving average filter, monthly check). Phase 2: test structural variations (how to filter, how to weight, what to do when an asset exits). Phase 3: try more sophisticated signals and see if they improve on whatever survived Phase 2.

The codebase is config-driven: parameters live in a YAML file, so adding a variant means editing config, not logic. Works with or without a coding agent.

The AI compressed the iteration cycle. A variant that would have taken an afternoon took about an hour, which is why I could test 12+ variants in a couple of weekends. But the research design was the governance doc's job: what to test, what counts as meaningful, when to stop.

## What the governance doc caught

I tested a gradual scaling approach: instead of binary in/out at the moving average, partial weight when price is near the threshold. The governance rules said: test two reasonable parameter values, if they diverge by more than a set amount, it's fragile. They diverged. Done.

I tested adding a managed futures ETF. The governance rules flagged that it was below its moving average 43% of all trading days. The binary filter is just incompatible with how that asset behaves. Killed.

Without pre-committed rules, I'd have kept going on both. The rules made the decisions boring, which is the point.

## What the governance doc didn't catch

The AI recommended I ship a version with inverse-volatility weighting, adjusting each fund's allocation based on recent volatility. It passed every governance criterion. Robust across time windows, good Sharpe.

But I wondered: is the volatility weighting actually doing anything? I ran the dumb test: equal weights (33/33/33) with the same filter.

Negligible difference. The moving average filter was doing all the work. The weighting scheme was dead code that happened to produce numbers.

The governance doc didn't catch this. It checks robustness, not whether components matter. That was just me being curious.

There was also a numerical bug: the code mixed two return conventions, so every metric was systematically off. The governance checks all passed because the error was consistent. Relative comparisons still held. I caught it when I replaced the custom backtesting code with quantstats and the numbers shifted.

The governance doc is good at one thing: preventing overfitting. It doesn't catch bugs or unnecessary complexity, and it's not a substitute for code review.

## Lessons

The AI is fast and consistent, but bad at stopping. It produces confident output regardless of whether the result is a 0.15 Sharpe improvement or a 0.015 one. It doesn't flag that a component isn't earning its keep. If you ask it to build a custom math function, it will. It'll look correct. It might be subtly wrong. Use a library for core math.

Build in a separate review step using a fresh AI session. The one that wrote the code never caught the return convention bug.

Write your evaluation criteria before you see any results, and start with the simplest version before adding anything. The governance doc is worth writing even if you keep it private. Deciding "what counts as meaningful improvement" before testing forces you to think about what you're actually looking for.

## Closing

The base strategy is Faber's (2007): three funds, equal weight, 200-day moving average, monthly check. Exit to cash for safety, redistribute for growth. Two minutes per month.

What this project adds: a systematic comparison showing that cash exit vs redistribution is the same strategy at different points on the risk/return frontier. Sharpe stays nearly constant; you're choosing drawdown tolerance, not finding alpha. Full results on the [strategy page](/projects/3fund/strategy/).

What I got out of it: a better understanding of how to use AI for research without fooling myself. The tool is fast and confident. Those are the same properties that make it dangerous.

---

Footnote: I also tested DMD/Koopman decomposition on correlation eigenvalue dynamics (Phase 3 of the plan). Every predictive mode I extracted was concurrent with well-known market indicators, not leading. Operators rearrange information; they don't create it. Probably obvious to practitioners, but I had to learn it firsthand.

