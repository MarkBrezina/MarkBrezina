# Algorithmic Trading Research Compendium

Prepared for Mark Brezina • Review date: 7 November 2026

Nine research papers/notes, corrected mathematics, implementation proposals and an explicit source-access record. No new backtests were run. See the source register for review depth.

# Paper 1 — Corrected foundations and time-series modelling

## Purpose and reading convention

This compendium turns the supplied material into a research reference for algorithmic trading. It combines source summaries, corrections, mathematical explanations, and proposed experiments. These are review notes, not newly peer-reviewed research papers. No strategy was backtested for this review, and no reported source result is presented as a return you should expect to earn.

References such as [L01] identify entries in the source register. Local PDF references use **PDF page numbers** unless explicitly described as printed pages or slide numbers. The register distinguishes full text availability from the depth of review: extracting every page is not the same as checking every proof, equation, implementation, and reference. Directories were reviewed as directories; their entire archives were not recursively reviewed.

The most useful separation is between four questions: What is the market doing? What happens next? What position should be held? How can that position be traded? A volatility estimate answers the first question. A return forecast addresses the second. An allocation rule addresses the third. An execution model addresses the fourth. Confusing these roles is a recurring source of misleading backtests.

## 1.1 A mathematical vocabulary for the project

Let F_t denote the information genuinely available at decision time t. A trading decision must be measurable with respect to F_t. This includes more than chronological row order: an exchange event may have an earlier timestamp but arrive after the decision; an economic series may later be revised; a candle's high and low are unknown until that candle finishes.

Write simple returns as R_t = P_t/P_(t-1) − 1 and log returns as r_t = log(P_t/P_(t-1)). Log returns add over time. Portfolio simple returns aggregate linearly using beginning-of-period weights, before accounting for cash flows and costs. A weighted average of asset log returns is generally not the exact portfolio log return. State the convention before reporting volatility, cumulative returns, or portfolio performance.

The conditional mean m_t = E[R_(t+1) | F_t] and conditional variance h_t = Var(R_(t+1) | F_t) are different prediction targets. Returns may have little linear autocorrelation while squared returns remain strongly predictable. A model can forecast risk usefully while offering no directional edge.

Weak or covariance stationarity means a constant finite mean and an autocovariance depending only on lag. Strict stationarity concerns invariance of the entire joint distribution under time shifts. Neither implies independent observations. A stationary spread can have substantial serial dependence, changing conditional volatility, and long stretches far from its mean.

White noise usually means zero mean, constant finite variance, and zero autocovariance at nonzero lags. It need not be Gaussian, independent, or identically distributed in every higher-order property. Gaussian white noise adds joint Gaussianity, which makes uncorrelated components independent. The repeated statement in the pasted notes that white noise is simply a normal random variable is therefore too restrictive. [L01, L08, user-supplied AR/MA notes]

Linear algebra enters through regression, covariance estimation, principal components, and portfolio optimisation. Positive semidefinite covariance matrices satisfy wᵀΣw ≥ 0. In practice, pairwise missing-data estimates can produce an invalid matrix; repair or use a coherent joint estimator before optimisation. Numerical conditioning matters: solving a linear system is preferable to explicitly forming a matrix inverse.

Probability, stochastic calculus, optimisation, and machine learning belong in the same toolkit but solve different problems. An Itô model is a statement about a process; a neural network is a function class; an optimiser chooses a decision under an objective. None supplies the objective, data quality, or economic mechanism automatically.

## 1.2 Autoregressive models

A convenient stationary AR(p) parameterisation centres the process:

$$
X_t-\mu=\sum_{i=1}^{p}\phi_i(X_{t-i}-\mu)+\epsilon_t.
$$

Equivalently, X_t = c + Σ φ_i X_(t−i) + ε_t, with c = μ(1 − Σφ_i). Omitting the intercept implicitly imposes a zero mean unless the data were centred. Define the backshift operator B X_t = X_(t−1), and φ(z) = 1 − Σφ_i z^i.

The usual **causal stationary** solution exists when every root of φ(z) lies outside the unit circle. The word causal matters: mathematical stationary noncausal solutions can sometimes be constructed using future innovations, but they are unsuitable for real-time forecasting. An AR(1) has a causal stationary solution for |φ| < 1, variance σ²_ε/(1−φ²), and autocorrelation ρ(k) = φ^|k|.

For distinct characteristic roots z_j, the AR autocorrelation can be expressed as a linear combination of z_j^(−|k|). Coefficients are determined by the covariance equations and initial conditions. Real roots contribute decaying terms; complex-conjugate roots can generate damped oscillations. Repeated roots introduce polynomial factors in lag. Thus the pasted expression captures an important pattern but needs assumptions and consistent notation; its coefficients are not simply the AR coefficients.

For centred data, the Yule–Walker equations are:

$$
\gamma(k)=\sum_{i=1}^{p}\phi_i\gamma(k-i),\quad k\geq1,
$$

$$
\sigma_\epsilon^2=\gamma(0)-\sum_{i=1}^{p}\phi_i\gamma(i).
$$

Writing Γ_(ij) = γ(i−j), solve Γφ = (γ(1), …, γ(p))ᵀ. Yule–Walker is **one estimator**, not a compulsory procedure. Conditional least squares, maximum likelihood, Burg methods, and regularised regression are alternatives. Their small-sample behaviour and treatment of initial observations differ.

For trading, a lagged-return AR is a useful baseline and a diagnostic of weak serial dependence. An AR on a stable economic spread can describe reversion. An AR on raw prices may largely learn persistence. A very accurate next-price prediction can therefore coexist with a useless next-return forecast. Always compare with the random-walk prediction P̂_(t+1) = P_t and with a zero-return forecast. [L01, chapters 4, 6–9]

## 1.3 Moving-average and ARMA models

An MA(q) process is a finite filter of innovations:

$$
X_t=\mu+\epsilon_t+\sum_{j=1}^{q}\theta_j\epsilon_{t-j}.
$$

Set θ_0 = 1. With white-noise innovations, the finite MA process is covariance-stationary for any finite coefficient values. **Invertibility** is a separate property: roots of θ(z) = 1 + Σθ_j z^j must lie outside the unit circle to recover innovations through a stable causal filter. Stationarity concerns the process; invertibility concerns its representation.

For k between 0 and q, γ(k) = σ²_ε Σ_(j=0)^(q−k) θ_j θ_(j+k). For |k| > q, the population autocovariance is zero. Sample autocorrelations fluctuate around zero, so a visual cutoff is a clue, not an exact model-order oracle. MA errors are latent, and fitting commonly uses innovations algorithms, state-space likelihood, or nonlinear optimisation. It is too strong to say every possible estimation method must be iterative nonlinear fitting.

An ARMA(p,q) combines both filters:

$$
\phi(B)(X_t-\mu)=\theta(B)\epsilon_t.
$$

Use a consistent MA sign convention; packages and textbooks sometimes differ. Check that AR and MA polynomials have no common factors before treating the order as identifiable. In the population, the PACF cuts off after p for a pure AR(p), while the ACF cuts off after q for a pure MA(q). Mixed models usually have neither clean cutoff.

For angular frequency ω in [−π, π], one conventional spectral density is:

$$
f_X(\omega)=\frac{\sigma_\epsilon^2}{2\pi}\frac{|\theta(e^{-i\omega})|^2}{|\phi(e^{-i\omega})|^2}.
$$

If frequency is measured in cycles per observation, replace ω by 2πf and use the corresponding density normalisation. Mixing angular and cycle conventions creates factors of 2π. The spectrum describes how covariance is distributed across frequencies; a spectral peak is not by itself an exploitable periodic return. [L01, chapters 4, 13–14]

## 1.4 Differencing, seasonality, and exogenous information

ARIMA(p,d,q) ordinarily means that the **differenced** series has AR order p and MA order q:

$$
\phi(B)(1-B)^dX_t=c+\theta(B)\epsilon_t.
$$

The pasted factorisation of an original polynomial into unit-root and stationary components is legitimate, but its p′ notation should not be confused with the standard order argument used by software. For d = 1 and a stationary differenced process, the expected increment is c/(1−Σφ_i). Some software calls a regression coefficient a trend or drift rather than using this exact intercept convention. For d > 1, deterministic terms imply higher-order polynomial behaviour in levels.

Differencing can remove stochastic trends but can also over-difference an already stationary series, amplifying noise and inducing an MA-like pattern. Deterministic detrending and differencing are not interchangeable. A full-sample fitted trend uses information from the future when applied to earlier timestamps. That is especially relevant to cumulative-return detrending, rolling “equilibria,” empirical mode decomposition, and fitted seasonal components.

Seasonal ARIMA adds lag-s polynomials and possibly (1−B^s)^D differencing. Crypto has no exchange close in the equity sense, but hour-of-day, day-of-week, and cross-market session effects can still exist. Model those effects using time information actually known at the forecast origin. A local intraday seasonality estimate should be fitted on past days, not on the current day's completed profile.

ARMAX and regression with ARMA errors are related but distinct. In a dynamic ARMAX equation, lagged dependent variables and inputs appear directly. In regression with ARMA errors, X_t = βᵀZ_t + n_t and φ(B)n_t = θ(B)ε_t. Their coefficient interpretations and impulse responses differ. An “exogenous” variable is not necessarily statistically exogenous, and it is not necessarily available before the target. Future values must either be known by design, separately forecast, or excluded.

Vector autoregressions model multiple series jointly. Cointegrated nonstationary series can instead motivate a vector error-correction model. Similar price charts and high correlation do not establish cointegration. Pair selection, hedge-ratio estimation, and residual testing must occur inside each training window. Testing a residual formed using the whole sample contaminates the test period.

ARFIMA replaces integer differencing with fractional differencing and can represent long memory under appropriate parameter restrictions. Structural breaks and regime mixtures can mimic long memory. Multiscale autoregression on trees is a different indexing construction, not merely an AR model with more lags. These extensions deserve a specific hypothesis before adding complexity. [L01, chapters 5, 10–11; user notes]

## 1.5 Exponential smoothing, Fourier methods, and Wold decomposition

Simple exponential smoothing updates a level through l_t = αX_t + (1−α)l_(t−1), for 0 < α ≤ 1. It is a causal low-pass filter when computed recursively. Holt and seasonal extensions add evolving trend or seasonality. These methods supply useful baselines for volume, spread, and volatility components. Smoothing a price series does not automatically identify an equilibrium price or predict reversal around it.

Fourier analysis decomposes signals into oscillatory components. For financial data, examine sampling, aliasing, leakage from finite windows, detrending, and uncertainty in spectral estimates. A full-sample transform can be descriptive, but a trading feature must be computed only on the trailing window. Selecting the strongest frequency after seeing the entire test period is another form of strategy selection.

Wold's theorem states that a centred covariance-stationary process can be decomposed into a linear innovation component and a component perfectly predictable from its infinite past:

$$
Y_t=\sum_{j=0}^{\infty}\psi_j\epsilon_{t-j}+\eta_t,\qquad \sum_{j=0}^{\infty}\psi_j^2<\infty.
$$

With the usual normalisation ψ_0 = 1, the ε_t are one-step linear innovations. The deterministic component here is “deterministic” in the prediction-theoretic sense, not an arbitrary nonstationary deterministic trend. Gaussianity is unnecessary. The theorem is about linear representation and second-order structure; it does not say a short MA model captures nonlinear predictability, nor does it guarantee a useful trading forecast. The vague “minimum delay?” note should be replaced by this innovations-based interpretation. [L01, stationary models and spectral chapters]

## 1.6 Estimation and model selection

Least squares minimises squared residuals; maximum likelihood maximises a distributional model's likelihood. They coincide in some Gaussian models but are not interchangeable in general. Quasi-maximum likelihood can estimate conditional mean or variance parameters under weaker distribution assumptions, with suitable robust uncertainty estimates.

AIC = 2k − 2 log L̂ and BIC = k log n − 2 log L̂ penalise fitted likelihood by model complexity. Count all estimated parameters consistently. Compare models fitted to comparable observations and likelihood definitions. These criteria do not account for trading fees, strategy searches, or the profitability of a forecast. A small AIC improvement is not an investment case.

A score or Lagrange-multiplier test evaluates how rapidly the likelihood would improve away from a restricted parameter value. It usually needs estimation only under the null. ARCH LM testing is one application. Likelihood-ratio, Wald, and score tests can agree asymptotically but differ materially in small samples or near parameter boundaries.

The Box–Jenkins workflow is iterative: identify transformations and plausible orders, estimate parameters, diagnose residuals, and evaluate genuine future forecasts. Diagnostics include residual ACF, remaining ARCH effects, outliers, parameter stability, and forecast calibration. A model can pass residual checks yet be economically unhelpful; diagnostics remove some failure modes rather than certify a trading strategy.

The augmented Dickey–Fuller regression typically tests a unit-root null through a lagged-level coefficient in a regression of differences. Choices of intercept, trend, and lag length matter. Rejection does not establish a stable economic equilibrium; non-rejection does not prove a unit root. Short samples, breaks, nonlinear dynamics, and nearly integrated processes complicate interpretation. Applying ADF to an intrinsic mode function is a unit-root test on that extracted component, not a general test of component “stability.” [L01, chapters 6–8]

## 1.7 A reusable forecasting protocol

1. Name the target, horizon, sampling clock, and decision time. Distinguish future return, future variance, volume, and order direction.
2. Define the available information set and remove records that would arrive too late.
3. Fit transformations, imputers, scalers, model orders, and parameters using training data only.
4. Compare against persistence, zero-return, seasonal-naive, and simple linear baselines as appropriate.
5. Evaluate at rolling forecast origins. Purge training observations whose forward label intervals overlap validation or test intervals.
6. Examine forecast errors by period, asset, liquidity, and regime. A pooled metric can conceal failure in the intended trading conditions.
7. Convert predictions into a specified position and execution policy; evaluate that policy separately.

The FPP2 toolbox chapter makes rolling-origin evaluation and the distinction between fitted residuals and forecast errors particularly clear. Its forecast metrics are useful for this layer; trading still needs additional accounting. [B02]

Current official statsmodels documentation exposes `AutoReg` and `statsmodels.tsa.arima.model.ARIMA`. `AutoReg` supports explicit lags, deterministic terms, and aligned exogenous inputs. Equal hold-back periods help compare different lag specifications on the same sample. Pin the installed version and check intercept/trend conventions. In R, `stats::ar` and `stats::arima` are base-package functions; `sarima` should not be described as a function from the base stats package. [S01, S02]

## 1.8 Reading map for Cryer and Chan

The supplied second-edition textbook has 501 PDF pages. Its value is the progression from processes to model identification, diagnostics, and prediction, with R examples. The following is a chapter-level synthesis and navigation map, not a claim that every exercise and proof was independently verified. Printed page numbers begin 13 pages after the start of the PDF. [L01]

| Chapter | Printed start | What to retain for trading research |
|---|---:|---|
| 1 Introduction | 1 | Define the time ordering and research question before fitting models. |
| 2 Fundamental concepts | 11 | Expectation, covariance, correlation, and stationarity are the language of later tests. |
| 3 Trends | 27 | Separate deterministic trends, stochastic trends, and regression uncertainty. |
| 4 Stationary models | 55 | AR, MA, and ARMA are benchmark models and interpretable filters. |
| 5 Nonstationary models | 87 | Differencing changes both the target and model interpretation. |
| 6 Model specification | 109 | ACF/PACF and unit-root analysis help propose models, not certify them. |
| 7 Parameter estimation | 149 | Estimation method and initial conditions affect fitted parameters. |
| 8 Diagnostics | 175 | Residual checks can expose misspecified mean and variance dynamics. |
| 9 Forecasting | 191 | Forecast recursion, error variance, and state-space ideas connect models to decisions. |
| 10 Seasonal models | 227 | Seasonal dependence needs an explicit calendar and sampling definition. |
| 11 Regression and interventions | 249 | Intervention analysis, outliers, spurious regression, and prewhitening matter for event studies. |
| 12 Heteroskedasticity | 277 | ARCH/GARCH model conditional variance rather than directional alpha. |
| 13 Spectral analysis | 319 | Frequency-domain structure is another description of dependence. |
| 14 Spectrum estimation | 351 | Finite-sample noise and smoothing influence apparent cycles. |
| 15 Threshold models | 383 | Dynamics can depend on state, motivating modest nonlinear alternatives. |
| 16 R appendix | 423 | Useful implementation context; reconcile old examples with installed package versions. |

---

# Paper 2 — Volatility, regimes, Bayesian inference, and simulation

## 2.1 Separate state estimation from return prediction

A useful market-state engine estimates several dimensions rather than forcing every observation into one label. Direction, conditional volatility, liquidity, jump activity, cross-asset participation, and strength of reversion need not move together. A market can trend upward while volatility falls, or remain directionless while liquidity deteriorates.

Define each label operationally. “Trending” might mean a persistent conditional return estimate over a stated horizon; “high volatility” might mean a training-calibrated percentile of forecast variance; “broad participation” might mean the fraction of eligible assets with positive returns over a past interval. None should be defined using the subsequent trade outcome and then passed back as a contemporaneous feature.

Keep a probabilistic output when uncertainty is substantial. For example, a model can report probabilities for up, down, and flat alongside a variance forecast and a liquidity score. The usefulness of these outputs is measured by calibration and the incremental value of conditioning a policy on them. More state labels create more opportunities to fit noise.

## 2.2 ARCH and GARCH

Let ε_t = sqrt(h_t) z_t, where z_t has conditional mean zero and conditional variance one. An ARCH(q) specification is h_t = ω + Σα_i ε²_(t−i). A GARCH(1,1) adds persistence in the variance itself:

$$
h_t=\omega+\alpha\epsilon_{t-1}^2+\beta h_{t-1}.
$$

Standard sufficient positivity restrictions are ω > 0 and α, β ≥ 0. Under the usual innovation assumptions, α + β < 1 gives a finite unconditional variance ω/(1−α−β). This is a second-moment condition, not the most general strict-stationarity condition. High estimated persistence may reflect genuine slow-moving risk, structural breaks, omitted seasonality, or model misspecification.

An EWMA variance update resembles a restricted GARCH recursion with no constant and weights summing to one. It is a useful operational baseline but does not have the same finite unconditional variance interpretation. A GARCH fit should beat such simple baselines on future variance or a relevant risk score before earning its added complexity.

Asymmetric models allow negative and positive shocks to affect variance differently. GJR-type models add an indicator-weighted squared shock; EGARCH evolves log variance. Standardised Student-t innovations can represent heavier tails than Gaussian errors, but they do not remove regime breaks or liquidity gaps. Check the distribution's variance convention and parameter constraints.

The supplied arch documentation separates conditional mean, volatility process, and innovation distribution, and describes analytic, simulation, and bootstrap forecast routes. The supplied URL exposes an older documentation build, so use it conceptually and reconcile examples with a pinned installed version. [L01, chapter 12; W25]

## 2.3 Measuring realised risk

For high-frequency log returns within a future interval, realised variance is the sum of squared returns. It estimates integrated variance under suitable sampling and process assumptions; observed market prices add bid-ask bounce, asynchronous trading, and jumps. Sampling more frequently does not indefinitely improve the estimate.

For minute-bar research, preserve both a simple trailing variance estimate and a robust alternative. Do not silently fill a long exchange outage with zero returns: this suppresses apparent volatility until the eventual jump. Record missingness and market status as data-quality variables.

Volatility of volatility concerns the variability of volatility itself. Its units depend on whether you use variance, standard deviation, log volatility, or changes in a forecast. A percentage change in a nearly zero volatility estimate can explode. Define a floor or model log volatility, and calibrate any floor on training data.

A “turbulence” score should likewise have a declared construction. A multivariate Mahalanobis return score, a change in realised volatility, and a jump indicator are different measures. Give them separate names and test their separate incremental value. A single dramatic-looking indicator can otherwise mix several mechanisms without explaining which matters for trading.

## 2.4 Reversion and the Ornstein–Uhlenbeck model

For a continuous spread X_t, the OU model is:

$$
dX_t=\kappa(\mu-X_t)dt+\sigma dW_t,\qquad \kappa>0.
$$

Its conditional mean reverts exponentially, and its stationary variance is σ²/(2κ). At regular spacing Δ, its exact transition has AR(1) coefficient φ = exp(−κΔ). The mean-reversion half-life is log(2)/κ, or −Δ log(2)/log φ for 0 < φ < 1. A negative AR coefficient produces alternating dependence and should not be plugged into this positive-OU half-life formula.

The estimated half-life is uncertain. Near φ = 1, a small coefficient error can produce a large change in the estimated reversion horizon. Report an interval or bootstrap distribution and assess parameter stability. An estimate of two hours does not mean every excursion will close in two hours.

A spread's equilibrium may change. If the hedge ratio or mean is repeatedly adjusted, separate realised trading profit from redefinition of the measuring stick. Retrospectively recomputing historical residuals with today's fitted hedge ratio can make a broken relationship appear stable.

Mean-reverting order flow, volatility, or volume is not the same as mean-reverting prices. If order flow returns from strongly positive to zero, price might simply stop rising. A reversal strategy needs evidence about future price changes conditional on the feature, not merely an ADF rejection for the feature itself. This distinction is central to the order-flow paper reviewed in Paper 4. [L01; L10]

## 2.5 Hidden states and change points

A hidden Markov model assigns a latent state S_t a transition matrix and models observations conditional on that state. Filtered state probabilities use data through t. Smoothed probabilities use later observations and are inappropriate as historical live-trading features unless delayed until those observations actually exist.

Fit state labels inside the training sample. State 0 in one fit need not have the same economic meaning as state 0 in another. Align states by their fitted properties or use continuous outputs. Track how often the model changes its state assignments when a new batch of data arrives.

Threshold autoregressions offer a simpler nonlinear alternative: parameters change when an observed lagged variable crosses a threshold. They can be easier to inspect than a latent-state model, although threshold selection still adds estimation risk. A change-point detector asks whether the data-generating relationship has shifted, not which profitable strategy should replace it. Detection delay and false alarms are part of its evaluation.

A useful experiment keeps the trading signal fixed while adding one state variable. If the conditioned model fails to improve net results on future data, a richer regime engine is not justified by attractive plots. [L01, chapter 15; independent research design]

## 2.6 What rough volatility contributes

Gatheral, Jaisson, and Rosenbaum study the roughness of log volatility and propose a rough fractional stochastic-volatility representation. Their empirical roughness parameter is around H = 0.1. This concerns volatility paths, not the Hurst exponent of traded-price returns. Apparent long-memory diagnostics can also arise in the model. The paper supports investigation of volatility forecasts and option models, not an automatic directional strategy. [P04]

For your research, first compare an EWMA forecast, GARCH, and a simple regression of future realised variance on past realised-variance aggregates. Only then add a rough-volatility specification. Use the same horizon and scoring rule across models. Better calibration can improve exposure control without increasing raw signal accuracy.

## 2.7 MCMC is an inference method

The supplied MCMC introduction develops Markov chains, invariant distributions, and simulation from a target distribution. A chain used by an inference algorithm is different from a model of how market regimes transition. Confusing the two leads to incorrect interpretations of an MCMC sample as a simulated future market path. [L02]

For target density π and proposal q(y | x), the Metropolis–Hastings acceptance probability is:

$$
a(x,y)=\min\left(1,\frac{\pi(y)q(x\mid y)}{\pi(x)q(y\mid x)}\right).
$$

The ratio removes an unknown common normalising constant. A symmetric proposal cancels the proposal-density ratio, but an asymmetric proposal does not. A Gibbs step draws from a full conditional distribution; as a Metropolis–Hastings construction its acceptance probability is one. Both methods can still explore a correlated or multimodal target very slowly.

Invariance says a distribution remains unchanged under a transition kernel. It does not by itself establish that a chain started anywhere will approach that distribution. Irreducibility, recurrence, and aperiodicity provide relevant conditions. For the AR(1) example in the notes, |β| < 1 gives a Gaussian invariant distribution when the innovations are Gaussian, with variance σ²/(1−β²).

For a scalar function of the chain, a common large-sample effective sample size approximation is n divided by 1 + 2Σρ(k), where ρ refers to its chain autocorrelation. The ESS can differ greatly across parameters. A million highly correlated draws are not a million independent posterior samples.

Burn-in does not prove convergence. Use multiple overdispersed chains, inspect traces and mixing, and examine uncertainty for the quantities that drive the trading decision. Routine thinning discards information and usually is not a cure for poor mixing; it can be reasonable when storage or downstream evaluation is the bottleneck. Transformations, better proposals, or different samplers address the underlying problem more directly.

Applications include uncertainty in a spread's mean-reversion speed, volatility parameters, latent states, and expected return. Keep prior information and likelihood assumptions explicit. A posterior produced from a misspecified likelihood can be precise and wrong. Fit the posterior only with information available before the forecast; an in-sample posterior predictive check is not an out-of-sample trading evaluation.

## 2.8 Simulated paths and their validation

Simulation is useful for stress tests, controlled comparisons, and training policies where real paths are scarce. It is dangerous when the same assumptions both create the simulated market and make the tested strategy profitable. A reversion agent trained on stationary OU paths has not learned what to do when a real spread permanently breaks.

Test at least marginal distributions, serial dependence, volatility persistence, cross-asset dependence, jumps, drawdown duration, and liquidity conditional on stress. Generate scenarios with changing regimes and parameters outside the fitted confidence region. Separate the distribution used for learning from the scenarios reserved for evaluation.

Signature-based scenario validation compares distributions of path features rather than only one-time marginals. The supplied paper develops signature-based tests for real-world economic scenarios, including insurance applications. It is a useful candidate for simulator diagnostics; failure to reject a two-sample test does not certify realism, particularly with limited power or a feature map insensitive to the relevant discrepancy. [P10]

The jump-diffusion Schrödinger-bridge paper studies a relative-entropy path-space interpolation problem with endpoint constraints and jump dynamics. Its theoretical contribution is relevant to constrained scenario generation. It does not identify the true market process from endpoints alone, and a plausible interpolation is not a validated return forecast. [P16]

The practical sequence is therefore: obtain a credible empirical baseline, build a deliberately limited simulator, identify which properties it reproduces, and test a policy against deliberate violations of those properties. The simulator is a research instrument whose weaknesses should be documented alongside the strategy's results.

---

# Paper 3 — A strategy atlas and a route from idea to hypothesis

## 3.1 What the strategy collection does and does not establish

The uploaded `Alphas.pdf` is Kakushadze and Serur's *151 Trading Strategies* (2018), a 361-page collection spanning options, stocks, ETFs, fixed income, indexes, volatility, FX, commodities, futures, structured products, convertibles, tax-related trades, specialised assets, real estate, cash, crypto, macro, and infrastructure. It includes an R backtesting appendix, an extensive bibliography, and a glossary. [L09]

It should be used as a taxonomy of mechanisms and constructions. A strategy payoff is not an empirical alpha. A historical anomaly is not a guarantee of current returns. Different named trades may express the same underlying exposure, and a profitable gross payoff can become unprofitable after financing, spread, fees, and execution.

The accompanying strategy index preserves source section numbers and printed starting pages. It contains 174 strategy-labelled or closely related entries when nested variants and weighted-regression reversion are counted. That is an indexing count, not a correction of the book's title or a claim of 174 independent opportunities.

## 3.2 A common template

For every candidate, record the economic mechanism, observable signal, instrument, holding horizon, entry and exit, position sizing, required data, and plausible failure. Distinguish risk-premium exposure from pricing inconsistency and from a statistical prediction. A risk premium can remain positive over long periods while producing severe losses exactly when liquidity is poor.

Write the cash-accounting identity before the signal becomes elaborate. For a position q_t held across the next price move, gross mark-to-market profit is q_t(P_(t+1)−P_t). Trading changes cash by the actual executed notional and fees. Derivatives add contract multipliers, funding, margin, and settlement. A spread chart is not a self-financing portfolio unless its weights and rebalancing are accounted for.

## 3.3 Momentum and trend

Price momentum ranks past returns or takes exposure in the direction of an asset's own past return. Earnings momentum uses information from corporate results; residual momentum removes estimated factor exposure before ranking. Single-, double-, and triple-moving-average rules, channels, and breakouts are related signal constructions with different smoothing and delay.

The hypothesis is persistence over a specified horizon, possibly because information diffuses gradually or flows persist. The central risks are reversal, crowded positioning, whipsaw, and paying repeatedly for small changes of opinion. A rule with many short trades can have attractive gross accuracy but poor net returns.

For crypto minute bars, a practical test compares a volatility-scaled past return with a moving-average gap and a breakout from a trailing range. Use a modest set of horizons, identical exits and costs, and a delayed-entry sensitivity test. A breakout level must exclude the current completed bar if the trade is claimed to occur before that bar closes.

Cross-sectional ranking adds a universe problem. Today's surviving liquid coins are not the historical universe. Use availability and liquidity observed at the time, and retain delistings where feasible. Market-neutral research needs a feasible short leg; a spot-only implementation may instead express relative preference by reallocating a long-only portfolio. [L09, sections 3, 4, 10]

## 3.4 Mean reversion and relative value

Pairs trading models a relationship between assets; cluster reversion compares each asset with its group; weighted-regression reversion removes common exposures before trading residuals. The important object is the tradable residual, not simply a stationary-looking normalisation of one price series.

If z_t = log P_A,t − β log P_B,t − a, the hedge ratio β defines sensitivity in log-price space. Translating this into trade quantities requires current prices, capital, contract specifications, and a rebalancing rule. A rolling β changes the position, creates turnover, and can introduce a moving target for the exit.

Test convergence conditional on entry, including excursions that never converge before the maximum holding period. A stop or timeout is an economic rule; excluding failed excursions from the sample creates a selection bias. Include correlated simultaneous positions in portfolio accounting rather than summing separate single-trade returns as if each had its own capital.

For the user's reversion research, compare a causal rolling residual with a training-fitted equilibrium, and measure whether the proposed equilibrium is stable. Test whether the signal forecasts opposite-signed future returns, merely reduced continuation, or only lower variance. These have different trading implications. [L09, sections 3.8–3.10; L01]

## 3.5 Value, carry, and factor combinations

Value compares price with a fundamental or model-based anchor. Carry is the return expected from holding an instrument under specified unchanged-market assumptions, such as an unchanged curve. Neither is equivalent to cheapness or guaranteed income. The anchor can be wrong; carry can compensate for crash or funding risk.

In equities, value requires point-in-time accounting data and release dates. In fixed income, expected carry and rolldown depend on the curve and financing. In FX, interest differentials interact with currency moves. In commodities, futures curve shape and delivery economics matter. In crypto, funding and basis trades require exchange-specific terms, collateral treatment, and access to both legs.

A multifactor portfolio combines signals after establishing comparable units and exposures. Combining ten correlated variants of momentum adds much less diversification than the count suggests. Estimate correlation of out-of-sample strategy returns and of exposures during stress. Weighting recent winners can itself become a highly variable strategy.

The catalogue's low-volatility and low-risk constructions should be interpreted as exposure choices with empirical motivations, not universal return laws. Leverage constraints, benchmark definitions, and financing can materially change outcomes. [L09, sections 3–10]

## 3.6 Options as payoff engineering

The large options chapter is best understood through a small set of building blocks. A long call pays max(S_T−K,0); a long put pays max(K−S_T,0). Trading profit also includes the premium, financing, exercise rules, and transaction costs. “In the money” describes intrinsic value, not profitability after premium.

| Family | Economic shape | Principal limitation |
|---|---|---|
| Covered call / protective put | Exchange some upside for premium, or pay for a downside floor | Premium and opportunity cost must be included. |
| Vertical spreads | Bounded directional exposure across strikes | Several legs create spreads and execution risk. |
| Calendar / diagonal spreads | Different maturities, sometimes different strikes | Exposure is to term structure and path-dependent exercise effects. |
| Straddle / strangle / guts | Exposure to large moves in either direction | Realised movement must justify the initial option prices. |
| Strap / strip / ratio structures | Directionally tilted convexity | Unbalanced short legs can create large tail exposure. |
| Butterflies / condors | Concentrated or bounded exposure to a region of terminal prices | A terminal payoff diagram hides interim Greeks and margin. |
| Synthetic forwards / boxes | Replication and financing relationships | Exercise, borrow, collateral, and fees prevent naive parity trades. |
| Collars / seagulls | Reshape existing risk with combinations of options | A low initial premium is not the same as low total risk. |

These constructions belong primarily in a derivatives research track. For spot-only algorithmic trading, their immediate value is conceptual: a strategy's payoff can be decomposed into direction, convexity, carry, and tail exposure. A high-win-rate reversion strategy often resembles selling insurance against a large persistent move. [L09, section 2]

## 3.7 Asset-class transfer map

| Catalogue area | Transferable idea | What prevents direct reuse in crypto spot |
|---|---|---|
| ETF rotation and dual momentum | Ranking plus an absolute trend filter | Fund mechanics and historical universes differ. |
| Bond bullet, barbell, and ladder | Concentration across horizons; cash-flow matching | Spot coins lack contractual cash flows and duration. |
| Bond butterflies and curve trades | Neutralise dominant factors; trade residual shape | Requires a curve, rate sensitivities, and financing. |
| CDS basis and swap spreads | Compare economically related claims | Credit, counterparty, documentation, and settlement matter. |
| Index cash-and-carry | Price a basis after financing and costs | Both legs, margin, borrow, and settlement are necessary. |
| Dispersion and variance risk premium | Distinguish component and aggregate risk | Requires suitable options or variance instruments. |
| FX triangular arbitrage | Enforce cross-rate consistency | Executable bid/ask, depth, and sequential fill risk dominate. |
| Commodity roll and hedging pressure | Term structure and participant demand | Delivery and inventory economics differ by market. |
| Futures calendar spreads | Relative value across maturities | Contract rolls and different margin/funding rules. |
| Structured credit and MBS | Model correlated defaults and optional cash flows | Highly specialised data and market access. |
| Convertible arbitrage | Separate credit, equity, and option exposures | Borrow and security-specific terms are central. |
| Macro announcement trading | Event-time information and changing participation | Release timestamps, revisions, and venue clocks. |
| Real estate and infrastructure | Illiquidity, cash-flow duration, diversification | Not a minute-frequency electronic trading problem. |

Tax-related strategies and the banking-regulation primers are historical educational material, not current jurisdiction-specific advice. The catalogue also contains cash-related headings describing money laundering and loan sharking. They are recorded for source completeness and excluded from the research candidates. [L09, sections 13–20; L11]

## 3.8 Market making, arbitrage, and event strategies

Market making earns a spread only if the realised combination of fills and subsequent price changes is favourable. Inventory risk, adverse selection, queue priority, cancellations, and forced liquidation can dominate quoted spread. Treat order placement as a separate policy from price forecasting; Paper 4 develops this distinction.

Triangular arbitrage should be evaluated using the correct side of each bid/ask quote and fees on every leg, with size limited by available depth. A positive product of three indicative exchange rates is not an executable profit. Latency can leave the strategy with an unintended directional position after the first fill.

Event strategies need a timestamped event definition. A merger announcement, earnings release, token listing, and monetary-policy announcement have different information channels and risks. A backtest using the final news article time or a revised event calendar can accidentally trade before information was available.

The Knuteson paper advances a controversial manipulation explanation for overnight versus intraday equity returns. Treat it as an argument to examine, not an established causal conclusion. A legitimate research adaptation is to decompose returns by session, test across market structures, and compare liquidity, news, and risk exposures. The paper is not evidence that a manipulative execution scheme is a sound trading strategy. [P11]

## 3.9 Position-size progressions do not create alpha

The Reverse Labouchere page describes a betting progression and explicitly says its author does not use it for trading. Its useful lesson is about path-dependent staking, not expected-return creation. [W29]

If a stake a_t is chosen from current information and the next unit payoff has conditional expectation μ_t, the expected incremental gain is a_t μ_t. For nonnegative stakes and nonpositive μ_t, changing stake size alone cannot make the conditional expected gain positive, under the usual integrability conditions. Stopping and resetting can change the distribution of outcomes and the timing of losses; they do not remove the underlying disadvantage.

Use position sizing to respect uncertainty, capital, correlation, liquidity, and loss tolerance. Do not use a win sequence as evidence that the next trade is better unless that dependence is separately demonstrated. A profitable small-stake record followed by a large loss is a predictable possibility under progressions.

## 3.10 First candidates for a research queue

The following priorities are a judgement about tractable experiments, not claims of profitability: volatility-normalised trend; causal residual reversion; conditional reversal after a large return and unusual volume; cross-sectional residual momentum; and simple risk targeting. They can begin with clean bars and interpretable rules.

Microprice and order-flow forecasting need event-level books and trades. Market making additionally needs a fill model. Option and credit strategies need different instruments and pricing infrastructure. Deep reinforcement learning should wait for a simulator with correct accounting and an established simple benchmark.

The catalogue's backtesting appendix is useful for translating forecasts into constrained holdings and illustrates an approximate cost adjustment to expected returns. It explicitly leaves survivorship bias untreated. Do not inherit that omission or assume intraday trading makes it harmless. Preserve the appendix as a historical implementation example, not a complete modern evaluation standard. [L09, PDF pages 126–133]

---

# Paper 4 — Order flow, microprice, execution, and interacting agents

## 4.1 Start with a faithful market record

An order book is a sequence of events, not just a matrix of prices and sizes. A usable record includes exchange timestamps, local receive timestamps where available, sequence numbers, snapshot/delta conventions, tick size, quantity units, trades, and recovery from gaps. Changes to price levels can be caused by cancellations, additions, and executions; candles usually cannot identify which occurred.

The midpoint is (best bid + best ask)/2. The last traded price is a different object. Queue imbalance often means (bid size − ask size)/(bid size + ask size), but depth range and normalisation must be specified. A size-weighted midpoint is also distinct from a model-based microprice.

Define the prediction horizon in clock time, event count, or next price change. “Predicting the next move” could mean direction conditional on a move, expected return after one second, or expected future midpoint after several transitions. These are different targets with different class balances and economic values.

## 4.2 The stationarised order-flow-imbalance paper

Wang, Teng, Hao, and Shi study order-flow imbalance in Dalian coke futures, using contract J2001 and October 2019 data for their regression analysis. They compare ordinary top-of-book OFI with a variant constructed from log queue sizes. The paper is short and useful as a feature-engineering example, but its statistical and trading interpretations require care. [L10, six PDF pages]

Let b_n, a_n be best bid and ask prices, and q^b_n, q^a_n their displayed sizes after event n. One standard event contribution is:

$$
e_n={\bf1}_{b_n\geq b_{n-1}}q^b_n-{\bf1}_{b_n\leq b_{n-1}}q^b_{n-1}-{\bf1}_{a_n\leq a_{n-1}}q^a_n+{\bf1}_{a_n\geq a_{n-1}}q^a_{n-1}.
$$

When bid price rises, the bid contribution is the new bid size; when unchanged, it is the size difference; when it falls, it is minus the previous size. The ask side has the corresponding reversed sign. Sum event contributions within the chosen observation interval. Both indicators fire at equality, producing the intended size difference.

The paper's log-OFI substitutes log queue sizes in the piecewise construction. It is **not** the logarithm of signed OFI. A signed log transform, sign(x)log(1+|x|), is another feature altogether. Log queue sizes require positive quantities; log1p changes the specification. Unit rescaling can affect replacement terms at price changes, so quantity units must be fixed.

The price labelled “mid-price” in the paper is the size-weighted expression (q^b a + q^a b)/(q^b+q^a), rather than the arithmetic midpoint. Because queue sizes enter both this target and the predictor, the mechanical relationship deserves particular attention. Reproduce results with arithmetic midpoint and executable returns as alternative targets.

The paper fits price changes over an interval to OFI measured over that same interval. This is primarily a **contemporaneous explanatory regression**. Its reported out-of-sample R² values, roughly 0.501 for OFI and 0.747 for log-OFI, do not establish that the feature was available before the price change. The paper also uses an uncentred R² denominator, sum of squared target values, which differs from the usual mean-centred definition.

Its examples use 1,500 in-sample and 300 out-of-sample observations per day. Trading examples use fixed thresholds and display cumulative profit, but the treatment of fees, fills, queues, and latency is insufficient for a transferable execution result. A reduced outlier range does not prove stationarity. Even genuine mean reversion of the feature would not by itself justify fading the price.

The appropriate replication has three stages. First reproduce the paper's same-interval regression and metric exactly. Second shift the target strictly forward and report deterioration. Third attach executable prices, delays, realistic costs, and a fixed policy. Keep these stages separate so explanatory power cannot be mistaken for tradable prediction.

## 4.3 Predicting participant behaviour

Cartea and coauthors' *Statistical Predictions of Trading Strategies in Electronic Markets* uses 16 weeks of Euronext Amsterdam data for ASML, ING, Ahold, and TomTom, starting in October 2021. Its distinctive advantage is identification of algorithms and members through regulatory data, allowing features related to their own orders, inventories, and cash. Public order-book feeds generally do not reveal this information. [L04]

The study constructs 53 pre-order features. They include book depth and imbalance, the participant's own presence, spread, returns and volatility, messages and aggressive flow at several horizons, the last order's volume, and accumulated inventory and cash. Some quantities are transformed using sign(x)log(1+|x|). Inventory is an intraday, single-venue proxy, not necessarily the participant's full economic position.

The first four weeks fit a principal-component transformation with 30 components explaining more than 90% of feature variation; it is then held fixed. Four-week training windows precede one-week deployments over 12 exercises. Logistic models classify direction; conditional models classify price and volume; random forests provide a benchmark. These are predictions of **the participant's next order characteristics**, not direct predictions of profitable asset returns.

For ASML, the paper reports order-weighted accuracies of approximately 70% for direction, 85% for price, and 62% for volume. Improvements over each algorithm's majority-class baseline are about 18, 4, and 3 percentage points respectively. Those differences are more informative than raw accuracy because repetitive behaviour can make the majority baseline strong. Random forests improve direction further in the reported comparison.

The fitted coefficients are also clustered into behaviour groups described as directional, opportunistic, and market-making. Registered liquidity-provider status does not map perfectly into inferred behaviour. This is a useful warning against assuming an institutional label identifies a uniform strategy.

The transferable ideas are pre-decision feature discipline, majority baselines, rolling deployment, and behavioural heterogeneity. The non-transferable shortcut is assuming public data contain the same member-specific information. The study explicitly leaves latency outside scope. It does not show that a retail trader can infer another participant's full inventory and trade ahead of its orders profitably.

A public-data adaptation could predict aggregate trade direction or price-change probability using only book and trade history. Its results must be reported as a new experiment. It should not inherit the paper's accuracy figures as a target or expected outcome.

## 4.4 From weighted midpoint to microprice

The reviewed Micro_Price notebook uses a discrete state space combining imbalance information and a cross-asset residual. Its example involves inverse equity ETFs, not crypto. It estimates transition matrices and accumulates expected price adjustments through matrix operations. The code contains an initial term of the form (I−Q)⁻¹RK and a finite sum of further transition adjustments. [R09]

The general principle is to estimate expected future price changes conditional on the current observable state. This can outperform a naive midpoint when book states contain information, but it is only as good as the transition estimates and stationarity assumptions. A weighted midpoint is a simple imbalance-based proxy; a state-transition microprice is a fitted model.

For implementation, estimate all transition probabilities in past data, smooth sparse states, inspect row sums, and check that any transient-state fundamental matrix is well-defined. Solve linear systems rather than explicitly inverting matrices. Finite horizons avoid pretending that a short-run book model predicts a long-run fundamental value.

Compare midpoint, weighted midpoint, imbalance-only regression, and the fitted microprice on identical future targets. Evaluate calibration by state and stability across assets, tick sizes, spread regimes, and busy versus quiet periods. A smaller midpoint forecast error may still fail to cover an executable spread.

## 4.5 Execution is an optimisation problem with constraints

The Obłój lecture deck develops execution under permanent, temporary, and transient price impact. It contains 74 slides arranged over 26 PDF pages. Its central contribution to this project is an explicit distinction between a desired position change and the cost of implementing that change. [L03]

In a simple liquidation model, remaining inventory x(t) starts at x_0 and finishes at zero. Temporary impact penalises trading speed; price uncertainty penalises remaining exposed inventory. With a declared objective convention:

$$
J(x)=\int_0^T\left[\eta\dot{x}(t)^2+\frac{\lambda\sigma^2}{2}x(t)^2\right]dt.
$$

The deterministic optimum is:

$$
x^*(t)=x_0\frac{\sinh(\kappa(T-t))}{\sinh(\kappa T)},\qquad \kappa=\sqrt{\frac{\lambda\sigma^2}{2\eta}}.
$$

As risk aversion tends to zero, this approaches constant-speed liquidation. More urgency or price risk favours earlier trading; more temporary impact favours spreading the trade. Factors of two vary with how risk aversion is defined, so copying κ without its objective is unsafe. Linear permanent impact contributes a cost that is independent of schedule under this simplified fixed-endpoint setup.

TWAP and volume-based benchmarks are reference schedules, not universally optimal policies. Participation limits, nonlinear impact, discrete ticks, changing liquidity, alpha decay, and limit-order fills alter the problem. If the signal is expected to disappear quickly, slow execution can sacrifice its value; if expected edge is small, aggressive execution can consume it entirely.

## 4.6 Transient impact and model robustness

A transient-impact model lets the price effect of trading decay through a kernel G. On a discrete grid, expected impact cost can take the quadratic form one-half ΔᵀΓΔ, with Γ_(ij) = G(|t_i−t_j|). Under suitable positive-definiteness and endpoint constraints, the optimal schedule is related to Γ⁻¹1.

The lecture examples show that superficially similar decay kernels can produce very different schedules, including implausibly oscillatory trades. This is a model-risk result: a numerically optimal strategy may exploit a weakness of the assumed impact model. It is not an instruction to create artificial volume or manipulate prices.

Test the model's behaviour on round trips, large sizes, finer time grids, and stressed liquidity. Include spread and trading costs; prohibit economically meaningless oscillation if the simulator otherwise rewards it. A strategy that earns money only by repeatedly exploiting its own simulated impact is evidence of a flawed environment.

## 4.7 Inventory, adverse selection, and evaluation

A market maker needs a valuation or short-horizon forecast, an inventory target, quote distances, quote sizes, and cancellation rules. Inventory limits should depend on capital and correlated risk, not simply the number of open assets. Two dozen coins with similar market beta can behave like one concentrated position in a selloff.

Report fill probability and markout conditional on a fill. A buy limit order can look attractive before it fills but be selected precisely when the price is about to fall. Unconditional future returns around all quote submissions do not measure this adverse-selection cost.

Maker fee zero does not mean trading cost zero. Costs include queue delay, missed fills, adverse selection, cancellation latency, residual inventory, and market-order liquidation. With OHLCV bars, touching a limit price is not sufficient proof of a fill; both barriers being hit inside a bar create an unknown event ordering.

Useful execution metrics include implementation shortfall, spread paid or captured, post-fill markout at several horizons, fill and cancellation rates, time to completion, inventory variance, and worst residual exposure. Aggregate them by market conditions. A single overall spread-capture figure can hide losses during the events that determine total performance.

## 4.8 Mean-field and partial-information models

The anticipatory-trading mean-field paper models a large trader interacting with a population of high-frequency traders. Inventory preferences and temporary/permanent impact determine whether the population supplies or consumes liquidity. Its relevance is strategic interaction and execution sensitivity; the conclusions are conditional on the model rather than a universal market-making advantage. [P14]

The partial-information mean-field paper studies heterogeneous agents filtering an unobserved state while trading with impact. Its equilibrium and approximation results explain how inference and control interact. It is a theoretical framework, not an empirically validated crypto signal. A practical first step is a filtered-state execution benchmark before attempting a full equilibrium solver. [P15]

---

# Paper 5 — Machine learning, rough paths, signatures, and control

## 5.1 What a learning system is being asked to learn

Separate representation learning, supervised prediction, unsupervised description, and control. A feature map transforms observations; a predictor estimates a target; a clustering algorithm groups observations; a trading policy chooses actions given state and constraints. Logistic regression is a classification model despite its name. Clustering does not become a return forecast until an independently evaluated relationship is attached to the clusters. [L08, L11]

A useful baseline hierarchy is persistence or zero forecast, linear or regularised regression, a modest tree ensemble, and only then more elaborate sequence models. Compare on identical forecast origins, targets, costs, and data. A model with more inputs may appear better because of information availability rather than architecture.

Financial data are particularly vulnerable to overlapping labels and repeated searches. If a target is the next 30-minute return, neighbouring minute-level labels share most of their price moves. Their errors are dependent, and adjacent train/test boundaries can overlap. Randomly shuffling these observations is not a valid simulation of deployment into the future.

## 5.2 Review of the supplied CNN–LSTM paper

The uploaded 2023 paper combines convolutional layers with bidirectional LSTMs for daily stock-price prediction. It describes OHLC data from several sources and markets, mean imputation, an 80/20 split, and both shuffled and unshuffled data. Its architecture includes convolutional filters, pooling, two bidirectional LSTM layers, dropout, and an Adam/MSE training objective. [L07]

The reported metrics include MSE 0.035, MAE 0.075, and R² approximately 0.9353 in one table. These numbers depend on target scale and preprocessing. They cannot be compared directly with unnormalised errors from another asset, nor converted into a trading win rate. Broad claims about 99% prediction performance should not be interpreted as 99% profitable trades.

The main concerns are the chronology of splitting, the definition and overlap of input windows, whether preprocessing was trained only on the training sample, and the absence of a fully specified costed trading policy. High price-level R² is particularly weak evidence because persistence is a strong baseline for many price series.

A bidirectional LSTM is not automatically a look-ahead error: both directions may traverse an input window that lies entirely before the decision. It leaks when the window contains observations beyond the decision time or when a sequence-wide computation is incorrectly used to produce earlier forecasts. Inspect the actual data boundaries rather than rejecting an architecture solely by its name.

A proper replication should predict a clearly defined future return or compare next-price error against persistence, preserve chronology, purge label overlap, fit preprocessing locally, and report uncertainty across multiple future periods. Converting the result into a position policy is a separate experiment. The supplied paper is a model idea and teaching example, not sufficient evidence for deploying that architecture.

## 5.3 Reverse-mode automatic differentiation

Automatic differentiation applies the chain rule to elementary operations in a computational graph. Forward mode propagates input perturbations; reverse mode propagates output sensitivities backwards. Reverse mode is attractive for a scalar objective with many parameters. It is neither symbolic algebra in the usual sense nor finite differencing. The supplied Auto-ED module and Paszke slides explain this computational perspective. [W26, W28]

For a simple independent example, let u = ax, v = u², and L = v + bx. Then dL/da = 2ux, dL/db = x, and dL/dx = 2ua + b. Reverse accumulation reuses local derivatives rather than perturbing every parameter and rerunning the whole function.

In finance, AD can calculate training gradients, sensitivities of pricing functions, and derivatives through differentiable simulators. It differentiates the implemented computation, including any mistaken accounting or unrealistic fill rule. A correct gradient of an incorrect simulator is still economically wrong.

Hard thresholds, discrete order types, exercise decisions, and sampled fills require special handling. Replacing them with smooth approximations changes the objective. Validate gradients against finite differences in small controlled cases and evaluate the final policy in the actual discrete environment. The PyTorch slides are historical; their architectural ideas remain useful, but old API examples should be checked against a pinned modern implementation.

## 5.4 What a path signature records

For a sufficiently regular d-dimensional path X over [s,t], its signature collects iterated integrals. The level-k coordinate associated with channels i_1,…,i_k is:

$$
S^{i_1\cdots i_k}(X)_{s,t}=\int_{s<u_1<\cdots<u_k<t}dX^{i_1}_{u_1}\cdots dX^{i_k}_{u_k}.
$$

Level zero is 1. Level one records increments. Level two records ordered interactions between channels, including antisymmetric area information. Higher levels capture more complex ordered interactions. Rough-path theory extends these ideas to irregular signals with a suitable enhanced-path interpretation. [P01; P02]

For a single smooth scalar path, level k is simply (X_t−X_s)^k/k!. Its signature therefore cannot distinguish arbitrary shapes with the same net increment. This is why feeding only a one-dimensional price path is often much less informative than expected. Adding time, volume, or another channel changes the representation.

For a geometric signature, the sum of the two second-order cross coordinates equals the product of the corresponding increments. Their difference contains signed area information. This explains why order matters: a price move followed by a volume surge can differ from a volume surge followed by the price move even if endpoints agree.

Signatures of concatenated paths combine through a tensor-algebra product. This composition property is useful for streaming and multiscale summaries. Truncating at depth m produces Σ_(k=1)^m d^k nonconstant coordinates. With five channels and depth three, that is 155; with ten channels and depth four, it is 11,110. Feature count grows rapidly.

The theoretical expressive properties need assumptions on path space, topology, and identifiability. They do not imply that a small finite training set will identify a profitable linear function. The pasted claim that signatures make linear regression one of the most powerful tools should be read as a motivation for a rich feature map, not a universal performance theorem or a requirement to combine signatures with an RNN.

## 5.5 Design a causal path representation

A useful first representation might contain elapsed time, cumulative log return, cumulative signed trade flow if observed, and a spread or volatility channel. Each channel needs units, a normalisation rule, and a missing-data convention. Do not call candle volume “signed trade flow” unless trade direction is actually available or explicitly estimated.

Time augmentation can preserve timing information that a reparameterisation-invariant representation otherwise loses. Lead–lag embeddings can encode additional variation information, but they increase dimension and depend on the discrete sampling convention. Interpolation is part of the model: a globally fitted spline can use future points when evaluated inside earlier windows. Prefer a construction whose information availability is explicit.

Scale channels using training data or a causal trailing rule. Log-signatures offer a more compact algebraic representation, but their coordinates are not interchangeable with raw signatures. Record depth, channels, augmentation, interpolation, and window length in the experiment identifier.

Begin with a regularised linear model on shallow signatures and compare against ordinary lagged returns, volatility, volume, and interaction features. The key question is whether the path representation adds future predictive value after controlling for its inputs and complexity. A signature model should not be credited for improvement obtained merely by adding extra market data.

## 5.6 Paper-by-paper map of the path-method sources

**Lyons, Differential Equations Driven by Rough Signals (1998).** This is foundational theory for differential equations driven by irregular signals, including the extra iterated-integral information needed for stable solution maps. The original supplied host failed; a publisher PDF was recovered, but text extraction corrupts many mathematical symbols. The notes use its conceptual role and do not claim a complete proof audit. [P01]

**Deep Signature Transforms (2019).** The paper combines learned augmentations and signature operations into trainable architectures. It motivates learning useful path representations rather than applying a single fixed signature once. Its empirical tasks support the modelling approach; they do not establish net trading profitability. A fair finance test should isolate learned augmentation from the signature operation itself. [P02]

**Signature kernel and a Goursat PDE.** For a smooth-path linear-kernel construction, the kernel solves a mixed-derivative PDE with boundary value one. This avoids explicitly expanding every signature coordinate, though pairwise kernel computation still has cost. The paper's Bitcoin experiment predicts a short future average price from a daily history window; its reported MAPE comparison is a forecasting result, not a costed trading return. [P03]

**Gamma Hedging and Rough Paths.** The paper develops pathwise hedging results using rough-path ideas and appropriate regularity assumptions. It links higher-order sensitivities to replication on irregular paths. The conclusions should not be extended to arbitrary jumps, illiquid hedges, or frictionless accuracy in a real account. Its natural application is a derivatives model-risk and hedging study. [P05]

**Neural RDE control.** The non-Markovian control paper uses neural rough differential equations to represent policies that depend on history. It offers a way to handle memory and varying time resolution in simulated control problems. For trading, compare any such policy with a simpler recurrent controller under the same simulator and evaluate on independently stressed dynamics. [P06]

**Stochastic control with rough paths.** Diehl, Friz, and Gassiat connect rough-path methods to stochastic control, including dynamic-programming and duality perspectives. Mathematical constructions involving anticipative information are not permission for a live policy to see future market data. The contribution is theoretical structure for control and approximation. [P07]

**Selective state-space foundations.** The supplied paper studies expressive foundations of deep selective state-space models through controlled dynamics and signature-related structure. It explains representation capacity, not the superiority of Mamba or any other architecture on financial data. Model choice still requires time-ordered empirical comparison. [P08]

**G-Signatures.** This work uses randomised signatures for global graph propagation. A possible financial adaptation is a time-varying asset graph, but graph construction, timing, and out-of-sample edges are separate research decisions. The paper's graph-learning results do not validate a cryptocurrency portfolio strategy. [P09]

The Oxford handwriting case study demonstrates an application of signature analysis outside finance. It is evidence that the representation can be useful in a particular recognition setting, not evidence about trading returns. The Cambridge rough-path textbook is a deeper theory route; only its landing page and contents were available in this review. QuantStart provides an introductory bridge, while primary papers anchor the technical discussion. [W14, B01, W15]

## 5.7 Reinforcement learning: reward design comes first

The statistical-arbitrage RL paper constructs mean-reversion spreads and trains a policy on simulated paths. Its stated reward uses the difference between the current spread and a true long-run mean, less a trading cost. The paper acknowledges that this mean is unavailable in real markets. Reported 2023 comparisons are therefore insufficient to bypass verification of reward, wealth, and execution accounting. [P13]

A shaped reward can accelerate learning, but a policy should ultimately be evaluated through a self-financing cash and inventory ledger. If a reward grants value for buying below a hypothetical equilibrium, it can make a trade appear successful before any favourable price move occurs. The agent's cumulative reward and realised account profit must be reported separately.

A minimal portfolio reward can start from the actual change in account equity, less an explicitly chosen inventory or risk penalty. This penalty is a preference, not a market cash flow. Define terminal liquidation, funding, margin, fees, rejected orders, and partial fills. Prevent the agent from exploiting missing penalties or unlimited borrowing.

Simulator randomisation is useful when it represents plausible uncertainty, such as changing impact, volatility, liquidity, and reversion speed. It is not enough to randomise parameters only inside a family that always rewards the strategy's core assumption. Include permanent relationship breaks and adverse regimes.

The FinRL contest repository is a source of tasks, starter workflows, and modelling ideas, including LLM-related components. A competition leaderboard measures performance under its own rules and data. The transferable element is the experiment structure, not an assumption that its score corresponds to production profitability. [R06]

## 5.8 An economical model-development sequence

First make the target and ledger correct. Next establish causal baselines. Then add useful state information and cost-aware thresholds. Only after those pieces are stable should a model receive greater capacity or a more complicated representation.

For each addition, freeze the rest of the experiment and ask what changed: ranking quality, forecast calibration, turnover, tail losses, or execution. Ablations are more informative than presenting only the best combined system. A strong final result should survive removing a fashionable component if the underlying economic mechanism is real, or clearly show why that component contributes information unavailable to the simpler model.

---

# Paper 6 — Portfolio construction, risk, sizing, and mathematical checks

## 6.1 Correct the portfolio formulas before using them

For simple asset returns R with conditional mean vector μ and covariance matrix Σ, a portfolio with beginning-of-period weights w has expected gross return wᵀμ and variance wᵀΣw. Its volatility is the square root of that variance:

$$
\sigma_p=\sqrt{w^\top\Sigma w}.
$$

The pasted notes' sum of weighted asset volatilities is not the general portfolio volatility. For nonnegative weights, it equals portfolio volatility only in special cases such as perfect positive correlation among all nonzero exposures. Ignoring covariance can substantially misstate diversification. [L08; mathematical correction]

For the usual long-only diversification ratio, use (Σw_i σ_i)/sqrt(wᵀΣw). Omitting the denominator's square root gives mismatched units. With short positions, interpretation and the definition of the numerator require additional care; do not silently transplant the long-only measure.

Downside deviation relative to target τ can be defined as sqrt(E[min(R−τ,0)²]). A Sortino ratio then divides expected excess return over the target by this deviation. It is not Jensen's alpha, and using downside variance without a square root changes its units. Specify whether the average includes all periods or only negative deviations, because both conventions appear in practice.

Jensen's alpha refers to performance relative to a specified factor-pricing benchmark, classically a market-model intercept in excess returns. Sharpe, Sortino, alpha, information ratio, profit factor, and win rate answer different questions. A high win rate does not establish a high expected return; a strategy can win small amounts frequently and lose a large amount occasionally.

## 6.2 Annualisation and dependent returns

The familiar square-root annualisation of Sharpe and volatility assumes conditions that fail under serial correlation or overlapping returns. For a sum of N stationary returns, variance includes covariance terms:

$$
\mathrm{Var}\left(\sum_{t=1}^{N}R_t\right)=N\gamma(0)+2\sum_{k=1}^{N-1}(N-k)\gamma(k).
$$

Use realised portfolio returns at a meaningful reporting frequency and account for dependence when estimating uncertainty. Overlapping 30-minute forecasts sampled every minute are not independent trades. Report gross and net results, exposure, turnover, number of independent market episodes, and drawdown duration alongside a Sharpe estimate.

Maximum drawdown is path-dependent and sample-length dependent. A backtest with no drawdown can reflect monotonic accounting, selective trade marking, or a very small sample rather than exceptional risk control. Include unrealised losses and an explicit mark convention. A closed-trades-only equity curve is not account equity.

## 6.3 Covariance and allocation

Sample covariance becomes unstable when the number of assets is large relative to effective observations. Highly correlated assets, unequal missing histories, and regime changes worsen conditioning. Shrinkage, factor structure, and conservative constraints often matter more than a sophisticated optimiser.

The unconstrained fully invested global minimum-variance portfolio is proportional to Σ⁻¹1, normalised to sum to one, when Σ is invertible. It may contain shorts and extreme weights. Adding nonnegativity, maximum weights, turnover penalties, and liquidity limits changes the problem and generally requires constrained optimisation.

An interpretable cost-aware objective is expected return minus a covariance risk penalty and a cost for changing holdings:

$$
\max_w\left\{\widehat{\mu}^{\top}w-\frac{\lambda}{2}w^{\top}\widehat{\Sigma}w-C(w-w_{old})\right\}.
$$

Expected-return estimates are often especially noisy. Compare the result with equal capital, inverse-volatility, and constrained minimum-variance allocations using the same eligible signals. The optimal in-sample weights can be the most fragile part of a strategy.

Risk contribution for asset i can be expressed as w_i(Σw)_i/σ_p. Equal risk contribution is not equal capital and is not the same as assigning weights proportional to inverse volatility when correlations vary. Cluster-based methods can reduce sensitivity to a full inverse but introduce sensitivity to clustering, ordering, and split rules.

## 6.4 Schur complements and hierarchical allocation

The exact linked Schur slide PDF could be identified in its repository but not decoded through the available retrieval route. A related primary paper and the author's explanatory pages were recovered. They connect hierarchical allocation with minimum-variance optimisation by retaining cross-group covariance through Schur complements. Later author notes clarify that exact recovery depends on the companion quantities and split rule, not merely substituting a new covariance block. [P12, S03, S04]

For a block covariance matrix Σ with diagonal blocks A and D and cross block B, the Schur complement of D is A−BD⁻¹Bᵀ. It is the covariance of a linear-regression residual; it equals a conditional covariance in the jointly Gaussian case. The exact block solution to Σx = 1 gives:

$$
x_A=(A-BD^{-1}B^\top)^{-1}(1_A-BD^{-1}1_D).
$$

This independent block-elimination calculation shows why the companion vector matters. Replacing A with the Schur complement but retaining an unmodified all-ones vector is not generally the same solution. For implementation, use linear solves and compare numerically with a direct constrained or unconstrained benchmark as appropriate.

A damping parameter can trade off noisy cross-block estimates against a simpler hierarchy. Select it on past validation windows, then freeze it for the next period. Check symmetry, permutation sensitivity, positive definiteness, weight concentration, and realised turnover. Mathematical equivalence under a known covariance does not guarantee superior out-of-sample performance when covariance is estimated.

## 6.5 Tail risk and scenario loss

Value at Risk is a loss quantile at a stated horizon and probability. Expected Shortfall averages the relevant tail under a carefully defined convention. Neither describes every aspect of loss timing, liquidity, or model uncertainty. For discrete distributions, use a definition of Expected Shortfall that handles mass at the quantile consistently.

Historical simulation preserves selected past observations but cannot invent an unseen market break. Parametric models extrapolate according to their assumptions. Scenario analysis can explicitly test exchange outages, gaps, collateral changes, correlated asset declines, missed exits, and delayed liquidation. These are assumptions to inspect, not probabilities established by the historical sample.

The QRM tutorial site is a route to the McNeil–Frey–Embrechts framework, with teaching material and code. Only the hub was reviewed here. It is a useful next reference for multivariate tails and aggregation, but the entire course was not read in this review. [W24]

## 6.6 A critical reading of the causal risk-neutral PDE paper

The supplied six-page *A Portfolio's Common Causal Conditional Risk-neutral PDE* attempts to combine common drivers, conditional probabilities, copulas, and a portfolio pricing PDE. The paper's claims should not be used as validated formulas without repairing several mathematical issues. The following are independent checks of the supplied text, not a blanket judgement about every possible model in this area. [L06]

First, a weighted sum of random variables does not generally have a CDF equal to a weighted sum of their CDFs. Conditional independence does not change this into a mixture identity. For independent standard-normal X and Y, Z = (X+Y)/2 has variance 1/2. At z = 1, its CDF is Φ(sqrt(2)), whereas the equally weighted mixture of the constituent CDFs gives Φ(1). These are different. A valid portfolio distribution requires integration of the joint distribution under the portfolio mapping.

Second, the expression presented as a Gaussian copula in equation 2 is a density, while the subsequent conditional-CDF identity requires a derivative of the copula CDF. At independence, the copula CDF is C(u,v)=uv and ∂C/∂v=u; the copula density is c(u,v)=1 and its derivative is zero. Substituting one for the other therefore fails even in the simplest case. This distinction was checked against the page image, not inferred solely from garbled extraction.

Third, a geometric-Brownian specification for a correlation variable does not enforce the correlation bound [−1,1]. A bounded transformation or an appropriately constrained process is required, along with positive semidefiniteness for a full correlation matrix. Correlation cannot be treated as an unrestricted price process.

Fourth, conditioning on variables described as causal does not by itself establish an identified causal model, a complete market, or a risk-neutral pricing measure. A pricing PDE needs clearly defined state variables, their dynamics under the relevant measure, tradable hedges or a specified pricing rule in an incomplete market, and consistent cash and financing terms. Removing a drift algebraically is not sufficient.

The illustrative historical plots do not constitute an out-of-sample economic test. A constructive research path would first build a coherent factor model and joint distribution, validate simulated portfolio distributions against direct sampling, and only then ask whether additional causal assumptions improve forecasting or pricing. Until those checks pass, keep this source in a speculative-methods folder rather than in the production formula library.

## 6.7 Position sizing under uncertainty

Sizing should connect expected edge, uncertainty, covariance, liquidity, and capital constraints. A volatility target controls estimated risk but can increase exposure after a quiet period just before a jump. Apply caps and stress the lag in the volatility estimate. Dollar neutrality does not imply beta neutrality, and beta neutrality does not eliminate basis or liquidity risk.

A small-edge Kelly approximation suggests a fraction proportional to expected excess return divided by variance in a simplified single-risk model. It is highly sensitive to estimation error and does not account automatically for jumps, constraints, or repeated parameter selection. It belongs as a theoretical comparator, not a default sizing instruction for an unproven strategy.

Use a conservative fixed-risk or capped notional baseline during research, then compare allocations while keeping signals unchanged. Avoid leverage merely to make a small estimated edge appear economically meaningful. For a spot-only venue, encode cash, inventory availability, minimum order value, and inability to short directly into the feasible set.

## 6.8 Multiple strategies and research selection

Diversifying across strategies works only if their losses differ in the circumstances that matter. Several reversion signals based on the same price series may all lose in a structural break. Estimate strategy covariance from genuine out-of-sample returns, preserving shared timestamps and overlapping capital use.

Keep a complete experiment register, including failed variants. If hundreds of parameter combinations are tested, the best result has a selection advantage. A final untouched period is useful only if it remains untouched during model choice. Repeatedly looking at it converts it into another validation set.

Report the distribution of results across folds and assets, sensitivity to nearby parameters, and outcomes under higher costs. Confidence intervals should respect time dependence, for example through appropriate block resampling. These are research-design proposals; this compendium does not claim that any supplied strategy has already passed them.

---

# Paper 7 — Derivatives, rates, credit, and numerical finance

## 7.1 Keep physical and pricing measures separate

Forecasting realised returns usually concerns a physical or real-world probability measure. Derivative valuation often uses a risk-neutral measure associated with a numeraire, with discounted traded assets satisfying the appropriate martingale property under stated assumptions. A risk-neutral expected payoff is not automatically a forecast of the payoff's real-world average.

The distinction matters when transferring pricing models into trading. A model that calibrates option prices accurately may be useful for hedging or relative-value comparisons, while providing little information about next-minute spot returns. Conversely, a return forecast need not price options consistently across strikes and maturities.

The six primers in `1703825251812.pdf` cover financial products, statistics, stochastic calculus, rate models, machine learning, and banking regulation. They are an accessible orientation rather than a substitute for detailed derivations or current market conventions. Their PDF ranges are 1–16, 17–30, 31–41, 42–50, 51–63, and 64–71 respectively. [L11]

## 7.2 Stochastic calculus and pricing

For dX_t = μ(t,X_t)dt + σ(t,X_t)dW_t and a smooth function f, Itô's formula includes a second-derivative term one-half σ² f_xx dt. That term is central to diffusion-based hedging and cannot be recovered by ordinary chain-rule intuition alone. The stochastic integral is defined through a limiting construction, with adaptedness and integrability conditions.

In the idealised Black–Scholes setting, a self-financing delta hedge leads to a pricing PDE with a diffusion term, financing term, and terminal payoff. The result depends on assumptions including a specified diffusion, continuous rebalancing, and suitable market access. Discrete trading, jumps, transaction costs, and uncertain volatility leave hedging error.

Delta measures first-order spot sensitivity, gamma second-order spot sensitivity, vega volatility sensitivity, and other Greeks describe additional exposures. Greek units depend on whether a parameter change is measured per unit or per percentage point. A delta-neutral position can retain substantial gamma, vega, jump, and funding risk.

The Derman repository contains a notebook reproducing ideas about discrete hedging and volatility-estimation uncertainty. It is useful for studying how hedge frequency and model uncertainty affect error. Its code is a historical research implementation with legacy API patterns; it was inspected but not executed or validated here. [R03]

## 7.3 Rate curves and short-rate models

Discount factors connect cash flows across maturities. A zero-coupon bond price is a discounted expectation under an appropriate pricing measure. An observed yield curve is not just one interest-rate number; different maturities and instruments imply a curve, and modern valuation conventions can require distinct discounting and projection curves.

Vasicek-type Gaussian short-rate models have mean reversion and can generate negative rates. CIR-type models use square-root diffusion and require care near zero; the familiar Feller condition is a sufficient boundary condition in the standard continuous model, while numerical discretisation can still need special treatment. Hull–White extensions introduce time-dependent terms to fit an initial curve. Lognormal short-rate constructions and multifactor models offer different flexibility and constraints.

Calibration seeks parameters that match selected market instruments under a pricing model. Statistical estimation seeks parameters consistent with historical dynamics. These are different objectives and may imply different drifts or risk premia. Avoid mixing a historically estimated process with risk-neutral valuation formulas without a measure-change argument.

The two interest-rate repositories are educational implementations associated with Damir Filipović's rate-modelling course. Their overlap means they are not independent empirical confirmations. They are starting points for curve and model exercises, not ready-made alpha systems. [R10, R11]

For trading research, the most transferable fixed-income ideas are factor exposures, curve shape, duration, convexity, and residual relative value. A price difference across instruments is meaningful only after their cash flows, optionality, and financing are aligned. [L09, fixed-income and futures chapters; L11]

## 7.4 Quanto credit default swaps

The Burgess slides explain credit protection settled in a currency different from the currency convention used for comparison. Premium and protection legs must be valued under a consistent measure, with dependence between FX and default. Both continuous covariance effects and an FX jump at default can contribute to a quanto adjustment. [L05]

The basic CDS logic balances discounted premium payments, including relevant accrual conventions, against expected protection payments after default. Changing the settlement currency changes the economic payoff and potentially the measure-specific default intensity. A simple historical FX correlation is not enough to identify the adjustment.

The slides present an illustrative jump-based scaling relation of the form s_quanto ≈ (1+γ)s_reference under their stated FX conventions. Its sign and interpretation depend on the currency quote and definition of γ. Do not reverse the quote direction while retaining the same formula.

A concrete arithmetic issue appears on PDF page 22, printed slide 21: a banking-sector FX jump of −35% is paired with a 55% multiplier. Under the slide's own 1+γ rule, the multiplier should be **65%**. This was checked visually. The worked sovereign example using 60% of 172.10 basis points gives 103.26 basis points, consistent with a −40% jump assumption.

The market anecdotes and quoted spreads are historical illustrations, not current prices. Some broad economic assertions in the deck are unsupported and should not be repeated as verified facts. The useful content is the dependence mechanism and pricing setup; empirical calibration needs appropriately timestamped CDS and FX data plus contractual details.

The connection to crypto is conceptual rather than a direct trading recipe. Exchange default, stablecoin depegging, collateral denomination, and asset returns can be dependent in stress. A portfolio that appears hedged in one unit of account can retain substantial joint credit/FX-like risk in another.

## 7.5 XVA and computational architecture

Andrey Chirikhin's 2021 presentation treats XVA as a hybrid path-dependent derivative on a portfolio and its collateral rules. Its main engineering proposal is to compress future trade values into regression coefficients, then reconstruct them in later simulations. Collateral dynamics become part of the payoff specification. This can reduce the transfer and storage of huge trade-by-scenario arrays. The original 33-slide transcript was accessible, though some displayed formulas were garbled. [W21]

For your framework, the transferable architecture is to separate reusable state generation, fitted valuation representations, and portfolio-level aggregation. Approximation error must be checked at future states as well as at today's price. A small average pricing error can conceal a large error in the stressed region that drives exposure or collateral.

CVA, DVA, funding, margin, and capital adjustments are related but not interchangeable concepts. Their definitions depend on contract, collateral, funding policy, and modelling choices. The presentation's regulatory context is historical; it should not be treated as an up-to-date legal or regulatory implementation guide.

## 7.6 Numerical methods and hardware acceleration

The linked Vitis quantitative-finance page is explicitly version 2020.1. It catalogues financial models and numerical engines, including closed-form methods, Monte Carlo, trees, and finite-difference approaches for models such as Black–Scholes, Heston, and several rate processes. It is a map of accelerated pricing computation, not a collection of demonstrated trading alphas. [W03]

Choose a numerical method based on dimension, payoff smoothness, path dependence, and error tolerance. Monte Carlo handles high-dimensional path-dependent payoffs but has sampling error. Trees and finite differences can be effective in lower-dimensional state spaces, with stability, boundary, and discretisation issues. Closed forms are fast when their assumptions fit the product.

Acceleration is valuable only after identifying the bottleneck. For minute-frequency signal research, data alignment and reproducibility may matter more than FPGA throughput. For a large scenario-pricing problem, approximation and memory layout may deliver more than a faster raw simulation kernel. Benchmark end-to-end runtime and numerical error, not only the innermost loop.

## 7.7 Polynomial chaos for discontinuous functions

The polynomial-chaos notebook reproduces work on approximating indicator functions with orthogonal polynomial expansions. It explores distribution-matched families, including Hermite, Legendre, Laguerre, and Jacobi constructions, and studies truncation error. “Indicator” here means a mathematical indicator such as 1_{X≥c}; it does not mean a technical trading indicator. [R12]

The relevant applications include digital-like payoffs, threshold events, and uncertainty quantification. A discontinuity can converge slowly or create oscillatory approximation behaviour. Validate error near the threshold and in the tail, not just average error across common states. Matching the polynomial family to an assumed distribution does not make the market follow that distribution.

This connects to XVA regression and signature representations through a shared idea: choose a function basis and estimate coefficients. Their bases, measures, and approximation guarantees differ. Do not transfer a convergence statement for one setting into another without checking its assumptions.

## 7.8 Where this track belongs in your roadmap

Keep derivative pricing and fixed-income modelling as a parallel research track if the immediate trading instruments are crypto spot. It strengthens risk reasoning, numerical methods, and future options research, but it should not delay basic signal validation and execution accounting.

The academic-paper directories for Stefano De Marco and Carol Alexander, the Mathematical Finance journal page, and the AQR research pages are valuable discovery routes. They were reviewed as indexes, not as fully read collections. Select individual papers by a concrete question, record the version, and add them to the source register when actually reviewed. [W17–W19, W30, W31]

---

# Paper 8 — Software, repositories, competitions, and research hubs

## 8.1 How to interpret this assessment

Repository access and documentation review are not a security audit, a successful installation, or a reproducible profitability result. This review inspected the supplied repositories' main documentation and selected notebooks where they were especially relevant. No repository was installed, connected to an exchange, or used to place an order. Status observations are a snapshot from 7 November 2026.

For software adoption, distinguish a research platform, a backtest engine, an execution bot, an indicator library, and a demonstration notebook. Many projects span several categories, but the missing pieces matter. A good feature library may have no order-state machine; an execution bot may provide little protection against research leakage.

## 8.2 Frameworks and production-oriented projects

| ID / project | Reviewed contribution | Decision-relevant limitation |
|---|---|---|
| R01 — Microsoft Qlib | Research workflow covering data, features, models, evaluation, portfolios, and execution research; useful separation of components. | Current main documentation mentions restricted-artifact-loading changes and temporarily unavailable official datasets. Pin a release and build a reproducible data route. Crypto or Revolut X integration was not established. |
| R14 — Freqtrade | Python crypto workflow with backtesting, optimisation, dry-run operation, and FreqAI-related facilities. | Listed venue support must be checked against the intended account and API; Revolut X support was not established by this review. Framework features do not validate a strategy's fills or alpha. |
| R15 — QuantConnect LEAN | Event-driven multi-asset engine with C# and Python interfaces and explicit modelling infrastructure. | More infrastructure than a small notebook study needs. Data, brokerage, and model configuration determine realism. The README was reviewed; no engine run was performed. |
| R20 — StockSharp | Broad C# ecosystem for data, strategy development, connectors, and graphical tooling. | Determine which components and services match the intended workflow and licensing. Connector breadth is not proof of a tested venue integration for this project. |
| R22 — Superalgos | Visual environment for data, strategy design, testing, and deployment workflows. | A graphical workflow still needs causal data and auditable accounting. Main documentation and metadata were reviewed, not every internal component. |
| R24 — Krypto-trading-bot | Configurable market-making infrastructure with a web interface and emphasis on speed. | Author speed claims are not measured end-to-end latency for your venue. Inventory, adverse selection, and market-data recovery need independent validation. |

Qlib is attractive as a conceptual template for separating data, model, portfolio, and execution layers. LEAN is a candidate when event-driven multi-asset simulation is the central requirement. Freqtrade is a candidate when its supported crypto execution workflow matches the chosen venue. These are conditional fit assessments, not a recommendation to migrate before checking the integration cost.

## 8.3 Specialist tools and legacy bots

| ID / project | What it offers | What to retain or verify |
|---|---|---|
| R02 — Signatory | Signature and log-signature operations integrated with PyTorch, including CPU/GPU support. | Reviewed README lists old Python/PyTorch compatibility ranges. Do not assume compatibility with the newest PyTorch. Benchmark a pinned environment before adopting it. |
| R16 — TriangularArbitrage | WebSocket quote processing, triangular opportunity calculations, and logging. | Treat it primarily as an opportunity monitor. Depth, sequential fills, latency, and fees on all legs determine executable results. |
| R17 — node-binance-trader | Historical Node/PostgreSQL framework for market data, signals, backtests, and trading. | README says archived/no longer maintained, although GitHub's archived flag was false in the retrieved metadata. Preserve that distinction; maintenance risk remains. |
| R18 — Kelp | Go-based market-making and liquidity tooling associated with Stellar and exchange integrations. | GitHub metadata marked the repository archived. Useful for design study, but not a default new deployment dependency. |
| R19 — Crypto-Signal | Technical-analysis indicators, market monitoring, and alerting. | An alert is not an executed, risk-managed trade. A separate ledger and execution policy are needed. |
| R23 — Zenbot | Historical Node/Mongo trading bot with strategy and paper-trading facilities. | GitHub metadata marked it archived; the retrieved last-push date was in 2022. Use as historical code reference with substantial compatibility caution. |
| R25 — Finance | Python examples and tooling across data, indicators, forecasting, backtesting, and portfolio analysis. | Useful breadth for learning. README behaviour is not an independent validation of its price alignment, costs, or borrow treatment. |

Maintenance status is a practical screening factor, not a declaration that every old project is unusable or every active project is safe. A small well-understood component can be easier to maintain than a large framework, but an exchange adapter requires ongoing attention to protocol changes.

## 8.4 Competitions and educational collections

**R04 — ACM UIC SIG-Algorithmic-Trading; R05 — Mohpann/AlgoTrading.** These provide educational strategy collections covering topics such as sentiment, reversion, options, and other trading ideas. Their documentation overlaps substantially. Treat them as related learning resources, not two independent confirmations of an edge. Convert each interesting example into a fully specified hypothesis before testing it.

**R06 — FinRL Contest 2025.** The repository supplies contest tasks and starter material involving stocks, crypto, LLM-related forecasting or factor workflows, and other competition components. It can help structure experiments. A contest's available information, objective, and simulator determine what its score means; production claims require separate evidence.

**R07 — Prosperity.** The README describes an IMC competition approach and a high ranking, including market making around stable synthetic anchors, regression, pair relationships, and basket reasoning. Rankings are self-reported in the reviewed repository. Synthetic products and competition rules can create relationships that do not exist in live crypto. Retain the disciplined inventory and product-specific reasoning, not the artificial fair values.

**R08 — Rank_3-Quant-Challenge-2025.** The README describes two LightGBM targets and cross-validation. The inspected final notebook also contains `train_test_split(..., shuffle=True)` and generic `GridSearchCV(cv=3)`. That distinction matters: this is not a chronological deployment test for serial financial data. It does not invalidate every possible competition setup, but the code should not be transplanted unchanged into a time-series alpha experiment. The inspected notebook blob SHA was `d49408b1f0aaf51203788aa1ab27c4c53240d901`.

**R13 — NK Securities volatility-curve prediction.** The README describes exploratory analysis and an ExtraTrees-based final approach with a claimed rank of 15. Only its documentation was reviewed here. For a volatility-surface adaptation, add positivity and relevant no-arbitrage checks, time-ordered validation, and sensitivity to sparse strikes and maturities before using predicted values in pricing or hedging.

## 8.5 Focused notebooks

**R03 — DermanPapers.** The selected discrete-hedging notebook is best used to study hedge error under model assumptions. It is not a live trading bot. Inspected blob: `99dbdfa47fef6ba07737fe8e60beabfbd0eb28fb`.

**R09 — Micro_Price.** The selected notebook combines imbalance and residual states in a transition-based price adjustment. Replace its instrument-specific ticks, states, and sample assumptions before applying the method elsewhere. Inspected blob: `dcbec1d03e12b76a6733ac0247ac97517373686c`.

**R10 and R11 — interest-rate course repositories.** These are related educational implementations. Use them for curve and pricing exercises with explicit conventions, not as independent evidence about trading performance.

**R12 — polynomial-chaos-indicator.** The large notebook was recovered through its Git blob after the ordinary file route did not return the contents. Its focus is orthogonal-polynomial approximation of mathematical indicator functions. Inspected blob: `587405ac25db3cf64c9e8ce08528d79b15570b2c`. Recovery of the notebook means it is not on the unreadable-file list.

## 8.6 Directories, communities, and publications

**R21 — best-of-algorithmic-trading.** This is a curated directory. Stars, activity, and collection rankings help discovery but do not measure economic performance, safety, or reproducibility. Linked child projects were not all audited.

**W02 — Seeking Alpha.** The supplied URL is a homepage, not a specified research article. It can help discover market narratives and candidate events; individual paywalled articles and the full archive were not reviewed.

**W17 and W18 — AQR Research and Perspectives.** These were reviewed as research-discovery indexes. They are suitable places to locate individual papers on factors, portfolio construction, and implementation. No claim is made that all indexed articles were read or that institutional results transfer to this trading setup.

**W19 — Mathematical Finance.** The journal landing page identifies a research venue rather than one supplied paper. An article-by-article literature review requires selecting and retrieving specific papers.

**W20 — Quantitative Finance Stack Exchange; W27 — Cross Validated AD discussion.** Community explanations can clarify terminology and implementation questions. They are leads to primary literature and documentation, not automatically authoritative results. The AD thread was accessible; technical explanations in this compendium rely on the teaching materials and independent derivation.

**W30 and W31 — De Marco and Alexander paper directories.** These provide author-organised bibliographies. They were read as indexes; individual linked papers were not all retrieved.

The Wilmott forum and Marcos Carreira repository-profile listing were inaccessible through the attempted routes. The individual Derman repository was accessible. A failure to read a profile page does not imply its separately linked repository could not be reviewed.

## 8.7 An adoption checklist tied to your project

Before choosing an engine, define the necessary instruments, order types, historical data, and decision frequency. Then check whether the engine can represent them without hiding assumptions. A minute-bar strategy and a queue-sensitive market maker should not share a simulator simply because both produce buy and sell signals.

Retain a small canonical ledger outside model code: cash, inventory, fills, fees, funding, realised and unrealised profit, and equity. Ensure research and replay use the same event and position conventions. Save model version, feature version, data cutoff, and configuration with each output.

Use a frozen dependency environment for a reproducible run. Pin the release or commit, record data checksums, and retain an installation note. None of the reviewed repositories' popularity or rankings removes the need for those steps.

Choose the smallest engine that faithfully represents the experiment, with a ledger independent of exchange adapters.

---

# Paper 9 — A concrete blueprint for later algorithmic-trading research

## 9.1 The intended outcome

The objective is a reproducible pipeline that can reject weak ideas cheaply and investigate stronger ones without changing definitions halfway through. This blueprint is an original synthesis of the reviewed material. All experiment specifications below are proposals, not completed tests or claims of profitability.

Keep three research tracks. The first uses bars for forecasting, regimes, and allocation. The second uses trades and order books for microstructure and execution. The third covers derivatives, rates, and scenario pricing. Share data standards and accounting where appropriate, but do not force evidence from one track to validate another.

For a first crypto implementation, use a small point-in-time liquid universe and simple feasible positions. Expand the asset set after data and accounting behave correctly. An example set of familiar symbols is a convenience for experiments, not a claim about their current venue availability or investment merits.

## 9.2 Data contracts

Each bar should include instrument identity, venue, interval start and end, timezone, open/high/low/close, base volume, quote volume where available, trade count where available, and a quality flag. Record whether the bar is final and when it becomes available. A bar named 12:00 might describe 12:00–12:01 or the minute ending at 12:00; this must not be guessed.

Maintain an instrument table with tick size, quantity step, minimum notional, contract multiplier, quote currency, listing interval, and relevant trading permissions. Derivatives need funding, expiry, settlement, and margin conventions. Exchange-specific limits must be verified before execution; they are not established by this review.

Store immutable raw records separately from corrected and aggregated data. Deduplicate by a declared key. Detect missing intervals, price/quantity anomalies, and inconsistent timestamps. Record corrections rather than overwriting evidence silently. Use data hashes and versioned manifests so a later result can be reproduced.

For cross-venue lead–lag work, record both source event time and local receipt time where possible. The exploitable lead is defined by what the strategy could have received before placing an order, not by retrospectively aligning ideal exchange clocks. Persistent price differences can reflect different currencies, fee structures, funding, or withdrawal constraints.

For macro and news features, preserve publication time, revision history, and timezone. A final downloaded series often contains revisions unavailable at the historical decision time. For sentiment, preserve the text version and timestamp used by the model.

## 9.3 Feature and target contracts

Every feature should declare its input fields, lookback, update time, units, missing-data treatment, and fitting requirements. Distinguish a deterministic trailing calculation from a learned transformation. PCA, clustering, scalers, winsorisation thresholds, imputation means, and feature selection are fitted components and belong inside training windows.

A forward log-return label can be y_(t,h) = log(P_(t+h)/P_t), but that price pair is not necessarily executable. Define the forecast label first, then define a trade-entry price and latency. If decisions occur at a completed bar close and trades execute on the next event, account for the gap between those timestamps.

For volatility, name the future interval and estimator. For order direction, define whether no-event intervals exist and how classes are balanced. For a triple-barrier label, declare which barrier is observed first and what happens when bar data cannot resolve ordering. Do not choose the convenient ordering retrospectively.

A useful feature registry includes lagged returns, trailing range, volatility, volume surprise relative to past seasonality, trend estimates, residual z-scores, breadth, spread, imbalance, and OFI when data support them. Each should have a neutral baseline and a small set of deliberately chosen horizons.

## 9.4 A causal validation layout

Use sequential training, validation, and test windows. For example, a proposed initial design might train on six months, validate on one month, and test on the next month, then roll forward. That schedule is a starting hypothesis; choose a cadence appropriate to the holding period, data volume, and model stability.

At a boundary, remove training labels whose future outcome interval reaches into the next evaluation block. If features or labels have more complicated event intervals, purge by actual interval overlap rather than by a guessed fixed row count. Any additional embargo should have a specific dependence rationale.

All assets sharing a market timestamp should respect the same temporal boundary. Splitting coins independently can allow a model to learn the same market shock from another asset's future observations. Cross-sectional transformations at time t are permissible only when all constituent inputs were actually available at t.

Hyperparameters and strategy thresholds are chosen on training/validation periods. The test period estimates the performance of that frozen choice. A final untouched period is reserved for the full selection procedure, not for repeatedly comparing variants until one looks good.

Report results by fold, asset, volatility regime, spread regime, and calendar period. Use appropriate dependence-aware uncertainty estimates. Do not use the number of minute rows as the number of independent observations when labels overlap heavily.

## 9.5 Costs and executable accounting

As an illustration using a 9-basis-point fee on each side, a buy and sell incurs approximately 18 basis points before spread, slippage, and market impact. This is an example assumption consistent with your earlier research, not verification of a current venue fee schedule. A 30-basis-point gross move leaves only about 12 basis points before those other costs.

When fills are already simulated at bid and ask, do not subtract the full spread again as a separate cost. When fills are simulated at midpoint, spread crossing must be represented somewhere. Likewise, determine whether fees are charged in base or quote currency and how that affects inventory and cash.

Use actual turnover, not a fixed fee per signal. Scaling in, partial exits, rebalancing, and forced liquidation all trade additional notional. Funding and borrow costs accrue with time and exposure. Do not treat a short position as free or automatically available.

For maker simulations, report results under conservative and optimistic fill assumptions, and identify the range as uncertainty rather than as two equally valid profit estimates. If the strategy only works when every touch fills immediately, that is a data requirement to resolve before further model optimisation.

## 9.6 Experiment register

| Experiment | Question and minimal design | Baseline and rejection signal |
|---|---|---|
| E01 — AR return baseline | Can lagged returns forecast strictly future returns at a few declared horizons? Fit inside rolling windows. | Zero return, persistence for price target. Reject if gains vanish after chronological validation. |
| E02 — Risk forecasting | Do GARCH or richer variance features beat a trailing/EWMA estimate? | Same future realised-variance target and scoring rule. Reject if complexity brings no stable calibration gain. |
| E03 — Trend | Does volatility-scaled trend persist after next-event entry and costs? | Simple momentum/MA rule with identical exits. Reject if only one sharp parameter choice works. |
| E04 — Causal reversion | Does a past-fitted residual forecast convergence of tradable holdings? | Fixed-threshold residual rule and random-entry controls. Reject if gains come from refitting the equilibrium. |
| E05 — Return/volume interaction | After a large drop, does low relative volume change future return distribution? | Match drop size, volatility, hour, and liquidity. Reject if the effect is only lower volatility or contemporaneous noise. |
| E06 — Regime conditioning | Does a filtered state improve a fixed signal? | Unconditioned signal with comparable exposure. Reject if state labels use smoothing or future returns. |
| E07 — OFI timing | How much same-interval explanatory power survives a strictly future target? | Lagged return, spread, imbalance, ordinary OFI. Reject if benefit disappears with realistic delay. |
| E08 — Microprice | Does a transition model improve future midpoint and executable markout? | Midpoint, weighted midpoint, simple regression. Reject if sparse states or unstable matrices drive gains. |
| E09 — Signature representation | Do shallow signatures add information beyond ordinary interactions? | Same inputs in regularised linear/tree models. Reject if extra channels explain the entire gain. |
| E10 — Allocation | Does covariance-aware allocation improve the same set of signals? | Equal capital and capped inverse volatility. Reject if turnover or weight instability consumes the benefit. |
| E11 — Execution policy | Does a limit/market policy improve implementation shortfall at fixed desired exposure? | Immediate execution and a simple schedule. Reject if simulated fill optimism explains results. |
| E12 — RL control | Can a learned policy improve independently marked account equity? | Fixed rules and simple recurrent policy in the same environment. Reject if reward and cash profit disagree. |

## 9.7 Controls that reveal false discoveries

Run a sign-flipped signal where economically meaningful and feasible. If both directions appear profitable under identical accounting, inspect the ledger, selection, or cost definition. The sign-flipped result need not be exactly the negative because costs, constraints, and stopping differ, but it should have an intelligible relationship.

Use matched random-entry controls with similar holding times, exposure, and trading conditions. A strategy may seem profitable simply because the sample market rose while it was usually long. Compare with exposure-matched passive returns as well as cash.

Delay entries by one or more events or bars. A collapsing result may indicate either a genuinely short-lived signal or an alignment error; the distinction requires data at the correct frequency. Do not preserve a supposedly minute-tradable edge by assuming fills before the signal is known.

Increase fees, spread, and latency within plausible ranges. Examine which cost term causes failure. Reduce position limits, remove the best asset, remove the best week, and evaluate nearby parameters. These sensitivity checks are useful when tied to a concrete concern; they should not become an unbounded search for a surviving variant.

Retain losing trades and unfilled orders. A portfolio view should reconcile exactly with the cash and inventory ledger. Include forced exits and end-of-sample liquidation. Decide beforehand whether a timeout exits at market or continues to hold; changing that rule after observing a recovery is hindsight.

## 9.8 Minimal implementation architecture

The data layer returns time-stamped observations and instrument metadata. The feature layer produces only available information. The model layer produces forecasts and uncertainty. The policy layer converts them into desired exposure. The portfolio layer resolves shared capital, risk, and constraints. The execution layer produces orders and fills. The ledger records resulting cash, inventory, costs, and equity.

Keep the model unaware of future fills and outcomes. Keep the ledger independent of the model so an attractive prediction cannot silently create profit. A common event interface can support historical replay, paper operation, and eventually live execution without changing economic definitions.

Persist each run's source-data identifiers, code version, dependency versions, feature configuration, model configuration, split intervals, cost assumptions, and random seeds. Save the complete decision and fill log for diagnostic samples. Summaries alone are insufficient when a result later looks suspicious.

## 9.9 What constitutes a useful result

A useful research result can be negative. Examples include finding that a strong OFI relationship is contemporaneous, that a regime filter adds no value after exposure matching, or that a model predicts volatility but not returns. Those findings prevent repeated wasted work and should be retained in the same register as promising results.

For a candidate to progress, require a reproducible chronological result, a feasible execution path, a plausible cost margin, stability across more than one market episode, and no unresolved accounting or timing defect. The exact statistical threshold should be chosen before the final evaluation, with the size of the research search in mind.

No criterion guarantees future profit. The purpose is to improve the quality of evidence and to make uncertainty visible. Research deployment can first mean replay and paper operation, where predicted fills, actual observable liquidity, and model drift can be compared without confusing a backtest with a live record.

## 9.10 Reading and implementation order

Start with Paper 1 and the corresponding Cryer–Chan chapters to establish definitions and forecasting discipline. Use Papers 2 and 6 to build risk estimates and an accounting-correct portfolio baseline. Use Paper 3 to select one mechanism, then implement the relevant experiment from this paper.

Read Paper 4 before collecting or modelling order-book data. Use Paper 5 when a simple baseline leaves a clear representation or memory problem. Use Paper 7 for pricing, hedging, and specialised risk questions. Consult Paper 8 only after the required workflow is clear enough to assess a framework's fit.

The source register and final access appendix preserve what was actually reviewed. The next literature update should add specific papers and their evidence status, not just more bookmarks. That turns the collection into a cumulative research record rather than a growing list of unrelated techniques.

---

# Appendix A — Corrections and remaining concept crosswalk

## A.1 Corrections to retain in your reference notes

| Original statement or implication | Corrected interpretation | Where developed |
|---|---|---|
| White noise is a normal random variable | Gaussianity is an additional assumption; white noise usually specifies second-order properties. | Paper 1 |
| AR stationarity requires roots outside the unit circle | This is the standard condition for the causal stationary representation. | Paper 1 |
| AR parameters must use Yule–Walker | Yule–Walker, least squares, likelihood and other estimators are available. | Paper 1 |
| MA and a moving average of prices are the same | MA(q) is a finite filter of innovations; a chart moving average is a filter of observations. | Paper 1 |
| ADF tests general stability | It tests a unit-root null under a specified regression, with power and specification limitations. | Paper 1 |
| Stationarity proves a reversion trade | Stationarity of a feature does not establish profitable future price reversal. | Papers 2 and 4 |
| ARIMA p always means the undifferenced polynomial's order | Standard ARIMA notation uses AR order after differencing. | Paper 1 |
| `sarima` is in R's base stats package | Base stats provides `ar` and `arima`; identify the package for a `sarima` function. | Paper 1 |
| Wold allows any deterministic trend | Its deterministic component remains within the covariance-stationary prediction framework. | Paper 1 |
| Hurst exponent is a rate-pricing model | It is a scaling/dependence parameter or statistic, with model-dependent interpretation. | Papers 2 and 7 |
| Portfolio volatility is the weighted sum of volatilities | Use sqrt(wᵀΣw); covariance matters. | Paper 6 |
| Diversification ratio divides by portfolio variance | The standard long-only ratio divides by portfolio standard deviation. | Paper 6 |
| Downside variance is downside deviation | Deviation requires the square root; specify the averaging convention. | Paper 6 |
| Sortino is the Jensen index | Sortino measures excess return per downside deviation; Jensen's alpha is benchmark-model performance. | Paper 6 |
| All asset statistics aggregate linearly | Returns and certain linear exposures can; volatility and many risk/performance ratios do not. | Paper 6 |
| Beta is total volatility | Beta is sensitivity to a specified benchmark, usually covariance divided by benchmark variance. | A.2 |
| OTM means over the money | It means out of the money. | A.3 |
| A call becomes profitable when its option price exceeds strike | Moneyness compares underlying price with strike; profit includes premium and costs. | Paper 7 / A.3 |
| Open interest guarantees liquidity | It measures outstanding contracts; executable depth and spread must be measured separately. | A.3 |
| Bid/ask are the last buy/sell order prices | Best bid/ask are the highest current bid and lowest current ask in the relevant book. | A.3 |
| Bid/ask quantity is number of orders | It usually measures displayed units or contracts at a price or across specified depth. | A.3 |
| IV guarantees the future size of price moves | IV is inferred from option prices under a convention/model, not a guaranteed realised outcome. | A.3 |
| Logistic regression is ordinary continuous-target regression | It is commonly used for classification probabilities. | Paper 5 |
| ECL means expected cure loss | ECL ordinarily means expected credit loss. Cure metrics need institution-specific definitions. | A.6 |
| High price R² establishes a trading edge | Compare persistence and future returns, then evaluate an executable policy. | Paper 5 |
| A bidirectional network necessarily leaks | Leakage depends on which observations lie in the decision's input window. | Paper 5 |
| A profitable RL reward equals profitable cash trading | Reconcile reward with a self-financing ledger and terminal inventory. | Paper 5 |
| A copula density can replace its CDF in conditional-CDF formulas | These are different objects; even the independence case demonstrates the error. | Paper 6 |
| A −35% jump implies a 55% multiplier under 1+γ | It implies 65%; the supplied quanto slide is arithmetically inconsistent. | Paper 7 |
| Zero maker fees mean free execution | Queue risk, adverse selection, inventory and liquidation remain. | Paper 4 |

These corrections concern the supplied notes and specific reviewed examples. They do not imply that every statement in every source was checked or that a source with one error has no useful material.

## A.2 Descriptive statistics and performance vocabulary

The arithmetic mean averages observations; the geometric mean of gross returns describes compound growth when those gross returns are positive. The harmonic mean weights reciprocals and is meaningful in particular rate-aggregation problems. Power means generalise several familiar means but do not create interchangeable estimators of investment performance.

Median and quantiles describe order rather than moments. Interquartile range is a robust spread measure. Average absolute deviation depends on the chosen centre. Coefficient of variation divides standard deviation by mean and becomes unstable or misleading near a zero or negative mean, as often occurs for short-horizon returns.

Skewness concerns asymmetry; kurtosis concerns a standardised fourth moment and is not simply “how peaked” a sample looks. Tail observations strongly influence both. State whether excess kurtosis subtracts the Gaussian value of three. L-moments are alternative summaries built from linear combinations of order-statistic expectations; their use still requires adequate data and a declared estimation method.

For a benchmark return M, beta_i = Cov(R_i,M)/Var(M). With fixed beginning-period weights and consistent observations, portfolio beta is the weighted sum of asset betas. Alpha is relative to a specified model and horizon. The Treynor ratio divides expected excess return by beta, making it unstable near zero beta and difficult to interpret for negative beta. The information ratio divides expected active return by tracking-error standard deviation. None should be compared across different horizons or benchmarks without alignment.

Statistical arbitrage describes trading based on statistical relationships under risk; it is not necessarily arbitrage in the strict no-risk sense. Statistical mechanics is a separate field whose tools can inspire market models, but an analogy does not establish a market law. [L08; independent mathematical clarifications]

## A.3 Option-chain terminology

A call gives its holder the contractual right to buy; a put gives the right to sell. European and American exercise conventions differ. For a call, in-the-money means underlying price exceeds strike; for a put it means underlying price is below strike. At-the-money means they are near or equal under the chosen convention. Out-of-the-money is the complementary intrinsic-value condition.

Open interest counts outstanding contracts, while volume counts traded contracts over a stated interval. A change in open interest depends on whether transactions open or close positions, and on exercise or expiry. It does not identify whether every trade is bullish or bearish.

Last traded price can be stale. Best bid and ask describe current displayed executable interest, subject to quantity, timing, and venue rules. A quote above the last trade need not imply a new directional signal. Displayed quantity is not the total hidden or future liquidity.

Implied volatility is the parameter that makes a chosen pricing convention match an observed option price, where such an inversion is well-defined. Different strikes and maturities produce a surface. IV embeds pricing, risk-premium, supply-demand, and model effects; it is not a promised realised volatility. [L08, L11]

## A.4 Stochastic-process map

The process names in the pasted outline are useful as a vocabulary, but they should be organised by what they model rather than memorised as interchangeable generators.

| Family | Names retained from the outline | Research role and limitation |
|---|---|---|
| Continuous Gaussian processes | Brownian/Wiener motion, Gaussian and Gauss–Markov processes, Brownian bridge | Diffusion shocks and conditional path simulation; Brownian and Wiener name the same standard process. |
| Constrained or conditioned paths | Bessel and squared-Bessel processes; Brownian excursion and meander | Useful in specialised boundary/conditioning problems; not default spot-return models. |
| Mean-reverting and state-dependent diffusions | OU, Vasicek, extended Vasicek, CIR, constant-elasticity-of-variance | Different drift, variance and boundary behaviour; use the model's actual assumptions. |
| Multiplicative diffusion | Geometric Brownian motion | Positive-price benchmark with restrictive return/volatility behaviour. |
| Jump and Lévy families | Poisson, gamma, inverse Gaussian, variance gamma, Cauchy, general Lévy processes | Discrete events and non-Gaussian increments; moments need not always exist. |
| Event intensities | Mixed Poisson, Cox, Hawkes, general point processes | Counts and clustered events; a Hawkes feedback mechanism requires stability and identification checks. |
| Long-memory/scaling constructions | Fractional Brownian motion, fractional Gaussian noise, multifractional Brownian motion | Dependence and local scaling; they are not automatically compatible with every Itô calculation. |
| Discrete dynamics | Bernoulli processes, random walks, Markov chains | Simple observations and state transitions; Markov is a property, not one distribution. |
| Population models | Birth–death, branching, Galton–Watson, Moran | Population/count mechanisms; relevance to markets needs an explicit analogy. |
| Bayesian random measures | Dirichlet process and Chinese restaurant construction | Flexible distributions/clustering; the construction should not be confused with an observed price process. |
| Broader random structures | Random fields, sets, matrices and media | Spatial, geometric, spectral or high-dimensional models; specialised extensions. |

A martingale satisfies E[M_(t+1) | F_t] = M_t under the relevant measure and integrability conditions. It is not a particular diffusion. Wald-type identities and optional-stopping arguments need their own assumptions; an arbitrary stopping rule does not make expected profit calculations automatically valid.

Noise colours describe spectral behaviour under conventions. White, pink, brown/red, blue, and violet noise are not a ranking of model quality. Fractional Gaussian noise is a specific increment process, while “coloured noise” is a broad description. A spectral label alone is insufficient to specify a return model.

Convergence in probability, almost surely, in distribution, and in mean square are different statements. A central limit theorem needs conditions; heavy tails and dependence can invalidate familiar approximations. Ergodicity relates time averages to distributional averages under assumptions. Conditional expectation and Radon–Nikodym derivatives underpin filtering and changes of measure. These are prerequisite concepts, not empirical evidence about a trading signal. [L08, L01, L02, L11]

## A.5 Methods and learning algorithms

Linear programming has linear objectives and constraints; quadratic programming adds a quadratic objective. Convexity can make global optimisation tractable, but estimation error remains. Stochastic programming handles uncertain future quantities; dynamic programming decomposes sequential decisions through a value function. Optimal stopping chooses when to act, while stochastic control chooses a process of actions.

Finite differences approximate derivatives on a grid. Finite elements approximate a solution in a chosen function space over a mesh. Ordinary and partial differential equations model different state dimensions, and neither guarantees an analytically solvable financial model. Least-squares Monte Carlo estimates conditional continuation values from simulated paths; it is a pricing/control method, not automatically a statistical-arbitrage strategy.

The supervised-learning list includes linear and regularised regression, support-vector methods, nearest neighbours, discriminant analysis, naive Bayes, trees, random forests, boosting, Gaussian-process regression, and neural networks. Their relevant differences include function class, regularisation, uncertainty, scaling, and computational cost. Lasso, ridge, and elastic net impose different coefficient penalties. Poisson regression models a count-related conditional mean under a link, not arbitrary continuous prices.

K-means, mean shift, DBSCAN, Gaussian mixtures, and hierarchical clustering define groups in different ways. Feature scaling and distance choices can dominate the resulting groups. A hidden Markov model additionally models temporal dependence in latent states. A mixture model alone does not enforce persistence.

Q-learning and SARSA differ in their value-update targets; temporal-difference learning is a broader family. DQNs approximate value functions with networks; deterministic actor–critic methods such as DDPG learn policies for continuous actions under their own assumptions. The useful comparison is empirical behaviour under the same faithful environment, not the sophistication of the name.

Keras, TensorFlow, scikit-learn, PyTorch, XGBoost, NLTK and spaCy are implementation ecosystems, not evidence of alpha. NLP, computer vision, image generation, rule engines and knowledge maps describe tasks or techniques. The Turing test and incidental chatbot/search/product links in the outline are background vocabulary; no trading claim follows from them. [L08, L11]

## A.6 Credit, insurance, and pensions

Probability of default, loss given default, and exposure at default describe different components of credit loss. ECL ordinarily expands to expected credit loss. Cure probabilities and recovery/cure quantities are model- or institution-specific; the outline's PC/LGC/EAC abbreviations should be defined against the actual data dictionary before use. Historical primers do not establish current regulatory requirements.

Life-insurance and pension modelling combine cash-flow timing, discounting, mortality, expenses, and sometimes multiple lives or states. A reserve is a valuation under specified assumptions, not simply cash already collected. Markov models can describe transitions such as active, disabled, retired, or deceased, with contract-specific benefits.

Non-life risk models often separate claim count from claim severity. In a simple compound-Poisson model, aggregate claims are the sum of random severities over a Poisson number of arrivals. Under its independence and finite-mean assumptions, expected claims over time t are λt E[X]. A classical positive net-profit condition compares premium inflow with that expectation; it does not imply zero ruin probability.

Extreme-value methods focus on tails, with threshold or block choices and substantial estimation uncertainty. Bayesian and credibility-style methods can combine prior and sample information. These ideas transfer to trading through scenario loss, tail dependence, and capital allocation, but an insurance claim process should not be assumed to describe market losses without evidence. [L08, L11]

---

# Appendix B — Strategy catalogue navigation index

Source: [L09], *151 Trading Strategies*. This index includes 174 strategy headings/variants and the weighted-regression reversion section. The count includes nested variants; it is not a count of independent validated alphas. Pages are **printed pages**; add one for the corresponding PDF page. Infrastructure is discussed in section 20, printed page 123, without individually numbered strategy headings.

Titles are retained for navigation. Paper 3 provides the economic synthesis. Illegal cash-related activities are identified only for completeness and excluded from research candidates.

## Options

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 2.2 | Covered call | 18 |
| 2.3 | Covered put | 18 |
| 2.4 | Protective put | 19 |
| 2.5 | Protective call | 19 |
| 2.6 | Bull call spread | 19 |
| 2.7 | Bull put spread | 20 |
| 2.8 | Bear call spread | 20 |
| 2.9 | Bear put spread | 20 |
| 2.10 | Long synthetic forward | 21 |
| 2.11 | Short synthetic forward | 21 |
| 2.12 | Long combo | 21 |
| 2.13 | Short combo | 22 |
| 2.14 | Bull call ladder | 22 |
| 2.15 | Bull put ladder | 23 |
| 2.16 | Bear call ladder | 23 |
| 2.17 | Bear put ladder | 23 |
| 2.18 | Calendar call spread | 24 |
| 2.19 | Calendar put spread | 24 |
| 2.20 | Diagonal call spread | 25 |
| 2.21 | Diagonal put spread | 25 |
| 2.22 | Long straddle | 26 |
| 2.23 | Long strangle | 26 |
| 2.24 | Long guts | 26 |
| 2.25 | Short straddle | 27 |
| 2.26 | Short strangle | 27 |
| 2.27 | Short guts | 27 |
| 2.28 | Long call synthetic straddle | 28 |
| 2.29 | Long put synthetic straddle | 28 |
| 2.30 | Short call synthetic straddle | 28 |
| 2.31 | Short put synthetic straddle | 29 |
| 2.32 | Covered short straddle | 29 |
| 2.33 | Covered short strangle | 29 |
| 2.34 | Strap | 30 |
| 2.35 | Strip | 30 |
| 2.36 | Call ratio backspread | 30 |
| 2.37 | Put ratio backspread | 31 |
| 2.38 | Ratio call spread | 31 |
| 2.39 | Ratio put spread | 31 |
| 2.40 | Long call butterfly | 32 |
| 2.40.1 | Modified call butterfly | 32 |
| 2.41 | Long put butterfly | 32 |
| 2.41.1 | Modified put butterfly | 33 |
| 2.42 | Short call butterfly | 33 |
| 2.43 | Short put butterfly | 33 |
| 2.44 | “Long” iron butterfly | 34 |
| 2.45 | “Short” iron butterfly | 34 |
| 2.46 | Long call condor | 34 |
| 2.47 | Long put condor | 35 |
| 2.48 | Short call condor | 35 |
| 2.49 | Short put condor | 35 |
| 2.50 | Long iron condor | 36 |
| 2.51 | Short iron condor | 36 |
| 2.52 | Long box | 37 |
| 2.53 | Collar | 37 |
| 2.54 | Bullish short seagull spread | 37 |
| 2.55 | Bearish long seagull spread | 38 |
| 2.56 | Bearish short seagull spread | 38 |
| 2.57 | Bullish long seagull spread | 39 |

## Stocks

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 3.1 | Price-momentum | 40 |
| 3.2 | Earnings-momentum | 41 |
| 3.3 | Value | 42 |
| 3.4 | Low-volatility anomaly | 42 |
| 3.5 | Implied volatility | 43 |
| 3.6 | Multifactor portfolio | 43 |
| 3.7 | Residual momentum | 44 |
| 3.8 | Pairs trading | 45 |
| 3.9 | Mean-reversion – single cluster | 46 |
| 3.9.1 | Mean-reversion – multiple clusters | 47 |
| 3.10 | Mean-reversion – weighted regression | 49 |
| 3.11 | Single moving average | 49 |
| 3.12 | Two moving averages | 50 |
| 3.13 | Three moving averages | 51 |
| 3.14 | Support and resistance | 51 |
| 3.15 | Channel | 52 |
| 3.16 | Event-driven – M&A | 52 |
| 3.17 | Machine learning – single-stock KNN | 53 |
| 3.18 | Statistical arbitrage – optimization | 55 |
| 3.19 | Market-making | 58 |
| 3.20 | Alpha combos | 59 |

## ETFs

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 4.1 | Sector momentum rotation | 61 |
| 4.1.1 | Sector momentum rotation with MA filter | 62 |
| 4.1.2 | Dual-momentum sector rotation | 62 |
| 4.2 | Alpha rotation | 63 |
| 4.3 | R-squared | 63 |
| 4.4 | Mean-reversion | 64 |
| 4.5 | Leveraged ETFs (LETFs) | 64 |
| 4.6 | Multi-asset trend following | 65 |

## Fixed income

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 5.2 | Bullets | 69 |
| 5.3 | Barbells | 69 |
| 5.4 | Ladders | 70 |
| 5.5 | Bond immunization | 70 |
| 5.6 | Dollar-duration-neutral butterfly | 71 |
| 5.7 | Fifty-fifty butterfly | 72 |
| 5.8 | Regression-weighted butterfly | 72 |
| 5.8.1 | Maturity-weighted butterfly | 73 |
| 5.9 | Low-risk factor | 73 |
| 5.10 | Value factor | 73 |
| 5.11 | Carry factor | 74 |
| 5.12 | Rolling down the yield curve | 74 |
| 5.13 | Yield curve spread (flatteners & steepeners) | 75 |
| 5.14 | CDS basis arbitrage | 75 |
| 5.15 | Swap-spread arbitrage | 76 |

## Indexes

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 6.2 | Cash-and-carry arbitrage | 77 |
| 6.3 | Dispersion trading in equity indexes | 77 |
| 6.3.1 | Dispersion trading – subset portfolio | 78 |
| 6.4 | Intraday arbitrage between index ETFs | 79 |
| 6.5 | Index volatility targeting with risk-free asset | 80 |

## Volatility

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 7.2 | VIX futures basis trading | 81 |
| 7.3 | Volatility carry with two ETNs | 82 |
| 7.3.1 | Hedging short VXX with VIX futures | 82 |
| 7.4 | Volatility risk premium | 83 |
| 7.4.1 | Volatility risk premium with Gamma hedging | 83 |
| 7.5 | Volatility skew – long risk reversal | 84 |
| 7.6 | Volatility trading with variance swaps | 84 |

## Foreign exchange

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 8.1 | Moving averages with HP filter | 85 |
| 8.2 | Carry trade | 86 |
| 8.2.1 | High-minus-low carry | 87 |
| 8.3 | Dollar carry trade | 88 |
| 8.4 | Momentum & carry combo | 88 |
| 8.5 | FX triangular arbitrage | 89 |

## Commodities

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 9.1 | Roll yields | 89 |
| 9.2 | Trading based on hedging pressure | 90 |
| 9.3 | Portfolio diversification with commodities | 90 |
| 9.4 | Value | 90 |
| 9.5 | Skewness premium | 91 |
| 9.6 | Trading with pricing models | 91 |

## Futures

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 10.1 | Hedging risk with futures | 92 |
| 10.1.1 | Cross-hedging | 93 |
| 10.1.2 | Interest rate risk hedging | 93 |
| 10.2 | Calendar spread | 94 |
| 10.3 | Contrarian trading (mean-reversion) | 95 |
| 10.3.1 | Contrarian trading – market activity | 95 |
| 10.4 | Trend following (momentum) | 96 |

## Structured assets

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 11.2 | Carry, equity tranche – index hedging | 99 |
| 11.3 | Carry, senior/mezzanine – index hedging | 99 |
| 11.4 | Carry – tranche hedging | 99 |
| 11.5 | Carry – CDS hedging | 100 |
| 11.6 | CDOs – curve trades | 100 |
| 11.7 | Mortgage-backed security (MBS) trading | 100 |

## Convertibles

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 12.1 | Convertible arbitrage | 101 |
| 12.2 | Convertible option-adjusted spread | 102 |

## Tax-related strategies

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 13.1 | Municipal bond tax arbitrage | 102 |
| 13.2 | Cross-border tax arbitrage | 103 |
| 13.2.1 | Cross-border tax arbitrage with options | 104 |

## Miscellaneous assets

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 14.1 | Inflation hedging – inflation swaps | 104 |
| 14.2 | TIPS-Treasury arbitrage | 105 |
| 14.3 | Weather risk – demand hedging | 106 |
| 14.4 | Energy – spark spread | 108 |

## Distressed assets

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 15.1 | Buying and holding distressed debt | 108 |
| 15.2 | Active distressed investing | 109 |
| 15.2.1 | Planning a reorganization | 109 |
| 15.2.2 | Buying outstanding debt | 109 |
| 15.2.3 | Loan-to-own | 110 |
| 15.3 | Distress risk puzzle | 110 |
| 15.3.1 | Distress risk puzzle – risk management | 110 |

## Real estate

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 16.2 | Mixed-asset diversification with real estate | 111 |
| 16.3 | Intra-asset diversification within real estate | 112 |
| 16.3.1 | Property type diversification | 112 |
| 16.3.2 | Economic diversification | 112 |
| 16.3.3 | Property type and geographic diversification | 113 |
| 16.4 | Real estate momentum – regional approach | 113 |
| 16.5 | Inflation hedging with real estate | 113 |
| 16.6 | Fix-and-flip | 114 |

## Cash

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 17.2 | Money laundering – the dark side of cash — excluded from implementation | 114 |
| 17.3 | Liquidity management | 115 |
| 17.4 | Repurchase agreement (REPO) | 115 |
| 17.5 | Pawnbroking | 115 |
| 17.6 | Loan sharking — excluded from implementation | 116 |

## Cryptocurrencies

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 18.2 | Artificial neural network (ANN) | 116 |
| 18.3 | Sentiment analysis – na ̈ıve Bayes Bernoulli | 120 |

## Global macro

| Section | Source strategy heading | Printed page |
|---|---|---:|
| 19.2 | Fundamental macro momentum | 122 |
| 19.3 | Global macro inflation hedge | 122 |
| 19.4 | Global fixed-income strategy | 123 |
| 19.5 | Trading on economic announcements | 123 |

---

# Appendix C — Source register

Coverage: 11 supplied attachments (10 PDFs totalling 1,075 pages and one text file), 78 distinct supplied web addresses, the pasted AR/MA notes, and five supplementary verification/recovery references. The repeated XVA address is counted once. The AR and MA words “link” did not include actual URLs.

Available means retrievable, not exhaustively proof-checked. See the review scope for each source. Repository descriptions are documentation observations, not installation or profitability validation.

## Supplied attachments

| ID | Source | Access and review scope |
|---|---|---|
| L01 | [Cryer and Chan — Time Series Analysis With Applications in R, second edition](https://chatgpt.com/library?file_id=libfile_33f7c4dfe27c8191be7c5baf5ea0c7d8&account_id=personal) | **available.** All pages extracted; contents and selected theory/forecasting sections reviewed. Chapter-level synthesis, not every proof or exercise. |
| L02 | [Waagepetersen — A quick introduction to Markov chains and MCMC](https://chatgpt.com/library?file_id=libfile_47c2f37b6518819196e5a50bdfd8ccb3&account_id=personal) | **available.** Full short text reviewed; inference and convergence concepts summarised. |
| L03 | [Jan Obloj — Optimal Execution & Algorithmic Trading](https://chatgpt.com/library?file_id=libfile_4885ea8a64c08191a35dc15de2f35275&account_id=personal) | **available.** All 74 slides present on 26 pages; structure and selected execution/impact derivations reviewed. |
| L04 | [Cartea et al. — Statistical Predictions of Trading Strategies in Electronic Markets](https://chatgpt.com/library?file_id=libfile_4fc72a5cbffc8191a1cd5a88dbc29efc&account_id=personal) | **available.** Methods, features, split design and principal results reviewed; not every supplementary table. |
| L05 | [Burgess — Quanto Credit Default Swaps: Theory, Pricing & Practice](https://chatgpt.com/library?file_id=libfile_80dd9d7adecc81918cc0c6acd8a842f5&account_id=personal) | **available.** Slides reviewed; selected formula and arithmetic pages visually checked. |
| L06 | [Rodriguez Dominguez — A Portfolio’s Common Causal Conditional Risk-neutral PDE](https://chatgpt.com/library?file_id=libfile_9d2c2deca0a48191ac512b0d5d36936d&account_id=personal) | **available.** Full text reviewed; key density/CDF and portfolio-distribution concerns checked. |
| L07 | [Aadhitya et al. — Predicting Stock Market time-series data using CNN-LSTM Neural Network model](https://chatgpt.com/library?file_id=libfile_a3d4dba8b2148191bbcac490c127b140&account_id=personal) | **available.** Full text extracted; model, splits, preprocessing and metrics reviewed. |
| L08 | [Supplied quantitative-finance outline](https://chatgpt.com/library?file_id=libfile_aab90f52f3d48191992e4277a1ecba68&account_id=personal) | **available.** Full text reviewed; definitions and portfolio formulas corrected. Incidental AI-product bookmarks not expanded. |
| L09 | [Kakushadze and Serur — 151 Trading Strategies](https://chatgpt.com/library?file_id=libfile_c36fe878c3208191be4ec8928f2033b2&account_id=personal) | **available.** All pages extracted; complete contents indexed; strategy families and selected sections/backtest appendix reviewed. Not every equation or cited paper verified. |
| L10 | [Wang et al. — High-frequency Statistical Arbitrage Strategy Based on Stationarized Order Flow Imbalance](https://chatgpt.com/library?file_id=libfile_f277b49f5d048191a891614c0d15c7b5&account_id=personal) | **available.** Full short paper reviewed; definitions, regression timing, metrics and limits examined. |
| L11 | [Amit Kumar Jha — six quantitative-finance primers](https://chatgpt.com/library?file_id=libfile_fdb03be30f48819191c28c615a0e8b37&account_id=personal) | **available.** All pages extracted; six constituent primers identified and surveyed. Introductory, historical treatment. |

## Web pages and directories

| ID | Source | Access and review scope |
|---|---|---|
| W01 | [PM Research article at iijjfds/7/3/171](https://www.pm-research.com/content/iijjfds/7/3/171) | **unavailable.** 403 access failure; article identity and body could not be verified. |
| W02 | [Seeking Alpha homepage](https://seekingalpha.com/) | **index only.** Homepage reviewed; no specific article supplied; archive/paywalled content not reviewed. |
| W03 | [Vitis quantitative finance models and methods, 2020.1](https://xilinx.github.io/Vitis_Libraries/quantitative_finance/2020.1/models_and_methods.html) | **available.** Model/method catalogue reviewed; historical version, not every engine implementation. |
| W04 | [ARCH — Wikipedia](https://en.wikipedia.org/wiki/Autoregressive_conditional_heteroskedasticity) | **available.** Reference page accessible; technical synthesis anchored in supplied textbook and official documentation. |
| W05 | [Exponential smoothing — Wikipedia](https://en.wikipedia.org/wiki/Exponential_smoothing#Basic_(simple)_exponential_smoothing) | **available.** Reference page accessible; basic topic included in Paper 1. |
| W06 | [Box–Jenkins method — Wikipedia](https://en.wikipedia.org/wiki/Box%E2%80%93Jenkins_method) | **available.** Reference page accessible; method discussed from textbook foundations. |
| W07 | [Augmented Dickey–Fuller test — Wikipedia](https://en.wikipedia.org/wiki/Augmented_Dickey%E2%80%93Fuller_test) | **available.** Reference page accessible; test interpretation corrected in Paper 1. |
| W08 | [Bayesian information criterion — Wikipedia](https://en.wikipedia.org/wiki/Bayesian_information_criterion) | **available.** Reference page accessible; criterion and limitations included. |
| W09 | [Akaike information criterion — Wikipedia](https://en.wikipedia.org/wiki/Akaike_information_criterion) | **available.** Reference page accessible; criterion and limitations included. |
| W10 | [Score test — Wikipedia](https://en.wikipedia.org/wiki/Score_test) | **available.** Reference page accessible; likelihood-based test summarised. |
| W11 | [Least squares — Wikipedia](https://en.wikipedia.org/wiki/Least_squares) | **available.** Reference page accessible; estimation distinctions included. |
| W12 | [Maximum likelihood — Wikipedia](https://en.wikipedia.org/wiki/Maximum_likelihood_estimation) | **available.** Reference page accessible; estimation distinctions included. |
| W13 | [Volatility clustering — Wikipedia](https://en.wikipedia.org/wiki/Volatility_clustering) | **available.** Reference page accessible; conditional-variance discussion in Paper 2. |
| W14 | [Oxford REF handwriting signature case study](https://results2021.ref.ac.uk/impact/54246aa8-a596-4952-be86-4f536d9360d0?page=1) | **recovered.** Landing page thin; five-page case-study PDF recovered and reviewed at application level. |
| W15 | [QuantStart rough-path signatures, Part 1](https://www.quantstart.com/articles/rough-path-theory-and-signatures-applied-to-quantitative-finance-part-1/) | **recovered.** Initial fetch failed; article recovered through search. Introductory overview used alongside primary papers. |
| W16 | [Shared Dropbox folder](https://www.dropbox.com/scl/fo/ms2mieh6cib84kuar9qj4/AMByEV1JNerhABk7qO0TnpI?rlkey=mgngaeef94yosmu235suhtdd6&e=1&dl=0) | **unavailable.** Folder contents and file list could not be retrieved; no claim about what it contains. |
| W17 | [AQR Research](https://www.aqr.com/Insights/Research) | **index only.** Research index reviewed; all linked papers not individually read. |
| W18 | [AQR Perspectives](https://www.aqr.com/Insights/Perspectives) | **index only.** Perspectives index reviewed; all linked articles not individually read. |
| W19 | [Mathematical Finance journal](https://onlinelibrary.wiley.com/journal/14679965) | **index only.** Journal homepage reviewed; full journal archive not obtained. |
| W20 | [Quantitative Finance Stack Exchange](https://quant.stackexchange.com/) | **index only.** Community homepage reviewed; individual questions not comprehensively reviewed. |
| W21 | [Andrey Chirikhin — XVA, The Front Office Way](https://www.slideshare.net/slideshow/xva-the-front-office-way/248296229) | **partial.** Original 33-slide text transcript reviewed; some equation glyphs corrupted. Duplicate supplied URL counted once. |
| W22 | [Wilmott forum](https://forum.wilmott.com/) | **unavailable.** 403 access failure; forum content not reviewed. |
| W23 | [Marcos Carreira repository listing](https://github.com/MarcosCarreira?tab=repositories) | **unavailable.** Profile/listing retrieval failed. Separately supplied DermanPapers repository was accessible. |
| W24 | [Quantitative Risk Management tutorial](https://qrmtutorial.org/) | **index only.** Course hub reviewed; all slides, videos, exercises and R code not reviewed. |
| W25 | [arch univariate documentation, supplied legacy path](https://bashtage.github.io/arch/doc/univariate/univariate.html) | **available.** Mean/volatility/distribution and forecasting documentation reviewed at overview level; retrieved build is old. |
| W26 | [Auto-ED reverse-mode automatic differentiation](https://auto-ed.readthedocs.io/en/latest/mod3.html) | **available.** Tutorial overview and computational-graph explanation reviewed. |
| W27 | [Cross Validated reverse-mode AD example](https://stats.stackexchange.com/questions/224140/step-by-step-example-of-reverse-mode-automatic-differentiation) | **available.** Discussion accessible; used as a learning lead rather than primary technical authority. |
| W28 | [Paszke et al. — Automatic differentiation in PyTorch slides](https://autodiff-workshop.github.io/slides/Paszke_ad_in_pytorch.pdf) | **available.** 57-page slide PDF accessible; selected AD concepts reviewed; not every slide/API audited. |
| W29 | [Reverse Labouchere — No Nonsense Forex](https://nononsenseforex.com/uncategorized/reverse-labouchere/) | **available.** Article reviewed; author distinguishes betting progression from own trading use. |
| W30 | [Stefano De Marco academic page](https://www.cmap.polytechnique.fr/~demarco/) | **index only.** Research directory surveyed; every linked work not reviewed. |
| W31 | [Carol Alexander academic papers](https://www.coalexander.com/academic-papers) | **index only.** Research directory surveyed; every linked paper not reviewed. |

## Research papers and presentations

| ID | Source | Access and review scope |
|---|---|---|
| P01 | [Lyons — Differential equations driven by rough signals (1998)](http://dmle.icmat.es/pdf/MATEMATICAIBEROAMERICANA_1998_14_02_01.pdf) | **partial.** Original host failed; 96-page publisher PDF recovered, but mathematical text encoding is damaged. Conceptual review, not proof verification. |
| P02 | [Deep Signature Transforms](https://arxiv.org/abs/1905.08494) | **available.** Full HTML available; abstract, architecture and selected sections reviewed. |
| P03 | [The Signature Kernel is the solution of a Goursat PDE](https://arxiv.org/abs/2006.14794) | **available.** Full HTML available; kernel construction and Bitcoin forecasting example reviewed. |
| P04 | [Volatility is rough](https://arxiv.org/abs/1410.3394) | **available.** Full HTML available; empirical roughness and modelling claims reviewed. |
| P05 | [Gamma Hedging and Rough Paths](https://arxiv.org/abs/2309.05054) | **available.** Full HTML available; abstract and selected theoretical discussion reviewed, not all proofs. |
| P06 | [A Neural RDE approach for continuous-time non-Markovian stochastic control problems](https://arxiv.org/abs/2306.14258) | **available.** Full HTML available; architecture, control framing and selected experiments reviewed. |
| P07 | [Stochastic control with rough paths](https://arxiv.org/abs/1303.7160) | **available.** Full HTML available; theoretical scope and selected discussion reviewed, not all proofs. |
| P08 | [Theoretical Foundations of Deep Selective State-Space Models](https://arxiv.org/abs/2402.19047) | **available.** Full HTML available; theoretical framing reviewed, not an empirical finance validation. |
| P09 | [G-Signatures: Global Graph Propagation With Randomized Signatures](https://arxiv.org/abs/2302.08811) | **available.** 27-page PDF available; abstract and method overview reviewed. |
| P10 | [Signature-based validation of real-world economic scenarios](https://arxiv.org/abs/2208.07251) | **available.** Full HTML available; testing/scenario-validation approach reviewed. |
| P11 | [Knuteson — Celebrating Three Decades of Worldwide Stock Market Manipulation](https://arxiv.org/abs/1912.01708) | **recovered.** Abstract route failed; eight-page PDF recovered. Argument and introduction reviewed; causal allegations not accepted as established facts. |
| P12 | [Schur Complementary Portfolios 2025 slides](https://github.com/microprediction/home/blob/main/presentations/Schur_Complementary_Portfolios_2025__slides_.pdf) | **unavailable.** Exact file identified, but binary PDF could not be retrieved/decoded. Related primary paper and author notes reviewed instead; see S03–S04. |
| P13 | [Ning and Lee — Advanced Statistical Arbitrage with Reinforcement Learning](https://arxiv.org/html/2403.12180v1) | **available.** Full HTML available; spread, reward, simulator and reported experiments reviewed. |
| P14 | [Mean Field Game of High-Frequency Anticipatory Trading](https://arxiv.org/abs/2404.18200) | **available.** Full HTML available; model and main implications reviewed, not every proof. |
| P15 | [Mean Field Games with Partial Information for Algorithmic Trading](https://arxiv.org/abs/1803.04094) | **available.** Full HTML available; filtering/control and equilibrium scope reviewed. |
| P16 | [Schrödinger Bridge Problem for Jump Diffusions](https://arxiv.org/abs/2411.13765) | **available.** Full HTML available; abstract and mathematical scope reviewed, not a complete proof audit. |

## Repositories

| ID | Source | Access and review scope |
|---|---|---|
| R01 | [microsoft/qlib](https://github.com/microsoft/qlib) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R02 | [patrick-kidger/signatory](https://github.com/patrick-kidger/signatory) | **available.** README/documentation reviewed. Recovered README.rst; initial README.md route failed. |
| R03 | [MarcosCarreira/DermanPapers](https://github.com/MarcosCarreira/DermanPapers) | **available.** README/documentation reviewed. Also inspected DiscreteHedging.ipynb. |
| R04 | [acm-uic/SIG-Algorithmic-Trading](https://github.com/acm-uic/SIG-Algorithmic-Trading) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R05 | [Mohpann/AlgoTrading](https://github.com/Mohpann/AlgoTrading) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R06 | [Open-Finance-Lab/FinRL_Contest_2025](https://github.com/Open-Finance-Lab/FinRL_Contest_2025/tree/main) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R07 | [Harshal73/Prosperity](https://github.com/Harshal73/Prosperity/tree/main) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R08 | [vamshikrishna-1234/Rank_3-Quant-Challenge-2025](https://github.com/vamshikrishna-1234/Rank_3-Quant-Challenge-2025) | **available.** README/documentation reviewed. Also inspected final submission notebook, including split code. |
| R09 | [xhshenxin/Micro_Price](https://github.com/xhshenxin/Micro_Price) | **available.** README/documentation reviewed. Also inspected 4. Micro_price.ipynb. |
| R10 | [jingmouren/lbegatti-interest-rate-modelling](https://github.com/jingmouren/lbegatti-interest-rate-modelling) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R11 | [danullm/InterestRates](https://github.com/danullm/InterestRates) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R12 | [fbourgey/polynomial-chaos-indicator](https://github.com/fbourgey/polynomial-chaos-indicator/blob/main/main.ipynb) | **available.** README/documentation reviewed. Also recovered and inspected main.ipynb through blob retrieval. |
| R13 | [vamshikrishna-1234/NK-SECURITIES-RANK_15-VOLATALITY-CURVE-PREDICTION](https://github.com/vamshikrishna-1234/NK-SECURITIES-RANK_15-VOLATALITY-CURVE-PREDICTION) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R14 | [freqtrade/freqtrade](https://github.com/freqtrade/freqtrade) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R15 | [QuantConnect/Lean](https://github.com/QuantConnect/Lean) | **available.** README/documentation reviewed. Recovered lowercase readme.md. |
| R16 | [tiagosiebler/TriangularArbitrage](https://github.com/tiagosiebler/TriangularArbitrage) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R17 | [bitcoinvsalts/node-binance-trader](https://github.com/bitcoinvsalts/node-binance-trader) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R18 | [stellar-deprecated/kelp](https://github.com/stellar-deprecated/kelp) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R19 | [CryptoSignal/Crypto-Signal](https://github.com/CryptoSignal/Crypto-Signal) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R20 | [StockSharp/StockSharp](https://github.com/StockSharp/StockSharp) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R21 | [joshmiddleton123-ux/best-of-algorithmic-trading](https://github.com/joshmiddleton123-ux/best-of-algorithmic-trading#bots--frameworks) | **available.** README/documentation reviewed. Directory only; child repositories not comprehensively audited. |
| R22 | [Superalgos/Superalgos](https://github.com/Superalgos/Superalgos) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R23 | [DeviaVir/zenbot](https://github.com/DeviaVir/zenbot) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R24 | [ctubio/Krypto-trading-bot](https://github.com/ctubio/Krypto-trading-bot) | **available.** README/documentation reviewed. No code execution or full code audit. |
| R25 | [shashankvemuri/Finance](https://github.com/shashankvemuri/Finance) | **available.** README/documentation reviewed. No code execution or full code audit. |

## Books

| ID | Source | Access and review scope |
|---|---|---|
| B01 | [Friz and Victoir — Multidimensional Stochastic Processes as Rough Paths](https://www.cambridge.org/core/books/multidimensional-stochastic-processes-as-rough-paths/DD9063039477D9A21F571861A5BB5D53) | **index only.** Book landing page and contents available; full book not obtained. |
| B02 | [Forecasting: Principles and Practice, second edition](https://otexts.com/fpp2/) | **available.** Book index and selected toolbox/accuracy material reviewed; not every chapter. |

## Videos

| ID | Source | Access and review scope |
|---|---|---|
| V01 | [Horvath — Deep Learning (Rough) Volatility](https://www.youtube.com/watch?v=O03erV5nYXA) | **unavailable.** Video content/transcript not retrievable. A page shell or metadata is not a viewing of the material. |
| V02 | [Gatheral — Rough volatility: An overview](https://www.youtube.com/watch?v=gW073Tnx7CE) | **unavailable.** Video content/transcript not retrievable. A page shell or metadata is not a viewing of the material. |
| V03 | [Oxford MCF YouTube channel](https://www.youtube.com/@OxfordMCF) | **unavailable.** Video content/transcript not retrievable. A page shell or metadata is not a viewing of the material. |
| V04 | [Quantitative finance seminar playlist](https://www.youtube.com/playlist?list=PLArBKNfJxuumDwk6qbcJEOC7GTlj63Oro) | **unavailable.** Video content/transcript not retrievable. A page shell or metadata is not a viewing of the material. |

## Supplementary checks and recovered sources

| ID | Source | Access and review scope |
|---|---|---|
| S01 | [Official statsmodels AutoReg documentation](https://www.statsmodels.org/stable/generated/statsmodels.tsa.ar_model.AutoReg.html) | **available.** API and alignment/hold-back details checked. |
| S02 | [Official statsmodels ARIMA documentation](https://www.statsmodels.org/stable/generated/statsmodels.tsa.arima.model.ARIMA.html) | **available.** Current API path checked. |
| S03 | [Cotton — Schur Complementary Allocation](https://arxiv.org/pdf/2411.05807) | **available.** Related 14-page primary paper recovered; selected derivations reviewed. Does not replace the unread exact slides. |
| S04 | [Schur author introduction and current clarification](https://schur.microprediction.org/intro.html) | **available.** Author explanation checked, including split-rule caveat for exact minimum-variance recovery. |
| S05 | [Publisher recovery of Lyons 1998](https://ems.press/journals/rmi/articles/5137) | **partial.** Publisher page and linked PDF recovered; mathematical text extraction remains damaged. |

---

# Appendix D — Pages and material that could not be fully read

This is the final appendix requested by the user. Access failures are distinguished from directories whose entire contents were never a single readable page. No missing source has been silently treated as reviewed. Access was checked on 7 November 2026.

## D.1 Exact supplied sources whose substantive content was unavailable

| Source | What could not be read | Outcome |
|---|---|---|
| [PM Research article](https://www.pm-research.com/content/iijjfds/7/3/171) — W01 | Article body; even its identity could not be verified reliably | Access returned 403. No article summary was invented. |
| [Dropbox folder](https://www.dropbox.com/scl/fo/ms2mieh6cib84kuar9qj4/AMByEV1JNerhABk7qO0TnpI?rlkey=mgngaeef94yosmu235suhtdd6&e=1&dl=0) — W16 | Folder listing and files | Retrieval failed. This is not evidence that the folder is empty. |
| [Wilmott forum](https://forum.wilmott.com/) — W22 | Forum content | Access returned 403. |
| [Marcos Carreira repository listing](https://github.com/MarcosCarreira?tab=repositories) — W23 | Profile's repository inventory | Listing retrieval failed; the separately linked DermanPapers repository was read. |
| [Schur Complementary Portfolios slides](https://github.com/microprediction/home/blob/main/presentations/Schur_Complementary_Portfolios_2025__slides_.pdf) — P12 | Exact slide PDF | File existence was verified, but binary content could not be retrieved/decoded. Related primary paper S03 and author explanation S04 were reviewed instead. |
| [Horvath rough-volatility video](https://www.youtube.com/watch?v=O03erV5nYXA) — V01 | Video and transcript | Not retrievable. |
| [Gatheral rough-volatility overview](https://www.youtube.com/watch?v=gW073Tnx7CE) — V02 | Video and transcript | Not retrievable. |
| [Oxford MCF channel](https://www.youtube.com/@OxfordMCF) — V03 | Individual videos and transcripts | Only a page shell/limited metadata was available. |
| [Seminar playlist](https://www.youtube.com/playlist?list=PLArBKNfJxuumDwk6qbcJEOC7GTlj63Oro) — V04 | Playlist's substantive video content | Only a page shell/limited metadata was available. |

## D.2 Partially readable sources and recovered alternatives

- **Lyons 1998, P01:** the [original host](http://dmle.icmat.es/pdf/MATEMATICAIBEROAMERICANA_1998_14_02_01.pdf) failed. The [publisher page](https://ems.press/journals/rmi/articles/5137) and linked 96-page PDF were recovered. Many extracted mathematical symbols are damaged; the full derivations were not reliably read or verified. A screenshot attempt was unavailable through that route.
- **XVA slides, W21:** the original 33-slide transcript was readable. Some mathematical expressions, including material in slides 10, 15 and 23, were corrupted in extracted text. The architectural summary is based on readable original slide text; it is not a reconstruction of every equation. The duplicated supplied address was processed once.
- **Cambridge rough-path textbook, B01:** the [landing page and contents](https://www.cambridge.org/core/books/multidimensional-stochastic-processes-as-rough-paths/DD9063039477D9A21F571861A5BB5D53) were accessible; the full book was not obtained.
- **AR and MA placeholders:** the pasted notes say “link” but supply no target URL for those two placeholders. Their mathematical content was reviewed against the supplied textbook and the definitions in the notes.

## D.3 Readable indexes whose entire archives were not reviewed

The [Seeking Alpha homepage](https://seekingalpha.com/), [AQR Research](https://www.aqr.com/Insights/Research), [AQR Perspectives](https://www.aqr.com/Insights/Perspectives), [Mathematical Finance journal](https://onlinelibrary.wiley.com/journal/14679965), [Quantitative Finance Stack Exchange](https://quant.stackexchange.com/), [QRM tutorial hub](https://qrmtutorial.org/), [De Marco bibliography](https://www.cmap.polytechnique.fr/~demarco/), [Alexander bibliography](https://www.coalexander.com/academic-papers), and [best-of-algorithmic-trading directory](https://github.com/joshmiddleton123-ux/best-of-algorithmic-trading#bots--frameworks) were reviewed at index level. Their nested papers, discussions, code, paywalled articles, and videos were not all individually read.

FPP2 was reviewed through its index and selected forecasting material, not every chapter. Repository review generally means README/documentation plus the explicitly identified notebooks. Mathematical papers with available full text were reviewed at the scope recorded in the register; full availability does not imply every proof was checked.

## D.4 Supplied attachments and successful recoveries

All ten supplied PDFs had extractable text across their 1,075 pages, and the supplied text file was readable. No entire supplied attachment remained inaccessible. Formula-heavy pages were not all visually verified; selected pages were checked where an interpretation or error depended on layout. The two sparse pages in the Cryer–Chan PDF were a series-editor page and a dedication, not missing chapters. The execution PDF contains all 74 slides across 26 PDF pages.

The QuantStart article, Oxford handwriting case-study PDF, Knuteson PDF, Signatory README, LEAN README, and polynomial-chaos notebook were recovered through alternative routes. They are therefore not counted as unreadable sources. The source register records those recoveries and the depth of review.
