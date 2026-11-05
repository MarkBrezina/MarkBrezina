# Reinforcement learning and stochastic control

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The seminar notes move from finite-state Bellman control to continuous-time and non-Markovian models, then to exploration, learning from logged actions, and hierarchical multi-agent control. The Danish shorthand has been translated and consolidated.

## State, action, and reward

| Element | Trading example | Research question |
|---|---|---|
| **Observation** | Time, prices, recent order-book events, book depth, fills, pending orders, inventory, and volatility. | Which variables are observable at the decision, and what information is lost in the chosen state? |
| **State** | A finite rolling feature vector, a latent-regime belief, or an encoding of an observed path. | Does the last seven frames hypothesis give a sufficient approximation for this action and horizon? |
| **Action** | Quote placement and size, cancel, take, wait, hedge, or set a target. | Which actions can be executed under the venue, inventory, and latency rules? |
| **Reward** | Equity change after fees, with an explicit inventory or risk penalty if desired. | How much of the result comes from execution, exposure, or reward shaping? |
| **Transition** | Arrival of market data or a fill, a timed decision, or a jump in regime. | Does the simulator reproduce action-dependent fills and dynamics? |

A policy optimised with knowledge of the realised future path is a hindsight benchmark. For a live-feasible comparison, give all policies the same information at each timestamp. Restrict the first experiment to a small finite quote grid and a position cap. A dynamic-programming solution on that grid and a hand-written skew rule provide benchmarks for a learned policy. See [market making](02-market-making-and-microstructure.md).

## Learning routes

- **Model-based.** Fit state transitions, order-arrival intensities, or a neural simulator. Use the model to plan or solve a control problem. Validate path statistics and action-conditioned fills separately from policy performance.
- **Model-free.** Use Q-learning, PPO, or actor–critic methods in a simulator with logged action and fill rules. Keep the same observation, constraints, and reward accounting as the baseline. Check whether training depends on unrealistic information or fill assumptions.
- **Continuous-time control.** Specify whether actions are continuous, updated at events, or approximated on a time grid. The source's “little q” refers to a continuous-time analogue studied in the cited paper. HJB equations and stochastic maximum principles are mathematical leads, with the relationship depending on the model. The source also records a regret-rate fragment $N^{3/4}$ with its theorem and assumptions still to identify. [Jia and Zhou, *q-Learning in Continuous Time*](https://arxiv.org/abs/2207.00713)
- **History-dependent control.** A path signature or neural rough differential equation can encode ordered observations. Signature regression, decision transformers, and recurrent or feedforward baselines give different ways to test whether the path adds useful information beyond a fixed-window state. [Neural RDE paper](https://arxiv.org/abs/2306.14258)
- **Event models.** The notes propose self-exciting or neural Hawkes processes for clustered order arrivals. Test whether event intensities improve simulation of spread, fills, cancellations, and bursts before embedding them in a decision policy.

## Exploration and evaluation

A greedy policy repeatedly chooses the action with the largest estimated value. Exploration varies choices to learn about other actions. Entropy regularisation changes the reward or policy objective to encourage a distribution over actions. The notes mention Rényi and Tsallis entropy as alternatives to investigate. Compare them at matched cost and risk budgets, with explicit exploration in simulation. Keep reward shaping visible, including terminal inventory and any penalties for taking or quote churn.

Counterfactual learning asks what might have happened under a different action. Historical logs directly reveal only what followed the action taken. An action-conditioned simulator or off-policy estimator needs adequate action support and an account of selection effects. Evaluate robustness to changes in the fill model, latency, costs, and market responses. Logged counterfactual claims should state which effects are observed, modelled, or unidentified.

Hierarchical multi-agent reinforcement learning is a possible extension. A slow allocator could send budgets or target inventory to a fast execution policy, while a shared ledger records joint outcomes. **Credit assignment** asks which action or layer contributed to those outcomes. Compare the combined policy against simpler fixed-budget and rule-based controls, and ablate each layer while holding the others constant. The full hybrid-market existence problem is set out in [Conjecture 4](05-tomorrow-conjectures.md).
