---
layout: page
title: Hierarchical Bayesian NFL Model
permalink: /projects/bayes-nfl/
project: bayes-nfl
---

**Status: work in progress.** I've used AI to modernize the code (PyMC v5, nflreadpy) and re-run the backtests. The results match what I had from an earlier version of this project, including the verdict on a time-varying variant I'd partly built years ago and killed once it was clear it didn't work. AI made it cheap to finish that buildout and land on the same conclusion on the full toolchain. A longer writeup in the shape of the [3Fund](/projects/3fund/) project is still to come. This page will be updated as that work lands.

## One-paragraph version

A hierarchical Bayesian model that predicts NFL game scores, in Python with PyMC v5. It is not built for raw predictive accuracy. A black-box ML model on play-by-play data would probably beat it there. The Bayesian framing is chosen for interpretability and calibrated uncertainty: every prediction decomposes into team attack, team defense, and home-field contributions, and the probability on each prediction is meaningful rather than a confidence score. On a four-season backtest (2022 through 2025) the base model averages 64% winner accuracy, which is in the neighborhood of the published Elo baselines, and on its higher-confidence picks (60% or more) it is correct 71% to 82% of the time depending on the season. Two variants were tried and rejected: adding rest-days and weather covariates did not meaningfully improve the base, and a time-varying variant that let team strengths drift week-to-week failed because it had too many parameters for the data.

## What the re-run actually told us

Re-running the backtests on the new stack reproduced the earlier findings. Most of what is interesting is in the posterior, not the accuracy column. Some notes from inspecting the 2024 fit:

**Team rankings are sensible.** The posterior means for the attack and defense parameters line up with what you'd expect from watching the 2024 season. Top attacks: DET, BUF, PHI, BAL, CIN, with Washington high, matching the Jayden Daniels rookie year. Worst attacks: the teams that changed coaches. Top defenses: PHI, LAC, DET, ARI, SEA, DEN. Worst defenses: CIN, DAL, ATL, NYJ, which were the actual defensive-collapse stories of the year. The model isn't finding anything new here. The value is that the decomposition a fan can sanity-check is the same decomposition the model uses to predict.

**Teams differentiate more on offense than on defense.** The model fits `sd_att = 0.172` and `sd_def = 0.059`. Team-to-team spread in offense is about three times the spread in defense. That is a structural claim the data made on its own, not something baked into the prior.

**The biggest misses are tail events, not systematic errors.** The largest confidence-wrong prediction of 2024 was Week 12 WAS vs DAL. The model said 78.6% WAS, with a predicted spread of about -11. Dallas won 34 to 26. The model got Washington's score about right (26 vs a predicted 28) and got Dallas's score wrong (34 vs a predicted 17, above the 95th percentile of the model's predictive distribution for DAL). The actual spread was at the 93rd percentile of the model's predicted spread distribution, which is what a calibrated model is supposed to get wrong a few percent of the time.

**The covariate rejection has a mechanism now.** The covariates variant fits three coefficients: rest-advantage, temperature, wind. All three posteriors have 90% credible intervals that straddle zero, and they barely move off the prior. The data doesn't resolve these effects. That is why the backtest Brier-score delta was essentially zero. Whatever signal was there got absorbed by the team attack and defense parameters.

## The tension the project is stuck on

Each parameter in the model has to be paid for by information, from data or from priors. The 4-season backtest sets a rough budget:

- 64 parameters (32 teams times attack and defense), paid for by roughly 480 regular-season games: well-identified.
- 512 parameters (32 teams times 8 weeks times attack and defense), paid for by roughly 120 games per rolling window: under-identified, and the posterior collapses back toward home-field advantage as the only thing that moves.

So "richer model" and "more data" are in tension. A richer simulation (drive-level, player-level, field-position-aware) needs more parameters to express the extra structure, and those parameters need more information per parameter. Pouring more rows into the same simple model doesn't help, which is exactly what the covariates result shows.

The path forward is structural rather than additive. Candidates: a hierarchical time prior (static team intercept with a tight random walk around it), a different likelihood (Skellam on the margin, or Dixon-Coles style correlation between home and away scores), or covariates that carry information the team parameters can't absorb (starting QB identity, specific injuries, weather past a sharp threshold).

## What's coming

- A backtest-results page with the full numbers, figures, and the time-varying post-mortem.
- A governance page on how the project is evaluated (calibration as the ranking metric, pre-registered variants, multi-season noise floors), adapted from the [3Fund governance doc](/projects/3fund/governance/).
