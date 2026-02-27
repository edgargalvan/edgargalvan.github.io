---
layout: page
title: Governance Framework
permalink: /projects/3fund/governance/
project: 3fund
audience: "Risk / compliance / reviewer"
---

# Pre-Committed Rules for the Researcher and the AI

I wrote a governance document before writing any code. Rules like: decide your
parameters before running a test. Compare every variant against the simplest
version that works. Run across multiple time windows, not just the one that
looks best. If two reasonable parameter choices give meaningfully different
results, the approach is fragile, kill it.

Over a couple of weekends I built a testing platform and used an AI coding
assistant to build and test variations of the Faber trend-following strategy.
The AI follows written rules literally. When a threshold isn't met, it kills the
strategy without attachment to the work it took to get there.

I wrote the strategy to constrain the AI, but found that it constrained me as
well, just differently. For example, when a strategy won 6 of 7 time windows
and looked compelling in a table, the governance doc said the full-period
improvement was below my pre-committed threshold. Since I wrote that threshold
before seeing the results, the strategy was DOA.

Pre-committed rules work differently on humans than on AIs. For the AI, they're
a stop condition. For me, they removed the temptation to rationalize.

## The setup

I structured the research in three phases. Phase 1: build a simple baseline
(three ETFs, moving average filter, monthly check). Phase 2: test structural
variations (how to filter, how to weight, what to do when an asset exits). Phase
3: try more sophisticated signals and see if they improve on whatever survived
Phase 2.

The codebase is config-driven: parameters live in a YAML file, so adding a
variant means editing config rather than logic and works with or without a coding
agent.

But, the AI compressed the iteration cycle. A variant that would have taken an
afternoon took about an hour, which is why I could cycle through my test plan in
a couple of weekends. The research design was the governance doc's job: what to
test, what counts as meaningful, when to stop.

## What the governance doc caught

I tested a gradual scaling approach: instead of binary in/out at the moving
average, partial weight when price is near the threshold. The governance rules
said: test two reasonable parameter values, if they diverge by more than a set
amount, it's fragile: drop it.

Without pre-committed rules, I might have been tempted to keep going on both. The
rules made the decisions boring, which is the point.

## What the governance doc didn't catch

Inverse-volatility weighting was set as the baseline from the start. It's a
legitimate technique: risk parity uses it to equalize risk contribution across
assets. The AI (following governance) recommended I "ship" the strategy since it
passed every governance criterion. Robust across time windows, good Sharpe.

I ran the dumb test: equal weights (33/33/33) with the same filter and found
negligible difference. With three assets of already-similar volatility,
inverse-vol shifts weights to something like 30/35/35, and a similar Sharpe to
equal weights. The moving average filter was doing all the work. The weighting
scheme was dead code that happened to produce numbers.

The governance framework only evaluates changes against a baseline. Inverse
volatility weighting was never run as an alternative, so there was nothing to
compare. The framework kept me from making things worse. It never asked whether
simpler (equal weight) would do.

The governance doc is helpful for preventing overfitting. It doesn't catch bugs,
it's not a substitute for code review or common sense.

## Lessons

Write your evaluation criteria before you see any results, and start with the
simplest version before adding anything. The governance doc is worth writing
even if you keep it private. Deciding "what counts as meaningful improvement"
before testing forces you to think about what you're actually looking for.

## Closing

The base strategy is Faber's (2007): three funds, equal weight, 200-day moving
average, monthly check. Exit to cash for safety, redistribute for growth. Two
minutes per month.

What this project adds: a systematic comparison showing that cash exit vs
redistribution is the same strategy at different points on the risk/return
frontier. Sharpe stays nearly constant; you're choosing drawdown tolerance, not
finding alpha. Full results on the [strategy page](/projects/3fund/strategy/).

What I got out of it: a better understanding of how to use AI for research
without fooling myself. The tool is fast and confident. Those are the same
properties that make it dangerous.

---

Footnote: I also tested DMD/Koopman decomposition on correlation eigenvalue
dynamics (Phase 3 of the plan). Every predictive mode I extracted was concurrent
with well-known market indicators, not leading. Operators rearrange information;
they don't create it. Probably obvious to practitioners, but I wanted to see it
first hand.

