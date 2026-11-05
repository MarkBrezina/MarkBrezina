# Source map and editorial conventions

[Research index](README.md)

The primary source is the supplied `quant bucket.md`, with 3,231 numbered text lines. Ranges below refer to that copy. Some original lines join the end of one topic to the start of another. The grouped ranges are an index to the source, with chapter links for the resulting summaries.

| Original lines | Subject | Destination |
|---|---|---|
| 1–117 | Trader comparisons, fixed fair value, EMA, cached regression, inventory and state. | [Market making](notes/02-market-making-and-microstructure.md) |
| 118–269 | Factor risk, residual reversion, pairs, optimisation, execution and repeated stat-arb outline. | [Statistical arbitrage](notes/03-stat-arb-trend-and-regimes.md) |
| 270–318 | Crypto/equity universe, model and feature leads, stack and deployment plan. | [Workflow](notes/01-research-workflow.md) · [Signals](notes/03-stat-arb-trend-and-regimes.md) |
| 319–496 | £50 learning guide, trading mechanics, demo accounts and repository leads. | [Workflow](notes/01-research-workflow.md) |
| 497–561 | Danish RL and stochastic-control seminar shorthand. | [Reinforcement learning](notes/06-reinforcement-learning-and-control.md) |
| 562–617 | C5 and the hierarchy of firm, portfolios, strategies, and execution. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 618–670 | Multi-asset trend, horizon weighting, costs, and portfolio governors. | [Portfolio](notes/04-portfolio-and-time-scales.md) |
| 671–748 | C4 or Conjecture X, hybrid market dynamics and existence. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 749–863 | C3, execution schedules, cost, footprint and market memory. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 864–933 | C2, multi-asset central price and joint future paths. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 934–977 | C1, re-simulation after a model switch and legacy inventory. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 978–1026 | Repeated instrument, feature, strategy and architecture notes, joined to the C1 post. | [Workflow](notes/01-research-workflow.md) · [Signals](notes/03-stat-arb-trend-and-regimes.md) |
| 1027–1204 | Second copy of the £50 learning guide. | [Workflow](notes/01-research-workflow.md) |
| 1205–1267 | Repeated RL seminar notes. | [Reinforcement learning](notes/06-reinforcement-learning-and-control.md) |
| 1268–1749 | Joined RL credit-assignment fragment and fixed-spread market-making variants, including reported P&L. | [Reinforcement learning](notes/06-reinforcement-learning-and-control.md) · [Market making](notes/02-market-making-and-microstructure.md) |
| 1750–1839 | Coarse-graining and apparent changes in corrections across horizons. | [Portfolio](notes/04-portfolio-and-time-scales.md) |
| 1840–1990 | Velvetfruit voucher and Hydrogel candidates, then a making/taking recap. | [Voucher research](notes/08-option-voucher-research.md) · [Market making](notes/02-market-making-and-microstructure.md) |
| 1991–2002 | ToE, MinMax and TMRW fragments. | [Workflow](notes/01-research-workflow.md) · [Signals](notes/03-stat-arb-trend-and-regimes.md) |
| 2003–2028 | Volatility and CTA questions, multivariate subordination, illiquid bonds. | [Rates and volatility](notes/09-rates-and-volatility-questions.md) |
| 2029–2039 | The Final Problem, utility and variational analogies. | [Conjectures](notes/05-tomorrow-conjectures.md) |
| 2040–2388 | Latency taxonomy, four portfolio layers, allocation, architecture and validation. | [Portfolio](notes/04-portfolio-and-time-scales.md) · [Workflow](notes/01-research-workflow.md) |
| 2389–2413 | Discount curves, forwards and money-market account formulas. | [Rates and volatility](notes/09-rates-and-volatility-questions.md) |
| 2414–2432 | Six embedded images without descriptive captions. | [Image references](#image-references) |
| 2433–2531 | Competition ranking leads and reports on five algorithmic and manual rounds. | [Prosperity casebook](notes/07-prosperity-casebook.md) |
| 2532–2599 | Simulator state, competitor behaviour, and price, width, size and inventory modelling. | [Prosperity casebook](notes/07-prosperity-casebook.md) · [Market making](notes/02-market-making-and-microstructure.md) |
| 2600–2760 | Round recap and community discussion of Kelp, Resin and baselines. | [Prosperity casebook](notes/07-prosperity-casebook.md) |
| 2761–2817 | Market-making literature and methods. | [Market making](notes/02-market-making-and-microstructure.md) · [Reinforcement learning](notes/06-reinforcement-learning-and-control.md) |
| 2818–2987 | Tutorial discussion, ATR/VWMP variants, round briefs and contract limits. | [Prosperity casebook](notes/07-prosperity-casebook.md) |
| 2988–3020 | NCSKEW, DUVOL, PTPI, crash-risk and momentum fragments. | [Statistical arbitrage](notes/03-stat-arb-trend-and-regimes.md) |
| 3021–3139 | April 2025 work notes on the backtester, ECM, crash and inventory rules, and future rounds. | [Prosperity casebook](notes/07-prosperity-casebook.md) · [Signals](notes/03-stat-arb-trend-and-regimes.md) · [Experiment register](notes/10-experiment-register.md) |
| 3140–3143 | Three embedded images. | [Image references](#image-references) |
| 3144–3227 | Existing synopsis, voucher candidate ranking and connections across the conjectures. | [Index](README.md) · [Voucher research](notes/08-option-voucher-research.md) · [Conjectures](notes/05-tomorrow-conjectures.md) |
| 3228–3231 | Final embedded image and trailing space. | [Image references](#image-references) |

## Image references

The Markdown includes eleven GitHub-hosted images with generic alt text. The image bytes were unavailable during review. Their positions are preserved below. The neighbouring text was used for the summaries.

| Original lines | Adjacent context |
|---|---|
| 2414, 2415, 2419, 2420, 2423, 2424 | Between the rate fragments and the competition ranking notes. |
| 2560 | Near the simulator interface and market-model notes. |
| 3140, 3141, 3142 | After the round-planning notes and before the existing synopsis. |
| 3228 | After the final synthesis. |

## Editorial choices

- The index consolidates repeated guides, chat-style replies, and promotional drafts. Personal coordination details and meeting links are omitted from the public-facing notes.
- Simulator rules, reported scores, empirical observations, conjectures, and proposed tests have separate labels. Score comparisons across different settings require the original run manifests.
- The notes use positive inventory for long positions. An inventory-reducing reservation price lowers both quotes when long. The conflicting quote direction is recorded in the market-making chapter. Fixed 20-point width and inside-spread quoting are kept as separate configurations.
- The forward-rate chapter supplies the subtraction of one and the accrual convention in the simple forward formula. The crash-risk chapter uses negative standardised skewness for NCSKEW. The trend chapter describes the proposed softmax weights as exponential weighting, with a Bayesian interpretation requiring a specified probabilistic model.
- Project labels, unverified study names, and fragments with missing definitions remain reading or research leads. Protocols in the experiment register are proposed comparisons derived from those notes.
