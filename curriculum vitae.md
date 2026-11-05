<!--
MASTER CV — Mark Brezina
This is the long-form reference record. Cut targeted CVs from it; don't send it as-is.
Hidden notes like this one don't render on GitHub but are visible in the raw file.
Open items are collected at the bottom.
Last updated: 2026-11-05
-->
# Name: Mark Daniel Balle Brezina 
London, UK · (+44) 7761 917 490 · [mark@brezina.dk](mailto:mark@brezina.dk) · [LinkedIn](https://www.linkedin.com/in/markbrezina95) · [GitHub](https://github.com/MarkBrezina)

UK right to work (partner visa, no sponsorship required)


# Name: Mark Daniel Balle Brezina 
Age: 31 \
Birthday: 4th, August, 1995 \
Height: 171cm \
Weight: 86kg \
Gender: Male \
IQ-Score: 159 (tested) \
Address: London, SE13 7UD, 6 Nuding Close
### Contact information 
UK-phone: (+44) 7761917490 \
DK-phone: (+45) 93904834 \
Private-email: mark@brezina.dk \
Work-email: cphtolon@gmail.com \
Linkedin: https://www.linkedin.com/in/coptolon/ \
GitHub: https://github.com/COPtoLON \
Wilmott-forum-username: CopenhagentoLondon, MarkBrezina \
Quantnet-forum-username: MarkBrezina 
### Availability 
• Immediately available for work. \
• Open to work: full-time, part-time, internship, contract, permanent. \
• Danish native, with citizenship. Available for work in Denmark. \
• European native, with citizenship. Available for work in all of the European Union, Iceland, Greenland, Norway, Switzerland and more. \
• UK partner visa holder, Right to work in the UK, no future sponsorship required. 

<!-- Full address used on applications: 6 Nuding Close, London SE13 7UD. Kept out of the rendered page in case the repo is public. -->

---

## Profile

Quantitative developer and researcher with experience in pension-fund risk analytics, credit-risk data quality at a bank, public-sector business intelligence and contact-centre reporting automation. I build data pipelines, risk and pricing models, simulation tooling and validation frameworks in Python, SQL and SAS. Alongside employment I run Tomorrow Capital Research, an independent venture covering systematic trading research, live execution infrastructure and a multiscale theory of market equilibrium. Studied Mathematical Statistics and Mathematical Physics at the University of Copenhagen.

**Featured project:** [Concursus](https://github.com/MarkBrezina/Concursus), a Python simulation library covering stochastic processes, particle physics, population dynamics and market models, with a benchmark that scores the market models against real data. [Details below](#featured-concursus).

---

## Skills

**Programming:** Python (pandas, NumPy, SciPy, scikit-learn, PyTorch/TensorFlow), SQL, SAS (incl. macros), R, MATLAB, VBA / VBScript, JavaScript (incl. Google Apps Script), C++, C#, HTML/CSS, Lua, Pine Script

**Data and platforms:** ETL pipeline design, exchange and broker REST APIs, Power BI (incl. DAX, MDX, Python), Excel (add-ins, automation), Google Sheets / Slides / Forms automation, Azure, Databricks, Supabase, Bloomberg, MSCI, Sustainalytics, Microsoft Navision and Dynamics, AS400 / IBM i

**Quantitative methods:** Probability and statistical inference, stochastic processes and SDE simulation (GBM, Ornstein–Uhlenbeck, CIR, Heston, Merton jump-diffusion, Hawkes), Monte Carlo and MCMC, time series (ARIMA/GARCH, stationarity and mean-reversion tests, variance ratios, Hurst exponent), Markov chains and Hidden Markov Models, Bayesian modelling, VaR/CVaR and stress testing, option pricing with stochastic volatility, limit-order-book microstructure, agent-based modelling, mean-field games

**Practice:** Data quality and reconciliation, model and performance validation, walk-forward testing, test design, KPI design, stakeholder and client reporting, teaching

---

## Experience

### Konecta — Customer Service Representative, Danish Market (Center Parcs account)
**London · May 2026 – Present**
*Management-information and reporting development alongside the representative role*

**Representative role**
- Handle inbound sales and service for the Danish market, plus overflow from the English-speaking foreign markets and outbound welcome calls, on the AS400 / IBM i booking system and the PVCP (Polo) booking platform.
- Highest booking retention in the Danish queue (every booking I have made has stayed booked), sales conversion above the European benchmark, and service handling time around one minute faster than the campaign standard.
- Rated 90–100% on listening, empathy, clarity and problem resolution in quality audits; 86.1% on the most recent service evaluation against a 75% target.
- Became a regular point of contact for Danish and Dutch colleagues on AS400 navigation, cancellation and modification rules, and call handling. Codified the most common rules into a shared booking calculator that derives cancellation fees, modification fees and refund entitlement from booking date, arrival date, add-ons and insurance status.

**Management information and reporting automation (self-initiated)**
- **Call-reason taxonomy.** With no agent-level call-reason reporting available, hand-logged every call (category, timestamp, note), derived a structured taxonomy of around a dozen reasons from the free text, and profiled inbound demand and outbound contact windows by hour — correcting for censoring caused by my own breaks, shift boundaries and long calls before using the results.
- **KPI reporting pipeline.** Replaced an AI-assisted (NotebookLM) reporting process that produced unreliable figures with a deterministic script-based pipeline. It reads six daily BI exports (daily, week-to-date and month-to-date across sales and service queues), rebuilds a seven-tab agent-level KPI pack for the Benelux and Danish markets, benchmarks against peer sites and scores conversion against target with RAG flags. In daily use by the account's management team.
- **Data quality controls.** Every published figure is reconciled against the source system's own subtotals, with cross-period checks (daily ≤ week-to-date ≤ month-to-date). Missing, stale or unreadable source files are flagged on the report rather than published as zeros, and an audit log records which source version produced each figure. Reports reset automatically at week and month boundaries.
- **Resilient ingestion.** Source files are republished daily under new identifiers, so they are resolved by folder and name, with automatic format conversion, timezone-safe date parsing and an incremental archive that rebuilds the week-to-date daily view from single-day extracts.
- **Welcome-call MI and client reporting.** Automated the daily import of the client's Excel extract from email, with de-duplication and dialling-format normalisation, feeding team and per-agent KPI tables and a weekly auto-generated slide used in client presentations to Center Parcs.
- **Best-time-to-call model.** Bayesian (Dirichlet–multinomial) model of outbound call outcomes by weekday and hour, with decay on older observations, ranking slots by expected value — contact likelihood weighed against negative customer reaction.
- **First-call-resolution and complaints analytics.** Built single-click repeat-call and complaint logging with access control and undo, feeding dashboards of reasons, trends and per-agent patterns. Consolidated the complaint, repeat-call and outbound-call tools into one Apps Script project (~2,000 lines, four modules) with a shared menu, an idempotent setup routine and isolated triggers.
- **Automated client slides.** Designed an idempotent daily slide export that tags its own output and redraws it in place, so repeated or failed runs never duplicate slides and slides are still found after the reader reorders the deck.
- **Testing.** Validated each tool before go-live against simulated test environments, including a strict mock of the Google Apps Script API (74 assertions across 8 scenarios), and an independent Python recalculation of every published figure from the raw data.

*Tools: Google Apps Script (JavaScript), Google Sheets / Slides / Forms, Python, AS400 / IBM i (5250), data reconciliation and QA, KPI design, Bayesian modelling*

---

### Tomorrow Capital Research — Founder, Quantitative Developer & Researcher *
**Remote (Copenhagen, then London) · January 2024 – Present**
*Independent research venture building systematic trading strategies, live execution infrastructure and the research behind them. Working towards a proprietary trading firm.*

**Live trading infrastructure**
- **Liquidity-gap mean-reversion bot (LiqGapMR).** Built a ~2,000-line Python trading system for crypto spot markets, first on Crypto.com and then ported to Revolut X behind a separate exchange-adapter layer, so the strategy, sizing, state machine and reporting stayed unchanged.
  - *Signal:* each minute, scans the most liquid spot pairs and tests every combination of 1/5/15/30/60-minute price drops against 0.3/0.5/1.0% thresholds. A trade needs both a qualifying drop and a last closed one-minute candle whose volume sits in the bottom 20% of the previous 200 candles — a sharp move on thin participation that is expected to revert.
  - *Positions and risk:* at most one position per asset, a fixed share of equity per position under a portfolio-level cap; exits at the pre-drop reversion target or after a maximum holding period; per-asset cool-downs.
  - *Costs:* entries must clear maker/taker fees, a configurable fee buffer and the exchange's whole-cent settlement rounding, so a signal is only taken if it pays after costs.
  - *Operational safety:* separate paper and live modes with their own state and journals; persisted state with atomic commits; reconciliation of local positions against the exchange (UUID client order IDs, a grace period for lagging order listings, tolerance for fees taken in the base asset); single-instance lock; graceful shutdown; Ed25519 API authentication; a supervisor script that restarts the bot after any exit.
  - *Reporting:* CSV trade journal and Telegram notifications for every entry and exit, plus scheduled portfolio summaries.
- **Deployment.** Runs on a dedicated machine. Live operation began in September 2026 with the aim of building a 12-month live trading log of returns and drawdowns.

<!-- CHECK before using live wording externally: which strategy/version is live, start date, capital (keep it as "small own capital" unless you want the number public), and results with fees included. Fill the results ledger in the TMRW workbook first. -->

**Quantitative research**
- **Mean-reversion and trend diagnostics** on 2.5–3.5 million one-minute crypto bars (BTC, with ETH, XRP and SOL for cross-asset work): ADF and KPSS stationarity tests, Hurst exponent, half-life estimation, Lo–MacKinlay variance ratios, Mann–Kendall trend tests and OLS slopes with HAC errors, run per period to separate mean-reverting from trending regimes. Decile studies of past against future returns at horizons up to 100 bars, with roughly 242,000 observations per decile.
- **Market-state fingerprinting.** Encoded recent price paths as discrete states (return-sign sequences, candle ranges, distances to moving averages, moving-average slopes and volume) over 12-hour, 24-hour, 3-day and 7-day windows, and mapped each state to empirical up/down probabilities 1 and 3 hours ahead. Earlier versions estimated Markov up/down-sequence tables and checked their accuracy on data not used to build them.
- **Statistical screening across assets.** Correlation and cointegration screens, cross-correlation, autocorrelation and partial-autocorrelation analysis, and empirical distributions of mean-reversion size and duration on hourly Binance data (2021–2024) for around 30 crypto pairs; calendar effects (monthly up/down probabilities, behaviour after yearly highs and lows).
- **Model exploration.** Hidden Markov Models for latent regimes, gradient boosting (XGBoost), LSTM forecasting and wavelet analysis, each compared against simple statistical baselines.
- **Strategy prototypes.** Bollinger and z-score mean reversion, local min/max reversion, trend following on rolling mean returns and positive/negative counters, 2σ breakouts, technical-indicator strategies (RSI, Fisher RSI, stochastic oscillator, ADX, MACD, Supertrend) and a combined multi-signal engine. Each was backtested with fees and scored on Sharpe ratio, drawdown and return.
- **Adaptive multi-strategy design.** Designed a layered system in which volatility and order-book regime classifiers allocate capital across options market making, mean reversion, pairs trading and follow-the-leader strategies, with a Greek-budget overlay and bandit-style online learning.

**Theory and working papers**
- **Aggregation Theory: A Multiscale Theory of Matching, Equilibrium and Control** (working paper, July 2026, 45 pp.). Models how individual order matching aggregates into market-level equilibria and trends. It separates three questions: what the market environment, agents and rules are; how to estimate a shifting equilibrium; and how an agent should act optimally within it. It covers mean-field games and mean-field control, open systems with agents entering and leaving, temporal aggregation and merging populations. It also states testable hypotheses (equilibrium restoration against migration) with a walk-forward test design on 279,120 one-minute BTC/USDT bars from January–July 2026.
- **A Multi-Layered Model of Financial Markets** (white paper, October 2025, 20 pp.). The market is modelled as an assets-and-venues graph, an agents graph, a hierarchy of rules, and a valuation surface running from fundamental value to traded price, all evolving over time. Applications include AI-driven strategies, systemic-risk and contagion modelling, and linking macro and micro analysis.
- **Research conjectures** (December 2025). *Path switching:* how to move optimally between strategies when the model of the market changes mid-trade. *Central pricing:* a multi-asset model splitting next-period prices into immediate order-flow impact and historical dependence, with mean-field effects from other participants.
- **Firm design.** Wrote the TMRW business plan. It sets out a four-layer operating model (business strategy, portfolio management, trading strategies, execution and market impact) and a pod-based organisation in which investment, risk, research, technology and operations each have a central function, with independent model validation, internal audit and infrastructure review.

**Websites and education**
- Building three connected sites: [Tomorrow Capital Research](https://tomorrowcapitalresearch.github.io/) (the research business), [Tomorrow Markets](https://tomorrowmarkets.github.io/) (a multiplayer trading and portfolio-challenge game built on a JavaScript order-book engine, with a Supabase back end) and [Tomorrow Library](https://tomorrowlibrary.github.io/) (an open quantitative-finance reading room for students and early-career quants).

*Tools: Python (pandas, NumPy, SciPy, statsmodels, scikit-learn, XGBoost, TensorFlow), REST exchange APIs (Binance, Crypto.com, Revolut X), SQL, C++, JavaScript, Supabase, LaTeX*

---

### Freelance Software Developer & Quantitative Consultant
**Self-employed · Remote · May 2024 – Present**

- **Web and game development (Jul – Sep 2026):** Built the website and web app for [Club Clue](https://www.clubclue.com/), a puzzle game for a colleague in which players solve romance-and-intrigue mysteries from clues (inspired by Murdle, re-themed around relationship drama).
- **Sports betting research (Oct – Nov 2025):** Designed probabilistic models for European football markets (1X2, goals and more) using historical performance, league effects and matchup dynamics. Modelled micro-events (shots, shots on target, corners, cards, fouls) with Poisson, negative binomial, Gamma, Normal and empirical distributions and conditional probabilities, with separate behaviour by team, league and competition stage. Built data intake for weather, line-ups, historical and player data, with a correlated model setup and bet-ordering across multiple bookmakers.
- **Strategy performance validation (Sep – Oct 2025):** Validated trading-book data for price integrity, transaction costs and execution consistency; recalculated CAGR, absolute and cumulative returns, volatility and Sharpe ratio from raw trades; documented methodological issues in backtesting assumptions and execution logic.
- **Low-latency crypto trading (Dec 2024 – Feb 2025):** Built a low-latency system for clients trading newly launched tokens as they reached the Raydium market.
- **Algorithmic scripting (May – Jul 2024):** Pine Script strategy on TradingView triggering trades on custom signals (session-time setups, exchange opening times, multi-day mean reversion).
- **Web development (May – Jul 2024):** Static HTML/CSS/JavaScript portfolio website for a hobby photographer.
- **Excel integrations (occasional):** Workbook enhancements in Python, C#, VBA and VBScript — dashboards, financial calculations and inter-application automation.

<!-- Raydium note (yours, kept private): built in Denmark around pump.fun launches; pump.fun isn't available in the UK. You've since researched other ways to use the same effect — if that becomes a TMRW strategy, record it there, not here. -->

*Tools: Python, Excel, VBA, C#, Pine Script, HTML/CSS/JavaScript*

---

### LD Pensions (LD Fonde) — Junior Quantitative Developer
**Copenhagen, Denmark · August 2023 – May 2024**
*Part-time student position, 15–35 hours per week, year-round. Danish pension fund managing and investing pension assets on behalf of its members.*

**Data pipelines and infrastructure**
- Built automated, scheduled Python ETL jobs (run through Windows Task Scheduler) that ingested, cleaned and unified data from Nykredit, Nordea, Danske Bank, Morgan Stanley, Bloomberg, MSCI and Sustainalytics into the fund's central database, so analytics were current at the start of each workday without manual collection.
- Integrated third-party APIs — the legacy Bloomberg API, MSCI ESG and Sustainalytics — into one Python environment covering market data, ESG metrics and analytics for equity and bond portfolios.

**Internal Python library ("LDF")**
- Developed the fund's on-premises Python package for quantitative analysis, risk modelling and data management, available only on LD's internal network.
- Refactored it from a functional to an object-oriented design, making the code easier to reuse, extend and maintain.
- Built an Excel add-in that exposed the library's functions inside Excel, so colleagues without Python could run the same analytics.

**Risk analytics and stress testing**
- Built modules for VaR, CVaR, credit, interest-rate, currency and derivatives risk, including fixed-income and yield-curve analytics, with particular focus on margin-call frameworks for currency futures and interest-rate-sensitive portfolios.
- Built stress-testing tools using quantile returns, drawdown analysis and log returns, giving portfolio managers scenario views of their mandates.
- **CIO project:** worked with the Chief Investment Officer to interpret Morgan Stanley's profit-forecasting data. Reverse-engineered the VaR, CVaR and waterfall-model calculations, built a local scenario simulator using geometric Brownian motion and square-root diffusion, and presented the findings to senior leadership.

**Portfolio framework**
- **Multi-tier hierarchy:** consolidated ETFs and asset classes (equities, bonds and others) into portfolios grouped under low-, medium- and high-risk mandates, linking database tables by asset ID so every holding could be traced through each portfolio level up to the mandate.
- **Return and risk monitoring:** return-on-investment and risk modules tracking daily exposures, aggregating market, interest-rate and currency risk from individual assets up to mandate level.
- **Visualisations:** asset allocation against compliance limits for proactive limit checks; each portfolio's NAV against its benchmark and peer portfolios; VaR plots showing potential drawdowns under adverse conditions.

**ESG analytics**
- Extended the library to calculate Weighted Average Carbon Intensity and Total Financed Emissions from MSCI, Bloomberg and Sustainalytics data, and added governance and social KPIs.

**Reporting and productivity**
- Added Python-based data preparation to Power BI dashboards used for executive reporting.
- Built VBA macros and Excel add-ins that automated budgeting tasks, email workflows and smaller recurring processes.

*Tools: Python, SQL, VBA, Power BI, Bloomberg/MSCI/Sustainalytics APIs, Windows Task Scheduler, VaR/CVaR, scenario simulation (GBM, square-root diffusion), portfolio construction*

<!-- To make this section stronger still, add whatever you can answer:
- Rough size of the data: how many sources/feeds, rows per day, number of instruments or mandates.
- Who used LDF and the add-in (how many people, which teams), and what they did manually before.
- Assets under management of the mandates you covered (only if public / allowed).
- Yield-curve work: what exactly — curve construction, duration/convexity, key-rate sensitivities?
- Margin-call framework: what it calculated and who acted on it.
- Anything that went live and stayed in use after you left. -->

---

### Lån & Spar Bank — Data Quality Analyst
**Copenhagen, Denmark · January 2023 – August 2023**
*Part-time student position, 15–35 hours per week, year-round. Danish retail bank.*

- Assessed and documented data-quality indicators for the bank's upcoming IRB system (2025 launch), analysing PD/PC, LGD/LGC, EAD and default-definition procedures across accuracy, completeness, consistency, validity, uniqueness and integrity.
- Worked with the head of IRB, IRB analysts and model validators to confirm the right variables were covered; produced data-quality reports and presented them to stakeholders.
- Mapped and diagrammed the SAS table architecture linking credit evaluation, IRB and transactional systems.
- Reviewed the code and data lineage of the high-risk customer evaluation process and presented findings to the heads of Credit, IT and Internal Audit.
- Built a SAS macro tool that tests database tables on five data-quality dimensions and traces upstream through every table that feeds them, checking referential integrity along the way.

*Tools: SAS, SQL, IRB framework (PD, LGD, EAD), data governance, Scrum*
*Training: SAS Institute courses in SAS and SQL*

---

### DJØF — VBA & Python Course Instructor
**Copenhagen, Denmark · January 2023 – August 2023**
*Danish Association of Lawyers and Economists.*

- Designed and taught Excel, VBA and Python courses, online and in person, from beginner to advanced level.
- Built advanced VBA material covering nested macros, custom functions and cross-application automation (VBA/VBScript).
- Based exercises on real tasks — automating routine processes, analysing datasets — and used Q&A sessions, breakout workshops and participant feedback to refine the material.

*Training: 3-day course in pedagogical presentation*

---

### Statistics Denmark (Danmarks Statistik) — Data Analyst & Developer
**Copenhagen, Denmark · August 2021 – August 2023**
*Part-time student position, 15–50 hours per week, year-round. Denmark's national statistics authority.*

**Timesheet and budgeting system**
- Administered the organisation's timesheet and budgeting platform, analysing hours worked, salary payments and budget distribution across departments.
- Ran the monthly SAS/SQL extractions behind budget and salary forecasts, employee-hour estimates and the reallocation of hours borrowed between departments and projects, used for resource planning.
- Integrated Microsoft Navision (bookkeeping), Microsoft Dynamics (CRM) and the legacy budget systems into SAS/SQL pipelines, giving finance, CRM and project management a consistent view of the same data.
- Took part in code reviews with colleagues to keep coding standards and data integrity consistent.

**Donations system**
- Helped manage the end-to-end donations system (C#, SQL, Excel), including documentation, development and maintenance.
- Built an API integration with the legacy donations platform so income, expenses and project usage could be monitored in close to real time.
- Gave the finance team a clear view of donation flows and forecasts of future budget movements.

**Project finance and analysis**
- Led development of a toolset in Excel, VBA/VBScript, SAS and SQL that consolidated project data — employee hours, overhead, CRM costs and external spending — into one place.
- Classified projects by profitability, project lead and expense flow, making resource allocation and returns on projects visible.
- Automated email reminders prompting project leaders to update or close completed and annual projects.
- Streamlined email processing, data retrieval and dashboard production around the project portfolio.

**Business intelligence and executive reporting**
- Built and maintained Power BI dashboards for senior management on fund allocation, project performance and key metrics, used in executive decision-making.
- Ran ad-hoc analyses in SAS, SQL and Excel of funding surpluses and shortfalls, employee utilisation and returns on major organisational initiatives.
- Worked with stakeholders across departments to support policy decisions with data and simplify internal processes.

**Other**
- Passed security vetting, including a criminal-record check.

*Tools: SAS, SQL, C#, VBA/VBScript, Excel, Power BI, Microsoft Navision, Microsoft Dynamics, ETL pipelines*
*Training: SAS Institute programmer course series; internal courses in Statistics with R I & II and Introduction to Demographic Statistics*

<!-- To make this section stronger still, add whatever you can answer:
- Scale: number of employees/departments/projects covered by the timesheet and budget work; size of the annual budget it fed.
- Which forecasts you produced (salary, hours) and what method — trend, seasonal, regression?
- How many projects the project-finance toolset tracked; what changed after it (time saved, projects closed, overspend caught).
- Who used the Power BI dashboards (director level, board?) and a decision they informed.
- What the donations API replaced (manual exports?) and how often it ran. -->

---

### MentorDenmark — Senior Mentor, Physics, Mathematics & Science
**Copenhagen, Denmark · August 2017 – August 2021**
*Denmark's largest tutoring company.*

- Top-5 senior mentor: over 1,000 hours tutoring mathematics, physics and science to students aged 7–23, with an average satisfaction rating of 4.7/5; students consistently reached above-average grades.
- Prepared more than 50 students for oral and written mathematics exams through curriculum review, mock exams and simulated examiner scenarios.

*Training: 2-day course in pedagogical presentation*

---

### Thomas Lystlund Studio — Data Analysis Intern
**Copenhagen, Denmark · March 2019 – May 2019**
*Single-project internship (unpaid).*

- Analysed cash flows and projected returns of forex trading bots the client was considering, identified likely scams and recommended safer alternatives.

---

## Other Employment

| Period | Role | Employer |
|---|---|---|
| Jul 2025 – Oct 2025 | Short-let cleaner (Soho, Waterloo, Mile End) | Coopy Properties, London |
| May 2024 – Jan 2025 | Train station service personnel — passenger assistance incl. disabled travellers, station safety and cleaning | DSB, Copenhagen |
| Aug 2021 – Aug 2022 | Store clerk, Copenhagen Airport | WHSmith |
| Sep 2020 – Mar 2021 | Home and school cleaner | Self-employed, Copenhagen |
| Apr 2019 – Aug 2020 | Store clerk (incl. forklift and pallet-jack operation) | Lidl, Copenhagen |
| Oct 2016 – Dec 2018 | Night-shift store manager, supervising two staff | MinKøbmand, Copenhagen |

*Training: Psychological first aid for trauma and shock, chemical safety, track safety (DSB); Copenhagen Airport Security Awareness (WHSmith)*

---

## Education

### BSc Mathematical Statistics — University of Copenhagen
**2021 – 2025 (not completed)**
- Bachelor project: *Option Pricing with Stochastic Volatility in the Heston Model.* An R/C++ package, developed with a company that now uses it commercially, covering Heston pricing by numerical integration of the characteristic function, Black–Scholes benchmarks, implied-volatility root-finding, model volatility surfaces fitted to AAPL option data, and Longstaff–Schwartz least-squares Monte Carlo for American options.
- First-year project: *Event Study in the Stock Market* (Apple's announced acquisition of AuthenTec).
- Relevant courses: Mathematical Statistics, Stochastic Processes I & II, Advanced Probability Theory, Life Insurance, General Insurance, Differential Geometry.

Thesis: “Option Pricing with Stochastic Volatility in the Heston Model” - developed as a library in R and used by the company EnvisionRisk \

### BSc Mathematical Physics — University of Copenhagen
**2019 – 2021 (withdrew)**
- First-year project: *Estimating Dark Matter Density from Spiral Galaxy Rotation Curves*.
- Relevant courses: Mechanics I & II, Thermodynamics, Electrodynamics, Quantum Mechanics I & II, Galactic Astrophysics, Complex Analysis I & II.

### Upper secondary (STX) — KNORD Lyngby
Physics A, Mathematics A, English A, Danish A, Chemistry B.
### Ordinary student exam - KNord Lyngby STX, Denmark Kongens Lyngby 
Subjects: Danish A, English A, History A, Mathematics A, Physics A, Chemistry B, German B, Biology C, Design C, Innovation C, Philosophy C, Religion C, Social Science C, Natural science introduction C, General language C \
GPA: approximately 8.5/12. 

---

## Projects *

### Featured: Concursus - Done
**2023 – Present** · [Repository](https://github.com/MarkBrezina/Concursus) · *Python (NumPy, SciPy, pandas, matplotlib); installable package `stochastics` v0.5*

A simulation library built on one idea: stochastic processes, population dynamics, finance and physics are all simulation problems, so they belong in one package with shared conventions. I designed the architecture, API, models and validation approach; large parts of the code were written with AI assistance, as the repository states. The package grew out of three earlier pieces of my own work (a 2023 stochastic-process class, galaxy-rotation and particle-swarm simulations, and an insurance model), kept in `legacy/`.

- **Baseline processes:** random walk, Wiener, arithmetic and geometric Brownian motion, Ornstein–Uhlenbeck, CIR, Poisson and compound Poisson, Brownian bridge, Merton jump-diffusion, Heston, fractional Brownian motion, Hawkes, regime switching and GARCH, plus general Euler–Maruyama and Milstein SDE solvers. Exact transition sampling is used wherever it exists, and every generator returns its theoretical mean, variance and terminal density so simulation can be checked against theory.
- **Finance:**
  - a price-time-priority limit order book;
  - an agent-based exchange simulating a full trading day with market makers, trend followers and noise traders;
  - market models of increasing complexity (single Heston path → correlated stream ensembles → internal-value streams with traders → news jumps);
  - draft trader profiles described by psychology (resilience, greed, patience) rather than strategy class;
  - an insurance module covering claim arrivals (Poisson, renewal, Hawkes), severity distributions, premium principles and Monte Carlo ruin probabilities.
- **Real-data benchmark:** scores each market model against real series (S&P 500, EUR/USD, one-minute BTC) on a fingerprint of twelve scale-free stylised facts — tail shape, volatility clustering, leverage effect, variance ratios. Errors are measured in units of the real series' own bootstrap sampling noise. Models are calibrated by simulated method of moments and checked out of sample on a holdout half. The benchmark also maps the trade-off between model fidelity and compute cost (Pareto frontier).
- **Population dynamics:** agent-based ecosystem (grass → rabbits → wolves), spatial SEIR epidemics with infection trees, rock–paper–scissors cyclic competition, wealth-exchange economies and Schelling segregation, with per-agent traits and conserved-quantity bookkeeping.
- **Physics:**
  - gravity and electromagnetism with velocity-Verlet integration, collisions, mergers and an exact energy ledger, so any energy drift is traced to the integrator or to a specific rule;
  - a "policy lab" in which the interaction laws themselves are designed and checked against Coulomb and Newtonian gravity as reference cases.
- **Quality:** around 175 unit tests across the subpackages, and eight showcase notebooks.

### TMRW — personal trading and analytics library
**2019 – Present** · [Repository](https://github.com/MarkBrezina/TMRW)

The research library behind Tomorrow Capital Research:
- data extraction from Yahoo Finance, Binance, Bloomberg, MSCI and Sustainalytics;
- technical and statistical indicators, Markov up/down tables and trend detection;
- portfolio metrics (VaR, CVaR, drawdown, annualised return);
- backtesting with configurable signals (mean reversion, trend following, pairs trading);
- Monte Carlo and scenario stress testing.

<!-- The "13% short-term return on real-time deployment" claim stays out. Add it back only with period, capital base, costs and whether gross or net. -->

### WunderConnectome — limit-order-book sequence prediction - Done
**2026** · [Repository](https://github.com/MarkBrezina/WunderConnectome)
Entry for the Wunder Fund Alpha Connectome competition: a GRU network trained on roughly 10,000 order-book sequences to predict two short-horizon targets. Finished in the top 20% of 5,700 active teams.
*Note: I honestly was very dissatisfied with my performance here, my concluding upload was simply the baseline model, with some additional features and extensive training. \
I had many great ideas while working and reflecting on this challenge, but none of them lead to anything conclusive. THis was my own fault, as I have not properly taken the time to understand how AI trading works.*


### Ctrl+Alt+DefeatTheMarket — IMC Prosperity unofficial guide
**2025 – Present** · [Repository](https://github.com/MarkBrezina/Ctrl-Alt-DefeatTheMarket) · *Python, pandas, NumPy, SciPy, networkx, matplotlib, plotly*

Research and trading framework for high-frequency market microstructure and systematic strategy design, built around the IMC Prosperity competition.
- **Microstructure:** order-book features (imbalance, order flow, VWAP, microprice, liquidity-weighted fair value); decomposition of price moves into order-flow, drift and mean-reversion components.
- **Strategies:** market-making, mean-reversion and drift-aware strategies under inventory and spread constraints; hybrid execution combining passive quoting with aggressive taking on mispricing; multi-signal blending to adjust quotes and inventory targets.
- **Testing:** tick-level backtester; robustness checks (detrending, z-scores, rolling regression forecasts); scenario tests across volatility, spread and liquidity regimes.
- **Execution and risk:** inventory-aware quoting with dynamic skew; trade classification, order placement and fill simulation.

### Algorithmic trading research
**2022 – Present** · [Repository](https://github.com/MarkBrezina/WTech)

Development and evaluation of trading algorithms — mean reversion, momentum and pattern detection — using stochastic processes, time-series analysis and statistical optimisation, with emphasis on reproducible testing across market conditions.
### Knowledgebase — mathematics and finance resources
**2023 – Present** · [Repository](https://github.com/MarkBrezina/Knowledgebase)

Curated study material in finance and mathematics — concept overviews, worked problems and references — intended to grow into a book and feeding the Tomorrow Library site.

<!-- Not yet listed:
- Market-States-Regimes, Foundational-Quantitative-Research, QPM, KB, BasicIT, Personal — add once each has a README that says what it shows.
- CONTROL (AI for trading) — still empty; leave off until there's code.
- GoldenDice and Galaksim are now superseded by Concursus (legacy/). Consider archiving GoldenDice or pointing its README at Concursus.
- ETL package (Yahoo, Alpha Vantage, Binance, Jupiter) — link pending.
- Event-study code — link pending. -->

---



## Competitions and Recognition

- **IMC Prosperity 3** algorithmic trading challenge — rank 107 of 12,000+ teams (top 1%)
- **IMC Prosperity 4** algorithmic trading challenge — rank 741 of 6,500 active teams (top 11.4%; 22,000 registered)
- **Wunder Fund Alpha Connectome** (2026) — rank 920 of 5,700 active teams (top 20%)
- **Børsen stock-picking challenge** — top 10% in three separate years: rank 2,800 of 36,000+ (2023); rank 468 of ~47,000 (2024, top 1%); rank 1,809 of ~28,500 (2026, top 6.3%)
- **Design for Change** awardee (2010) — international competition for innovative solutions to everyday problems
- **MENSA** member since 2018 (assessed IQ ≥ 158)
- LinkedIn skill assessments passed: R, Python, VBA, Excel, MATLAB
- **Games:** StarCraft II (mid Masters league), Teamfight Tactics (Gold), active chess player
- **Steam marketplace trading:** grew €100 to ~€300 over three years trading Counter-Strike cases
- Strength training and fitness

<!-- MENSA membership no. 2018170. The IQ figure and the games line are fine in this master record; leave them off most employer CVs (games can stay for trading firms that like StarCraft/chess).
Chess rating left out on purpose — 330 reads as a beginner rating; say "active chess player" unless you have a stronger figure. -->

---

## Professional Development

### Lectures and conferences

| Date | Organiser | Talk / event |
|---|---|---|
| 2025-02-13 | Oxford Mathematical and Computational Finance | Eghbal Rahimikia — Revisiting Large Language Models in Finance |
| 2025-02-27 | Quant Insider | Andrey Chirikhin — Introduction to Credit Derivatives |
| 2025-03-08 | Bangalore Quant Club | Peter Cotton — Machine Learning for Hierarchical Portfolio Optimization Using Schur Complements |
| 2025-03-11 | Imperial College London | Daniel Lacker — Non-asymptotic Perspectives on Mean-Field Approximations and Stochastic Control |
| 2025-03-12 | CQF Institute | Portfolio Management in Quant Finance Conference |
| 2025-04-08 | CQF Institute | Wim Schoutens — Omnipresent Model Risk |
| 2025-05-07 | CQF Institute | Laura Ballotta — The Term Structure of Implied Correlations Between S&P and VIX Markets |
| 2025-06-04 | CQF Institute | Volatility and Risk in Quant Finance Conference |
| 2025-06-10 | CQF Institute | Financial Reasoning Agents: In-Context Reinforcement Learning and Test-Time Compute |
| 2025-07-09 | CQF Institute | Optimization of Strategic and Tactical Asset Allocation for Multi-Asset Portfolios |
| 2025-08-12 | CQF Institute | Modelling Non-Maturing Deposits |
| 2025-09-09 | CQF Institute | How to Add Currencies to a Portfolio: Currency Benchmarks and Active Currency Management |
| 2025-09-17 | CQF Institute | AI and Machine Learning in Quant Finance Conference |
| 2025-10-14 | London Quant Group | Instability: Debt, Inflation and AI's Impact on Investing |
| 2025-11-04/05 | CQF Institute | Annual Quant Insights Conference |
| 2025-11-06/07 | London–Paris Bachelier Workshop | 9th London–Paris Bachelier Workshop |
| 2025-11-12 | CQF Institute | Samit Ahlawat — Generating Realistic Economic Scenarios for Stress Testing Portfolios Using Generative AI |
| 2025-11-19 | CQF Institute | London Society Meeting: Market Microstructure |
| 2025-12-03 | CQF Institute | Peter Simpson & Alexander Serechenko — The Quant's AI Advantage: Leveraging AI for Natural Language Assistance in Trading Analytics |
| 2025-12-09 | CQF Institute | Stephen Weston — Why So Much Science Doesn't Hold Up and What Quantitative Minds Can Do About It |
| 2025-12-16 | London Quant Group | Annual Debate — Ed Fishwick & Henry Neville |
| 2026-01-15 | London Quant Group | Semyon Malamud — Limits to (Machine) Learning |
| 2026-02-11 | CQF Institute | Lorne Switzer — Trade Wars to Market Wars: Quantifying Geopolitical Impact on Equity Returns |
| 2026-02-17 | London Quant Group | Edward Fishwick — What We Learned About Risk |
| 2026-02-26 | S&P Global | London Quant Intelligence Forum 2026 |
| 2026-03-11 | CQF Institute | Portfolio Management in Quant Finance Conference |
| 2026-03-18 | CQF Institute | Leif Andersen — American Option Pricing by the Finite Difference Method: Tips and Tricks from the Trenches |
| 2026-03-18 | DefconQ × Squarepoint | KDB Developers Meetup |
| 2026-04-14 | London Quant Group | Simon Polbennikov — Better Together: Fundamental and Systematic Views in Credit Investing |
| 2026-04-29 | CQF Institute | John Guerard — ESG Investing and Portfolio Selection |
| 2026-05-12 | London Quant Group | Robert Macrae — Adventures in High-Dimensional Space |
| 2026-06-09 | London Quant Group | Brunello Rosa — Making Sense of This World and the Resurgent Chinese Empire |
| 2026-07-14 | — | Maxime Bonelli — Alternative Data, Traditional Expertise and Ownership |
| 2026-09-13 | Bloomberg | Bloomberg Data and Tech Summit |
| 2026-10-13 | London Quant Group | Manav Chaudhary — The Anatomy of the Treasury Market |

<!-- CHECK: 2026-06-09 title had "(...)" in your notes — fill in the full title. 2026-07-14 organiser missing. 2026-09-13 is a Sunday — confirm the date. Your note says you stopped logging for a while, so there may be gaps between June and October 2026. -->

- Unknown, multiple - Henrik Hørsløv Larsen - beer meetup 
- 7/2/2025 - Peter Pommergård Holm - beer meetup 
- 13/2/2025 - Oxford Mathematical and Computational Finance - Eghbal Rahimikia - Revisiting Large Language Models in Finance 
- 27/2/2025 - Quant Insider – Andrey Chirikhin – Introduction to Credit Derivatives 
- 8/3/2025 - Bangalore Quant Club - Peter Cotton - Machine Learning for Hierachical Portfolio Optimization Using Schur Complements 
- 11/3/2025 - Imperial College - Daniel Lacker - Non asymptomatic perspectives on mean field approximations and stochastic control 
- 12/3/2025 - CQF Institute - Portfolio Management in Quant Finance Conference 
- 8/4/2025 - London Quant Group & Bloomberg - Smart Money, Digital Currencies and the Emerging New World Order 
-  7/5/2025 - CQF Institute - Professor Laura Ballotta - The Term Structure of Implied Correlations Between S&P and VIX Markets 
- 11/5/2025 - London Investors and Traders Group - Discussion & Networking 
- 12/5/2025 - Great Britain Crypto, Forex/Gold, Stock/Index Trading Community Meetup! 
- 13/5/2025 - London Quant Group - Artur Sepp - The Science and Practice of Trend
- ??? Bloomberg event
- 4/6/2025 - CQF Institute - Volatility conference
- 5/6/2025 - Learning the values of illiquid bonds
- 11/6/2025 - Financial Reasoning Agents: In-Context Reinforcement Learning and Test-Time Compute' with Nicole Königstein


### Certificates

**2025**
- [Citi — Markets Quantitative Analysis](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/8eNRcRqBZM9HLvwGw/fAuNBibfF4dzrgtDY_8eNRcRqBZM9HLvwGw_nCjQg8GM2BpQ9yNYB_1736301928896_completion_certificate.pdf) (Forage)
- [Standard Bank — Risk Management](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/kkKXfgG5FckTX8Toc/9ud3PRZGf8w6NeHgh_kkKXfgG5FckTX8Toc_nCjQg8GM2BpQ9yNYB_1736293804035_completion_certificate.pdf) (Forage)
- [Goldman Sachs — Risk](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/MBA4MnZTNFEoJZGnk/ETGMhLB5eCrYjcH8o_MBA4MnZTNFEoJZGnk_nCjQg8GM2BpQ9yNYB_1736298583661_completion_certificate.pdf) (Forage)
- [Standard Chartered — Credit Analyst](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/ZsgybEFo5RxyHxwXL/TwxTEuyPugfgMGm8H_ZsgybEFo5RxyHxwXL_nCjQg8GM2BpQ9yNYB_1736299213999_completion_certificate.pdf) (Forage)
- [Lloyds Banking Group — Data Science](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Zbnc2o4ok6kD2NEXx/EuvC8GPjkZ6xaiP9p_Zbnc2o4ok6kD2NEXx_nCjQg8GM2BpQ9yNYB_1736293191281_completion_certificate.pdf) (Forage)
- [Goldman Sachs — Excel Skills for Business](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/MBA4MnZTNFEoJZGnk/eLeZHcwX7CqLhhps7_MBA4MnZTNFEoJZGnk_nCjQg8GM2BpQ9yNYB_1736289958952_completion_certificate.pdf) (Forage)
- [Citi — Finance](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/8eNRcRqBZM9HLvwGw/Cf35qdHjFPqLFRhgy_8eNRcRqBZM9HLvwGw_nCjQg8GM2BpQ9yNYB_1736291737128_completion_certificate.pdf) (Forage)
- [J.P. Morgan Chase & Co. — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Sj7temL583QAYpHXD/E6McHJDKsQYh79moz_Sj7temL583QAYpHXD_nCjQg8GM2BpQ9yNYB_1736293037192_completion_certificate.pdf) (Forage)
- [Quantium — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/32A6DqtsbF7LbKdcq/jhiG2W9K8KLZK8nXP_32A6DqtsbF7LbKdcq_nCjQg8GM2BpQ9yNYB_1736292440025_completion_certificate.pdf) (Forage)
- [Goldman Sachs — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/MBA4MnZTNFEoJZGnk/NPdeQ43o8P9HJmJzg_MBA4MnZTNFEoJZGnk_nCjQg8GM2BpQ9yNYB_1736289909624_completion_certificate.pdf) (Forage)
- [Standard Bank — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/kkKXfgG5FckTX8Toc/JMcJbndBmiDBvy9pj_kkKXfgG5FckTX8Toc_nCjQg8GM2BpQ9yNYB_1736293158178_completion_certificate.pdf) (Forage)
- [Commonwealth Bank — Software Development](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/2sNmYuurxgpFYawco/xv8eSGu7nksKNiCQj_2sNmYuurxgpFYawco_nCjQg8GM2BpQ9yNYB_1736292703452_completion_certificate.pdf) (Forage)
- [Citi — Investment Banking](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/8eNRcRqBZM9HLvwGw/amBSJDTDDFcYiKq9Z_8eNRcRqBZM9HLvwGw_nCjQg8GM2BpQ9yNYB_1736289659348_completion_certificate.pdf) (Forage)

**2024**
- [J.P. Morgan Chase & Co. — Quantitative Research](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/J.P.%20Morgan/bWqaecPDbYAwSDqJy_JPMorgan%20Chase%20&%20Co._nCjQg8GM2BpQ9yNYB_1706747633338_completion_certificate.pdf) (Forage)
- [Standard Bank — Actuarial](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Standard%20Bank%20/wf8DwPiWqcRzs3KDF_Standard%20Bank%20_nCjQg8GM2BpQ9yNYB_1706747691149_completion_certificate.pdf) (Forage)
- [PGIM — Fixed Income](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/PGIM/5tB9veyHmFh4egh65_PGIM_nCjQg8GM2BpQ9yNYB_1706747334656_completion_certificate.pdf) (Forage)
- [Cognizant — Artificial Intelligence](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Cognizant/5N2ygyhzMWjKQmgCK_Cognizant_nCjQg8GM2BpQ9yNYB_1706744853085_completion_certificate.pdf) (Forage)
- [AIG — Underwriting](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/AIG/6miFEe83Y7PdzWJD6_AIG_nCjQg8GM2BpQ9yNYB_1706744611697_completion_certificate.pdf) (Forage)
- [Bank of America — Investment Banking](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Bank%20of%20America/Hi8MJQE5T3MeTRWQR_Bank%20of%20America_nCjQg8GM2BpQ9yNYB_1706743366428_completion_certificate.pdf) (Forage)
- [J.P. Morgan Chase & Co. — Investment Banking](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/JPMorgan%20Chase/YD2kY95RQxQtXxFTS_JPMorgan%20Chase_nCjQg8GM2BpQ9yNYB_1706743099951_completion_certificate.pdf) (Forage)
- [Commonwealth Bank — Introduction to Data Science](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Commonwealth%20Bank/smwfytX3mcLboA9bf_Commonwealth%20Bank_nCjQg8GM2BpQ9yNYB_1706744285551_completion_certificate.pdf) (Forage)
- [Quantium — Data Analytics](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Quantium/NkaC7knWtjSbi6aYv_Quantium_nCjQg8GM2BpQ9yNYB_1706744085389_completion_certificate.pdf) (Forage)
- [BCG — Data Science](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/BCG%20/Tcz8gTtprzAS4xSoK_BCG_nCjQg8GM2BpQ9yNYB_1706744880479_completion_certificate.pdf) (Forage)
- [Accenture — Data Analytics and Visualization](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Accenture%20North%20America/hzmoNKtzvAzXsEqx8_Accenture%20North%20America_nCjQg8GM2BpQ9yNYB_1706742179438_completion_certificate.pdf) (Forage)
- [J.P. Morgan Chase & Co. — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/J.P.%20Morgan/R5iK7HMxJGBgaSbvk_J.P.%20Morgan_nCjQg8GM2BpQ9yNYB_1706746848174_completion_certificate.pdf) (Forage)
- [Standard Bank — Software Engineering](https://forage-uploads-prod.s3.amazonaws.com/completion-certificates/Standard%20Bank%20/8Mgcdiuo2mfSRCfjc_Standard%20Bank%20_nCjQg8GM2BpQ9yNYB_1706746105628_completion_certificate.pdf) (Forage)
- [Fundamentals of Quantitative Modeling](https://coursera.org/share/29d2b0531431e672bbbc3716439bee9d) (Coursera, Wharton)
- [CME Group — Introduction to Futures](https://github.com/CtoL95/Test/blob/main/CME%20Introduction%20to%20Futures.pdf)
- [CME Group — Introduction to Treasuries](https://github.com/CtoL95/Test/blob/main/CME%20Introduction%20to%20Treasuries.pdf)
- [CME Group — Long Options: Underlying and Volatility Impact](https://github.com/CtoL95/Test/blob/main/CME%20Long%20Options_%20Underlying%20and%20Volatility%20Impact.pdf)
- [CME Group — Understanding Futures Spreads](https://github.com/CtoL95/Test/blob/main/CME%20Understanding%20Futures%20Spreads.pdf)

**2021**
- [GARP — Foundations of Financial Risk](https://github.com/CtoL95/Test/blob/main/Foundations_of_Financial_Risk.pdf)
- [CFI — Introduction to Banking](https://credentials.corporatefinanceinstitute.com/524a2a7e-2bf8-4ae9-a0ac-2603b9e59645#acc.Nipz3Xqa)
- [CFI — Fundamentals of Credit](https://credentials.corporatefinanceinstitute.com/c7e5e7e2-ecec-49db-82bd-eadece36b10a#acc.pKbN1mSr)
- [Copenhagen Airport — Security Awareness](https://github.com/CtoL95/Test/blob/main/Security%20Awareness%20K%C3%B8benhavns%20Lufthavn%2C%20DK%20version.pdf)

**2020**
- [Microsoft (edX) — Professional Certificate in Data Analysis: Essential Skills](https://credentials.edx.org/credentials/5470fff383e1445f86fb1174a0dbab5a/)
- [Columbia University (edX) — Professional Certificate in Corporate Finance](https://credentials.edx.org/credentials/c3f74af6ade049a2857e2f30ef81c4c2/)
- [IMF (edX) — Macroeconometric Forecasting](https://courses.edx.org/certificates/8bf91f734bf7464c881c9ff61edd1d8c)
- [IMF (edX) — Macroeconomic Diagnostics](https://courses.edx.org/certificates/02e1984556594aa79033f8b14cc5c1db)
- [IMF (edX) — Financial Market Analysis](https://courses.edx.org/certificates/f7994f95d3554a4791bff62b770f6d54)
- [NYIF (edX) — Risk Management and Credit Principles](https://courses.edx.org/certificates/f2a38a40301f46d18aeaf269a54a0878)
- [NYIF (edX) — Ratio Analysis](https://courses.edx.org/certificates/2e9cb658318348d4a1ba75fa1acdce80)
- [World Bank (edX) — The Future of Work: Preparing for Disruption](https://courses.edx.org/certificates/5c696f67576c420989d859d6a201aaf0)
- [World Bank (edX) — Unlocking Investment and Finance in Emerging Markets and Developing Economies](https://courses.edx.org/certificates/98f9428afa2b42aa93cf316d13935ff2)
- [LinkedIn Learning — Become a Data Scientist](https://www.linkedin.com/learning/certificates/ec62ce4f36d64dbe064544533519c3f65ca19e95424d11f8246ea5c23d2e5ade)
- [LinkedIn Learning — Advance Your Skills in Python](https://www.linkedin.com/learning/certificates/498fc9d230524f23060c80e10b39104e1de568572852418d855a60d268472534)
- [LinkedIn Learning — Advance Your Python Skills for Data Science](https://www.linkedin.com/learning/certificates/797299e19c57fa257790df3a6f985a4778ea1f72d68304b8162bbdc6ab9780af)
- [LinkedIn Learning — Getting Started with R for Data Science](https://www.linkedin.com/learning/certificates/db08e02f98cf05df162bea2ee2e05252d2a241a3aa05ed7ceff79e1b84c54d61)
- [LinkedIn Learning — Master Key Financial Analyst Skills](https://www.linkedin.com/learning/certificates/8312aa1bbbf8b237d63388ca7edc710f81d29bb911b31576d7a6f0f2f6e589ce)
- [LinkedIn Learning — Getting Started in Finance and Accounting](https://www.linkedin.com/learning/certificates/8d831a982becf9f943394377e6eb85abb2a4ffffe8cac39b1317106e8db398ba)
- [LinkedIn Learning — Getting Started in Finance and Bookkeeping](https://www.linkedin.com/learning/certificates/8d00d39513032ae2ec43bfb350fc7d6522aa23b3c7657ceffa62d104896fdf88)
- [LinkedIn Learning — Getting Started as a Business Analyst](https://www.linkedin.com/learning/certificates/41455540e531d4b8d086b9686f6c65ef60d52f3881ab38a004c2c9b1d315795a)

**2019**
- [LinkedIn Learning — Master Microsoft Word](https://www.linkedin.com/learning/certificates/048c0b30fe9eed3001e98d79c426b5cd4f65d4bda95b29ebad50dbe5d66d451d)
- [LinkedIn Learning — Master In-Demand Professional Soft Skills](https://www.linkedin.com/learning/certificates/45e07396af21f07548621605ef6b1db8f3206557b6d2c3fda9526929e573f971)
- [LinkedIn Learning — Improve Your Organizational Skills](https://www.linkedin.com/learning/certificates/137a5099ab65aa75e21b4e03c7522bf0fc59447a5711d4d4f6f16604938365b2)
- [LinkedIn Learning — Improve Your Interoffice Politics Skills](https://www.linkedin.com/learning/certificates/ec61e527b5d2a1bab29a7e2a5aa8a9c07b7a81f141f5e46a9784fa03771560f0)
- [LinkedIn Learning — Advance Your Career as an Individual Contributor](https://www.linkedin.com/learning/certificates/00d00324e1197ecb6c154fb51e5179790876b1a0dbd42846c58096f20272a9d9)

**Language certificates**
- [LanguageCert — ESOL International B2 (English)](https://github.com/CtoL95/Test/blob/main/LanguageCert%20-%20ESOL%20Written.pdf)
- [Trinity College London — GESE Grade 2 (CEFR A1)](https://certificates.trinitycollege.com/3c9b7eda-0793-41d5-94e4-8d4f58dd8967#acc.H0LPcsja)

<!-- The GESE A1 certificate is a visa requirement; leave it off employer-facing CVs, since it understates your English next to the B2. -->

---

## Memberships

- CQF Institute, Wilmott Forum, ML-Quant, Quantpedia
- MENSA (2018 – present)
- UNF — Danish Youth Association for Natural Science (2013 – 2017); UNF Physics Camp (2015)
- ATU — Danish Academy of Talented Youth (2014 – 2016)

---

## Languages

- **Danish:** native
- **English:** fluent (LanguageCert B2)
- **Swedish, Norwegian:** conversational
- **German, Dutch:** good comprehension, basic spoken

---


LanguageCert Level 1 Certificate in ESOL International (Listening, Reading, Writing, 
Speaking) (Communicator B2) - 2024 
• Listening 49/50 
• Reading 41/50 
• Writing 30/50 
• Speaking 50/50 
Mensa Denmark Certificate - since 2018 
I have been tested by Mensa Denmark to have a score in the upper 1% of the danish population, 
with 42/45 correct on the test. Online tested to have IQ ≥158 
I'm the investment advisor and custodian for my aunt Jeannette Brezina, since 2024 
Member: Mensa, Wilmott Forum, CQF institue, London quant group, Oxford-man, ML-quant, 
Quantpedia 
Reviewer and donator(2024–2025) for Anton Vorobets’s upcoming book on portfolio 
construction and risk management. 
Fitness enthusiast: committed to working out and maintaining an active lifestyle. 
Active creative: I enjoy painting nature and drawing people in my pastime. It has come to good 
use in graphics design for resumes and websites 
UNF (Danish Youth Foundation of Natural Science) member (2010–2013) 
ATU (Danish Academy of Talented Youths) member (2014–2016). 
Awards 
• Top 0.5% (#107 of 12.5K team) ranking in IMC Prosperity 3 Algorithmic-Trading 
challenge. 2025 
• Top 2% (#456 of 47K individuals) ranking in Børsen stock trading challenge 2024. 
• Top 10% (#4000 of 42K individuals) ranking in Børsen stock trading challenge 2023. 
• StarCraft II: Ranked Masters League player 
• Passed LinkedIn Skill Assessments in R, Python, VBA, Excel, and MATLAB. 
• I was a participant at the 2015 UNF Fysik Camp. I was one of 50 participants out of 
everyone (expected more than 1000) who sought to participate. 
• Participant and north zealand runner-up for European business games 2014 
• Design for Change Awardee (2010) for a design competition. Crowdbins was a project 
for the Design For change competition in Denmark, my group was awarded and won the 
danish competition with Crowdbins. An app and trashbin interconnected system to 
inform and relieve sanitation workers. 
o Newspaper article 1 - link unattached at the moment 
o Newspaper article 2 - link unattached at the moment 
o Official Video - link unattached at the moment 


<!--
OPEN ITEMS — resolve before cutting targeted CVs from this file

1. University dates conflict:
   - This file: Physics 2019–2021, Statistics 2021–2025.
   - Hospitality CV: Physics 2017–2019.
   - Your project list says the 2020 event study was your "first year as a mathematics student",
     which fits Statistics starting 2019, not 2021.
2. Student-role titles at LD Pensions, Lån & Spar and Statistics Denmark: confirm against contracts or references.
3. TCR live trading: confirm which strategy is live, the start date and results (fees included) before quoting any of it.
4. Lectures table: 2026-06-09 full title, 2026-07-14 organiser, 2026-09-13 date (a Sunday).
5. LD Pensions / Statistics Denmark: answer the prompts under each section to add scale and outcomes.

Resolved in this update:
- Konecta title → Customer Service Representative (your edit).
- Prosperity 4 → 741 of 6,500 active teams.
- GitHub → all project links now point to github.com/MarkBrezina.
- Statistics degree → shown as "not completed" (same meaning as "dropped out", reads more neutrally).

Kept out on purpose:
- WTECH incorporation / funding / recruitment claims, and the "13% short-term return".
- Notes about unpaid clients, unrecorded overtime and managers' appreciation.
- The mortality-modelling R files (Lee–Carter, CBD, P-splines) in R_programs.zip — they are companion code from
  Macdonald, Richards & Currie's textbook, not your own work.
- Costa-specific profile text from the service CVs.
-->
