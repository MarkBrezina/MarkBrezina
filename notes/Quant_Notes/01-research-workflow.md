# Research workflow, learning, and implementation

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The source mixes an introductory guide with a design for an event-driven research and trading system. The introductory material is a route into the mechanics. The engineering notes define how a hypothesis would move from recorded data to a simulated order, a position, and a measurable result.

## Learning path in the £50 guide

The guide treats £50 as a learning constraint. It starts with probability, conditional probability, distributions, descriptive statistics, and hypothesis testing. The Python exercises move from variables, control flow, lists, dictionaries, and functions to NumPy arrays, pandas time series, and charts in Matplotlib. A first exercise is to load a price CSV, calculate simple returns, plot prices and returns, and check timestamps, missing data, and units.

| Exercise | What to observe |
|---|---|
| Inspect the order book | Distinguish price, displayed size, spread, depth, and the venue timestamp. |
| Place a simulated market order | Compare the reference price with the actual fills, including any consumption of multiple price levels. |
| Place a simulated limit order | Track whether it rests, becomes marketable, partially fills, or expires. Being at the best price and being first in queue are different conditions. |
| Cancel and amend | Measure when the original order stops being executable and whether queue position changes. |
| Reconcile the ledger | Match every fill with its position change, cash flow, fee, and subsequent mark. |
| Compare simulation modes | Historical replay tests repeatable hypotheses. A demo or paper account tests order interfaces and state transitions. Each uses a different approximation to live fills. |

The original guide named the Binance Spot Test Network, OKX demo, Coinbase, Robinhood, Saxo, and QuantConnect. These are historical research leads. A venue comparison needs the relevant jurisdiction, account type, product, API permissions, and data entitlement at the time of the study. The old guide also discussed the psychology of a very small live trade. Its reusable research point is that paper and live operation differ in execution, operational feedback, and behaviour.

Learning leads include Khan Academy for probability and statistics, official Python documentation, and structured courses on Coursera, DataCamp, and Quantra. QuantStart and Investopedia appear as glossary and discovery sources. arXiv and SSRN are useful places to find research papers. QuantConnect, Zipline, and Backtrader are backtesting leads. The guide also pointed to *COPtoLON/WTech*, *COPtoLON/Knowledgebase*, *COPtoLON/KB*, and *COPtoLON/TMRW*. WTech is described in the notes as an algorithmic trading research repository, and Knowledgebase as a collection spanning finance, insurance, mathematics, and AI. KB and TMRW are catalogue leads with incomplete descriptions.

## Distinguish the three clocks

| Clock | Meaning | Example |
|---|---|---|
| **Data clock** | Frequency and resolution of observations. | Individual trade and book events, minute bars, daily closes. |
| **Decision and execution clock** | Time from observation to decision, order arrival, acknowledgement, and fill. | Fast quoting requires timely book state. A daily allocator may still rely on a fast execution adapter. |
| **Risk and holding clock** | Time over which exposure is intended to persist. | Inventory recycling, multi-hour residual reversion, multi-week trend, or long-term value. |

The old guide distinguishes infrastructure-sensitive market making and latency arbitrage from signals with more time to execute. Co-location and specialised software are design choices when measured latencies determine whether the trade is still available. A signal measured on L2 data can still have a five-minute or one-hour forecast horizon. See [portfolio time scales](04-portfolio-and-time-scales.md).

## System boundary

~~~mermaid
flowchart TD
    A["Replay or live feed"] --> B["Normalised event stream"]
    B --> C["Features and state"]
    C --> D["Signals and targets"]
    D --> E["Netting and pre-trade limits"]
    E --> F["Order manager"]
    F --> G["Acknowledgements and fills"]
    G --> H["Cash and position ledger"]
    H --> E
    H --> I["Risk and monitoring"]
    G --> C
~~~

The early design named Python for research and training, with C++ and bindings such as pybind11 for measured bottlenecks. Keep the order contract and ledger consistent across implementations. A faster serialization path has little value if the strategy is waiting on data, a network round trip, a rate limiter, or a liquidity opportunity.

- **Data and features.** Store exchange and receive timestamps, sequence numbers where available, units, event types, and the instrument and venue. Separate original events from reconstructed state and derived signals. Flag stale and missing sides.
- **Strategy.** Emit intended positions or orders with a decision time, strategy identifier, expected holding horizon, and reference price. Distinguish a fair value, a directional forecast, and the maximum price worth paying.
- **Portfolio and risk.** Net overlapping targets, check gross and net exposure, estimate capital and liquidity usage, and size using stressed fills of live orders. Maintain limits on inventory, leverage, concentration, and drawdown. Treat stop orders as execution instructions whose realised price depends on liquidity.
- **Order manager.** Handle order identifiers, rate limits, authentication and nonces, routing, partial fills, rejects, and cancels. Reconcile uncertain state before a retry so an acknowledgement timeout cannot silently duplicate an order.
- **Monitoring.** Record P&L and the cash and position ledger, quote age, round-trip latency, fill rate, rejects, spread costs, slippage, and the difference between predicted and realised risk. Add alerts and a defined safe state for stale data, a position mismatch, or missing acknowledgements.

## Research sequence

1. **Prototype.** Reproduce one simple baseline in Python on fixed sample data. Check data and order invariants independently of P&L.
2. **Replay.** Use the same strategy contract in historical simulation and paper operation. Fix the latency, queue, fee, marking, and terminal-inventory assumptions before comparing strategies.
3. **Test portability.** Re-run across sessions, products, volatility states, and replay rules. Use the holdout for evaluation after tuning decisions are fixed.
4. **Integrate.** Run individual strategies and the netted portfolio through the same order, risk, and ledger interfaces. Compare paper and replay results under matched information and event timing.
5. **Profile and stage.** Measure latency before moving a path to C++. Stage releases using caps and monitoring thresholds. Reconcile positions, orders, and cash when switching versions.

The source mentions a small number of workstream labels, including *ToE* and *TMRW*, without a complete specification. The recorded ToE strategy fragment concerns local extrema and is indexed under [signal design](03-stat-arb-trend-and-regimes.md). Operational test ideas are collected in the [experiment register](10-experiment-register.md).
