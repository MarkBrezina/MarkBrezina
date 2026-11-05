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
	






Round 1
Products:
	• Amethysts – A “stable” asset priced near 10,000 seashells, rarely deviating from that fair value.
	• Starfruit – A more volatile asset whose price “random-walked” through hundreds of seashells.
Key Observations/Strategies:
	1. Market-Making on Amethysts:
		○ Most teams treated 10,000 as the “fair” or “true” price.
		○ They sold (took or quoted) whenever bids exceeded 10,000 and bought whenever asks went below 10,000.
		○ A big factor was the position limit (often ±20); many implemented “liquidation logic” to flatten positions so they could continue taking advantage of profitable trades.
	2. Estimating Starfruit’s Fair Price:
		○ Price wasn’t fixed, but teams noted that short-term fluctuations often revert or stay near a local midpoint.
		○ Rolling midprice or rolling average was common, but many discovered a large “market-maker bot” quoting consistent deep liquidity on both sides.
			§ They realized that the hidden “fair” used by the game was essentially an average of these large-lot bid/ask quotes—trading around that level was best.
		○ Some tested simpler methods (e.g., linear regression on the last few timesteps), but found the biggest improvement came from ignoring micro-noise and anchoring to the “heavy-lot” quotes.
	3. Outcome:
		○ Teams who systematically market-made with an accurate fair price and actively flattened inventory tended to rank high.
		○ Many top-tier solutions netted 30k–40k seashells or more in profit for Round 1.

Round 2
Products/Mechanics:
	• Orchids – A new product with an apparently complicated price driver: sunlight, humidity, shipping costs, import/export tariffs, etc.
	• Orchids could be traded locally or on a “south archipelago” exchange, incurring tariffs/costs and position rules (only import/export if it moves your net position toward zero).
Key Observations/Strategies:
	1. Weather & Price Modeling Fell Short:
		○ The data (sunlight/humidity) looked important but provided only weak or spurious correlations—teams who tried advanced regressions on it mostly concluded it was a “red herring.”
	2. Cross-Exchange Arbitrage:
		○ A large, seemingly “insatiable” buyer on the local market would often buy huge volumes of Orchids if quoted only slightly above the best bid.
		○ Meanwhile, the import side from the foreign market often had favorable net costs (tariffs, shipping) that ended up paying the seller rather than costing them.
		○ Thus, the big money-maker was to sell locally (at a price above your cost) and import from the south archipelago to flatten your position, capturing a near risk-free profit.
		○ Some code “two lines” achieved enormous volumes and 60k–100k+ seashells in a matter of hours (or 500k–1M in extended simulations).
	3. Adaptive Sell Quotes:
		○ Teams refined how high to place their local market’s sell prices, factoring in import tariffs and shipping.
		○ If tariffs were low, the trade might not be profitable at all—some then tried alternative small alpha signals from humidity/sunlight, but in the text, no one reported major success from that approach.
Outcome:
	• This round led to huge profit differentials. Teams who recognized and exploited cross-exchange arbitrage jumped far ahead; those who missed it or discovered it too late might have drastically lower or even negative PnL.

Round 3
Products:
	• Gift baskets – A basket containing some combination of chocolate bars, strawberries, and roses (e.g., 4 chocolate, 6 strawberries, 1 rose).
	• Each basket component was individually tradable as well.
Key Observations/Strategies:
	1. Basket vs. “Synthetic Basket” Arbitrage:
		○ The basket’s price frequently drifted away from the sum of its contents.
		○ Teams performed a spread trade:
			§ “Spread” = (Gift Basket Price) – (4 × Chocolate + 6 × Strawberries + 1 × Rose).
			§ Because the basket should trade near the sum of its components, that difference often mean-reverted around a stable level (e.g., 0, 300, 370, etc.).
		○ Many used z-score or hard thresholds to buy the cheaper side and sell the more expensive side.
	2. Leading vs. Lagging Components:
		○ Some tested whether gift baskets “led” or “lagged” changes in their components. Results were mixed; the simpler approach of pure mean reversion or ratio bounding typically sufficed.
	3. Position Limits & Slippage:
		○ Because of small position caps, timing mattered—teams wanted to deploy capital exactly at large mispricings and exit quickly.
		○ Overfitting was a risk: many found real-round results were worse than carefully tuned backtests.
Outcome:
	• Most participants ended up with moderate gains from spread trading.
	• More advanced or carefully optimized mean-reversion strategies netted higher PnL.

Round 4
Products:
	• Coconuts – Hovered around ~10,000 seashells.
	• Coconut coupons – Essentially a 10,000-strike call option expiring in ~250 days; near the money.
Key Observations/Strategies:
	1. Black-Scholes (BS) Pricing & Implied Vol:
		○ Participants recognized the coupon as an option and used standard option pricing.
		○ Implied volatility stabilized around ~16%, so many used a mean-reversion approach on implied vol:
			§ Buy coupons when implied vol was “cheap,” short them (or sell) when implied vol was “rich,” and hedge delta with the underlying coconuts.
	2. Delta Hedging & Partial Hedges:
		○ Because of position caps (e.g., 600 coupons vs. 300 coconuts), some couldn’t fully hedge. This introduced extra directional risk.
		○ A few accepted that risk in hopes of higher returns (positive vega plus leftover delta).
	3. Advanced Trick: Using Old Data for Perfect Foresight
		○ Some discovered that the coconut data was suspiciously similar (or identical) to a prior Prosperity’s data. By mapping old data to the current round, they could essentially “see the future.”
		○ That led to extremely high PnL for the small number of teams who discovered it early.
Outcome:
	• “Straight” BS-based volatility trading + partial hedging yielded moderate profits.
	• Teams leveraging data re-use and perfect forward knowledge achieved spectacular results (e.g., 1M+ seashells).

Round 5
Mechanic Change:
	• Trade history was de-anonymized, letting participants see exactly who bought or sold to whom.
	• Some participants noticed that certain characters (e.g., “Rianna,” “Olivia,” “Remy,” etc.) consistently “bought at bottoms” or “sold at tops,” providing a near-perfect trade signal if you mirrored them.
Key Observations/Strategies:
	1. Identifying Trader “Alpha”:
		○ A big final-round improvement was following an NPC who always “timed” roses or another product effectively.
		○ People also used the data to replicate specific whales or “smart players.”
	2. Extending Old Data “Hack”:
		○ Some again leveraged data from prior Prosperity competitions or earlier rounds, discovering near-perfect correlations in certain assets. This could be used to time big swings for massive gains.
	3. Last Push for Gains:
		○ Because this was the final, teams risked bigger bet sizes if it might jump them in rank.
		○ Many mention applying partial or no delta hedging, dynamic programming on trade sequences, or choosing only signals they trusted to avoid overfitting.
Outcome:
	• Round 5 often saw big leaps in rank from teams who discovered strong signals in the newly revealed trade logs.
	• Some soared to multi-million seashell profits by combining re-labeled data from older competitions with well-tuned final trades.

Other Shared Themes / Strategies
	1. Market-Making + Position Management:
		○ Almost every round featured attempts to quote both sides with a small edge around a “fair” price.
		○ Inventory (position) constraints forced participants to flatten out or “liquidate” if they got too long/short, so they could keep capturing new edges.
	2. Mean Reversion & Spread Trades:
		○ Repeatedly used for starfruit, orchids, gift baskets vs. components, and later the coconut option.
		○ Checking the difference (or ratio) between two correlated prices and betting on the gap returning to normal.
	3. Heuristic Tuning / Grid Search:
		○ Many participants mention using small offline backtesters to “grid search” or “hyper-tune” parameters (spread widths, how many ticks away to quote, how far to deviate before taking a position, etc.).
	4. Data Reuse or External Info:
		○ In Rounds 4 and 5, some strategies exploited exact matchups to previous years’ data or used newly revealed trade-partner names for alpha.
	5. Manual Rounds (where relevant):
		○ Some mention a separate “manual challenge” each round, frequently solved with simulations, closed-form solutions for specific puzzle-like setups, or pure guesswork.

Final Takeaways
	• Round 1 & 2: Basic market-making, simple fair-value estimates, or cross-exchange arbitrage can yield big gains.
	• Round 3: Spread/mean reversion on baskets vs. underlying components is typically straightforward but profitable.
	• Round 4: Option pricing with Black–Scholes or a volatility-based approach + delta hedging.
	• Round 5: Tapping de-anonymized trade logs for alpha signals or reusing old competition data for near-omniscient “predictions.”

Despite AWS/Lambda technical glitches, participants overall enjoyed blending theory (Black–Scholes, regression, mean reversion) with practical heuristics (detecting big bots, position flattening, two-line arbitrage). Several teams soared in the standings by uncovering “hidden” data patterns or by carefully backtesting and tuning.


1. Naive or “Sample” Strategy
	• Participants refer to an initial “sample” or “tutorial” algorithm provided by the organizers (which often performs poorly, sometimes even going negative).
	• Many try to improve upon or replace this baseline with something more adaptive (e.g., changing the “fair price,” adding spreads, or building market-making logic).

2. Simple “Buy Low, Sell High” / Fair-Value Approaches
	• Multiple users mention the idea of computing a “fair price” (possibly a moving average) for Kelp or Resin and trading around it (buy below fair, sell above fair).
	• This is typically a momentum/mean-reversion or basic market-making approach, adjusting quotes up or down based on whether the market price is “cheap” or “expensive.”

3. Market Making vs. Trend Tracking for Kelp and Resin
	• Some discuss market making on the more stable product (Resin) and trend-trading or momentum on the more volatile product (Kelp).
	• The logic:
		1. Resin is stable, so you can quote tight spreads and capture small, low-risk profits.
		2. Kelp moves more dramatically, potentially yielding bigger directional gains (or losses).

4. Experimenting With “Volume-Based” or Parameter-Tuning Strategies
	• A few participants describe attempts to tweak volumes, risk limits, or “spread sizes” in code.
	• One user mentions going “100% short at the first iteration” because of how their volume logic was coded.
	• Others speak of “hypertuning” moving-average or learning-rate parameters on the short tutorial data—even though they acknowledge it may not generalize to the hidden dataset.

5. “Worst PnL” or “Negative Infinity” Approach
	• A small side conversation focuses on intentionally losing as much as possible (e.g., to test backtester bugs, or just for fun). Some mention easily generating extreme losses by removing position constraints or skewing orders.
	• This is obviously not a strategy people want to use to win—just a code curiosity.

6. Arbitrage-Like or Low-Risk “Grinding” Strategies
	• A few references to past competitions suggest that some top strategies end up looking like near-arbitrage or “market making that does minimal directional risk.”
	• Because of the position limits and many repeated trades, purely “gambling” or taking massive directional bets typically doesn’t pay off long-term.

7. Manual Trading vs. Algorithmic Execution
	• There’s curiosity about “manual trading,” but it seems mostly overshadowed by the focus on algorithmic submissions.
	• People wonder if they have to “sit at their desk for 72 hours” to do manual trades. The general consensus is that it’s not truly real-time—submissions and manual trades happen before the round ends.

8. General Themes
	• Adaptation to Bots: The environment’s “hidden bots” fill orders dynamically; a good strategy must be robust to those bots’ behavior.
	• Backtester vs. Reality: The official competition engine uses hidden bot logic that the open-source backtester can’t replicate exactly, so participants see different results when they upload code.
	• Position Limits: Hard caps on how big a position (long or short) you can hold, preventing infinite leverage.
	• Time Constraints: Code must run efficiently under competition server limits (some participants report timeouts with heavier logic).

In Short
Overall, the most frequently mentioned strategies are:
	• Basic market making with fair-value quotes.
	• Trend or mean-reversion trades on the volatile asset (Kelp).
	• Arbitrage-like or very low-risk quoting (especially for stable Resin).
	• Parameter tweaking on the tutorial data (acknowledging it might not generalize to hidden rounds).
No single “advanced” or unified approach dominates the discussion yet; most participants are iterating on simple or moderate-complexity methods, trying to beat the naive sample strategy and handle the challenge’s hidden aspects.


1. Avellaneda–Stoikov (AS) Model
Key Ideas
Risk Aversion and Inventory Control
	• The model incorporates a parameter (𝛾) for risk aversion. As your net position (inventory) grows, the model automatically adjusts your bid and ask prices to reduce exposure and the risk of adverse moves.
Dynamic Spread Setting
	• Quotes are updated continuously based on market volatility and inventory levels. The spread widens when inventory is large or volatility is high, and narrows otherwise.
Poisson (or Exponential) Arrival Models
	• The model frequently assumes that incoming market orders arrive randomly (e.g., via Poisson processes), which helps estimate the probability of filling on either side of the spread.
Why It’s Popular
The Avellaneda–Stoikov framework is one of the earliest academic “optimal market making” models well-suited for limit order book markets. It explicitly combines risk management (controlling inventory) with optimal quoting decisions in a fairly tractable mathematical form.
2. Extensions of Avellaneda–Stoikov
Several researchers have extended the AS model to account for more realistic market microstructure features:
Order Book Depth / Queue Position
	• Extensions consider the size of the posted quotes and how far they are in the queue, because fill probability depends on how many orders are ahead of you.
	• Some queue-based models (e.g., based on Markov or fluid approximations) try to estimate the chance of execution for different quote placements.
Stochastic Volatility or Price Dynamics
	• Rather than assuming a simple Brownian motion for the underlying price, advanced versions incorporate stochastic volatility, jumps, or correlated factors.
Discrete-Time vs. Continuous-Time
	• Real systems often implement quoting logic at discrete intervals (milliseconds to seconds) rather than a purely continuous process.
3. Cartea & Jaimungal Framework
Key Contributions
Risk and Execution Cost
	• Similar to AS, but puts additional focus on measuring execution costs (e.g., fees, slippage, the cost of hedging or crossing the spread).
Market Impact Modeling
	• In certain versions, the model acknowledges that large trades or persistent quoting behavior can move prices or alter arrival rates.
Practical Uses
	• Many HFT and prop-trading firms adapt Cartea & Jaimungal (and coauthors’) ideas, as they offer a flexible continuous-time approach and a robust method for deriving closed-form or semi-closed-form controls for quoting strategies.
4. Queue-Based and Limit Order Book Models
Motivation
	• In real electronic markets, your fill probability depends heavily on where your order sits in the limit order queue. Being first in queue can be more valuable (higher chance of execution) than being last.
Key Features
	• Queue Dynamics: Models approximate how orders arrive and cancel in the order book.
	• Priority Positioning: Some approaches treat the order book as a discrete set of “levels,” analyzing the probability of getting filled if you move your quotes in or out by one tick.
	• Optimal Tick Placement: Decide whether to quote at the best bid/ask, one tick away, or deeper in the book to balance fill probability and profitability.
5. Reinforcement Learning and Machine Learning Approaches
Growing Trend
	• Many real-world HFT firms complement (or replace) classical stochastic control methods with machine learning (ML) or reinforcement learning (RL).
	• Why? These models can adapt to non-stationary markets and learn complex patterns not well captured by simpler assumptions.
Typical Pipeline
	• State Representation: Observations about the current order book state, inventory, volatility regime, time of day, etc.
	• Actions: Adjusting bid and ask quotes, or deciding not to post at all.
	• Rewards: Often tied to realized PnL, inventory penalties, and fill rates.
Challenges
	• High-dimensionality, need for large datasets, risk of overfitting, and the requirement for very low-latency decisions.
6. Other Real-World Considerations
Adaptive Parameters
	• Risk aversion (𝛾), volatility estimates, and fill probabilities are not fixed; they must be re-estimated or adapted in near-real time based on incoming data.
7. Summary
Most professional market makers—whether in equities, futures, FX, or crypto—use stochastic control or reinforcement learning approaches inspired by (and building upon) academic models like Avellaneda–Stoikov. They incorporate:
	• Dynamic Inventory Control: Adjusting quotes in response to changing inventory.
	• Realistic Microstructure: Modeling how quotes get filled (queue position, fill rates).
	• Adaptive Risk Aversion: Widening spreads or reducing size in volatile conditions.
	• High-Frequency Implementation: Needing robust, latency-sensitive technology.
The final “secret sauce” typically comes from fitting model parameters to real order flow, calibrating fill probabilities, hedging costs, and adjusting strategies for short-term patterns. Over time, these models are continually refined in response to new data, market regime changes, and technological innovations in HFT.



1. Naive “Sample” Algorithm (Acceptable Price = 10)
	• Several users refer to a tutorial or sample script that places orders around a hard-coded “acceptable price” (e.g., 10).
	• This is presumably the default strategy provided in the tutorial round.
	• Many participants note that it often results in negative or suboptimal PnL, illustrating that more sophisticated logic is needed.

2. Simple Two-Product Exploit (Resin vs. Kelp)
	• The challenge environment features at least two products:
		○ Rainforest Resin, described as “stable.”
		○ Kelp, described as “volatile” (goes up and down).
	• Some chat members suggest that you can exploit the stability of one product versus the fluctuations of the other—e.g., by taking offsetting positions, looking for mean reversion, or capturing momentum in the volatile asset.
	• This amounts to a basic spread-trading or relative value approach between the stable and volatile product.

3. Experimentation and Iterative Refinement
	• Many participants mention trial-and-error: uploading scripts, checking the resulting PnL graph in the dashboard, and iterating.
	• The environment’s dynamic, partially hidden nature (unknown future data, unknown exact bot behavior) pushes teams to experiment with different quotes or price levels.

4. Observing (and Reacting to) the Bots
	• The Prosperity game environment includes built-in “bots” that simulate an active marketplace.
	• The community notes that these bots are “dynamic”:
		○ They are not simply matching your orders one-for-one; instead, they place their own trades, effectively shaping market conditions.
		○ A good strategy might therefore adapt to how these bots move prices or fill orders, rather than using a single static approach.

5. Inventory / Exposure Control
	• Although not deeply explained in the chat, there are references to an exposure limit and the possibility of ending up with an unwanted “stuck” short or long position.
	• A few participants hint at the need to manage inventory over time so the PnL curve does not crash if the bot moves the market against large unhedged positions.

6. Data Uncertainty & Hidden Testing
	• Users confirm that only 2,000 ‘tutorial’ data points are visible for testing, while 10,000 hidden data points will determine final results.
	• Thus, the main “strategy” for many participants is to find a robust approach that does well both on the sample data and (hopefully) on unseen data—i.e., avoid overfitting to the 2k tutorial ticks.

7. No Single “Advanced” Method Yet
	• Some participants ask about book recommendations, or mention they want to try more sophisticated approaches (machine learning, dynamic order placement, etc.). However, no specific advanced algorithm (like Avellaneda–Stoikov or a known RL method) is explicitly detailed in the conversation.
	• In essence, the community acknowledges that to succeed, they likely need something beyond the naive “acceptable price = 10” method—yet they are still exploring how to get there.

In Short
	• The only concrete “strategy” spelled out is the naive sample algorithm included with the tutorial.
	• Most of the conversation revolves around participants troubleshooting, discussing partial ideas (e.g., exploiting stable vs. volatile assets), or clarifying how the environment/bots work.
	• Everyone is aiming to refine or replace the naive approach by experimenting with real-time quotes, monitoring the resulting PnL, and iterating toward something profitable.


General: 

1. Indexing feasible strategies for prospective rounds: 
- Stock Splits 
- Dividend Based 
- Premiums 

2. Generating + Finding Prospective Manual Trading Questions. 

3. Indexing niche optimisations for core strategies from new research and old Repos. 


05/04: Notable Finds with Saeid: 

ATR: In essence, dividing logic upon 3 liquidity states:
	• Default: Spread = ATR/2
	• Liquid: Spread = ATR/1
	• Highly illiquid (0 Trades in last N time-steps): ATR/1 if Market Depth > X

Market Making: VWMP
Momentum: Semi-OU FV: Mean + Theta(Mean-Midprice)
[ATR signal]

Find out: Tune Denominators to test Penalty logic. 

Adjustment factor = 1 – Penalty(Mod(pos)/PL): 
Symmetric Order Size adjustment Based on distance from PL with a set parameter.

Market Regime Switching:  Volatility Assessment 


 Strategy 1: Total PNL around 3500


FOR KELP: 
Default: Bid = round(VWMP) - atr//2, Ask = round(VWMP) + atr//2
Liquid: Bid = round(VWMP) - atr//1, Ask = round(VWMP) + atr//1
Non-Liquid with no trades in last 300: Bid = round(VWMP) - atr//1, Ask = round(VWMP) + atr//1

FOR RESIN: 
Default: Bid = round(OU) - atr//2, Ask = round(VWMP) + atr//2
Liquid: Bid = round(VWMP) - atr//1, Ask = round(VWMP) + atr//1
Non-Liquid with no trades in last 300: Bid = round(VWMP) - atr//1, Ask = round(VWMP) + atr//1

Strategy 2: Total PNL around 2800

FOR KELP: 
Default: Bid = round(VWMP - atr/2), Ask = round(VWMP + atr/2) 
Liquid: Bid = round(VWMP - atr/1), Ask = round(VWMP + atr/1)
Non-Liquid with no trades in last 300: Bid = round(VWMP - atr/1), Ask = round(VWMP+ atr/1)

FOR RESIN: 
Default: Bid = round(OU - atr/2), Ask = round(VWMP + atr/2)
Liquid: Bid = round(OU - atr/1), Ask = round(OU) + atr/1)
Non-Liquid with no trades in last 300: Bid = round(VWMP - atr/1, Ask = round(VWMP + atr/1)




TESTING WITH SAEID: 
1. KELP(SEPERATE VS TOGETHER) 
2. RESIN(SEPERATE VS TOGETHER)

Kelp: 
	• Base Trade Size 
	• ATR period
	• VWMP Pop timeframe
	• Agro-Denom + Penalty 
	• Max(ATR,1)*****

Resin: 
	• Theta 
	• ATR period
	• Base Trade Size 
	• Agro-Denom + Penalty 
	• Max(ATR,1)****
	
	

Consider: 






1. Algorithm challenge
The first two tradable products are introduced: STARFRUIT and AMETHYSTS. While the value of the AMETHYSTS has been stable throughout the history of the archipelago, the value of STARFRUIT has been going up and down over time. Develop your initial trading strategy and write your first Python program to get off to a good start in this world of trading and market making. Even if the price of a product moves very little or in a very unpredictable way, there might still be clever ways to profit if you both buy and sell.
Position limits for the newly introduced products:
	• STARFRUIT: 20
	• AMETHYSTS: 20

2. Algorithm challenge
The first two tradable products are introduced: STARFRUIT and AMETHYSTS. While the value of the AMETHYSTS has been stable throughout the history of the archipelago, the value of STARFRUIT has been going up and down over time. Develop your initial trading strategy and write your first Python program to get off to a good start in this world of trading and market making. Even if the price of a product moves very little or in a very unpredictable way, there might still be clever ways to profit if you both buy and sell.
Position limits for the newly introduced products:
	• STARFRUIT: 20
	• AMETHYSTS: 20

3. Algorithm challenge
Cupid has landed in our archipelago and infected almost every inhabitant. Next to the products from the previous two rounds, the GIFT_BASKET is now available as a tradable good. This lovely basket contains three things:
	1. Four CHOCOLATE bars
	2. Six STRAWBERRIES
	3. A single ROSES
All four of the above products can now also be traded on the island exchange. Are the gift baskets a bit expensive for your taste? Then perhaps see if you can get the contents directly.
Position limits for the newly introduced products:
	• CHOCOLATE: 250
	• STRAWBERRIES: 350
	• ROSES: 60
	• GIFT_BASKET: 60

4. Algorithm challenge
Our inhabitants are crazy about coconuts. So crazy even, that they invented a new tradable good, COCONUT_COUPON. The coupons will give you the right to buy COCONUT at a certain price by the end of the round and can be traded as a separate good. Of course you will have to pay a premium for these coupons, but if you play your coconuts right, SeaShells spoils will be waiting for you on the horizon.
Coconut Coupons give the right to buy Coconut at a certain price at some time in future. Certain price = 10000 and some time in future = 250 trading days, each round is 1 trading day.
Position limits for the newly introduced products:
	• COCONUT: 300
	• COCONUT_COUPON: 600

5. Algorithm challenge
The final round of the challenge is already here! And surprise, no new products are introduced for a change. Dull? Probably not, as you do get another treat. The island exchange now discloses to you who the counterparty is you have traded against. This means that the counter_party property of the OwnTrade object is now populated. Perhaps interesting to see if you can leverage this information to make your algorithm even more profitable?

class OwnTrade:
    def __init__(self, symbol: Symbol, price: int, quantity: int, counter_party: UserId = None) -> None:
        self.symbol = symbol
        self.price: int = price
        self.quantity: int = quantity
        self.counter_party = counter_party




1. cross-sectional regression specifications to forecast skewness in returns 

two approaches to measure crash risk: 


first measure: is the negative conditional return skewness (NCSKEW), which is measured as the inverse of the third central moment of Ink's return over time N scaled by the variance of Ink's return over time N raised to 3/2. 






second measure: is the down-to-up volatility (DUVOL), which differs the former as DUVOL does not involve the third movement mentioned above. This method is more preferable since it can reduce the influence from a the bias of extreme movements. This method is to calculate the log of the ratio of the standard deviation on the down days to the standard deviation on the up days, which is actually a measure of crash likelihood. 







PTPI calculated by trading value outperforms the others since it only becomes volatile before and within each crash. there is no any volatility (or signal) in the non-crash period.

investigate the relationship between the volatility of PTPIs and the days of there is crash after occurrence of each signal: 

Signal determined by trading Value and Trading Prive

There is negative relationship between the value of the indicator’s volatility and the number of days until a crash. 

It is suggested that if the PTPI becomes more volatile, there is an increasing probability of the occurrence of a crash. These results strongly support that trading value is able to provide early warning signals than using trading volume and close price, which implies that trading value is the best proxy of liquidity in the stock market. 


Skewness to take account of momentum to forecast price movement
Pairs trading, doing linear regressions to identify statistical arbitrage
Day 1 (07/04/2025): 

Saeid : 
I finished the volatility targeting. Volatility Targeting acts as a "risk manager" module alongside the trader module.
I also summarized all the IMC2 Manual trading in the Google sheet.

James:
We went through the code of "commented . Py". 
We have to find a way to orchestrate running the trader.py alongside the backtester.py

Minjae : 
ARMA – The stuff that is already out there is good, we have to take snippets of each it would be perfect. 
He got a referral code from somebody in IMC => Basically we're in spotlight.

Mark :  
Look at previous winners. Looked at MM companies. 
	• 2 assets : stable and volatility
	• There is also a question about arbitrage.
	• Also there is options : Market making for short-term alpha (Stoikov) . Looking at volatility but adjusting inventory with Gamma, Beta?!
Raymond: 
Try to grid-search the other teams approach (commented---.py)
We have to make our back-tester to work.

===============================================================
Day2 – 08/04/2025

James and Minjae:
They found out that the returns are moving in the same bound. They are going to find a relationship on top of it. James is going to work on ECM with Saeid.


Raymond:
He implemented Sakarov(?) model. He is working on the risk aversion and volatility at the moment.
He implemented a pair trading which is bad at the moment but progressing.
Maybe he works on SMA/LMA to see if it works.

Saeid
Trying to implement ECM and find a way for crash detection module and also "clearing inventory module"


Round 1 : James and Saeid – 19:26: 

ECM Viability Condition: I(1) [Not stationary at current state but Stationary at a differenced Time]
Confident that this condition is satisfied from testing: [Pure data, Log Returns]


Trading Logic:
 
1. Run Error Correcting Model on (either Xi on Yi or Yi on Xi) for last N timesteps. 
2. Run Phillips-O on the Residuals for last M time steps.  
2A. [If stationary -> infer Theo of dependent variable based on independent variable previously used in the ECM -> Stat Arb. 
2B. [If non-stationary -> Switch Regime to prevent shock affect]

Dependent Variable Signal (Take theo as our FV) : Trade Theo z-score
Independent Variable Signal (set a good clasic FV model): ATR on a VWMP or some other Momentum Trading Stategy???
Stable Stock Signal (10,000): Stanford cardinal stealing/Semi-OU+ATR/or Idk 


Need to find: 
	• Grid search for N [for more accurate ECM calc]
	• Grid search for M. [More sudden the shocks, the higher our M]
	• Which product is our dependent and independent to max profits and protect from Ink Shocks. [Prediction: Use Ink as our independent as its volatility will make us more competitive]
	• Determining which regime we would want to use when Co-integration relationship is lost (Incoming Ink Shock)


Considerations:  Using VECM - Combines VAR with ECM: meaning that should a Lead Lag Dynamic be apparent, we can incorporate that into our prediction for independent variable IFF the independent variable is the lagger. 

How can we detect a ink crash? 
	• Kalman? [Hard to Implement]
	• Johanson IFFFFFFF lead lag dynamic exists [Condition 2 look more tho and also vector autoregression is memory intensive at every timestep?] maybe with a VECM? But Once again probably memory intensive 
	• Error Correction term -> when alpha becomes insignificant [this is inference]
Want to try some sort of evaluation tecnique for pattern detection of the orderbook priror to crash. Suspect that orderbook patterns might be consistent and predictive and hence can be used to also predict size of the crash? 
	



	1. We all need to read through Saeid's List of previous competitors (& perhaps also current competitors)
	2. We need to find previous years datasets or other relevant datasets to backtest against.
We've got Jaspers backtester, data repo, SP500, options data

	1. We need to build ideas on how to best solve the challenges for each round
		a. Currently going through rounds
	2. Note down everything and anything you find relevant in each of the "Ideas - Name" Pages
	
	
Manual trading might be significantly more profitable
	
	
Ideas on rounds

Goal 1: profitability
Goal 2: speed

Market making round
Assumption: round 1 will be the same as the tutorial round
Play around with undercuts for the resin ( stable one )
Tweak the kelp pricing model.
Considerations: 
There may be changes in regime, there may be periods where inventory is bring us down, there may be periods where shorting is more profitable to going long.
Use hidden markov model

Options round
Option pricing - pricing model with numpy 
Build volatility surface, if the smile is above sell, if below then buy
Solutions for movements in the markets. 
Adjust to volatility smile

Arbitrage round
Considering "Orchids round", transport, fees.

Considerations:
Some indicators may be red herrings
2nd last year, imported from other island and selling at ludicrous prices at own island.
Consider whether you'll hedge or not


Additional considerations:
Check for skewness for momentum
	

<img width="1504" height="602" alt="image" src="https://github.com/user-attachments/assets/41f82867-c3d1-46c9-8455-2d659d882861" />
<img width="1504" height="806" alt="image" src="https://github.com/user-attachments/assets/714c1fee-f932-4891-9b0e-03f617e83079" />
<img width="1028" height="944" alt="image" src="https://github.com/user-attachments/assets/5e8d959f-2d74-4546-9560-079abc1884ab" />



<img width="740" height="4488" alt="image" src="https://github.com/user-attachments/assets/b3c06ac4-7f4e-4bc2-a2c2-4775ce7a917c" />



