# Portfolio construction and time scales

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The multiscale design gives each layer a different objective and target inventory. A short-lived market-making inventory and a multi-week investment can coexist if their targets, risk budgets, and execution are reconciled at portfolio level.

## Trading layers

| Layer | Illustrative holding horizon | Signals and tactics in the notes | Target and risk |
|---|---|---|---|
| **1. Microstructure and HFT** | Sub-second to seconds or minutes. | Stable or adaptive market making, basket and cross-venue discrepancies, order-flow imbalance, queue placement, and event-driven liquidity taking. | Inventory target close to flat. Tight live caps, shock and stale-data controls, and explicit treatment of end-of-session inventory. |
| **2. Short-term** | Minutes to a few days. | Intraday extremes, pairs and residual reversion, event volatility, news and sentiment, or delta-hedged options. Book signals can refine entry and exit. | Small directional or relative exposures. Size by risk and liquidity. Specify time exits, stops, and hedges. |
| **3. Medium-term** | Days to weeks. | Trend, longer-cycle reversion, sector rotation, macro-event volatility, and cross-asset relations. An example lead links commodity and currency moves. | Cross-asset risk, regime-sensitive sizing, and macro hedges where appropriate. Schedule larger orders using participation, TWAP or VWAP benchmarks. |
| **4. Strategic investing** | Weeks to years. | Value and quality, DCF and EV/EBITDA, macro cycles, yield curve and PMI, risk parity, and thematic holdings. | Strategic directional and factor exposures, lower turnover, portfolio stress tests, and review as fundamentals change. |

The source also sketches an infrastructure taxonomy. Microseconds to milliseconds are associated with queue competition and latency arbitrage. Milliseconds to seconds are associated with bursts, rapid events, and some hedging. Minutes to hours are associated with pairs, mean reversion, and execution schedules. Days to years support trend, carry, and volatility-risk-premium research. Months and beyond cover factor allocation and strategic capital. These are illustrative categories. A slow position decision can still generate fast child-order decisions. A near-flat market-making **target** still allows temporary inventory between fills.

## Multi-horizon trend model

**Signals.** For asset $i$ and horizon $h$, compare the fast and slow EMAs of log price or use the cumulative log return over $h$. Divide by a volatility estimate with matching units and horizon. Candidate slow-to-fast ratios in the notes are 2–8. Candidate horizons are 5–10, 20, 60, 120, and 250 trading days or their defined equivalents. Squash each score with $s_{i,h}=\tanh(m_{i,h}/c)$, with candidate $c$ around 2–4, then combine $\sum_h w_h s_{i,h}$. Bound the resulting positions separately.

**Weights.** Start with fixed, equal horizon weights. One adaptive proposal gives weight proportional to $\exp(\eta P_h)$, where $P_h$ is an after-cost performance score based only on outcomes available at the decision. Shrink the weights toward equal and account for correlated experts and turnover. This is an exponential weighting rule. A Bayesian interpretation would require a likelihood or a specified probabilistic performance model. A second proposal conditions weights on trend or chop features such as variance ratio, autocorrelation, an ADX-style indicator, Hurst proxy, realised-volatility slope, and drawdown slope. Test the hypothesis that reducing short-horizon weight in choppy markets improves after-cost performance.

**Sizing.** Convert signal strength to asset exposure using the inverse of a volatility estimate and a budget. Then rescale using aggregate predicted risk and liquidity. The notes use 10% annualised volatility as an example target, and suggest a positive drawdown measure $D_t$ with a risk multiplier $\mathrm{clip}(1-aD_t,\lambda_{\min},1)$. Compare any drawdown governor with a fixed budget. Check the resulting response after volatility shocks, including how the strategy regains exposure. Any rotation into proposed diversifiers such as rates, FX, or gold needs to be assessed under the same stress and correlation assumptions.

## Shared allocation and execution

For positions $x$ expressed in a consistent exposure unit, a common static benchmark is

$$
\max_x\ \mu^\top x-\frac{\gamma}{2}x^\top\Sigma x
-\kappa\,\mathrm{TC}(x-x_{\mathrm{prev}}).
$$

Expected returns, covariance, and transaction costs should correspond to the decision horizon and position units. Liquidity, concentration, gross exposure, asset-class budgets, turnover, and factor exposures can enter as constraints. With positive semidefinite covariance and convex costs and constraints, this form gives a tractable benchmark. Nonlinear market impact, integer quantities, and hard switching rules can require a different treatment.

| Interaction | What to measure |
|---|---|
| **Fast information into slow layers** | Do order-flow and liquidity signals improve short- and medium-term entry prices or only increase turnover? |
| **Slow targets into fast execution** | Does access to a slow layer's intended inventory help the maker or execution layer quote and unwind? Compare independent orders with netted orders. |
| **Strategies share capital and liquidity** | Aggregate exposures, capacity, margin, pending orders, and common factor risk. Measure how the same shock affects several strategies. |
| **Capital reallocates across horizons** | Compare fixed budgets, performance-based weighting, and market-state weights. Price transition trades and allow for correlated errors across layers. |
| **Common monitoring** | Review position and liquidity usage, P&L by strategy and net portfolio, drawdown, VaR and stress loss, realised versus predicted turnover and risk, and the health of each signal. |

Behavioural leads in the source include event reactions, herding, panic, and institutional flow such as ETF creations and redemptions. Model these through measured features with a known information time. The mean-field interpretation treats aggregated actions of a group as a driver of individual outcomes. That is a modelling assumption to test. Market-impact studies in this archive frame own flow as an execution effect and disruptive quoting as a market-integrity risk or simulator stressor.

## The scale-separation hypothesis

The source compares tick fluctuations, short-term reversals, and slower persistent deviations with coarse-graining in other systems. This is a research analogy. Averages can suppress some short-lived fluctuations, while sums and correlated observations have different scaling properties. Aggregation can also hide reversals or make estimated persistence sensitive to how bars are defined.

A proposed test uses the same underlying events to construct several resolutions. Measure the number of excursions per unit time, their magnitude in comparable volatility units, time to correction, and the chance of a continuation. Compare overlapping and non-overlapping sampling, and use null processes with known noise, trend, or mean reversion. Distinguish an apparent correction in the observed price from inventory changes caused by the trading policy. A useful outcome would say **at which horizons and conditions** each type of behaviour becomes distinguishable. See [Conjecture 5](05-tomorrow-conjectures.md) for the firm-wide allocation question.
