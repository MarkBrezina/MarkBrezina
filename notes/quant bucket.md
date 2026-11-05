The final problem.

	- Action : Highest expected utility
	- Utility takes into account cost and groupings
	- Minimize free energy(risk)
	- Maximumize conserved energy(return)
	- Certain properties to play dynamics

Variational principle???


Layer	Time horizon	Strategy		
Ultra-low-latency	µs -> ms	Market-making (spread capture, order-book imbalance)		
		Latency arbitrage
		Basket arbitrage
		Cross-exchange arbitrage
		
Low-latency	ms -> s	Event driven/news		
		Micro-momentum/burst breakout
		Gamma-scalping/delta hedges
Medium-latency	Minutes -> hours	Mean reversion		
		Pairs/statistical arbitrage
		Event driven/news
		
		VWAP/TWAP execution algos
Long-latency	Days -> years	Trend-following CTAs - long/short for trends		
		FX carry
		Vol-risk-premium
Ultra-long-latency	Months -> xyz	Factor investing		
		Risk-parity/All-weather
		Macro / Private deals
		holding
Layer 1: High-Frequency Trading (HFT)
• Time Frame: Ultra-low frequency (seconds to minutes)
• Strategies: Market-making, statistical arbitrage
• Position Goal: 0 (neutral position)
• Objective: Capture small, frequent profits while minimizing risk
• Key Considerations:
	• Order book dynamics
	• Liquidity provision
	• Risk management
1. Layer 1 (HFT)
	○ Time Horizon: Sub-seconds to a few seconds/minutes.
	○ Objectives:
		§ Market-making; capturing bid–ask spread.
		§ Statistical arbitrage with a near-zero net position target.
		§ Fine-grained order-execution tactics, exploiting microstructure.
	○ Key Traits:
		§ Extremely fast decision-making.
		§ Position goal ≈ 0 at all times to minimize overnight risk.
		§ Focus on liquidity provision and capturing small, repeated profits.


Layer 1: Ultra-High Frequency (HFT)
Objective: Neutralize inventory risk (position goal ≈ 0).
Time Frame: Milliseconds to seconds.
Key Strategies:
	1. Market-Making:
		○ MFG Integration: Model competitor market-makers’ behavior to optimize bid-ask spreads and queue positions.
		○ Execution: Use latency-optimized order placement (e.g., predictive cancellations, sniping).
	2. Statistical Arbitrage:
		○ Exploit microstructural mispricings (e.g., ETF vs. constituents, cross-exchange spreads).
		○ Risk: Dynamic inventory control using reinforcement learning (RL) to avoid adverse selection.
	3. Liquidity Detection:
		○ Identify hidden liquidity or iceberg orders via order-book imbalance signals.
Risk & Execution:
	• No overnight exposure: Auto-liquidate residual positions.
	• Hard Circuit Breakers: Kill switch for latency spikes or abnormal volatility.

HFT Layer (Ultra-low Frequency, Position Goal = 0)
Focus on market-making and statistical arbitrage, using mean field games to model how traders interact, particularly in quoting bid-ask prices to profit from spreads. Execution speed is critical, emphasizing profit-taking rather than predictions, given the millisecond time frames.

Layer 1 - HFT (Ultra-low Frequency, Position Goal = 0)
This layer focuses on market-making and statistical arbitrage, aiming for a market-neutral position over time, as indicated by the position goal of 0. Market-making involves continuously quoting bid and ask prices to profit from the spread, requiring low-latency execution systems to update quotes rapidly as market conditions change, as noted in the XTB article on HFT (The Two Faces of a High Frequency Trade | XTB). Statistical arbitrage, particularly pairs trading, identifies correlated securities, creates a mean-reverting spread, and trades when the spread deviates from its mean, as described in the arXiv paper on advanced statistical arbitrage (Advanced Statistical Arbitrage with Reinforcement Learning).
Mean field games are particularly apt here, modeling how HFTs interact with other market participants. The arXiv paper "Mean Field Game of High-Frequency Anticipatory Trading" (Mean Field Game of High-Frequency Anticipatory Trading) shows HFTs supply liquidity based on inventory aversion, lowering costs for large traders if temporary market impact is significant. Given the user's note to run only "taking profit" on HFT, this layer prioritizes execution efficiency, using quantitative methods like order flow imbalance (from ScienceDirect, High-frequency Statistical Arbitrage Strategy Based on Stationarized Order Flow Imbalance) rather than predictive modeling, due to the ultra-fast nature where latency negates forecasting benefits.
Risk Measurements and Execution: Risk focuses on execution risk and slippage, with strategies designed to minimize adverse selection by better-informed traders, as per the SIAM Journal article (Buy Low, Sell High: A High Frequency Trading Perspective | SIAM Journal on Financial Mathematics). Execution methodology is critical, with the user noting that immaculate execution can profit even with basic ideas, such as pushing bid prices up to sell higher, aligning with HFT's speed advantage.
Human Behavior: Given the short time frames, human behavior is less relevant, but flash crashes (mentioned in Wikipedia) suggest HFT can amplify market reactions, requiring robust risk controls.


4.1 Layer 1 (HFT)
	1. Data Infrastructure
		○ Subscribe to Level II order book data with sub-millisecond latency.
		○ Build an in-memory order book for real-time analysis.
	2. Microstructure Analytics
		○ Market-Making: Spread capture algorithms; “pinging” liquidity.
		○ Stat Arb: Pairs/triples trades across correlated instruments (e.g., index futures vs. underlying stocks).
	3. Risk & Execution
		○ Strict position limits to end every day with net zero or near-zero.
		○ Latency-optimized execution (co-location, direct market access).




Layer 2: Short-Term (Minutes to Days)
Objective: Capture mean-reversion/volatility (position goal = ±x).
Time Frame: Minutes to 3 days.
Key Strategies:
	1. Mean-Reversion:
		○ Pairs trading (cointegrated assets), oversold/bought RSI signals.
		○ MFG Integration: Model herd behavior during momentum spikes.
	2. Volatility Arbitrage:
		○ Trade implied vs. realized volatility gaps (e.g., gamma scalping).
		○ Exploit earnings announcements or macroeconomic news.
	3. Sentiment-Driven Fades:
		○ Use NLP on news/social media to fade extreme sentiment (FOMO/panic).
Risk & Execution:
	• Stop-Losses: Time-decaying thresholds (e.g., wider initially, tighter near expiry).
	• Position Sizing: Inverse volatility scaling.
	• AI Integration: LSTM/Transformer models for intraday price forecasts.

Layer 2: Short-Term Trading
• Time Frame: Short-term (a few minutes to a few days)
• Strategies: Mean-reversion, volatility trading
• Position Goal: +- x (target position size)
• Objective: Capitalize on short-term market fluctuations
• Key Considerations:
	• Technical analysis
	• Market sentiment
	• Risk-reward optimization
1. Layer 2 (Short-Term)
	○ Time Horizon: A few minutes up to a few days.
	○ Objectives:
		§ Mean reversion.
		§ Short-term volatility trading (e.g., trading around expected earnings announcements, news catalysts, or technical signals).
	○ Key Traits:
		§ Position goal ≠ 0, but still relatively small.
		§ Higher frequency than long-horizon, but less than HFT.
		§ Trades or holds positions for short windows to capitalize on rapid but not instantaneous market moves.


Short to Long Term Layers (Position Goal = ±x)
For short (minutes to days), medium (days to weeks), and long (weeks to years) time frames, implement mean-reversion, volatility trading, and for longer terms, fundamental analysis inspired by investors like Bridgewater and Buffett. Use AI for predictions, with risk management scaling positions based on market conditions and considering human behavior like herding.

Layer 2 - Short (Minutes to Days, Position Goal = ±x)
This layer extends to mean-reversion strategies like pairs trading and volatility trading, with position goals allowing directional bets (±x). Mean-reversion bets on prices returning to their mean, using statistical models like cointegration tests, as seen in the ScienceDirect paper on Chinese futures (An innovative high-frequency statistical arbitrage in Chinese futures market). Volatility trading could involve options strategies like straddles or strangles, betting on price swings, which requires forecasting volatility levels.
Predictions and AI: The user mentions running prediction + risk + AI stuff, suggesting AI models for short-term price forecasts, possibly using neural networks or statistical models, as noted in Investopedia for modern statistical arbitrage (Statistical Arbitrage). Conditional logic adapts to scenarios like volatility spikes, scaling positions accordingly.
Risk Measurements: Use value-at-risk (VaR) and stress testing, considering potential losses from rapid price movements. The strategic framework may involve derivatives like futures for arbitrage, with short selling for mean-reversion plays.
Human Behavior: Consider behavioral finance aspects like panic selling during volatility spikes, adjusting strategies to exploit or mitigate such reactions, as these can dominate short-term market moves.
Execution Methodology: While less critical than HFT, ensure efficient order execution to capture short-term opportunities, possibly using algorithmic order types to minimize market impact.


4.2 Layer 2 (Short-Term: Minutes to Days)
	1. Signal Generation
		○ Mean reversion from intraday extremes or daily pivot points.
		○ Volatility plays around events (earnings, news).
	2. Position Sizing & Risk
		○ Leverage smaller than HFT but bigger directional bets than Layer 1.
		○ Automatic “take profit” and “stop-loss” triggers.
	3. Execution
		○ A mix of limit and market orders, bridging speed (seconds/minutes) with cost control.
		○ Use signals from Layer 1’s microstructure analysis to refine entry/exit.







Layer 3: Medium-Term (Days to Weeks)
Objective: Trend-following/macro patterns (position goal = ±y).
Time Frame: 3 days to 3 weeks.
Key Strategies:
	1. Momentum Trading:
		○ Breakout detection (e.g., Bollinger Bands, Ichimoku).
		○ MFG Integration: Model institutional order flow (e.g., CTA positioning).
	2. Volatility Regimes:
		○ Switch between long/short gamma strategies based on volatility clustering.
	3. Cross-Asset Arbitrage:
		○ Commodity-currency correlations (e.g., AUD/USD vs. iron ore).
Risk & Execution:
	• Dynamic Hedging: Use options/futures to hedge tail risks.
	• AI Integration: Graph neural networks (GNNs) for cross-asset dependency modeling.

Layer 3: Medium-Term Trading
• Time Frame: Medium-term (a day to a few weeks)
• Strategies: Mean-reversion, volatility trading
• Position Goal: +- x (target position size)
• Objective: Balance short-term gains with medium-term stability
• Key Considerations:
• Fundamental analysis
• Market trends
• Position sizing
1. Layer 3 (Medium-Term)
	○ Time Horizon: A few days up to a few weeks.
	○ Objectives:
		§ Mean reversion on longer cycles (e.g., cyclical trends, seasonal patterns).
		§ Volatility trading on macro events (e.g., central bank announcements).
	○ Key Traits:
		§ Larger position sizes than short-term strategies, depending on broader market or sector trends.
		§ More thorough risk analytics, including macro factors and cross-asset correlations.


Layer 3 - Medium (Days to Weeks, Position Goal = ±x)
Similar to the short term, this layer includes mean-reversion and volatility trading but extends to trend-following strategies, recognizing longer market trends. The position goal of ±x allows for directional bets, potentially using moving averages for trend identification, as part of a diversified strategy.
Predictions and AI: AI models forecast medium-term trends, possibly incorporating alternative data (from QuantLink, The Evolution of Statistical Arbitrage: Rise of Alternative Data and Shorter Holding Periods), enhancing insights beyond technical data. Conditional logic adjusts for market regime changes, like shifts from trending to mean-reverting.
Risk Measurements: Scale positions based on drawdowns, using metrics like maximum drawdown and Sharpe ratio, ensuring sustainability over weeks. The strategic framework may involve options for hedging volatility, and short selling for trend-following against overbought/oversold conditions.
Human Behavior: Consider herding behavior during market trends, where traders follow momentum, potentially amplifying moves. This can be modeled to exploit or counteract, depending on strategy.
Execution Methodology: Focus on minimizing market impact over days, using VWAP or TWAP algorithms for large orders, ensuring efficient entry and exit.

4.3 Layer 3 (Medium-Term: Days to Weeks)
	1. Strategy Design
		○ Exploit cyclical patterns, sector rotation, or multi-day news flow.
		○ Mean reversion / trend-following hybrids: switch strategies based on risk regime.
	2. Risk & Hedging
		○ Incorporate macro hedges (e.g., index futures or options to offset broad market risk).
		○ Evaluate cross-asset correlations—bonds, currencies, commodities—depending on the triggers.
	3. Execution
		○ Larger order blocks over multiple sessions.
		○ VWAP/TWAP to minimize market impact.


Layer 4: Long-Term Investing

• Time Frame: Long-term (a few weeks to years)
• Strategies: Bridgewater Associates-style risk parity, Warren Buffett-inspired value investing
• Position Goal: +- x (target position size)
• Objective: Achieve steady, long-term growth
• Key Considerations:
• Fundamental analysis
• Risk management
• Portfolio optimization

1. Layer 4 (Long-Term)
	○ Time Horizon: A few weeks to multiple years.
	○ Objectives:
		§ Thematic or fundamental investing (a la Bridgewater, Warren Buffett).
		§ Accumulation of strategic positions for fundamental value or macro growth.
	○ Key Traits:
		§ Typically directional, either net long or net short.
		§ Focus on deep fundamental analysis, mean-field game modeling at the macro level, and possibly ESG or large-scale portfolio constraints.


Layer 4: Long-Term (Weeks to Years)
Objective: Macro/value investing (position goal = ±z).
Time Frame: 3 weeks to years.
Key Strategies:
	1. Fundamental Valuation:
		○ Discounted cash flow (DCF) models, EV/EBITDA screens.
	2. Macroeconomic Regimes:
		○ Factor investing (e.g., value, quality) based on yield curve/PMI phases.
	3. Black Swan Prep:
		○ Tail-risk hedging (e.g., long volatility, gold, crypto).
Risk & Execution:
	• Portfolio Rebalancing: Quarterly adjustments with transaction cost optimization.
	• AI Integration: Causal inference models to isolate macroeconomic drivers.

Layer 4 - Long (Weeks to Years, Position Goal = ±x)
Inspired by Bridgewater Associates’ global macro strategies and Warren Buffett’s value investing, this layer focuses on fundamental analysis and macroeconomic trends. Bridgewater, known for economic trend analysis, and Buffett, for buying undervalued companies, suggest a long-term view using valuation metrics like P/E ratios and macroeconomic indicators like GDP growth.
Predictions and AI: Use AI for long-term forecasts, analyzing fundamentals and economic data, possibly using machine learning for trend prediction, as part of a quantitative approach to value investing. Conditional logic adapts to economic cycles, like recessions or booms.
Risk Measurements: Focus on long-term drawdowns, using metrics like Sortino ratio, with position sizing based on risk tolerance and valuation. The strategic framework may involve options for hedging macroeconomic risks, and long positions in undervalued assets.
Human Behavior: Consider market cycle herding, where investors follow trends like tech bubbles, impacting long-term valuations. This can be modeled to identify overvalued/undervalued sectors, aligning with Buffett’s contrarian approach.
Execution Methodology: Less critical due to longer horizons, but ensure cost-effective execution, possibly using passive strategies like index funds for broad exposure, with active trades for specific opportunities.

4.4 Layer 4 (Long-Term: Weeks to Years)
	1. Fundamental & Macro Research
		○ Evaluate company fundamentals, macroeconomic projections, and economic cycles.
		○ Potentially incorporate mean field game aspects at institutional scale (forecasting pension funds, central bank moves, etc.).
	2. Portfolio Construction
		○ Concentrated or diversified positions?
		○ Long-biased or balanced with derivatives for downside protection.
	3. Monitoring & Rebalancing
		○ Low turnover but consistent fundamental review.
		○ Adjust or hedge positions if fundamentals or macro environment shift abruptly.



2. Core Elements Carried Over from Model Version 1
	1. Predictive Models & Conditional Logic
		○ Each layer should incorporate specialized forecasting:
			§ Layer 1: Microstructure forecasting (limit order book dynamics, short-term price changes).
			§ Layer 2, 3, 4: Increasingly macro or fundamental signals (e.g., news, economic data, sector rotation).
		○ Conditional logic triggers trades or changes strategy based on scenario detection (e.g., breakout, volatility spike, overbought/oversold signals).
	2. Risk Measurement & Statistical Considerations
		○ Each layer has a different risk management horizon:
			§ Layer 1: Tight risk controls to prevent large intraday drawdowns.
			§ Layer 2: Slightly longer risk tolerance, but stops or scaling rules on trades spanning days.
			§ Layer 3 & 4: Broader drawdown thresholds, but require deeper scenario analysis (e.g., monthly or quarterly stress tests).
	3. Strategic Framework (When to Push vs. Follow)
		○ In addition to observing the market, each layer may push the market under certain conditions:
			§ Layer 1: Aggressive liquidity-taking or “quote stuffing” to move quotes upward/downward if the model deems it profitable.
			§ Layer 2 & 3: More discretionary “push” based on perceived short- or medium-term market momentum.
			§ Layer 4: Typically less emphasis on “pushing,” more on fundamental reallocation or accumulation.
	4. Incorporation of Human Behavior
		○ Behavior-driven signals become more pronounced in longer time frames:
			§ Layer 1: Human emotion noise is minimal at tick level—though “FOMO spikes” can show up as fleeting microstructure anomalies.
			§ Layers 2 & 3: Reactions to earnings, macro announcements, and sentiment can drive mispricings.
			§ Layer 4: Herding, panic selling, or mania phases (think “dot-com bubble”) significantly affect long-horizon valuations.
	5. Execution Methodology
		○ While sophisticated predictions matter, sometimes execution alone can yield alpha:
			§ Layer 1: Smart order routing, queue position strategies, hidden/iceberg orders.
			§ Layer 2 & 3: Time-weighted average price (TWAP), volume-weighted average price (VWAP) to accumulate or dispose of positions stealthily.
			§ Layer 4: Minimizing market impact over days/weeks of order flow.

3. Mean Field Games & Cross-Layer Interaction
	• Mean Field Game Perspective:
In a mean field game, each agent is small relative to the market, but the aggregate actions of many agents shift the dynamics.
		○ Layer 1: The algorithm effectively is one of many HFT market makers. Profits depend on anticipating the reaction of other HFTs.
		○ Layer 2 & 3: Incorporate short-term signals and the probable behavior of many medium-frequency traders.
		○ Layer 4: Models large institutional movements (mutual funds, pensions, etc.). By factoring in the collective state of the market, the algorithm can forecast inflows/outflows or “herd” movements.
	• Cross-Layer Synergy:
		○ Layer 1 → Layers 2 & 3: High-frequency order-flow data can feed short- and medium-term signals (e.g., changes in liquidity, volume spikes).
		○ Layers 2 & 3 → Layer 4: Medium-horizon indicators, combined with fundamentals, can inform a bigger directional stance.
		○ Layer 4 → Layers 1, 2, 3: Large positions from Layer 4 might generate short-term anomalies or constraints in liquidity—enabling arbitrage or rebalancing strategies in lower layers.

5. Unifying the Layers: Governance & Orchestration
	1. Common Risk Dashboard
		○ Consolidates exposure across all layers.
		○ Summaries of daily VaR (Value at Risk), portfolio drawdown, and liquidity usage.
	2. Capital Allocation
		○ Dynamically allocate capital to each layer based on performance and market conditions.
		○ Example: If the short-term strategies are consistently profitable while HFT is struggling in a low-vol environment, reallocate margin and risk budget to short-term.
	3. Technology & AI Integration
		○ Shared ML/AI Infrastructure:
			§ Reinforcement learning or advanced ML models might extract common signals across time horizons.
			§ Cloud-based HPC for training; on-premises or co-located servers for real-time inference.
		○ Data Lake:
			§ Unified data ingestion (market data, fundamental data, news sentiment, etc.).
			§ Cleaning and feature engineering pipelines reusable across layers.
	4. Regulatory & Compliance
		○ HFT and short-term strategies can trigger regulatory scrutiny (e.g., market manipulation concerns).
		○ Auditing logs of trades, reasoning, and “algo kill-switch” if performance or risk deviates drastically.

6. Iterative Development & Testing
	1. Prototyping in Sandboxes
		○ Test each layer separately in simulated or paper-trading environments.
		○ Evaluate latency, fill rates, and slippage for the HFT layer.
		○ Evaluate short/medium/long layers in backtesting with robust walk-forward testing.
	2. Integration Phase
		○ Combine the layers with a single portfolio manager.
		○ Test synergy: do HFT signals help short/medium layers improve entry timing? Does the long-term layer’s fundamental stance act as a hedge or anchor?
	3. Deployment & Ongoing Monitoring
		○ Incremental release of each layer to production, starting with small capital allocations.
		○ Expand allocations as alpha and risk metrics confirm the strategies.


Unified Framework Components
	1. Prediction Engine:
		○ HFT: Order-book dynamics + MFG competitor modeling.
		○ Short/Medium: Hybrid models (LSTM + GARCH). Short and medium swings. Market regime…
		○ Long: Macro-factor models + causal AI. Port opt. etc.
	2. Risk Management:
		○ Hierarchical VAR: Aggregate risk across layers.
		○ Risk engine: Hedging, limits, etc.
		○ Liquidity Buffers: Reserve capital for Layer 1 shocks.
	3. Behavioral Overlays:
		○ Sentiment Gauges: Crowd panic/euphoria indicators (Layer 2/3).
		○ Institutional Flow: Track ETF creations/redemptions (Layer 4).
	4. Execution Synergy:
		○ Use Layer 1 to "scout" liquidity for Layer 2/3 trades and to execute trades.
		○ Layer 4 macro signals inform Layer 3 volatility positioning.

Continuously monitor and adjust position sizes, risk exposure, and strategy allocations across all layers
Next Steps
	1. Develop a detailed design document outlining the architecture, components, and workflows for each layer.
	2. Implement and backtest the individual strategies and layers using historical data.
	3. Integrate the layers and test the overall framework using simulated trading environments.
	4. Refine the model based on performance metrics, risk assessments, and market feedback.



F(0,1,2)=P(0,1)/P(0,2) −1=e^(−1 ∗y(0,1)+2∗y(0,2) )
If the discount curve P(t,T) is increasing, the simple forward rate is Negative

If the yield curve is increasing, the instanteous forward curve is above the yield curve.

If the yield curve is increasing. The forward yield is 


If the yield curve is increasing. The simple forward rate is


If the term structure of the simple spot rates is decreasing and we assume L(t,T0) * L(t,T1) > 0, the simple forward rate is not between

Value of money market account A=P⋅e^rt
A is the final amount
P is the initial investment
r is the short rate
t is the number of years
e is euler's number

The Yield of a money market account A=P(1+simple rate)


sdweweawewaewae

<img width="814" height="488" alt="image" src="https://github.com/user-attachments/assets/6b5ac8ee-2ebd-462c-9050-d60ccf4f1f33" />
<img width="730" height="761" alt="image" src="https://github.com/user-attachments/assets/9bd0e8c7-6dfa-4d95-a33c-0ad5565d432c" />



<img width="325" height="46" alt="image" src="https://github.com/user-attachments/assets/8815f4ff-8c7b-4bd4-b3f4-b03205c34d84" />
<img width="740" height="557" alt="image" src="https://github.com/user-attachments/assets/74cf60d8-70cc-4c60-bc88-139816fb05c4" />


<img width="697" height="278" alt="image" src="https://github.com/user-attachments/assets/f117e0ea-9a6a-43d3-b9f6-bdf52593b3c4" />
<img width="566" height="266" alt="image" src="https://github.com/user-attachments/assets/a1fb92e6-5ded-45ef-ab98-958db32f9654" />








Rank		Name	Algo description	Improvement notes
#2 Overall	IMC2	Linear Utility		
#2 Overall	IMC1	Stanford Cardinal		
#9 Overall	IMC2	Jasper (Von Merle)	Market-making	Soft/hard liquidation procedures at less than usual prices to get out of prolonged positions.

Linear regression on starfruit, but not profitable?
			        - Buy/sell price from maximum volume
			        - Trading starfruit around 10k
			
#13 Overall	IMC2	Kang, Kim, Lee		
#30 Overall	?	Gabriel Romon		
#57 Overall	IMC1	Michael Okon		
#91 Overall		Nicolas Sinott		
#X >350 Overall	IMC1	Academiq-AI		
#381 Overall		David Teather		
				
				
				

The challenge is run in rounds. The main focus is market-making
With the opportunity to do Manual Trading as well.

Round 1

Algo
The algo side of round 1 appeared to be a continuation of the tutorial round on new data. I tried to improve my market making strategy, but couldn't improve on my tutorial code, so ended up submitting that without any changes.

In this round, we worked on pearls and bananas separately. For pearls, we realized that they had a stable price of around 10,000, but there were times when the best bid was 10,002 and the best ask was 9,998. Additionally, there were trades even at -5, -4, +4 and +5 of the mean price (10k), which meant that all we had to do for pearls was market-take and market-make around 10k.
For bananas, we ran a linear regression on the last few timesteps of banana prices to predict the next price. Using this to market-take worked reasonably, and market-making along with market-taking worked quite well. We tried to implement LR-on-the-fly, but this didn't work as well (and with the Lambda error bugs that were prevalent through the competition, it was for the best to skip this approach). We noticed that the banana trends were linear-ish across days (ie, day 2 was more similar to day 1 in LR coefficients than day -1 and so on), which could give an additional boost in PnL.

Manual
How to trade with secretive fish. A challenging and interesting… | by Edgar Maddocks | Medium

The manual trading was straightforward in this round. Based on the results of Round 1, we were in 9th place.



Round 2

Algo
The trick was to continuously short sell to the limit at a price at which you could immediately convert profitably back to 0 in the next iteration, hoping for other bots in the simulated market to fill your orders. This got me to 60k profit at first, and to 109k minutes later after I realized you can convert your position to 0 and then immediately go short again in the same iteration. ~Jasper

Manual
For the manual side of round 2 I calculated the profit of each possible answer, and ended up submitting seashells -> pizza slices -> wasabi roots -> seashells -> pizza slices -> seashells, which gave the maximum profit of 113,938.8.

For the manual round, we submitted a guess of 9,777 based on a simulation. This was good enough to net us some profit, with the median price of the bottom 50% being 9,685. Shockingly, we got a score of below -120k in the algorithmic trading in this round, which dropped us from 9th place to... 926th place!!!





Round 3

Algo
After the round opened, it was clear that the algo side of round 3 was basically a copy-paste of last year's round 4. As such, I started off with building a strategy based on the difference between the value of gift baskets and the combined value of the contents of a gift basket. When this difference crosses certain thresholds, the strategy would go 100% long or 100% short on gift baskets. I later extended it to mirror the position on the individual products of the gift basket, which did reasonably well (although overfit).
After hearing other participants reaching 600k in backtests and failing to figure that out myself, I decided to start trading the gift baskets and its components at different thresholds, and performed several grid searches to find good thresholds for the 30k iterations of example data. This resulted in a profit on everything but roses, on which I lost over 36k. Following these results I disabled my trading on components for future rounds, as I deemed it too risky against too little potential profits.

Manual
For the manual challenge of round 3 I combined my gut feeling, some some calculations I ran on guesses of the number of participants choosing each tile, and the spreadsheet containing user-reported expeditions created by noah1921 on Discord. Based on these sources, I figured it was unlikely to earn >75k from an expedition, while >25k seemed very doable, making two expeditions the best amount. I decided on expeditions 83 and 87, mostly based on noah1921's spreadsheet, and ended up with a profit of 78,577. The maximum profit of this manual round was 106,291, which was a bit unfortunate.

Round 4

Algo
Round 4 had unfortunate scheduling issues. Because both round 1 and round 3 were extended by 24 hours following major production incidents that made submissions impossible, round 4 fell entirely within a pre-planned weekend holiday of mine to Luxembourg to hike part of the Mullerthal trail with my father. I didn't plan on bringing my laptop, meaning all my round 4 work had to be done on a phone.
The evening before leaving I built a custom Streamlit app to convert email/password to ID token, to use my submitter (converted the CLI to work inside Streamlit), to download leaderboard data (for my own leaderboard website), and to list submissions. On the way to Luxembourg I decided to try some Python editors in the Play Store, and found Pydroid 3 to be the most capable one, because it allows installing and running arbitrary PyPI packages like my backtester. I also found an app called Desktop FullScreen Web Browser, which let me bypass Prosperity's anti-mobile devices "feature" (that happens to be resilient against the "Show desktop version" features in Chrome and Firefox).
For the algo side of round 4 my inability to effectively analyze the provided data was pretty annoying. While it was clear that coupons were meant as an option-like instrument with coconuts being the underlying assets, I had some trouble turning that knowledge into a profitable strategy. Based on hints from other participants I eventually tried to apply the Black-Scholes model to calculate the expected price of the coupons, and then traded around that. This worked, and following discussions with other participants I was able to tune my Black-Scholes parameters to getting 420k in backtests. The profit I got on coupons in backtests was pretty close to the profit I got at the end of the round, which was nice to see.
I also created a simple directional trading strategy for coconuts themselves, expecting them to mostly go upwards or downwards during a trading day. This assumption failed catastrophically at the end of the round, leaving me with a 77k loss on coconuts against a 184k profit on coupons. Following these results I stopped using this strategy in round 5.

Manual
For the manual challenge I figured most people would probably go for round 2's optimal bids of 952/978. Because I thought some people would increase their high bid to be at or above the average, I decided to go for 952/980 instead. The average bid ended up being 980, which left me with a nice profit of 102,592, which appeared to be the maximum profit when comparing to results others posted on Discord.


Round 5

Algo
Round 5's algo challenge introduces de-anonymized trades data. I initially stared at generated charts for a while and found four credible signals, and then ran a grid-search consisting of 3,600 backtests to find signals I might've missed by eye, finding nothing new. These are the signals I found:
	• If Vladimir sells CHOCOLATE to Remy, then go long on CHOCOLATE.
	• If Remy sells CHOCOLATE to Vladimir, then go short on CHOCOLATE.
	• If Rihanna/Rhianna sells ROSES to Vinnie, then go long on ROSES.
	• If Vinnie sells ROSES to Rihanna/Rhianna, then go short on ROSES.
My final strategy for each product is as follows, and all of these ended up being profitable in round 5:
	• AMETHYSTS: market-making around 10,000.
	• STARFRUIT: market-making around the popular mid price.
	• ORCHIDS: arbitrage between the north and south archipelagos.
	• CHOCOLATE: directional trading based on trades between Vladimir and Remy.
	• STRAWBERRIES: directional trading based on parameters overfit on round 3 data.
	• ROSES: directional trading based on trades between Rihanna/Rhianna and Vinnie.
	• GIFT_BASKET: directional trading based on parameters overfit on round 3 data.
	• COCONUT: nothing, I couldn't find a consistently profitable strategy for it.
	• COCONUT_COUPON: trading around the expected value of the product calculated using the Black-Scholes model.

Manual

For the manual side of this round I read the Iceberg news and decided that for electric blankets (sell), sleds (sell), ice sculptures (buy), PS6 (buy), and moustache serum (sell) I felt confident about which direction I thought they would move, and that for icicle earrings (buy) I was a little less confident but was still willing to trade. For refrigerators, lava lamps, and hot chocolate I didn't feel certain enough about their direction, so I decided not to trade them. I allocated 10% of my capital to icicle earrings, and then divided the remaining 90% equally between the other products. This resulted in a manual profit of 115k, with the best manual result for this round being 149k, and the optimal being 157k.


Additional thoughts
it appears the trick to getting multi-million algo profits in rounds 4 and 5 was to exploit the coconut data in the Prosperity 2 being a (near-)exact match with the coconut data in Prosperity 1. Congratulations to Puerto Vallerta for figuring that out first during round 4 (presumably), and to the other three teams in the final top 4 that seemingly figured it out too during round 5.
Prosperity 2 was yet again an interesting competition, with some challenges being very similar to last year's with others being brand-new. It was a bit disappointing to see two rounds being extended due to the website going down for extended periods of time, especially after Prosperity 1's issues with AWS Lambda errors. Hopefully Prosperity 3 is plagued with fewer issues. Nonetheless, I thank Prosperity's organizers for keeping the competition going, and all other participants for creating a lively online community together for the duration of the competition.
TradingState.order_depths['KELP'] =

Market
Essentially Trading state:

Timestamp, 
Listings, 

TraderData, 
Position, 

Own_trades, 
Order_depths, 
Market_trades, 
Observations.

Agents
We don't know what triggers what behaviour with these.
They post buy-sell orders, shown in order depth.
Posting ludicrous buy-sell orders, don't effect these.

Trader
Position limits 
Takes TradingState.
Runs through products in the market.
Considers order_depths
returns orders to market.

<img width="1195" height="890" alt="image" src="https://github.com/user-attachments/assets/2d589a55-24ea-44b3-a5db-b1fbe12ea5d8" />


	1. Mid-price
		a. Drift
		b. Volatility
		c. Risk premium, jump probabilities

dS_t=μ_t dt+σ_t  dW_t+∫▒J_t  dN_t

	1. Spread
		a. High volatility - widen spread
		b. Low volatility - narrow spread
	2. Bid price
		a. Predict future optimal bid
		b. Positive drift - raise bid
		c. Negative drift - lower bid
	3. Bid amount
		a. High volatility - Smaller amount
		b. Low volatility - larger amount
	4. Ask price
		a. Predict future optimal ask
		b. Positive drift - lower ask
		c. Negative drift - raise ask
	5. Ask amount
		a. High volatility - Smaller amount
		b. Low volatility - larger amount
	6. Inventory
		a. High volatility - avoid large inventory
		b. Low volatility - more open to risk
		c. Adjust for jumps
		d. Crossing the spread to flatten positions?
		e. Risk limits, maximum inventory bounds, order throttling
	
