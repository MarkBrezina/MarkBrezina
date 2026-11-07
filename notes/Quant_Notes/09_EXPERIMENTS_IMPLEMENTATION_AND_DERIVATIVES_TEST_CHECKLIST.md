# 09 — Implementation Roadmap: Strategy Mechanisms, Derivatives, Macro and Event-Time Experiments

> This is the implementation companion to [Sheet 08](08_STRATEGY_TAXONOMY_OPTIONS_COMMODITIES_MACRO_AND_INTRINSIC_TIME.md) and the original [research roadmap](05_RESEARCH_REGISTER_ROADMAP_AND_MASTER_PROMPT.md). **All experiments are proposals.** Reproduce figures or performance numbers only from actual auditable datasets.

## 1. Scope and recommended separation

Use three interchangeable research layers and keep them *distinct*:

1. **Signals / opportunity estimates** — alpha expected return, mean reversion, carry, volatility mispricing, event-driven changes, order-book first-passage probabilities.
2. **State and uncertainty** — market regime, volatility and jump risk, uncertainty intervals, liquidity, information flow, cross-asset graph, structural breaks, macro vintages.
3. **Decision / execution / attribution** — inventory targets, instruments, options Greeks, margin, entry/exit, placement, impact, capital usage, realized P&L.

Never force a signal model to also handle the constraints and economics of an option portfolio, and never label an apparently precise estimate a trading edge until a self-financing portfolio with costs passes validation.

## 2. A practical strategy interface

A strategy's minimal output should be a **forecast and confidence/uncertainty**, not a brokerage order:

```python
@dataclass(frozen=True)
class Forecast:
    timestamp: pd.Timestamp
    asset: str
    horizon_bars: int
    expected_return: float       # explicit units and horizon
    uncertainty: float           # defined forecast dispersion / CI
    mechanism: str              # persistence, convergence, carry, etc.
    strategy_id: str
    state_version: str
    data_asof: pd.Timestamp

@dataclass(frozen=True)
class Target:
    asset: str
    target_notional: float
    max_slippage_bps: float
    max_holding_bars: int
    order_policy: str            # passive, aggressive, hybrid

# forecast -> risk-budget and optimizer -> target -> simulator/broker
```

For derivative instruments also include standardized contract identifier, expiry, underlying, strike, right, exercise style, multiplier, quote currency, initial/margin requirements, and theoretical Greeks at the *as-of* time.

## 3. Precommit research splits and contamination controls

- Preserve an immutable data manifest (venue, symbol, UTC, timestamps, adjustments, archived revisions, source, hash).
- Separate *discovery*, *parameter selection*, *locked confirmation*, and *fresh paper/live monitoring*.
- Features must be available at decision time; release timestamps and macro revisions cannot be retroactively applied.
- A filled limit order requires compatible matching/queue rules. Maker fee=0 is not fill probability=100% and not cost=0.
- Split by **calendar time**, not random observations. Purge overlapping labels and use embargo where forecasts share future outcomes.
- Control the **researcher degrees of freedom**: keep an experiment index covering failed as well as successful trials.
- Use confidence intervals, bootstrap appropriate for serial dependence, multiple-testing discounts, and sensitivity to trade-cost assumptions.

## 4. First implementation queue

| Phase | Work package | Output | Stop/continue criterion |
|---|---|---|---|
| P0 | Rebuild standardized backtest and P&L accounting | Trade ledger, equity, exposure, drawdown, fees | Unit tests pass; accounting invariant to reporting method |
| P0 | OHLCV data integrity + execution assumptions | Data contract, no-look-ahead checks | Reconciliation under venue/time conventions |
| P0 | Simple momentum/reversion/carry signals | Baseline strategy library | Correct positions and causal features |
| P1 | Directional-change event engine | Up/down event, extreme, overshoot, waiting time | Stable under streaming replay |
| P1 | Residual mean-reversion and macro fair-value | Causal factor fit, interval, residual | Held-out stable spread conditional behavior |
| P1 | Spot–futures basis and carry | Forward curve, roll-aware P&L | Survives realistic funding/roll/spread costs |
| P1 | Regime-conditioned position sizing | Linear, tanh, overextension-shaped reaction functions | Outperforms fixed sizing on locked risk/utility measure |
| P1 | Cross-asset graph/positioning features | Node centrality, flow z-scores | Incremental held-out value over simple factors |
| P2 | Simple option payoff/risk engine | Contract profit/Greeks/stress; parity checks | Replication tests and arbitrage constraints pass |
| P2 | Option IV/RV relative value | Hedging ledger and IV/RV signals | After fees, margin and crash scenarios acceptable |
| P2 | QNM or richer implied-vol surface | Arbitrage-free fit / smile stress | Better out-of-sample fit and stable Greeks |
| P3 | Structural PDE and latent dealer/fund models | Observable map, bounds, identifiability | Demonstrably identifiable or flagged underdetermined |

## 5. Mandatory baselines for each hypothesis

**Trend/momentum:** buy-and-hold; volatility-scaled time-series momentum; random/sign-flipped entries; matched-horizon average return.

**Mean reversion:** stationary AR(1)/OU; naive rolling Z-score; no-trade and random-entries; transaction-cost-aware random walk benchmark.

**Carry:** maturity-normalized curve slope; simple funded roll return; momentum-only; cross-sectional carry ranking; control for macro beta.

**Directional change:** same signal on equally spaced bars; realized-vol threshold; simple breakout/reversal; threshold sensitivity and event count controls.

**Positioning:** last-position persistence; volatility/price momentum; no look-ahead COT time; crowding forecasts adjusted for non-public data availability.

**Options:** no hedge vs delta hedge vs discrete/continuous simulated hedge; naive theoretical and market-implied vol; funding/collateral; liquidity; stress jumps.

**Nonlinear sizing:** linear clipped; tanh; flat/capped; vol target; risk-budgeted; same alpha signal held fixed.

## 6. Candidate output metrics

- **Prediction:** IC, correlation, Brier/log loss for probabilistic events, calibration, stable risk forecast, confidence interval coverage.
- **Trades:** net bps/trade, realized spread, entry slippage, fill rate by order placement, adverse selection, gross versus net, exposure hours.
- **Portfolio:** realized net return and annualization, drawdown, expected shortfall, Calmar/Sortino/Sharpe, liquidity-at-risk, funding, worst-day/week, factor exposures.
- **Statistics:** sample size and effective sample size, event overlap, uncertainty bars, parameter stability, subperiod/asset tests, multiple trials tested.
- **Options:** vega/gamma/delta/theta paths, margin use, assignment, max scenario loss, spread/vol-surface sensitivity, tail sensitivity, realized-versus-implied mismatch.
- **Macro:** revisions, release-time errors, covariate instability, cross-country changes, identification validity.

## 7. Recipe: directional-change event tests

1. Choose `δ` from a **predefined grid**, optionally normalized to a volatility estimate known before the event.
2. Reconstruct historical events in one streaming pass; store exact **confirmation bar/time**.
3. Test next `h` physical-time returns and next `k` intrinsic-time events separately.
4. Condition by spread cost, volatility, shock signs and event intensity.
5. Distinguish *observed after event* from *ex post identified extreme*.
6. Estimate net strategy returns only after filling/latency assumptions.

### Common false positive

An event annotated on an extremum when the algorithm did not know it was an extremum is a perfect example of **future information contamination**.

## 8. Recipe: carry/macro research

1. Align front/back futures contracts, delivery dates, multiplier, settlement and roll convention.
2. Compute common units (log forward basis per annum, spot vs future where spot is observable, collateral/funding).
3. Estimate **ex ante** carry, not an already realized return.
4. Add seasonality, inventory, rates, USD, energy equity and other covariates.
5. Test carry-only; momentum-only; carry-plus-momentum; nonlinear sizing; regime switch.
6. Compare **fully realized rolling futures portfolio P&L**; a calendar spread can mark away from convergence before expiry.
7. Report basis sensitivity, delivery or negative-price anomalies, and limitations of spot data.

## 9. Recipe: option portfolio construction from target payoff slope

Given strikes `K_j` and desired slope of `G(S_T)` in each segment:

1. Choose base slope `b` (requires underlying if nonzero below first strike).
2. For each strike compute `Δb_j=after_slope-before_slope`.
3. Hold `Δb_j` calls, respecting unit contract multipliers.
4. Add required intercept with cash/discount bond.
5. Verify payoff for all interval boundaries and random prices.
6. Convert payoff to profit by subtracting each initial premium/debit and funding cost.
7. Test margin and downside constraints; distinguish European expiry payoff from interim assignment/liquidity risk.

**Unit tests:** long/short parity; monotonicity and convexity of single-call prices across strike; exactly replicated synthetic-forward payoff; correct signs for put spreads, butterflies, condors, straddles; finite maximum loss for any **defined-risk** combination.

## 10. Recipe: volatility-risk-premium strategy tests

1. Quote-filter option market data to liquid contracts and realistic executable bid/ask at each time.
2. Estimate expected future realized variance using ONLY lagged data (HAR, EWMA, GARCH, stochastic model, ML calibration).
3. Construct IV forecast gap for matching **expiry and annualization**; ensure option pricing convention consistent.
4. Separate naked position from discretely hedged and (if feasible) continuously hedged theoretical comparison.
5. Simulate gamma/vega shocks, volatility surface changes, gap risk, jump risk and spread widening.
6. Calculate P&L attribution and funding/margin; report worst tails and net expectancy.
7. Evaluate hedge-frequency sensitivity; a frequent hedge pays spread repeatedly.
8. Roll sample through structural eras with unchanged strategy construction, accounting for changes in options availability.

A positive historical average short-volatility premium is compensation for **bearing rare convex losses**; it is not an arbitrage.

## 11. Recipe: test a fund-position inference model

Observed public positioning `y_t` may be modeled as `y_t = Σ_l w_l r_{t-l} + ε_t`. Fit ridge/elastic-net against the same lags and risk-scaled returns; compare a shallow neural network only after the linear model. Use correctly delayed reports and instrument adjustments. Check stability of coefficients, out-of-sample direction changes, and strategy attribution. Never assert the residual is exclusively discretionary trading.

## 12. Portfolio interpretation and an explicit decision objective

A unified candidate target is:

\[
\max_{w_t}\quad w_t^\top\hat\mu_t-
\frac{\lambda_t}{2}w_t^\top\hat\Sigma_tw_t-
C(w_t-w_{t-1})-
\psi\,\mathrm{TailRisk}(w_t),
\]

subject to market, margin, inventory, leverage and execution constraints. Forecasts and risk parameters must be **as-of** values. Calibrate cost to the actual fill model; treat these as proposed optimization mechanics, not a guaranteed solution.

For multi-stage decisions, Bellman recursion is an alternative or extension; fit state transitions with careful uncertainty bounds and constrained action spaces. Use a static robust optimization baseline before sophisticated RL.

## 13. Source provenance, research status and CSV register

Every new row added in `research_register.csv` is a **hypothesis to investigate**, not a completed experiment. Log reproducible hashes, code commit IDs, outcome measures, actual costs, and why a hypothesis passed or failed. A strategy lacking reliable options chains, position reports, or futures curves should remain `DATA_BLOCKED` rather than rely on invented availability.

## 14. Completion criteria

A research item can be marked `CONFIRMED` only when:

- Data origin and as-of chronology are documented.
- A real test on a locked sample exists and an independent baseline was included.
- The exact instrument, financing, and execution path is modelled.
- Statistical uncertainty and alternate explanations are documented.
- Drawdown, tail risk and live deployment blockers are known.
- No post-hoc selection of horizons, alphas or thresholds contaminated the final confirmation.

**Core rule:** Build the simplest mechanism first, force it to fail on credible counterexamples, and add complexity only when out-of-sample evidence requires it.
