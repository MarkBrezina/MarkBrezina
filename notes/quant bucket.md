* 

* Uses a cross / recent-book averaging approach
* Stores old bids and asks
* Computes weighted average bid and ask from recent cached books
* Trades if the current book is favourable relative to these averages by some threshold
* Uses EMA as a dynamic fair value
* Adjusts quotes based on current inventory:
   * flat position: symmetric around EMA
   * long position: shifts quotes downward to encourage selling / reduce further buying
   * short position: shifts quotes upward to encourage buying back

So this team builds inventory-aware quoting directly into the pricing.

* Uses a hard-coded regression forecast on recent cached mid-prices
* Predicts next price from a rolling banana cache
* Converts that forecast into lower/upper acceptable prices
* Then trades and quotes around those bounds

So BANANAS here is the most explicitly forecast-driven of the three.
 
 
 
 
 
 
 Stable asset with a near 100% stable mid price around K.
There will be occasional mispricings, where either bid or ask will move to meet the mid price at K.
If we can assume that this is a simple asset utilised by IMC to get everyone started. The behaviour is simply continuously straightforward. We simply want to market make and market take on given opportunities.
 
For future we want to do
 

1. Market making
2. Market taking in accordance with fair value
3. Inventory skew to de-risk

 
 

* Uses a fixed fair value of 10,000
* Simply buys below fair value and sells above fair value
* Very straightforward mean-reversion / fair-value taking logic
* Posts both sides around the default price:
   * buy at 9999
   * sell at 10001
* This is more of a pure market-making setup than the first team

 
 

* Uses explicit acceptable bid/ask bounds around 10,000
* Aggressively combines:
   * market taking when quotes are favourable
   * market making by undercutting current best price
   * inventory-dependent quote adjustment

This is much richer execution logic than the first two teams.
 
 
 All of them store some form of historical information:

* cached bids/asks
* past prices
* EMA
* rolling banana price cache
* PnL and trade history

So they are not purely reactive to the current book; they are stateful strategies.

* Tracks:
   * positions
   * PnL
   * cash
   * EMA prices
   * past prices
* Keeps extensive state:
   * positions
   * traded volume
   * per-product cash PnL
   * rolling caches
   * signals from market participants
   * multiple product-specific parameters

 
 
 
 
 
 
 
 profit-seeking is always constrained by risk/inventory bounds.
trading edge and breadth.
 
 
 
 
Even the more advanced code is still built from understandable pieces:

* fixed fair value
* moving averages / EMA
* short rolling regression / cached-price forecast
* simple undercutting of current best bid/ask
* inventory skew

So the common style is practical and competition-oriented: simple models with direct execution rules.

* Logic is split into reusable methods like:
   * reset_from_state
   * buy_product
   * sell_product
   * continuous_buy
   * continuous_sell

 Current position
Inventory control.
Every implementation keeps track of current position and ensures it does not exceed the allowed max long or short positionGoal
Harvest relative mispricings while staying neutral to known risks (market, sector, style), controlling turnover, and being honest about costs.
A. Universe & data hygiene (stat arb lives or dies here)

* Liquid universe only (ADV, spread, borrow constraints if shorting).
* Survivorship-bias-free data, corporate actions, delisting returns.
* Intraday data helps, but you can do robust daily stat arb too.

B. Risk model + neutralization (the “spine”)
Build a factor model (daily or intraday):

* Exposures BBB: market beta, sector/industry, size, value, momentum, quality, low-vol, plus custom microstructure factors if you have them.
* Returns model: r=Bf+ϵr = B f + \epsilonr=Bf+ϵ

Estimate:

* Factor covariance Σf\Sigma_fΣf (shrinkage)
* Specific risk Σϵ\Sigma_\epsilonΣϵ (diagonal / clustered)

This lets you:

* Neutralize to factors (target B⊤x≈0B^\top x \approx 0B⊤x≈0)
* Size positions by predicted risk
* Avoid hidden factor bets

C. Alpha: residual mean reversion + “context signals”
Start with residuals:

* Fit returns (or prices) to factors and compute residual return ϵ^i(t)\hat{\epsilon}_i(t)ϵ^i(t).
* Construct a mispricing score:
   * zi(t)=EMA(ϵ^i,h1)−EMA(ϵ^i,h2)robust_scale(ϵ^i)z_i(t) = \frac{\text{EMA}(\hat{\epsilon}_i, h_1) - \text{EMA}(\hat{\epsilon}_i, h_2)}{\text{robust\_scale}(\hat{\epsilon}_i)}zi(t)=robust_scale(ϵ^i)EMA(ϵ^i,h1)−EMA(ϵ^i,h2)
   * Or cointegration/Kalman for pairs/baskets (more below).

Add context signals to avoid trading “cheap for a reason”:

* Liquidity/volume shock filters
* Vol regime filters (mean reversion dies in crisis correlation spikes)
* Earnings/news blackout windows (if equities)
* Short-term reversal + intraday microprice (if you have microstructure)

Ensemble of stat-arb alphas

* Cross-sectional residual reversion (broad)
* Industry/sector relative value (within clusters)
* Pairs/baskets (cointegration-based)
* Event mean reversion (post-earnings drift is not mean reversion; avoid mixing)

D. Structure: clustering + basket construction
Instead of naive pairs:

* Build clusters by industry + correlation of residual returns.
* Within each cluster, define baskets via:
   * PCA on residual returns
   * Graph methods (minimum spanning tree / community detection)
   * Or cointegration tests to find stationary spreads

Advanced pairs/baskets (Kalman filter)
Model spread state:

* yt=βtxt+ϵty_t = \beta_t x_t + \epsilon_tyt=βtxt+ϵt, with βt\beta_tβt evolving slowly
* Trade when the spread deviates from its filtered mean by a robust z-score.

This adapts hedge ratios through time (critical in real markets).
E. Portfolio construction: constrained mean-variance with costs
Solve:
max⁡x α⊤x − γ2x⊤Σx − κ⋅TC(Δx)\max_x \ \alpha^\top x \ - \ \frac{\gamma}{2} x^\top \Sigma x \ - \ \kappa \cdot \text{TC}(\Delta x)xmax α⊤x − 2γx⊤Σx − κ⋅TC(Δx)
Subject to:

* Dollar neutrality: ∑ixi=0\sum_i x_i = 0∑ixi=0
* Beta neutrality: b⊤x=0b^\top x = 0b⊤x=0 (and sector/style neutrality)
* Position bounds by liquidity / borrow
* Turnover constraints
* Gross exposure cap

This optimizer is the stat-arb “engine”. Your alpha can be simple; the constraints and costs make it real.
F. Execution: the hidden edge
Stat arb is turnover-heavy, so execution is not optional:

* Schedule trades to minimize impact (TWAP/VWAP with participation caps)
* Spread-aware limit orders for passive fills when signal half-life allows
* Cancel/replace logic to avoid adverse selection

G. Risk controls that separate pros from backtests

* Correlation spike detector: if cross-asset correlation jumps, reduce risk (stat arb correlation blowups are real).
* Crowding proxy: if many names move together in your book, cut exposure.
* Stop trading / widen entry bands in high vol regime.
* Kill-switch for borrow cost spikes / short squeezes.Goal
Harvest relative mispricings while staying neutral to known risks (market, sector, style), controlling turnover, and being honest about costs.
A. Universe & data hygiene (stat arb lives or dies here)
   * Liquid universe only (ADV, spread, borrow constraints if shorting).
   * Survivorship-bias-free data, corporate actions, delisting returns.
   * Intraday data helps, but you can do robust daily stat arb too.
B. Risk model + neutralization (the “spine”)
Build a factor model (daily or intraday):
   * Exposures BBB: market beta, sector/industry, size, value, momentum, quality, low-vol, plus custom microstructure factors if you have them.
   * Returns model: r=Bf+ϵr = B f + \epsilonr=Bf+ϵ
Estimate:
   * Factor covariance Σf\Sigma_fΣf (shrinkage)
   * Specific risk Σϵ\Sigma_\epsilonΣϵ (diagonal / clustered)
This lets you:
   * Neutralize to factors (target B⊤x≈0B^\top x \approx 0B⊤x≈0)
   * Size positions by predicted risk
   * Avoid hidden factor bets
C. Alpha: residual mean reversion + “context signals”
Start with residuals:
   * Fit returns (or prices) to factors and compute residual return ϵ^i(t)\hat{\epsilon}_i(t)ϵ^i(t).
   * Construct a mispricing score:
      * zi(t)=EMA(ϵ^i,h1)−EMA(ϵ^i,h2)robust_scale(ϵ^i)z_i(t) = \frac{\text{EMA}(\hat{\epsilon}_i, h_1) - \text{EMA}(\hat{\epsilon}_i, h_2)}{\text{robust\_scale}(\hat{\epsilon}_i)}zi(t)=robust_scale(ϵ^i)EMA(ϵ^i,h1)−EMA(ϵ^i,h2)
      * Or cointegration/Kalman for pairs/baskets (more below).
Add context signals to avoid trading “cheap for a reason”:
   * Liquidity/volume shock filters
   * Vol regime filters (mean reversion dies in crisis correlation spikes)
   * Earnings/news blackout windows (if equities)
   * Short-term reversal + intraday microprice (if you have microstructure)
Ensemble of stat-arb alphas
   * Cross-sectional residual reversion (broad)
   * Industry/sector relative value (within clusters)
   * Pairs/baskets (cointegration-based)
   * Event mean reversion (post-earnings drift is not mean reversion; avoid mixing)
D. Structure: clustering + basket construction
Instead of naive pairs:
   * Build clusters by industry + correlation of residual returns.
   * Within each cluster, define baskets via:
      * PCA on residual returns
      * Graph methods (minimum spanning tree / community detection)
      * Or cointegration tests to find stationary spreads
Advanced pairs/baskets (Kalman filter)
Model spread state:
   * yt=βtxt+ϵty_t = \beta_t x_t + \epsilon_tyt=βtxt+ϵt, with βt\beta_tβt evolving slowly
   * Trade when the spread deviates from its filtered mean by a robust z-score.
This adapts hedge ratios through time (critical in real markets).
E. Portfolio construction: constrained mean-variance with costs
Solve:
max⁡x α⊤x − γ2x⊤Σx − κ⋅TC(Δx)\max_x \ \alpha^\top x \ - \ \frac{\gamma}{2} x^\top \Sigma x \ - \ \kappa \cdot \text{TC}(\Delta x)xmax α⊤x − 2γx⊤Σx − κ⋅TC(Δx)
Subject to:
   * Dollar neutrality: ∑ixi=0\sum_i x_i = 0∑ixi=0
   * Beta neutrality: b⊤x=0b^\top x = 0b⊤x=0 (and sector/style neutrality)
   * Position bounds by liquidity / borrow
   * Turnover constraints
   * Gross exposure cap
This optimizer is the stat-arb “engine”. Your alpha can be simple; the constraints and costs make it real.
F. Execution: the hidden edge
Stat arb is turnover-heavy, so execution is not optional:
   * Schedule trades to minimize impact (TWAP/VWAP with participation caps)
   * Spread-aware limit orders for passive fills when signal half-life allows
   * Cancel/replace logic to avoid adverse selection
G. Risk controls that separate pros from backtests
   * Correlation spike detector: if cross-asset correlation jumps, reduce risk (stat arb correlation blowups are real).
   * Crowding proxy: if many names move together in your book, cut exposure.
   * Stop trading / widen entry bands in high vol regime.
   * Kill-switch for borrow cost spikes / short squeezes.
 
Assets	Cryptocurrencies vs Equities
Horizon	Ultra-low latency(sub-ms), Medium-freq(hours to days)
Exchanges	Binance, OKX, Coinbase, Robinhood
Tech	Python, C++ for speed
Finance	Leverage: flexible
Features	moving averages, volatility, order-flow imbalances, on-chain indicators, Momentum, reversal, carry, post-earnings-drift and low-volatility effects, PCA factor rotation, breakout detectors, or ML predictors, RSI/oscillator-based signals, transaction volume, whale movements, staking rates

Strategies	ULL
	Mid-freq
	• market making - quoting and rebalancing orders to capture bid-ask spreads and liquidity rebates. Position goals stay near zero.
	• cross-exchange / triangular arb
	• Order-Book Microstructure: Models that analyze order flow imbalance and momentum ignition to enter positions before large moves. Such strategies require sub-millisecond tick-by-tick processing.
	• Statistical arbitrage - Avellaneda–Lee s-scores
	• Pairs trading - ?
	
	
	real-time level-2 order-book imbalance, expected‐fill price and implied spread as primary predictors for 5 min–1 h signals. 
	
Practitioners filter spoofing by conditioning on filled orders only or using Bookmap-style depth-surfaces.
	
	Transformer variants (Informer, Autoformer, TimeXer) are now favoured for 15 m–1 d price forecasting
	
	momentum/trend, mean-reversion, LSTM/Transformer predictors, DeFi arb.
Long-term pairs trading, Machine Learning Models, Fundamental/On-Chain Signals, DeFi/DEX Arbitrage
Methods	Execution Engine: A fast, event-driven order manager. It receives “send order” commands from strategies and handles all API-level details (rate-limits, retries, smart routing). Critical code (order serialization, nonce management, latency-critical paths) can be written in C++ with Python bindings (e.g. via PyBind11) for speed. The engine tracks outstanding orders and confirmations.
	
	- Deep-representation models (Transformers, T-GAT, FinGPT)
	- Mean-field/broker games for order-flow toxicity
	- RL execution agents, dynamic Kelly sizing, rough-vol surfaces, Chebyshev interp.
	- Temporal Graph-Attention Networks better than GARCH at vol forecasts
	
	Machine Learning Module: Off-board model training (Python/TF or PyTorch) that updates predictive models online or at intervals. Examples: LSTM networks for short-term price movement​
Modern reinforcement-learning approaches (e.g. deep Q-networks) may also be explored for strategy development, though robust financial RL remains cutting-edge.


	• Graph-signal processing HAR-extensions embed realised-volatility vectors on a stock-correlation graph and transform them in the graph-spectral domain for next-day risk signals 

	• On-chain analytics for crypto: desks scrape funding rates, NVT-style utilisation ratios and liquidation clusters; tutorials at PrimeXBT summarise the standard metric set PrimeXBT.
Risk	Position Limits: No strategy can exceed its defined exposure.
	Stop-Loss / Take-Profit: Pre-set thresholds automatically exit trades to cap losses/profits​
	Portfolio-Level Checks: Monitors total drawdown and triggers system-wide safe modes if severe volatility occurs.
	Real-Time Alerts: Automated alerts/notifications if any risk rule is violated.
Architecture	data→ feature engine → strategy modules → C++ order manager → risk manager → monitoring/logging.
Plan	prototype → paper-trade → C++ optimisation → staged release → live risk dashboards.
Prototyping & Backtesting →Simulation & Paper Trading →C++ Optimization→Data Pipeline→Integration Testing→Monitoring Setup→Iterate & Tune




A Beginner's Step-by-Step Guide to Exploring Quantitative Trading with £50
Section 1: Introduction - Welcome to Quantitative Trading
What is Quantitative Finance and Trading? (Simplified)
Quantitative finance, often shortened to "quant finance," represents a field within finance that relies heavily on mathematical and statistical modeling to inform financial decisions. Instead of relying solely on intuition, news, or traditional financial analysis, quantitative approaches use data, algorithms, and computational power to identify opportunities, manage risk, and execute strategies. It's about applying scientific and mathematical rigor to the complexities of financial markets.   
Quantitative trading is a specific application of these principles, focusing on the development and execution of trading strategies based on quantitative analysis. These strategies aim to buy and sell financial assets—such as stocks, bonds, commodities, or cryptocurrencies—often in an automated or semi-automated fashion. The core idea is to leverage mathematical models to detect patterns, statistical relationships, or market inefficiencies that can be systematically exploited for potential profit. This involves rigorous testing of ideas using historical data (backtesting) before deploying them in live markets.   
What are High-Frequency (HFT) and Mid-Frequency Trading (MFT)?
Within the realm of quantitative trading, strategies are often categorized by their trading frequency or holding period. High-Frequency Trading (HFT) sits at the extreme end of this spectrum. It is a form of algorithmic trading characterized by:
	• Extraordinary Speed: Trades are executed in microseconds (millionths of a second) or milliseconds (thousandths of a second).   
	• High Volume: A vast number of orders are generated and executed throughout the trading day.   
	• Short Holding Periods: Positions are typically held for fractions of a second up to a few minutes, rarely overnight.   
	• Sophisticated Technology: HFT relies on powerful computers, complex algorithms, and high-speed data connections.   
A critical concept in HFT is latency. Latency refers to the time delay experienced when data travels from one point to another, such as from the exchange's matching engine to a trader's system, or the time taken for an order to travel to the exchange and be processed. HFT firms invest heavily to minimize latency, as even a tiny speed advantage can be crucial. This often involves co-location, which means placing their trading servers in the same physical data center as the exchange's servers to reduce the physical distance data needs to travel.   
Mid-Frequency Trading (MFT) occupies a space between HFT and traditional, slower trading strategies. While still automated and faster than human traders, MFT typically involves holding periods that might range from minutes to hours or even across a trading day. MFT strategies may focus less on pure speed arbitrage and more on analyzing factors like order flow (the sequence and size of buy and sell orders hitting the market), identifying short-term predictive signals, or reacting to specific market events or HFT-driven patterns. MFT generally requires less extreme low-latency infrastructure compared to HFT, though speed and efficient execution remain important.   
Setting Realistic Expectations: The £50 Journey
It is absolutely crucial to understand the realities of HFT from the outset. Executing genuine HFT strategies requires immense capital investment in specialized hardware, ultra-fast software, direct data feeds from exchanges, co-location services, and skilled personnel. These costs run into millions of pounds, making HFT fundamentally inaccessible for retail traders, especially those starting with a budget of £50.   
Therefore, the £50 budget allocated for this exploration should not be viewed as trading capital for HFT or even significant MFT participation. Instead, it should be considered a resource for learning and familiarization. This budget might cover access to an online course, purchase of an introductory book, or, much later, funding a tiny real-money account purely for understanding the psychological dynamics of trading.
The primary goal of this journey, given the constraints, is to build foundational knowledge. This involves understanding the core concepts of quantitative finance, learning the necessary mathematical and programming tools, exploring market mechanics, and simulating trading strategies in a risk-free environment. The objective shifts from doing HFT, which is impossible under these circumstances, to learning about HFT and the broader field of quantitative trading. The significant financial and technological barriers mean that simulation and knowledge acquisition are the only practical avenues available. This constraint fundamentally shapes the approach outlined in this guide, prioritizing education and experimentation over any expectation of profit from HFT activities.   
Section 2: Building Your Foundation
Before diving into complex strategies or specific tools, establishing a solid foundation in market basics, essential mathematical concepts, and programming is paramount. Attempting to understand advanced quantitative trading without these prerequisites is unlikely to be productive.
Essential Concepts: Markets, Assets, Orders
	• Markets: Financial markets are platforms where assets are traded. Key examples include: 
		○ Stock Markets: Exchanges like the New York Stock Exchange (NYSE) or London Stock Exchange (LSE) where shares representing ownership in publicly listed companies are bought and sold.   
		○ Cryptocurrency Exchanges: Platforms like Binance, Coinbase, or OKX facilitate the trading of digital assets (cryptocurrencies) based on blockchain technology.   
	• Assets: The items being traded. For this initial exploration, the focus will be on: 
		○ Stocks (Equities): Represent partial ownership in a corporation. Owning stock may entitle the holder to dividends (a share of company profits) and capital appreciation if the stock price increases.   
		○ Cryptocurrencies: Digital or virtual currencies secured by cryptography, operating on decentralized networks (like blockchains). Examples include Bitcoin (BTC) and Ethereum (ETH). While many other asset classes exist (e.g., commodities, derivatives ), stocks and crypto provide accessible starting points.   
	• Order Types: Instructions given to a broker or exchange to execute a trade. Two fundamental types are: 
		○ Market Order: An instruction to buy or sell immediately at the best currently available market price. It prioritizes speed of execution over the exact price achieved.   
		○ Limit Order: An instruction to buy or sell only at a specified price or better. A buy limit order executes only at the limit price or lower, while a sell limit order executes only at the limit price or higher. Limit orders provide price control but do not guarantee execution if the market never reaches the specified price. They are essential tools in quantitative strategies for precise entry and exit points.   
	• Limit Order Books (LOB): This is the electronic list maintained by an exchange showing all active limit orders for a specific asset. It displays the quantity of shares or tokens available to be bought (bids) and sold (asks/offers) at various price levels, ranked by price. The LOB provides transparency into market depth (supply and demand at different prices) and is a critical data source for many HFT and MFT strategies, such as market making, which involves simultaneously posting bid and ask orders.   
	• Basic Trading Ideas: 
		○ Long vs. Short: Going "long" means buying an asset hoping its price will rise. Going "short" involves selling an asset (often borrowed) expecting its price to fall, aiming to buy it back cheaper later. Short selling is more complex and carries different risks.   
		○ Diversification: The principle of spreading investments across different assets or strategies to reduce overall risk. The idea is captured by the adage "don't put all your eggs in one basket".   
Key Skills: Introduction to Probability & Statistics
Quantitative trading is fundamentally based on mathematics. Understanding probability and statistics is not optional; it's essential for developing, testing, and understanding trading models.   
	• Why Math Matters: Probability helps quantify the likelihood of future market movements or the success rate of a strategy. Statistics provides the tools to analyze historical data, identify patterns, measure risk, and determine if a strategy's performance is statistically significant or just due to luck.   
	• Learning Resources: 
		○ Khan Academy: Offers excellent, free introductory courses on basic probability (calculating probabilities, understanding independent and dependent events, sample spaces) and statistics (mean, variance, distributions like the Normal distribution, hypothesis testing). These resources are ideal for building foundational understanding without initial cost.   
		○ Coursera: Provides more structured courses, often from universities, covering these topics within a financial context. Examples include the Wharton Quantitative Modeling course or other beginner-level finance specializations. These may involve costs but offer a more formal learning path.   
Learning Python for Finance
A programming language is needed to implement and test quantitative ideas. Python has become the dominant language in quantitative finance for several reasons:
	• Extensive Libraries: Python boasts a rich ecosystem of libraries specifically designed for data analysis (Pandas), numerical computation (NumPy), machine learning (scikit-learn, TensorFlow, PyTorch), and visualization (Matplotlib, Seaborn), significantly speeding up development.   
	• Readability and Ease of Learning: Python's syntax is generally considered cleaner and easier to learn compared to other languages like C++, making it more accessible for beginners.
	• Community Support: A large and active community means ample resources, tutorials, and support are available online.
The initial steps for learning Python specifically for finance typically involve :   
	1. Python Fundamentals: Grasping core concepts like variables, data types (integers, floats, strings, booleans), basic operators (arithmetic, comparison, logical), and control flow (if-else statements, loops).
	2. Data Structures: Learning to work with lists (ordered, mutable collections) and potentially dictionaries (key-value pairs).
	3. NumPy: Understanding NumPy arrays, which are fundamental for efficient numerical operations on large datasets. This includes array creation, indexing, slicing, and mathematical functions.   
	4. Pandas: This library is crucial for financial data analysis. Learning to use Pandas DataFrames for importing, cleaning, manipulating, and analyzing time-series data (like stock or crypto prices) is essential.   
	5. Matplotlib: Acquiring basic data visualization skills using Matplotlib to create plots like line charts (for prices over time), histograms (for return distributions), and scatter plots to visually explore data and results.   
Recommended Resources:
	• DataCamp: Offers interactive courses tailored to finance, such as "Introduction to Python for Finance" and tutorials covering algorithmic trading basics using Pandas and Matplotlib. Interactive platforms can be effective for hands-on learning.   
	• Other Platforms: Coursera , Udemy , Codecademy, freeCodeCamp, and numerous free online tutorials (e.g., official Python documentation, YouTube channels) provide alternative learning paths.   
Developing proficiency in basic probability, statistics, and Python programming forms the bedrock upon which any understanding of quantitative trading, especially HFT/MFT, must be built. These skills are non-negotiable prerequisites. Without this foundation, attempting to decipher or implement complex strategies, including those hinted at in advanced resources, will be ineffective. The learning process must start here.   
Section 3: Exploring the Landscape - Platforms and Tools
Once the foundational concepts and skills are being developed, the next step is to become familiar with the trading environment. This involves understanding the specific exchanges available and utilizing practice tools like demo accounts.
Introduction to Your Exchanges
The query specified access to several platforms, each with its own characteristics:
	• Binance, OKX, Coinbase: These are large, global cryptocurrency exchanges. They offer trading in a wide variety of cryptocurrencies (spot trading) and often provide access to more complex products like futures and options (derivatives). They are known for high liquidity (ease of trading without significantly impacting the price) and comprehensive Application Programming Interfaces (APIs) for automated trading. Coinbase also offers a platform called Coinbase Advanced, targeted at more experienced traders with features like advanced charting and lower fees.   
	• Robinhood: This platform gained popularity, particularly in the US, for its commission-free trading model, primarily accessed via a mobile app. It offers stocks, Exchange-Traded Funds (ETFs), options, and some cryptocurrencies. While user-friendly for beginners, it's important to be aware of regulations like the Pattern Day Trading (PDT) rule in the US, which applies to margin accounts and can restrict frequent trading if account equity is below $25,000. * **Saxo Bank:** This is a more traditional online brokerage based in Denmark, offering access to a broader range of global markets and asset classes, including stocks, forex (foreign exchange), Contracts for Difference (CFDs), futures, options, and bonds. It provides multiple platform options (SaxoInvestor for beginners, SaxoTraderGO/PRO for advanced traders) and integrates with third-party tools like TradingView. Fee structures and account minimums may differ from crypto-native exchanges or commission-free brokers. ### The Power of Practice: Paper Trading / Demo Accounts Given the complexity and risks of trading, especially quantitative trading, practicing in a simulated environment is essential before committing real capital. This is achieved through paper trading or demo accounts. * **Definition:** A demo account provides a simulated trading environment that mimics live market conditions but uses virtual (fake) money. Trades are placed based on real-time or slightly delayed market data, allowing users to experience the trading process without financial risk. * **Essential Benefits:** * **Risk-Free Learning:** The most significant advantage is the ability to experiment and make mistakes without losing real money. * **Platform Familiarization:** Users can learn the layout and functionality of a trading platform, including how to place different order types (market, limit), use charting tools, monitor positions, and manage a virtual portfolio. * **Strategy Testing:** Demo accounts allow traders to test their strategies under realistic (though not perfect) market conditions to see how they might perform. * **Emotional Preparation (Limited):** While it cannot fully replicate the stress of real trading, paper trading can help users become accustomed to market volatility and the experience of winning and losing trades, albeit without the real emotional weight. * **Important Limitations:** It's crucial to understand that demo accounts are not perfect replicas of live trading: * **Execution Simulation:** They often don't accurately simulate the order book dynamics. Orders might fill instantly at the trigger price, ignoring factors like available liquidity (volume at a given price) or slippage (the difference between the expected fill price and the actual fill price), which are significant in real markets, especially for large orders or fast-moving conditions. * **Latency:** Demo systems may not replicate the real-world latencies involved in order routing and execution, making them unsuitable for testing true HFT strategies. * **Psychological Differences:** Trading without real money at stake removes the powerful emotions of fear and greed, which significantly influence decision-making in live trading. Success in a demo account doesn't guarantee success with real capital. * **System Priority:** Demo account performance might be de-prioritized compared to live trading systems during periods of high market activity. ### Table 3.1: Demo Account Availability on Specified Platforms To facilitate the choice of a platform for initial practice, the following table summarizes the demo account availability and features for the specified exchanges, based on the gathered information. This consolidation provides a quick reference to identify suitable starting points for risk-free exploration. | Platform | Demo Account Available? | How to Access | Asset Types Supported (in Demo) | Duration/Limits | Key Notes/Sources | | :------------ | :---------------------- | :---------------------------------------------------------------------------- | :------------------------------------- | :------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | | **Binance** | Yes | Via Binance Spot Test Network (testnet.binance.vision); requires GitHub login | Crypto (Spot) | No explicit duration mentioned; uses separate API keys | Simulates spot trading; no real order book simulation; separate from main Binance account | | **OKX** | Yes | Via "Demo Trading" mode within the main platform interface | Crypto (Spot, Futures, Options likely) | Virtual funds provided ($ simulated); can add more | Designed to closely replicate real trading features; good for practicing various crypto instruments | | Coinbase | No (Directly) | N/A (Simulation possible via third-party platforms like QuantConnect) | Crypto (via external simulation) | N/A | Coinbase platform itself focuses on live trading; paper trading requires connecting API to external simulation environments | | Robinhood | No | N/A | N/A | N/A | Platform does not offer a paper trading or demo account feature | | Saxo Bank | Yes | Via specific Demo Account sign-up on their website | Stocks, Forex, CFDs, Futures (likely) | 20 days | Provides $100,000 virtual cash; uses SaxoTraderGO and SaxoTraderPRO platforms |   
Based on this comparison, for practicing the cryptocurrency trading mentioned in the query, OKX and the Binance Spot Test Network appear to be the most suitable starting points among the listed options, as they offer dedicated crypto demo environments. Saxo Bank provides a demo account for traditional assets and derivatives but it is time-limited. Coinbase and Robinhood lack native paper trading functionalities, making them less ideal for initial risk-free learning and experimentation directly on their platforms. The choice between OKX and Binance Testnet might depend on user preference for interface or the desire to simulate futures trading (which OKX demo likely supports more directly).
Section 4: Your £50 Action Plan: Getting Started
This section outlines a practical, step-by-step plan designed for a beginner to start exploring quantitative trading concepts with a £50 budget, focusing on learning and simulation.
Step 1: Choose a Foundational Learning Path
	• Recommendation: Begin with freely available, high-quality resources to maximize the budget. 
		○ Mathematics: Utilize Khan Academy for foundational understanding of probability and statistics. Work through modules covering basic probability rules, sample spaces, distributions, and descriptive statistics.   
		○ Programming: Start learning Python basics using free online tutorials, documentation (python.org), or the free tiers of platforms like DataCamp or Codecademy. Focus initially on syntax, data types, control structures, and functions.   
	• Structured Courses (Optional): If the budget allows after considering potential minimal real-trade exposure later, or if a more structured approach is preferred, consider enrolling in an introductory online course. Platforms like Coursera offer relevant beginner courses in finance, accounting, investing, or quantitative modeling. Prioritize courses that build a strong conceptual understanding rather than promising complex trading strategies immediately.   
Step 2: Set Up a Demo Account
	• Recommendation: Based on the analysis in Section 3, start with a platform offering accessible crypto paper trading. The OKX Demo Trading mode or the Binance Spot Test Network are recommended choices.   
	• Access Instructions (Simplified): 
		○ OKX: Log into the OKX platform, navigate to the "Assets" or equivalent menu, and select the "Demo Trading" option to switch to the simulated environment. Virtual funds should be automatically allocated.   
		○ Binance Spot Test Network: Visit the testnet website (testnet.binance.vision), log in using GitHub credentials, navigate to the API key section, and generate a new HMAC_SHA256 key, saving the key and secret securely. This key pair would then be used to connect a compatible application (like potentially a self-written Python script later, or a platform like QuantConnect set to paper trading mode using Binance data).   
Step 3: Make Your First Paper Trades
	• Objective: The goal here is purely mechanical understanding and platform familiarization, not generating virtual profit.
	• Exercises: 
		○ Choose a major cryptocurrency like Bitcoin (BTC) or Ethereum (ETH).   
		○ Practice placing a Market Order to buy a small virtual amount. Observe how quickly it fills and at what price.   
		○ Practice placing a Limit Order to buy below the current market price. Observe if it sits in the order book and if it executes when/if the price drops to that level.   
		○ Practice placing a Sell Limit Order above the current market price and observe its behavior.   
		○ Familiarize yourself with the platform's interface for viewing open orders, trade history, and portfolio balance.   
		○ Explore any available charting tools within the demo environment to view price history.
Step 4: Basic Python Practice
	• Integration: Apply the Python skills learned in Step 1 in parallel with paper trading exploration.
	• Exercises: 
		○ Find sample historical price data online (many sources offer free CSV files for major assets) or create simple Python lists representing price movements.
		○ Use the Pandas library to load this data into a DataFrame.   
		○ Calculate daily or hourly simple returns (e.g., (price_today - price_yesterday) / price_yesterday).
		○ Use Matplotlib to create a basic line plot of the price data to visualize trends.   
Step 5: (Optional & Cautious) Using the £50 for Real Exposure
	• Important Disclaimer: This step should only be considered after significant practice in a demo account and a clear understanding of the risks involved. Reiterate that the £50 budget is insufficient for serious trading, especially HFT, and any funds used are at high risk of loss. This is an educational exercise in psychology, not an investment strategy.
	• Recommendation: If deemed appropriate for learning, allocate a very small portion of the budget (e.g., £10 - £20) for this exercise. Choose a platform with low minimum deposit requirements and low trading fees (Binance or OKX might be suitable, but verify their current terms ).   
	• Activity: 
		○ Deposit the small amount into the chosen live account.
		○ Select a major, highly liquid cryptocurrency (e.g., BTC or ETH).
		○ Place one or two very small limit orders based on simple criteria learned during practice.
		○ Observe: Pay close attention to the execution process, the fees charged, and, most importantly, the emotional response to having real money at stake. Note feelings of anxiety, excitement, fear, or greed.   
		○ Treat any outcome (profit or loss) as an educational experience. The primary value is understanding the psychological dimension that paper trading cannot fully replicate. This bridge between simulation and reality, even on a micro-scale, provides a valuable lesson, reinforcing the need for discipline and risk management. However, this step must be approached as paying for a lesson, not as an attempt to generate returns.   
Section 5: Understanding the Provided Resources (COPtoLON Repos)
The query specifically requested incorporating information about four GitHub repositories associated with the user COPtoLON (Mark Brezina ). It's important to assess their content and relevance for a beginner embarking on this learning path.   
Overview of the Repositories
	• COPtoLON/WTech: This repository is explicitly labeled as an "Algorithmic trading repository". Its content, based on the README and file structure glimpses, includes notes and potential code related to: 
		○ Strategy Development: Concepts like defining an "ideal state," detecting market states (trend, risk), matching tactics to states, and considering opponent positions.   
		○ Execution: Notes on optimal execution considering market impact, timing, and cost.   
		○ Advanced Topics: Mentions of leverage, options, futures, multi-asset trading, HFT market making, and medium-term horizons.   
		○ Research Areas: Lists numerous advanced research topics, including Markov models, mean reversion analysis, convergence theorems, machine learning models (autoencoders, MLP, LightGBM, XGBoost, CatBoost), specific cross-validation techniques for time-series data, mean field games, and event-based HFT.   
  
	• COPtoLON/Knowledgebase: Pinned on the user's profile and described as "A collection of information for quantitative finance, insurance, mathematics and AI". This suggests it's a curated repository of notes, links, or documents relevant to these fields, likely Markdown files, similar in concept but distinct from GitHub's built-in organizational knowledge base feature.   
	• COPtoLON/KB: Also pinned but lacks a description in the provided materials. Given the common use of "KB" for knowledge base and the existence of command-line knowledge base tools like kb-manager , this might be a personal system for organizing notes, possibly using such a tool, or related to the Knowledgebase repo. Its exact content requires direct exploration.   
	• COPtoLON/TMRW: Pinned but no description or content details were available in the snippets. The name is ambiguous ("Tomorrow"? Acronym?). Its purpose and content need direct investigation on GitHub..   
Relevance for a Beginner
	• Assessment: The content visible, particularly in the WTech repository , clearly indicates that these resources are not designed as introductory educational material. They represent working notes, research ideas, and potentially code fragments from someone actively engaged in the field at a relatively advanced level.   
	• The concepts mentioned (e.g., hidden Markov models, autoencoders for feature generation, specific time-series cross-validation methods, mean field games) require a significant prerequisite understanding of mathematics, statistics, machine learning, and financial markets, far beyond the scope of the foundational learning outlined in Sections 2 and 4.
	• Conclusion: These specific GitHub repositories are unsuitable as starting points for a complete beginner with no prior background in quantitative finance or programming. Attempting to learn from them directly would likely lead to confusion and discouragement.
How to Approach Them Later in Your Journey
While not suitable for initial learning, these repositories can still hold value later in the educational process:
	• Bookmark for Future Reference: Suggest saving the links to these repositories.
	• Revisit After Foundational Learning: Once a solid grasp of Python programming, statistics, basic trading principles, and perhaps introductory quantitative finance concepts (potentially through courses like those on Coursera or DataCamp) is achieved, revisiting these repositories could be beneficial.
	• Gain Context and Insight: WTech , in particular, can offer a glimpse into the types of problems, research questions, and techniques that practitioners in algorithmic trading explore. It illustrates the depth and complexity involved.   
	• Explore Curated Information: The Knowledgebase repository might contain valuable summaries, links, or references that become understandable and useful once the necessary background knowledge is acquired.   
These repositories should be viewed not as instructional guides but as potential aspirational examples or future exploration paths. They showcase the complexity and scope of quantitative and algorithmic trading. Their immediate value for a beginner lies in demonstrating the level of knowledge required in the field, rather than providing direct learning steps. They represent a destination one might work towards, not the starting line.   
Section 6: Diving Deeper - HFT/MFT Resources
After establishing foundational knowledge and gaining basic practical experience through simulation, the next phase involves delving deeper into the specific concepts relevant to High-Frequency and Mid-Frequency Trading.
Key Concepts Revisited
Understanding HFT and MFT requires familiarity with several core ideas:
	• Latency and Co-location: As previously mentioned, minimizing the time delay (latency) in receiving market data and sending orders is paramount, especially in HFT. Co-location, placing servers within the exchange's data center, is a primary method used by HFT firms to achieve the lowest possible latency. Understanding the physical and technological aspects of speed is crucial.   
	• Order Book Dynamics: HFT/MFT strategies often rely heavily on analyzing the Limit Order Book (LOB). This involves monitoring the flow of incoming orders, the depth of liquidity at different price levels, imbalances between bids and asks, and the bid-ask spread (the difference between the best bid and the best ask price). Strategies like market making directly interact with the LOB by posting simultaneous buy and sell limit orders to capture the spread. Order flow analysis attempts to predict short-term price movements based on the patterns of incoming orders.   
	• Market Microstructure: This academic and practical field studies the processes and rules governing trading in financial markets. It examines how prices are formed, how trades are executed, the role of different market participants (like market makers), the impact of order types, and the effects of exchange architecture on trading outcomes. A solid understanding of market microstructure is essential for designing effective HFT/MFT strategies.   
	• Common Strategy Archetypes (Brief Overview): 
		○ Arbitrage: Exploiting temporary price discrepancies of the same or similar assets across different markets or forms (e.g., buying a stock on one exchange and simultaneously selling it for slightly more on another). HFT arbitrage often focuses on minute, fleeting discrepancies requiring high speed ("latency arbitrage" ).   
		○ Market Making: Providing liquidity to the market by continuously quoting buy (bid) and sell (ask) prices for a security, aiming to profit from the bid-ask spread. This requires managing inventory risk.   
		○ Statistical Arbitrage ("Stat Arb"): Using quantitative models to identify short-term mispricings between related assets based on historical correlations or patterns.
		○ Event-Driven Trading: Designing strategies to react extremely quickly to specific news releases, economic data publications, or other market events.   
		○ MFT-Specific Strategies: These might include analyzing tick data (individual trades) for order flow patterns, "stop hunting" (attempting to trigger stop-loss orders), or exploiting HFT behaviors like front-running countermeasures.   
Recommended Reading/Learning
To deepen understanding of these concepts, consider exploring resources such as:
	• Investopedia: Provides accessible articles explaining key terminology and concepts related to HFT , latency , market making , arbitrage , and the limit order book.   
	• QuantStart: This website focuses on quantitative trading and often features articles and learning paths covering strategy identification, backtesting, and execution systems, potentially including introductory material relevant to HFT/MFT concepts.   
	• Quantra (QuantInsti): This platform offers specialized online courses in algorithmic and quantitative trading. Courses like "Trading in Milliseconds" specifically cover MFT topics such as market microstructure, latency types, order flow calculation (e.g., Lee-Ready algorithm), and MFT strategies. Be aware that such specialized courses typically involve costs.   
Understanding the Challenges (Reiteration)
While learning about HFT/MFT is valuable, it's vital to constantly remember the practical limitations for retail participation:
	• Insurmountable Barriers: The high costs of technology and data, the need for co-location, the sophisticated algorithms, data access limitations, and regulatory hurdles effectively preclude retail traders from competing directly in the HFT space.   
	• Technological Arms Race: HFT is characterized by a continuous "arms race," where firms constantly invest in faster technology and algorithms to maintain a competitive edge, making any advantage fleeting.   
	• Market Impact Concerns: HFT has faced criticism regarding its potential to exacerbate market volatility, contribute to "flash crashes" (sudden, severe price drops followed by a quick recovery), and create an uneven playing field.   
Given these realities, a retail trader's focus should adapt. Since direct participation in the ultra-low latency HFT game is not feasible, a more practical approach involves learning the concepts of HFT/MFT to understand market dynamics, but directing practical efforts towards areas where retail traders can potentially operate. This might involve developing algorithmic strategies on slightly longer timeframes (minutes, hours, days) where infrastructure costs and microsecond latency are less critical , or exploring MFT strategies that rely more on clever analysis (like order flow ) rather than pure speed, using the tools and platforms realistically available.   
Section 7: Next Steps and Continuous Learning
Embarking on the journey into quantitative trading is a marathon, not a sprint. It requires dedication, continuous learning, and a systematic approach.
Recap of the Learning Path
The recommended progression for a beginner starting with minimal capital and aiming to understand quantitative trading, including HFT/MFT, is as follows:
	1. Build the Foundation: Master basic probability, statistics, and Python programming.   
	2. Familiarize with Platforms: Use demo accounts (like OKX or Binance Testnet) to understand trading mechanics and platform interfaces risk-free.   
	3. Understand Trading Mechanics: Practice placing different order types and observing market behavior in simulation.   
	4. Apply Programming Skills: Use Python for basic financial data analysis and visualization.   
	5. (Optional) Gain Psychological Insight: Consider a very small, controlled real-money trade to experience the emotional aspect, accepting the high likelihood of loss as an educational cost.   
	6. Deepen Quant Knowledge: Study core quantitative finance concepts, market microstructure, and the theory behind different strategy types (including HFT/MFT) using resources like Investopedia, QuantStart, or specialized courses.   
	7. Explore Advanced Topics: Once fundamentals are strong, revisit more complex resources like the COPtoLON repositories or academic papers for deeper insights.   
Importance of Patience and Persistence
Quantitative finance is an intellectually demanding field. Progress requires significant time investment, consistent effort, and the resilience to work through challenging concepts and inevitable setbacks. There are no shortcuts to expertise, and profitability (especially in algorithmic trading) is never guaranteed. Approach it as a continuous learning process.   
Further Learning Avenues
As foundational knowledge grows, consider exploring these avenues:
	• Advanced Courses: Platforms like Coursera, edX, Quantra, and university programs offer more advanced courses in financial engineering, machine learning for trading, derivatives, and specific algorithmic trading techniques.   
	• Online Communities: Engage thoughtfully with online communities focused on quantitative finance and algorithmic trading (e.g., forums on platforms like QuantConnect , specific subreddits, or sites like Elite Trader ). Be critical of information and focus on learning rather than seeking "get rich quick" signals.   
	• Reputable Blogs and Research: Follow blogs from established quantitative finance professionals and researchers (some examples mentioned in ). Explore academic research repositories like SSRN or arXiv q-fin for cutting-edge ideas (though these are often highly technical).   
	• Backtesting Platforms: Eventually, learning to use a dedicated backtesting platform (e.g., QuantConnect , Zipline, Backtrader) becomes crucial. These platforms allow rigorous testing of trading strategies on historical data to assess potential performance and identify flaws before considering live deployment.   
Final Encouragement
The path to understanding quantitative trading is challenging but intellectually rewarding. While direct participation in HFT might be out of reach for a retail beginner with limited capital, the process of learning the underlying mathematics, programming, market mechanics, and strategic thinking provides a deep and valuable understanding of how modern financial markets operate. Frame this journey as an opportunity for intellectual growth and skill development, focusing on the knowledge gained along the way.
Conclusion
This guide provides a structured, step-by-step approach for a beginner with a £50 budget to begin exploring the world of quantitative finance, with a particular interest in High-Frequency Trading (HFT) and Mid-Frequency Trading (MFT). The analysis underscores several key conclusions and recommendations:
	1. HFT Accessibility: Direct participation in HFT is realistically impossible for a retail beginner with a £50 budget due to the prohibitive costs of technology, data feeds, and low-latency infrastructure.   
	2. Focus on Foundational Learning: The primary goal must shift from doing HFT to learning about quantitative trading. The £50 budget is best utilized for educational resources and simulation, not significant trading capital. Mastery of foundational mathematics (probability, statistics) and programming (Python with libraries like Pandas, NumPy, Matplotlib) is the essential first step.   
	3. Leverage Simulation: Paper trading (demo accounts) is a critical tool for risk-free learning of platform mechanics, order types, and basic strategy testing. Platforms like OKX and the Binance Spot Test Network offer suitable starting points for crypto simulation among the specified options.   
	4. Provided Repositories (COPtoLON): The specified GitHub repositories contain advanced notes and research ideas. They are not suitable for beginners but can serve as aspirational examples or resources for future exploration once foundational knowledge is established.   
	5. Realistic Path Forward: A practical progression involves building foundational skills, utilizing demo accounts extensively, potentially undertaking a minimal real-money trade for psychological insight (treating it as tuition), and then delving into deeper quantitative concepts and market microstructure. For applying algorithmic principles, focusing on MFT concepts or strategies on slower timeframes (minutes to days) is more realistic for retail participants than attempting HFT.   
Quantitative trading is a complex and evolving field. Success requires patience, continuous learning, and a rigorous, data-driven approach. By following a structured learning path and maintaining realistic expectations, a beginner can gain valuable knowledge and skills, building a strong foundation for future exploration in this fascinating domain.   
Sources used in the report



I hindsight can man godt finde den optimale policy
Lille Q læring, hvis appr. Af proces kan laves markov, hvis 7 sidste frame, så løses.

Reinforcement learning I kontinuert tid og rum.

Rough differential equations
[2306.14258] A Neural RDE approach for continuous-time non-Markovian stochastic control problems
Solving and Learning non-Markovian Stochastic Control problems in continuous-time with Neural RDEs | OpenReview
[2207.00713] q-Learning in Continuous Time

Fudan gruppe mht. Stoch control. Maximum principper, bellman ligner


	1. Markov states for "hvor er vi ved"

Model-baseret, forudse underlæggende dynamic - NN

Model-fri, PPO, soft, critic metoder -simuleret

Lille Q, regret bounds, kontinuert kalibrering og optimering, N^(3/4)
Decision transformer, signature regression, path signature, signature transform, 
Rough paths.
Feedforward, test, der hvor du har en edge.

Counter-factual learning, opdatere på det der skete og på det vi forventede ville ske med andre handlinger. Explainable AI.

Ikke arg max
Aksel Fristrup
12:35 PM
greedy policy
exploration
online learning
Aksel Fristrup
12:36 PM
exploration-exploitation tradeoff

From <https://meet.google.com/trr-ntbq-pcx> 

Entropy
Aksel Fristrup
12:36 PM
exploration-exploitation tradeoff
entropy regularisation
Aksel Fristrup
12:37 PM
renyi, tsallis
Aksel Fristrup
12:39 PM
reward shaping

From <https://meet.google.com/trr-ntbq-pcx> 

Selv-eksiterende
Neural-hawkes process
Parametrisere

hierarchical reinforcement learning
Aksel Fristrup
12:50 PM
H-MARL

From <https://meet.google.com/trr-ntbq-pcx> 

Credit assignment, hvad gjorde at jeg fik reward

onjecture 5 — The Final Problem
Portfolio · Optimisation · Multiscale
Modern financial firms are built as hierarchies of decision makers: firm → portfolio → strategy → execution. Each layer has its own objectives, constraints and key performance indicators, yet all of them act on the same underlying market and the same pool of capital.
Fix a single firm-wide portfolio A, with total capital V, trading over a universe of assets with associated features (prices, returns, volatilities, liquidity, funding, fundamentals and microstructure attributes). The portfolio A is decomposed into internal components:
	• strategy pods and sub-portfolios,
	• risk and hedging books,
	• execution and routing policies,
	• and any further sub-components (desks, regions, time-horizons).
Each component X in this hierarchy:
	• controls a local state (positions, risk exposures, cash, inventory),
	• faces its own constraints (risk limits, turnover, liquidity, leverage, drawdown, stealth, capital budget),
	• and optimises a local objective functional (e.g. Sharpe, PnL, risk-adjusted return, tracking error, execution cost).
At the same time, the firm as a whole optimises a global objective functional over the full configuration of all components: long-horizon risk–return, drawdown and survival probabilities, capital efficiency, and systemic risk.
Problem.
Specify a mathematical framework (objective, constraints and decision rules) that, given:
	• the hierarchical decomposition of the firm (components and sub-components),
	• the feature set of all traded assets and strategies, and
	• the evolving market state and information flow,
produces a configuration of decisions (capital allocations, positions, risk limits and execution policies) that is simultaneously:
	• globally optimal for the firm-wide objective (risk–return, survival, capital efficiency), and
	• locally optimal for every component and sub-component, given its constraints and its local view of the environment.
In other words, the same capital and information must support a configuration where the whole portfolio A is optimal inside the market, and each internal piece of A is optimal inside its own local neighbourhood of constraints and interactions.
The conjecture.
There exists a principled, implementable multiscale optimisation principle — a single rule, or variational framework — that maps the full state of the market and the firm (prices, returns, risks, liquidity, fundamentals, correlations and hierarchical structure) into such a configuration of decisions, and that:
	• applies uniformly across layers (firm, portfolio, strategy, execution),
	• respects all local constraints and risk budgets, and
	• yields configurations that are stable under feedback from market dynamics and from the actions of other agents.
The challenge is to write down this multiscale principle explicitly and to show that, under realistic assumptions on markets and constraints, its solutions are indeed simultaneously globally and locally optimal in the above sense.





I am formally proposing "Conjecture 5"—a challenge I view as the "Final Problem" of my work in market dynamics.

The problem: Fix a firm-wide portfolio A. Decompose it into strategy pods, risk books, and execution policies. Each component optimizes a local functional (Sharpe, PnL, execution cost) under local constraints. Simultaneously, the firm optimizes a global functional (survival, capital efficiency).

The Conjecture: There exists a principled multiscale framework—likely a variational principle akin to energy minimization —that yields a configuration of decisions that is simultaneously globally and locally optimal.

This aligns with the multi-layer framework we have developed, integrating:

Assets & Agents Graphs: To capture the diffusion of shocks and information.

Hybrid Control: To handle regime switches and boundary conditions at "jumps".

Risk-Aware Bellman Control: Driving action selection through a unified prediction-strategy-risk stack.

I dub this "The Final Problem" because I believe it is the concluding issue in constructing a truly self-learning financial architecture.

I’ve already proposed solutions internally that leverage these hybrid dynamics and layered value surfaces. I am curious to see who else is looking at the market through this lens.



hashtag#QuantitativeAnalysis hashtag#ControlTheory hashtag#FinancialEngineering hashtag#HFT hashtag#MarketDynamics


Goal
Capture persistent directional drift across assets while surviving regime shifts (crashes, choppy ranges) through ensembles + dynamic risk control + cost-aware portfolio optimization.
A. Data & universe
	• Multi-asset universe: futures (rates, equity index, FX, commodities) is ideal; equities/crypto also works.
	• Use continuous returns, robust rolls, corporate actions (if equities).
	• Compute tradeable vol and liquidity proxies (bid-ask/spread estimates, ADV, realized intraday range if available).
B. Trend signal: an ensemble of “trend estimators”
Use multiple, partially redundant estimators; average them with dynamic weights.
Core “trend strength” (risk-scaled)
For each asset iii, horizon hhh:
	• Price trend: mi,h(t)=EMAh(pi)(t)−EMAkh(pi)(t)σi,h(t)m_{i,h}(t) = \frac{\text{EMA}_h(p_i)(t) - \text{EMA}_{k h}(p_i)(t)}{\sigma_{i,h}(t)}mi,h​(t)=σi,h​(t)EMAh​(pi​)(t)−EMAkh​(pi​)(t)​
		○ (EWMAC style, with k∈[2,8]k\in[2,8]k∈[2,8] depending on how slow you want the anchor)
	• Or time-series momentum: mi,h(t)=ri(t−h,t)σi,h(t)m_{i,h}(t) = \frac{r_{i}(t-h,t)}{\sigma_{i,h}(t)}mi,h​(t)=σi,h​(t)ri​(t−h,t)​
Then squash to avoid crazy leverage:
	• si,h(t)=tanh⁡ ⁣(mi,h(t)c)s_{i,h}(t) = \tanh\!\left(\frac{m_{i,h}(t)}{c}\right)si,h​(t)=tanh(cmi,h​(t)​) with c≈2c\approx 2c≈2–4.
Ensemble across horizons
	• Typical horizons: 5–10, 20, 60, 120, 250 trading days (or equivalents).
	• Base combo: si(t)=∑hwh(t) si,h(t)s_i(t) = \sum_h w_h(t)\, s_{i,h}(t)si​(t)=∑h​wh​(t)si,h​(t)
C. Dynamic weighting of horizons (what makes it “advanced”)
Instead of fixed whw_hwh​, learn them online:
Option 1: Bayesian model averaging (fast + stable)
	• Each horizon “expert” has recent performance PhP_{h}Ph​ measured after costs.
	• wh(t)∝exp⁡(η Ph(t))w_h(t) \propto \exp(\eta\, P_h(t))wh​(t)∝exp(ηPh​(t)), with shrinkage toward equal weights.
Option 2: Regime-conditioned weights
	• Define a chop / trend regime using features like:
		○ Hurst proxy, variance ratio, ADX-ish measures, autocorr of returns, drawdown slope, realized vol slope.
	• If “choppy”: down-weight short horizons (they whipsaw); if “trend”: allow them.
D. Position sizing: volatility targeting + drawdown governor
Raw target risk per asset:
	• posi(t)=Rbudgetσ^i(t)⋅si(t)\text{pos}_i(t) = \frac{R_{\text{budget}}}{\hat{\sigma}_i(t)} \cdot s_i(t)posi​(t)=σ^i​(t)Rbudget​​⋅si​(t)
Then apply portfolio-level governors:
	• Vol target: scale all positions to hit portfolio vol (e.g., 10% annualized).
	• Drawdown control: when DD breaches thresholds, reduce risk smoothly:
		○ λ(t)=clip(1−α⋅DD(t),λmin⁡,1)\lambda(t) = \text{clip}(1 - \alpha \cdot \text{DD}(t), \lambda_{\min}, 1)λ(t)=clip(1−α⋅DD(t),λmin​,1)
	• Crisis overlay (optional): if equity vol spikes / correlation jumps, shift risk budget to diversifiers (rates, gold, FX) rather than “cash only”.
E. Portfolio construction: cost-aware convex optimization
This is where most “serious” systems differ from a toy trend model.
Choose weights xxx to maximize expected utility:
max⁡x μ⊤x − γ2x⊤Σx − κ⋅TC(x−xprev)\max_x \ \mu^\top x \ - \ \frac{\gamma}{2} x^\top \Sigma x \ - \ \kappa \cdot \text{TC}(x-x_{\text{prev}})xmax​ μ⊤x − 2γ​x⊤Σx − κ⋅TC(x−xprev​) 
Subject to:
	• Risk budget by asset class
	• Leverage caps
	• Concentration limits
	• Turnover limits
	• Optional: correlation-aware constraints (don’t double-load the same macro bet)
Key: estimate Σ\SigmaΣ with a robust covariance (shrinkage, factor model, EWMA with jumps).
F. Execution model (minimum needed for realism)
	• Trade scheduling: spread trades across day; cap participation rate.
	• Impact model: cost increases with ∣Δx∣/ADV|\Delta x|/\text{ADV}∣Δx∣/ADV and with volatility.
	• Slippage: penalize turnover and avoid trading in illiquid times.
G. Monitoring & kill-switches
	• Live drift: compare realized vs predicted risk and turnover.
	• Signal health: if “expert” performance collapses across many assets, reduce weight.
	• Structural break tests on costs/liquidity regime.Conjecture X — Existence of Optimal Strategies in the Full Multi-Layer Market Model
Hybrid geometry · Mean-field games · Optimal control
We model financial markets as a coupled, multi-layer system:
	• a time-varying market/asset graph and value surface,
	• a network of interacting agents with utilities and information links,
	• a rule hierarchy governing dynamics,
	• and (in the most general version) a geometric/hybrid frame where agents move on a manifold with continuous flows and discrete jumps.
In the simplified, finite, fully discrete version of this environment, the decision problem of a fully-informed agent reduces to a Markov Decision Process, and classical Bellman theory guarantees existence of an optimal policy, computable by dynamic programming.
The conjecture asks whether an analogous guarantee holds in the full geometric–informational, hybrid, multi-agent setting, where the state is infinite-dimensional (fields + agents), time is hybrid (flows + jumps), and interactions are game-theoretic / mean-field rather than a single MDP.

Informal statement
	Can we prove that, in the full Tomorrow Markets framework, there exists at least one optimal strategy (or equilibrium strategy profile) for agents, and characterize it in a reasonably “simple” class (Markov, feedback, or geodesic/variational policies)?
In other words: once you accept the market model as a hybrid geometric–informational multi-agent system, is it mathematically guaranteed that there is a best possible way to trade (or a best equilibrium configuration of strategies) consistent with the rules of the world?

Formal version (single optimal agent)
Let
	• XtX_tXt​ denote the global state (fields, graphs, agents) evolving as a hybrid dynamical system with flows and jumps on X=M×SX = M \times SX=M×S, where MMM is a (possibly time-varying) manifold and SSS indexes discrete modes/frames.
	• A distinguished “Tomorrow” agent observes XtX_tXt​ and chooses controls at∈A(Xt)a_t \in A(X_t)at​∈A(Xt​).
	• The agent’s monetary performance is
J(π)=E[∑t=0Tγt(R(Xt,π(Xt))−L(Xt,π(Xt)))],J(\pi) = \mathbb{E}\Big[\sum_{t=0}^{T} \gamma^t \big(R(X_t,\pi(X_t)) - L(X_t,\pi(X_t))\big)\Big],J(π)=E[t=0∑T​γt(R(Xt​,π(Xt​))−L(Xt​,π(Xt​)))], 
for 0<γ≤10<\gamma\le 10<γ≤1, with reward RRR and loss LLL defined by the full market model.
Conjecture (Existence of optimal strategy).
Under natural regularity conditions on:
	• the hybrid state dynamics (well-posed flows and jump maps on M×SM \times SM×S),
	• the action sets A(x)A(x)A(x) (compact, measurable),
	• and the reward/loss structure (bounded or suitably integrable, Markov in (Xt,at)(X_t,a_t)(Xt​,at​)),
there exists an optimal admissible policy π\*\pi^\*π\* such that
J(π\*)=sup⁡πJ(π),J(\pi^\*) = \sup_{\pi} J(\pi),J(π\*)=πsup​J(π), 
and π\*\pi^\*π\* can be chosen from a structured class, e.g.:
	• a Markov / feedback policy π\*(x)\pi^\*(x)π\*(x) depending only on the current state,
	• or a variational/geometric policy that realizes a least-action trajectory on the manifold (solution of an HJB / Euler–Lagrange system associated with the full model).
A stronger version asks for constructive existence: that π\*\pi^\*π\* can be obtained as the limit of a convergent sequence of algorithms (e.g. value iteration / policy iteration / actor–critic) implemented within the same market simulator.

Multi-agent / equilibrium extension
In the many-agent setting, each agent iii has its own objective Ji(π1,…,πN)J_i(\pi_1,\dots,\pi_N)Ji​(π1​,…,πN​) defined over the same hybrid geometric field, with coupling via prices, fields, and information graphs.
Extended Conjecture (Existence of geometric–informational equilibrium).
For the full multi-agent model (or for its mean-field limit), under suitable compactness/continuity/monotonicity assumptions, there exists at least one equilibrium strategy profile:
	• a Nash equilibrium in feedback strategies, or
	• a mean-field game equilibrium (solution of a coupled HJB–Fokker–Planck system on the manifold),
compatible with the hybrid dynamics and information network.

What “solving” the conjecture would look like
Any of the following would count as a serious solution:
	1. Existence theorem (single agent):
Precise assumptions + proof that an optimal feedback policy π\*\pi^\*π\* exists for the full hybrid geometric–informational model (not just the finite, discretized MDP case already covered in the drafts).
	2. Constructive characterization:
A scheme (e.g. an HJB on M×SM \times SM×S, or a convergent dynamic-programming / RL algorithm) that provably converges to π\*\pi^\*π\*.
	3. Equilibrium existence (multi-agent / mean-field):
A proof that, in the interacting-agent version, at least one Nash / mean-field equilibrium exists, formulated on the manifold with hybrid dynamics.
Bonus bonus points if the solution reveals structure (e.g. optimal policies are geodesic, layered across time-scales, or admit a nice decomposition across the strategy/prediction/risk modules already defined in the framework).



𝗜𝘀 𝘁𝗵𝗲 𝗺𝗮𝗿𝗸𝗲𝘁 𝗮 𝗠𝗮𝗿𝗸𝗼𝘃 𝗗𝗲𝗰𝗶𝘀𝗶𝗼𝗻 𝗣𝗿𝗼𝗰𝗲𝘀𝘀? 𝗢𝗿 𝗶𝘀 𝗶𝘁 𝗮 𝗛𝘆𝗯𝗿𝗶𝗱 𝗚𝗲𝗼𝗺𝗲𝘁𝗿𝗶𝗰 𝗦𝘆𝘀𝘁𝗲𝗺?
Most reinforcement learning in finance assumes the former. It’s neat, discrete, and solvable via dynamic programming.

At 𝗧𝗼𝗺𝗼𝗿𝗿𝗼𝘄 𝗖𝗮𝗽𝗶𝘁𝗮𝗹 𝗥𝗲𝘀𝗲𝗮𝗿𝗰𝗵, we operate on the latter. And that leads us to 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟰.

We model the market as a coupled, multi-layer system where the global state Xₜ evolves as a 𝗵𝘆𝗯𝗿𝗶𝗱 𝗱𝘆𝗻𝗮𝗺𝗶𝗰𝗮𝗹 𝘀𝘆𝘀𝘁𝗲𝗺 on a manifold M x S.
• Time flows continuously but jumps discretely.
• Agents interact via mean-field games, not static environments.
• Information propagates through a time-varying graph.

𝗧𝗵𝗲 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲: Once you accept this full "Tomorrow Markets" framework, is it guaranteed that an 𝗼𝗽𝘁𝗶𝗺𝗮𝗹 𝘀𝘁𝗿𝗮𝘁𝗲𝗴𝘆 (or equilibrium profile) actually exists?

We are looking for the existence of an admissible policy π*—potentially a geometric policy realizing a "least-action" trajectory—that solves the full HJB-Fokker-Planck system on the manifold.

Note: Because this problem sits at a uniquely interesting postion to us, 𝘄𝗲 𝗮𝗿𝗲 𝗻𝗼𝘁 𝗿𝗲𝗹𝗲𝗮𝘀𝗶𝗻𝗴 𝗮 𝗽𝗿𝗼𝗯𝗹𝗲𝗺 𝗣𝗗𝗙 𝗳𝗼𝗿 𝘁𝗵𝗶𝘀 𝗰𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲. The description on the website is the only public information available.

If you have an intuition for control theory on hybrid manifolds, we want to know.

👉 Full informal and formal statement here:🔗 https://lnkd.in/dPKxZcKc
@TomorrowCapital
hashtag#mathfinance hashtag#stochasticcalculus hashtag#HJB hashtag#geometrictopology hashtag#quantresearch hashtag#TomorrowCapitalResearch
hashtag#QuantitativeFinance hashtag#QuantFinance hashtag#AlgorithmicTrading hashtag#SystematicInvesting hashtag#MeanFieldGames hashtag#GameTheory hashtag#Optimization

From <https://www.linkedin.com/in/markbrezina95/recent-activity/all/> 

Conjecture 3 — Stealth-Optimal Execution Distributions
Execution · Market Impact · Stealth
Motivation
This conjecture has already attracted special interest from a couple of other founders.
The underlying problem is simple to state. Given an algorithmic trading strategy with strong returns, it is almost inevitable that capital will accumulate. As capital scales, order sizes and average position sizes grow — and firms consistently observe that returns deteriorate. A large part of this degradation comes from market impact and loss of execution stealth.
We are therefore looking at two sides of the same coin:
	• Market impact: how much our orders move prices against us.
	• Execution stealth: how detectable our trading footprint and intent become to other participants.
Big, obvious orders (for example a single multi-million buy sweep) tend to:
	• push the order book and short-term imbalance in our disfavoured direction, and
	• leave a visible footprint that other firms can learn from over time.
This can lead to alpha decay: competitors infer the structure of the strategy, trade ahead of it, or “vulture” around it with sandwich-like behaviour. Execution design is therefore not a side quest — it sits alongside inventory control, risk management, and alpha discovery as a core problem.
This conjecture asks for a principled way to design stealth-optimal execution distributions: how to spread a given order across time, size, and accounts to maximise stealth and minimise slippage / cost.
Informal problem statement
Suppose an algorithmic trading signal (alpha) fires and prescribes a net position change of notional size Q (e.g. Q = 1,000,000 USD) to be executed over an acceptable time horizon T = [0, τ], from signal time t = 0 to a hard time limit τ.
We want to determine:
	A joint distribution over execution times and order sizes (and, optionally, accounts) that maximises stealth and minimises slippage / cost over the horizon T.
The order flow here is not the entire portfolio inventory, but the total notional associated with this specific signal. The strategy may trigger repeatedly over time, either to build a position into an attractive opportunity or to gradually unwind into a profit-taking region.
Modelling set-up
We can model an execution schedule in either of two ways:
	• Discrete trades: a sequence of trades {(tk, qk)} with 0 ≤ tk ≤ τ and ∑ qk = Q.
	• Continuous order-size density: a function ϕ(t) on [0, τ] such that ∫0τ ϕ(t) dt = Q.
We assume, at minimum, the following microstructure ingredients:
	1. Impact / stealth trade-off (square-root visibility).
The “loss of stealth” or footprint associated with a trade of size q grows approximately as a square-root law, footprint(q) ∝ √|q|. Executing the full notional Q at t = 0 effectively destroys stealth; executing tiny slices (e.g. 10 USD) changes stealth only negligibly.
	2. Account-splitting constraint.
The trader may split Q across at most Nmax = 100 accounts, Q = ∑i=1N Qi with N ≤ Nmax, where each account i follows its own execution schedule. This rules out the degenerate “infinitely many infinitesimal accounts” solution.
	3. Execution cost / slippage.
For simplicity, each individual trade incurs:
		○ a base cost modelled as a fixed k% rate (covering immediate slippage and explicit fees), and
		○ a market impact component consistent with the square-root visibility law above.
Optionally, transaction costs may shrink by a factor c% as order size increases, capturing fee discounts for larger tickets.
	4. Cross-impact and opposite flow.
One may allow for inverse impact or partial cancellation when buy and sell orders of similar size interact in opposite directions over short horizons.
	5. Market “memory” / recycle rate.
The market (and counterparties) can be assumed to have a finite memory horizon: after some time, or after recycling accounts, previous order flow becomes less informative. For example, one could assume:
		○ up to 100 “fresh” accounts can be created per month, and
		○ each new account starts with a “stealth bonus” that decays with its cumulative activity.
Conjecture — Stealth-Optimal Execution Distributions
Under a reasonable microstructure model incorporating:
	• a square-root–type impact / detectability relationship in trade size,
	• optional cross-impact between opposing flows,
	• account-splitting constraints N ≤ 100, and
	• size-dependent transaction costs,
there exists an optimal joint distribution of:
	• execution times,
	• trade sizes, and
	• account splits (up to 100 accounts),
that:
	• maximises stealth — in the sense of minimising information leakage or detectability of the underlying strategy; and
	• minimises expected slippage and total execution cost relative to the alpha’s target price path,
subject to:
	• t ∈ [0, τ],
	• ∑ qk = Q (or ∫0τ ϕ(t) dt = Q),
	• N ≤ 100.
Challenge
The challenge is to:
	1. Formulate a precise optimisation problem over time–size–account distributions, specifying:
		○ a stealth / detectability functional,
		○ an impact and cost model, and
		○ any market-memory or recycle-rate dynamics.
	2. Prove existence and (where possible) uniqueness of stealth-optimal execution distributions under this model.
	3. Design explicit algorithms which, given an alpha signal, a total notional Q, and a horizon τ, produce a stealth-optimal execution schedule, including:
		○ how to split Q across up to 100 accounts, and
		○ how to allocate each account’s flow over time.



𝗪𝗵𝗲𝗻 𝘆𝗼𝘂𝗿 𝗮𝗹𝗽𝗵𝗮 𝗴𝗲𝘁𝘀 𝗯𝗶𝗴, 𝗶𝘁 𝘀𝘁𝗮𝗿𝘁𝘀 𝘁𝗼 𝗯𝗲𝘁𝗿𝗮𝘆 𝘆𝗼𝘂.

In algorithmic trading, a strategy that works will eventually attract capital.
More capital → bigger orders → larger footprints → other firms start to see you.

At some point, performance doesn’t decay because the signal “stopped working” –
it decays because 𝘆𝗼𝘂𝗿 𝗼𝘄𝗻 𝘀𝗶𝘇𝗲, 𝗺𝗮𝗿𝗸𝗲𝘁 𝗶𝗺𝗽𝗮𝗰𝘁 𝗮𝗻𝗱 𝗹𝗼𝘀𝘀 𝗼𝗳 𝘀𝘁𝗲𝗮𝗹𝘁𝗵 kill it.

That’s the motivation behind
𝗧𝗼𝗺𝗼𝗿𝗿𝗼𝘄’𝘀 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟯 — 𝗦𝘁𝗲𝗮𝗹𝘁𝗵-𝗢𝗽𝘁𝗶𝗺𝗮𝗹 𝗘𝘅𝗲𝗰𝘂𝘁𝗶𝗼𝗻 𝗗𝗶𝘀𝘁𝗿𝗶𝗯𝘂𝘁𝗶𝗼𝗻𝘀.

Very informally:
Given a profitable alpha, a total notional 𝑄 and a time horizon [0, 𝜏],
how should you distribute your trades over time, size and accounts
to maximise stealth and minimise slippage / cost – under realistic microstructure (square-root impact, account limits, fee schedules, market “memory”, etc.)?

We allow:
 • splitting flow across up to ~100 accounts (no infinite-account tricks),
 • square-root–type visibility in order size,
 • size-dependent transaction costs and possible cross-impact,
 • a finite “memory horizon” where markets gradually forget older flow.

The conjecture asks for the existence and construction of a 𝘀𝘁𝗲𝗮𝗹𝘁𝗵-𝗼𝗽𝘁𝗶𝗺𝗮𝗹 𝗲𝘅𝗲𝗰𝘂𝘁𝗶𝗼𝗻 𝗱𝗶𝘀𝘁𝗿𝗶𝗯𝘂𝘁𝗶𝗼𝗻:
an execution policy that jointly chooses:
 • when to trade,
 • how big to trade, and
 • how to allocate flow across accounts,

so that information leakage is minimised and expected cost/slippage is controlled.

🔍 I’ve written this up more formally (with a proper optimisation framing and open questions) as part of 𝗧𝗼𝗺𝗼𝗿𝗿𝗼𝘄’𝘀 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲𝘀 for Tomorrow Capital Research.

If you work on:
 • execution / market microstructure,
 • optimal execution under constraints,
 • inventory & alpha decay, or
 • multi-agent / game-theoretic models of markets,

I’d love to hear your thoughts, references, or counter-examples.
👉 Full conjecture is here:
🔗 https://lnkd.in/dPKxZcKc

Also be sure to read Lorenzo M. Zorzi Chiarioni's solution to the initial problem statement
hashtag#quantfinance hashtag#algorithmictrading hashtag#marketmicrostructure hashtag#optimalexecution hashtag#marketimpact hashtag#hft hashtag#alphadecay hashtag#research hashtag#TomorrowCapitalResearch

From <https://www.linkedin.com/in/markbrezina95/recent-activity/all/> 

Conjecture 2 — The Central Pricing Problem
Pricing · Market Microstructure · Mean-Field Effects
We extend the Central Price Problem from a single asset to a system of N > 3 interacting assets.
Setup
Consider N > 3 generalised assets with stochastic price paths {P(i)t}, i = 1, …, N, observed at discrete times 0 = t0 < t1 < … < tT, with prices known for all t ≤ T and unknown for t > T.
For concreteness, one may think of an underlying asset A, its option A* and swap A ̃; another underlying B with B* and B ̃; and more generally underlying assets A, B, C, D, … with associated derivatives A*, A ̃, B*, B ̃ and so on.
	1. Immediate vs historical dependence. The one-step-ahead price of each asset admits a decomposition P(i)T+1 as a weighted combination of an “immediate impact” term and a “historical dynamics” term. The relative weight α(i)T ∈ [0, 1] may vary over time, but the weights always sum to one.
	2. Historical dynamics. The historical term may depend on past prices {P(j)t} for t ≤ T, cross-asset and cross-derivative correlations, order flow, liquidity, volatility regimes, fundamentals, and longer-horizon features such as drift and mean reversion.
	3. Immediate impact and behavioural effects. The immediate impact term captures order-book mechanics and behavioural/flow effects, such as large buy orders pushing prices upward, short-horizon supply–demand imbalances, and stylised mean-reversion patterns around local extremes, while still allowing for a persistent long-horizon drift.
	4. Information set. The modeller has access to (i) the full joint price and volume history of all N assets up to time T, (ii) the financial statements and fundamentals of the corresponding issuers, and (iii) the aggregate positions of other market participants, so that pricing must be treated as a mean-field problem rather than a single-agent one.
	5. Own-price impact. Your orders move the mid-price according to a simple impact rule, for example: new_mid = current_mid + (your_price - current_mid) * sqrt(order_volume) . This is a toy model; researchers are free to propose more realistic impact specifications.
Statement of the Conjecture
Under the multi-asset, mean-field setting above, the conjecture states that there exists:
	1. Short-horizon central price estimator. A statistically significant and practically accurate formula (or class of models) for the “central” or “immediate” price of each asset at time T + 1, constructed from observable market and participant features at or before time T.
	2. Multi-step confidence regions. A statistical procedure that produces confidence intervals (or more general confidence sets) for the joint price paths {P(i)t} over t = T + 1, …, T + H, with H ≥ 5 and i = 1, …, N, achieving empirically valid coverage probabilities (“reasonable accuracy”) under realistic market conditions.
In other words, there should exist a non-trivial, empirically testable mapping from historical dynamics (prices, volumes, correlations, regimes, fundamentals) and immediate impact (order-book state, flows, participant positions, your own trades) to short-term central prices and medium- horizon price ranges for a system of interacting assets.
Research Note
Special interest is reserved for solutions that embed this multi-asset formulation into a unified pricing framework that simultaneously covers:
	• the market-maker’s very short-horizon pricing problem,
	• the high-frequency trading pricing problem, and
	• the medium- to long-term asset pricing problem,
under a common set of assumptions linking microstructure, participant behaviour and longer-term price formation.



In markets, “price” sounds like a simple object: last traded, mid, mark.

In reality, it’s where 𝘮𝘪𝘤𝘳𝘰𝘴𝘵𝘳𝘶𝘤𝘵𝘶𝘳𝘦, 𝘣𝘦𝘩𝘢𝘷𝘪𝘰𝘶𝘳 and 𝘮𝘢𝘤𝘳𝘰 all collide.

Across multiple correlated assets and derivatives, with order flow constantly pushing and pulling, the practical question becomes:
• How do we 𝗱𝗲𝗳𝗶𝗻𝗲 𝗮𝗻𝗱 𝗲𝘀𝘁𝗶𝗺𝗮𝘁𝗲 𝗮 𝗰𝗲𝗻𝘁𝗿𝗮𝗹 𝗽𝗿𝗶𝗰𝗲 – consistently – when your own trades move the market?

At Tomorrow Capital Research, this is what sits behind
𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟮 – “𝗧𝗵𝗲 𝗖𝗲𝗻𝘁𝗿𝗮𝗹 𝗣𝗿𝗶𝗰𝗶𝗻𝗴 𝗣𝗿𝗼𝗯𝗹𝗲𝗺”.

We look at a system of N>3 interacting assets (underlyings + derivatives) and ask whether one can construct:

 • A short-horizon 𝗰𝗲𝗻𝘁𝗿𝗮𝗹 𝗽𝗿𝗶𝗰𝗲 𝗲𝘀𝘁𝗶𝗺𝗮𝘁𝗼𝗿 that combines:
 • immediate impact / order-book effects
 • historical dynamics (regimes, volatility, correlations, fundamentals)
 • And a 𝗺𝘂𝗹𝘁𝗶-𝘀𝘁𝗲𝗽 𝗰𝗼𝗻𝗳𝗶𝗱𝗲𝗻𝗰𝗲 region for future prices across single assets, with empirically valid coverage under realistic conditions.

Conceptually, we decompose the next price into:
 • an 𝗶𝗺𝗺𝗲𝗱𝗶𝗮𝘁𝗲 component driven by order flow, impact and short-term behaviour, and
 • a 𝗵𝗶𝘀𝘁𝗼𝗿𝗶𝗰𝗮𝗹 component driven by longer-horizon dynamics, regimes and fundamentals,

with weights that evolve over time and across assets – and with the extra twist that pricing is a 𝗺𝗲𝗮𝗻-𝗳𝗶𝗲𝗹𝗱 problem: you’re part of the crowd you’re trying to model.

We’ve now formalised this as 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟮 𝗼𝗻 𝘁𝗵𝗲 𝗧𝗼𝗺𝗼𝗿𝗿𝗼𝘄 𝗖𝗮𝗽𝗶𝘁𝗮𝗹 𝗥𝗲𝘀𝗲𝗮𝗿𝗰𝗵 𝘄𝗲𝗯𝘀𝗶𝘁𝗲, including a multi-asset setup, explicit impact terms and a mean-field flavour linking:
 • market-making,
 • high-frequency trading, and
 • medium-/long-term asset pricing
into a single “central pricing” framework.

You can find the updated conjecture here (full mathematical setup + notation):
https://lnkd.in/dPKxZcKc

I’d love to hear from people thinking about similar questions in market microstructure, HFT, asset pricing, execution, control or mean-field models:

 • Notes, sketches, partial results
 • Links to related papers
 • Reflections from live trading / risk systems

If this sparks anything – even a rough idea – I’m very happy to read it, discuss it, or point others towards your work.

@tomorrowcapitalresearch
hashtag#quantitativefinance hashtag#algorithmictrading hashtag#marketmicrostructure hashtag#highfrequencytrading hashtag#assetpricing hashtag#stochasticcontrol hashtag#meanfieldgames hashtag#riskmanagement hashtag#centralprice hashtag#TomorrowCapitalResearch

From <https://www.linkedin.com/in/markbrezina95/recent-activity/all/> 

Conjecture 1 — Dynamic reallocation between strategies under path switching
Stochastic Processes · Path Stitching · Strategy Reallocation
Consider a single risky asset with price process (St) observed at discrete times 0 = t0 < t1 < … < tN = T. We are given a family of models {ℳk} under which we can simulate Monte Carlo paths, and a finite collection of trading strategies {π¹, …, πK} (e.g. mean-reversion, trend-following).
	1. Initial regime (0 → T₁). At time t = 0, choose an initial model ℳ₁ and simulate a set of paths {X(1,i)t} on [0, T₁]. Using a chosen objective (e.g. risk-adjusted return), select a “best” strategy π(1) (for example, mean-reversion) and trade according to its position trajectory θ(1)t over [0, T₁]. Once realised prices on [0, T₁] are known, we can evaluate how well (ℳ₁, π(1)) fit the realised path.
	2. Event and model update at T₂. At some later time T₂ ∈ (T₁, T), a significant event or regime change occurs. The original model ℳ₁ and strategy π(1) are no longer believed to be optimal beyond T₂. We adopt a new model ℳ₂ and a new “target” strategy π(2) (for example, trend-following) with position trajectory θ(2)t on [T₂, T₃], where T₃ > T₂ is the time by which the transition to the new regime should be complete.
	3. Re-simulation in the new regime (T₂ → T₃). Starting from the realised price ST₂, simulate a new family of paths {X(2,j)t} on [T₂, T₃] under ℳ₂ and obtain the positions implied by π(2). The portfolio at T₂ currently holds the legacy position θ(1)T₂, but we want to move toward θ(2)t over [T₂, T₃], subject to frictions (transaction costs, liquidity, risk limits).
The core problem on [T₂, T₃] is to define a dynamically consistent transition from the old position to the new one. For instance, introducing a control process wt ∈ [0, 1] representing the fraction of capital allocated to the new regime, the realised position at time t could be written as
θt = (1 − wt) · θ(1)t + wt · θ(2)t,   t ∈ [T₂, T₃].
We then seek a rule for wt (or, in discrete time, for allocations at t₁, t₂, …, tn between T₂ and T₃) that trades off:
	• closeness to the new “optimal” path θ(2)t,
	• trading costs and turnover, and
	• risk and capital constraints along the entire transition.
Challenge. Given a sequence of model/strategy pairs (ℳ₁, π(1)), (ℳ₂, π(2)), … and simulated paths plus realised prices at update times 0 < T₁ < T₂ < T₃ < …, design either:
	1. A mathematical framework (loss functional, cost functional, and a notion of “shortest” or most coherent adjustment between position paths), or
	2. An explicit algorithm that, at each update time Tk, outputs a dynamically consistent sequence of positions θt₁, …, θtn or allocations between strategies on [Tk, Tk+1],
describing how capital should be reallocated from the old regime to the new one in a way that is path-wise coherent, cost-efficient, and robust across Monte Carlo scenarios.


In algorithmic trading, there 𝘪𝘯𝘦𝘷𝘪𝘵𝘢𝘣𝘭𝘺 comes a day when one strategy stops being enough.

There is no “one-size-fits-all” algorithm – only “one-fits-most, for a while”. Regimes change, models get updated, and suddenly you’re facing a very real, very practical question:
 • How do we 𝗺𝗼𝘃𝗲 𝗳𝗿𝗼𝗺 𝗼𝗻𝗲 𝘀𝘁𝗿𝗮𝘁𝗲𝗴𝘆 𝘁𝗼 𝗮𝗻𝗼𝘁𝗵𝗲𝗿, 𝗼𝗽𝘁𝗶𝗺𝗮𝗹𝗹𝘆?

At Tomorrow Capital Research, this is exactly what sits behind 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟭 – “𝗗𝘆𝗻𝗮𝗺𝗶𝗰 𝗿𝗲𝗮𝗹𝗹𝗼𝗰𝗮𝘁𝗶𝗼𝗻 𝗯𝗲𝘁𝘄𝗲𝗲𝗻 𝘀𝘁𝗿𝗮𝘁𝗲𝗴𝗶𝗲𝘀 𝘂𝗻𝗱𝗲𝗿 𝗽𝗮𝘁𝗵 𝘀𝘄𝗶𝘁𝗰𝗵𝗶𝗻𝗴”. 

When a model update or regime shift hits, we’re forced to think about:
 • Transition speed between old and new strategies
 • Optimal inventory adjustments
 • Transaction cost and liquidity constraints
 • How to stay dynamically consistent as new information arrives 

We’ve now updated 𝗖𝗼𝗻𝗷𝗲𝗰𝘁𝘂𝗿𝗲 𝟭 𝗼𝗻 𝘁𝗵𝗲 𝗧𝗼𝗺𝗼𝗿𝗿𝗼𝘄 𝗖𝗮𝗽𝗶𝘁𝗮𝗹 𝗥𝗲𝘀𝗲𝗮𝗿𝗰𝗵 𝘄𝗲𝗯𝘀𝗶𝘁𝗲 to better reflect this broader transition problem – not just “switching a model” but 𝘳𝘦𝘢𝘭𝘭𝘰𝘤𝘢𝘵𝘪𝘯𝘨 𝘢𝘭𝘰𝘯𝘨 𝘢𝘯 𝘦𝘯𝘵𝘪𝘳𝘦 𝘱𝘢𝘵𝘩 under frictions and risk limits. 

The updated conjecture is available on our site, and I’ve attached the corresponding PDF to this post for anyone who wants the full mathematical setup and notation. 
https://lnkd.in/dPKxZcKc

I’d love to hear from people thinking about similar questions in portfolio optimisation, control, execution or mean-field / stochastic control:
 • Notes, sketches, partial results
 • Links to related papers
 • Reflections from live trading transitions

If this sparks any ideas, I’m very happy to read them, discuss them, or simply point others towards your work.

@tomorrowcapitalresearch
hashtag#quantitativefinance hashtag#algorithmictrading hashtag#portfoliooptimization hashtag#execution hashtag#stochasticcontrol hashtag#marketmicrostructure hashtag#meanfieldgames hashtag#pathswitching hashtag#TomorrowCapitalResearchAssets	Cryptocurrencies vs Equities
Horizon	Ultra-low latency(sub-ms), Medium-freq(hours to days)
Exchanges	Binance, OKX, Coinbase, Robinhood
Tech	Python, C++ for speed
Finance	Leverage: flexible
Features	moving averages, volatility, order-flow imbalances, on-chain indicators, Momentum, reversal, carry, post-earnings-drift and low-volatility effects, PCA factor rotation, breakout detectors, or ML predictors, RSI/oscillator-based signals, transaction volume, whale movements, staking rates

Strategies	ULL
	Mid-freq
	• market making - quoting and rebalancing orders to capture bid-ask spreads and liquidity rebates. Position goals stay near zero.
	• cross-exchange / triangular arb
	• Order-Book Microstructure: Models that analyze order flow imbalance and momentum ignition to enter positions before large moves. Such strategies require sub-millisecond tick-by-tick processing.
	• Statistical arbitrage - Avellaneda–Lee s-scores
	• Pairs trading - ?
	
	
	real-time level-2 order-book imbalance, expected‐fill price and implied spread as primary predictors for 5 min–1 h signals. 
	
Practitioners filter spoofing by conditioning on filled orders only or using Bookmap-style depth-surfaces.
	
	Transformer variants (Informer, Autoformer, TimeXer) are now favoured for 15 m–1 d price forecasting
	
	momentum/trend, mean-reversion, LSTM/Transformer predictors, DeFi arb.
Long-term pairs trading, Machine Learning Models, Fundamental/On-Chain Signals, DeFi/DEX Arbitrage
Methods	Execution Engine: A fast, event-driven order manager. It receives “send order” commands from strategies and handles all API-level details (rate-limits, retries, smart routing). Critical code (order serialization, nonce management, latency-critical paths) can be written in C++ with Python bindings (e.g. via PyBind11) for speed. The engine tracks outstanding orders and confirmations.
	
	- Deep-representation models (Transformers, T-GAT, FinGPT)
	- Mean-field/broker games for order-flow toxicity
	- RL execution agents, dynamic Kelly sizing, rough-vol surfaces, Chebyshev interp.
	- Temporal Graph-Attention Networks better than GARCH at vol forecasts
	
	Machine Learning Module: Off-board model training (Python/TF or PyTorch) that updates predictive models online or at intervals. Examples: LSTM networks for short-term price movement​
Modern reinforcement-learning approaches (e.g. deep Q-networks) may also be explored for strategy development, though robust financial RL remains cutting-edge.


	• Graph-signal processing HAR-extensions embed realised-volatility vectors on a stock-correlation graph and transform them in the graph-spectral domain for next-day risk signals 

	• On-chain analytics for crypto: desks scrape funding rates, NVT-style utilisation ratios and liquidation clusters; tutorials at PrimeXBT summarise the standard metric set PrimeXBT.
Risk	Position Limits: No strategy can exceed its defined exposure.
	Stop-Loss / Take-Profit: Pre-set thresholds automatically exit trades to cap losses/profits​
	Portfolio-Level Checks: Monitors total drawdown and triggers system-wide safe modes if severe volatility occurs.
	Real-Time Alerts: Automated alerts/notifications if any risk rule is violated.
Architecture	data→ feature engine → strategy modules → C++ order manager → risk manager → monitoring/logging.
Plan	prototype → paper-trade → C++ optimisation → staged release → live risk dashboards.
Prototyping & Backtesting →Simulation & Paper Trading →C++ Optimization→Data Pipeline→Integration Testing→Monitoring Setup→Iterate & Tune




A Beginner's Step-by-Step Guide to Exploring Quantitative Trading with £50
Section 1: Introduction - Welcome to Quantitative Trading
What is Quantitative Finance and Trading? (Simplified)
Quantitative finance, often shortened to "quant finance," represents a field within finance that relies heavily on mathematical and statistical modeling to inform financial decisions. Instead of relying solely on intuition, news, or traditional financial analysis, quantitative approaches use data, algorithms, and computational power to identify opportunities, manage risk, and execute strategies. It's about applying scientific and mathematical rigor to the complexities of financial markets.   
Quantitative trading is a specific application of these principles, focusing on the development and execution of trading strategies based on quantitative analysis. These strategies aim to buy and sell financial assets—such as stocks, bonds, commodities, or cryptocurrencies—often in an automated or semi-automated fashion. The core idea is to leverage mathematical models to detect patterns, statistical relationships, or market inefficiencies that can be systematically exploited for potential profit. This involves rigorous testing of ideas using historical data (backtesting) before deploying them in live markets.   
What are High-Frequency (HFT) and Mid-Frequency Trading (MFT)?
Within the realm of quantitative trading, strategies are often categorized by their trading frequency or holding period. High-Frequency Trading (HFT) sits at the extreme end of this spectrum. It is a form of algorithmic trading characterized by:
	• Extraordinary Speed: Trades are executed in microseconds (millionths of a second) or milliseconds (thousandths of a second).   
	• High Volume: A vast number of orders are generated and executed throughout the trading day.   
	• Short Holding Periods: Positions are typically held for fractions of a second up to a few minutes, rarely overnight.   
	• Sophisticated Technology: HFT relies on powerful computers, complex algorithms, and high-speed data connections.   
A critical concept in HFT is latency. Latency refers to the time delay experienced when data travels from one point to another, such as from the exchange's matching engine to a trader's system, or the time taken for an order to travel to the exchange and be processed. HFT firms invest heavily to minimize latency, as even a tiny speed advantage can be crucial. This often involves co-location, which means placing their trading servers in the same physical data center as the exchange's servers to reduce the physical distance data needs to travel.   
Mid-Frequency Trading (MFT) occupies a space between HFT and traditional, slower trading strategies. While still automated and faster than human traders, MFT typically involves holding periods that might range from minutes to hours or even across a trading day. MFT strategies may focus less on pure speed arbitrage and more on analyzing factors like order flow (the sequence and size of buy and sell orders hitting the market), identifying short-term predictive signals, or reacting to specific market events or HFT-driven patterns. MFT generally requires less extreme low-latency infrastructure compared to HFT, though speed and efficient execution remain important.   
Setting Realistic Expectations: The £50 Journey
It is absolutely crucial to understand the realities of HFT from the outset. Executing genuine HFT strategies requires immense capital investment in specialized hardware, ultra-fast software, direct data feeds from exchanges, co-location services, and skilled personnel. These costs run into millions of pounds, making HFT fundamentally inaccessible for retail traders, especially those starting with a budget of £50.   
Therefore, the £50 budget allocated for this exploration should not be viewed as trading capital for HFT or even significant MFT participation. Instead, it should be considered a resource for learning and familiarization. This budget might cover access to an online course, purchase of an introductory book, or, much later, funding a tiny real-money account purely for understanding the psychological dynamics of trading.
The primary goal of this journey, given the constraints, is to build foundational knowledge. This involves understanding the core concepts of quantitative finance, learning the necessary mathematical and programming tools, exploring market mechanics, and simulating trading strategies in a risk-free environment. The objective shifts from doing HFT, which is impossible under these circumstances, to learning about HFT and the broader field of quantitative trading. The significant financial and technological barriers mean that simulation and knowledge acquisition are the only practical avenues available. This constraint fundamentally shapes the approach outlined in this guide, prioritizing education and experimentation over any expectation of profit from HFT activities.   
Section 2: Building Your Foundation
Before diving into complex strategies or specific tools, establishing a solid foundation in market basics, essential mathematical concepts, and programming is paramount. Attempting to understand advanced quantitative trading without these prerequisites is unlikely to be productive.
Essential Concepts: Markets, Assets, Orders
	• Markets: Financial markets are platforms where assets are traded. Key examples include: 
		○ Stock Markets: Exchanges like the New York Stock Exchange (NYSE) or London Stock Exchange (LSE) where shares representing ownership in publicly listed companies are bought and sold.   
		○ Cryptocurrency Exchanges: Platforms like Binance, Coinbase, or OKX facilitate the trading of digital assets (cryptocurrencies) based on blockchain technology.   
	• Assets: The items being traded. For this initial exploration, the focus will be on: 
		○ Stocks (Equities): Represent partial ownership in a corporation. Owning stock may entitle the holder to dividends (a share of company profits) and capital appreciation if the stock price increases.   
		○ Cryptocurrencies: Digital or virtual currencies secured by cryptography, operating on decentralized networks (like blockchains). Examples include Bitcoin (BTC) and Ethereum (ETH). While many other asset classes exist (e.g., commodities, derivatives ), stocks and crypto provide accessible starting points.   
	• Order Types: Instructions given to a broker or exchange to execute a trade. Two fundamental types are: 
		○ Market Order: An instruction to buy or sell immediately at the best currently available market price. It prioritizes speed of execution over the exact price achieved.   
		○ Limit Order: An instruction to buy or sell only at a specified price or better. A buy limit order executes only at the limit price or lower, while a sell limit order executes only at the limit price or higher. Limit orders provide price control but do not guarantee execution if the market never reaches the specified price. They are essential tools in quantitative strategies for precise entry and exit points.   
	• Limit Order Books (LOB): This is the electronic list maintained by an exchange showing all active limit orders for a specific asset. It displays the quantity of shares or tokens available to be bought (bids) and sold (asks/offers) at various price levels, ranked by price. The LOB provides transparency into market depth (supply and demand at different prices) and is a critical data source for many HFT and MFT strategies, such as market making, which involves simultaneously posting bid and ask orders.   
	• Basic Trading Ideas: 
		○ Long vs. Short: Going "long" means buying an asset hoping its price will rise. Going "short" involves selling an asset (often borrowed) expecting its price to fall, aiming to buy it back cheaper later. Short selling is more complex and carries different risks.   
		○ Diversification: The principle of spreading investments across different assets or strategies to reduce overall risk. The idea is captured by the adage "don't put all your eggs in one basket".   
Key Skills: Introduction to Probability & Statistics
Quantitative trading is fundamentally based on mathematics. Understanding probability and statistics is not optional; it's essential for developing, testing, and understanding trading models.   
	• Why Math Matters: Probability helps quantify the likelihood of future market movements or the success rate of a strategy. Statistics provides the tools to analyze historical data, identify patterns, measure risk, and determine if a strategy's performance is statistically significant or just due to luck.   
	• Learning Resources: 
		○ Khan Academy: Offers excellent, free introductory courses on basic probability (calculating probabilities, understanding independent and dependent events, sample spaces) and statistics (mean, variance, distributions like the Normal distribution, hypothesis testing). These resources are ideal for building foundational understanding without initial cost.   
		○ Coursera: Provides more structured courses, often from universities, covering these topics within a financial context. Examples include the Wharton Quantitative Modeling course or other beginner-level finance specializations. These may involve costs but offer a more formal learning path.   
Learning Python for Finance
A programming language is needed to implement and test quantitative ideas. Python has become the dominant language in quantitative finance for several reasons:
	• Extensive Libraries: Python boasts a rich ecosystem of libraries specifically designed for data analysis (Pandas), numerical computation (NumPy), machine learning (scikit-learn, TensorFlow, PyTorch), and visualization (Matplotlib, Seaborn), significantly speeding up development.   
	• Readability and Ease of Learning: Python's syntax is generally considered cleaner and easier to learn compared to other languages like C++, making it more accessible for beginners.
	• Community Support: A large and active community means ample resources, tutorials, and support are available online.
The initial steps for learning Python specifically for finance typically involve :   
	1. Python Fundamentals: Grasping core concepts like variables, data types (integers, floats, strings, booleans), basic operators (arithmetic, comparison, logical), and control flow (if-else statements, loops).
	2. Data Structures: Learning to work with lists (ordered, mutable collections) and potentially dictionaries (key-value pairs).
	3. NumPy: Understanding NumPy arrays, which are fundamental for efficient numerical operations on large datasets. This includes array creation, indexing, slicing, and mathematical functions.   
	4. Pandas: This library is crucial for financial data analysis. Learning to use Pandas DataFrames for importing, cleaning, manipulating, and analyzing time-series data (like stock or crypto prices) is essential.   
	5. Matplotlib: Acquiring basic data visualization skills using Matplotlib to create plots like line charts (for prices over time), histograms (for return distributions), and scatter plots to visually explore data and results.   
Recommended Resources:
	• DataCamp: Offers interactive courses tailored to finance, such as "Introduction to Python for Finance" and tutorials covering algorithmic trading basics using Pandas and Matplotlib. Interactive platforms can be effective for hands-on learning.   
	• Other Platforms: Coursera , Udemy , Codecademy, freeCodeCamp, and numerous free online tutorials (e.g., official Python documentation, YouTube channels) provide alternative learning paths.   
Developing proficiency in basic probability, statistics, and Python programming forms the bedrock upon which any understanding of quantitative trading, especially HFT/MFT, must be built. These skills are non-negotiable prerequisites. Without this foundation, attempting to decipher or implement complex strategies, including those hinted at in advanced resources, will be ineffective. The learning process must start here.   
Section 3: Exploring the Landscape - Platforms and Tools
Once the foundational concepts and skills are being developed, the next step is to become familiar with the trading environment. This involves understanding the specific exchanges available and utilizing practice tools like demo accounts.
Introduction to Your Exchanges
The query specified access to several platforms, each with its own characteristics:
	• Binance, OKX, Coinbase: These are large, global cryptocurrency exchanges. They offer trading in a wide variety of cryptocurrencies (spot trading) and often provide access to more complex products like futures and options (derivatives). They are known for high liquidity (ease of trading without significantly impacting the price) and comprehensive Application Programming Interfaces (APIs) for automated trading. Coinbase also offers a platform called Coinbase Advanced, targeted at more experienced traders with features like advanced charting and lower fees.   
	• Robinhood: This platform gained popularity, particularly in the US, for its commission-free trading model, primarily accessed via a mobile app. It offers stocks, Exchange-Traded Funds (ETFs), options, and some cryptocurrencies. While user-friendly for beginners, it's important to be aware of regulations like the Pattern Day Trading (PDT) rule in the US, which applies to margin accounts and can restrict frequent trading if account equity is below $25,000. * **Saxo Bank:** This is a more traditional online brokerage based in Denmark, offering access to a broader range of global markets and asset classes, including stocks, forex (foreign exchange), Contracts for Difference (CFDs), futures, options, and bonds. It provides multiple platform options (SaxoInvestor for beginners, SaxoTraderGO/PRO for advanced traders) and integrates with third-party tools like TradingView. Fee structures and account minimums may differ from crypto-native exchanges or commission-free brokers. ### The Power of Practice: Paper Trading / Demo Accounts Given the complexity and risks of trading, especially quantitative trading, practicing in a simulated environment is essential before committing real capital. This is achieved through paper trading or demo accounts. * **Definition:** A demo account provides a simulated trading environment that mimics live market conditions but uses virtual (fake) money. Trades are placed based on real-time or slightly delayed market data, allowing users to experience the trading process without financial risk. * **Essential Benefits:** * **Risk-Free Learning:** The most significant advantage is the ability to experiment and make mistakes without losing real money. * **Platform Familiarization:** Users can learn the layout and functionality of a trading platform, including how to place different order types (market, limit), use charting tools, monitor positions, and manage a virtual portfolio. * **Strategy Testing:** Demo accounts allow traders to test their strategies under realistic (though not perfect) market conditions to see how they might perform. * **Emotional Preparation (Limited):** While it cannot fully replicate the stress of real trading, paper trading can help users become accustomed to market volatility and the experience of winning and losing trades, albeit without the real emotional weight. * **Important Limitations:** It's crucial to understand that demo accounts are not perfect replicas of live trading: * **Execution Simulation:** They often don't accurately simulate the order book dynamics. Orders might fill instantly at the trigger price, ignoring factors like available liquidity (volume at a given price) or slippage (the difference between the expected fill price and the actual fill price), which are significant in real markets, especially for large orders or fast-moving conditions. * **Latency:** Demo systems may not replicate the real-world latencies involved in order routing and execution, making them unsuitable for testing true HFT strategies. * **Psychological Differences:** Trading without real money at stake removes the powerful emotions of fear and greed, which significantly influence decision-making in live trading. Success in a demo account doesn't guarantee success with real capital. * **System Priority:** Demo account performance might be de-prioritized compared to live trading systems during periods of high market activity. ### Table 3.1: Demo Account Availability on Specified Platforms To facilitate the choice of a platform for initial practice, the following table summarizes the demo account availability and features for the specified exchanges, based on the gathered information. This consolidation provides a quick reference to identify suitable starting points for risk-free exploration. | Platform | Demo Account Available? | How to Access | Asset Types Supported (in Demo) | Duration/Limits | Key Notes/Sources | | :------------ | :---------------------- | :---------------------------------------------------------------------------- | :------------------------------------- | :------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | | **Binance** | Yes | Via Binance Spot Test Network (testnet.binance.vision); requires GitHub login | Crypto (Spot) | No explicit duration mentioned; uses separate API keys | Simulates spot trading; no real order book simulation; separate from main Binance account | | **OKX** | Yes | Via "Demo Trading" mode within the main platform interface | Crypto (Spot, Futures, Options likely) | Virtual funds provided ($ simulated); can add more | Designed to closely replicate real trading features; good for practicing various crypto instruments | | Coinbase | No (Directly) | N/A (Simulation possible via third-party platforms like QuantConnect) | Crypto (via external simulation) | N/A | Coinbase platform itself focuses on live trading; paper trading requires connecting API to external simulation environments | | Robinhood | No | N/A | N/A | N/A | Platform does not offer a paper trading or demo account feature | | Saxo Bank | Yes | Via specific Demo Account sign-up on their website | Stocks, Forex, CFDs, Futures (likely) | 20 days | Provides $100,000 virtual cash; uses SaxoTraderGO and SaxoTraderPRO platforms |   
Based on this comparison, for practicing the cryptocurrency trading mentioned in the query, OKX and the Binance Spot Test Network appear to be the most suitable starting points among the listed options, as they offer dedicated crypto demo environments. Saxo Bank provides a demo account for traditional assets and derivatives but it is time-limited. Coinbase and Robinhood lack native paper trading functionalities, making them less ideal for initial risk-free learning and experimentation directly on their platforms. The choice between OKX and Binance Testnet might depend on user preference for interface or the desire to simulate futures trading (which OKX demo likely supports more directly).
Section 4: Your £50 Action Plan: Getting Started
This section outlines a practical, step-by-step plan designed for a beginner to start exploring quantitative trading concepts with a £50 budget, focusing on learning and simulation.
Step 1: Choose a Foundational Learning Path
	• Recommendation: Begin with freely available, high-quality resources to maximize the budget. 
		○ Mathematics: Utilize Khan Academy for foundational understanding of probability and statistics. Work through modules covering basic probability rules, sample spaces, distributions, and descriptive statistics.   
		○ Programming: Start learning Python basics using free online tutorials, documentation (python.org), or the free tiers of platforms like DataCamp or Codecademy. Focus initially on syntax, data types, control structures, and functions.   
	• Structured Courses (Optional): If the budget allows after considering potential minimal real-trade exposure later, or if a more structured approach is preferred, consider enrolling in an introductory online course. Platforms like Coursera offer relevant beginner courses in finance, accounting, investing, or quantitative modeling. Prioritize courses that build a strong conceptual understanding rather than promising complex trading strategies immediately.   
Step 2: Set Up a Demo Account
	• Recommendation: Based on the analysis in Section 3, start with a platform offering accessible crypto paper trading. The OKX Demo Trading mode or the Binance Spot Test Network are recommended choices.   
	• Access Instructions (Simplified): 
		○ OKX: Log into the OKX platform, navigate to the "Assets" or equivalent menu, and select the "Demo Trading" option to switch to the simulated environment. Virtual funds should be automatically allocated.   
		○ Binance Spot Test Network: Visit the testnet website (testnet.binance.vision), log in using GitHub credentials, navigate to the API key section, and generate a new HMAC_SHA256 key, saving the key and secret securely. This key pair would then be used to connect a compatible application (like potentially a self-written Python script later, or a platform like QuantConnect set to paper trading mode using Binance data).   
Step 3: Make Your First Paper Trades
	• Objective: The goal here is purely mechanical understanding and platform familiarization, not generating virtual profit.
	• Exercises: 
		○ Choose a major cryptocurrency like Bitcoin (BTC) or Ethereum (ETH).   
		○ Practice placing a Market Order to buy a small virtual amount. Observe how quickly it fills and at what price.   
		○ Practice placing a Limit Order to buy below the current market price. Observe if it sits in the order book and if it executes when/if the price drops to that level.   
		○ Practice placing a Sell Limit Order above the current market price and observe its behavior.   
		○ Familiarize yourself with the platform's interface for viewing open orders, trade history, and portfolio balance.   
		○ Explore any available charting tools within the demo environment to view price history.
Step 4: Basic Python Practice
	• Integration: Apply the Python skills learned in Step 1 in parallel with paper trading exploration.
	• Exercises: 
		○ Find sample historical price data online (many sources offer free CSV files for major assets) or create simple Python lists representing price movements.
		○ Use the Pandas library to load this data into a DataFrame.   
		○ Calculate daily or hourly simple returns (e.g., (price_today - price_yesterday) / price_yesterday).
		○ Use Matplotlib to create a basic line plot of the price data to visualize trends.   
Step 5: (Optional & Cautious) Using the £50 for Real Exposure
	• Important Disclaimer: This step should only be considered after significant practice in a demo account and a clear understanding of the risks involved. Reiterate that the £50 budget is insufficient for serious trading, especially HFT, and any funds used are at high risk of loss. This is an educational exercise in psychology, not an investment strategy.
	• Recommendation: If deemed appropriate for learning, allocate a very small portion of the budget (e.g., £10 - £20) for this exercise. Choose a platform with low minimum deposit requirements and low trading fees (Binance or OKX might be suitable, but verify their current terms ).   
	• Activity: 
		○ Deposit the small amount into the chosen live account.
		○ Select a major, highly liquid cryptocurrency (e.g., BTC or ETH).
		○ Place one or two very small limit orders based on simple criteria learned during practice.
		○ Observe: Pay close attention to the execution process, the fees charged, and, most importantly, the emotional response to having real money at stake. Note feelings of anxiety, excitement, fear, or greed.   
		○ Treat any outcome (profit or loss) as an educational experience. The primary value is understanding the psychological dimension that paper trading cannot fully replicate. This bridge between simulation and reality, even on a micro-scale, provides a valuable lesson, reinforcing the need for discipline and risk management. However, this step must be approached as paying for a lesson, not as an attempt to generate returns.   
Section 5: Understanding the Provided Resources (COPtoLON Repos)
The query specifically requested incorporating information about four GitHub repositories associated with the user COPtoLON (Mark Brezina ). It's important to assess their content and relevance for a beginner embarking on this learning path.   
Overview of the Repositories
	• COPtoLON/WTech: This repository is explicitly labeled as an "Algorithmic trading repository". Its content, based on the README and file structure glimpses, includes notes and potential code related to: 
		○ Strategy Development: Concepts like defining an "ideal state," detecting market states (trend, risk), matching tactics to states, and considering opponent positions.   
		○ Execution: Notes on optimal execution considering market impact, timing, and cost.   
		○ Advanced Topics: Mentions of leverage, options, futures, multi-asset trading, HFT market making, and medium-term horizons.   
		○ Research Areas: Lists numerous advanced research topics, including Markov models, mean reversion analysis, convergence theorems, machine learning models (autoencoders, MLP, LightGBM, XGBoost, CatBoost), specific cross-validation techniques for time-series data, mean field games, and event-based HFT.   
  
	• COPtoLON/Knowledgebase: Pinned on the user's profile and described as "A collection of information for quantitative finance, insurance, mathematics and AI". This suggests it's a curated repository of notes, links, or documents relevant to these fields, likely Markdown files, similar in concept but distinct from GitHub's built-in organizational knowledge base feature.   
	• COPtoLON/KB: Also pinned but lacks a description in the provided materials. Given the common use of "KB" for knowledge base and the existence of command-line knowledge base tools like kb-manager , this might be a personal system for organizing notes, possibly using such a tool, or related to the Knowledgebase repo. Its exact content requires direct exploration.   
	• COPtoLON/TMRW: Pinned but no description or content details were available in the snippets. The name is ambiguous ("Tomorrow"? Acronym?). Its purpose and content need direct investigation on GitHub..   
Relevance for a Beginner
	• Assessment: The content visible, particularly in the WTech repository , clearly indicates that these resources are not designed as introductory educational material. They represent working notes, research ideas, and potentially code fragments from someone actively engaged in the field at a relatively advanced level.   
	• The concepts mentioned (e.g., hidden Markov models, autoencoders for feature generation, specific time-series cross-validation methods, mean field games) require a significant prerequisite understanding of mathematics, statistics, machine learning, and financial markets, far beyond the scope of the foundational learning outlined in Sections 2 and 4.
	• Conclusion: These specific GitHub repositories are unsuitable as starting points for a complete beginner with no prior background in quantitative finance or programming. Attempting to learn from them directly would likely lead to confusion and discouragement.
How to Approach Them Later in Your Journey
While not suitable for initial learning, these repositories can still hold value later in the educational process:
	• Bookmark for Future Reference: Suggest saving the links to these repositories.
	• Revisit After Foundational Learning: Once a solid grasp of Python programming, statistics, basic trading principles, and perhaps introductory quantitative finance concepts (potentially through courses like those on Coursera or DataCamp) is achieved, revisiting these repositories could be beneficial.
	• Gain Context and Insight: WTech , in particular, can offer a glimpse into the types of problems, research questions, and techniques that practitioners in algorithmic trading explore. It illustrates the depth and complexity involved.   
	• Explore Curated Information: The Knowledgebase repository might contain valuable summaries, links, or references that become understandable and useful once the necessary background knowledge is acquired.   
These repositories should be viewed not as instructional guides but as potential aspirational examples or future exploration paths. They showcase the complexity and scope of quantitative and algorithmic trading. Their immediate value for a beginner lies in demonstrating the level of knowledge required in the field, rather than providing direct learning steps. They represent a destination one might work towards, not the starting line.   
Section 6: Diving Deeper - HFT/MFT Resources
After establishing foundational knowledge and gaining basic practical experience through simulation, the next phase involves delving deeper into the specific concepts relevant to High-Frequency and Mid-Frequency Trading.
Key Concepts Revisited
Understanding HFT and MFT requires familiarity with several core ideas:
	• Latency and Co-location: As previously mentioned, minimizing the time delay (latency) in receiving market data and sending orders is paramount, especially in HFT. Co-location, placing servers within the exchange's data center, is a primary method used by HFT firms to achieve the lowest possible latency. Understanding the physical and technological aspects of speed is crucial.   
	• Order Book Dynamics: HFT/MFT strategies often rely heavily on analyzing the Limit Order Book (LOB). This involves monitoring the flow of incoming orders, the depth of liquidity at different price levels, imbalances between bids and asks, and the bid-ask spread (the difference between the best bid and the best ask price). Strategies like market making directly interact with the LOB by posting simultaneous buy and sell limit orders to capture the spread. Order flow analysis attempts to predict short-term price movements based on the patterns of incoming orders.   
	• Market Microstructure: This academic and practical field studies the processes and rules governing trading in financial markets. It examines how prices are formed, how trades are executed, the role of different market participants (like market makers), the impact of order types, and the effects of exchange architecture on trading outcomes. A solid understanding of market microstructure is essential for designing effective HFT/MFT strategies.   
	• Common Strategy Archetypes (Brief Overview): 
		○ Arbitrage: Exploiting temporary price discrepancies of the same or similar assets across different markets or forms (e.g., buying a stock on one exchange and simultaneously selling it for slightly more on another). HFT arbitrage often focuses on minute, fleeting discrepancies requiring high speed ("latency arbitrage" ).   
		○ Market Making: Providing liquidity to the market by continuously quoting buy (bid) and sell (ask) prices for a security, aiming to profit from the bid-ask spread. This requires managing inventory risk.   
		○ Statistical Arbitrage ("Stat Arb"): Using quantitative models to identify short-term mispricings between related assets based on historical correlations or patterns.
		○ Event-Driven Trading: Designing strategies to react extremely quickly to specific news releases, economic data publications, or other market events.   
		○ MFT-Specific Strategies: These might include analyzing tick data (individual trades) for order flow patterns, "stop hunting" (attempting to trigger stop-loss orders), or exploiting HFT behaviors like front-running countermeasures.   
Recommended Reading/Learning
To deepen understanding of these concepts, consider exploring resources such as:
	• Investopedia: Provides accessible articles explaining key terminology and concepts related to HFT , latency , market making , arbitrage , and the limit order book.   
	• QuantStart: This website focuses on quantitative trading and often features articles and learning paths covering strategy identification, backtesting, and execution systems, potentially including introductory material relevant to HFT/MFT concepts.   
	• Quantra (QuantInsti): This platform offers specialized online courses in algorithmic and quantitative trading. Courses like "Trading in Milliseconds" specifically cover MFT topics such as market microstructure, latency types, order flow calculation (e.g., Lee-Ready algorithm), and MFT strategies. Be aware that such specialized courses typically involve costs.   
Understanding the Challenges (Reiteration)
While learning about HFT/MFT is valuable, it's vital to constantly remember the practical limitations for retail participation:
	• Insurmountable Barriers: The high costs of technology and data, the need for co-location, the sophisticated algorithms, data access limitations, and regulatory hurdles effectively preclude retail traders from competing directly in the HFT space.   
	• Technological Arms Race: HFT is characterized by a continuous "arms race," where firms constantly invest in faster technology and algorithms to maintain a competitive edge, making any advantage fleeting.   
	• Market Impact Concerns: HFT has faced criticism regarding its potential to exacerbate market volatility, contribute to "flash crashes" (sudden, severe price drops followed by a quick recovery), and create an uneven playing field.   
Given these realities, a retail trader's focus should adapt. Since direct participation in the ultra-low latency HFT game is not feasible, a more practical approach involves learning the concepts of HFT/MFT to understand market dynamics, but directing practical efforts towards areas where retail traders can potentially operate. This might involve developing algorithmic strategies on slightly longer timeframes (minutes, hours, days) where infrastructure costs and microsecond latency are less critical , or exploring MFT strategies that rely more on clever analysis (like order flow ) rather than pure speed, using the tools and platforms realistically available.   
Section 7: Next Steps and Continuous Learning
Embarking on the journey into quantitative trading is a marathon, not a sprint. It requires dedication, continuous learning, and a systematic approach.
Recap of the Learning Path
The recommended progression for a beginner starting with minimal capital and aiming to understand quantitative trading, including HFT/MFT, is as follows:
	1. Build the Foundation: Master basic probability, statistics, and Python programming.   
	2. Familiarize with Platforms: Use demo accounts (like OKX or Binance Testnet) to understand trading mechanics and platform interfaces risk-free.   
	3. Understand Trading Mechanics: Practice placing different order types and observing market behavior in simulation.   
	4. Apply Programming Skills: Use Python for basic financial data analysis and visualization.   
	5. (Optional) Gain Psychological Insight: Consider a very small, controlled real-money trade to experience the emotional aspect, accepting the high likelihood of loss as an educational cost.   
	6. Deepen Quant Knowledge: Study core quantitative finance concepts, market microstructure, and the theory behind different strategy types (including HFT/MFT) using resources like Investopedia, QuantStart, or specialized courses.   
	7. Explore Advanced Topics: Once fundamentals are strong, revisit more complex resources like the COPtoLON repositories or academic papers for deeper insights.   
Importance of Patience and Persistence
Quantitative finance is an intellectually demanding field. Progress requires significant time investment, consistent effort, and the resilience to work through challenging concepts and inevitable setbacks. There are no shortcuts to expertise, and profitability (especially in algorithmic trading) is never guaranteed. Approach it as a continuous learning process.   
Further Learning Avenues
As foundational knowledge grows, consider exploring these avenues:
	• Advanced Courses: Platforms like Coursera, edX, Quantra, and university programs offer more advanced courses in financial engineering, machine learning for trading, derivatives, and specific algorithmic trading techniques.   
	• Online Communities: Engage thoughtfully with online communities focused on quantitative finance and algorithmic trading (e.g., forums on platforms like QuantConnect , specific subreddits, or sites like Elite Trader ). Be critical of information and focus on learning rather than seeking "get rich quick" signals.   
	• Reputable Blogs and Research: Follow blogs from established quantitative finance professionals and researchers (some examples mentioned in ). Explore academic research repositories like SSRN or arXiv q-fin for cutting-edge ideas (though these are often highly technical).   
	• Backtesting Platforms: Eventually, learning to use a dedicated backtesting platform (e.g., QuantConnect , Zipline, Backtrader) becomes crucial. These platforms allow rigorous testing of trading strategies on historical data to assess potential performance and identify flaws before considering live deployment.   
Final Encouragement
The path to understanding quantitative trading is challenging but intellectually rewarding. While direct participation in HFT might be out of reach for a retail beginner with limited capital, the process of learning the underlying mathematics, programming, market mechanics, and strategic thinking provides a deep and valuable understanding of how modern financial markets operate. Frame this journey as an opportunity for intellectual growth and skill development, focusing on the knowledge gained along the way.
Conclusion
This guide provides a structured, step-by-step approach for a beginner with a £50 budget to begin exploring the world of quantitative finance, with a particular interest in High-Frequency Trading (HFT) and Mid-Frequency Trading (MFT). The analysis underscores several key conclusions and recommendations:
	1. HFT Accessibility: Direct participation in HFT is realistically impossible for a retail beginner with a £50 budget due to the prohibitive costs of technology, data feeds, and low-latency infrastructure.   
	2. Focus on Foundational Learning: The primary goal must shift from doing HFT to learning about quantitative trading. The £50 budget is best utilized for educational resources and simulation, not significant trading capital. Mastery of foundational mathematics (probability, statistics) and programming (Python with libraries like Pandas, NumPy, Matplotlib) is the essential first step.   
	3. Leverage Simulation: Paper trading (demo accounts) is a critical tool for risk-free learning of platform mechanics, order types, and basic strategy testing. Platforms like OKX and the Binance Spot Test Network offer suitable starting points for crypto simulation among the specified options.   
	4. Provided Repositories (COPtoLON): The specified GitHub repositories contain advanced notes and research ideas. They are not suitable for beginners but can serve as aspirational examples or resources for future exploration once foundational knowledge is established.   
	5. Realistic Path Forward: A practical progression involves building foundational skills, utilizing demo accounts extensively, potentially undertaking a minimal real-money trade for psychological insight (treating it as tuition), and then delving into deeper quantitative concepts and market microstructure. For applying algorithmic principles, focusing on MFT concepts or strategies on slower timeframes (minutes to days) is more realistic for retail participants than attempting HFT.   
Quantitative trading is a complex and evolving field. Success requires patience, continuous learning, and a rigorous, data-driven approach. By following a structured learning path and maintaining realistic expectations, a beginner can gain valuable knowledge and skills, building a strong foundation for future exploration in this fascinating domain.   
Sources used in the report



I hindsight can man godt finde den optimale policy
Lille Q læring, hvis appr. Af proces kan laves markov, hvis 7 sidste frame, så løses.

Reinforcement learning I kontinuert tid og rum.

Rough differential equations
[2306.14258] A Neural RDE approach for continuous-time non-Markovian stochastic control problems
Solving and Learning non-Markovian Stochastic Control problems in continuous-time with Neural RDEs | OpenReview
[2207.00713] q-Learning in Continuous Time

Fudan gruppe mht. Stoch control. Maximum principper, bellman ligner


	1. Markov states for "hvor er vi ved"

Model-baseret, forudse underlæggende dynamic - NN

Model-fri, PPO, soft, critic metoder -simuleret

Lille Q, regret bounds, kontinuert kalibrering og optimering, N^(3/4)
Decision transformer, signature regression, path signature, signature transform, 
Rough paths.
Feedforward, test, der hvor du har en edge.

Counter-factual learning, opdatere på det der skete og på det vi forventede ville ske med andre handlinger. Explainable AI.

Ikke arg max
Aksel Fristrup
12:35 PM
greedy policy
exploration
online learning
Aksel Fristrup
12:36 PM
exploration-exploitation tradeoff

From <https://meet.google.com/trr-ntbq-pcx> 

Entropy
Aksel Fristrup
12:36 PM
exploration-exploitation tradeoff
entropy regularisation
Aksel Fristrup
12:37 PM
renyi, tsallis
Aksel Fristrup
12:39 PM
reward shaping

From <https://meet.google.com/trr-ntbq-pcx> 

Selv-eksiterende
Neural-hawkes process
Parametrisere

hierarchical reinforcement learning
Aksel Fristrup
12:50 PM
H-MARL

From <https://meet.google.com/trr-ntbq-pcx> 

Credit assignment, hvad gjorde at jeg fik rewardSpread Capture vs. Inventory Risk
With a fixed 20pt spread, your theoretical max is capturing that 20pts on every round-trip. Your constraints make inventory skewing the central optimization lever.

The Optimal Framework: Skewed Market Making
Reservation Price
Instead of quoting symmetrically around mid, shift your centre based on inventory:
skew = (inventory / max_inventory) * half_spread
reservation_price = mid - skew
bid = reservation_price - half_spread
ask = reservation_price + half_spread
Worked examples (half_spread = 10, max_inventory = 80):
Inventory	Skew	Bid	Ask
0	0	mid - 10	mid + 10
+40	+5	mid - 15	mid + 5
+80	+10	mid - 20	mid ← dump aggression
-40	-5	mid - 5	mid + 15
-80	-10	mid	mid + 20
This elegantly:
	• Makes your ask cheaper when long → encourages offloading
	• Makes your bid higher when short → encourages covering
	• Never requires you to cross the spread or trade at a loss

Hard Rules at Inventory Limits
if inventory >= +80:  cancel bid entirely, only quote ask
if inventory <= -80:  cancel ask entirely, only quote bid
Don't just skew — remove the order that would worsen your position.

Profit Optimisation Levers
1. Skew Aggressiveness (most impactful)
The half_spread multiplier in skew can be tuned:
	• Conservative (skew = inventory/max * 5): quote closer to mid more often, more fills, slower inventory clearance
	• Aggressive (skew = inventory/max * 10): prioritise flat inventory, fewer fills but less risk
Given your data mean-reverts strongly, a moderate skew is likely optimal — the price will come back to you anyway.
2. Quote Refresh Trigger
Since you can only have 1 set live at a time, reprice on:
	• Every new mid-price tick
	• Every fill (immediately repost adjusted for new inventory)
3. The Core P&L Equation
profit = (fills × avg_spread_captured) - (inventory_at_close × mark_to_market_loss)
Your goal is to maximise fills while keeping end-of-session inventory near zero. The skew formula directly addresses both.

What to Simulate/Backtest
Run over your 30k ticks varying two parameters:
for risk_aversion in [0.3, 0.5, 0.7, 1.0]:        # skew multiplier
    for inventory_limit in [40, 60, 80]:            # soft limit to one-side only
        simulate → record: total_pnl, max_drawdown, fill_rate
Given the strong mean reversion in your data, you'll likely find lower risk aversion wins because inventory self-corrects.

Key Insight
Your constraint of 1 order set at a time actually simplifies things — you're not competing for queue priority. The entire optimisation reduces to: how aggressively do you skew based on inventory? The formula above is provably near-optimal for mean-reverting assets under these constraints (this is the simplified Avellaneda-Stoikov result).




Reading the Chart
What you're showing appears to be order book or tick data — likely bid (green), ask (red), and mid/last price (black) — oscillating around 10,000 over ~30,000 time steps. The spread looks roughly ±20 pts from mid, with occasional spikes to ±30.

Relevant HFT Strategies for This Type of Asset
1. Market Making (Most Applicable)
This is the classic fit for mean-reverting, range-bound price action like this:
	• Post limit orders on both sides of the spread
	• Earn the bid-ask spread repeatedly
	• Key risk: Inventory accumulation during directional moves (the occasional large spikes)
	• Edge here: The price clearly mean-reverts to ~10,000, so adverse selection risk is relatively low
2. Statistical Arbitrage / Pairs Trading
If the red/green/black lines are correlated instruments (e.g., futures vs spot, or two correlated assets):
	• Trade the spread between them when it deviates from its mean
	• The visual spread between red and green looks relatively stable → potential cointegration
3. Mean Reversion
	• The data oscillates tightly around 10,000 with no visible trend
	• Short when price spikes to ~10,020–10,030, long when it dips to ~9,975–9,985
	• Works well here given the clear reversion behavior

What the Chart Suggests to Avoid
Strategy	Why It Fails Here
Momentum / trend-following	No sustained directional moves visible
Breakout strategies	Spikes revert quickly, no follow-through
Low-frequency positioning	Edge is in the microstructure, not macro direction

Critical Practical Considerations
	• Latency: At 30,000 ticks, co-location and microsecond execution matter enormously
	• Fill rate vs adverse selection tradeoff: Tighter quotes = more fills but more toxic flow
	• Spike management: Those ~30pt outlier moves need hard inventory limits or hedging rules
	• Transaction costs: At this scale, fees can easily exceed the signal edge


Given your specific constraints—a fixed spread of 20, a hard inventory limit of $\pm 80$, and the ability to route only one bid-ask pair at a time—your optimization problem shifts from determining the optimal spread to determining the optimal skew.
Since you cannot widen or narrow the spread (it is fixed at 20, meaning a half-spread of 10), your only mechanism for managing inventory and maximizing profit is to shift your quotes asymmetrically around the true mid-price.
Here is the step-by-step mathematical and logical framework to optimize this:
1. The Inventory Skew Logic (The Reservation Price)
When your inventory is exactly 0, your quotes should be perfectly symmetrical around the market mid-price ($s$). You place your Bid at $s - 10$ and your Ask at $s + 10$.
However, as you accumulate inventory, you must calculate a "Reservation Price" ($r$). This is your own internal, risk-adjusted valuation of the asset. The standard formula for this is:
$r = s - q \gamma \sigma^2$
Where:
	• $s$ = The current mid-price.
	• $q$ = Your current inventory (from -80 to +80).
	• $\gamma$ = Your risk aversion parameter (this is what you need to optimize).
	• $\sigma^2$ = The variance of the asset's price over a short, rolling window.
2. Placing Your 1 Set of Orders
Because your spread is fixed at 20, you anchor your single set of orders to your new reservation price, not the market mid-price:
	• Optimal Bid: $r - 10$
	• Optimal Ask: $r + 10$
How this makes you money while protecting inventory:
Let's say you are dangerously long with an inventory of $q = +70$. The formula will calculate a reservation price $r$ that is lower than the market mid-price $s$.
	• Your Ask ($r + 10$) will drop closer to the market mid-price, making it highly competitive. You are likely to get filled, shedding your long inventory.
	• Your Bid ($r - 10$) will drop further away from the market mid-price, drastically reducing the chance you accidentally buy more and hit your +80 limit.
3. Handling the $\pm 80$ Hard Boundary
To optimize for profit, you must avoid crossing your maximum inventory limit, as hitting it usually forces you to halt trading or execute a market order (crossing the spread) to liquidate, which destroys your accumulated edge.
You must implement a strict boundary condition in your logic:
	• If $q \ge 80$: Cancel the Bid entirely. Only quote the Ask at $s + 10$ (or aggressively at $s + 9$ to ensure a fill).
	• If $q \le -80$: Cancel the Ask entirely. Only quote the Bid at $s - 10$.
4. How to Optimize for Maximum Profit (Finding $\gamma$)
To extract the absolute most profit, you need to find the "Goldilocks" value for $\gamma$ (Risk Aversion). You do this by running a grid-search optimization in a backtest.
	• If $\gamma$ is too low: Your quotes remain too symmetrical. You will capture the spread frequently, but you will constantly slam into your $\pm 80$ inventory limit, forcing you to stop trading or take losses on liquidation.
	• If $\gamma$ is too high: You are too terrified of holding inventory. You will skew your quotes so violently at an inventory of just $\pm 10$ that you will stop getting filled entirely, leaving money on the table.
	• The Optimum: You want the lowest possible $\gamma$ that mathematically keeps your inventory within the $[-79, +79]$ range for 99% of the trading session.
Are you currently writing the backtest for this strategy, or are you trying to deploy it directly into a live paper-trading environment to gauge the fill rates?

Based on the visual characteristics of the chart you've uploaded, we are looking at a highly noisy, high-frequency time series (the black line) enveloped by an upper bound (the red line) and a lower bound (the green line).
In the context of High-Frequency Trading (HFT), this visual perfectly illustrates the dynamics of Algorithmic Market Making, specifically using an inventory-management framework. The black line represents the asset's mid-price, while the red and green lines represent the optimal Ask and Bid quotes, respectively.
For an asset that behaves like this—exhibiting heavy micro-structure noise (rapid short-term oscillations) combined with broader macro-wandering (like the sharp dip around step 17,000)—the optimal HFT strategy is Inventory-Aware Market Making, often modeled after the classic Avellaneda-Stoikov framework.
Here is a breakdown of how the optimal strategy operates for this type of asset:
1. Capturing the Bid-Ask Spread (Market Making)
The core objective is to simultaneously place limit orders at the bid (green line) and the ask (red line).
	• Because the asset exhibits high "tick bounce" (rapid oscillation back and forth), limit orders placed on both sides are likely to be filled frequently.
	• Your profit is the spread minus transaction costs. You are essentially being paid to provide liquidity to the market.
2. Inventory Management (The Avellaneda-Stoikov Model)
The biggest risk in market making is directional risk. If you look at the chart around time step 16,000 to 17,000, the asset drops sharply. If you keep your quotes perfectly symmetrical around the mid-price during a drop, your bid (buy) orders will be continuously filled, and your ask (sell) orders will be ignored. You will end up holding a massive, depreciating long position ("toxic inventory").
To optimize this, your strategy must dynamically adjust a reservation price (an inventory-adjusted mid-price). The classic continuous-time formula for this is:
$r = s - q \gamma \sigma^2$
Where:
	• $r$ = Reservation price (your new "internal" mid-price)
	• $s$ = Actual market mid-price
	• $q$ = Your current inventory (positive if long, negative if short)
	• $\gamma$ = Your risk aversion coefficient
	• $\sigma$ = The asset's volatility
How it looks on your chart: When you accumulate too much long inventory (e.g., during a price drop), the formula lowers your reservation price. This shifts both your red and green lines down. Your ask price becomes more competitive (increasing the chance of selling to reduce inventory), and your bid price drops further away (decreasing the chance of buying more).
3. Fading and Micro-Mean Reversion
If you are not acting as a pure market maker, a secondary optimal strategy is Statistical Arbitrage / Mean Reversion.
	• The bands (red and green) can be calculated using a rolling short-term standard deviation (like micro-Bollinger Bands).
	• When the black line spikes up and touches the red line, you trigger a short sell, expecting the micro-noise to revert it back to the mean.
	• When it drops to the green line, you buy.
Key Considerations for Production
	• Adverse Selection: The strategy must incorporate order book imbalance signals. If the order book suddenly becomes extremely top-heavy (massive sell pressure), the algorithm needs to widen the spread or pull the bid entirely to avoid getting run over by informed institutional flow.
	• Latency: Strategies that rely on capturing the spread in highly noisy assets require ultra-low latency infrastructure (FPGA, co-location). If you are slow, you will only get filled when the market is moving against you.
Are you looking to implement this logic in a specific asset class (like crypto or equities), or are you currently in the process of backtesting a specific market-making model?



From this chart alone, there is no provably optimal HFT strategy. But for an asset that looks like this, the best candidate is usually:
adaptive mean-reversion market making with strict inventory control and selective taking only at extremes.
Why:
	• The series looks range-bound around a stable center, not strongly trending. 
	• It shows frequent short-term oscillation. 
	• The move size looks modest relative to the repeated back-and-forth, which is exactly where spread capture tends to dominate. 
	• If you have an inventory cap like ±80, uncontrolled directional bets will usually hurt more than help. 
What usually works best
1. Core strategy: two-sided market making
Post both bid and ask around a fair value estimate.
Your fair value should not just be raw mid. It should be something like:
	• denoised mid 
	• microprice / queue-weighted price 
	• short rolling fair value 
	• maybe slightly adjusted by imbalance/order flow 
A simple quote model is:
	• bid = fair value - half_width - inventory_skew + alpha_skew 
	• ask = fair value + half_width - inventory_skew + alpha_skew 
Where:
	• half_width expands when volatility rises 
	• inventory_skew pushes quotes to reduce risk 
	• alpha_skew leans quotes when short-term signal is strong 
For a chart like this, this is usually the main PnL engine.

2. Inventory is the real game
With HFT market making, inventory management often matters more than prediction.
For ±80 inventory, I would usually structure it like this:
	• 0 to 20: quote normally 
	• 20 to 50: start skewing harder 
	• 50 to 80: prioritize flattening over earning spread 
	• near 80: stop quoting the risky side entirely 
Example:
	• if long, raise both bid and ask, but raise the bid less aggressively or even pull it 
	• if short, lower both quotes, but lower the ask less aggressively or pull it 
That prevents getting stuck with inventory during adverse moves.

3. Only take when the edge is clearly better than making
For this kind of asset, aggressive market taking is usually not the default best strategy.
Why it often underperforms:
	• you pay spread 
	• you consume liquidity at the wrong time 
	• you interfere with your maker inventory 
	• many apparent “signals” are just noise 
So instead of frequent taking, only take when price is far from fair value, such as:
	• best ask is well below fair value by more than your execution threshold 
	• best bid is well above fair value by more than your execution threshold 
That threshold should be larger than:
	• fees 
	• adverse selection risk 
	• expected short-term noise 
	• the value of keeping inventory free for market making 
In practice: take rarely, make constantly.

Best structure for this kind of asset
Layer 1: fair value estimator
Use a robust fair value, for example:
	• top-of-book microprice 
	• denoised VW mid 
	• short rolling regression or EWMA 
	• optionally adjusted by imbalance/order flow 
Example:

f_t=〖"microprice" 〗_t+β_1⋅〖"imbalance" 〗_t+β_2⋅〖"orderflow" 〗_t
 
But keep it simple. Overfitting is a big danger.

Layer 2: market making engine
Quote around fair value with dynamic width:

〖"width" 〗_t=max⁡("min_width" ⓜ,c_1⋅"spread" +c_2⋅σ_t )
 
So when the market is noisy, you widen out.
When it is calm, you tighten.

Layer 3: inventory skew
Shift reservation price by inventory:

r_t=f_t−k⋅q_t
 
Where:
	• f_t  is fair value 
	• q_t  is current inventory 
	• k is inventory penalty 
This is usually one of the most important pieces.

Layer 4: selective taking
Only cross the spread when:

∣"market price"−f_t∣>"take threshold"
 
and the signal agrees with:
	• imbalance 
	• order flow 
	• recent reversion tendency 
	• available inventory capacity 
This should be small in frequency and high in quality.

What I would avoid
For an asset like this, I would avoid:
Pure trend following
Unless you can prove persistent drift, trend following will often get chopped up.
Always-on Z-score taking
A Z-score alone often fires too often in noisy HFT data.
Better:
	• use Z-score only as a secondary filter 
	• require agreement with microstructure signal 
	• or only use it to adjust quote skew, not to trigger full crossing orders 
Symmetric fixed thresholds without inventory context
“Buy below X, sell above Y” can work in backtests, but often breaks once inventory, queue priority, and adverse selection matter.

Likely best practical strategy
If I had to choose one implementation for this chart, it would be:
Adaptive mean-reversion market maker
with:
	• fair value = denoised microprice / VW mid 
	• dynamic quote width from local vol + spread 
	• strong inventory skew 
	• hard position-based throttling 
	• aggressive taking only at large dislocations 
	• optional short-term imbalance/orderflow adjustment 
That is usually better than:
	• pure market taking 
	• pure threshold mean reversion 
	• naive trend following 

Decision rule intuition
Think of it like this:
	• normal state → make both sides 
	• slightly stretched price → skew quotes toward reversion 
	• very stretched price → take selectively 
	• inventory too large → stop optimizing edge, prioritize flattening 
That tends to be the robust answer.

What “optimal” usually means here
For this type of asset, the best strategy is usually the one that maximizes:

"spread capture"−"adverse selection"−"inventory risk"−"overtrading cost"
 
So the optimal strategy is not “predict direction perfectly.”
It is usually:
earn the spread repeatedly while not getting trapped.

A strong implementation recipe
A very good starting setup would be:
	• Fair value: denoised microprice 
	• Width: base_width + vol_adjustment 
	• Inventory skew: moderate to strong 
	• Max quote size: shrink as inventory grows 
	• Taking: only beyond fair ± n * local_std 
	• Filters for taking: 
		○ imbalance agrees 
		○ order flow agrees 
		○ not already near inventory limit 
Example logic:
	• if price near fair: make both sides 
	• if ask << fair and inventory not too long: buy take + keep ask posted to unwind 
	• if bid >> fair and inventory not too short: sell take + keep bid posted to unwind 
	• if inventory large: only quote flattening side 

My blunt answer
For an asset like this, the best HFT strategy is usually not a fancy predictor.
It is usually:
inventory-aware market making around a stable fair value, with very sparse and high-conviction mean-reversion taking.
That is the setup I would trust most.


Good — this constraint actually simplifies things a lot. You’re basically in a single-asset, fixed-spread, inventory-constrained microstructure game.
Let me be blunt:
👉 With a constant ~20 spread, the optimal strategy is NOT prediction-heavy.
👉 It’s queue capture + inventory cycling + selective edge-taking.

🔥 The core reality of your setup
	• Spread ≈ 20 
	• Mid oscillates tightly around ~10000 
	• No cross-asset alpha 
	• Max inventory = ±80 
	• Only one set of orders at a time 
So your PnL sources are:
1. Spread capture (dominant)
Earn ~20 per round trip
→ This is your main engine
2. Mean reversion edge (secondary)
Occasional mispricing vs fair value
→ Smaller but higher-conviction trades

🧠 The actual optimisation problem
You want to maximize:

"PnL"="fill rate"×"spread"−"adverse selection"−"inventory cost"
 
So you are balancing:
	• Quote tight → more fills, worse selection 
	• Quote wide → fewer fills, better quality 
	• Too aggressive → inventory blow-up 
	• Too passive → no fills 

✅ Optimal strategy structure (for your constraints)
1. Always-on market making (this is your core)
Since spread is fixed (~20), you want to sit:
	• bid = best_bid + 1 
	• ask = best_ask - 1 
👉 You are stepping inside spread by 1 tick
This gives:
	• priority in queue 
	• max fill probability 
	• still ~18 spread capture 

2. Your real edge = inventory-aware skew
This is where most people underperform.
Use:

r_t=f_t−k⋅q_t
 
	• f_t: fair value (mid or slightly denoised mid) 
	• q_t: inventory 
	• k: skew strength 
Practical effect:
Inventory	Action
0	symmetric quotes
+40	shift quotes down
+80	only sell
-40	shift quotes up
-80	only buy
👉 This prevents inventory getting stuck at extremes.

3. Quote sizing is just as important
You should NOT always quote max size.
Instead:
Inventory	Size
0	max size (e.g. 20–30)
40	reduced
60	small
80	zero on risky side

4. Only ONE alpha for taking (keep it simple)
Since you already observed:
	taking reduces PnL from 2300 → 1600
That tells you:
👉 Your taking logic is too frequent or too weak
Fix:
Only take when:

∣price−fair∣>"large threshold"
 
Not ±1
Not small Z-score
👉 Think rare, high conviction
Example:
	• Buy only if ask << fair by 8–10 ticks 
	• Sell only if bid >> fair by 8–10 ticks 
And only if:
	• inventory allows it 
	• not already skewed heavily 

5. Don’t mix signals incorrectly
This is likely what hurt your PnL.
Bad combination:
	• market making + frequent taking 
	• no coordination between them 
Good combination:
👉 Taking should complement inventory
Example:
	• You’re short → only take BUY opportunities 
	• You’re long → only take SELL opportunities 
Never fight your own book.

🧩 The optimal control logic (this is the key)
At each step:

Step 1: compute fair value
	• simple mid or denoised mid 

Step 2: compute inventory skew
	• shift reservation price 

Step 3: decide regime
Case A: normal inventory (|q| < 40)
→ full market making
Case B: medium inventory (40–70)
→ skew quotes + reduce size
Case C: high inventory (70–80)
→ only quote flattening side

Step 4: check for rare take
Only if:
	• strong mispricing 
	• AND aligns with inventory 
	• AND not already overloaded 

💡 What actually maximizes profit here
Given your constraints, the winning setup is:
🥇 Primary driver
tight market making with queue priority
🥈 Secondary driver
inventory skew (very important)
🥉 Tertiary
rare, well-filtered taking

🚫 What to avoid (this is killing your PnL)
❌ Over-trading (most common mistake)
	• taking too often 
	• reacting to noise 
❌ Z-score spam
	• HFT noise breaks Z-score signals 
	• use it only as filter, not trigger 
❌ Symmetric logic
	• treating long and short the same 
	• ignoring inventory state 

⚙️ Concrete “optimal” parameter intuition
If spread ≈ 20:
	• Quote inside by: 1 tick 
	• Expected capture: ~18 
	• Inventory skew: 
		○ ~1 tick per 10–15 units 
	• Take threshold: 
		○ ≥ 6–10 ticks away from fair 
	• Max size: 
		○ 20–30 near neutral 
		○ taper to 0 near limits 

🧠 The key mental model
You are not predicting price.
You are:
	harvesting oscillation while managing inventory risk

🔥 Final answer (compressed)
For your exact setup, the optimal HFT strategy is:
👉 Inventory-skewed inside-spread market making
+
👉 rare, high-threshold mean-reversion taking aligned with inventory


You’re circling something real—but the way you’ve phrased it is almost right and needs sharpening to avoid a misleading conclusion.
The intuition you have (what’s right)
There is a general pattern across systems (quantum → statistical → macroscopic):
	• Small-scale systems → fast, frequent fluctuations, constant “micro-corrections” 
	• Large-scale systems → slower dynamics, larger, rarer adjustments 
	• Aggregation → noise averages out, leaving smoother but more persistent deviations 
This shows up everywhere:
	• Quantum systems → constant fluctuations, decoherence corrections 
	• Molecular systems → thermal noise averaging into temperature 
	• Markets → tick noise vs structural moves 
	• Climate → weather vs long-term shifts 
So your idea of:
	“Corrections become rarer but larger as scale increases”
…is directionally correct.

Where it breaks (important correction)
The key mistake is thinking that “corrections” are actively happening at all scales in the same way.
They’re not.
What actually changes is:
	Whether the system is dominated by noise or by structure.

A better framing
1. At the quantum / micro level
	• Everything is fluctuation-dominated 
	• There is no clear “imbalance” vs “correction” 
	• It’s just continuous stochastic motion 
	• “Correction” is not meaningful—it’s all noise 

2. At mesoscopic levels (statistical physics)
	• You start getting restoring forces 
		○ e.g. diffusion, mean reversion, entropy maximisation 
	• Imbalances exist, but: 
		○ They are corrected continuously 
		○ Via many small interactions 
This is closest to your “constant corrections” idea.

3. At macroscopic / aggregated levels
Now something qualitatively different happens:
	• Small fluctuations cancel out 
	• Only large, coherent imbalances survive aggregation 
	• Systems develop: 
		○ inertia 
		○ thresholds 
		○ nonlinear responses 
So:
	Corrections are not rarer because the system waits
	They are rarer because only large deviations are visible at that scale

The key principle (this is the core insight)
Across scales, what changes is:
	Signal-to-noise filtering via aggregation
Mathematically / conceptually:
	• Micro level → high variance, low persistence 
	• Macro level → low variance (relative), high persistence 
So:
	• Small imbalances → disappear in aggregation 
	• Large imbalances → propagate → require large corrections 

Your idea, refined
A more accurate version of your statement would be:
	At small scales, systems are dominated by continuous stochastic fluctuations that appear as constant micro-adjustments.
	As we move to larger scales, aggregation filters out small fluctuations, so only larger, more persistent imbalances remain—these manifest as rarer but larger corrections.

Why this matters (especially for you / trading)
This maps directly to what you’ve been working on:
	• Order book level 
		○ constant noise → microstructure → spread capture 
	• Short-term signals 
		○ small mean reversion → frequent, small trades 
	• Longer horizons 
		○ trend / drift → fewer, larger moves 
And the trap you’ve been hitting:
	Trying to apply micro-level “corrections” logic at macro scale (or vice versa)
That’s why combining:
	• market making 
		○ directional holding 
…keeps conflicting.
They operate on different correction regimes.

Even deeper (if you want to push this)
What you’re describing is closely related to:
	• Renormalisation / coarse-graining 
	• Central Limit Theorem 
	• Mean field approximations 
	• Scale separation in stochastic systems 
And in markets:
	• microstructure noise vs alpha 
	• liquidity vs information 
	• HFT vs positional trading

	Voucher surface relative-value with delta hedge
This is the best candidate.
Why it is attractive
The voucher strip is orderly, monotone, and strongly tied to Velvetfruit spot, but the time-value surface is where the action is. The ATM/near-ATM options carry the largest extrinsic value and the strongest useful beta to spot.
Recommended universe
• Primary: VEV_5000, VEV_5100, VEV_5200, VEV_5300
• Hedge: VELVETFRUIT_EXTRACT
Signal construction
At each snapshot:
1. Compute intrinsic value and time value.
2. Fit a rolling smooth curve to time value as a function of strike, using the last 50 to 200 snapshots.
3. Identify residuals:
• buy undervalued voucher if residual < -1.5σ to -2.5σ
• sell overvalued voucher if residual > +1.5σ to +2.5σ
4. Hedge spot exposure with Velvetfruit using a rolling beta or local delta proxy.
Working hedge ratios from this file
• VEV_5000: about 0.65
• VEV_5100: about 0.58
• VEV_5200: about 0.44
• VEV_5300: about 0.27
Entry and exit
• Enter on residual z-score breach with both the voucher and the hedge two-sided.
• Exit when residual mean reverts inside |z| < 0.5.
• Time stop: 5 to 30 snapshots.
• Hard stop: if the corresponding hedge ratio changes materially or a best side disappears.
Risk controls
• Max net spot delta close to zero after hedge
• Per-strike gross exposure cap
• Kill switch if top-of-book disappears
• Reject VEV_6000 and VEV_6500
• Reject signals when spread exceeds median plus 2σ
Sensitivity to missing-side events
High. If any leg loses a best side, the apparent mispricing can be fake because executable parity disappears. This strategy should trade only when both the voucher and hedge have live two-sided books.
Velvetfruit spot and voucher market making
This is the best “steady throughput” strategy if you must produce continuous flow.
Why it is attractive
• Velvetfruit spot spread is stable at around 5 ticks
• Near-ATM vouchers have real informational linkage to spot
• Surface is well-behaved, so inventory can be hedged
Recommended instruments
• VELVETFRUIT_EXTRACT
• VEV_5000
• VEV_5100
• VEV_5200
• selectively VEV_5300
Quoting rules
• Base half-width:
• Spot: 0.5x to 1.2x median spread
• Vouchers: 0.8x to 1.5x median spread
• Inventory skew:
• widen adverse side as inventory grows
• lean quotes in direction of hedge need
• Reprice frequency:
• every snapshot if underlying or surface moves
• immediate cancel if missing side appears in any hedge leg
Entry and exit logic
This is continuous quoting, but operational rules matter:
• Quote only when both sides are present
• Quote size proportional to visible depth and inverse to churn
• Pull quotes during high-churn bursts or when the underlier moves sharply without voucher confirmation
• Hedge voucher fills into spot immediately if inventory exceeds threshold
Risk controls
• Spot inventory cap
• Option vegas/gammas approximated by moneyness bucket limits
• Stop quoting OTM tails during missing-side conditions
• Volatility throttle when one-snapshot spot move exceeds 2 to 3 median moves
Sensitivity to missing-side events
Moderate to high. Missing-side events in OTM vouchers make passive bid quoting non-viable and can poison model-based fair values. Keep the curve-trader active in 5000–5300, not 6000+.
Deep-ITM conversion or synthetic hedge
This is less of a pure alpha strategy and more of a structural hedge / basis control strategy.
VEV_4000 and VEV_4500 behave almost like synthetic spot:
• average time value around 0.01
• extremely tight relationship to Velvetfruit
• deep ITM basis is nearly flat
Use case
• hedge spot inventory with deep ITM vouchers when spot book is temporarily less attractive
• exploit temporary basis deviations if the exchange permits voucher conversion or exercise with low friction
Signal
Track:
[ B_t = C_t + K - S_t ]
For deep ITM calls, this basis should stay very near a small constant.
Entry
• Buy call / sell spot if B_t is too low
• Sell call / buy spot if B_t is too high
• Only when both books are two-sided and conversion rules are known
Exit
• Exit when basis returns toward rolling median
• Hard exit on spread blowout or side disappearance
Important caution
The current file does not include exercise style, maturity, settlement timing, or fees. So this is a design recommendation, not a backtested production strategy.
Latency-sensitive stale-quote taking
This is only a secondary idea because the observed lead/lag structure is weak.
Across the voucher ladder, the best measured lag is basically lag 0, not lag +1. That means the vouchers usually move in sync with Velvetfruit at the snapshot granularity.
So the strategy should be very selective:
• trigger only on large Velvetfruit moves
• look for option quotes that did not reprice by an adequate fraction of expected beta
• take liquidity only if your latency is materially better than the market and raw event timestamps confirm the delay
Parameter ranges
• spot move trigger: 2 to 4 ticks or top decile of spot moves
• stale option threshold: option move < 30% to 40% of expected beta-adjusted move
• hold period: 1 to 3 snapshots
• cancel if the option reprices before your order would realistically arrive
Sensitivity to missing-side events
Very high. Missing sides can make “stale” quotes look attractive when they are simply non-executable.
Liquidity taking around blip proxies
A proper orderflow-blip strategy cannot be validated from this file because the event stream is missing. But a book-blip fade can still be designed.
Use only in:
• VELVETFRUIT_EXTRACT
• VEV_5000
• VEV_5100
• VEV_5200
Signal idea
• detect a one-snapshot outlier move with no supporting depth continuation
• require partial reversal in the next snapshot or two
• fade the move with small size
Rules
• Enter against a move that is at least 4x to 5x the median one-snapshot move
• Require spread to stay normal or narrow
• Hold 1 to 5 snapshots
• Stop if price extends another 1 to 2 ticks against you
Sensitivity
Moderate. This is useful only if the signal is built from the reconstructed state, not from duplicated trade-print columns.
Hydrogel standalone market making
This is the best idea for Hydrogel given the absence of Hydrogel vouchers.
Hydrogel has:
• stable center near 10,000
• mean spread around 15.7
• no strong cross-asset alpha from Velvetfruit
That makes it suitable for simple passive inventory-managed market making, not for pair trading.
Rules
• quote around midpoint using 0.5x to 1.0x of median spread
• symmetric size at low inventory, strong skew once inventory exceeds threshold
• reduce quoting during visible churn bursts
• mark-to-market off midpoint, not off last_price
Risk controls
• strict inventory cap
• time stop for stale inventory
• disable if visible depth collapses or update churn exceeds threshold

From <https://chatgpt.com/c/69ebeb1a-216c-83eb-8608-4757ab90eb2d> 

 
Combine quoting and taking logic
All three approaches, in one way or another, are about deciding:

* when to take liquidity because the current price looks favourable
* when to make liquidity by posting bids and asks around an estimated fair price

So the common trading pattern is not just prediction, but prediction plus execution logic.


Products
	• ToE
ToE
	• MinMax Strategy
		○ Find the trend
		○ find the dX = 0
		○ find the d^2 X = - or d^2 X= +
	• TMRW
		○ Expand
		○ Establish
		○ Excel
		○ etc.
# Volatility questions

**Artur Sepp**

CTA trend following https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3167787

fx more mean reversion - autocorrelation useful.



- “In your modeling or trend-following framework, how do you differentiate between temporary volatility spikes—say from market noise or events—and more structural regime changes in volatility dynamics? Are there specific statistical signals or filters you rely on?”

- “Do you integrate macroeconomic or cross-asset signals into your volatility forecasts for longer horizons, and if so, how do you balance that with the responsiveness needed for short-term risk models?”

- “Given the increasing heterogeneity in volatility behavior across time scales, what methods do you find most robust for forecasting short- vs long-term volatility? Do you see machine learning techniques overtaking traditional models like HAR or GARCH for this purpose?”

- "“How do you recommend adjusting asset exposure or inventory in anticipation of—or in response to—extreme volatility events? Do you use forward-looking metrics like implied skew, or backward-looking stress test models?”"


**Laura Ballotta** \
Can I have access to your paper on "multivariate additive subordination with applications in finance" ?

# illiquid bonds questions



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

There are three bodies of work in what you shared: competition strategy notes, the Tomorrow Capital Research conjectures, and earlier design and learning notes.

1. Competition strategy (IMC Prosperity-style round)

The ranking table scores seven strategies by return, Sharpe, complexity and requirements:

Strategy	Return / quality	Complexity
Voucher-surface relative value on VEV_5000–5300 vs Velvetfruit	High / high	High
Velvetfruit spot + voucher market making	Medium-high	Medium
Deep-ITM basis on VEV_4000/4500	Medium	High
Hydrogel standalone market making	Medium	Low-medium
Stale-quote taking, blip fades, Hydrogel–Velvetfruit stat arb	Low to medium-low	—

Document 1 gives the detail behind each row:

Voucher relative value. Fit a rolling curve of time value against strike, trade residuals beyond about ±1.5–2.5σ, delta-hedge with spot (betas of roughly 0.65, 0.58, 0.44 and 0.27), and exit inside |z| < 0.5.
Market making. Quote spot and near-ATM vouchers with inventory skew.
Deep-ITM basis. Track C + K − S, which should be nearly flat.
Stale-quote taking. Weak, because the vouchers move at lag 0 with spot.
Blip fades. Fade outlier moves of 4–5× the median move.
Hydrogel market making. Stable around 10,000 with a mean spread of ~15.7 and no cross-asset alpha.
Recurring warning. Missing book sides create fake mispricings, so trade only when every leg has a two-sided book.

Documents 2–4 answer “what’s optimal for a range-bound asset with a ±80 limit” (2 and 3 are duplicates). They all reach the same answer:

Inventory-skewed market making around a robust fair value, using the reservation price r = FV − k·q (Avellaneda–Stoikov).
Tiered inventory regimes, shrinking size as inventory grows, and pulling the risky side near the limit.
Taking only rarely, at large dislocations, and in the direction that reduces inventory. Your observation that taking cut PnL from 2300 to 1600 is diagnosed as thresholds set too tight.
Document 4 adds tuning advice: grid-search the risk-aversion parameter so inventory stays within the limit about 99% of the time.

Your own notes on teams’ code (from the Prosperity tutorial round) group them into three archetypes:

A fixed fair value of 10,000 with simple take and make.
An EMA fair value with inventory-skewed quotes.
A regression forecast on cached Banana prices.

Your takeaways from these:

Every team keeps state.
Every team strictly enforces position limits.
They all build from simple parts.
Each team’s edge is “prediction plus execution logic,” meaning the combination of quoting and taking.
2. Tomorrow Capital Research conjectures (with LinkedIn posts)
C1, strategy reallocation under path switching: when the regime or model changes at T₂, how to move from the old position path θ⁽¹⁾ to the new θ⁽²⁾ through a blend weight wₜ. The transition has to balance closeness to the target against cost and risk, and stay robust across Monte Carlo scenarios.
C2, the central pricing problem: for N > 3 interacting assets and derivatives, the next price is a time-varying mix of an immediate-impact term and a historical-dynamics term. The setting is mean-field and includes your own price impact. The conjecture claims there exist a short-horizon central-price estimator and valid confidence regions for horizons H ≥ 5. A bonus is a framework unifying market-making, HFT and long-term pricing.
C3, stealth-optimal execution: split a notional Q over [0, τ] across time, size and up to 100 accounts. The model has square-root visibility, size-dependent costs, cross-impact and a finite market memory. The aim is to minimise detectability and slippage, with existence, uniqueness and an algorithm wanted. The post credits Lorenzo M. Zorzi Chiarioni with a solution to the initial version.
C4 / “X”, existence of optimal strategies in the full model: markets as a hybrid (flow plus jump) multi-agent system on a manifold M×S. The question is whether an optimal feedback or geodesic policy exists, or a Nash or mean-field equilibrium solving an HJB–Fokker–Planck system. You deliberately released no PDF for this one.
C5, “The Final Problem”: a firm decomposed into portfolio, strategy and execution layers, each with local objectives. The conjecture is that a single variational, multiscale principle makes the whole configuration globally and locally optimal at once. You frame it as the closing problem of a self-learning financial architecture.
3. Design, learning and research notes
Trend-following system (document 11):
Multi-horizon EWMAC or time-series momentum signals, risk-scaled and squashed with tanh.
Horizon weights learned online, by Bayesian averaging or regime conditioning.
Volatility targeting plus a drawdown governor.
Cost-aware mean-variance optimisation with constraints.
An execution model and kill switches.
Stat-arb design (in your message):
A clean, liquid universe and a factor risk model.
Residual mean-reversion alpha with “cheap for a reason” filters.
Clustering, baskets and Kalman-filter pairs.
A neutral, constrained optimiser.
Execution and correlation-spike risk controls.
Early architecture notes (document 5):
A crypto/equities, ultra-low-latency vs mid-frequency strategy map.
A pipeline running data → features → strategies → C++ order manager → risk → monitoring.
A staged path from prototype to live trading.
The beginner’s £50 guide (document 5): a generic guide to learning quant trading on a small budget. It also reviews your COPtoLON repos and calls them advanced research notes.
RL meeting notes with Aksel Fristrup (Danish, document 5):
Continuous-time q-learning and neural rough differential equations for non-Markovian control.
Path signatures and decision transformers.
Model-based vs model-free (PPO, actor-critic) approaches.
Exploration, with entropy regularisation (Rényi, Tsallis) and reward shaping.
Neural Hawkes processes, hierarchical multi-agent RL, credit assignment and counterfactual learning.
Multiscale idea (documents 2–3): corrections look constant and small at micro scale and rare and large at macro scale. The refinement is that aggregation filters noise, so only persistent imbalances survive. This explains why market making and directional holding conflict: they live in different correction regimes.
The common thread

The competition notes are a small, concrete instance of the conjectures:

Fair value is a toy version of C2.
Inventory skew and execution are small-scale versions of C3.
Switching between making, taking and hedging regimes is C1.
Running several products and strategies under shared position limits is C5 in miniature.

The RL notes are the toolkit you’d use to attack C4.

<img width="740" height="4488" alt="image" src="https://github.com/user-attachments/assets/b3c06ac4-7f4e-4bc2-a2c2-4775ce7a917c" />



