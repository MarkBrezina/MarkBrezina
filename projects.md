## Projects *

### Featured: Concursus - Done
**2023 – Present** · [Repository](https://github.com/MarkBrezina/Concursus) · *Python (NumPy, SciPy, pandas, matplotlib); installable package `stochastics` v0.5*

A simulation library built on one idea: stochastic processes, population dynamics, finance and physics are all simulation problems, so they belong in one package with shared conventions. I designed the architecture, API, models and validation approach; large parts of the code were written with AI assistance, as the repository states. The package grew out of three earlier pieces of my own work (a 2023 stochastic-process class, galaxy-rotation and particle-swarm simulations, and an insurance model), kept in `legacy/`.

- **Baseline processes:** random walk, Wiener, arithmetic and geometric Brownian motion, Ornstein–Uhlenbeck, CIR, Poisson and compound Poisson, Brownian bridge, Merton jump-diffusion, Heston, fractional Brownian motion, Hawkes, regime switching and GARCH, plus general Euler–Maruyama and Milstein SDE solvers. Exact transition sampling is used wherever it exists, and every generator returns its theoretical mean, variance and terminal density so simulation can be checked against theory.
- **Finance:**
  - a price-time-priority limit order book;
  - an agent-based exchange simulating a full trading day with market makers, trend followers and noise traders;
  - market models of increasing complexity (single Heston path → correlated stream ensembles → internal-value streams with traders → news jumps);
  - draft trader profiles described by psychology (resilience, greed, patience) rather than strategy class;
  - an insurance module covering claim arrivals (Poisson, renewal, Hawkes), severity distributions, premium principles and Monte Carlo ruin probabilities.
- **Real-data benchmark:** scores each market model against real series (S&P 500, EUR/USD, one-minute BTC) on a fingerprint of twelve scale-free stylised facts — tail shape, volatility clustering, leverage effect, variance ratios. Errors are measured in units of the real series' own bootstrap sampling noise. Models are calibrated by simulated method of moments and checked out of sample on a holdout half. The benchmark also maps the trade-off between model fidelity and compute cost (Pareto frontier).
- **Population dynamics:** agent-based ecosystem (grass → rabbits → wolves), spatial SEIR epidemics with infection trees, rock–paper–scissors cyclic competition, wealth-exchange economies and Schelling segregation, with per-agent traits and conserved-quantity bookkeeping.
- **Physics:**
  - gravity and electromagnetism with velocity-Verlet integration, collisions, mergers and an exact energy ledger, so any energy drift is traced to the integrator or to a specific rule;
  - a "policy lab" in which the interaction laws themselves are designed and checked against Coulomb and Newtonian gravity as reference cases.
- **Quality:** around 175 unit tests across the subpackages, and eight showcase notebooks.

### TMRW — personal trading and analytics library
**2019 – Present** · [Repository](https://github.com/MarkBrezina/TMRW)

The research library behind Tomorrow Capital Research:
- data extraction from Yahoo Finance, Binance, Bloomberg, MSCI and Sustainalytics;
- technical and statistical indicators, Markov up/down tables and trend detection;
- portfolio metrics (VaR, CVaR, drawdown, annualised return);
- backtesting with configurable signals (mean reversion, trend following, pairs trading);
- Monte Carlo and scenario stress testing.

<!-- The "13% short-term return on real-time deployment" claim stays out. Add it back only with period, capital base, costs and whether gross or net. -->

### WunderConnectome — limit-order-book sequence prediction - Done
**2026** · [Repository](https://github.com/MarkBrezina/WunderConnectome)
Entry for the Wunder Fund Alpha Connectome competition: a GRU network trained on roughly 10,000 order-book sequences to predict two short-horizon targets. Finished in the top 20% of 5,700 active teams.
*Note: I honestly was very dissatisfied with my performance here, my concluding upload was simply the baseline model, with some additional features and extensive training. \
I had many great ideas while working and reflecting on this challenge, but none of them lead to anything conclusive. THis was my own fault, as I have not properly taken the time to understand how AI trading works.*


### Ctrl+Alt+DefeatTheMarket — IMC Prosperity unofficial guide
**2025 – Present** · [Repository](https://github.com/MarkBrezina/Ctrl-Alt-DefeatTheMarket) · *Python, pandas, NumPy, SciPy, networkx, matplotlib, plotly*

Research and trading framework for high-frequency market microstructure and systematic strategy design, built around the IMC Prosperity competition.
- **Microstructure:** order-book features (imbalance, order flow, VWAP, microprice, liquidity-weighted fair value); decomposition of price moves into order-flow, drift and mean-reversion components.
- **Strategies:** market-making, mean-reversion and drift-aware strategies under inventory and spread constraints; hybrid execution combining passive quoting with aggressive taking on mispricing; multi-signal blending to adjust quotes and inventory targets.
- **Testing:** tick-level backtester; robustness checks (detrending, z-scores, rolling regression forecasts); scenario tests across volatility, spread and liquidity regimes.
- **Execution and risk:** inventory-aware quoting with dynamic skew; trade classification, order placement and fill simulation.

### Algorithmic trading research
**2022 – Present** · [Repository](https://github.com/MarkBrezina/WTech)

Development and evaluation of trading algorithms — mean reversion, momentum and pattern detection — using stochastic processes, time-series analysis and statistical optimisation, with emphasis on reproducible testing across market conditions.
### Knowledgebase — mathematics and finance resources
**2023 – Present** · [Repository](https://github.com/MarkBrezina/Knowledgebase)

Curated study material in finance and mathematics — concept overviews, worked problems and references — intended to grow into a book and feeding the Tomorrow Library site.

<!-- Not yet listed:
- Market-States-Regimes, Foundational-Quantitative-Research, QPM, KB, BasicIT, Personal — add once each has a README that says what it shows.
- CONTROL (AI for trading) — still empty; leave off until there's code.
- GoldenDice and Galaksim are now superseded by Concursus (legacy/). Consider archiving GoldenDice or pointing its README at Concursus.
- ETL package (Yahoo, Alpha Vantage, Binance, Jupiter) — link pending.
- Event-study code — link pending. -->

---

