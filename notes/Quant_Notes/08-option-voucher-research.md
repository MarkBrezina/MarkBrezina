# Velvetfruit vouchers and Hydrogel

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The source compares seven trading candidates for a snapshot dataset containing Velvetfruit spot, a ladder of Velvetfruit vouchers, and Hydrogel. Its qualitative ranking puts voucher-surface relative value first, then spot and voucher market making. The ranking describes research priority. Backtest dates, actual execution results, and a common cost model would be needed to compare return or Sharpe across candidates.

## Reported characteristics

| Item | Observation in the text | Research implication |
|---|---|---|
| `VELVETFRUIT_EXTRACT` | Spot spread around 5 ticks. | Use the executable underlying book for hedges and test spot-only making as a baseline. |
| `VEV_5000`, `VEV_5100`, `VEV_5200`, `VEV_5300` | Nearer the money in the sample, with time value and working spot betas around **0.65, 0.58, 0.44, and 0.27**, respectively. | Primary universe for a fitted time-value curve and beta-hedged residual tests. |
| `VEV_4000` and `VEV_4500` | Deep in the money in the sample, with average time value reported around **0.01**. | Candidate synthetic spot or basis hedge if settlement and conversion rules support it. |
| `VEV_6000` and `VEV_6500` | Far-out strikes with weaker two-sided liquidity. | Use liquidity and quote-quality gates before including tail strikes in a fit or execution test. |
| Spot–voucher timing | The strongest measured alignment was generally at snapshot **lag zero**. | Same-snapshot linkage supports a hedge. Event-level ordering would be needed to establish an exploitable stale quote. |
| Hydrogel | Centre near **10,000**, mean spread about **15.7**, and no strong measured cross-asset signal with Velvetfruit in the supplied text. | Standalone inventory-managed making is the first baseline. A pair trade needs separate evidence. |

The file leaves exercise style, maturity, settlement timing, and fees unspecified for this sample. Keep the voucher and hedge observations synchronised. Flag crossed, stale, or missing sides, deduplicate copied trade-print fields, and retain the reason a quote failed a quality check. A one-sided quote can distort a midpoint, parity estimate, and hedge signal simultaneously.

## 1. Voucher-surface relative value

The proposed benchmark is a smooth cross-strike time-value curve. For a call-like voucher, a simple simulator diagnostic subtracts $(S_t-K)^+$ from the observed voucher price. The intrinsic and carry conventions must match the contract when interpreting that difference. Where expiry, forward, and discount information are known, a model-implied volatility surface is a second benchmark.

1. On each eligible snapshot, calculate intrinsic and time-value diagnostics for all usable strikes. Use recent observations over a candidate **50–200 snapshot** window. Align by spot, moneyness, and time to maturity if those move materially.
2. Fit a smooth strike curve. For a same-snapshot prediction test, withhold the quote being scored from the reference fit or fit on prior data only. Compute the residual and scale it using a historical residual series with an explicit floor.
3. Test buying below **−1.5 to −2.5** residual standard deviations and selling above **+1.5 to +2.5**. Compare model residual size with the spread and use the executable side.
4. Hedge signed option units with the opposite spot exposure using the reported rolling-beta idea or a pricing delta. Convert multipliers and units before ordering. A realised rolling beta is an estimate of local co-movement, with a sampling and regime dependence of its own.
5. Test exit inside an absolute z-score of **0.5** or after **5–30 snapshots**. Define how the strategy handles a large beta shift, a spread above the training median plus two estimated standard deviations, or a missing hedge side.

In a one-for-one unit convention, ten long vouchers with a 0.58 spot beta imply a candidate short hedge of 5.8 spot units, before rounding and caps. Report the remaining delta, concentration by strike, gross exposure, partial hedge fills, and the combined markout after cost. Fit stability, executable relative value, and profitability after hedging are separate outputs.

## 2. Make markets in spot and near-the-money vouchers

The source proposes quoting Velvetfruit spot and the 5,000, 5,100, and 5,200 strikes, with 5,300 included selectively. Candidate spot quote **half-widths** are **0.5–1.2 times** the median observed spot spread. Voucher half-widths are **0.8–1.5 times** their median spread. These ratios need a common quote-unit definition and a tick floor.

Set the quote centre from fair value and lean it based on inventory and the hedge requirement. Size by visible depth and reduce size when quote churn or spot volatility rises. One candidate throttle uses a one-snapshot spot move greater than **2–3 times** the typical absolute move. When the underlying or fitted curve moves, re-evaluate the quotes, with pending cancels still counted as live until acknowledged. If the hedge book becomes one-sided, a defensive policy should price the risk of a voucher fill whose hedge cannot execute. Test immediate hedging, threshold hedging, and an inventory limit by strike or moneyness as separate variants.

## 3. Deep-in-the-money basis

The source monitors $B_t=C_t+K-S_t$ for the deep-ITM strikes. Under European parity with forward $F$ and discount factor $D$, that quantity also contains the put value and carry terms $DF-S_t+K(1-D)$. In a low-carry toy setting with negligible put value it can be close to a constant. Compute the basis from both executable combinations, using voucher ask and spot bid for a long-voucher trade, and voucher bid and spot ask for the reverse. Track fees, leg risk, expiry and exercise, and contract multipliers.

The notes consider buying the call and selling spot when the basis is low, reversing those legs when the basis is high, and exiting near a rolling median. A conversion or exercise trade requires the actual mechanism to be specified. The main research value of the deep-ITM leg is as an alternative hedge and a check on the observed surface.

## 4. Stale-quote and blip experiments

| Candidate | Proposed trigger and horizon | Evidence needed |
|---|---|---|
| **Stale voucher quote** | Spot moves **2–4 ticks** or enters the top decile of moves. Voucher moves less than **30–40%** of the expected beta-adjusted move. Proposed holding period **1–3 snapshots**. | Raw events must show that the voucher quote survived until an order could arrive. Lag-zero snapshot correlation leaves the ordering unresolved. |
| **Book blip fade** | One-snapshot move **4–5 times** the median absolute move, normal spread, weak depth continuation, and partial reversal in the next one or two snapshots. Candidate hold **1–5 snapshots**, with a further **1–2 tick** adverse move as a stop condition. | Enter after any required confirmation. Reconstruct the book from actual state changes and use a floor when typical moves round to zero. Full order-flow inference requires event data. |
| **Hydrogel maker** | Quote around a valid midpoint with a candidate half-width of **0.5–1.0 times** the median spread. Shrink risky-side size and skew around inventory. | Check markouts, stale inventory, depth collapse, and churn. Use an aligned midpoint mark for accounting, with a separate liquidation mark when relevant. |
| **Hydrogel–Velvetfruit pair** | A lower-priority lead from the strategy ranking. | Establish a stable relation and a hedge with positive incremental performance after costs before ranking it with the single-product baseline. |

Across the studies, report market-quote prediction, theoretical residual, executable entry and exit, and hedged P&L separately. See [market-making economics](02-market-making-and-microstructure.md) and the [experiment register](10-experiment-register.md).
