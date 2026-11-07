# 08 — Strategy Taxonomy, Options, Commodity Carry, Macro Fair Value, Positioning and Intrinsic Time

> **Intake:** User-provided continuation, 2026-11-06. **Status:** Technical notes and a research blueprint, **not** a claim that any strategy is profitable. Historical performance claims copied from user-supplied summaries are **unverified** unless separately cited. This sheet supplements [02](02_MARKET_STATES_MICROSTRUCTURE_AND_STRATEGY_RESEARCH.md), [03](03_PORTFOLIO_CONTROL_EXECUTION_OPTIMISATION_AND_VALIDATION.md), [06](06_REFERENCED_PAPERS_CRITICAL_REVIEW.md) and [07](07_TREND_MEAN_REVERSION_AND_RESEARCH_PROGRAMME.md).

## 0. The central organizing principle: distinguish *why*, *what*, *when* and *how*

Most named trading strategies are recombinations of a smaller collection of **economic mechanisms**, implemented with different **instruments**, **frequencies**, **signals** and **execution techniques**.

| Dimension | Examples | Common source of confusion |
|---|---|---|
| Economic return mechanism | Trend persistence; mean reversion; carry/risk premia; providing liquidity; information advantage; convergence/arbitrage | A label for an asset class is not an edge |
| Signal / observable | Moving average, news sentiment, order-flow imbalance, inventory, option IV minus RV, funding rate, fair-value residual | A predictive feature is not an executable trade |
| Instrument | Cash equity, spot crypto, future, option, swap, bond, ETF, cross-currency pair | The same mechanism can be expressed in many instruments |
| Horizon | Ultra-high frequency; intraday; swing; multi-month; strategic | Momentum is not necessarily shorter than trend following: horizons overlap |
| Execution | Maker, taker, limit, smart order routing, VWAP/TWAP, RFQ, delta hedge | Execution often determines whether a theoretical edge survives costs |
| Decision policy | Threshold; optimization; stochastic control; heuristic; RL; discretionary overlay | AI, ML and genetic algorithms do not themselves create a risk premium |

### Economic mechanisms and candidate strategy families

1. **Persistence / positive feedback:** trend following, time-series momentum, cross-sectional momentum, breakout continuation, news continuation, positioning momentum, swing trends.
2. **Reversion / convergence:** mean reversion, pairs, cointegration, relative-value spreads, dislocations, gap filling, calendar spread reversion when economically justified, overextension reversals.
3. **Carry and compensated risk bearing:** commodity roll/carry, interest-rate term premia, FX carry, credit spread, variance/volatility insurance premium, short convexity with tail liability.
4. **Liquidity provision / microstructure:** market making, quote placement, queue management, rebates, spread capture, adverse-selection management, order-book reaction, statistical arbitrage at short horizons.
5. **Information and catalysts:** fundamentals, macro changes, earnings, sentiment, news, flows, predictable demand and supply, event-driven repricing.
6. **Surface, basis and structural relative value:** option skew/term structure, cross-asset basis, futures versus cash, funding basis, locations/grades, basket replication, borrow/funding dislocations.
7. **Portfolio construction / overlays:** risk parity, vol targeting, hedging, cross-factor diversification, tail hedges, inventory constraints, dynamic leverage — improve implementation, not automatically alpha.
8. **Adaptive modeling and search:** AI forecasting, Bayesian filtering, regime-dependent policies, reinforcement learning, evolutionary optimization — methods used to estimate or choose among economic exposures.

#### Corrections to preserve

- Momentum can reflect herding, under-reaction, slow information diffusion, flows, risk premia, and microstructure. Herd mentality is one candidate explanation, **not a proven universal cause**.
- Momentum can be minute-based, monthly, or multi-year; trend following and momentum are often overlapping descriptions. **A weakening trend does not guarantee mean reversion**.
- Breakouts can represent momentum or event-driven repricing; failed breakouts can become reversion setups.
- Swing trading labels a **holding horizon**, not uniquely a trend-following strategy. Swing trades may fade, follow, or hedge a trend.
- HFT/scalping/order-book trading describe **latency or data granularity**; market-making is a business model with spread capture and inventory risks; ultra-high-frequency is a subset of high-frequency activity.
- Futures and options calendar spreads can trade *carry, volatility, storage scarcity, forward volatility, seasonality or reversion* — never assume all calendar spreads should converge on your horizon.
- Volatility can be traded via options/variance derivatives or volatility-linked products, but a VIX-linked product does **not** provide simple direct ownership of spot VIX; futures curve and roll exposure matter.
- Selling naked options creates potentially very large losses; a naked call has theoretically unbounded underlying-price exposure. Appropriate research starts with defined-risk positions and exact margin/liquidation modeling.

## 1. Strategy map by style and asset class

| Broad label | Main actual mechanisms | Useful controls |
|---|---|---|
| Momentum / herd / breakout | Trend/continuation, flows, event repricing | Volume persistence, exhaustion, volatility, liquidity |
| Mean reversion / pairs / spread | Relative value, equilibrium restoration, transient dislocation | Stationarity, half-life, stability of reference mean |
| Swing | Holding period; either persistence or reversal | Stop, time exit, state transition |
| Scalping / order book / HFT | Short-horizon alpha, liquidity provision, execution | Queue position, spread, fees, impact, fill realism |
| Market making / UHFT | Spread/fees/rebates, optional inventory premia | Adverse selection, volatility, quote stale time |
| Fundamental / macro | Information, value, risk premium, regime repricing | Release chronology, publication revisions, lags |
| Options, spreads, structures | Volatility, convexity, carry, implied correlations, directional exposure | Greeks, margin, term structure, tail stress |
| Bonds and rates derivatives | Carry/roll, duration, curve, spreads, macro | DV01, convexity, credit, funding |
| AI-driven | Depends on underlying signals/objective and execution | Leakage, baseline comparison, stability |

### 1.1 Breakout versus reversion at the same event

Define a signed excursion from reference `m_t` as `e_t = p_t - m_t`. The **same threshold crossing** can produce opposite hypotheses:

- Trend: `sign(E[r_{t,t+h} | e_t > c, state]) > 0` and persistence beyond costs.
- Reversion: `sign(E[r_{t,t+h} | e_t > c, state]) < 0` and a stable reference level exists.
- Neutral: expected move is indistinguishable from zero; costs dominate.

A regime engine should **measure conditional outcomes**, not select whichever narrative sounds appealing.

## 2. Options — building blocks and precise payoff algebra

Use `C_K` = long European call at strike `K`; `P_K` = long European put at strike `K`, both with common maturity `T`, contract multiplier `m` where relevant. Initial premiums **must be accounted for separately**. For terminal underlying `S_T`:

\[
(C_K)_T=(S_T-K)^+,\quad(P_K)_T=(K-S_T)^+,
\]
\[
(-C_K)_T=-(S_T-K)^+,\quad(-P_K)_T=-(K-S_T)^+.
\]

**Terminal profit** = terminal payoff minus opening debit (or plus opening credit), minus commissions, financing and relevant hedging costs. Portfolio construction uses signed sums of option payoffs.

### 2.1 Position inventory

| Structure | Signed position | Economic exposure |
|---|---|---|
| Long call | `+C_K` | Positive delta, long convexity; max premium loss |
| Short call | `-C_K` | Negative delta, short convexity; naked position can lose without upper bound |
| Long put | `+P_K` | Negative delta, long convexity; max premium loss |
| Short put | `-P_K` | Positive delta, short convexity, substantial downside risk |
| Long straddle | `+C_K + P_K` | Long volatility/convexity; needs sufficiently large realized move to pay premium |
| Short straddle | `-C_K - P_K` | Short volatility/convexity; short crash and upside jump risk |
| Long strangle | `+P_{K_1} + C_{K_2}`, `K_1<K_2` | Long tails with a central dead zone |
| Short strangle | `-P_{K_1} - C_{K_2}` | Premium collection with substantial tail liability |
| Covered call | `+S - C_K` | Long stock with upside capped; downside remains |
| Cash-secured put | `-P_K + cash reserve` | Downside equity purchase exposure, capped premium income |
| Bull call debit spread | `+C_{K_1} - C_{K_2}` | Bounded bullish payoff, `K_1<K_2` |
| Bear call credit spread | `-C_{K_1} + C_{K_2}` | Bounded bearish payoff, `K_1<K_2` |
| Bear put debit spread | `+P_{K_2} - P_{K_1}` | Bounded bearish payoff, `K_1<K_2` |
| Bull put credit spread | `-P_{K_2} + P_{K_1}` | Bounded bullish payoff, `K_1<K_2` |
| Long call butterfly | `+C_{K_1}-2C_{K_2}+C_{K_3}` | Peak near middle strike for equal wing widths |
| Short call butterfly | `-C_{K_1}+2C_{K_2}-C_{K_3}` | Inverse of long butterfly |
| Long call condor | `+C_{K_1}-C_{K_2}-C_{K_3}+C_{K_4}` | Plateau around middle strikes; wings bounded |
| Short call condor | `-C_{K_1}+C_{K_2}+C_{K_3}-C_{K_4}` | Inverse of long condor |
| Iron condor | Short OTM put spread + short OTM call spread | Defined-risk, short range/vol exposure, depends on strike ordering |
| Calendar spread | Long one maturity and short another at selected strike | Term structure, theta/vega and path-dependent mark-to-market |
| Diagonal spread | Cross-maturity and cross-strike | Joint term/strike risk |

**Label caveat:** 'long' and 'short' condor or butterfly names sometimes refer to **premium/debit** or **volatility exposure** in practitioner shorthand. Always specify legs and strikes. Equal-width wings are needed for a textbook symmetric butterfly; condors need not use equal widths.

### 2.2 Reverse engineering a payoff from a desired slope curve

Any continuous piecewise-linear payoff on `S_T` can be constructed from cash, underlying, and calls at strike knots:

\[
G(S_T)=a+bS_T+\sum_{j=1}^J\Delta b_j (S_T-K_j)^+,
\]

where `Δb_j` is the *change in desired slope* at strike `K_j`. For knot set 60, 70, 100, 110, 120, determine successive slope changes and use that many signed calls. With only calls and no stock, the pre-first-strike payoff slope must be zero. Distinguish **payoff shape** from the **profit** curve after initial premium and financing.

### 2.3 Put–call parity (European, no dividends)

\[
C_t(K,T)-P_t(K,T)=S_t-Ke^{-r(T-t)}.
\]

With constant continuous dividend yield `q`, replace `S_t` by `S_t e^{-q(T-t)}`. General forward form:

\[
C-P=D(t,T)(F_{t,T}-K).
\]

Under the no-dividend, positive-rate assumptions, a larger `r` increases `C-P`; a larger strike decreases it; a larger spot increases it. Changing time to expiry requires specifying the discount and dividends — don't turn it into an unqualified monotonic statement. This identity applies to **European** same-expiry/strike contracts, not generally as exact equality to American options. Source: [Options Industry Council](https://www.optionseducation.org/advancedconcepts/put-call-parity).

### 2.4 Discrete option valuation exercise

Terminal states `S_T ∈ {80,100,120}` with probabilities `{1/4,1/2,1/4}` yield physical-expectation payoff:

\[
E[(S_T-K)^+]=\tfrac14(80-K)^++\tfrac12(100-K)^++\tfrac14(120-K)^+.
\]

**Do not automatically discount physical expectations and call them option prices.** Arbitrage-free valuation requires risk-neutral/state-price probabilities consistent with the traded underlying, discount rate and market completeness. The given probabilities may be physical rather than pricing probabilities.

### 2.5 Greeks and risk decomposition

For an option portfolio `V(S,σ,t,r,...)`, a small-change expansion is:

\[
dV\approx\Delta\,dS+\tfrac12\Gamma(dS)^2+\nu\,dσ+\Theta\,dt+\rho_r\,dr+\text{cross terms}.
\]

Track delta, gamma, vega, theta, vanna, volga, correlation, carry, skew, and bid/ask execution. `Delta-neutral` does not mean `risk-free`.

## 3. Statistical distribution notes, estimation and portfolio utility

### 3.1 Student-t and non-normal returns

If `Z~N(0,1)` and `U~χ²_ν` are independent, `T=Z/sqrt(U/ν)~t_ν`. For the **standard, centered** Student-t:

- Variance is `ν/(ν−2)` if `ν>2`; undefined for smaller degrees of freedom.
- Symmetry gives skewness zero when defined (`ν>3`).
- Ordinary kurtosis equals `3+6/(ν−4)` for `ν>4`; excess kurtosis equals `6/(ν−4)`.
- Fat tails make Gaussian VaR, confidence intervals and Sharpe-based inferential approximations fragile.

Noncentral chi-squared emerges from squared sums of shifted independent normal variables, with noncentrality `λ=Σμ_i²` when each has unit variance.

**Sample-mean/sample-variance independence** is special to normal observations; do not presume it for heavy-tailed, autocorrelated trading returns.

### 3.2 Confidence intervals under realistic time-series dependence

The provided DAX-2017 exercise reports an illustrative 95% daily-mean interval of approximately `[-0.03%,0.13%]`. Recalculate before quoting it as an independently verified estimate. IID t-intervals are not automatically valid for serially dependent, heteroskedastic daily or high-frequency returns; use HAC errors or block bootstrap where warranted.

### 3.3 Exponential utility and mean-variance objective

For wealth `W` and risk aversion `λ>0`, take `U(W)=-e^{-λW}`. If **terminal wealth** is Gaussian `W~N(μ,σ²)`, then:

\[
E[U(W)]=-\exp[-λμ+\tfrac12λ²σ²],
\]

so maximizing expected exponential utility is equivalent to maximizing:

\[
\boxed{\mu-\tfrac λ2σ^2}.
\]

This equivalence requires the assumptions. Non-Gaussian tails, ruin constraints, leverage, and nonlinear payoffs change the economics.

## 4. Portfolio drivers, projective features, systemic PDEs and sell decisions

### 4.1 Portfolio *representation* versus portfolio *decision*

A candidate representation chain is `prices → returns/volatility → factors/exposures → optimized weights`. Different feature spaces preserve different characteristics:

- Distribution shape (tail risk, skew, volatility clustering).
- Trajectory structure (path, path signature, directional change).
- Common factors (market, rates, growth, liquidity, sector).
- Networks (cross-asset coupling, shock propagation).
- Causal candidates (identification via instruments or exogenous changes).

A 'projective' interpretation of finance must state its map, geometry, invariants, and empirical advantage; the slides alone do not establish a new mathematical portfolio theory.

### 4.2 PDE / structural dynamics as a proposal, not a solved model

An implicit model `F(t,y,∂_t y,x_i,∂_{x_i}y,...) = 0` must specify states, boundary/initial conditions, regularity, observability, numerical method and stability. A generic PDE written on a slide is not yet a calibrated price generator. The hypotheses 'drivers optimality' and 'commonality of drivers' need operational definitions and tests against simple factor models.

### 4.3 Sell signals

Entry and exit are distinct predictive tasks. Research exits under *fixed time*, *fixed loss*, *volatility stop*, *signal reversal*, *regime change*, *trailing stop*, *profit-taking*, and *optimal stopping* policies. A high win rate can coexist with negative expectancy if exits create rare large losses. Equity shorting introduces borrow, recall and asymmetric financing constraints.

## 5. Directional-change (DC) intrinsic time and overshoots

Directional changes measure market activity in **price-move events** rather than evenly spaced clock time. For threshold `δ>0`, track a running extreme and change direction only after a counter-move of size `δ` from that extreme. The continuing movement after a DC confirmation before the next reversal is an **overshoot**.

### 5.1 Causal event algorithm (percentage threshold)

```python
mode = 'up'                 # one arbitrary initial state
extreme = price[0]
for t, p in enumerate(price[1:], 1):
    if mode == 'up':
        extreme = max(extreme, p)
        if (extreme - p) / extreme >= delta:
            emit(t, event='down_DC', old_extreme=extreme)
            mode, extreme = 'down', p
    else:
        extreme = min(extreme, p)
        if (p - extreme) / extreme >= delta:
            emit(t, event='up_DC', old_extreme=extreme)
            mode, extreme = 'up', p
```

**Critical:** The previous peak/trough is only confirmed **after** the reversal threshold is met. Retrospective assignment of events to the extremum bar creates look-ahead leakage. Report DC event times, overshoot magnitude, elapsed physical time, event frequency, and time since threshold confirmation. Source: [Glattfelder & Olsen, Theory of Intrinsic Time (2024)](https://arxiv.org/abs/2406.07354) and [Directional-change agent-based model (2019)](https://doi.org/10.1080/14697688.2019.1669809).

### 5.2 Intrinsic-time research program

For `δ ∈ {0.1%,0.2%,0.3%,0.5%,1%}` (illustrative, calibrate from asset volatility), test:

- Mean and distribution of overshoot/threshold ratio, not only average.
- Waiting time `E[Δt|δ,state]`, event intensity `N(δ)`, time-of-day variation.
- Trend/reversion after `up_DC` and `down_DC` confirmations, by regime.
- Whether event-space features outperform equally costly clock-time features.
- Noise thresholds in relation to spreads, realized volatility and fees.
- Non-overlapping train/test windows and execution immediately **after** confirmation.

DC event timing can be a *state detector*, but it need not create alpha.

## 6. Market clearing, investor demand and factor regressions

A candidate demand model is:

\[
\Delta q_{i,t}=-G_i\Delta p_t+\lambda_t\eta_t+u_{i,t}.
\]

Impose weighted net demand/supply balance:

\[
\sum_i S_i\Delta q_{i,t}=0.
\]

For compatible dimensions and stable positive inverse-demand loadings, solve:

\[
\Delta p_t=\frac{\sum_i S_i(\lambda_t\eta_t+u_{i,t})}{\sum_i S_iG_i}
=\beta_t\eta_t+\epsilon_t.
\]

This illustrates how reduced-form factor regressions can arise from aggregate demand, but **does not by itself identify a causal price-impact coefficient**. To recover demand slopes from variation in holdings or supply, require valid instruments/exogenous shifts, exclusion restrictions, timing, and error structure. Cross-investor `E[u_i u_j]=0` by itself is not sufficient to establish instrumental validity.

**Test:** Compare contemporaneous regressions with causal lead/lag regressions; use external instrument or natural experiment if making a causal claim. Account for correlated investor shocks and endogenous order placement.

### 6.1 Liquidity/safe haven regime change

The supplied slides hypothesize changes in dealers, foreign purchasers and public central-bank backstops since the late 1990s/2012. Treat as a **period- and asset-specific historical thesis**, not an established universal fact. Measure dealer balance-sheet capacity, Treasury bid/ask, depth, market impact, official purchases, inflation sensitivity and stock–bond correlation on common windows.

## 7. Macroeconomic fair-value models and cross-asset relative value

A generic fair-value factor model:

\[
\hat P_t=\beta_0+\sum_{j=1}^m\beta_j x_{j,t},\quad e_t=P_t-\hat P_t.
\]

Illustrative WTI variables from notes: CADUSD, energy equity ETF XOP, inflation breakevens, oil-specific inventory, rates, demand measures. The listed price-regression model can be spurious if variables are nonstationary and not cointegrated. Prefer returns/changes or validated cointegration/error-correction specifications.

### 7.1 Fair-value trade proposal

\[
\pi_t=\begin{cases}
+1& e_t<-\varepsilon_t,\\
-1& e_t>\varepsilon_t,\\
0&\text{otherwise}.
\end{cases}
\]

Choose `ε_t` based on forecast uncertainty + implementation costs. Position sizes depend on volatility and confidence, not just sign. Add **publication lag** for fundamentals and macro revisions.

### 7.2 Cross-asset examples from notes

- **Commodity FX:** WTI/CAD, Brent/NOK, copper/CLP, coal/AUD (economic dependence is time-varying; control broad USD and risk sentiment).
- **Oil equities:** XOP or hedger/non-hedger basket with beta, hedging and sector exposures.
- **Refined products / inflation:** RBOB gasoline versus CPI swaps; account for refining, retail pass-through, taxes, weights, delivery, timeliness. Claim that gasoline drives `>70%` of CPI variance requires a stated country, sample and decomposition and is **not verified here**.
- **Spot vs futures:** Basis, funding, carry, convenience yield, calendar-spread structure, inventory signal.

**Reject:** naive regressions whose residual reverts only in the training sample.

## 8. Volatility risk premium (VRP), insurance economics and regimes

Options transfer risks between hedgers/insurance buyers, inventory hedgers, dealers and speculators. A simplified variance premium measure is:

\[
VRP_t=IV_t^2-E_t^P[RV_{t,T}],
\]

with `IV` and expected realized variance over the **same horizon**, properly standardized. Some academic literature uses the opposite sign, `E^P[RV]-E^Q[RV]`. Always write the definition.

### 8.1 Long/short option carry and hedging

For a delta-hedged **long** option, a schematic continuous hedge P&L is:

\[
\Pi_{long}\approx \frac12\int_0^T\Gamma_tS_t^2
(\sigma_{real,t}^2-\sigma_{impl,t}^2)dt
-\text{hedge/transaction costs}
\]

under restrictive assumptions, with rate/dividend/carry effects handled consistently. This is a heuristic decomposition, not a static exact formula for all option models.

- If implied volatility exceeds realized volatility, selling sufficiently hedged volatility can earn a premium **before tail events and costs**; holding short gamma creates exceptional loss risk.
- Delta hedging changes exposure; it does **not** guarantee a profitable buyer and seller, nor eliminate gamma/jump/vol-of-vol risk.
- Option payoffs are asymmetric and tail sensitive. Analyze VaR/ES, mark-to-market jumps, margin and liquidity stress; average Sharpe alone is inadequate.

Academic energy-options evidence reports time-varying risk premia and distinct call/put/maturity behavior; claims that *unhedged straddles always lose* or *delta-hedged straddles always gain* are **false as universal rules**. Source: [Jacobs & Li, Option Returns, Risk Premiums, and Demand Pressure in Energy Markets (2022)](https://doi.org/10.1016/j.jbankfin.2022.106687).

### 8.2 Proposed oil VRP subperiods from supplied presentation [UNVERIFIED]

- 2000–2014: insurance/hedging-heavy phase.
- 2014–2020: options financialization and shale producer impacts.
- 2021 onward: changing producer consolidation, retailer/dealer participation, call/put premium ordering.

Do not treat the period boundaries or reported Sharpe >1 as established without original data, contract selection, premium definitions, hedge rules and publication source. Test structural breaks (sup-Wald/Bai-Perron, rolling priors), survivorship, option selection and historical quote quality.

## 9. Quadratic Normal Model (QNM) and volatility surface trading

A local-volatility option pricing equation for forward `F` can be written schematically as:

\[
\partial_t V+\tfrac12 \sigma_{loc}(F,t)^2\partial_{FF}V=0
\]

when working in a normalized forward measure and suppressing carry/discount terms, with terminal payoff. The supplied notes describe a **perturbation around a normal model** with quadratic shape `ε(F)=a+bF+cF²`. The precise coefficient definitions and expansion order depend on the original QNM derivation and should not be invented from the summary.

Calibration targets: ATM straddle, OTM strangle, costless collar, selected strikes/maturities. Compare to Bachelier, Black-76, SABR (if appropriate), shifted-lognormal, interpolated implied volatility. Require static no-arbitrage checks: call price decreasing and convex in strike, monotonic total variance where appropriate, and reasonable wings.

### 9.1 Volatility portfolio modules

| Sleeve | Core return driver | Key risks |
|---|---|---|
| Gamma / delta-hedged volatility | Implied-realized difference, dynamic hedging | Gap losses, funding, realized gamma costs |
| Vega / smile | Relative richness across strikes, sticky-delta vs sticky-strike | Surface model error, skew change |
| Term structure / calendar | Forward variance and maturity roll | Event jump, forward vol change |
| Crude / refined cross-product | Correlation and relative implied/realized vol | Basis, correlation breakdown |
| Correlation / dispersion | Implied vs realized covariance | Joint crash, hedge error |
| Exotic/quanto/digital | Path and cross-currency contingent payoff | Model dependence, liquidity, nonlinear Greeks |

Calendar spread options (CSOs), Asian price options (APOs), swaptions, extendible options and digitals require explicit payoff conventions and data availability; names alone are insufficient.

## 10. Commodity storage, carry, momentum and squeezes

### 10.1 Theory of storage

Stylized arbitrage relationship when cash-and-carry is feasible:

\[
F(t,T)=S_t\exp[(r+c-y)(T-t)],
\]

where `c` is cost of storage/finance (consistent units) and `y` is convenience yield. Low inventories can increase convenience yield and induce backwardation; high inventory/capacity constraints can create steep contango. These are **states of the forward curve**, not mechanical trades.

- **Contango:** futures price typically higher at longer maturities, depending on precise contract pair; can reflect storage/financing/carry.
- **Backwardation:** front is higher than further dated futures, often compatible with scarcity/convenience yield, but also risk premia and seasonality.
- **Carry is not simply positive expected profits.** Roll yield and holding-period total return differ; risk-premium compensation and term-structure changes matter.

Strong sources: [Gorton, Hayashi & Rouwenhorst, The Fundamentals of Commodity Futures Returns](https://www.nber.org/papers/w13249), [Federal Reserve, Revisiting the Forecasting Potential of Futures Prices (2026)](https://www.federalreserve.gov/econres/notes/feds-notes/revisiting-the-forecasting-potential-of-futures-prices-20260903.html) and [Koijen et al., Carry](https://doi.org/10.1016/j.jfineco.2017.11.002).

### 10.2 Carry vs momentum

Let `F_{t,M}` and `F_{t,N}` be different maturities. Raw difference `C_t=F_{t,M}-F_{t,N}` is an initial *term spread*, but comparable carry requires accounting for tenor, unit, roll convention, and funding. Momentum `M_t=P_t-MA_n(P)` is a different signal.

Carry tends to be a slower structural signal; momentum may adjust quicker to new market states. They can agree, disagree or be combined with dynamic weights. The claim `WTI carry 19.7% annualized over 30 years` from the user's slide notes is **not independently reproduced**.

### 10.3 Commodity storage squeeze from boundary-constrained inventory

Stylized latent inventory:

\[
dX_t=\kappa(\bar X-X_t)dt+\sigma dW_t,\quad X\in[0,1].
\]

Physical constraints require a **boundary treatment** (reflection, absorbing state, controlled drift, or nonlinear scarcity penalty). Simply writing an unconstrained OU equation and declaring stockouts/tank-tops is mathematically insufficient. Define spread `f(x,t)=S(x,t)-F(x,t)` and terminal/boundary squeeze payoffs; solutions may involve PDE first-passage density or Green's functions. Dirac delta distributions are mathematical idealizations and do not directly represent observable futures prices.

Proposed test: does inventory scarcity (or its reliable proxy) explain asymmetric spot-futures basis extremes beyond carry/seasonality? For crypto, order-book capacity and exchange balances are **analogies**, not the same physical storage mechanism.

## 11. CTAs, trend, commodities history and positioning

### 11.1 Historical narrative [TO VERIFY]

The supplied lecture describes Commodities Corp, Helmut Weymar and foundations of systematic macro/commodity trading. The specific claim that trend following *originated* as a drawdown overlay at this firm is too strong without primary historical evidence; trend trading predates particular funds. Treat its historical/ownership assertions as reading leads rather than verified facts.

### 11.2 Reverse engineering behavior from flows / positions

Think of a fund or CTA as a latent signal-processing system:

\[
q_{i,t}=f_i(r_{t-1},...,r_{t-L};\,\sigma_t,\text{constraints})+\epsilon_{i,t}.
\]

A linear filter or shallow neural-network approximation can estimate holdings from price histories. A published study reports that regularized trend regressions explain **over 40% of speculative position changes on average in its 23-commodity sample** — not evidence that a universal one-layer NN explains exactly 50% of hedge-fund holdings or that the remainder is purely discretionary. Source: [Tracking speculative trading](https://www.sciencedirect.com/science/article/pii/S1386418122000635).

**Identification issue:** observed aggregate holdings plus prices do not uniquely determine funds' trading policies. Different agents, risk constraints and feedback loops can generate similar net positions. Use CFTC/COT and reported positioning only with publication lag and account for reporting categories.

### 11.3 Follow-the-flow versus fade-the-crowded-trade

**Follow-the-flow:** `signal_t=sign(Δpositioning_t)` subject to price momentum and liquidity confirmation. **Fade crowding:** reverse exposure only when positioning is extreme *and* catalysts or price/flow evidence indicate exhaustion. An extreme z-score alone does not force reversion.

Use dynamic thresholds on the historical distribution of positioning, correct release times, and compare both to naive momentum and carry baselines.

## 12. Reaction functions and signal blending

Let `x_t` be a standardized forecast strength and `R(x_t)` a target exposure. A nonlinear candidate supplied in notes:

\[
R(x)=x\exp\{\tfrac12(1-x^2)\}.
\]

Its derivative is `R'(x) = exp((1-x²)/2) * (1-x²)`, so its magnitude rises until `|x|=1`, then **decreases for larger |x|**; the intention is to de-risk extreme signals that may reverse. It is **not** automatically optimal without a calibrated conditional expected return / downside model.

Compare against linear clipping `R(x)=clip(cx,-1,1)`, `tanh(cx)`, threshold sizing, volatility targeting, and Kelly-fraction/capped utility sizing.

### 12.1 Carry/momentum blend

Let `C_t` be a comparable carry score and `M_t` a momentum score, then:

\[
z_t=w_C(t)\tilde C_t+w_M(t)\tilde M_t.
\]

`w(t)` may be fixed, shrinkage-optimized or regime-conditioned using only past data. Alternative sign rule from notes: `π_CM(F_t)=sign(C_t-MA(C_t))`; this is **carry-momentum**, not the same as blending raw carry and return momentum. Compare both.

## 13. Fixed income, macro and rate derivatives research block

Mechanisms: duration/rate expectations, yield-curve carry and rolldown, term premium, inflation breakevens, credit spread, liquidity premium, convexity and optionality. Instruments: nominal bonds, TIPS/linkers, futures, swaps, swaptions, caps/floors, curve spreads and options.

- `ΔP/P≈−D_mod Δy+ 1/2 Convexity(Δy)²` for small yield changes; account for duration vs DV01 units.
- Curve-steepener/flatteners require DV01-neutral sizing; a spread in yields alone does not create a market-neutral trade.
- Bond return = carry + rolldown + yield move + credit/FX/funding effects where relevant.
- Interest-rate derivatives have collateral, day-count, tenor, OIS discounting and margin details absent in a spot crypto engine.

## 14. Fundamental, macro, information and AI strategies

**Fundamental/macro:** use pre-announced observations, consensus surprises, revisions, seasonal releases, real-time vintages, valuation anchors and structural models. **News sentiment:** can be momentum, reversal or information-driven depending on subsequent path; timestamp source, dissemination and source latency, not just published article time. **AI-driven:** compare autoregression, linear/ridge, tree ensembles, sequence networks, transformer foundations, agent teams and RL policies on *identical* out-of-sample data and costs.

Suggested performance targets: prediction calibration, directional/probabilistic scores, signal-to-trade conversion, gross and net PnL, tail concentration and regime stability. Eliminate narrative labels if there is no measurable gain.

## 15. End-to-end research cases from this intake

| ID | Research question | Necessary data | Primary benchmark | Falsification |
|---|---|---|---|---|
| S01 | Price persistence vs post-extreme reversal? | OHLCV, regimes, costs | Time-series momentum vs random/sign-flip | No incremental net edge |
| S02 | Does DC intrinsic time improve entry timing? | Tick or high-resolution bars | Fixed-time trend/reversion | No out-of-sample uplift |
| S03 | Is fair-value residual stationary and tradable? | Vintage macro + prices | Return regression / naive spread | Nonstationary OOS, fee drag |
| S04 | Is commodity carry premium distinct from momentum? | Continuous futures, rolls, storage | 12m TSMOM, simple term spread | No net marginal improvement |
| S05 | Are flow extremes followed by reversals? | COT/positions + timestamp | Momentum continuation | Extremes persist or become untradable |
| S06 | Does implied minus expected realized variance pay? | Options chains + realized vol | No-trade, delta-hedged buy-and-hold | Negative costs/tail-adjusted value |
| S07 | Does nonlinear reaction function improve sizing? | Stable signal + pnl | Linear clip, tanh, vol target | Utility/ES worsens |
| S08 | Can option payoffs be replicated precisely? | Same-expiry chains, funding | No-arbitrage parity | Residual within bid-ask; violations only due to costs |
| S09 | Does storage scarcity explain oil basis? | Inventory, front/second futures | Simple carry + seasonal model | No incremental held-out fit |
| S10 | Can a calibrated PDE predict observations? | Named system, state, parameters | Linear factor model | Poor predictive quality/identifiability |
| S11 | Are hedge-fund positions explainable by past returns? | Reported positions / prices | Ridge linear filter | Held-out R² ≤ baseline |
| S12 | Do VRP regime breaks survive re-estimation? | Option quote history, hedging | Constant VRP estimate | Instability/non-significance |

## 16. Evidence and claim audit — important corrections

1. **Straddles:** neither delta hedging nor a volatility-risk-premium argument makes both long and short sides profitable; sign, funding and hedge costs matter.
2. **Calendar spreads:** no automatic mean reversion, regardless of market.
3. **Options implied volatility:** not the same as realized volatility or a VIX future price.
4. **Put–call parity:** use consistent European exercise, dividends, strike, expiry, funding and discount factor.
5. **Regression with macro price levels:** spurious regression is a serious danger; causal interpretation needs identification.
6. **Market clearing:** simple aggregate demand algebra is not a causal proof.
7. **Distribution tails:** t distribution moments exist only above stated degree-of-freedom thresholds; inferential formulas need dependence-aware standard errors.
8. **CTA reverse engineering:** aggregate positioning is not individual trading policy; the screenshot's '50% NN' figure remains unverified.
9. **Oil carry/VRP performance:** historical annualization, Sharpe and period boundaries are not verified from presentation notes alone.
10. **Squeeze PDE:** boundaries and delta terminal conditions require precise mathematical definition.
11. **Directional-change extremum:** it is not causally confirmed until the reversal threshold triggers.
12. **Quant classification:** a strategy may combine multiple mechanisms; taxonomies are practical research maps, not mutually exclusive laws.

## 17. Curated bibliography

- Moskowitz, Ooi and Pedersen (2012), [Time Series Momentum](https://doi.org/10.1016/j.jfineco.2011.11.003).
- Koijen, Moskowitz, Pedersen and Vrugt (2018), [Carry](https://doi.org/10.1016/j.jfineco.2017.11.002).
- Gorton, Hayashi and Rouwenhorst, [The Fundamentals of Commodity Futures Returns](https://www.nber.org/papers/w13249).
- Federal Reserve (2026), [Revisiting the Forecasting Potential of Futures Prices](https://www.federalreserve.gov/econres/notes/feds-notes/revisiting-the-forecasting-potential-of-futures-prices-20260903.html).
- IMF (2010), [How Commodity Price Curves and Inventories React to a Short-Run Scarcity Shock](https://www.elibrary.imf.org/view/journals/001/2010/222/article-A001-en.xml).
- Options Industry Council, [Put/Call Parity](https://www.optionseducation.org/advancedconcepts/put-call-parity).
- Jacobs and Li (2022), [Option Returns, Risk Premiums, and Demand Pressure in Energy Markets](https://doi.org/10.1016/j.jbankfin.2022.106687).
- Glattfelder and Olsen (2024), [The Theory of Intrinsic Time](https://arxiv.org/abs/2406.07354).
- [Agent-based modelling in directional-change intrinsic time (2019)](https://doi.org/10.1080/14697688.2019.1669809).
- [Tracking speculative trading](https://www.sciencedirect.com/science/article/pii/S1386418122000635).

## 18. Final operating discipline

A research claim must be written as **mechanism → mathematical signal → conditional probability/expectation → executable action → realized cost and risk → out-of-sample result**. Never infer net profitability from an appealing story, high gross Sharpe or an elegant PDE alone.
