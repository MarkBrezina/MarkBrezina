# 11 — Options-Implied Densities, Quant Python Stack, Neural Models, Enterprise P&L and Live Trading Architecture

> **Intake:** User's continuation, 2026-11-06 (Parts 22–23, 25–26, 28–29). Complements the option payoffs and volatility-risk premium in [08](08_STRATEGY_TAXONOMY_OPTIONS_COMMODITIES_MACRO_AND_INTRINSIC_TIME.md), model selection in [04](04_MODELS_ARCHITECTURE_MACRO_RESOURCES_AND_CAREER.md), and execution/validation in [03](03_PORTFOLIO_CONTROL_EXECUTION_OPTIMISATION_AND_VALIDATION.md). Treat implementation suggestions as **[IMPLEMENT]** and financial effects as **[HYPOTHESIS]** until independently validated.

## 1. Crypto options and risk-neutral densities (RNDs)

### 1.1 What an options-implied density is

Under an idealized arbitrage-free model with deterministic discount factor `D(0,T)`, European call prices across strikes at a fixed expiry satisfy

\[
C_0(K,T)=D(0,T)\,\mathbb E^{\mathbb Q}[(S_T-K)^+],
\]

and (where derivatives exist)

\[
\boxed{q_T(K)=\frac{1}{D(0,T)}\frac{\partial^2 C_0(K,T)}{\partial K^2}}.
\]

This is the **Breeden–Litzenberger** recovery of an implied *risk-neutral* terminal density `q_T`, not an estimate of objective/physical frequency `p_T` without additional assumptions. The source paper is [Breeden & Litzenberger (1978)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2642349).

Key distinctions:

- **Risk-neutral probability** (`Q`) is a pricing object incorporating state prices and risk premia, not an empirical observed frequency.
- **Physical probability** (`P`) is relevant for realized coverage, stop probabilities and actual expected investment outcomes.
- A low risk-neutral quantile can be a useful **market-implied stress threshold**; it is not automatically a statistically calibrated 5% real-world Value at Risk.
- In practice, finite noisy quotes, bid–ask spreads, interest/carry inputs, American exercise and invalid prices make numerical second derivatives unstable.
- Always enforce call monotonicity and convexity in strike, maturity consistency/calendar no-arbitrage, and appropriate forwards/discount factors before extracting a density.

### 1.2 Ethereum application from the user's screenshot notes

The source notes describe ETH options across a high-volatility window (March–April 2024) with visibly wider implied tail ranges in higher-IV states. That is an **illustrative historical observation**, not proof of forecast coverage or a unique volatility model.

Possible features at each market timestamp `t` and expiry `T`:

- Implied median and `Q` 5th/25th/75th/95th percentiles.
- Forward, ATM implied variance, skew and risk reversal.
- Tail-shape coefficients, IV-surface principal components, term slope.
- Distance of spot to risk-neutral 5% threshold in volatility units.
- Difference between risk-neutral forecast and historical physical-distribution forecast at the same horizon.
- Residual of realized moves against priced tails; coverage calibration over a rolling sealed test period.

### 1.3 Proposed options-density experiment

1. Acquire historical **timestamped bid/ask option chain**, contract specifications, settlement/index prices, forward or basis, rates, and a synchronized underlying/hedge price.
2. Apply quote filters: nonpositive bids, crosses, implausible wings, stale timestamps, missing strikes, liquidity limits, duplicate/outlier contracts.
3. Fit a no-static-arbitrage smile across strikes and reconstruct a discrete RND. Compare spline, SVI-compatible or arbitrage-aware representations without overfitting sparse wings.
4. Calculate risk-neutral quantiles and compare with **future physical outcomes** only at the option horizon (no mixing tenors).
5. Benchmark against Gaussian-GARCH, historical block bootstrap, t-distribution, and empirically calibrated physical density forecasts.
6. Evaluate proper scoring rules (CRPS/log score with tail diagnostics), quantile coverage, PIT uniformity, forecast reliability and trading costs if any strategies are proposed.
7. Test market regimes separately. A good density forecast is not automatically a positive net P&L strategy; the market premium paid for tail protection must be included.

**Hypothesis:** Changes in RND width/skew may improve regime prediction or option hedging relative to underlying-only features. **Falsifier:** no calibrated incremental score or net decision value in genuinely held-out windows.

## 2. Free/open quant science stack — components versus claims

The user lists seven widely used tools. These are **options** for a modular architecture, not a requirement to install all of them or evidence that a system will be profitable.

| Tool | Role | Practical suitability / caution |
|---|---|---|
| [NumPy](https://numpy.org/) | Core arrays, linear algebra, simulation | Efficient vectorization; beware excess copies and floating-point issues |
| [pandas](https://pandas.pydata.org/) | Time series, joins, features, tabular analysis | Preserve point-in-time alignment, exchange calendars, memory budgets |
| [OpenBB](https://github.com/OpenBB-finance/OpenBB) | Open financial-data integration/platform | Access varies with vendor credentials, source licensing and API coverage |
| [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib) | Risk measures, portfolio and factor allocation | Confirm optimizer formulation, solver availability and current version |
| [Zipline-reloaded](https://github.com/stefan-jansen/zipline-reloaded) | Historical event-driven/vectorized strategy simulation | Equity calendar conventions may not match crypto 24/7, Level-2 or maker order fills |
| [Pyfolio-reloaded](https://github.com/stefan-jansen/pyfolio-reloaded) | Tear sheets and portfolio diagnostics | Check project maintenance and dependency compatibility; use QuantStats/own reports as alternatives |
| [Alphalens-reloaded](https://github.com/stefan-jansen/alphalens-reloaded) | Factor predictive diagnostics | Forward returns must be calculated without leakage; factor performance is not executable profits |
| [IBKR APIs](https://www.interactivebrokers.com/) | Brokerage orders and market-data interface | Brokerage access requires account, entitlements, exchange permissions; not a universal free live-feed source |

Avoid assuming maintained status based on original package names: `zipline`, `pyfolio` and `alphalens` have multiple forks. Pin versions and test installation in the specific Python environment. NumPy and pandas are **tools**, not a full streaming architecture. The data and execution systems must be separated.

### 2.1 Practical supplemental libraries

- `scipy`, `statsmodels`, `arch`, `scikit-learn`: numerical optimization, inference, ARCH/GARCH and baseline models.
- `polars`, `pyarrow`, `duckdb`: large cached minute/second data and efficient columnar persistence.
- `numba`, `cython`, `pybind11`, C++/Rust: speed up measured hot paths while preserving scientific reproducibility.
- `pytorch`, `jax`, `blackjax`: neural learning and probabilistic inference. Do not confuse inference model speed with end-to-end order latency.
- `cvxpy`, `riskfolio-lib`: convex/allocation formulations, with solver/timing checks.
- `pytest`, `hypothesis`, `ruff`, `mypy`: reliability and programmatic invariants; for a trading system, tests are part of the risk controls.
- `fastapi`, message queues and exchange WebSocket APIs: ingest/control plane; verify ordering and idempotency.
- See [04](04_MODELS_ARCHITECTURE_MACRO_RESOURCES_AND_CAREER.md) for Qlib, Kronos, TradingAgents, WorldMonitor and BlackJAX references.

### 2.2 Data lake schema recommendation

Store files outside Git with immutable manifests. Suggested format:

```text
data/
  venue=binance/asset=BTCUSDT/frequency=1m/year=2025/month=10/part.parquet
  venue=revolutx/asset=BTCUSDT/frequency=trade/year=2026/month=11/part.parquet
  venue=.../asset=.../frequency=l2_delta/year=.../month=.../part.parquet
  contracts/options_master.parquet
  contracts/futures_master.parquet
  calendars/releases_with_publication_time.parquet
  manifests/{provider,license,checksum,asof,timezone,schema_version}.json
```

Minimum columns: UTC event timestamp; local exchange sequence; ingest timestamp; symbol/contract/venue; price and size; bid/ask; event type; provenance; raw/adjusted or revision status; session/calendar; schema version. Historical fee schedules and tick sizes require **effective dates**.

**Rule:** Signals at decision time `t` can only use data observable **before** the corresponding order submission. A bar ending at `t` must not be assumed tradeable at the same price and timestamp if exchange/bar dissemination happens after that instant.

## 3. Activation functions for financial machine learning

Given a preactivation `x`, common activations include:

| Function | Equation | Property / usage caution |
|---|---|---|
| Sigmoid | `1/(1+exp(-x))` | Bounded [0,1], useful for binary probability heads; saturates |
| Tanh | `(exp(x)-exp(-x))/(exp(x)+exp(-x))` | Bounded [-1,1], zero-centred; saturation at extremes |
| ReLU | `max(0,x)` | Simple, sparse; negative preactivations can die |
| Leaky ReLU | `max(ax,x)`, fixed `a>0` | Provides small negative slope |
| PReLU | `max(ax,x)`, learned `a` | Adds parameters; monitor instability |
| ELU | `x` for `x>0`, `a(exp(x)-1)` otherwise | Smooth negative saturation |
| SELU | `lambda * ELU_alpha(x)` | Self-normalization depends on network/initialization/dropout assumptions |
| Softplus | `log(1+exp(x))` | Smooth positive mapping; numerical stable implementation required |
| Softsign | `x/(1+abs(x))` | Smooth bounded output, slow tails |
| Hard sigmoid | Piecewise-linear clipped approximation | Fast but piecewise non-smooth |
| Swish/SiLU | `x * sigmoid(x)` | Smooth non-monotonic activation |
| Mish | `x * tanh(softplus(x))` | Smooth non-monotonic activation |

**Model-selection principle:** An activation is part of a numerical learning architecture, **not an alpha mechanism**. Benchmark activations after fixing architecture, input features, data budget and training protocol. Financial models can fit noise extremely well; stability across instruments and sealed periods is more informative than train loss.

For trading output heads: a price return forecast may use identity; volatility output needs a nonnegative mapping (`softplus` or exponential); categorical regime probabilities use `softmax`; position output can use bounded `tanh` then rescale/risk-gate. Calibrate probabilities rather than relying on activation output shape alone.

### 3.1 Sample research blueprint for multiwindow SMA model

A clean SMA base strategy is **not** an inheritance hierarchy by necessity. Either an OOP class or stateless functional interface works; the critical requirements are causal features, consistent execution and auditability.

```python
from dataclasses import dataclass
import pandas as pd

@dataclass(frozen=True)
class SMAParams:
    short: int = 20
    middle: int = 30
    long: int = 100


def sma_target(close: pd.Series, p: SMAParams) -> pd.Series:
    if not (0 < p.short < p.middle < p.long):
        raise ValueError("Require 0 < short < middle < long")
    s = close.rolling(p.short, min_periods=p.short).mean()
    m = close.rolling(p.middle, min_periods=p.middle).mean()
    l = close.rolling(p.long, min_periods=p.long).mean()
    target = ((s > m) & (m > l)).astype(float)
    # Target at bar close t can first be executed at t+1 or later.
    return target.where(l.notna(), 0.0).shift(1).fillna(0.0)
```

The returned series is a *desired exposure* after a one-bar signal delay, not a trade-fill simulator. Test the opposite (bear trend) state, no-trade neutral state, costs, jumps, rebalancing and position sizing separately. Compare 20/30/100 with 5/15/30 and 5/20/40, and include a fixed-parameter holdout. An equity backtest using MSFT/GOOGL/NVDA/TSLA/AMZN needs survivorship-aware historical universes to avoid selecting future winners.

## 4. Enterprise risk and P&L architecture — "Superfly" inspiration

The user's notes describe a conceptual enterprise risk engine dubbed "Superfly". No original specification or vendor manual has been provided; treat this as a **requirements vision**, not a documented product architecture.

### 4.1 Risk should be computable at multiple aggregation levels

At time `t`, define positions `q_i`, instrument values `V_i`, state vector `x` (spot, rates, vol surfaces, FX, credit, correlations). Total marked value:

\[
V_{\mathrm{portfolio}}(x,t)=\sum_i q_i(t)V_i(x,t)+\mathrm{cash}(t).
\]

The risk system should reconcile aggregate and bottom-up exposures:

- **P&L:** market move, new trades, fees, funding, dividends, option decay, carry and adjustments.
- **Spot/FX:** delta or factor sensitivity by asset, region, desk, legal entity.
- **Rates:** key-rate DV01, curve principal components, basis, convexity.
- **Options:** delta, gamma, vega grid by expiry/strike, theta, vanna/volga where meaningful, jump/stress.
- **Credit:** spread DV01, jump-to-default, ratings/sector/basis; exposure to funding and liquidity.
- **Portfolio:** concentration, covariance/PCA loadings, factor decomposition, stress and conditional expected shortfall.
- **Event/calendar:** Fed/BoE/ECB dates, CPI, earnings, expiries, margin calls and contract rolls, with point-in-time timestamps.
- **Risk data:** source freshness, stale quote flags, missing Greeks, unsupported instruments, wrong units, currency mismatches and reconciliation failures.

For linear risks, `sum(position sensitivities)` should reconcile with `aggregate sensitivity`. Nonlinear stress P&L **must** reprice rather than assume Greeks aggregate into exact finite shock P&L. Correlation exposures depend on modelling convention and should not be sold as exact causal risk.

### 4.2 Risk engine quality checks

1. `zero positions -> zero directional risk` (except residual funding/cash obligations).
2. `portfolio value = cash + sum marked instrument values` with reconciled currency conversions.
3. `delta of long + identical short = 0` under matched valuation inputs.
4. `rates DV01 by bucket -> aggregate DV01` within tolerance.
5. `pre/post trade limits` check against projected fill and current outstanding orders.
6. `P&L attribution residual` logged and investigated, not silently discarded.
7. `stale market data` causes degraded-mode controls and/or safe halt.
8. Stress tests include correlation breaks, volatility jumps, liquidity disappearance, funding changes, exchange halt and connectivity loss.

### 4.3 Financial measurement and latency

A useful design is **two-speed risk**:

- **Fast pretrade:** notional, gross/net inventory, current open orders, max order size, worst-case exposure under pending fills, live spreads/depth, stale-data cutoff, kill switches.
- **Full slower risk:** curves/Greeks, scenario repricing, factor/PCA decomposition, expected shortfall, model uncertainty and cross-asset stress.

Ultra-low latency execution need not force an expensive full portfolio Monte Carlo run on each market tick. Fast bounds protect execution; asynchronous deep risk analysis feeds limits and alerts.

## 5. End-to-end live trading: canonical separation of concerns

```text
 External exchanges and feeds
   |-- WebSocket trades, quotes, depth deltas
   |-- REST account, balances, fee schedules and order acknowledgments
   |-- Macro data and releases with timestamps (research)
   v
 [Raw Event Capture + Append-only Journal]
   -> [Sequence/clock validation] -> [Order-book reconstruction]
   -> [Feature Store / State Estimator] -> [Signal & Strategy Policies]
   -> [Portfolio & Inventory Optimizer] -> [Pretrade Risk Gate]
   -> [OMS / Idempotent Order State Machine] -> [Exchange Adapter]
                                           |         ^
                                           v         |
                                     [Fills / ACKs / Cancels]
                                           v
 [Positions & Double-entry-style Ledger] -> [PnL attribution / limits / alerts]
                 |                                   |
                 +--------> [Research/audit data lake] <---+
                                      |
                           [Backtest / paper trading / replay]
```

**Keep exactly the same signal, risk and accounting definitions across backtest, paper and live**, while allowing a realistic execution adapter and exchange venue model to differ.

### 5.1 Reliability invariants

- Distinguish `order intent`, `submitted`, `acknowledged`, `partially filled`, `cancel requested`, `canceled`, `rejected` and `filled`. A cancel request is **not** proof cancellation occurred.
- All exchange interactions are replayable/auditable. Stable client IDs and idempotent handling protect against retries and duplicated messages.
- On reconnect, reconcile exchange orders/fills/balances against local state before resuming risk-taking.
- Run risk on *possible* pending fills, not only confirmed positions; otherwise simultaneous open buy orders can exceed capital limits.
- Use bounded exposure, explicit handling of missing book snapshots and out-of-order deltas, and an independently operating kill switch.
- Track maker **fill probability and adverse selection**, not just quoted fee rate. A zero maker fee does not make execution free.
- Authentication credentials and personal data live outside Git; no live exchange keys in notebooks, logs or demo repositories.

### 5.2 Language and software choices

- **Python** for research, data science, strategy orchestration and prototyping.
- **C++/Rust/Cython/Numba** only where measurements show material compute/latency bottlenecks, with integration tests preserving numerical and state-machine correctness.
- **SQL/DuckDB/Parquet** for analytics plus append-only raw event storage; a relational database alone can become a throughput or replay bottleneck.
- **WebSocket and async adapters** for live ingestion and order workflows; deterministic research replay remains necessary.
- **Excel** can be a monitoring/reporting interface but should not act as the sole authoritative order ledger or safety controller.

### 5.3 Example project tree

```text
trading_system/
  contracts/            # Symbol metadata, tick sizes, lot sizes, expiry, rates
  adapters/             # Exchange-specific market/account/order integration
  ingest/               # Sequence, clock and integrity
  orderbook/            # L2 snapshot/delta reconstruction
  features/             # Strictly causal indicators and stores
  market_state/         # Vol, regime, network, uncertainty
  models/               # Baselines, ML, posterior inference
  strategies/           # Signals and expected executable value
  portfolio/            # Allocation, inventory and constraints
  risk/                 # Pretrade and full-book risk
  execution/            # OMS, makers/takers, retries and routing
  ledger/               # Orders, fills, cash, positions, PnL attribution
  simulation/           # Replay, fees, book, partial fills, scenarios
  reporting/            # Equity and attribution, factor and stress reports
  experiments/          # Frozen configs and evaluation outputs
  tests/                # Unit, property-based, integration and replay tests
  notebooks/            # Exploratory analyses; no production authority
```

## 6. Prioritized research and implementation proposals

| ID | Experiment/feature | Needed data | Baseline | Pass condition |
|---|---|---|---|---|
| SYS-01 | Options RND recovery, shape + quantile calibration | Point-in-time chain, forward, rates, realized expiry spot | Gaussian/GARCH + fitted physical density | No-arbitrage smile and incremental physical quantile score |
| SYS-02 | Option surface shape regimes | IV level/skew/term data | ATM IV and realized volatility | Out-of-time gain in regime/risk prediction |
| SYS-03 | SMA 20/30/100 regression tests | Asset historical candles | Buy-and-hold, fixed one-horizon trend, random | Causal alignment and net performance beyond costs |
| SYS-04 | Neural activations controlled benchmark | Frozen market-feature datasets | Logistic/GBDT and ReLU MLP | Stable uplift in sealed forecast and net trading utility |
| SYS-05 | Risk reconciliation | Position/fill/market repricing fixtures | Independent accounting truth table | Zero unexplained material delta, FX, rates and PnL residuals |
| SYS-06 | OMS crash/reconnect/replay | Simulated out-of-order/dropped exchange messages | Deterministic state simulator | No unhedged duplicate fills and fully reconciled restart |
| SYS-07 | Two-speed risk gate | Synthetic realistic OMS tick events | Naive per-order checks | Bounded risk with measured predictable latency |
| SYS-08 | Framework choice study | Same samples/model in Qlib/own backtest/Zipline | Current minimal working core | Same decisions and accounting within known tolerance |

## 7. References and provenance

- [Breeden & Litzenberger (1978), option-implied state prices](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2642349) — established theoretical link under stated idealized conditions.
- [Riskfolio-Lib repository](https://github.com/dcajasn/Riskfolio-Lib), [OpenBB repository](https://github.com/OpenBB-finance/OpenBB), [Zipline-reloaded](https://github.com/stefan-jansen/zipline-reloaded) — software reference targets, not evidence of strategy returns.
- [Pyfolio-reloaded](https://github.com/stefan-jansen/pyfolio-reloaded), [Alphalens-reloaded](https://github.com/stefan-jansen/alphalens-reloaded) — verify dependency versions/maintenance.
- User's supplied screenshot notes: ETH March–April 2024 risk-neutral density expansion, 20/30/100 SMA class design, “Superfly” risk expression, and modular Python/C++/C# trading bot; these are **[USER_NOTES]**, not independently observed datasets or proprietary firm manuals.
