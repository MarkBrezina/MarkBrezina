**Yes, I’d revisit the Heston project.** You already have the foundations of a pricing engine and experience fitting volatility surfaces. I’d rebuild enough of it to answer one trading question: **when Heston disagrees with an option quote it hasn’t seen, does that disagreement predict subsequent repricing you could capture after spreads and hedging costs?** Start with one underlying, one holding period, and a simpler surface fit as a benchmark.

I formatted the notes as a single GitHub-ready `README.md`. I couldn’t make a downloadable file available in this session. I treated the actuarial R package as an earlier proposal and the Heston pricing engine as your completed bachelor project. I translated the Danish annotations and added proposed test definitions where the sketches left choices open. The supplied sketch has blank probability cells, and the other figures appear as placeholders in the source notes.

---

# Quantitative Finance Research Notes

A research archive covering systematic trading, derivative pricing, market and credit risk, portfolio construction, and actuarial modelling. The notes distinguish established methods, proposed experiments, and titles collected for further reading.

## 1. Research tools and data

### Python ecosystem

| Tool | Role | Where it fits |
|---|---|---|
| **PyQL** | Enthought’s Cython bindings to the QuantLib C++ library. QuantLib also has its own Python bindings. | Instrument valuation, term structures, calendars, and model calibration. Check the original imports and versions when rebuilding old code. [PyQL](https://github.com/enthought/pyql) · [QuantLib](https://www.quantlib.org/) |
| **OpenBB** | Python interface for investment research and data access through provider extensions. | A convenient way to explore datasets and standardise research queries. Record the underlying provider, since coverage, entitlements, and timestamps depend on it. [OpenBB Python documentation](https://docs.openbb.co/python/) |
| **Vollib** | Black, Black–Scholes, and Black–Scholes–Merton pricing, Greeks, and implied volatility. Older code may import `py_vollib`. | A reference for validating vanilla prices and implied volatility inversion. [Vollib documentation](https://vollib.org/documentation.html) |
| **Blankly** | Framework for building, backtesting, and deploying algorithmic strategies through exchange integrations. | A reference for organising strategy callbacks and broker adapters. Validate the specific exchange integration, order types, and fill behaviour before relying on it. [Blankly](https://github.com/blankly-finance/blankly) |
| **Backtrader** | Event-driven strategy backtesting. | Evaluate indicators, signals, order rules, and portfolio state through time. Execution assumptions still need to match the venue and data. [Backtrader documentation](https://www.backtrader.com/docu/) |
| **Pyfolio** | Portfolio and strategy performance analysis, including tear sheets. | Analyse returns, positions, and transactions from a backtest or trading ledger. [Pyfolio Reloaded](https://github.com/stefan-jansen/pyfolio-reloaded) |
| **Empyrical** | Functions for common return, performance, and risk statistics. | Calculate comparable metrics across experiments. Record frequency, annualisation, benchmark, and risk-free rate conventions. [Empyrical Reloaded](https://github.com/stefan-jansen/empyrical-reloaded) |
| **Riskfolio-Lib** | Portfolio optimisation and strategic asset allocation. | Compare objectives and constraints under the same inputs, including variance- and tail-risk-based allocations. [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib) |
| **yfinance** | Python access to Yahoo Finance data. | Quick exploratory work. Check adjustment conventions, coverage, and provider terms before drawing conclusions or redistributing data. [yfinance documentation](https://ranaroussi.github.io/yfinance/) |
| **Interactive Brokers TWS API** | Broker interface through Trader Workstation or IB Gateway. | Retrieve entitled market data and manage orders, positions, and account state. Test contract identifiers, reconnect behaviour, and order reconciliation in a paper account. [IBKR TWS API](https://www.interactivebrokers.com/docs/tws-api/doc/introduction) · [IB Gateway](https://www.interactivebrokers.com/docs/tws-api/doc/architecture/the-trader-workstation/the-ib-gateway) |
| **Alpha Vantage** | Financial data API. `alpha_vantage` is a separate Python wrapper for it. | Pull data through the API directly or through the wrapper. Check endpoint entitlements and coverage for the intended dataset. [API documentation](https://www.alphavantage.co/documentation/) · [Python wrapper](https://github.com/RomelTorres/alpha_vantage) |

### From observation to experiment

- **Market events and live observations.** Record the instrument, venue, clock time, event time, data available at the decision, and what appeared unusual. A live observation can motivate a hypothesis. Its value comes from specifying what would count as evidence against it.
- **Price patterns.** Convert descriptions such as “three up days,” “a yearly low,” or “extreme behaviour” into computable definitions. Freeze the lookback, observation time, target, and eligible instruments before testing.
- **Data sources.** Record vendor, symbol mapping, timezone, adjustment policy, data release time, missing periods, and usage rights. For macroeconomic data, distinguish the observation period, first release, and subsequent revisions.
- **Literature and white papers.** Use papers and technical documentation to locate the model and its assumptions. Blogs, articles, search engines, and seminar notes are discovery sources. Follow technical claims back to the original paper or documentation where possible. [IMF eLibrary](https://www.elibrary.imf.org/) · [BIS research and publications](https://www.bis.org/list/research/index.htm)
- **AI and machine learning.** Keep prediction, regime estimation, scenario generation, and decision-making as separate model roles. For multimodal models that combine prices, macro data, or text, align each input by when it became available. Compare each modality alone against the combined model.
- **Conditional Kolmogorov complexity.** A reading lead for the shortest program that describes an object \(x\) when given \(y\) as auxiliary input. Exact complexity is generally uncomputable. A practical experiment could compare held-out predictive compression or description length with and without the extra information. [Kolmogorov complexity reference](https://pubmed.ncbi.nlm.nih.gov/33286182/). [PubMed](https://pubmed.ncbi.nlm.nih.gov/33286182/?utm_source=chatgpt.com)

A reproducible experiment keeps the **observation**, **hypothesis**, **model specification**, **data version**, **test period**, and **result** separate. If a hypothesis changes after examining results, give the revised experiment its own record and evaluation period.

## 2. How risk management evolved

The original historical notes closely follow Tobias Adrian’s *Risk Management and Regulation*. The history connects advances in financial modelling with changes in trading practice and in the way supervisors assess financial institutions. [Adrian, IMF, 2018](https://www.imf.org/en/publications/departmental-papers-policy-papers/issues/2018/08/01/risk-management-and-regulation-45664). [imf.org](https://www.imf.org/en/publications/departmental-papers-policy-papers/issues/2018/08/01/risk-management-and-regulation-45664?utm_source=chatgpt.com)

| Period | Development | Significance |
|---|---|---|
| **1970s** | Fischer Black, Myron Scholes, and Robert C. Merton developed foundational ideas in option valuation and dynamic replication. | A portfolio of an option and its hedge can be analysed through its sensitivity to underlying risk factors. [Black and Scholes](https://doi.org/10.1086/260062) · [Merton](https://dspace.mit.edu/entities/publication/6e05a2fa-478f-41e6-9e69-a06532002236) |
| **1980s** | The expansion of bond trading increased the importance of measuring interest-rate, spread, and inventory risk. | Financial institutions began applying quantitative valuation and hedging more systematically. [Adrian](https://www.imf.org/en/publications/departmental-papers-policy-papers/issues/2018/08/01/risk-management-and-regulation-45664) |
| **1988** | Basel I introduced a common international capital framework focused primarily on credit risk. | Risk-weighted exposures became a basis for comparing capital adequacy across internationally active banks. [1988 Basel Accord](https://www.bis.org/publications/198807-standards-international-convergence-capital-measurement-and-capital-standards) |
| **Late 1980s and early 1990s** | The 1987 equity crash and the episodes involving Drexel Burnham Lambert and Salomon Brothers form part of the historical backdrop in Adrian’s account. | They raised questions about market exposures, liquidity under stress, and governance. [Adrian](https://cepr.org/publications/dp12422) |
| **1996–98** | The Basel Committee issued the Market Risk Amendment in January 1996. Its implementation target was the end of 1997, which accounts for the 1998 date in the original notes. | It added market-risk capital requirements and permitted approved internal models subject to conditions. [Basel Committee history](https://www.bis.org/committees/bcbs/history) |
| **2004–06** | Basel II introduced a more risk-sensitive capital framework. Its three pillars covered minimum capital, supervisory review, and market discipline through disclosure. | Model calibration and internal controls gained importance. Risk-sensitive requirements can reinforce a downturn if measured risk rises as lending capacity falls. [Basel Committee history](https://www.bis.org/committees/bcbs/history) · [Adrian](https://cepr.org/publications/dp12422) |
| **2007–09 and after** | The crisis exposed weaknesses in leverage, liquidity, risk measurement, governance, and incentives. Subsequent reforms expanded capital and liquidity requirements, stress testing, buffers, and resolution arrangements. | Risk is assessed across an institution’s balance sheet and under adverse scenarios, alongside the risks of individual positions. [Basel Committee history](https://www.bis.org/committees/bcbs/history) |

The revised Basel market-risk internal models framework uses **97.5% expected shortfall** for modellable risk factors, with prescribed liquidity horizons and stress calibration. This is a regulatory capital specification. An internal risk report still needs to define its own horizon, valuation basis, and intended use. [Basel Framework, MAR33](https://www.bis.org/committees/bcbs/basel-framework/standard/mar/33/inforce/2023-01-01/published/2020-06-05). [Bank for International Settlements](https://www.bis.org/committees/bcbs/history?utm_source=chatgpt.com)

**RiskMetrics** and **CreditMetrics** are distinct J.P. Morgan research references. RiskMetrics describes a framework for mapping positions to market factors and estimating market loss. CreditMetrics focuses on changes in credit value from rating migrations and defaults, with portfolio dependence affecting aggregate loss. [RiskMetrics technical document](https://www.msci.com/research-and-insights/paper/1996-riskmetrics-technical-document) · [CreditMetrics technical document](https://www.msci.com/research-and-insights/paper/creditmetrics-technical-document). [MSCI](https://www.msci.com/research-and-insights/paper/1996-riskmetrics-technical-document?utm_source=chatgpt.com)

The original note says “Bernstein.” Peter L. Bernstein’s *Against the Gods* is a plausible reading reference. The intended book remains an open bibliographic item. [Publisher’s entry](https://www.wiley.com/en-us/against-the-gods-the-remarkable-story-of-risk-p-9780471295631). [Wiley](https://www.wiley.com/en-us/against-the-gods-the-remarkable-story-of-risk-p-9780471295631?utm_source=chatgpt.com)

## 3. Valuation, risk measurement, and backtesting

### Start with the balance sheet and positions

Assets and liabilities respond to rates, spreads, inflation, exchange rates, and behaviour such as prepayment or early withdrawal. Equity is the residual balance-sheet interest. A firm can have a positive accounting equity position and still face funding pressure if cash obligations arrive before assets can be sold or financed.

| Risk | Practical question | Example measurement |
|---|---|---|
| **Market** | How does the value of the position change when market factors move? | Delta, DV01, spread sensitivity, scenario P&L, VaR, expected shortfall |
| **Credit** | How do migration, default, recovery, and changing exposure affect loss? | PD, LGD, EAD, rating-transition scenarios, concentration |
| **Market liquidity** | What price and size are available when exiting a position? | Spread, depth, market impact, days to liquidate |
| **Funding liquidity** | Can the portfolio meet margin calls and other cash obligations? | Cash forecasts, liquidity gaps, collateral and funding stress |
| **Operational and model** | How do data errors, system failures, bad assumptions, or miscalibration affect decisions? | Reconciliations, sensitivity tests, independent valuation, failure scenarios |

### Map factors to values

Write instrument value as \(V=f(t,X,\theta)\), where \(X\) contains market factors and \(\theta\) contains model parameters. The first-order change from a market move is approximately \(\nabla_X V^\top\Delta X\). A second-order approximation adds curvature and cross-factor terms. Full revaluation applies each scenario to the complete pricing function. This matters for options, barriers, callable instruments, and large stresses.

An exposure map should specify the curve or surface, tenor, currency, shock convention, and unit for each factor. For example, a desk can report DV01 as a positive amount lost on a one-basis-point yield rise, or report a signed P&L sensitivity. Record the convention in the output. To calculate aggregate risk, apply a common scenario to all positions and **sum the scenario P&L before computing portfolio quantiles**. [McNeil, Frey, and Embrechts, *Quantitative Risk Management*](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management). [Princeton University Press](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management?utm_source=chatgpt.com)

### VaR, expected shortfall, and capital

Let \(L\) denote a loss, with larger values representing worse outcomes. Value-at-Risk at confidence \(\alpha\) is the loss quantile

$$
\operatorname{VaR}_\alpha(L)=\inf\{l:\Pr(L\le l)\ge\alpha\}.
$$

Expected shortfall, commonly called CVaR in optimisation, averages the loss quantiles in the tail.

$$
\operatorname{ES}_\alpha(L)=
\frac{1}{1-\alpha}\int_\alpha^1\operatorname{VaR}_u(L)\,du.
$$

The quantile-average definition handles probability masses at the VaR threshold. For example, suppose losses are 0 with probability 94%, 10 with probability 4%, and 40 with probability 2%. The 95% VaR is 10. The worst 5% includes all the probability at 40 and three percentage points of the probability at 10, giving an ES of 22. [Rockafellar and Uryasev](https://sites.math.washington.edu/~rtr/papers/rtr179-CVaR1.pdf). [sites.math.washington.edu](https://sites.math.washington.edu/~rtr/papers/rtr179-CVaR1.pdf?utm_source=chatgpt.com)

A **coherent** risk measure satisfies monotonicity, cash translation, positive homogeneity, and subadditivity. Convex risk measures allow a broader treatment of how risk changes with scale. Expected shortfall is coherent for integrable losses. VaR can fail subadditivity for some joint loss distributions. A capital calculation adds a choice about what losses are covered and over what horizon. Regulatory capital, internal economic capital, and accounting provisions answer different questions. [Artzner, Delbaen, Eber, and Heath](https://onlinelibrary.wiley.com/doi/pdfdirect/10.1111/1467-9965.00068). [Wiley Online Library](https://onlinelibrary.wiley.com/doi/pdfdirect/10.1111/1467-9965.00068?utm_source=chatgpt.com)

### How to test the risk model

1. **Fix the forecast and realised P&L definitions.** State the horizon, valuation time, treatment of intraday trades, and how positions and market inputs are captured.
2. **Check exception frequency.** At a 99% VaR threshold, a well-calibrated continuous one-day loss model produces exceptions about 1% of the time. A short sample provides little evidence at that level.
3. **Check exception clustering.** A model can have roughly the right number of breaches while missing a sustained volatility regime. Examine sequences and regime-specific results as well as the aggregate rate.
4. **Examine the loss beyond VaR.** Compare tail severity and stress scenarios. Review whether misses come from volatility, dependence, a pricing approximation, or a change in exposures.
5. **Respect overlapping horizons.** If daily forecasts predict ten-day losses, adjacent observations share returns. Use evaluation and inference that account for the overlap. [Kupiec](https://fedinprint.org/item/fedgfe/34596/original) · [Christoffersen](https://ideas.repec.org/a/ier/iecrev/v39y1998i4p841-62.html). [Fed in Print](https://fedinprint.org/item/fedgfe/34596/original?utm_source=chatgpt.com)

**Stochastic control** extends this from measuring exposure to choosing actions as the state changes. A trading example is choosing how much to hedge at each step given inventory, volatility, transaction costs, and loss limits. An insurance example is choosing a reinsurance or capital policy given the insurer’s surplus and claims environment. The state, feasible actions, and objective must be specified before the mathematical control problem is meaningful.

## 4. Financial time series, extremes, and dependence

### Empirical properties

Financial returns often exhibit heavy tails, volatility clustering, changing dependence, and different behaviour across assets and sampling frequencies. Examine returns, absolute returns, and squared returns separately. A series can have weak linear dependence in returns and substantial persistence in their magnitudes. Longer-interval returns can change the apparent tail behaviour and reduce sample size. Overlapping aggregation introduces additional dependence. [Rama Cont, *Empirical Properties of Asset Returns*](https://doi.org/10.1080/713665670). [doi.org](https://doi.org/10.1080/713665670?utm_source=chatgpt.com)

| Model | What it describes | Diagnostic question |
|---|---|---|
| **ARMA** | Conditional mean through lagged values and lagged innovations. | Is there stable predictive structure in the mean after accounting for sampling and costs? |
| **ARCH** | Conditional variance driven by previous squared innovations. | How much does recent surprise magnitude explain subsequent variance? |
| **GARCH** | Conditional variance driven by previous squared innovations and previous conditional variance. | How persistent is volatility, and how quickly does a forecast revert after a shock? |
| **SARIMAX** | Seasonal ARIMA with external regressors. | Does the extra information improve forecasts using values available at the forecast timestamp? |
| **Multivariate volatility** | Time-varying covariance across several series. BEKK is one research lead in the reading list. | Does dynamic dependence improve portfolio risk forecasts over a simpler covariance benchmark? |

For GARCH(1,1), let \(\epsilon_t\) be the innovation from the mean model and \(h_t\) its conditional variance. A standard specification is

$$
h_t=\omega+\alpha\epsilon_{t-1}^2+\beta h_{t-1}.
$$

Under the usual unit-variance innovation assumptions, \(\alpha+\beta<1\) gives a finite unconditional variance in the standard specification. The innovation distribution, sampling frequency, and sample period affect tail forecasts. Useful comparisons include a rolling variance, an exponentially weighted estimator, and a GARCH model with heavy-tailed innovations. Evaluate the volatility forecast directly as well as through VaR exceptions or portfolio decisions. [Engle, ARCH](https://web.pdx.edu/~crkl/ec572/readings/Engle82.pdf) · [Bollerslev, GARCH](https://public.econ.duke.edu/~boller/Published_Papers/joe_86.pdf). [web.pdx.edu](https://web.pdx.edu/~crkl/ec572/readings/Engle82.pdf?utm_source=chatgpt.com)

### Extreme value theory

- **Block maxima.** Divide the observations into blocks and model the maximum loss in each block. The block size controls the trade-off between having enough maxima to estimate the model and approaching the asymptotic setting.
- **Peaks over threshold.** Choose a high threshold and model the amounts by which losses exceed it, commonly using a generalised Pareto model. Inspect parameter stability across nearby thresholds. Volatility clustering can produce bursts of exceedances, so account for dependence through the modelling or inference.
- **Extreme observations and jumps.** A large observed return can arise from a jump, high continuous volatility, or a data issue. A Brownian path is continuous, yet a finite-interval observation can still be extreme. A jump study needs a sampling scheme, a continuous-volatility benchmark, and controls for market microstructure. [McNeil, Frey, and Embrechts](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management). [Princeton University Press](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management?utm_source=chatgpt.com)

### Copulas and aggregate risk

A copula couples the marginal distributions of individual factors into a joint distribution. For two continuous factors, transform their values through their marginal CDFs into \(U\) and \(V\). Upper-tail dependence asks how \(\Pr(U>u\mid V>u)\) behaves as \(u\) approaches one. The lower-tail analogue examines joint adverse outcomes near zero. This separates questions about extreme co-movement from questions about ordinary linear correlation.

In a portfolio simulation, the marginal models determine how large individual shocks can be. The dependence model determines which shocks tend to arrive together. A useful comparison holds the marginal distributions fixed and varies the dependence specification, then measures the effect on portfolio VaR, ES, and stressed losses. [McNeil, Frey, and Embrechts](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management). [Princeton University Press](https://press.princeton.edu/books/hardcover/9780691166278/quantitative-risk-management?utm_source=chatgpt.com)

## 5. Portfolio construction, sensitivities, and entropy pooling

### Mean–variance, CAPM, and Black–Litterman

**Markowitz mean–variance optimisation** chooses weights \(w\) using expected returns \(\mu\) and covariance \(\Sigma\). For example, minimise \(w^\top\Sigma w\) subject to a target \(w^\top\mu\), a budget constraint, and position limits. Varying the return target traces the efficient frontier. The mathematical optimisation needs estimates of means and covariances. Joint normality is one setting in which those moments describe the whole joint return distribution, and quadratic utility is another route to a mean–variance objective.

The practical limitations in the original notes are important. Expected returns are hard to estimate. Historical covariances can shift. Volatility treats upside and downside deviations symmetrically. Concentrated, fragile weights can result when input errors are large relative to the differences between assets. Compare allocations against equal weight and constrained minimum variance, and examine turnover and risk across periods. [Markowitz, *Portfolio Selection*](https://doi.org/10.1111/j.1540-6261.1952.tb01525.x). [Wiley Online Library](https://doi.org/10.1111/j.1540-6261.1952.tb01525.x?utm_source=chatgpt.com)

**CAPM** is an equilibrium model relating an asset’s expected excess return to market beta. It offers a benchmark for systematic exposure and performance attribution. Empirical results depend on the market proxy, sample, and stability of beta. An estimated beta describes exposure to the selected market series. Turning that exposure into an expected return requires the model’s equilibrium assumptions. [Sharpe, *Capital Asset Prices*](https://doi.org/10.1111/j.1540-6261.1964.tb02865.x). [Wiley Online Library](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1540-6261.1964.tb02865.x?utm_source=chatgpt.com)

**Black–Litterman** combines a prior on expected returns, often backed out from a benchmark portfolio, with explicit views and uncertainty about them. A view can describe an absolute expected return or a relative return between assets. The posterior is easier to audit when the record includes the starting weights, implied prior returns, view portfolios, view magnitudes, confidence, and final change in weights. [Black and Litterman, *Global Portfolio Optimization*](https://rpc.cfainstitute.org/research/financial-analysts-journal/1992/faj-v48-n5-28). [rpc.cfainstitute.org](https://rpc.cfainstitute.org/research/financial-analysts-journal/1992/faj-v48-n5-28?utm_source=chatgpt.com)

### Model the drivers, then measure the sensitivities

The seminar notes point toward portfolio optimisation using sensitivities to common drivers such as inflation, rates, or market volatility. Alejandro Rodríguez Domínguez has public working material on neural-network sensitivities and common drivers. The original notes also mention Miralta Bank, “Versa,” and “projective spaces.” The intended tool and geometric formulation remain follow-up items. [Rodríguez Domínguez, working paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4234186) · [Causal portfolio optimisation preprint](https://arxiv.org/abs/2504.05743). [papers.ssrn.com](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4234186\&utm_source=chatgpt.com)

A practical experiment could model each asset’s value or return as a function of common drivers. Let \(J_{ij}=\partial V_i/\partial X_j\) be the local sensitivity of asset \(i\) to driver \(j\). With factor covariance \(\Sigma_X\), the local factor-driven covariance of asset value changes is approximately \(J\Sigma_XJ^\top\). Add a residual covariance model where appropriate. For a vector \(n\) of position units, \(J^\top n\) gives the portfolio’s local driver exposures.

This creates distinct research questions.

- Do nonlinear sensitivities improve held-out P&L or exposure forecasts compared with a linear factor model?
- Do sensitivities and factor volatility change enough across regimes to justify updating allocations?
- Does a simulated factor shock match the intended economic scenario once correlated drivers and residual risks are included?
- What identification assumptions support interpreting a sensitivity as a **causal** effect? Differentiating a fitted predictor gives its local model response. A causal interpretation requires additional assumptions about how the input changes.

### Entropy pooling

Attilio Meucci’s entropy pooling starts with joint scenarios \(x_i\) and prior probabilities \(p_i>0\). It chooses posterior probabilities \(q_i\) that satisfy the views while minimising the relative entropy from the prior.

$$
\min_q \sum_i q_i\log(q_i/p_i)
\qquad
\text{subject to } q_i\ge 0,\quad \sum_i q_i=1
$$

Additional constraints express the views. For example, a stress-event probability can be written as a sum of the posterior weights of the stress scenarios. Views can also concern weighted averages, ranges, or relative orderings. The original notes mention Anton Vorobets as a later explainer and raise the use of fully flexible Monte Carlo scenarios. [Meucci, *Fully Flexible Views*](https://arxiv.org/abs/1012.2848) · [Vorobets, *Entropy Pooling Fundamentals*](https://antonvorobets.substack.com/p/entropy-pooling-fundamentals). [arxiv.org](https://arxiv.org/abs/1012.2848?utm_source=chatgpt.com)

**Illustration.** Suppose the prior probabilities of growth, stagnation, and stress are 50%, 35%, and 15%. Impose a 30% posterior probability of stress. Relative-entropy minimisation preserves the growth-to-stagnation probability ratio within the remaining 70%, giving approximately **41.18%, 28.82%, and 30.00%**. The same idea works on a large set of historical or simulated joint scenarios.

The prior deserves its own validation. Examine the marginal tails, dependence, and support for the events named in the views. Reweighting can raise the probability of a scenario already in the set. A missing event requires new scenarios or a revised prior. Generative AI can suggest stress narratives to translate into explicit shocks. The assigned probabilities and dynamics need empirical or economic justification. Report weight concentration and compare results under alternative priors.

### Why CVaR can be easier to optimise than VaR

For losses \(L_i(w)\) with scenario probabilities \(q_i\), the Rockafellar–Uryasev representation is

$$
\min_{w,z}\left[
z+\frac{1}{1-\alpha}\sum_i q_i\max\{L_i(w)-z,0\}
\right].
$$

Here \(z\) is a candidate tail threshold. Introducing one nonnegative slack per scenario yields a **linear programme** when scenario losses are affine in holdings and the portfolio constraints are linear. Cardinality constraints, some impact models, and dynamic trading decisions can change the optimisation problem. The original note’s “Ursayev” refers to **Stanislav Uryasev**. [Rockafellar and Uryasev, *Optimization of Conditional Value-at-Risk*](https://sites.math.washington.edu/~rtr/papers/rtr179-CVaR1.pdf). [sites.math.washington.edu](https://sites.math.washington.edu/~rtr/papers/rtr179-CVaR1.pdf?utm_source=chatgpt.com)

## 6. Credit risk and bank modelling

Credit involves a **creditor** extending funds or another contractual benefit to an **obligor**, who undertakes to repay or perform. Creditworthiness concerns the obligor’s capacity and willingness to meet those commitments. A credit rating is one representation of that assessment.

| Component | Meaning |
|---|---|
| **Credit migration** | An improvement or deterioration in assessed credit quality. A downgrade can change the value of debt before a missed payment occurs. |
| **Spread risk** | Changes in compensation over a benchmark curve. Credit spreads can reflect expected loss, risk premia, liquidity, and instrument characteristics. |
| **Default risk** | Failure to meet obligations under the applicable contractual or modelling definition. Exposure and recovery determine the resulting loss. |
| **Reference and rate risk** | A floating-rate loan can reprice after a benchmark rate change. That rate move can affect both the contract’s cash flows and the borrower’s ability to pay. Model those channels separately. |

### PD, LGD, and EAD

- **Probability of default (PD)** is the probability of default over a defined horizon for a specified obligor or exposure.
- **Loss given default (LGD)** is the fraction of the defaulted exposure lost, under the chosen recovery and discounting conventions.
- **Exposure at default (EAD)** is the amount exposed when default occurs. Revolving credit can change before default through additional drawdowns or repayments.

A simple expected loss calculation uses \(\mathrm{PD}\times\mathrm{LGD}\times\mathrm{EAD}\). With a 2% PD, a fixed EAD of £100,000, and a 45% LGD, expected loss is £900 over the specified horizon. When EAD and LGD are stochastic and dependent, the underlying calculation is \(\mathbb E[\mathbf 1_{\mathrm{default}}\mathrm{EAD}\,\mathrm{LGD}]\). Align the horizon, population, conditioning, and definitions when comparing estimates. [Basel Framework, treatment of expected losses](https://www.bis.org/committees/bcbs/basel-framework/standard/cre/35/inforce/2023-01-01/published/2020-03-27). [Bank for International Settlements](https://www.bis.org/committees/bcbs/basel-framework/standard/cre?allChapters=true\&chapter=50\&utm_source=chatgpt.com)

### Bonds, credit derivatives, and counterparty exposure

A corporate bond is a contractual claim on an issuer’s cash flows. A defaultable zero-coupon bond with unit face value and zero recovery pays one at maturity only if the issuer survives. Under a risk-neutral pricing model its value is the discounted expectation of that survival-contingent payoff. A coupon bond adds scheduled cash flows and a recovery convention.

A **credit default swap** exchanges a premium leg for protection on specified credit events affecting a reference entity. The protection seller and buyer also create counterparty exposures to each other. Netting, collateral, and periodic marking change those exposures. A unilateral valuation considers the default risk of one party. A bilateral valuation considers both parties. A **vulnerable claim** is a claim whose payoff or value is affected by the party responsible for paying it. A useful stress scenario allows the counterparty’s credit quality to deteriorate at the same time as exposure increases. [Basel Framework, counterparty credit risk](https://www.bis.org/committees/bcbs/basel-framework/standard/cre?allChapters=true&chapter=50). [Bank for International Settlements](https://www.bis.org/committees/bcbs/basel-framework/standard/cre?allChapters=true\&chapter=50\&utm_source=chatgpt.com)

### Scorecards, stress testing, and provisioning

| Topic | Role | Research question |
|---|---|---|
| **Scorecards** | Turn borrower or account attributes into a risk ranking, often followed by probability calibration. | Does the model separate risk well, and are predicted default rates calibrated across vintages, segments, and economic conditions? |
| **Dual time dynamics (DTD)** | A plausible interpretation of the fragment is credit modelling with both account age and calendar conditions. Origination vintage gives another way to organise the data. | How can seasoning, underwriting changes, and the macro environment be separated when calendar time equals vintage plus age? [Breeden](https://ideas.repec.org/a/eee/csdana/v51y2007i9p4761-4785.html) |
| **SARIMAX** | One possible time-series model for a segment-level loss or delinquency rate with macro predictors. | Do the drivers add stable forecasting value, and are their historical releases aligned with each forecast? |
| **CCAR** | U.S. Federal Reserve capital planning and supervisory assessment context, related to bank stress testing. | How do capital and losses evolve under a prescribed adverse scenario? [Federal Reserve stress tests and capital planning](https://www.federalreserve.gov/supervisionreg/stress-tests-capital-planning.htm) |
| **CECL** | U.S. GAAP approach to estimating expected credit losses over the relevant contractual term for covered assets. | How should historical experience, current conditions, and reasonable and supportable forecasts inform the allowance? [FASB Topic 326](https://storage.fasb.org/ASU_2016-13.pdf) · [Federal Reserve CECL FAQ](https://www.federalreserve.gov/supervisionreg/topics/faq-new-accounting-standards-on-financial-instruments-credit-losses.htm) |

A scorecard’s discrimination, an accounting allowance, and capital under stress are separate outputs. A model can rank borrowers well while producing systematically biased probabilities. A stress projection answers a conditional scenario question, while an allowance uses its applicable measurement basis and forecast assumptions. [ideas.repec.org](https://ideas.repec.org/a/eee/csdana/v51y2007i9p4761-4785.html?utm_source=chatgpt.com)

## 7. Trading experiments from the sketch

### Hypothesis A — Does the sequence of recent daily directions predict the next day?

The left-hand table lists every one-, two-, and three-day binary direction history. Read each state from **oldest to newest**. `UUD`, for example, means two up days followed by one down day. The proposed output is the probability of an up or down day **after** that state.

| History length | States |
|---|---|
| 1 day | `U`, `D` |
| 2 days | `UU`, `UD`, `DU`, `DD` |
| 3 days | `UUU`, `UUD`, `UDU`, `UDD`, `DUU`, `DUD`, `DDU`, `DDD` |

**Proposed specification.** Define `U` as a strictly positive close-to-close return and `D` as a strictly negative one. Keep flat sessions as `F` and break histories across missing sessions. The original two-column table can show probabilities conditional on a non-flat next day. Report the flat outcome rate and total sample size alongside them.

For a state \(c\), the basic binary estimate is

$$
\widehat p(U\mid c,\text{non-flat})=
\frac{n(c,U)}{n(c,U)+n(c,D)}.
$$

Fit the counts on historical training data. With sparse states, a smoothed estimate can shrink the probability toward the unconditional or shorter-history rate. Retain raw counts so a striking probability can be distinguished from a well-populated state.

**Evaluation.** Compare the unconditional forecast, the previous-day-only model, and the two- and three-day models in chronological folds. Use log loss, Brier score, calibration plots, and confidence intervals for each state. The economic test needs a separate return target based on the first executable price after the signal. A sign forecast can improve while net P&L deteriorates if the winning moves are small, the losing moves are large, or execution absorbs the difference.

### Hypothesis B — What happens after a new yearly high or low?

The right-hand table asks for the chance of **up** after a yearly low and **down** after a yearly high, measured over **7 days** and **1 month**.

| Signal at decision time | 7 calendar days later | 1 calendar month later |
|---|---|---|
| New trailing-year low | Estimate probability of a positive forward return | Estimate probability of a positive forward return |
| New trailing-year high | Estimate probability of a negative forward return | Estimate probability of a negative forward return |

**Proposed specification.** For an initial crypto study, use a trailing 365-calendar-day window of completed daily closes, excluding today’s close from the comparison set. Trigger a new low if today closes below every eligible prior close, and a new high if it closes above every eligible prior close. An equities variant can use 252 previous trading sessions. A calendar-year-to-date version is a separate test using data available up to the signal date.

Measure the terminal close-to-close return to the first eligible close on or after the horizon date. For the month horizon, use a calendar-month increment with an explicit month-end convention. Test the original reversal hypothesis alongside the possibility of continuation. Report the full return distribution, counts, and conditional probabilities at both horizons.

**Controls for both studies.**

1. Freeze the symbol universe, event definitions, thresholds, and horizons before evaluation.
2. Compare with unconditional probabilities and controls matched on prior trend, volatility, and, where relevant, calendar effects.
3. Track repeated highs or lows during the same trend. Use date-aware inference for overlapping returns and correlated events across assets.
4. At a train/test boundary, exclude training observations whose outcomes mature in the evaluation period.
5. Report predictive skill and net trading performance separately. A high hit rate alone leaves return magnitudes and costs unmeasured.

## 8. Earlier project proposal — An actuarial R package

### Scope and supervision

The original proposal was to build an R library for **general insurance or life insurance**. General insurance gives the clearest first scope for the listed functions, which centre on claim arrivals, severity, aggregate losses, premiums, and insurer surplus. Life insurance would be a distinct project around mortality, survival, contingent cash flows, and assumptions such as lapses.

A supervisor can guide the subject, challenge the mathematical choices, and discuss the design as it develops. Confirm the programme’s assessment arrangements separately, including the supervisor’s role in examination. An industry project could start from a real modelling question and use an agreed dataset, with synthetic or otherwise shareable examples for the public repository.

### Proposed package functions

| Component | Deliverable | Verification |
|---|---|---|
| **1. Data reading** | Import policies, exposures, claim counts, claim amounts, and dates using a documented schema. | Check units, duplicates, claim-to-policy joins, missing values, and zero-claim exposures. |
| **2. Exploratory analysis** | Exposure totals, claim frequency, severity, loss ratios where premiums exist, and segment summaries. | Reconcile counts, exposure, and total payments against the source or a small independent calculation. |
| **3. Stochastic simulation** | Simulate Poisson arrivals, general renewal arrivals, and a Markov or Markov-modulated extension. | Compare simulated interarrival times, count distributions, and state transitions with model targets. |
| **4. Aggregate claims** | Simulate a total claim amount process from claim arrivals and severities. | Check simulated means, variances, and selected quantiles against analytic or independent references. |
| **5. Premium calculations** | Calculate expected claims and apply an explicit safety loading. | Show how the premium changes under frequency, severity, and exposure assumptions. |
| **6. Surplus and ruin** | Simulate insurer surplus and estimate finite-horizon ruin probability. | Evaluate the process at claim arrivals, report Monte Carlo uncertainty, and compare against a benchmark where available. |

### Frequency, severity, and surplus

Let \(N_t\) be the claim count up to time \(t\) and \(X_i\) the claim severities. The aggregate claim process is \(S_t=\sum_{i=1}^{N_t}X_i\). With independent identically distributed severities independent of the claim count,

$$
\mathbb E[S_t]=\mathbb E[N_t]\mathbb E[X],
\qquad
\operatorname{Var}(S_t)=
\mathbb E[N_t]\operatorname{Var}(X)+
\operatorname{Var}(N_t)\mathbb E[X]^2.
$$

For Poisson arrivals at rate \(\lambda\), these reduce to \(\lambda t\mathbb E[X]\) and \(\lambda t\mathbb E[X^2]\). A renewal model allows a different interarrival distribution. A Markov-modulated model can vary claim intensity by state. Fit and verify the Poisson baseline first so the incremental value of each extension can be assessed. [Actuar risk and ruin vignette](https://stat.ethz.ch/CRAN/web/packages/actuar/vignettes/risk.pdf). [stat.ethz.ch](https://stat.ethz.ch/CRAN/web/packages/actuar/vignettes/risk.pdf?utm_source=chatgpt.com)

In the basic surplus model, \(U_t=u+ct-S_t\), with initial capital \(u\) and premium income rate \(c\). Under an expected-value premium principle, a safety loading \(\eta\) gives \(c=(1+\eta)\lambda\mathbb E[X]\). For example, two expected claims per year with mean severity 1,000 give expected annual claims of 2,000. A 20% safety loading sets annual premium income to 2,400 in this model. Estimate finite-horizon ruin as the proportion of simulated paths on which surplus falls below zero. The estimate depends on initial capital, claim tails, premium timing, and the chosen horizon.

### Rating and implementation

A frequency GLM can model a policy’s expected count as \(\mu_i=e_i\exp(x_i^\top\beta)\), where \(e_i\) is exposure. The log exposure enters as an offset. For a Poisson model the deviance contribution is \(2[y_i\log(y_i/\mu_i)-(y_i-\mu_i)]\), with the first term set to zero when \(y_i=0\). Minimising Poisson deviance for a fixed dataset corresponds to maximising Poisson likelihood. A tree model is a possible nonlinear benchmark. Use a count or rate objective that handles exposure explicitly. Apply monotonic constraints only where the proposed direction has a domain justification and test the effect on held-out calibration.

R can serve as the primary implementation, with SAS or Excel for independent reconciliation where useful. Python can reproduce selected examples or connect to the package through an interface. C++ is an option for a measured simulation bottleneck. Docker and a small R API are deployment extensions once the calculation and data contracts are stable. A GARCH-based economic scenario generator is another extension if investment returns or financial conditions become part of the surplus model. [Posit Plumber documentation](https://opensource.posit.co/software/plumber/). [opensource.posit.co](https://opensource.posit.co/software/plumber/?utm_source=chatgpt.com)

### R ecosystem and reference material

| Package or resource | Role in the proposal |
|---|---|
| `insuranceData` | Example datasets for non-life claim frequency and severity. [CRAN](https://cran.r-project.org/package=insuranceData) |
| `insurancerating` | Analytical techniques and examples for insurance rating. [Package documentation](https://www.stats.bris.ac.uk/R/web/packages/insurancerating/vignettes/getting-started.html) |
| `ChainLadder` | Reserving methods and diagnostics, including Mack, Munich, bootstrap, and multivariate chain-ladder methods, with additional curve-fitting and GLM material. [Project documentation](https://mages.github.io/ChainLadder/) |
| `cplm` | Tweedie compound Poisson models, including Monte Carlo EM and Bayesian methods. [CRAN reference manual](https://search.r-project.org/CRAN/refmans/cplm/html/cplm-package.html) |
| `lossDev` | Historical pointer to the Laws and Schmid Bayesian loss development work, including skewed heavy-tailed errors and structural changes in development. [Research paper](https://www.casact.org/sites/default/files/presentation/annual_2009_handouts_c1-laws_schmid.pdf) |
| `actuar` | Loss distributions, compound simulation, credibility, and risk and ruin calculations. A benchmark for checking a prototype. [CRAN reference manual](https://search.r-project.org/CRAN/refmans/actuar/html/actuar-package.html) |
| `favir` | Historical lead for *Formatted Actuarial Vignettes in R*, described in the original notes as consistently formatted Sweave examples. |
| `mondate` | Date arithmetic expressed in months. Useful for development periods and reporting cohorts. [CRAN](https://cran.r-project.org/package=mondate) |
| `lifecontingencies` | Actuarial calculations for life-contingent cash flows. [CRAN](https://cran.r-project.org/package=lifecontingencies) |
| `LifeInsuranceContracts` | Framework for traditional life insurance contracts. [CRAN](https://cran.r-project.org/package=LifeInsuranceContracts) |

The original resource list includes Nigel de Silva’s [*R Examples for Actuaries*](https://toolkit.pbworks.com/f/R%20Examples%20for%20Actuaries%20v0.1-1.pdf), the [interactive introduction to R for actuaries](https://www.casact.org/sites/default/files/presentation/annual_2009_handouts_c11-driscoll.pdf), and the GIRO material on [*An Actuarial Toolkit*](https://mages.gitlab.io/publication/actuarial-toolkit/). R-SIG-insurance is a historical lead. The original mailing-list URL returned a 404 during verification. 

## 9. Derivative pricing and the Heston bachelor project

My bachelor project studied **option pricing with stochastic volatility**. I worked with volatility-surface fitting and built a pricing engine using the Heston model. The original working notes considered AAPL options, the challenge of collecting option-chain data, and the problem of fitting short expiries and extreme strikes.

### Instruments and exercise rules

| Instrument | Contract feature | Main modelling consideration |
|---|---|---|
| **European call or put** | The holder has the right to buy or sell at strike \(K\), respectively, with exercise at expiry. | Discounted payoff under a pricing measure. The vanilla terminal payoffs are \((S_T-K)^+\) and \((K-S_T)^+\). |
| **American option** | Exercise is allowed on specified days up to and including expiry, subject to contract terms. | Compare immediate exercise with continuation value. Dividends and rates can affect the exercise boundary. |
| **Bermudan option** | Exercise is allowed on a specified set of dates. | Value continuation at each eligible date, using backward induction or simulation. |
| **Asian option** | Payoff depends on an average over the underlying price path. | Observation schedule and averaging convention affect valuation and hedging. |
| **Barrier option** | A price level can activate or terminate the claim. | Monitoring frequency, jumps, and path discretisation affect barrier crossing. |
| **Forward** | An agreement today to transact at a specified future date and price. | Financing, income, and carry determine the forward relationship. |
| **Interest-rate swap** | Exchanges cash flow streams, commonly fixed versus floating. | Discounting and forward projection depend on curves and cash flow conventions. |
| **Swaption** | Option to enter a swap. A payer swaption gives the right to pay fixed and receive floating. | Forward swap rate, annuity, expiry, tenor, and swaption volatility. |
| **Bond futures** | Futures on a bond or eligible deliverable basket. | Delivery rules, conversion factors, basis, and the cheapest-to-deliver bond where applicable. |

An **option chain** is a set of quotes and contract details across strikes and expiries. For research, bid and ask describe the quoted spread at a timestamp. Size describes displayed liquidity at those quotes. Last trade, volume, and open interest provide different information. A directional prediction from chain features is an empirical hypothesis that requires its own test. [OCC equity options specifications](https://www.theocc.com/clearance-and-settlement/clearing/equity-options-product-specifications) · [Cboe quote data fields](https://datashop.cboe.com/option-quote-intervals). [Equity Options](https://www.theocc.com/clearance-and-settlement/clearing/equity-options-product-specifications?utm_source=chatgpt.com)

### Pricing measures and interest rates

The **physical measure \(\mathbb P\)** describes observed or forecasted outcomes. The **risk-neutral measure \(\mathbb Q\)** is a pricing measure used with the appropriate numeraire. Calibrating Heston to option quotes estimates pricing dynamics under \(\mathbb Q\). Forecasting realised volatility or running a trading backtest requires data and assumptions under \(\mathbb P\). Rates affect discounting, forward prices, and, for American contracts, exercise. Specify what stays fixed when calculating the effect of a rate change. [Tomas Björk, *Arbitrage Theory in Continuous Time*](https://academic.oup.com/book/32596). [Oxford Academic](https://academic.oup.com/book/32596?utm_source=chatgpt.com)

The **Vasicek** short-rate model uses a mean-reverting Gaussian process. The **Cox–Ingersoll–Ross** model uses mean reversion and a diffusion term proportional to the square root of the rate. Their standard forms are

$$
dr_t=\kappa(\theta-r_t)\,dt+\sigma\,dW_t,
\qquad
dr_t=\kappa(\theta-r_t)\,dt+\sigma\sqrt{r_t}\,dW_t,
$$

respectively. Vasicek permits negative rates. The standard CIR process with admissible parameters is nonnegative. Compare them by how well they represent the rate environment, reproduce the term structure for the intended task, and behave under shocks. [Vasicek](https://doi.org/10.1016/0304-405X%2877%2990016-2) · [Cox, Ingersoll, and Ross](https://ideas.repec.org/a/ecm/emetrp/v53y1985i2p385-407.html). [ScienceDirect](https://doi.org/10.1016/0304-405X%2877%2990016-2?utm_source=chatgpt.com)

### Black–Scholes–Merton as a quoting convention

Use \(F(T)\) for the forward price, \(D(T)\) for the discount factor, \(K\) for the strike, \(T\) for time to expiry in years, and \(\sigma\) for annualised volatility. The European call formula in forward form is

$$
C=D(T)\big[F(T)\Phi(d_1)-K\Phi(d_2)\big],
\qquad
d_1=\frac{\log(F(T)/K)+\tfrac12\sigma^2 T}{\sigma\sqrt T},
\qquad
d_2=d_1-\sigma\sqrt T.
$$

Put–call parity for matching European contracts is \(C-P=D(T)(F(T)-K)\). Implied volatility inverts the pricing formula for a given market price, forward, discount factor, strike, and expiry. Check price bounds before inversion. Use a bracketed solver and verify that repricing at the returned volatility recovers the input price. In the wings or close to expiry, low vega can turn a small price error into a large IV error.

The constant-volatility assumption makes Black–Scholes–Merton a useful common scale. The market surface is formed by the different volatilities required to explain different quotes on that scale. Greeks describe local sensitivities. Higher-order Greeks such as vanna and volga help analyse combined spot and volatility moves and nonlinear volatility exposure. Their value for hedging depends on the surface dynamics and the convention used to bump it. [Black and Scholes](https://doi.org/10.1086/260062) · [Merton](https://dspace.mit.edu/entities/publication/6e05a2fa-478f-41e6-9e69-a06532002236) · [Vollib](https://vollib.org/documentation.html). [doi.org](https://doi.org/10.1086/260062?utm_source=chatgpt.com)

### Smile, surface, and surface dynamics

- **Moneyness** compares strike with spot or forward. A useful coordinate is log-forward moneyness \(k=\log(K/F(T))\). State the sign convention.
- **Smile or skew** describes how implied volatility varies across moneyness at a fixed expiry. A **surface** extends this across maturities. Total implied variance \(w(k,T)=\sigma_{\mathrm{imp}}^2(k,T)T\) is a common fitting coordinate.
- **Sticky strike** holds the quoted volatility at a fixed strike when spot moves, for a given maturity and scenario convention. **Sticky delta** holds the quote at a fixed option delta, so the corresponding strike can move. These are distinct scenarios for how a smile responds to spot. Compare them against observed changes when assessing hedges.
- **Static arbitrage checks.** At a fixed expiry, European call prices should be decreasing and convex in strike and obey their price bounds. Calendar checks need consistent forward, carry, and maturity conventions. An SVI fit with suitable constraints provides a benchmark for surface interpolation. [Gatheral and Jacquier, *Arbitrage-free SVI Volatility Surfaces*](https://arxiv.org/abs/1204.0646) · [Derman, implied volatility dynamics](https://www.math.columbia.edu/~smirnov/Derman.pdf). [arxiv.org](https://arxiv.org/abs/1204.0646?utm_source=chatgpt.com)

### Heston dynamics and calibration

With deterministic rates and dividend yield in the simple equity specification, the risk-neutral dynamics are

$$
\frac{dS_t}{S_t}=(r-q)\,dt+\sqrt{v_t}\,dW_t^S,
\qquad
dv_t=\kappa(\theta-v_t)\,dt+\xi\sqrt{v_t}\,dW_t^v,
\qquad
d\langle W^S,W^v\rangle_t=\rho\,dt.
$$

The five parameters are initial variance \(v_0\), mean reversion speed \(\kappa\), long-run variance \(\theta\), volatility of variance \(\xi\), and spot–variance shock correlation \(\rho\). The European vanilla pricer uses the characteristic function and numerical integration. Short maturities, extreme strikes, and parameter combinations near numerical boundaries are useful test cases. FFT is a possible implementation route when the pricing workload benefits from evaluating many strikes together. [Heston, 1993](https://academic.oup.com/rfs/article-abstract/6/2/327/1574747) · [Cui, del Baño Rollin, and Germano, Heston calibration](https://arxiv.org/abs/1511.08718). [Oxford Academic](https://academic.oup.com/rfs/article-abstract/6/2/327/1574747?utm_source=chatgpt.com)

The original notes proposed minimising squared differences between calculated prices and market prices, while making IV the primary diagnostic. These choices can be made explicit.

| Objective | What it emphasises | What to check |
|---|---|---|
| **Squared price error** | Fit in currency or premium points. | The most expensive contracts can dominate the objective. Break out errors by expiry and moneyness. |
| **Spread-scaled price error** | Fit relative to the quoted spread, using a floor on the denominator. | Results can be sensitive to stale or very tight quotes. Report the fraction of model prices within bid and ask. |
| **Implied-volatility error** | Fit in annualised volatility units. | Very low-vega quotes can produce unstable IVs. Report quote exclusions and inspect the wings separately. |
| **Robust loss** | Reduces the influence of unusually large residuals. | Inspect where the downweighted quotes occur. A systematic miss at short expiries is useful model evidence. |

Calibrate with several starting values. Compare objective values, fitted parameters, and held-out errors. If different parameter vectors produce similar prices, the surface may identify combinations of parameters better than each parameter individually. A daily parameter series is itself an object to inspect for instability.

### AAPL and SPX are different research tracks

**SPX** gives a cash-settled European-style contract, which aligns with the standard Heston vanilla pricer. Its series have different expiry and settlement conventions, so use the exact contract metadata. **AAPL** preserves the original single-stock idea. Standard U.S. equity options have American exercise and share delivery. For AAPL, discrete dividends, early exercise, and corporate-action adjustments belong in the contract and pricing treatment. A lattice, finite-difference model, or suitable approximation can benchmark the early-exercise premium. Barone–Adesi–Whaley and Longstaff–Schwartz are two distinct references for American valuation. [Cboe SPX](https://www.cboe.com/tradable-products/sp-500/spx-options) · [OCC equity options](https://www.theocc.com/clearance-and-settlement/clearing/equity-options-product-specifications) · [Barone–Adesi and Whaley](https://doi.org/10.1111/J.1540-6261.1987.TB02569.X) · [Longstaff and Schwartz](https://doi.org/10.1093/RFS/14.1.113). [Cboe](https://www.cboe.com/tradable-products/sp-500/spx-options?utm_source=chatgpt.com)

## 10. Rebuilding the thesis as a trading research project

### Milestone 1 — Reproduce one surface end to end

1. **Recover the thesis logic.** Identify the pricing convention, parameterisation, optimiser, calibration objective, and data transformations used in the original work. Build a synthetic case where you know the inputs.
2. **Get one timestamped options chain.** Capture contract identifier, expiry and settlement, strike, call or put, multiplier, bid, ask, quote size and timestamp, and aligned inputs for the forward and discount curve. The old notes mention departmental Bloomberg access as a possible data lead. Cboe also documents historical quote intervals with NBBO and size. Check availability and redistribution terms before designing the public dataset. [Cboe Option Quote Intervals](https://datashop.cboe.com/option-quote-intervals). [datashop.cboe.com](https://datashop.cboe.com/option-quote-intervals?utm_source=chatgpt.com)
3. **Build the narrow calculation path.** Normalise quotes, estimate the forward, calculate moneyness, invert bid, mid, and ask IV where feasible, and build the surface. Preserve quote rejection reasons so the wings and short maturities remain visible in diagnostics.
4. **Fit three benchmarks.** Use a simple flat-volatility baseline, a flexible surface benchmark such as constrained SVI, and Heston. Plot residuals by expiry and moneyness. Report price error, IV error, bid/ask coverage, and parameter stability.
5. **Check numerical behaviour.** Validate price bounds, parity, strike monotonicity and convexity, IV round trips, and agreement with an independent pricer on a small test grid.

### Milestone 2 — Separate interpolation from forecasting

A withheld-strike test asks whether the model can infer prices of contracts excluded from the fit **at the same timestamp**. A withheld-date test asks whether a rule built using past information forecasts future market prices or volatility. Both matter. A model with good interpolation can still be a weak predictor, and a model with modest interpolation error can still extract a useful dynamic feature.

Use a chronological replay with a fixed quote timestamp and decision lag. Retain the same contract identifiers between the signal and exit. Account for the passage of time, changes in the forward, and differences in expiry and moneyness when interpreting the markout. Compare Heston with the simpler surface under the same quote filters and splits.

### Milestone 3 — Test one residual signal

**Proposed first experiment.** On each eligible SPX snapshot, fit the surface using a predefined reference set of liquid quotes. Score the quotes excluded from that fit. Define a residual as market IV minus fitted IV for the same contract. Ask whether unusually positive or negative residuals predict **the next session’s market repricing**. Compare against a no-change forecast and the same signal derived from the simpler surface.

The replay should answer three questions in order.

1. **Does the signal predict a move in the market quote?** A residual can shrink because the fitted surface moves toward the contract. Record the change in the actual market quote separately.
2. **Could the move be captured at executable prices?** For a long, use the first eligible ask after the decision and a bid at exit. Reverse those sides for a short. Apply the contract multiplier, fees, size limits, and a slippage or fill assumption consistent with the data. Report how much of the apparent opportunity is contained within the spread.
3. **What exposures explain the P&L?** Compare raw and delta-hedged markouts using a specified hedge instrument and schedule. Track hedge costs, gamma, vega, theta, financing or margin assumptions, and stress losses. A delta hedge leaves exposure to changes in volatility, jumps, and other factors.

Set the entry threshold, quote-quality filters, horizon, and Greek limits on development data. Show trade counts, net P&L distribution, sensitivity to higher costs, and results across days and expiry buckets. A residual may reflect liquidity, quote staleness, tail-risk compensation, or a stable misspecification in Heston. The case for further trading research is **incremental held-out performance at executable quotes** compared with the simpler benchmark.

This rebuild also works as a portfolio project on its own. A public version can use synthetic chains to demonstrate the engine, calibration, and replay. A documented input schema allows licensed historical quotes to be used locally when available.

## 11. Reading catalogue

These **61 titles and topic leads** came from the original notes. Capitalisation and obvious spelling were cleaned up. The questions are prompts for reading or replication. Author, venue, year, and findings still need checking for entries without a linked reference elsewhere in the document.

### Derivatives, hedging, and market models

| Title or topic lead | Question to investigate |
|---|---|
| Bermudan option valuation under state-dependent models | How do state-dependent dynamics change continuation values and exercise decisions? |
| Option-implied objective measures of market risk with leverage | What assumptions connect option-implied distributions to physical risk estimates when leverage matters? |
| The sustainable Black–Scholes equations | How is sustainability defined, and where does it enter the valuation model? |
| A robustness analysis of least-squares Monte Carlo for R&D real options valuation | How sensitive are exercise values to basis functions, path counts, and exercise timing? |
| On geometric Brownian motion with alternating trend | How are trend switches specified, and can they be identified from noisy prices? |
| Trajectory-based market models. Arbitrage and pricing intervals | What path restrictions determine admissible prices and hedges? |
| A statistical test for the Heston model | Which aspects of the model does the test identify, and how does it behave under alternatives? |
| Threshold random walk structures in finance | Can threshold-dependent dynamics improve prediction over a simple random-walk benchmark? |
| Risk-adjusted dynamic hedging strategies | Which risk measure and cost assumptions change the hedge relative to delta replication? |
| The effects of curvature and elevation of the probability weighting function on options prices | Which prices or smile features respond most to changes in probability weighting? |
| Pricing and hedging basket options under shifted asymmetric jump-diffusion processes | How do asymmetric jumps and cross-asset dependence affect price and hedge error? |
| Black–Scholes option sensitivity using high-order Greeks | When do higher-order terms materially improve P&L attribution or scenario approximation? |

### Market risk, time series, and portfolios

| Title or topic lead | Question to investigate |
|---|---|
| Evaluation of volatility forecasts in a VaR framework | Can volatility models be ranked consistently by forecast error and VaR performance? |
| Maximum empirical likelihood inference for outliers in autoregressive time series | How does the procedure distinguish outliers from persistent dynamics? |
| The role of fund size and returns to scale in the performance of mutual funds | How are fund capacity, flows, strategy, and performance separated? |
| Evaluating correlations in European government bond spreads | How do benchmark choice, common factors, and crisis periods change estimated dependence? |
| Measuring the impact of behavioural choices on market prices | What variation identifies a behavioural effect on prices? |
| A note on natural risk statistics, OWA operators, and generalised Gini functions | How do different rank-weighting schemes encode preferences over loss distributions? |
| Direct multi-step estimation and time series classification | When does direct horizon-specific fitting outperform iterated forecasts or simpler classifiers? |
| Combinatorial nonlinear goal programming for ESG portfolio optimisation and dynamic hedge management | How do discrete constraints and competing objectives change feasible portfolios? |
| RedES, a risk measure in a Pareto–Lévy stable framework with clustering | How is the measure defined, and how sensitive is it to tail and clustering assumptions? |
| Monetary risk functionals on Orlicz spaces produced by set-valued risk maps and random measures | What integrability and aggregation conditions permit a well-defined monetary risk measure? |
| A probability inequality related to Mardia’s kurtosis | What assumptions yield the inequality, and what can it diagnose about multivariate data? |
| Risk measurement using the mixed tempered stable distribution | Does a flexible tail distribution improve held-out risk forecasts enough to justify estimation complexity? |
| BEKK element-by-element estimation of a volatility matrix. A portfolio simulation | How do the estimates behave in high dimensions, and how much do they improve portfolio forecasts? |
| On a data mining framework for the identification of frequent pattern trends | How are recurring patterns defined, screened, and validated out of sample? |
| A portfolio model for the risk management in public pension | How are liability risk, funding objectives, and investment constraints represented? |

### Credit, corporate finance, and public economics

| Title or topic lead | Question to investigate |
|---|---|
| Optimal cut-off points for multiple causes of business failure models | How do competing failure causes and asymmetric error costs change a classification threshold? |
| The common pool problem of intergovernmental interactions and fiscal discipline. A Stackelberg approach | How do sequential fiscal decisions affect incentives and the equilibrium? |
| Probability of default. A modern calibration approach | How are portfolio default rates, calibration targets, and time variation reconciled? |
| Development of a Basel II-compliant LGD model. A case study | How are recoveries, costs, discounting, incomplete workouts, and downturn conditions handled? |
| Integrated industrial and financial analysis into a rating methodology for corporate risk detection. The case of the Vicenza manufacturing firms | What industry variables add information beyond financial statements? |
| Corporate finance… What else? The case of the productive chain networks in north-east Italy and the scaffolding finance adopted by their leader | How do financial arrangements within production networks transmit or absorb risk? |
| Modelling the latent components of personal happiness | Which observations identify the latent factors, and how stable are they across samples? |

### Insurance, reserving, mortality, and pensions

| Title or topic lead | Question to investigate |
|---|---|
| Robust paradigm applied to parameter reduction in actuarial triangle methods | How much structure can be removed while preserving reserve accuracy? |
| Unlocking reserve assumptions using retrospective analysis | Which assumptions can be inferred or challenged from historical reserve revisions? |
| Spatial statistical tools to assess mortality differences in Europe | How are geographic dependence, small populations, and uncertainty handled? |
| Stochastic control for insurance | What state, action, and objective define the insurer’s decision problem? |
| Can personal dependency paths help to estimate life expectancy free of dependency? | How can transitions between dependency states inform healthy or independent life expectancy? |
| The estimation of standard deviation of premium risk under Solvency II | Which data and assumptions drive the estimate of underwriting variability? |
| The solvency capital requirement management for an insurance company | How do reinsurance, asset allocation, and capital interact under the chosen framework? |
| Alternative assessments of the longevity trends | How sensitive are projections to cohort, period, and extrapolation assumptions? |
| Empirical evidence on predictive accuracy of survival models | Which metrics capture useful calibration and discrimination under censoring? |
| Run-off error in the outstanding claims reserves evaluation | How should reserve uncertainty be measured over future development periods? |
| Stochastic mortality models. Application to CR mortality data | How do alternative mortality models perform on the dataset named in the title? |
| Pricing and hedging variable annuities | How do embedded guarantees, policyholder behaviour, and market risks affect the hedge? |
| A multivariate approach to project the long-run relationship between mortality indices for Canadian provinces | How can shared mortality trends coexist with differences between populations? |
| Measuring and managing longevity risk. Empirical evidence from the Italian pension market | How is longevity exposure quantified and translated into a risk management decision? |
| Risk processes with normal inverse Gaussian claims and premiums | What does this distribution change about surplus dynamics and ruin estimates? |

### Applied mathematics and numerical methods

| Title or topic lead | Question to investigate |
|---|---|
| Analysing differential equations with uncertainties via the Liouville–Gibbs theorem. Theory and applications | How is uncertainty propagated from inputs to the distribution of solutions? |
| Solving a time-space-fractional Cauchy problem with constant coefficients by finite differences | What stability and convergence properties does the scheme have? |
| On modification of an adaptive stochastic mirror descent algorithm for convex optimisation problems with functional constraints | How does adaptation handle noisy gradients and constraint violations? |
| Inductive description of quadratic Lie and pseudo-Euclidean Jordan triple systems | Which algebraic structures are classified and what makes the inductive construction work? |
| Comparative study of numerical methods for the standard FitzHugh–Nagumo equation | Which schemes best preserve the solution’s qualitative dynamics at practical step sizes? |
| Analytical solution of the neutron diffusion equation in reflected reactors using a modified differential transform method | How are boundary matching and accuracy evaluated? |
| Second-order perturbed state-dependent sweeping process with subsmooth sets | What regularity conditions support existence and stability? |
| Membrane hydrogen mixture separation. Modelling and analysis | How do transport assumptions and boundary conditions determine predicted separation? |
| An efficient hybrid conjugate gradient method for unconstrained optimisation | What convergence guarantee and benchmark support the proposed modification? |
| On polynomial decay of the wave equation with Wentzell conditions | How do the boundary conditions affect the energy decay rate? |
| Solutions of the fractional Verhulst model by modified analytical and numerical approaches | How does the fractional term change growth behaviour and numerical convergence? |
| Is it worthwhile considering orthogonality in generalised polynomial chaos expansions applied to stochastic models? | How do basis choice, input dependence, truncation, and numerical stability interact? |
