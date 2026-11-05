# IMC Prosperity and simulator casebook

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The notes combine several rounds, seasons, community retrospectives, and internal experiments. Product names and rule sets below identify the case being discussed. Reported scores are in simulator units and retain the limited context in which they were recorded.

## Competition state and replay

The trader consumes `TradingState` and returns product orders, with conversion and persistent strategy data handled according to the round interface. The recorded fields include timestamp, listings, order depths, current positions, own trades, market trades, observations, and `traderData`. One example reads `order_depths["KELP"]`. A later-round fragment shows an `OwnTrade` with symbol, price, quantity, and an optional `counter_party`. Treat the available fields and their release time as part of the test specification.

Simulated counterparties can place and fill orders independently. Official matching, conversion timing, hidden agent behaviour, and execution budgets may differ from local replay. Reconcile decisions and fills from a short sample before optimising parameters. Deliberately poor strategies in the community discussion are useful negative tests for sizing, limits, and simulator accounting.

| Baseline | What the source observed or proposed |
|---|---|
| **Tutorial sample** | An example script used a hard-coded acceptable price such as 10. Its main use was confirming the interface and exposing the need for a product-specific fair value. |
| **Fixed anchor** | PEARLS and AMETHYSTS appear in different starter cases with a reference near 10,000. Compare taking favourable displayed quotes, quoting both sides, and soft or hard inventory unwind. |
| **Moving anchor** | BANANAS was modelled from recent cached prices with a small regression. STARFRUIT variants used a rolling mid, a regression, or the midpoint of the largest displayed bid and ask levels. |
| **Inventory-aware maker** | Combine estimated fair value, quote prices and sizes, and book-specific limits. A common trader keeps separate caches, fills, P&L, and exposures for each instrument. |

A PEARLS retrospective mentions snapshots with a bid around 10,002 and an ask around 9,998, and trades roughly four or five points from 10,000. These are reported simulator observations. To reproduce the trade, inspect event ordering, executable size, and simultaneous availability. The BANANAS notes describe cached-coefficient regression working better in one account than repeated on-the-fly fits. Validate that comparison by date or round, including any timeouts and data resets.

## Round-by-round research cases

### AMETHYSTS and STARFRUIT

AMETHYSTS is the stable-anchor case. Buy below and sell above a candidate fair value of 10,000, with the maker and taker components separately accounted for. STARFRUIT is the drifting case. One useful test compares the midpoint of dominant displayed depth with the ordinary best-bid/ask midpoint, a rolling average, and a short regression. The large-lot reference is a hypothesis about a simulated participant, so test periods where that participant quotes differently or is absent. The round fragments give **20 units** as the position limit for each. A retrospective reports 30,000–40,000 simulator units or more for some strong submissions, with individual manifests absent from the notes.

### ORCHIDS and conversion

The ORCHIDS case compares the local book with a foreign market and a conversion facility. Compute delivered import cost from the foreign ask plus shipping and import tariffs under the round convention. For an export, use the executable foreign bid and subtract the applicable costs. Model the sign of tariffs from the rules. Some configurations in the notes produced favourable net conversion costs.

One reported tactic repeatedly sold locally and imported to reduce the short. A replay must apply local fill probability, conversion delay, per-iteration restrictions, and the maximum permitted exposure. A participant reported about 60,000, then 109,000 after modelling a convert-and-reopen action in the same iteration. Another account describes a score below −120,000 and a rank fall from 9 to 926. Treat these as different reported outcomes, with different implementation and data assumptions. Sunlight and humidity are candidate inputs. The retrospectives suggest testing their incremental value after accounting for directly executable cross-market prices and fees.

### GIFT_BASKET and its synthetic equivalent

The basket consists of **4 CHOCOLATE, 6 STRAWBERRIES, and 1 ROSES** in the quoted round. The mark spread is

$$
P_{\mathrm{basket}}-(4P_{\mathrm{choc}}+6P_{\mathrm{straw}}+P_{\mathrm{roses}}).
$$

Estimate a plausible spread centre on training data. The notes mention centres such as 0, 300, and 370 from different accounts. For an executable long-spread trade, pay the basket ask and sell the component basket at component bids. For the opposite direction, use the basket bid and component asks. Track fill order, partial legs, and the availability of conversion or substitution. A persistent basket premium can be an equilibrium feature of the game.

| Product | Position limit quoted in the round fragment |
|---|---:|
| CHOCOLATE | 250 |
| STRAWBERRIES | 350 |
| ROSES | 60 |
| GIFT_BASKET | 60 |

A fully hedged 60-unit basket position would use 360 strawberries. From a flat book with those limits, the strawberry leg caps a fully hedged position at 58 baskets before depth, fees, and pending orders. The source describes large grid searches on roughly 30,000 example iterations, inconsistent component performance, and one reported loss of more than 36,000 on ROSES. Keep the basket-only, fully hedged, and partially hedged variants distinct. A lead–lag hypothesis for components needs timestamp-aligned tests separate from a spread-reversion study.

### COCONUT and COCONUT_COUPON

The round description treats the coupon as a call on COCONUT with strike **10,000**, with an initial time convention of **250 trading days** and one trading day per round. The quoted position limits are **300** for COCONUT and **600** for COCONUT_COUPON. The notes report a model IV around **16%** in one analysis. Confirm the maturity, volatility scaling, and settlement conventions before using those values.

A candidate strategy prices the coupon with Black–Scholes, calculates the residual or IV, and hedges the executed option position using the underlying. Delta, contracts, and allowed spot inventory determine whether the hedge can be complete. Jasper van Merle reports about 420,000 in a tuned coupon backtest, about 184,000 on coupons in the round, and a loss of about 77,000 from a separate directional COCONUT strategy. The source also describes some teams aligning a reused simulated price path with older competition data. This is a useful **data-contamination case**. Screen near-duplicate paths before calling a period a holdout, and separate option pricing performance from any advantage due to advance knowledge of the underlying path. [Van Merle, Prosperity 2 writeup](https://github.com/jmerle/imc-prosperity-2)

### Counterparty signals in the final round

The later rounds exposed simulated counterparty information. The retrospective pairs the following trades with candidate directions. The identities are game participants or bots, and the signal timing depends on when the simulator disclosed the trade.

| Product | Reported seller → buyer | Candidate response |
|---|---|---|
| CHOCOLATE | Vladimir → Remy | Long CHOCOLATE. |
| CHOCOLATE | Remy → Vladimir | Short CHOCOLATE. |
| ROSES | Rihanna or Rhianna → Vinnie | Long ROSES. |
| ROSES | Vinnie → Rihanna or Rhianna | Short ROSES. |

Van Merle describes an initial visual screen followed by about 3,600 backtests. Its final product mix used fixed-anchor AMETHYSTS, depth-anchored STARFRUIT, ORCHIDS conversion, counterparty signals in CHOCOLATE and ROSES, basket and STRAWBERRIES rules tuned on earlier data, and theoretical pricing for the coupon. It dropped the directional COCONUT rule. Test identity-based signals with point-in-time fields and a chronology that separates search from evaluation. A match against a simulator participant can reveal the rules of that particular game. [Van Merle, Prosperity 2 writeup](https://github.com/jmerle/imc-prosperity-2)

## Kelp, Resin, and the ATR/VWMP prototypes

A separate set of notes describes Kelp as the moving product and Rainforest Resin as the steadier product. The community discussion mentions **2,000** tutorial observations and **10,000** hidden observations for one setup. A fixed anchor, depth-weighted midpoint, and volatility-sensitive quote are candidate baselines. A Kelp–Resin spread needs its own economic and statistical justification. A hedge relationship requires separate evidence beyond Resin’s stability.

**Inputs.** The working notes refer to a volume-weighted market price (**VWMP**) and an **ATR** measure for quoting width, with a minimum of one tick discussed in testing. Define whether VWMP uses trades, displayed book weights, or a filtered quote centre. Use explicit bars and a price unit for ATR or a comparable range estimate. A “semi-OU” anchor is recorded as $m+\theta(m-S_t)$. With positive $\theta$ it moves beyond the mean when the current price is away from the mean. A conventional partial move toward the mean would start from $S_t$ and add a fraction of $m-S_t$. Test the two anchors as separate specifications.

| State in the prototype | Candidate half-width | Notes on anchor and rounding |
|---|---|---|
| Default | ATR divided by 2. | Kelp uses VWMP. One Resin variant centres the bid on the semi-OU estimate and the ask on VWMP. |
| “Liquid” | ATR. | Wider than the default in the recorded implementation. Test whether this reflects more adverse selection or a labelling issue. |
| No trades for a window, with depth considered in one draft | ATR. | The recorded window includes **300** time steps. Confirm depth and quote availability before a passive fill assumption. |

Two reported variants use different rounding. One rounds the anchor first, then subtracts or adds a floor-divided ATR offset. Another rounds the resulting quote price after adding or subtracting a fractional ATR offset. The draft also varies the anchor for Resin, so the **roughly 3,500** and **2,800** totals reflect variants with several differences. Log quote and fill differences before interpreting the P&L gap. Position-based size uses a candidate multiplier $1-\mathrm{Penalty}(|q|/L)$, clipped to a feasible range. Keep size reduction and centre skew as separate toggles.

The April 2025 working notes record a volatility-targeting module alongside the trader, efforts to connect `trader.py` with `backtester.py`, and trials of ARMA, moving averages, pairs, ECM, crash detection, and clearing inventory. One model name appears as “Sakarov(?)” and remains an unresolved lead. A practical grid varies trade size, ATR window, VWMP lookback, size-penalty strength and denominator, the semi-OU coefficient, and the width floor. Run Kelp and Resin independently and together. Test a fixed fallback against a volatility or HMM-based state switch. See [stat-arb and regimes](03-stat-arb-trend-and-regimes.md).

## Manual challenge notes

| Puzzle | Method described | Reported example |
|---|---|---|
| Secretive fish or bid guessing | Reason about the hidden price distribution and bids made by others. Use a simulation to assess sensitivity. | One account records a guess of **9,777**, with a bottom-half median near **9,685**. The exact rule version needs the original round brief. |
| Currency conversion | Enumerate all legal conversion paths and multiply the rates net of fees. | A route through seashells, pizza slices, wasabi roots, seashells, pizza slices, and seashells reportedly yielded **113,938.8**. |
| Expeditions | Compare tile payoffs and congestion, then decide how many entries are worth their cost. | A two-expedition choice of **83 and 87** reportedly yielded **78,577**, versus a cited best result of **106,291**. |
| Two-bid puzzle | Model how thresholds depend on the crowd's chosen bids. | An account used **952 and 980**, anticipating a higher average than an earlier 952/978 idea, and reported **102,592**. |
| News-based product selection | Rank simulated products by directional conviction, include a smaller uncertain allocation, and abstain where the signal is weak. | One account allocated 10% to icicle earrings and split the rest across five stronger calls. It reported about **115,000**, with reported comparisons of 149,000 and a hindsight 157,000. |

The news example bought ice sculptures and PS6, sold electric blankets, sleds, and moustache serum, and abstained on refrigerators, lava lamps, and hot chocolate. These are historical simulator decisions. The future-round planning notes also mention stock splits, dividend-like mechanics, product premia, options surfaces, and transport fees as scenario types to prepare for.

The source names several writeup leads, including Linear Utility, Stanford Cardinal, Jasper van Merle, Kang/Kim/Lee, Gabriel Romon, Michael Okon, Nicolas Sinott, Academiq-AI, and David Teather. Its rankings mix editions and contain gaps. Keep the edition, round, dataset, and submission manifest with any comparison. The durable experiment is a simple fair-value and inventory baseline under consistent rules, followed by a measured test of each new feature.
