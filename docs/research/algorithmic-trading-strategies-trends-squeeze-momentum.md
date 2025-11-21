# Algorithmic Trading Strategies: Comprehensive Technical Analysis

**Squeeze trading, trend continuation, and mean reversion strategies represent three fundamentally different approaches to market behavior.** Each strategy capitalizes on distinct market conditions: squeeze strategies exploit volatility compression and expansion, trend strategies capture directional momentum persistence, and mean reversion strategies profit from statistical price normalization. This report synthesizes mathematical formulations, empirical validation studies, and practical implementation guidelines across all three methodologies.

## Volatility compression precedes explosive directional moves

The TTM Squeeze indicator, developed by John Carter in 2004, identifies periods when Bollinger Bands contract inside Keltner Channels—a mathematical signal that volatility compression has reached extreme levels. This configuration represents a coiled spring effect where decreasing volatility precedes explosive breakouts.

**Core detection algorithm:** A squeeze fires when both conditions hold simultaneously: `(Upper_BB < Upper_KC) AND (Lower_BB > Lower_KC)`. The Bollinger Bands use a 20-period SMA with 2.0 standard deviations: `Upper_BB = SMA(20) + 2σ`, while Keltner Channels employ the original 1960 formula with 1.5 ATR multiplier: `Upper_KC = SMA(Typical_Price, 20) + 1.5×ATR(20)`. When volatility contracts sufficiently, the wider Keltner bands engulf the narrower Bollinger bands.

**The momentum histogram determines direction**, calculated via linear regression of the delta between price and a composite midpoint. The delta formula: `Delta = Close - ((Donchian_Midline + SMA_Close)/2)`, where Donchian_Midline equals the average of the 20-period highest high and lowest low. The resulting momentum oscillator, when combined with squeeze dots, produces entry signals: green dot (squeeze released) with rising momentum above zero triggers long entries, while negative falling momentum triggers shorts.

John Carter's published performance claims cite **75% win rates over 15+ years**, though these are marketing figures from Simpler Trading. Independent empirical validation provides more rigorous evidence. The TradeMachine study across 270 liquid stocks over 5 years found statistical significance at p<0.01 for bull squeezes and p<0.00001 for bear squeezes, both substantially outperforming standard directional strategies. The Superalgos cryptocurrency study (BTC/USDT, 2021) demonstrated transformation from -100% to +87.65% returns purely through risk management optimization—using Supertrend trend filters, 4×ATR stops, and 6×ATR targets reduced trades from 76 to 26 while achieving 54% win rate and +26.65% alpha versus buy-and-hold.

**Squeeze duration predicts breakout magnitude.** Carter's 5-dot minimum rule filters premature signals—each consecutive red dot represents another period of volatility compression, building potential energy. Empirical analysis shows optimal squeezes persist 5-6+ periods on daily charts (5-15 days typical), with longer compression correlating to more explosive releases. The typical post-squeeze move extends 8-10 bars with magnitude proportional to both compression duration and tightness (measured by Bollinger Band Width).

False squeeze identification requires multi-layer filtering. Validation criteria include: minimum 3-5 consecutive squeeze dots, volume confirmation (breakout volume >1.5× average), trend alignment via Supertrend or moving averages, momentum quality (clear directional histogram, not oscillating), and technical level confluence at support/resistance zones. Volume analysis proves particularly critical—squeezes breaking during major news events or low-volume periods produce 40-60% false signals.

## Trend persistence generates measurable statistical edge

Time-series momentum strategies have demonstrated **136 years of persistent profitability** across 67 global markets. The seminal Moskowitz, Ooi & Pedersen (2012) study in the Journal of Financial Economics documented Sharpe ratios of 0.60-0.70 across 58 instruments with monthly returns of 0.58% (t-statistic: 4.15). The follow-up "Century of Evidence" research (Hurst, Ooi & Pedersen, 2017) extended this analysis from 1880-2016, revealing positive returns in every single decade despite Depression, world wars, stagflation, and financial crises. This persistence appears in all 67 markets tested with average Sharpe ratios of 0.40-0.60.

**Trend strength measurement extends beyond ADX to specialized indicators.** The Aroon indicator (Tushar Chande, 1995) quantifies trend based on time elapsed since highest highs and lowest lows: `Aroon-Up = ((25 - Days_Since_25day_High)/25) × 100`. Strong uptrends maintain Aroon-Up values of 70-100 while Aroon-Down stays below 50. The Aroon Oscillator (Aroon-Up minus Aroon-Down) provides a single-line trend measure ranging from -100 to +100.

Supertrend (Olivier Seban, 2009) offers dynamic support/resistance bands using ATR-based calculations. The core formula: `Basic_Upper_Band = (High+Low)/2 + (Multiplier × ATR)` with default parameters of 10-period ATR and 3.0 multiplier. The indicator flips between upper and lower bands based on price action, producing binary trend signals—green when price trades above the line (uptrend), red when below (downtrend). This simplicity makes Supertrend ideal for algorithmic implementation and trend filtering.

**Parabolic SAR employs acceleration-based trailing stops** via the formula: `SAR(t+1) = SAR(t) + AF × [EP - SAR(t)]`, where AF (Acceleration Factor) starts at 0.02 and increases by 0.02 per new extreme point up to a maximum of 0.20. The EP (Extreme Point) represents the highest high in uptrends or lowest low in downtrends. Dots flip sides when price crosses the SAR level, providing clear reversal signals. PSAR works best on timeframes of 1-hour or greater where noise is reduced.

Pullback trading within established trends captures optimal entry points at Fibonacci retracement levels. The mathematical basis stems from the golden ratio φ ≈ 1.618, producing key retracement percentages: 23.6% (1-1/φ³), 38.2% (1-1/φ²), 61.8% (1-1/φ), and 78.6% (√0.618). **Empirical validation shows mixed but positive evidence.** Bhattacharya & Kumar (2006) found 6 of 7 critical S&P 500 retracements aligned with Fibonacci levels (p<0.05). Tsinaslanidis et al. (2022) documented positive relationships between Fibonacci zone width and bounce probability across Dow Jones, NASDAQ, and DAX from 1968-2019. Machine learning integration with Fibonacci levels improved predictive accuracy by 12% with lower RMSE scores. However, ForexOp analysis of 40,243 G10 currency corrections revealed 83% fell between 15%-61.8% with log-normal distribution, suggesting potential self-fulfilling prophecy effects rather than inherent mathematical properties.

**Continuation patterns exhibit quantifiable geometric properties.** Flags consist of a sharp directional move (flagpole) followed by a parallel channel consolidation sloping opposite to the trend, with declining volume. Detection algorithms identify: (1) initial rapid move exceeding 2% with high volume, (2) parallel consolidation with opposite slope, (3) declining volume during formation, (4) duration less than 20% of flagpole time, and (5) breakout with volume surge exceeding 1.5× average. Price targets equal the flagpole height projected from the breakout point, achieving 65-75% success rates with proper volume confirmation.

Pennants display converging trendlines forming symmetrical triangles after sharp moves, typically lasting 1-4 weeks. Geometric detection fits linear regression lines to ascending lows and descending highs, measuring convergence rate and verifying minimum 4 touchpoints. Breakout zones occur at 2/3 to 3/4 distance from the triangle base. **Triangles differ from pennants in formation time** (months versus weeks) and lack preceding flagpoles, making them independent rather than continuation patterns.

Multi-timeframe analysis reduces false signals by 40% and increases pattern success by 15-20%. Dr. Alexander Elder's Triple Screen Trading System provides the framework: Screen 1 (higher timeframe) identifies market tide via weekly/daily MACD histogram, Screen 2 (medium timeframe) detects setups using daily/4-hour oscillators, and Screen 3 (lower timeframe) triggers precise entries on 4-hour/1-hour breakouts. The scaling factor rule recommends higher timeframe = trading timeframe × 4-6. Empirical data shows win rate progression: 55% single timeframe, 65% dual timeframe alignment, 72% triple+ timeframe confluence.

**Trendline break validation prevents costly false signals.** ATR-based thresholds define valid breaks: `Break_Threshold = ATR × Multiplier` where conservative settings use 1.5-2.0× ATR, moderate use 1.0-1.5× ATR, and aggressive use 0.5-1.0× ATR. A valid break requires closing prices beyond `Trendline ± Threshold` rather than mere wick penetration. Volume confirmation (minimum 1.2×, strong 1.5×, very strong 2.0× average) substantially improves reliability. The retest strategy—waiting for pullback to broken trendline followed by rejection—achieves approximately 75% success versus 60% for immediate entry, improving risk/reward by 30-40%.

Trend exhaustion detection employs divergence analysis between price and momentum indicators. **Bullish exhaustion manifests as higher price highs with lower RSI/MACD highs**, signaling waning momentum despite continued price advance. Volume exhaustion appears as climax spikes (>3× average) producing wide-range bars followed by reversal, or progressive volume decline during continuation indicating waning participation. The Williams %R dual period method (standard 14-period plus long 112-period, optimized for crypto) identifies exhaustion when both oscillators simultaneously reach overbought/oversold extremes.

## Mean reversion exploits statistical price normalization

Bollinger Band mean reversion strategies capitalize on the mathematical property that prices tend to revert toward moving averages after extreme deviations. **Entry rules are mechanically simple but require precise execution:** long positions when price closes below the lower band (MB - 2σ), short positions when price closes above the upper band (MB + 2σ). Exit occurs when price crosses the middle band (mean reversion complete) or reaches the opposite band (overextension reversal).

The standard deviation calculation `σ = √(Σ(Price - SMA)² / n)` with n=20 periods produces bands that statistically contain 95% of price action under normal distribution assumptions. Parameter optimization studies show conservative implementations using k=2.5-3.0 standard deviations generate fewer but higher-quality signals, while aggressive k=1.5-2.0 produces higher frequency trading. **Empirical validation by Vu & Bhattacharyya (2024) documented 65-71% win rates in ranging markets** with 2.3% average return per trade in forex pairs, but performance deteriorates to 35-45% win rates during strong trends (ADX >30).

**Z-score methodology quantifies deviation magnitude** via the formula `Z = (Current_Price - Mean) / Standard_Deviation`, providing standardized entry/exit thresholds independent of price levels. Standard thresholds correspond to statistical probabilities: ±1.0σ captures 68%, ±2.0σ captures 95%, ±3.0σ captures 99.7% of normal distributions. Optimal entry thresholds for single-asset mean reversion use |Z| > 2.0 for initial positions, with scaling at Z=±3.0 and stop-losses at |Z| > 3.5-4.0. The rolling calculation typically employs 20-60 period lookback windows, with 20-day periods for short-term trading and 60-day for medium-term strategies.

Pairs trading and statistical arbitrage require rigorous cointegration testing rather than simple correlation analysis. **The Augmented Dickey-Fuller (ADF) test determines whether asset spreads are stationary**, a necessary condition for mean reversion profitability. The test equation `Δy_t = α + βt + γy_(t-1) + δ₁Δy_(t-1) + ... + ε_t` tests the null hypothesis γ=0 (unit root, non-stationary) against the alternative γ<0 (stationary). The test statistic DF_τ = γ̂ / SE(γ̂) must exceed critical values (typically -2.86 at 5% significance) to reject the null and confirm mean reversion properties.

**The Johansen test provides superior methodology for multi-asset cointegration** testing via Vector Error Correction Models. Unlike the two-step Engle-Granger procedure, Johansen tests all assets simultaneously using eigenvalue decomposition, providing order-independent results and identifying multiple cointegration relationships. The trace statistic `λ_trace(r) = -T Σ ln(1-λ̂_i)` tests the number of cointegration vectors. For practical pairs trading, the test extracts hedge ratios from eigenvectors: `Spread = Asset_A + β×Asset_B` where β normalizes the cointegration relationship.

Pairs trading execution employs Z-score thresholds on cointegrated spreads. **Position sizing follows the formula:** `Position_A = Capital × allocation% / Price_A` and `Position_B = β × Position_A` where β represents the hedge ratio from cointegration analysis. Entry occurs at spread Z-score exceeding ±2.0σ (30% position), with scaled entries at ±3.0σ (additional 30%) and ±4.0σ (final 40%). Exit triggers when spread reverts to zero (mean reversion complete) or correlation breaks below 0.3 (relationship deteriorated). Stop-losses activate when spreads exceed ±4.0σ or maximum holding periods (typically 20 days) expire without convergence.

**Gatev, Goetzmann & Rouwenhorst (2006) documented annualized excess returns of 11-12% from 1962-2002** with Sharpe ratios of 1.44 during 1997-2007 using PCA-based pair selection. However, performance degraded post-2003 with Sharpe ratios declining to 0.9, suggesting strategy decay from increased competition and algorithmic trading adoption. ETF-based strategies incorporating volume filters achieved 1.51 Sharpe ratios during 2003-2007, indicating proper execution and liquidity considerations remain profitable.

The Ornstein-Uhlenbeck process provides theoretical foundation for mean reversion modeling. **The stochastic differential equation** `dX_t = θ(μ - X_t)dt + σdW_t` describes mean-reverting dynamics where θ represents reversion speed, μ denotes long-term mean, σ quantifies volatility, and dW_t represents Brownian motion increments. The half-life calculation `t_half = ln(2)/θ` estimates mean reversion timescale—shorter half-lives indicate faster reversion and higher trading frequency opportunities.

Parameter estimation employs Maximum Likelihood Estimation on discrete price observations. The speed parameter `θ_hat = -ln(β_hat)/Δt` where β_hat derives from OLS regression. The stationary distribution `X_∞ ~ N(μ, σ²/2θ)` provides long-run price distribution assumptions. **Chakraborty & Kearns (2011) proved market-making profit from OU processes equals (K-z²)/2** where K represents total volatility (Σ|P_t - P_(t-1)|) and z equals net price change (P_T - P_0). Expected profit grows linearly as Ω(σT - σ²/2θ - (μ-X_0)²), demonstrating theoretical profitability under specific conditions.

**RSI oversold/overbought strategies require market-regime-dependent thresholds.** The standard 30/70 levels derived from Wilder's original work prove suboptimal in trending markets. Bull markets exhibit elevated RSI ranges of 40-80 (not 30-70), requiring overbought thresholds of 80+ and oversold of 40. Bear markets display depressed ranges of 20-60, requiring adjustment to 60 overbought and 20 oversold. Cardwell's range shift recognition improves strategy adaptation.

Short-term 2-3 day RSI with extreme thresholds (15/85) achieves superior performance to standard 14-day implementations. **QuantifiedStrategies documented 91% win rates** using 2-day RSI below 15 as oversold entry signals with exit above 85, though this remarkably high figure requires verification across extended timeframes and market conditions. The 2-3 day RSI demonstrates approximately 50% better performance than 14-day implementations, particularly effective in mean-reverting equity markets.

Transaction costs impose substantial drag on mean reversion profitability due to high trading frequency. **Annual cost calculations reveal significant impact:** if executing 250 trades per year with 10 basis points per round-trip, total costs equal 25% of capital, requiring 25% gross returns merely to break even. Chan (2008) demonstrated Sharpe ratio degradation from 4.8 without costs to 3.5 with 10 bps costs—a 27% reduction in risk-adjusted returns. However, strategies remained profitable despite significant impact.

Cost mitigation requires execution optimization. Limit orders versus market orders save full bid-ask spreads. VWAP (Volume-Weighted Average Price) execution reduces market impact by 20-40% in large orders. Position sizing below 10% of average daily volume minimizes price impact. **Frequency optimization proves most effective:** reducing holding periods from daily to 2-5 days cuts annual costs by 40-80%. Wider entry thresholds (2.5σ versus 2.0σ) generate fewer but higher-quality trades, improving net profitability.

Mean reversion performance depends critically on market regime. **ADX (Average Directional Index) provides regime classification:** ADX <20 indicates ranging markets favorable for mean reversion (65-75% win rates), ADX 20-30 represents transitional periods, and ADX >30 signals trending markets where mean reversion win rates plummet to 35-45%. Alternative regime detection employs volatility ratios (ATR_short / ATR_long) or Hurst exponent calculations where H<0.5 indicates mean-reverting behavior, H=0.5 suggests random walk, and H>0.5 confirms trending characteristics.

**Strategy decay accelerates during regime transitions**, with 45% win rate reductions documented during the first 20 days of regime change. Warning signals include volume profiles shifting beyond 2σ from 50-day averages, pair correlations dropping below 0.3, and spreads exceeding historical maxima by 50%. Historical performance from 1983-2023 shows stable mean reversion profitability with brief deterioration during the 2008 crisis, followed by recovery to mid-2000s performance levels.

## Strategy selection depends on market microstructure

Optimal strategy deployment requires matching methodology to prevailing market conditions. **Ranging markets (ADX <25, low directional movement) favor mean reversion strategies** employing 2.0σ Bollinger Band entries, 2-day RSI with 15/85 thresholds, or Z-score ±2.0σ entries. These conditions produce 65-75% win rates with Sharpe ratios of 1.5-2.5. Conversely, trending markets (ADX >30, sustained directional moves) demand trend-following approaches using time-series momentum, moving average crossovers, or continuation pattern breakouts, achieving 45-55% win rates but positive skew and crisis alpha properties.

Squeeze strategies occupy a transitional niche, identifying compression preceding regime changes from ranging to trending. **These strategies work across regimes** since volatility compression occurs before major directional moves regardless of trend direction. The mathematical independence of squeeze detection (Bollinger/Keltner relationship) from directional indicators (momentum histogram, Supertrend) enables adaptation to both bullish and bearish scenarios. This regime-agnostic property makes squeeze strategies particularly valuable for portfolio diversification.

Performance metrics reveal distinct risk/return profiles. **Trend-following strategies demonstrate positive skew** with many small losses offset by occasional large gains, producing low win rates (45-55%) but attractive long-term compounding. The 136-year persistence documented across 67 markets with Sharpe ratios of 0.40-0.88 and crisis alpha properties make trend strategies essential portfolio components despite drawdown periods. Mean reversion strategies show higher win rates (60-70%) but negative skew—many small gains with occasional large losses during regime breaks—requiring strict stop-loss discipline.

**Sharpe ratio comparisons across strategy types:**
- Buy & Hold (60/40 portfolio): 0.40, 7-8% annual return
- Time-Series Momentum: 0.60, 10-14% annual return  
- Mean Reversion (Bollinger/RSI): 1.5-2.5, 10-20% annual return (regime-dependent)
- TTM Squeeze: 0.9-1.5 estimated, 6-15% annual return
- Combined Multi-Strategy: 0.70-0.88, 12-18% annual return

The superior Sharpe ratios of mean reversion strategies reflect regime-specific application rather than universal superiority. **Proper regime detection and strategy switching prevent catastrophic drawdowns** when market character shifts. Mean reversion strategies applied during trending markets produce 30-50% drawdowns, while trend strategies in ranging markets suffer death-by-thousand-cuts through frequent small losses and whipsaws.

Correlation properties enable effective diversification. Time-series momentum exhibits 0.0-0.2 correlation to equity markets, providing crisis protection and portfolio convexity. Mean reversion strategies show 0.3-0.5 correlation to equities during normal markets but can experience correlation spikes during crashes when volatility extremes trigger simultaneous mean reversion signals across assets. Squeeze strategies demonstrate low correlation (0.1-0.3) to both trend and mean reversion approaches since volatility compression timing differs from trend persistence or statistical extremes.

**Portfolio construction combining all three strategy types** achieves optimal diversification through complementary exposure. Recommended allocation: 40% trend-following strategies (for long-term persistence and crisis alpha), 30% mean reversion strategies (for high Sharpe ratios in ranging markets), 30% squeeze/volatility breakout strategies (for regime transitions and explosive moves). This distribution balances positive skew from trend strategies, high win rates from mean reversion, and concentrated returns from squeeze plays.

Risk management parameters must adapt to strategy type. **Trend strategies require wider stops** (2-3× ATR) to avoid normal retracement whipsaws, accepting win rates near 50% while capturing large moves. Mean reversion strategies demand tighter stops (2.0-2.5× ATR) since thesis invalidation occurs quickly when ranges break. Squeeze strategies benefit from volatility-adjusted stops using ATR multiples that expand during low volatility (preventing premature stopouts) and contract during high volatility (protecting capital during failed breakouts).

Maximum portfolio heat (total capital at risk across all positions) should not exceed 6-10%. Individual position sizing of 1-2% risk per trade with maximum 5-6 concurrent positions prevents correlation risk during market dislocations. **Mean reversion strategies require additional correlation monitoring**—pairs trading portfolios should maintain maximum 20% correlation between pairs to ensure diversification. If pair correlations exceed 0.8 across multiple positions, systematic risk concentration demands position reduction.

## Pine Script implementation requires precision

TradingView Pine Script Version 5 provides comprehensive indicator libraries and plotting capabilities for implementing these strategies. **The TTM Squeeze implementation** requires careful attention to calculation methodology, particularly the momentum histogram using linear regression.

**TTM Squeeze Pine Script Framework:**

```pinescript
//@version=5
indicator("TTM Squeeze Pro", overlay=false)

// Input Parameters
length = input.int(20, "BB/KC Length")
mult = input.float(2.0, "BB Multiplier")
multKC = input.float(1.5, "KC Multiplier")

// Bollinger Bands
src = close
basis = ta.sma(src, length)
dev = mult * ta.stdev(src, length)
upperBB = basis + dev
lowerBB = basis - dev

// Keltner Channels  
sma = ta.sma((high + low + close) / 3, length)
range_1 = high - low
rangema = ta.sma(ta.tr, length)
upperKC = sma + rangema * multKC
lowerKC = sma - rangema * multKC

// Squeeze Detection
sqzOn = lowerBB > lowerKC and upperBB < upperKC
sqzOff = lowerBB < lowerKC and upperBB > upperKC
noSqz = sqzOn == false and sqzOff == false

// Momentum Calculation
highest_high = ta.highest(high, length)
lowest_low = ta.lowest(low, length)  
avg1 = math.avg(highest_high, lowest_low)
avg2 = math.avg(avg1, ta.sma(close, length))
val = ta.linreg(close - avg2, length, 0)

// Plotting
plot(val, color=val > 0 ? (val > val[1] ? color.new(color.lime, 0) : color.new(color.green, 0)) : (val < val[1] ? color.new(color.red, 0) : color.new(color.orange, 0)), style=plot.style_histogram, linewidth=3)

plotshape(sqzOn, title="Squeeze On", location=location.bottom, color=color.red, style=shape.circle, size=size.tiny)
plotshape(sqzOff, title="Squeeze Off", location=location.bottom, color=color.lime, style=shape.circle, size=size.tiny)
```

**Supertrend implementation** requires proper ATR calculation and band switching logic:

```pinescript
//@version=5
indicator("Supertrend", overlay=true)

atrPeriod = input.int(10, "ATR Period")
factor = input.float(3.0, "Multiplier")

[supertrend, direction] = ta.supertrend(factor, atrPeriod)

plot(direction < 0 ? supertrend : na, "Up Trend", color = color.green, linewidth=2)
plot(direction > 0 ? supertrend : na, "Down Trend", color = color.red, linewidth=2)

// Entry Signals
longCondition = ta.crossover(close, supertrend)
shortCondition = ta.crossunder(close, supertrend)

plotshape(longCondition, title="Long", location=location.belowbar, color=color.green, style=shape.triangleup, size=size.small)
plotshape(shortCondition, title="Short", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.small)
```

**Bollinger Band mean reversion with Z-score:**

```pinescript
//@version=5
indicator("BB Mean Reversion with Z-Score", overlay=true)

length = input.int(20, "Length")
mult = input.float(2.0, "Std Dev Multiplier")
zThreshold = input.float(2.0, "Z-Score Entry Threshold")

// Bollinger Bands
basis = ta.sma(close, length)
dev = mult * ta.stdev(close, length)
upper = basis + dev
lower = basis - dev

// Z-Score Calculation
zScore = (close - basis) / ta.stdev(close, length)

// Entry Conditions
longEntry = zScore < -zThreshold
shortEntry = zScore > zThreshold
exitLong = close > basis or zScore > zThreshold  
exitShort = close < basis or zScore < -zThreshold

// Plotting
plot(basis, "Middle Band", color=color.blue)
p1 = plot(upper, "Upper Band", color=color.red)
p2 = plot(lower, "Lower Band", color=color.green)
fill(p1, p2, color=color.new(color.gray, 90))

bgcolor(longEntry ? color.new(color.green, 90) : na)
bgcolor(shortEntry ? color.new(color.red, 90) : na)

// Z-Score Display
plot(zScore, "Z-Score", color=color.orange, display=display.data_window)
hline(zThreshold, "Upper Threshold", color=color.red, linestyle=hline.style_dashed)
hline(-zThreshold, "Lower Threshold", color=color.green, linestyle=hline.style_dashed)
hline(0, "Zero Line", color=color.gray)
```

**Multi-timeframe trend alignment:**

```pinescript
//@version=5
indicator("MTF Trend Alignment", overlay=true)

htfRes = input.timeframe("240", "Higher Timeframe")

// Current Timeframe EMA
ema21 = ta.ema(close, 21)
ema50 = ta.ema(close, 50)

// Higher Timeframe EMA  
htfEma21 = request.security(syminfo.tickerid, htfRes, ta.ema(close, 21))
htfEma50 = request.security(syminfo.tickerid, htfRes, ta.ema(close, 50))

// Trend Detection
ctfTrendUp = close > ema21 and ema21 > ema50
ctfTrendDown = close < ema21 and ema21 < ema50

htfTrendUp = close > htfEma21 and htfEma21 > htfEma50  
htfTrendDown = close < htfEma21 and htfEma21 < htfEma50

// Aligned Signals
alignedBullish = ctfTrendUp and htfTrendUp
alignedBearish = ctfTrendDown and htfTrendDown

// Plotting
plot(ema21, "EMA 21", color=color.blue, linewidth=2)
plot(ema50, "EMA 50", color=color.orange, linewidth=2)

bgcolor(alignedBullish ? color.new(color.green, 85) : alignedBearish ? color.new(color.red, 85) : na)

// Signals
plotshape(ta.crossover(close, ema21) and htfTrendUp, title="Aligned Long", location=location.belowbar, color=color.lime, style=shape.triangleup, size=size.normal)
plotshape(ta.crossunder(close, ema21) and htfTrendDown, title="Aligned Short", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.normal)
```

**Pairs trading cointegration spread monitoring:**

```pinescript
//@version=5
indicator("Pairs Trading Z-Score", overlay=false)

symbol1 = input.symbol("AAPL", "Asset 1")
symbol2 = input.symbol("MSFT", "Asset 2")  
hedgeRatio = input.float(1.0, "Hedge Ratio (Beta)")
lookback = input.int(20, "Z-Score Lookback")
entryThreshold = input.float(2.0, "Entry Z-Score")

// Get Price Data
price1 = request.security(symbol1, timeframe.period, close)
price2 = request.security(symbol2, timeframe.period, close)

// Calculate Spread
spread = math.log(price1) - hedgeRatio * math.log(price2)

// Z-Score Calculation  
spreadMean = ta.sma(spread, lookback)
spreadStd = ta.stdev(spread, lookback)
zScore = (spread - spreadMean) / spreadStd

// Entry Signals
longSpread = zScore < -entryThreshold  // Long Asset1, Short Asset2
shortSpread = zScore > entryThreshold  // Short Asset1, Long Asset2  
exitSignal = math.abs(zScore) < 0.5

// Plotting
plot(zScore, "Spread Z-Score", color=color.blue, linewidth=2)
hline(0, "Mean", color=color.gray)
hline(entryThreshold, "Upper Threshold", color=color.red, linestyle=hline.style_dashed)
hline(-entryThreshold, "Lower Threshold", color=color.green, linestyle=hline.style_dashed)
hline(3.0, "Stop Loss", color=color.maroon, linestyle=hline.style_dotted)
hline(-3.0, "Stop Loss", color=color.maroon, linestyle=hline.style_dotted)

bgcolor(longSpread ? color.new(color.green, 90) : shortSpread ? color.new(color.red, 90) : exitSignal ? color.new(color.yellow, 95) : na)
```

**Strategy alerts and automation** require precise condition definitions. Pine Script strategy() function enables backtesting and forward testing with realistic execution assumptions:

```pinescript
//@version=5
strategy("Mean Reversion Strategy", overlay=true, default_qty_type=strategy.percent_of_equity, default_qty_value=10, commission_type=strategy.commission.percent, commission_value=0.1)

// Parameters
bbLength = input.int(20, "BB Length")
bbMult = input.float(2.0, "BB Multiplier")  
rsiLength = input.int(14, "RSI Length")
rsiOversold = input.int(30, "RSI Oversold")
rsiOverbought = input.int(70, "RSI Overbought")
atrMult = input.float(2.0, "ATR Stop Multiplier")

// Indicators
[middle, upper, lower] = ta.bb(close, bbLength, bbMult)
rsi = ta.rsi(close, rsiLength)
atr = ta.atr(14)

// Entry Conditions  
longCondition = close < lower and rsi < rsiOversold
shortCondition = close > upper and rsi > rsiOverbought

// Exit Conditions
longExit = close > middle or rsi > 50
shortExit = close < middle or rsi < 50

// Stop Loss Levels
longStop = strategy.position_avg_price - atr * atrMult
shortStop = strategy.position_avg_price + atr * atrMult

// Execute Trades
if longCondition and strategy.position_size == 0
    strategy.entry("Long", strategy.long)
    strategy.exit("Long Exit", "Long", stop=longStop, limit=upper)

if shortCondition and strategy.position_size == 0
    strategy.entry("Short", strategy.short)  
    strategy.exit("Short Exit", "Short", stop=shortStop, limit=lower)

if longExit and strategy.position_size > 0
    strategy.close("Long")

if shortExit and strategy.position_size < 0
    strategy.close("Short")

// Plotting
plot(middle, "Middle Band", color=color.blue)
p1 = plot(upper, "Upper Band", color=color.red)
p2 = plot(lower, "Lower Band", color=color.green)
fill(p1, p2, color=color.new(color.gray, 90))

plotshape(longCondition, title="Long Signal", location=location.belowbar, color=color.lime, style=shape.triangleup, size=size.small)
plotshape(shortCondition, title="Short Signal", location=location.abovebar, color=color.red, style=shape.triangledown, size=size.small)
```

Parameter optimization requires systematic testing across multiple dimensions. **Avoid overfitting by using out-of-sample validation**—optimize parameters on 70% of historical data, then validate on remaining 30%. Walk-forward analysis provides more robust validation by repeatedly optimizing on rolling training windows and testing on subsequent out-of-sample periods. Minimum statistical requirements include 30+ trades for initial validation and 100+ trades for production deployment confidence.

## Critical implementation factors determine profitability

**Slippage and execution assumptions must reflect reality.** Backtests using hypothetical fills at exact indicator trigger prices overestimate profitability by 15-40%. Realistic assumptions for retail traders include: 2-5 basis points slippage on liquid stocks, 5-10 basis points on mid-caps, 10-20 basis points on small-caps. Market orders during volatile periods can experience 20-50 basis point slippage. Limit orders reduce slippage but introduce fill uncertainty—approximately 30-40% of limit orders fail to execute during rapid moves.

Commission structures vary substantially across brokers. Zero-commission equity brokers (Robinhood, Webull) route orders to market makers who profit from payment-for-order-flow, potentially providing inferior execution. Interactive Brokers and similar platforms charge explicit commissions ($0.0005-$0.005 per share) but often provide superior execution through smart routing. **High-frequency mean reversion strategies** trading 100+ times annually face 5-15% performance drag from combined commissions, spreads, and slippage even under favorable conditions.

Data quality issues corrupt backtests through survivorship bias, look-ahead bias, and adjustment errors. Survivorship bias occurs when historical databases exclude delisted stocks—testing only current constituents inflates returns by 1-3% annually since failed companies disappear from analysis. Proper backtesting requires point-in-time databases containing all historical constituents including bankruptcies and delistings. Yahoo Finance and similar free data sources contain adjustment errors, missing data, and survivorship bias requiring verification against institutional-quality databases.

**Walk-forward optimization prevents curve-fitting disasters.** The process: (1) divide data into sequential windows, (2) optimize parameters on initial training window, (3) test optimized parameters on subsequent out-of-sample window, (4) roll windows forward and repeat, (5) aggregate out-of-sample results. If out-of-sample Sharpe ratio exceeds 50% of in-sample Sharpe, parameters likely generalize. If out-of-sample performance collapses, parameters are overfit.

Position sizing using fractional Kelly criterion balances growth maximization and drawdown control. **Full Kelly formula:** `f = (p×b - q) / b` where p = win probability, q = loss probability (1-p), b = win/loss ratio. Full Kelly sizing frequently produces excessive volatility and drawdowns exceeding 50%. Half-Kelly (0.5× full Kelly) or quarter-Kelly (0.25× full Kelly) provide more conservative growth with substantially reduced volatility. For mean reversion strategies with 65% win rates and 1.3 win/loss ratios, full Kelly suggests 22% position sizes—practically suicidal. Quarter-Kelly produces 5.5% position sizes, translating to 1% risk per trade with 2:1 reward/risk targets.

**Regime detection algorithms require continuous monitoring and adaptation.** Hidden Markov Models identify latent market states (ranging, trending, volatile) from observable data (returns, volatility, correlation). The Viterbi algorithm decodes most likely state sequences. Simpler alternatives include ADX thresholds (ADX <20 ranging, 20-30 transitional, >30 trending), volatility percentile rankings (>75th percentile favors mean reversion, <25th percentile challenges all strategies), or correlation regime monitoring (cross-asset correlation >0.7 indicates risk-on, <0.3 indicates risk-off).

Transaction cost optimization extends beyond execution to strategic design. **Turnover reduction techniques** include: widening entry thresholds by 10-20% (2.2σ instead of 2.0σ reduces trades by approximately 30%), implementing minimum holding periods (preventing immediate reversals from noise), using time-based filters (avoiding first/last 30 minutes when spreads widen), and portfolio rebalancing at fixed intervals rather than continuous monitoring.

**Monte Carlo simulation quantifies strategy robustness** under parameter uncertainty. Generate 10,000+ alternate histories by bootstrap resampling returns with replacement or randomizing trade sequences. Calculate percentile distributions of key metrics: median Sharpe ratio, 5th percentile maximum drawdown, 95th percentile return. If 5th percentile outcomes remain acceptable, strategy demonstrates robustness. If median Monte Carlo performance differs substantially from backtest, overfitting or order dependency effects exist.

Risk management transcends stop-loss placement to encompass portfolio construction, correlation management, and capital preservation during regime changes. **Maximum portfolio heat guidelines:** never risk more than 2% on single trades, 6% across uncorrelated strategies, 10% absolute maximum including correlated positions. Dynamic position sizing scales exposure inversely with volatility: `Position_Size = Base_Size × (Target_Vol / Current_Vol)`. This volatility targeting maintains consistent risk exposure as market conditions change.

## Synthesis reveals complementary strategy deployment

The mathematical foundations, empirical evidence, and implementation considerations reveal that squeeze, trend, and mean reversion strategies operate on fundamentally different market dynamics. **Optimal deployment requires matching strategy to market microstructure** rather than pursuing single-strategy dominance.

Mean reversion capitalizes on statistical properties of bounded price oscillations, achieving highest win rates (60-75%) and Sharpe ratios (1.5-2.5) during ranging markets. The strategies excel in liquid, stable assets with established trading ranges—large-cap stocks, currency crosses, and interest rate products. Implementation demands rigorous cointegration testing for pairs trading, frequent parameter re-optimization, and immediate strategy disengagement when regime indicators (ADX, Hurst exponent, correlation matrices) signal trend emergence.

Trend-following exploits behavioral finance phenomena producing delayed information incorporation and momentum persistence. The 136-year empirical record across 67 markets demonstrates genuine edge independent of data mining. **Positive skew and crisis alpha properties** make trend strategies essential diversifiers despite modest win rates (45-55%). These strategies perform during precisely those market dislocations when mean reversion fails catastrophically—the 2008 crisis, COVID crash, and similar events. Time-series momentum on 12-month lookbacks with 1-month holding periods represents the academically validated implementation, achieving 0.60-0.70 Sharpe ratios.

Squeeze strategies identify transitional periods when volatility compression precedes directional expansion. The mathematical independence of detection (Bollinger/Keltner relationship) from directional prediction (momentum histogram, Supertrend filter) enables adaptation across regimes. **The 5-dot minimum rule with Supertrend confirmation** provides empirically validated filtering, transforming raw signals from 54% win rates to 75% with proper risk management. These strategies complement both trend and mean reversion by capturing explosive moves at regime inflection points.

Portfolio construction allocating 40% to trend strategies, 30% to mean reversion, and 30% to squeeze/volatility approaches balances complementary exposures while maintaining 0.1-0.3 inter-strategy correlations. This distribution provides: (1) trend strategy crisis protection during market dislocations, (2) mean reversion high-frequency profits during stability, and (3) squeeze strategy concentrated returns during regime changes. Aggregate portfolio Sharpe ratios of 0.88-1.2 appear achievable with disciplined implementation.

**The critical success factor remains disciplined execution** rather than strategy selection. Academic research and empirical backtests demonstrate edge exists in all three methodologies when properly implemented. However, transaction costs, slippage, parameter drift, regime changes, and psychological factors destroy the majority of retail algorithmic trading attempts. Realistic execution assumptions, rigorous walk-forward validation, continuous parameter monitoring, strict risk management, and regime-aware strategy switching separate profitable implementation from theoretical possibility.