# Statistical arbitrage, signals, and regime detection

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The source's statistical arbitrage design starts with a liquid universe, removes exposures to known common drivers, and looks for repeatable movement in the remaining spread or residual. Directional trend, event trading, and volatility forecasting use some of the same inputs but different targets and validation tests.

## Factor residuals and relative value

1. **Define the tradable universe.** Filter by average daily volume, spread, realistic order size, and, where shorting is needed, borrow. Use historically available membership, corporate-action adjustments, delisting returns, and consistent venue and timestamp conventions. For futures, document rolls and the mapping from a continuous signal series to the traded contracts.
2. **Estimate a factor model.** In $r_t=B_t f_t+\epsilon_t$, $B_t$ maps assets to market, sector, size, value, momentum, quality, low-volatility, or custom factors. Estimate factor covariance and specific risk using only the training information at each forecast date. A basic risk model is $B\Sigma_f B^\top+\Sigma_\epsilon$, with shrinkage or clustering where useful.
3. **Define a signed mispricing score.** A source candidate divides the difference between fast and slow EMAs of a fitted residual return by a robust rolling scale of that residual. Decide whether the object is a residual return, a cumulative residual, or a stationary price spread. Those objects can have different dynamics and holding periods. A positive score suggests a potential short for a reversion test and a potential long for a continuation test.
4. **Estimate where reversion is plausible.** Test liquidity and volume shocks, volatility and correlation regimes, earnings or news windows, and a microprice or order-flow confirmation. “Cheap for a reason” means the residual can reflect a fundamental change, bad price alignment, or an exposure missing from the model.
5. **Compare alpha families.** Broad cross-sectional residual reversion, within-industry relative value, cointegrated pairs or baskets, and event-conditioned reversion each need a separate benchmark. Persistent post-earnings drift is a distinct event hypothesis.
6. **Constrain the portfolio.** Dollar neutrality $\mathbf 1^\top x=0$ and beta or factor neutrality $B^\top x\approx0$ are different constraints. Use borrow and liquidity bounds, gross and turnover caps, and cost estimates. Factor crowding can appear even when the dollar book is flat. [Avellaneda and Lee, *Statistical Arbitrage in the U.S. Equities Market*](https://doi.org/10.1080/14697680903124632)

### Pair and basket methods

| Candidate | What it adds | Failure mode to test |
|---|---|---|
| Fixed hedge ratio and residual z-score | Transparent baseline with a training-period hedge ratio and an estimated long-run spread. | A persistent level shift, changing hedge ratio, or common factor exposure. |
| Clustering, PCA, or graph baskets | Restricts the search to economic peers or groups with related residual returns. Graph leads include minimum spanning trees and community detection. | A cluster chosen from the whole sample can leak future information. Correlated returns alone leave level-spread stability unresolved. |
| Kalman hedge ratio | Models $Y_t=\beta_t X_t+\epsilon_t$ with slowly evolving $\beta_t$. | The filter can absorb a structural break into the hedge ratio. Test its update speed and estimation error. |
| Cointegration, ECM, or VECM | Connects a level relationship with adjustment in the differenced series, optionally using lagged changes in multiple assets. | Changes in integration order, cointegrating vector, or adjustment coefficients. |

### ECM experiment from the competition notes

Check whether both price levels are compatible with $I(1)$ behaviour and whether a fitted combination of the levels is stationary. Similar return ranges are a separate observation. A bivariate error-correction candidate regresses $\Delta Y_t$ on the lagged level residual $Y_{t-1}-a-\beta X_{t-1}$ and lagged differences. Estimate the level relation in a rolling window of length $N$, assess it over a declared window or test sample $M$, and use an appropriate residual-based cointegration test such as Phillips–Ouliaris with its cointegration critical values. Johansen tests and a VECM are leads when a system formulation is warranted.

The source considers trading the dependent product against its estimated fair value when the relation is stable. It leaves the other product on a separate fair-value rule such as ATR/VWMP or momentum. During an Ink shock or weakening adjustment, the proposed action is to reduce or suspend the spread strategy and compare it with a defensive fallback. Try each economic orientation as a declared model variant. An insignificant error-correction coefficient in a short window is an uncertain indicator, so compare it with residual size, liquidity, and simple change-point rules. Update frequency matters for an implementation that cannot refit a multivariate model at every snapshot.

## Strategy and feature leads

| Family | Ideas from the source | Useful test boundary |
|---|---|---|
| Trend and momentum | EWMAC, time-series momentum, breakouts, RSI and other oscillators, SMA/LMA, sector rotation, PCA factor rotation. | Compare with simple time-series momentum over a fixed horizon. Separate trend capture from volatility scaling. |
| Mean reversion and extrema | Residual and spread reversion, rolling fair value, MinMax under the *ToE* label, and volatility bands. | The MinMax idea finds a trend, a near-zero first difference, then a sign of curvature. Causal estimates and a confirmation lag prevent a future extremum from becoming an earlier entry. |
| Microstructure and execution | L2 imbalance, expected fill price, effective spread, signed flow, bursts, depth replenishment, and short-term reversal. Candidate forecast horizons include 5 minutes to 1 hour. | Compare midpoint-only, depth-only, and executed-flow signals. Track feature timestamps and whether quotes survive until arrival. Depth visualisations are useful inspection tools. |
| Crypto and cross-venue | Cross-exchange and triangular price discrepancies, funding and carry, staking rates, large on-chain transfers, liquidation clusters, NVT-style usage measures, and DeFi/DEX price gaps. | Align timestamps, units, and executable legs. Include transfer, funding, inventory, and fees in the return calculation. |
| Macro and event | Earnings, releases, macro trend and sentiment, implied versus realised volatility, fundamental and low-volatility effects. | Use point-in-time release timestamps and a matched non-event baseline. Test drift, reversal, and volatility as different outcomes. |
| ML and forecasting | ARMA, LSTM, Transformers such as Informer, Autoformer and TimeXer, temporal graph attention, graph-spectral HAR, and FinGPT as a text or representation lead. | Use simple, comparable return or volatility baselines and a fixed out-of-sample split. The source's claims about relative model performance are research questions. |
| Sizing and numerical methods | Dynamic Kelly, rough volatility, Chebyshev interpolation, and RL execution. | Validate the forecast distribution, tail sensitivity, numerical approximation error, and execution assumptions as separate components. |

## Crash risk and state switching

The source records two distribution diagnostics. **NCSKEW** measures the negative of standardised return skewness, with finite-sample conventions to be specified. A large positive value indicates stronger left skew. **DUVOL** compares the volatility of down and up observations using a log ratio. State the return series, threshold for defining up and down, and whether the implementation uses standard deviations or variances.

A further fragment labelled **PTPI** claims that an indicator constructed from trading value warned of crashes earlier than variants based on trading volume or closing price. The file gives no reproducible indicator formula or study identifier. Keep that as a source lead. A practical design would define a crash label and prediction window, compare price, size, trading value, and book-shape features, and report false alarms, detection lead time, and conditional error in crash magnitude. Use blocks or chronological folds because neighbouring labels share future returns.

HMMs, volatility buckets, changes in spread dynamics, and simple correlation-spike rules are proposed state estimators. Separate the model's inferred state from the action taken in that state. For example, a high-volatility observation can trigger smaller size, wider entry thresholds, or a pause. Test each action against its own baseline. Execution controls include participation caps, TWAP or VWAP schedules, spread-aware passive orders when the signal permits waiting, and circuit breakers for abrupt changes in borrow or liquidity. See [portfolio construction](04-portfolio-and-time-scales.md) and [the experiment register](10-experiment-register.md).
