<!--
MASTER CV — Mark Brezina
This is the long-form reference record. Cut targeted CVs from it; don't send it as-is.
Hidden notes like this one don't render on GitHub but are visible in the raw file.
Open items are collected at the bottom.
Last cleaned: 2026-11-05
-->

# Mark Brezina

London, UK · (+44) 7761 917 490 · [mark@brezina.dk](mailto:mark@brezina.dk) · [LinkedIn](https://www.linkedin.com/in/markbrezina95) · [GitHub](https://github.com/MarkBrezina)

UK right to work (partner visa, no sponsorship required)

<!-- Full address used on applications: 6 Nuding Close, London SE13 7UD. Kept out of the rendered page in case the repo is public. -->

---

## Profile

Quantitative developer and analyst with experience across pension-fund risk analytics, credit-risk data quality at a bank, public-sector business intelligence and contact-centre reporting automation. I build data pipelines, risk and pricing models, and validation tooling in Python, SQL and SAS, and run independent research in systematic trading and market microstructure. Studied Mathematical Statistics and Mathematical Physics at the University of Copenhagen.

---

## Skills

**Programming:** Python (pandas, NumPy, SciPy, scikit-learn, PyTorch/TensorFlow), SQL, SAS (incl. macros), R, MATLAB, VBA / VBScript, JavaScript (Google Apps Script), C++, C#, HTML/CSS, Lua, Pine Script

**Data and platforms:** ETL pipeline design, Power BI, Excel (add-ins, automation), Google Sheets / Slides / Forms automation, Azure, Databricks, Supabase, Bloomberg, MSCI, Sustainalytics, AS400 / IBM i

**Quantitative methods:** Probability and statistical inference, stochastic processes (GBM, Ornstein–Uhlenbeck, square-root diffusion), Monte Carlo and MCMC, time series (ARIMA/GARCH), Hidden Markov Models, Bayesian modelling, VaR/CVaR and stress testing, option pricing with stochastic volatility (Heston), limit-order-book analytics

**Practice:** Data quality and reconciliation, model and performance validation, test design, KPI design, stakeholder and client reporting, teaching

---

## Experience

### Konecta — Customer Service Representative, Danish Market (Center Parcs account)
**London · May 2026 – Present**
*Management-information and reporting development alongside the service representative role* \
**Representative role**
- Handle inbound sales and service for the Danish market, plus Foreign Markets(English speakers) overflow and outbound welcome calls, on the AS400 / IBM-i booking system and the PVCP booking platform(Polo system).
- Highest average booking retention in the Danish queue, 100% of my bookings stay, sales conversion above the European benchmark, and service handling time around one minute faster than the campaign standard.
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

*Tools: Google Apps Script (JavaScript), Google Sheets / Slides / Forms, Python, AS400 / IBM-i (5250), data reconciliation and QA, KPI design, Bayesian modelling*

---

### Tomorrow Capital Research — Quantitative Developer & Researcher
**Remote · January 2024 – Present**
*Independent research venture developing systematic trading strategies and infrastructure.*

**Research**
- **Market-state detection:** custom state classifiers, Hidden Markov Models, decision-tree frameworks, autocorrelation analysis and stationarity tests.
- **Trend and reversion analysis:** Markov-chain trend detection, local reversion triggers, resistance-level identification and regression methods.
- **Strategy selection:** prototype engine that switches between trend-following, mean-reversion and event-driven reversion strategies based on current market conditions.

**Systems**
- **Backtesting:** framework for testing strategies against historical data and simulated environments.
- **Forecasting:** predictive models using MCMC, Monte Carlo simulation and ARIMA/GARCH processes.
- **Data pipelines:** ingestion from multiple APIs and databases covering price feeds, order-book depth, options chains, macroeconomic data and news sentiment.
- **Architecture:** progressed from prototype to faster, more general versions in Python, SQL and C++, with separate modules for risk monitoring, PnL tracking and on-demand backtesting; connectivity across multiple exchanges and asset classes (crypto, equities).

*Tools: Python, SQL, C++, Monte Carlo methods, HMMs, ARIMA/GARCH, OOP, modular architecture*

*Note: this needs SIGNIFICANTLY MORE DATA*


---

### Freelance Software Developer & Quantitative Consultant
**Self-employed · Remote · May 2024 – Present**
- **Sports betting research (Oct – Nov 2025):** Designed probabilistic models for European football markets (1X2, goals and more) using historical performance, league effects and matchup dynamics. Modelled micro-events (shots, shots on target, corners, cards, fouls) with Poisson, negative binomial, Gamma, Normal and empirical distributions and conditional probabilities. Built data intake for weather, line-ups, historical and player data, with a correlated model setup and bet-ordering across multiple bookmakers.
- **Strategy performance validation (Sep – Oct 2025):** Validated trading-book data for price integrity, transaction costs and execution consistency; recalculated CAGR, absolute and cumulative returns, volatility and Sharpe ratio from raw trades; documented methodological issues in backtesting assumptions and execution logic.
- **Low-latency crypto trading (Dec 2024 – Feb 2025):** Built a low-latency system for clients trading newly listed tokens on the Raydium market. *Note: This was done back in Denmark with the pump.fun website, I have later found that the website isn't available in the UK, for good reasons, but I have since discovered that this particular Alpha can be utilised in other ways and have done much more research on it.*
- **Algorithmic scripting (May – Jul 2024):** Pine Script strategy on TradingView triggering trades on custom signals (session-time setups, exchange opening times, multi-day mean reversion).
- **Web and game development (July 2026 - September 2026):** For a colleague I setup a free website and webdevelopment of an app, a game involving clues and puzzle solutions for romance crime solving, the website and link is https://www.clubclue.com/ The inspiration was taken from Murdle, but the theme was pivoted towards romance, intrigue and relationship drama.
- **Web development (May – Jul 2024):** Static HTML/CSS/JavaScript portfolio website for a hobby photographer.
- **Excel integrations(Sporadic):** Workbook enhancements in Python, C#, VBA and VBScript — dashboards, financial calculations and inter-application automation.

*Tools: Python, Excel, VBA, C#, Pine Script, HTML/CSS/JavaScript*

---

### LD Pensions — Junior Quantitative Developer
**Copenhagen, Denmark · August 2023 – May 2024**
*Part-time student position, 15–35 hours per week, year-round. Danish pension fund managing assets on behalf of members and the public*
- **Data pipelines:** Automated, scheduled Python ETL ingesting and cleaning data from Nykredit, Nordea, Bloomberg, MSCI, Sustainalytics, Morgan Stanley and Danske Bank into the central database, so analytics were up to date at the start of each workday.
- **Internal Python library ("LDF"):** On-premises package for quantitative analysis, risk modelling and data management, refactored from functional to object-oriented design, with an Excel add-in exposing it to non-technical users.
- **Risk analytics:** Modules for VaR, CVaR, credit, interest-rate, currency and derivatives risk, with emphasis on margin-call frameworks for currency futures and rate-sensitive portfolios.
- **CIO project:** Reverse-engineered the VaR, CVaR and waterfall calculations in Morgan Stanley's profit-forecasting data, built a scenario simulator (geometric Brownian motion, square-root diffusion) and presented findings to senior leadership.
- **Portfolio framework:** Multi-tier hierarchy consolidating ETFs and asset classes into low-, medium- and high-risk mandates, linked across database tables by asset ID; daily return and risk monitoring aggregated from holding to mandate level; allocation-vs-limit, NAV-vs-benchmark and VaR visualisations.
- **ESG analytics:** Weighted Average Carbon Intensity and Total Financed Emissions from MSCI, Bloomberg and Sustainalytics data, plus governance and social KPIs.
- **Productivity:** Python-driven data preparation for Power BI dashboards; VBA macros and Excel add-ins automating budgeting and email workflows; stress-testing tools using quantile returns, drawdown analysis and log returns.

*Tools: Python, SQL, VBA, Power BI, Bloomberg/MSCI APIs, VaR/CVaR, scenario simulation*

Note: I want this work experience to have significantly more information about what work I did while being employed here, as this job was likely one of the positions where I have done the most work mentally, programmaticly, structurally, other than Tomorrow Capital Research.

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
- Based exercises on real tasks — automating routine processes, analysing datasets — and refined material from participant feedback.
*Training: 3-day course in pedagogical presentation*

---

### Statistics Denmark — Data Analyst & Developer
**Copenhagen, Denmark · August 2021 – August 2023**
*Part-time student position, 15–50 hours per week, year-round. Denmark's national statistics authority.*
- **Timesheet and budgeting:** Administered the timesheet and budgeting platform; ran monthly SAS/SQL extractions for budget and salary forecasts, hour estimates and reallocation of borrowed hours; integrated Navision (bookkeeping), Dynamics (CRM) and legacy budget systems into SAS/SQL pipelines.
- **Donations system:** Helped run the donations system (C#, SQL, Excel) and built an API integration with the legacy platform for monitoring income, expenses and project usage.
- **Project finance:** Built an Excel/VBA/SAS/SQL toolset consolidating project data (hours, overhead, CRM and external costs), classifying projects by profitability, owner and expense flow, with automated email reminders to project leads.
- **Executive reporting:** Power BI dashboards on fund allocation and project performance for senior management; ad-hoc analysis of funding gaps, utilisation and returns on major initiatives.
- Took part in code reviews; passed security vetting including a criminal-record check.

*Tools: SAS, SQL, C#, VBA/VBScript, Power BI, Navision, Dynamics*
*Training: SAS Institute programmer course series; internal courses in Statistics with R I & II and Demographic Statistics*

Note: I want this work experience to have significantly more information about what work I did while being employed here, as that similarly, I did A LOT OF WORK here like with Tomorrow capital research and LD Pensions

---

### MentorDenmark — Senior Mentor, Physics, Mathematics & Science
**Copenhagen, Denmark · August 2017 – August 2021**
*Denmark's largest tutoring company.*
- Top-5 senior mentor: over 1,000 hours tutoring mathematics, physics and science to students aged 7–23, with an average satisfaction rating of 4.7/5.
- Prepared more than 50 students for mathematics exams through curriculum review, mock exams and simulated examiner scenarios.

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
| Sep 2020 – Mar 2021 | Home and school cleaner | Self-employed, Copenhagen |
| Aug 2021 – Aug 2022 | Store clerk, Copenhagen Airport | WHSmith |
| Apr 2019 – Aug 2020 | Store clerk (incl. forklift and pallet-jack operation) | Lidl, Copenhagen |
| Oct 2016 – Dec 2018 | Night-shift store manager, supervising two staff | MinKøbmand, Copenhagen |

*Training: Psychological first aid for trauma and shock, chemical safety, track safety (DSB); Copenhagen Airport Security Awareness (WHSmith)*

---

## Education

### BSc Mathematical Statistics — University of Copenhagen
**2021 – 2025 (Dropped out)**
- Bachelor project: *Option Pricing with Stochastic Volatility in the Heston Model* — R/C++ package for Heston pricing and volatility-surface work, developed with a company that now uses it commercially.
- First-year project: *Event Study in the Stock Market* (Apple's announced acquisition of AuthenTec).
- Relevant courses: Mathematical Statistics, Stochastic Processes I & II, Advanced Probability Theory, Life Insurance, General Insurance, Differential Geometry.

### BSc Mathematical Physics — University of Copenhagen
**2019 – 2021 (withdrew)**
- First-year project: *Estimating Dark Matter Density from Spiral Galaxy Rotation Curves*.
- Relevant courses: Mechanics I & II, Thermodynamics, Electrodynamics, Quantum Mechanics I & II, Galactic Astrophysics, Complex Analysis I & II.

### Upper secondary (STX) — KNORD Lyngby
Physics A, Mathematics A, English A, Danish A, Chemistry B.

---

## Projects

### Ctrl+Alt+DefeatTheMarket — IMC Prosperity unofficial guide
*Python, pandas, NumPy, SciPy, networkx, matplotlib, plotly*

Research and trading framework for high-frequency market microstructure and systematic strategy design, built around the IMC Prosperity competition.
- **Microstructure:** order-book features (imbalance, order flow, VWAP, microprice, liquidity-weighted fair value); decomposition of price moves into order-flow, drift and mean-reversion components.
- **Strategies:** market-making, mean-reversion and drift-aware strategies under inventory and spread constraints; hybrid execution combining passive quoting with aggressive taking on mispricing; multi-signal blending to adjust quotes and inventory targets.
- **Testing:** tick-level backtester; robustness checks (detrending, z-scores, rolling regression forecasts); scenario tests across volatility, spread and liquidity regimes.
- **Execution and risk:** inventory-aware quoting with dynamic skew; trade classification, order placement and fill simulation.

### TMRW — algorithmic trading and AI framework
**2019 – Present** · [Repository](https://github.com/CopenhagenToLondon/TMRW)

End-to-end research and trading framework and personal Python library: data extraction and ingestion (Yahoo Finance, Binance, Bloomberg, MSCI, Sustainalytics), backtesting with configurable signals (mean reversion, trend-following, pairs trading), Monte Carlo and scenario stress testing, VaR/CVaR stops and dynamic position sizing. Research into Markov-chain models, deep learning for short-horizon forecasts, mean-field games and adaptive reinforcement-learning agents.

<!-- The "13% short-term return on real-time deployment" claim was removed. Add it back only with period, capital base, costs and whether gross or net. -->

### Algorithmic trading research
**2022 – Present** · [Repository](https://github.com/CopenhagenToLondon/WTech)

Development and evaluation of trading algorithms — mean reversion, momentum and pattern detection — using stochastic processes, time-series analysis and statistical optimisation.

### GoldenDice — stochastic simulation package
**2022 – Present** · [Repository](https://github.com/CopenhagenToLondon/GoldenDice)

Classes and scripts for simulating stochastic processes in finance (Brownian motion, square-root diffusion, Ornstein–Uhlenbeck) with Monte Carlo path generation for pricing, risk analysis, scenario testing and sensitivity analysis.

### Galaksim — galaxy rotation simulations
**2018 – 2025** · [Repository](https://github.com/CtoL95/Galaksim)

Multi-particle simulation of spiral-galaxy rotational dynamics, exploring rotation curves, mass distribution and emergent spiral structure.

### Knowledgebase — mathematics and finance resources
**2023 – Present** · [Repository](https://github.com/CtoL95/Knowledgebase)

Curated study material in finance and mathematics — concept overviews, worked problems and references — intended to grow into a book.

<!-- Not yet listed (no public material or still in progress):
- AI for Trading (github.com/CopenhagenToLondon/CONTROL) — repo empty
- ETL package (Yahoo, Alpha Vantage, Binance, Jupiter) — link pending
- Tomorrow Markets, Tomorrow Quant Archive website
- Wunder Fund Alpha Connectome competition entry -->

---

## Competitions and Recognition

- **IMC Prosperity 3** algorithmic trading challenge — rank 107 of 12,000+ teams (top 1%)
- **IMC Prosperity 4** algorithmic trading challenge — rank 988 (top 15%)
- **Børsen stock-picking challenge** — rank 468 of ~47,000 (2024, top 1%); rank 2,800 of 36,000+ (2023, top 10%); rank 1,800 of ~28,500 (2026, top 7%)
- **Design for Change** awardee (2010) — international competition for innovative solutions to everyday problems
- **MENSA** member since 2018
- LinkedIn skill assessments passed: R, Python, VBA, Excel, MATLAB
- StarCraft II: top 5% of players; active chess player
- Steam marketplace trading: grew €100 to ~€300 over three years trading Counter-Strike cases

<!-- CHECK: Prosperity 4 field size — notes said "6,500 (22,000?)". 988/6,500 ≈ top 15%; 988/22,000 ≈ top 5%.
Børsen 2026: 1,800/28,500 = 6.3%, rounded to "top 7%". -->

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

<!-- TODO: add events attended between Dec 2025 and now that are missing, and anything after May 2026. -->

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

<!-- Add any others (Czech, Swedish, Dutch?) with honest levels. -->

---

<!--
OPEN ITEMS — resolve before cutting targeted CVs from this file

1. Konecta job title: confirm against contract (Advisor / Sales Advisor / Representative).
2. Student-role titles at LD Pensions, Lån & Spar and Statistics Denmark: confirm against contracts or references.
3. University dates conflict:
   - Master CV: Physics 2019–2021, Statistics 2021–2025.
   - Hospitality CV: Physics 2017–2019.
   - Projects list says the 2020 event study was your "first year as a mathematics student",
     which fits Statistics starting 2019, not 2021.
   - Also confirm whether the Heston work is the bachelor project and whether "on hold" is still accurate.
4. Galaksim: listed both as "2018–Present (ongoing)" and "2018–2025 (finished)". Used 2018–2025.
5. GitHub: header points to MarkBrezina; project links point to CopenhagenToLondon and CtoL95.
   Consider transferring or forking the repos so one account holds everything.
6. DSB: master CV said "2024–Now"; service CVs say May 2024 – Jan 2025. Used the latter.
7. Prosperity 4 field size (see Competitions).
8. Freelance start: master said 2023, later version said May 2024. Used May 2024.

Removed from earlier drafts:
- WTECH entry (incorporation, funding, recruitment) — merged into Tomorrow Capital Research.
- "13% short-term return" — needs period, capital base and cost basis.
- Notes about clients not paying, unrecorded overtime, and managers' appreciation.
- Konecta portfolio write-up and README — keep these as separate files (e.g. /portfolio/konecta.md).
- Costa-specific profile text from the service CVs.
-->
