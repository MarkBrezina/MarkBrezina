# Rates, volatility, and practitioner questions

[Research index](../README.md) · [Source map](../SOURCE_MAP.md)

The source ends several threads with questions and formula fragments. This page preserves the questions and makes the rate conventions explicit. The original heading for illiquid bonds contains little accompanying text, so the questions in that subsection are proposed research prompts.

## Discount factors, zero yields, and forwards

Let $D(t,T)$ be the time-$t$ price of a unit payment at $T$. A continuously compounded zero yield $y(t,T)$ gives $D(t,T)=\exp[-(T-t)y(t,T)]$. For $t<T_1<T_2$ with accrual fraction $\delta>0$, the simple forward rate is

$$
F_{\mathrm{simple}}(t,T_1,T_2)
=\frac{D(t,T_1)/D(t,T_2)-1}{\delta}.
$$

With $t=0$, $T_1=1$, $T_2=2$, and $\delta=1$, the source example becomes $\exp[-y(0,1)+2y(0,2)]-1$. The subtraction is needed after converting the discount-factor ratio to exponential form. If the discount factor rises from $T_1$ to $T_2$, the ratio is below one and the corresponding simple forward is negative.

With the same time convention, the continuously compounded forward yield over $[T_1,T_2]$ is

$$
\frac{(T_2-t)y(t,T_2)-(T_1-t)y(t,T_1)}{T_2-T_1}.
$$

If the continuous zero curve rises strictly across the two maturities, this forward yield is above the longer zero yield. For a differentiable curve the instantaneous forward is $-\partial_T\log D(t,T)=y(t,T)+(T-t)\partial_T y(t,T)$. It exceeds the zero yield where the zero curve has positive slope.

A simple-compounded spot curve uses $D(t,T)=1/[1+(T-t)L(t,T)]$ in the same year-fraction convention. If $L(t,T_1)>L(t,T_2)$, both have the same nonzero sign, and both discount factors are positive, the formula gives a simple forward below the longer simple spot rate. The direction of comparisons between rates depends on the compounding convention and accrual periods.

A money-market account with constant continuously compounded rate $r$ grows an initial amount $P$ to $P e^{rt}$. Over a single simple-interest accrual period $\delta$, the amount is $P(1+r_{\mathrm{simple}}\delta)$. For a time-varying instantaneous rate, the continuous account uses the integral of the rate over time.

## Volatility and trend-following questions

The source records four questions for Artur Sepp, alongside a CTA trend-following reading lead and a note that FX autocorrelation might help detect mean reversion. The latter is a hypothesis to test by currency, sampling interval, and period.

| Question from the notes | A concrete comparison |
|---|---|
| How do short-lived volatility spikes differ from structural shifts? | Compare rolling volatility, a change-point or state model, and event-conditioned rules by detection delay, false alarms, and subsequent risk. |
| When do macro and cross-asset features improve longer-horizon volatility forecasts? | Add point-in-time macro and related-market features to a model that already uses recent realised volatility. Separate short and long forecast horizons. |
| Which models forecast volatility robustly across scales? | Compare HAR and GARCH with a machine-learning or graph-based model on the same realised-volatility target, loss function, and splits. |
| How should inventory and risk budgets respond to extreme volatility? | Replay sizing and hedge policies using realised-risk measures, historical stresses, and forward-looking inputs such as implied skew where available. Measure costs and realised downside. |

The CTA reference is carried forward as a reading lead. [Sepp, The Science and Practice of Trend-following Systems](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3167787)

A separate question asks for Laura Ballotta's work on multivariate additive subordination. The institutional record is a starting point for studying joint dynamics and dependence through stochastic time changes. [*Multivariate Additive Subordination with Applications in Finance*](https://openaccess.city.ac.uk/id/eprint/33833/)

## Illiquid bonds

The notes identify illiquid bonds as a topic. Three proposed questions connect it to the rest of the archive.

1. **Fair value and observations.** Which quotes, transactions, issuer comparables, and curve marks are available at each time? How much of a measured return comes from stale marks updating?
2. **Risk factors.** How do interest rates, issuer spreads, liquidity and optionality contribute to changes in value? Which sensitivities can be checked with scenario revaluation?
3. **Execution and confidence.** How does executable size change the bid or offer? Can a central price model report a wider uncertainty region when trade evidence is sparse?

These questions connect the [central pricing conjecture](05-tomorrow-conjectures.md) with the [statistical-arbitrage data and risk framework](03-stat-arb-trend-and-regimes.md).
