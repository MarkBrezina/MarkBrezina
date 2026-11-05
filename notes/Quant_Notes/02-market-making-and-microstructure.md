# Market making and microstructure

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The notes compare simple stateful traders and a more detailed fixed-width quoting case. The recurring design is to estimate fair value, choose the price and size of each quote, and use aggressive orders selectively. The same position and pending-order state must govern both making and taking.

## Three fair-value archetypes

| Archetype | Estimate | Order rule | State to retain |
|---|---|---|---|
| **Fixed anchor** | An approximately stable competition price of 10,000. | Take offers below fair and bids above fair when the edge covers costs. One example posts 9,999 and 10,001. Another uses explicit acceptable bounds and improves existing quotes by one tick when allowed. | Positions, fills, cash, and remaining capacity on each side. |
| **Recent book and EMA** | Weighted averages of recent bids and asks, and an EMA of an agreed reference price. | Compare the current executable book with the cached benchmark, quote around the EMA, and skew by inventory. | Recent books, weights, EMA, P&L, and inventory. |
| **Rolling regression** | A one-step mid-price forecast from a cache, illustrated on BANANAS. | Convert the forecast into acceptable buy and sell bounds. Compare fixed coefficients with rolling refits. | Mid-price history, regression parameters, market data timestamps, and product limits. |

These approaches can share reset, buy, sell, and quote-management logic. A stable anchor observed in a game is a parameter of that test. A real market requires an estimate that can fail or move.

## Quote construction

Let $b$ and $a$ be the best bid and ask, with sizes $v_b$ and $v_a$. The midpoint is $(a+b)/2$. A simple top-of-book queue-weighted price is

$$
m_{\mathrm{qw}}=\frac{a v_b+b v_a}{v_b+v_a}.
$$

A large bid queue pushes this reference toward the ask. It is a displayed-depth signal. Executed signed flow, event-based order-flow imbalance, and cancellation or replacement rates are separate inputs. A practical fair value $f_t$ can be a fixed anchor, filtered mid, EMA, volume-weighted book reference, or queue-weighted price with a fitted flow adjustment. Record how each is calculated and evaluate its next-move prediction before attributing trading P&L to it. [Cont, Kukanov, and Stoikov, *The Price Impact of Order Book Events*](https://arxiv.org/abs/1011.6402)

With inventory $q_t>0$ representing long units, a simple reservation price is

$$
r_t=f_t+\ell_t-kq_t,\qquad
b_t^{\mathrm{quote}}=r_t-h_t,\qquad
a_t^{\mathrm{quote}}=r_t+h_t,
$$

where $\ell_t$ is an optional alpha adjustment, $k$ is the inventory penalty per unit, and $h_t$ is the quote half-width. **Long inventory lowers both quotes**, improving the chance of selling and reducing the aggressiveness of the bid. **Short inventory raises both quotes**. One passage in the source gives the opposite sign. The worked table, reservation-price formula, and inventory-reduction objective agree with the convention here.

The classic Avellaneda–Stoikov finite-horizon model includes a term of the form $q\gamma\sigma^2(T-t)$ in the reservation-price shift under its price and fill assumptions. The notes use simpler inventory and volatility penalties for replay. Estimate quote width and placement separately from the inventory skew if the simulator or venue permits it. Other research routes include Cartea–Jaimungal-style control with execution costs, queue models, jump or stochastic-volatility price dynamics, and learned fill intensities. [Avellaneda and Stoikov, *High-frequency Trading in a Limit Order Book*](https://doi.org/10.1080/14697680701381228)

Two simpler variants from the notes use a variance-sensitive centre $r_t=f_t-q_t\gamma\widehat{\mathrm{Var}}_t$ or a dynamic half-width $h_t=\max(h_{\min},c_s(a_t-b_t)+c_\sigma\widehat\sigma_t)$. Choose the volatility window and price units explicitly. Quote width, centre shift, and size are separate levers. An estimated rise in volatility can increase the width, increase the inventory penalty, or reduce the size, with each change tested separately.

### Fixed 20-point quote width, position cap ±80

This is one configuration from the notes. It assumes one live pair of quotes, a 10-point half-width, and a fair reference near 10,000. Use $r=f-10q/80$ before tick rounding.

| Inventory | Skew subtracted from fair | Bid | Ask | Limit treatment |
|---:|---:|---:|---:|---|
| −80 | −10 | $f$ | $f+20$ | Quote the bid only. |
| −40 | −5 | $f-5$ | $f+15$ | Size to available capacity. |
| 0 | 0 | $f-10$ | $f+10$ | Symmetric baseline. |
| +40 | +5 | $f-15$ | $f+5$ | Size to available capacity. |
| +80 | +10 | $f-20$ | $f$ | Quote the ask only. |

Candidate variants use skew multipliers 0.3, 0.5, 0.7, and 1.0, and soft one-sided thresholds around 40, 60, or 70 units beneath the hard cap. A rough operating rule in the notes quotes normally below 40 units, reduces the risky side between 40 and 70, and quotes only toward flat between 70 and 80. Other candidate settings include about one tick of skew per 10–15 units and size around 20–30 units near neutral. These are grid points for replay. One draft proposes keeping absolute inventory below the limit on 99% of ticks as a soft occupancy target. For a hard symmetric cap $L$, confirmed inventory $q$, and outstanding buy and sell quantities $Q_b$ and $Q_s$, the remaining buy and sell capacities are $\max(0,L-q-Q_b)$ and $\max(0,L+q-Q_s)$. Include pending cancels and outstanding takes until they are reconciled. Test time spent near the limit, final inventory, and adverse tails alongside the average P&L.

The source also discusses **one-tick price improvement** when quoting inside a 20-point market spread. That would leave 18 points between the two quotes if the tick is one point. It is a separate experiment from a mandated 20-point own width. Changing the quote can change queue priority and fill composition. Another chart description gives half-widths around 20–30 points. Measure the full observed spread from aligned bid and ask columns before choosing which interpretation applies.

## Making, taking, and inventory control

| State | Candidate action | Measurement |
|---|---|---|
| Normal market, inventory near target | Post both sides within limits, using a fixed width or a declared dynamic half-width rule. | Fill share, execution price, markout, and quote churn. |
| Moderately stretched fair-value residual | Shift the quote centre or size on one side, with a directional signal kept separate from inventory skew. | Incremental edge after controlling for changes in fills and inventory. |
| Large executable dislocation | Compare fair value with the current best ask for a buy or best bid for a sell. Take only if the gap clears a cost and uncertainty threshold and the order fits the inventory budget. | P&L and fill quality versus an otherwise identical maker-only baseline. |
| Inventory near a cap | Reduce or remove orders that increase exposure. Compare passive unwinding with a rule that crosses the spread under defined risk conditions. | Time to flatten, lost maker opportunities, and exit cost. |
| Missing side, stale book, shock, or unusually fast churn | Reduce size, widen quotes where allowed, cancel, or switch to a conservative state according to the replay rules. | Stale fills, partial fills, inventory tail, and recovery time. |

The source reports approximately **2,300** for one maker-only run and **1,600** after adding a taking rule. The run manifest needed to attribute the difference is absent. Compare the same sample, fill rules, sizes, costs, and end mark while varying only taking. Candidate experiments include a wide trigger around 6–10 ticks from fair, a rule tied to local volatility, and a rule that permits taking only to reduce an existing position. A z-score can also be a filter for quoting rather than a direct order trigger.

Quote refresh can react to meaningful reference-price changes, fills, and inventory state. At high volatility, the notes consider wider quotes and smaller size on both sides. A price forecast and a desire to exit inventory are distinct reasons for moving one side more aggressively. The source's short-term mid-price model sketch adds a jump term to a drift and diffusion process, motivating shock scenarios even when the observed series usually stays near its centre.

## Measure fills and economics

Mark the strategy with cash plus inventory value. If cash includes all trades and fees and $m_t$ is the chosen mark, net P&L is $(C_T+q_T m_T)-(C_0+q_0 m_0)$. Report a terminal liquidation calculation too if the strategy must end flat. Decompose trade-level performance with signed markouts $s_i(m_{t_i+h}-p_i)$, where $s_i=+1$ for buys and $-1$ for sells. This separates the entry price from price movement after the fill. The posted spread is a hypothetical round-trip gross capture for matched fills at those prices.

- **Fill model.** Track queue assumptions, trade-through requirements, partial fills, cancel delay, priority on amend, and simulator-specific matching. A quote touching the market is a weaker fill assumption than an actual trade reaching that order's estimated queue position.
- **Cost model.** Account for spread paid when taking, fees or rebates, price impact where applicable, and exit costs. Run with higher delay and less favourable fill assumptions.
- **Inventory model.** Record current and worst-case projected inventory, occupancy of inventory bands, directional mark-to-market, and forced-unwind losses. Check both sides after every event.
- **Evaluation.** A range-bound chart and about 30,000 observations motivate a test. They leave queue behaviour, latency sensitivity, and the distribution of adverse runs to be measured. Compare fixed anchor, EMA, microprice, no skew, linear skew, and inventory-tiered variants before adding complexity.

See the [Prosperity casebook](07-prosperity-casebook.md) for specific products and ATR/VWMP variants, and the [voucher study](08-option-voucher-research.md) for multi-leg quoting and hedge constraints.
