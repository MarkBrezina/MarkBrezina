# Tomorrow Capital Research conjectures

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The source contains working statements and public-facing drafts of five conjectures. Each asks a different question about a common market and trading system. The formulations below preserve the research claims as conjectures and give explicit starting models for experiments or proofs.

| Conjecture | Decision or object | Evidence or mathematical result sought |
|---|---|---|
| **C1. Dynamic reallocation under path switching** | Move a legacy position toward a new target after a model update. | A feasible, cost-aware transition across future scenarios. |
| **C2. The central pricing problem** | Estimate the next central price and the distribution of future paths for interacting assets. | Incremental predictive accuracy, calibrated joint prediction regions, and a credible treatment of own impact. |
| **C3. Stealth-optimal execution distributions** | Schedule an order across time, size, and eligible routes. | A defined trade-off between cost, risk, and information leakage, plus existence or construction under specified assumptions. |
| **C4. Existence of optimal strategies in the full model** | Choose a policy or equilibrium in a hybrid, interacting system. The longer draft also calls this “Conjecture X.” | Precise conditions for existence, a constructive characterisation, or an equilibrium result. |
| **C5. The Final Problem** | Allocate capital, risk limits, positions, and execution across a hierarchy. | Conditions under which locally optimal decisions implement a firm-wide optimum. |

## C1. Dynamic reallocation under path switching

At the start, a model $\mathcal M_1$ generates price paths and a policy $\pi_1$ is selected using a stated criterion. An event or update at $T_2$ changes the model to $\mathcal M_2$ and the target strategy to $\pi_2$. Re-simulate from the **realised** state at $T_2$, including the current price, holdings, and any pending orders. The old strategy has already left inventory behind. The execution problem is to get from that inventory to the new target by an intended completion time $T_3$.

The source proposes blending the two position paths via

$$
\theta_t=(1-w_t)\theta_t^{(1)}+w_t\theta_t^{(2)},\qquad 0\le w_t\le 1.
$$

A natural first objective trades off deviation from the new target against turnover, trading cost, and risk along the path. A policy can update $w_t$ or the target position as new information arrives. Decide whether intermediate paths must stay in the convex hull of the two policies, whether $w_t$ must increase, and what happens if the deadline conflicts with hard risk or liquidity constraints. The initial trade must begin from the actual portfolio, including partial executions.

**Proposed comparison.** Replay immediate switching, a fixed linear ramp, a wait-and-switch rule, and rolling cost-aware optimisation using the same shocks and constraints. Measure net P&L, time to reach the new target, maximum intermediate exposure, deadline shortfall, and turnover. A separate stress test reverses the regime signal before the first transition completes.

## C2. The central pricing problem

The draft considers $N>3$ interacting instruments, potentially including underlyings, options, and swaps. It separates two drivers of the next price. One is **immediate**, using book state, flow, liquidity, and impact. The other is **historical or structural**, using past prices, regimes, cross-asset relations, fundamentals, and drift or reversion. The relative importance can differ by asset and time.

One representation of the proposal is

$$
\widehat P_{i,t+1}=\alpha_{i,t}\,g_i(X_t)
+(1-\alpha_{i,t})\,h_i(Z_t),\qquad 0\le\alpha_{i,t}\le1.
$$

Choose a target first, such as the next executable midpoint, a future trade price, or a latent fair value with a measurement model. The source's proposed information set includes full history, fundamentals, and aggregate participant positions. A feasible test needs observable proxies for aggregate positioning and a record of when each feature became available. The draft also gives a toy square-root own-impact update that scales a quote-to-mid difference by order size. A calibrated version needs a reference volume, signed order direction, time scale, and separate treatment of transient and persistent impact.

The second deliverable is a prediction region for the **joint future price path** across assets at horizons of at least five steps. Coverage of individual prices is easier to achieve than simultaneous path coverage. Report both, along with interval width, accuracy conditional on regime, and performance relative to a persistence forecast. Compare immediate-only, history-only, and blended predictors. For derivatives, test consistency across linked prices under their contract and carry conventions.

## C3. Stealth-optimal execution distributions

A parent order of size $Q$ is prescribed by an alpha with a deadline $\tau$. The source uses an illustrative notional of 1 million USD and a toy upper bound of 100 allocation channels or accounts. Choose times $t_k\in[0,\tau]$ and sizes $q_k\ge 0$, with $\sum q_k=Q$ for a completed order. A continuous model instead chooses an execution rate $\phi_t$ with integral $Q$. Any applied routing formulation would use eligible routes with their actual access, participation, and capacity constraints.

The drafts identify several separate mechanisms.

- **Impact.** Own flow can change prices and execution cost. A square-root-shaped response is one candidate and needs calibration in volume and price units. If a price impact per unit scales as $\sqrt{q}$, total cost for a slice scales differently from the price response.
- **Detectability.** Other participants may infer a parent order's direction or size from child trades. Measure this as a change in a specified observer's ability to classify or forecast the order, controlling for background activity. The draft's “memory” is a hypothesis about how signals decay over time.
- **Costs and schedule constraints.** Compare proportional fees, fixed costs, liquidity, participation bounds, and alpha decay while waiting. Opposing flow or cross-impact can be modelled, with any offsets estimated from data.
- **Fragmentation.** A limit on routes and minimum trade size can remove degenerate scheduling choices. The source also speculates about identity-based resets. For a public research prototype, represent memory through observable flow and a decay kernel, and test whether venue or time fragmentation actually changes the footprint.

A tractable first problem minimises expected implementation shortfall plus penalties for price risk and measured information leakage. Benchmark against immediate execution, TWAP, VWAP or volume participation, and a cost-risk scheduler. Include the opportunity cost of unfinished quantity at the deadline and compare equal realised completion rates. Report a frontier between execution cost and detectability if the two goals conflict. Existence, uniqueness, and constructive algorithms depend on the chosen objective and feasible set. [Almgren and Chriss, *Optimal Execution of Portfolio Transactions*](https://doi.org/10.21314/JOR.2001.041)

The source credits Lorenzo M. Zorzi Chiarioni with a response to the initial formulation. The version and assumptions covered by that response remain to be identified.

## C4. Optimal policies in a hybrid, interacting market

The research model combines an asset graph, an information graph of agents, price or value fields, and a hierarchy of rules. A continuous state on a manifold $\mathcal M$ evolves within modes $s\in\mathcal S$, with jumps between modes or states. Agents can have different objectives and information. Asset prices and observations can respond to collective decisions.

The simplest benchmark is a single agent in a finite, fully observed finite-horizon MDP. Backward induction chooses a feasible action using immediate reward and the value of its continuation. In the richer formulation, an admissible policy $\pi$ seeks to attain the supremum of expected discounted rewards less losses. The source asks when an optimal policy can be taken in feedback form, whether a geometric or variational characterisation is useful, and whether a convergent algorithm can compute it.

| Extension | Research requirement |
|---|---|
| Continuous state with regime jumps | Specify well-posed dynamics, jump maps, action sets, reward integrability, and the conditions needed to select an optimal control. |
| Partial information | Identify the information state or belief used by an admissible policy. Test whether a fixed history window retains relevant state. |
| Constructive solution | Relate the dynamic programme or hybrid HJB equation to the candidate solution, then establish convergence under the stated assumptions. |
| Interacting agents | Specify how each policy affects prices and transitions. Distinguish finite-agent Nash equilibrium from a mean-field limit. |
| Mean-field formulation | Couple the control or value equation with the evolution of the distribution of agents, including consistent jump and boundary conditions. |

A feedback policy, a least-action or geodesic interpretation, and a solution to a coupled HJB–Fokker–Planck system are different proposed mathematical deliverables. The notes give a finite-state benchmark and leave the full existence question open. The [control notes](06-reinforcement-learning-and-control.md) catalogue methods for restricted versions of this problem.

## C5. The Final Problem

Fix a firm-wide portfolio with total capital $V$. Its sub-books include strategy pods, hedging and risk books, and order execution. Each controls local holdings, cash, or exposure and may optimise P&L, risk-adjusted return, tracking, or implementation cost. The firm evaluates shared outcomes such as net return, drawdown, survival, capital efficiency, and systemic exposure. Decisions are coupled through the same assets, funds, liquidity, and market feedback.

The conjecture asks for an implementable multiscale rule that makes the configuration optimal at firm level **and** locally optimal at each layer given its information and constraints. Arbitrary local objectives can disagree with the global objective. The tractable question is what allocation, constraint, or shadow-price mechanism aligns them.

**Proposed first model.** Give each desk a concave local objective $g_i(x_i)$ and a resource demand $A_ix_i$ under a firm budget $\sum_i A_ix_i\le b$. Under suitable convex feasibility and duality conditions, a common resource price $\lambda$ lets desks optimise $g_i(x_i)-\lambda^\top A_ix_i$. Compare the resulting allocations, with feasibility and complementary slackness checked, against the central optimiser. Then introduce cross-book risk and netted execution one at a time. Shared covariance, endogenous prices, private observations, and nonlinear feedback raise questions absent from the separable model.

The source also frames expected utility as combining return, risk, costs, and groupings, using “free energy” and a variational principle as analogies. A mathematical version needs a defined state, units, objective, information flow, and stability criterion. The competition provides small analogues of the programme. These include fair value for C2, inventory or strategy transitions for C1, scheduling for C3, a simulator for testing restricted C4 policies, and shared limits for C5.
