# Experiment register

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

This register turns the source's open questions into proposed comparisons. Each comparison has a baseline and an observable outcome. The numerical results cited in the chapters are historical reports. The entries below are plans for reproducible tests.

| ID | Research question | Baseline and change | Main outputs |
|---|---|---|---|
| **MM-01** | Does inventory skew improve a fixed-width maker? | Same fair value, width, and order sizes. Compare symmetric quotes, linear skew, and one-sided inventory tiers under the same cap. | Net P&L, signed markout, time near the cap, end inventory, liquidation cost. |
| **MM-02** | Does taking add value to making? | Maker-only, then wide residual thresholds, then inventory-reducing takes only. Hold latency, queue, and ledger rules fixed. | Incremental P&L, lost passive fills, spread paid, and changes in inventory occupancy. |
| **MM-03** | Does the book give a better fair value? | Compare fixed anchor, raw midpoint, EMA, dominant-depth midpoint, and queue-weighted price. | Prediction error, executable markout, fill distribution, and robustness when depth or a participant disappears. |
| **MM-04** | What explains the ATR/VWMP variant gap? | Fix one product, anchor and replay. Vary rounding, then width mapping, then inventory penalty. | Quote-by-quote differences, P&L attribution, fills, and width-floor events. |
| **RV-01** | Do factor residuals revert after realistic costs? | Unfiltered residual z-score, then add one news, volatility, liquidity, or correlation condition at a time. | Held-out conditional returns, factor leakage, turnover, borrow, net P&L, and tail losses. |
| **RV-02** | When is the moving-product pair usable? | Fixed spread, rolling ECM, and regime-gated ECM or VECM. Fix fit window, stationarity assessment, and refit frequency for each run. | Residual stability, false switches, reaction to Ink-like shocks, hedge errors, and after-cost returns. |
| **RV-03** | Do basket and conversion prices leave executable spreads? | Compare midpoint-only marks with executable legs, position limits, fees, and permitted conversions. | Net spread, completion rate, unhedged legs, P&L versus mark and conversion benchmark. |
| **VO-01** | Does a voucher residual predict executable repricing? | Simple intrinsic/time-value or constant-vol baseline, then cross-strike smooth fit, then beta hedge. Use withheld quotes and chronological holdouts. | Calibration and bid/ask coverage, realised option markouts, hedge slippage, residual delta, and net returns. |
| **VO-02** | Are snapshot blips and lagged vouchers tradeable? | Same-snapshot relation and no-trade benchmark. Add event ordering, confirmation lag, and conservative order arrival. | Quote survival, tradable lead, adverse moves after the signal, and retained net edge after fees. |
| **TR-01** | Do adaptive horizon weights improve a trend portfolio? | Equal fixed weights, then performance weighting or trend/chop gating, with matched risk and cost assumptions. | Net performance, effective exposure, turnover, regime sensitivity, and contribution by horizon. |
| **SC-01** | Do apparent corrections change across scales? | Matched return and noise nulls with the same sampling and event definitions as the observed series. | Excursion rate per unit time, volatility-scaled magnitude, time to reversal, and continuation probability. |
| **CR-01** | Can a crash proxy add warning time? | Volatility-only trigger, then skewness, down/up volatility, trade value, or depth features. | False-alarm rate, lead time, calibration, and error in predicted crash size. |
| **C1** | Which transition handles a model switch best? | Immediate switch, fixed ramp, and re-optimisation from actual inventory. | Cost, tracking to new target, peak risk, and unfinished trades at the deadline. |
| **C2** | Does a joint model improve central-price forecasts? | Persistence, immediate-only, history-only, and a time-varying blend. | Error by instrument and horizon, marginal and joint prediction coverage, width, and coherence across linked prices. |
| **C3** | Can a schedule reduce detectable footprint at equal completion? | Immediate, TWAP, VWAP or participation, and a cost-risk scheduler. Evaluate a separate leakage observer. | Implementation shortfall, price risk, fill completion, and leakage for comparable parent orders. |
| **C4 / RL-01** | Does a richer state or learned policy improve constrained decisions? | Finite-state dynamic programme and rule-based skew. Add longer history or a learned policy under matching constraints. | Out-of-sample policy value, constraint violations, and sensitivity to simulator or latency assumptions. |
| **C5** | Does coordinated allocation improve shared capital use? | Independent strategy targets, netted targets, and joint or resource-priced allocation. | Firm-level net P&L, risk, concentration, turnover, feasible allocations, and each strategy's local objective. |

## Minimum reproducibility record

- **Data.** Dataset and version, source, universe, instrument and contract definitions, trading calendar, quote and event semantics, gaps, and information-availability timestamps.
- **Experiment.** Hypothesis ID, baseline, single change, parameter search, feature warm-up, training and evaluation dates, outcome horizon, and the assumptions used to split overlapping examples.
- **Execution.** Decision lag, placement or conversion rules, queue and fill assumptions, fee and cost units, exposure and order-size limits, partial fills, cancellation timing, and rejected orders.
- **Accounting.** Cash and position ledger, P&L marking, terminal liquidation convention, benchmark return, and capital denominator. Preserve enough order and fill events to reproduce the result.
- **Outputs.** Counts, uncertainty or variation across folds, net performance, drawdown, failure cases, and the result of adverse cost and fill assumptions. A strategy that trades fewer periods should be compared on its risk, capital usage, and eligible opportunity set as well as average trade P&L.

The implementation notes start from matching the trader and backtester interfaces. A small hand-checked replay makes the subsequent grids interpretable. The [source map](../SOURCE_MAP.md) identifies the relevant original sections and the passages with missing figures.
