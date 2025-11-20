Date: 2025-01-19 Response: 6

## Technical Specification: Exit Decision Support Framework
**Version 1.0 | Research-Grade Implementation Specification**

---

## Executive Summary

**Indicator Name:** MKN Exit Framework (MEF)  
**Purpose:** Decision support for profit-taking and exit timing (NOT entry generation)  
**Target User:** Manual investors with good entry intuition, poor exit discipline  
**Complexity:** Low (3 core components, 4 decision rules)  
**Expected Accuracy:** 60-70% exit signal reliability (friction-adjusted, realistic for OHLCV)  
**Development Time:** 16 hours (2 days)  
**Lines of Code:** ~300 (Pine Script v5/v6)

---

## Research Foundation

### Core Citations

**[1] Supertrend Trend Detection**
> "Olivier Seban (2009). Binary trend signal: Green when price > band (uptrend), 
> Red when price < band (downtrend). Default: ATR(10), Multiplier 3.0"
> 
> **Source:** Algorithmic Trading Strategies, Section "Trend Strength Indicators"

**[2] Support/Resistance Level Detection**
> "Volume Profile achieves 75-85% accuracy with sufficient volume data (100+ bars). 
> Ensemble methods combining multiple detection approaches improve robustness."
> 
> **Source:** S/R Algorithmic Research, Algorithm 1 + Algorithm 5

**[3] Multi-Indicator Momentum Analysis**
> "Combining volume, RSI, and MACD reduces individual indicator errors. 
> Expected composite accuracy: 60-70% for trend continuation prediction."
> 
> **Source:** Algorithmic Trading Strategies, Section "Trend Exhaustion"

**[4] Exit Timing Performance Bounds**
> "OHLCV data alone, detection accuracy peaks at 65-75% for multi-day patterns. 
> Exit timing with OHLCV typically achieves 60-70% accuracy after friction costs."
> 
> **Source:** Institutional Order Flow Detection, Section "Detection Capability"

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    MKN EXIT FRAMEWORK (MEF)                     │
│                  Decision Support Dashboard                      │
└─────────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
     ┌──────▼──────┐   ┌─────▼─────┐   ┌──────▼──────┐
     │  COMPONENT 1 │   │ COMPONENT 2│   │ COMPONENT 3 │
     │    TREND     │   │   S/R      │   │  MOMENTUM   │
     │  DETECTION   │   │  LEVELS    │   │   HEALTH    │
     └──────┬──────┘   └─────┬─────┘   └──────┬──────┘
            │                 │                 │
            └─────────────────┼─────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │  DECISION ENGINE  │
                    │   (4 Exit Rules)  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  VISUAL OUTPUT    │
                    │ • Chart overlays  │
                    │ • Info dashboard  │
                    │ • Alert triggers  │
                    └───────────────────┘
```

---

## Component 1: Trend Detection

### 1.1 Mathematical Definition

**Method:** Supertrend indicator (Seban, 2009)

**Calculation:**
```
Step 1: Calculate basis
basis = (high + low) / 2

Step 2: Calculate ATR bands
atr = ATR(atrLength)
upperBand = basis + (multiplier × atr)
lowerBand = basis - (multiplier × atr)

Step 3: Apply trailing logic
IF close[1] ≤ upperBand[1] THEN
    supertrend = lowerBand
ELSE IF close > upperBand THEN
    supertrend = upperBand
ELSE
    supertrend = supertrend[1]

Step 4: Determine direction
direction = close > supertrend ? -1 : 1  // -1 = bullish, 1 = bearish
```

**Pine Script Implementation:**
```pinescript
// Use built-in ta.supertrend() function
[supertrend, direction] = ta.supertrend(multiplier, atrLength)

// Trend classification
isBullish = direction < 0
isBearish = direction > 0
```

### 1.2 Parameters (Research-Backed)

**[Citation: Algorithmic Trading Strategies, Supertrend section]**

| Parameter | Default | Range | Asset-Specific Adjustment |
|-----------|---------|-------|---------------------------|
| `atrLength` | 10 | 7-14 | 10 (universal) |
| `multiplier` | 3.0 | 2.0-4.0 | Stocks: 2.5, Crypto: 3.5, Futures: 3.0 |

**Justification:**
> "Default parameters: 10-period ATR and 3.0 multiplier. Works best on 
> timeframes 1-hour or greater where noise is reduced."

**Asset-Specific Defaults:**
```pinescript
// Auto-detect asset type
assetMultiplier = syminfo.type == "crypto" ? 3.5 :
                  syminfo.type == "futures" ? 3.0 :
                  2.5  // stocks (less volatile)

// User can override
multiplier = input.float(assetMultiplier, "Supertrend Multiplier", 
                         minval=2.0, maxval=4.0, step=0.1)
```

### 1.3 Visual Output Requirements

**Chart Overlay:**
```
Line Style: Solid, 3px width
Color: 
  - Green (#00E676) when direction = bullish
  - Red (#FF5252) when direction = bearish
  
Background:
  - Light green (color.new(#00E676, 95)) when bullish
  - Light red (color.new(#FF5252, 95)) when bearish
```

**Info Table Row:**
```
Label: "TREND"
Value: "UPTREND ↑" (green) or "DOWNTREND ↓" (red)
```

### 1.4 Expected Accuracy

**[Citation: Algorithmic Trading Strategies, "Trend Persistence"]**
> "Time-series momentum strategies demonstrated Sharpe ratios of 0.60-0.70 
> across 58 instruments with 136 years of data."

**Realistic Expectation:**
- **Trend identification:** 70-75% (established baseline)
- **Flip timing:** 65-70% (lag inherent to trailing stops)
- **False flips:** 25-30% (whipsaws during ranging markets)

**Failure Modes:**
- ❌ Ranging markets (ADX < 20): High false flip rate
- ❌ Volatile regime transitions: Delayed trend change recognition
- ✅ Strong trends (ADX > 30): Excellent performance

---

## Component 2: Support/Resistance Levels

### 2.1 Mathematical Definition

**Method:** External ensemble indicator integration (sr-algo5-ensemble.pine)

**Input Specification:**
```pinescript
// Import top 3 levels only (reduce visual clutter)
srLevel1 = input.source(close, "S/R Level 1 (Strongest)",
                        tooltip="Connect to Top S/R Level 1 from sr-algo5-ensemble")
srLevel2 = input.source(close, "S/R Level 2",
                        tooltip="Connect to Top S/R Level 2 from sr-algo5-ensemble")
srLevel3 = input.source(close, "S/R Level 3",
                        tooltip="Connect to Top S/R Level 3 from sr-algo5-ensemble")
```

**Proximity Detection:**
```
Definition: Price is "at level" when within tolerance band

tolerance = 0.02  // 2% default (user adjustable)

atLevel1 = |close - srLevel1| / close < tolerance
atLevel2 = |close - srLevel2| / close < tolerance
atLevel3 = |close - srLevel3| / close < tolerance

atAnyLevel = atLevel1 OR atLevel2 OR atLevel3

levelType = (close > srLevel1 AND atLevel1) ? "RESISTANCE"
          : (close < srLevel1 AND atLevel1) ? "SUPPORT"
          : (close > srLevel2 AND atLevel2) ? "RESISTANCE"
          : (close < srLevel2 AND atLevel2) ? "SUPPORT"
          : (close > srLevel3 AND atLevel3) ? "RESISTANCE"
          : (close < srLevel3 AND atLevel3) ? "SUPPORT"
          : "BETWEEN LEVELS"
```

### 2.2 Parameters

| Parameter | Default | Range | Justification |
|-----------|---------|-------|---------------|
| `tolerance` | 2.0% | 0.5-5.0% | Daily charts: 2%, Intraday: 1%, Weekly: 3% |
| `maxLevels` | 3 | 1-5 | Visual clarity (more = clutter) |

**[Citation: S/R Research, Ensemble section]**
> "Top 5 strongest levels achieve 85-95% hold rate (theoretical projection, 
> unvalidated). Practical: 65-75% with OHLCV-only methods."

### 2.3 Visual Output Requirements

**Chart Overlay:**
```
Level 1 (Strongest):
  - Line: Solid white (#FFFFFF), 2px width
  - Style: Horizontal line (plot.style_line)
  - Label: "S/R #1" at right edge of chart

Level 2:
  - Line: Dashed gray (#808080, 70% opacity), 1.5px width
  - Label: "S/R #2"

Level 3:
  - Line: Dotted gray (#808080, 50% opacity), 1px width
  - Label: "S/R #3"

Proximity Highlight:
  - If atAnyLevel: Yellow vertical band (color.new(#FFEB3B, 80))
```

**Info Table Row:**
```
Label: "NEAREST LEVEL"
Value: 
  - "At Resistance #1 (0.5% away)" if approaching resistance
  - "At Support #2 (1.2% away)" if approaching support
  - "Between Levels" if in no-man's land
```

### 2.4 Expected Accuracy

**Realistic Expectation:**
- **Level identification:** 70-78% (ensemble ALPHA, unvalidated)
- **Level holds (price reverses):** 65-75% (after friction)
- **Breakthrough detection:** 70-75% (valid breaks vs. false breaks)

**Failure Modes:**
- ❌ Low volume breakouts: False S/R signals (requires volume confirmation)
- ❌ Regime shifts: Historical levels become irrelevant
- ✅ High volume rejections: Strong validation signal

---

## Component 3: Momentum Health Composite

### 3.1 Mathematical Definition

**Method:** Weighted ensemble of Volume, RSI, and MACD

**Sub-Component A: Volume Health (40% weight)**

**[Citation: Institutional Order Flow, Volume Analysis]**
> "Volume ratio (current / average) identifies institutional participation. 
> Thresholds: 1.3x = elevated, 1.5x = high, 2.0x = extreme."

```
Calculation:
volRatio = volume / SMA(volume, 20)

Score:
IF volRatio ≥ 2.0 THEN volScore = 100  // Extreme (Tier 3)
ELSE IF volRatio ≥ 1.5 THEN volScore = 85  // High (Tier 2)
ELSE IF volRatio ≥ 1.3 THEN volScore = 70  // Elevated (Tier 1)
ELSE IF volRatio ≥ 1.0 THEN volScore = 50  // Normal (Tier 0)
ELSE volScore = 25  // Below average (warning)
```

**Sub-Component B: RSI Health (30% weight)**

**[Citation: Algorithmic Trading Strategies, "RSI Oversold/Overbought"]**
> "Standard 30/70 levels prove suboptimal in trending markets. Bull markets 
> exhibit elevated ranges 40-80, bear markets 20-60."

```
Calculation:
rsi = RSI(close, 14)

Trend-Adjusted Score:
IF trend = UPTREND THEN
    IF rsi > 60 THEN rsiScore = 100      // Strong bullish momentum
    ELSE IF rsi > 50 THEN rsiScore = 75  // Healthy bullish
    ELSE IF rsi > 40 THEN rsiScore = 50  // Weakening
    ELSE rsiScore = 25                    // Divergence/reversal warning
    
ELSE IF trend = DOWNTREND THEN
    IF rsi < 40 THEN rsiScore = 100      // Strong bearish momentum
    ELSE IF rsi < 50 THEN rsiScore = 75  // Healthy bearish
    ELSE IF rsi < 60 THEN rsiScore = 50  // Weakening
    ELSE rsiScore = 25                    // Divergence/reversal warning
```

**Sub-Component C: MACD Health (30% weight)**

**[Citation: Algorithmic Trading Strategies, "Trend Exhaustion"]**
> "MACD histogram divergence (price new high, MACD lower high) signals 
> momentum weakening."

```
Calculation:
[macdLine, signalLine, histogram] = MACD(close, 12, 26, 9)

Alignment Check:
macdBullish = macdLine > signalLine
macdBearish = macdLine < signalLine

Trend-Adjusted Score:
IF trend = UPTREND THEN
    IF macdBullish AND histogram > histogram[1] THEN macdScore = 100  // Accelerating
    ELSE IF macdBullish THEN macdScore = 75  // Confirming
    ELSE macdScore = 0  // Diverging (warning)
    
ELSE IF trend = DOWNTREND THEN
    IF macdBearish AND histogram < histogram[1] THEN macdScore = 100  // Accelerating
    ELSE IF macdBearish THEN macdScore = 75  // Confirming
    ELSE macdScore = 0  // Diverging (warning)
```

**Composite Calculation:**
```
momentumHealth = (volScore × 0.4) + (rsiScore × 0.3) + (macdScore × 0.3)

Classification:
IF momentumHealth ≥ 75 THEN state = "STRONG"
ELSE IF momentumHealth ≥ 50 THEN state = "HEALTHY"
ELSE IF momentumHealth ≥ 25 THEN state = "WEAKENING"
ELSE state = "EXHAUSTED"
```

### 3.2 Parameters

| Parameter | Default | Range | Justification |
|-----------|---------|-------|---------------|
| `volPeriod` | 20 | 10-50 | Standard MA lookback |
| `rsiLength` | 14 | 7-21 | Wilder's original (14) |
| `macdFast` | 12 | 8-15 | Standard MACD (12, 26, 9) |
| `macdSlow` | 26 | 20-30 | Standard MACD |
| `macdSignal` | 9 | 7-12 | Standard MACD |
| `volWeight` | 0.4 | 0.2-0.6 | Highest weight (institutional signal) |
| `rsiWeight` | 0.3 | 0.2-0.4 | Medium weight |
| `macdWeight` | 0.3 | 0.2-0.4 | Medium weight |

**Weight Justification:**
Volume receives highest weight (40%) because:
1. Institutional participation indicator
2. Cannot be easily manipulated (unlike price)
3. Confirms genuine conviction vs. low-volume moves

### 3.3 Visual Output Requirements

**Info Table Display:**
```
┌─────────────────┬───────────────────────────────────┐
│ MOMENTUM        │ HEALTHY (68/100) ⚠️               │
├─────────────────┼───────────────────────────────────┤
│ ├─ Volume       │ ELEVATED (1.4x avg) [Score: 70]  │
│ ├─ RSI          │ 58 (Neutral) [Score: 75]         │
│ └─ MACD         │ BULLISH ✓ [Score: 75]            │
└─────────────────┴───────────────────────────────────┘

Color Coding:
- STRONG (75-100): Green (#00E676)
- HEALTHY (50-74): Lime (#C6FF00)
- WEAKENING (25-49): Yellow (#FFEB3B)
- EXHAUSTED (0-24): Red (#FF5252)
```

**Gauge Visualization (Optional Enhancement):**
```
Horizontal bar showing 0-100 scale:
[████████████████░░░░] 68/100 HEALTHY

Implementation using plot() with filled area:
plot(momentumHealth, "Momentum Gauge", momentumColor, 
     style=plot.style_area, display=display.data_window)
```

### 3.4 Expected Accuracy

**Realistic Expectation:**
- **Composite prediction:** 60-70% (combining reduces individual errors)
- **Volume alone:** 65-70% (strongest single indicator)
- **RSI alone:** 55-60% (moderate, trend-dependent)
- **MACD alone:** 55-60% (moderate, lagging)

**Improvement from Ensemble:**
```
Single indicator error rate: 35-40%
Combined (independent errors): 35% × 35% × 35% = 4.3% (theoretical)
Realistic (correlated errors): 30-40% (accounting for correlation)

Net accuracy: 60-70% (validated range)
```

---

## Decision Engine: Exit Rules

### 4.1 Rule Hierarchy

**Priority Order (Execute first match):**
1. **CRITICAL EXIT:** Trend break (immediate risk)
2. **PROFIT TARGET:** At resistance + weak momentum (opportunity to lock gains)
3. **WARNING:** Momentum exhaustion (prepare to exit)
4. **CAUTION:** Volume divergence (watch closely)
5. **HOLD:** Default state (no exit trigger)

### 4.2 Rule Definitions

**Rule 1: CRITICAL EXIT - Trend Break**

**Trigger Condition:**
```
trendBroken = ta.crossunder(close, supertrend) AND barstate.isconfirmed

Explanation:
- Close crosses below Supertrend line
- Bar must be confirmed (not repainting)
- Immediate exit signal (trend invalidated)
```

**Output:**
```
Visual: Red X marker (shape.xcross, size.large) above bar
Label: "EXIT: TREND BROKEN"
Alert: "CRITICAL: Trend line broken, exit position"
Table Decision: "EXIT NOW (Trend Broken)" [RED]
```

**Expected Accuracy:** 70-75% (trend breaks predict continuation in new direction)

---

**Rule 2: PROFIT TARGET - At Resistance + Weak Momentum**

**Trigger Condition:**
```
atResistanceLevel = atLevel1 AND (close > srLevel1 * 0.98)  // Within 2% above level
weakMomentum = momentumHealth < 50  // Weakening or Exhausted

takeProfitSignal = atResistanceLevel AND weakMomentum AND isBullish

Explanation:
- Price approaching/touching resistance
- Momentum below 50 (not strong enough to break through)
- Trend still bullish (not broken yet)
- Highest probability reversal zone
```

**Output:**
```
Visual: Yellow diamond (shape.diamond, size.normal) above bar
Label: "TAKE PROFIT"
Alert: "Profit opportunity: At resistance with weak momentum"
Table Decision: "TAKE PROFIT (Resistance + Weak)" [YELLOW]
```

**Expected Accuracy:** 60-70% (confluence of two factors improves odds)

---

**Rule 3: WARNING - Momentum Exhaustion**

**Trigger Condition:**
```
momentumExhausted = momentumHealth < 25  // Score 0-24

Explanation:
- Volume declining (below 1.0x average)
- RSI showing divergence or weakness
- MACD not confirming trend
- Exit soon, though not immediate crisis
```

**Output:**
```
Visual: Orange circle (shape.circle, size.small) above bar
Label: "WEAK MOMENTUM"
Alert: "Warning: Momentum exhausted, consider exit"
Table Decision: "EXIT (Momentum Dead)" [ORANGE]
```

**Expected Accuracy:** 55-65% (weakest signal, but important early warning)

---

**Rule 4: CAUTION - Volume Divergence**

**Trigger Condition:**
```
priceNewHigh = close > ta.highest(close[1], 20)  // New 20-bar high
volumeDeclining = volume < ta.sma(volume, 5)     // Below 5-bar average

volumeDivergence = priceNewHigh AND volumeDeclining AND isBullish

Explanation:
- Price making new highs
- But volume declining (participation waning)
- Classic exhaustion pattern
- Not immediate exit, but watch closely
```

**Output:**
```
Visual: Yellow label at high
Text: "⚠️ VOLUME DIVERGENCE"
Alert: "Caution: Price diverging from volume"
Table Decision: "CAUTION (Volume Diverging)" [YELLOW]
```

**Expected Accuracy:** 50-60% (divergences can persist in strong trends)

---

**Rule 5: HOLD - Default State**

**Trigger Condition:**
```
noExitSignals = NOT (trendBroken OR takeProfitSignal OR 
                     momentumExhausted OR volumeDivergence)
```

**Output:**
```
Table Decision:
IF momentumHealth ≥ 75 THEN "HOLD (Strong Momentum)" [GREEN]
ELSE IF momentumHealth ≥ 50 THEN "HOLD (Healthy)" [LIME]
ELSE "WATCH (Weakening)" [YELLOW]
```

**No visual markers on chart (reduces clutter)**

---

## Visual Output Specification

### 5.1 Chart Overlays

**Layer 1: Background Fill**
```pinescript
// Trend background (subtle)
bgcolor(isBullish ? color.new(#00E676, 95) : color.new(#FF5252, 95))

// Proximity to S/R (when within tolerance)
bgcolor(atAnyLevel ? color.new(#FFEB3B, 90) : na, title="At S/R Level")
```

**Layer 2: S/R Levels**
```pinescript
// Level 1 (strongest)
plot(srLevel1, "S/R #1", color.new(#FFFFFF, 0), linewidth=2, style=plot.style_line)

// Level 2
plot(srLevel2, "S/R #2", color.new(#808080, 30), linewidth=1.5, style=plot.style_linebr)

// Level 3
plot(srLevel3, "S/R #3", color.new(#808080, 50), linewidth=1, style=plot.style_circles)

// Labels at current bar (right edge)
if barstate.islast and not na(srLevel1)
    label.new(bar_index, srLevel1, "S/R #1", 
              style=label.style_label_left, color=color.white, size=size.small)
```

**Layer 3: Supertrend Line**
```pinescript
trendColor = isBullish ? color.new(#00E676, 0) : color.new(#FF5252, 0)
plot(supertrend, "Trend Line", trendColor, linewidth=3, style=plot.style_line)
```

**Layer 4: Signal Markers**
```pinescript
// Critical exit
plotshape(trendBroken, "Trend Broken", shape.xcross, location.abovebar,
          color.new(#FF5252, 0), size=size.large, text="EXIT")

// Take profit
plotshape(takeProfitSignal, "Take Profit", shape.diamond, location.abovebar,
          color.new(#FFEB3B, 0), size=size.normal, text="PROFIT")

// Weak momentum
plotshape(momentumExhausted, "Momentum Weak", shape.circle, location.abovebar,
          color.new(#FF9800, 0), size=size.small, text="WEAK")

// Volume divergence label
if volumeDivergence
    label.new(bar_index, high * 1.02, "⚠️ VOLUME DIVERGENCE",
              color=color.new(#FFEB3B, 20), textcolor=color.black,
              style=label.style_label_down, size=size.normal)
```

### 5.2 Info Table (Dashboard)

**Position:** Top-right corner  
**Dimensions:** 2 columns, 8 rows  
**Border:** 1px solid gray

**Table Structure:**
```
┌─────────────────┬───────────────────────────────────────────┐
│ ROW 0 (Header)  │ MKN EXIT FRAMEWORK           [colspan=2]  │
├─────────────────┼───────────────────────────────────────────┤
│ ROW 1           │ TREND            │ UPTREND ↑ [Green]      │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 2           │ POSITION         │ +8.5% above line       │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 3           │ NEAREST LEVEL    │ Resistance #1 (1.2%)   │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 4           │ MOMENTUM         │ HEALTHY (68/100) ⚠️     │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 5           │ ├─ Volume        │ ELEVATED (1.4x) [70]   │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 6           │ ├─ RSI           │ 58 [75]                │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 7           │ └─ MACD          │ BULLISH ✓ [75]         │
├─────────────────┼──────────────────┼─────────────────────────┤
│ ROW 8 (Footer)  │ DECISION         │ HOLD (Healthy) [Lime]  │
└─────────────────┴──────────────────┴─────────────────────────┘
```

**Implementation Pseudocode:**
```pinescript
var table dashboard = table.new(position.top_right, 2, 9, 
                                border_width=1, border_color=color.gray)

if barstate.islast
    // Header (merged cells)
    table.cell(dashboard, 0, 0, "MKN EXIT FRAMEWORK", 
               bgcolor=color.new(#1976D2, 30), text_color=color.white,
               text_size=size.normal)
    table.merge_cells(dashboard, 0, 0, 1, 0)
    
    // Row 1: Trend
    table.cell(dashboard, 0, 1, "TREND", 
               bgcolor=color.new(#424242, 70), text_color=color.white)
    trendText = isBullish ? "UPTREND ↑" : "DOWNTREND ↓"
    trendColor = isBullish ? #00E676 : #FF5252
    table.cell(dashboard, 1, 1, trendText,
               bgcolor=color.new(#424242, 70), text_color=trendColor)
    
    // Row 2: Position relative to trend
    distFromTrend = (close - supertrend) / supertrend * 100
    posText = str.tostring(distFromTrend, "#.0") + "% " + 
              (distFromTrend > 0 ? "above line" : "below line")
    table.cell(dashboard, 0, 2, "POSITION", ...)
    table.cell(dashboard, 1, 2, posText, ...)
    
    // Row 3: Nearest S/R level
    nearestLevel = atLevel1 ? "Resistance #1 (0.5%)" : 
                   atLevel2 ? "Support #2 (1.2%)" :
                   "Between Levels"
    table.cell(dashboard, 0, 3, "NEAREST LEVEL", ...)
    table.cell(dashboard, 1, 3, nearestLevel, ...)
    
    // Row 4: Momentum composite
    momText = momentumState + " (" + str.tostring(momentumHealth, "#") + "/100)"
    momColor = momentumHealth >= 75 ? #00E676 :
               momentumHealth >= 50 ? #C6FF00 :
               momentumHealth >= 25 ? #FFEB3B :
               #FF5252
    table.cell(dashboard, 0, 4, "MOMENTUM", ...)
    table.cell(dashboard, 1, 4, momText, text_color=momColor, ...)
    
    // Row 5-7: Sub-components (indented with "├─" and "└─")
    volText = volRatio >= 2.0 ? "EXTREME" : volRatio >= 1.5 ? "HIGH" :
              volRatio >= 1.3 ? "ELEVATED" : "NORMAL"
    table.cell(dashboard, 0, 5, "├─ Volume", ...)
    table.cell(dashboard, 1, 5, volText + " (" + str.tostring(volRatio, "#.0") + "x) [" + 
               str.tostring(volScore, "#") + "]", ...)
    
    // ... similar for RSI and MACD
    
    // Row 8: Decision (footer with merged cells)
    decision = trendBroken ? "EXIT NOW (Trend Broken)" :
               takeProfitSignal ? "TAKE PROFIT (Resistance + Weak)" :
               momentumExhausted ? "EXIT (Momentum Dead)" :
               volumeDivergence ? "CAUTION (Volume Diverging)" :
               momentumHealth >= 75 ? "HOLD (Strong Momentum)" :
               momentumHealth >= 50 ? "HOLD (Healthy)" :
               "WATCH (Weakening)"
    
    decisionColor = (trendBroken or momentumExhausted) ? #FF5252 :
                    (takeProfitSignal or volumeDivergence) ? #FFEB3B :
                    momentumHealth >= 75 ? #00E676 :
                    momentumHealth >= 50 ? #C6FF00 :
                    #FFEB3B
    
    table.cell(dashboard, 0, 8, "DECISION", 
               bgcolor=color.new(decisionColor, 50), text_color=color.white)
    table.cell(dashboard, 1, 8, decision,
               bgcolor=color.new(decisionColor, 50), text_color=color.white,
               text_size=size.normal)
```

### 5.3 Alert Conditions

**Alert 1: Trend Broken (CRITICAL)**
```pinescript
alertcondition(trendBroken, 
               title="CRITICAL: Trend Broken",
               message="EXIT: {{ticker}} closed below trend line at {{close}}. Trend broken.")
```

**Alert 2: Take Profit Opportunity**
```pinescript
alertcondition(takeProfitSignal,
               title="Profit Opportunity",
               message="PROFIT: {{ticker}} at resistance ({{close}}) with weak momentum. Consider exit.")
```

**Alert 3: Momentum Exhaustion**
```pinescript
alertcondition(momentumExhausted,
               title="Momentum Warning",
               message="WARNING: {{ticker}} momentum exhausted ({{momentumHealth}}/100). Watch closely.")
```

**Alert 4: Volume Divergence**
```pinescript
alertcondition(volumeDivergence,
               title="Volume Divergence",
               message="CAUTION: {{ticker}} new high on declining volume. Exhaustion possible.")
```

---

## Input Parameters (User Configuration)

### 6.1 Trend Settings
```pinescript
// Group: "Trend Detection"
atrLength = input.int(10, "ATR Length", minval=5, maxval=20,
                      tooltip="Periods for ATR calculation (default: 10)")

assetMultiplier = syminfo.type == "crypto" ? 3.5 : 
                  syminfo.type == "futures" ? 3.0 : 2.5
multiplier = input.float(assetMultiplier, "Supertrend Multiplier", 
                         minval=2.0, maxval=4.0, step=0.1,
                         tooltip="Auto-adjusted by asset type. Crypto: 3.5, Stocks: 2.5")
```

### 6.2 S/R Settings
```pinescript
// Group: "Support/Resistance Levels"
srLevel1 = input.source(close, "S/R Level 1 (Strongest)",
                        tooltip="Connect to Top S/R Level 1 from sr-algo5-ensemble")
srLevel2 = input.source(close, "S/R Level 2")
srLevel3 = input.source(close, "S/R Level 3")

tolerance = input.float(2.0, "Level Tolerance (%)", minval=0.5, maxval=5.0, step=0.1,
                        tooltip="Distance to consider 'at level' (default: 2%)")
```

### 6.3 Momentum Settings
```pinescript
// Group: "Momentum Health"
volPeriod = input.int(20, "Volume MA Period", minval=10, maxval=50)
rsiLength = input.int(14, "RSI Length", minval=7, maxval=21)
macdFast = input.int(12, "MACD Fast", minval=8, maxval=15)
macdSlow = input.int(26, "MACD Slow", minval=20, maxval=30)
macdSignal = input.int(9, "MACD Signal", minval=7, maxval=12)

volWeight = input.float(0.4, "Volume Weight", minval=0.2, maxval=0.6, step=0.05,
                        tooltip="Weight in momentum composite (default: 0.4)")
rsiWeight = input.float(0.3, "RSI Weight", minval=0.2, maxval=0.4, step=0.05)
macdWeight = input.float(0.3, "MACD Weight", minval=0.2, maxval=0.4, step=0.05)
```

### 6.4 Display Settings
```pinescript
// Group: "Display"
showSRLevels = input.bool(true, "Show S/R Levels",
                          tooltip="Display horizontal S/R lines on chart")
showDashboard = input.bool(true, "Show Info Dashboard")
showSignals = input.bool(true, "Show Exit Signals (markers)")
enableAlerts = input.bool(false, "Enable Alerts",
                          tooltip="Send notifications on exit signals")
```

---

## Implementation Pseudocode

```pinescript
//@version=5
indicator("MKN Exit Framework", shorttitle="MEF", overlay=true)

// ============================================================================
// SECTION 1: INPUTS
// ============================================================================
[... Parameter inputs from Section 6 ...]

// ============================================================================
// SECTION 2: COMPONENT 1 - TREND DETECTION
// ============================================================================
[supertrend, direction] = ta.supertrend(multiplier, atrLength)
isBullish = direction < 0
isBearish = direction > 0

// Plot trend line
trendColor = isBullish ? color.new(#00E676, 0) : color.new(#FF5252, 0)
plot(supertrend, "Trend Line", trendColor, linewidth=3)
bgcolor(isBullish ? color.new(#00E676, 95) : color.new(#FF5252, 95))

// ============================================================================
// SECTION 3: COMPONENT 2 - S/R LEVELS
// ============================================================================
toleranceDecimal = tolerance / 100.0
atLevel1 = math.abs(close - srLevel1) / close < toleranceDecimal
atLevel2 = math.abs(close - srLevel2) / close < toleranceDecimal
atLevel3 = math.abs(close - srLevel3) / close < toleranceDecimal
atAnyLevel = atLevel1 or atLevel2 or atLevel3

// Plot S/R levels
if showSRLevels
    plot(srLevel1, "S/R #1", color.white, linewidth=2)
    plot(srLevel2, "S/R #2", color.new(#808080, 30), linewidth=1.5)
    plot(srLevel3, "S/R #3", color.new(#808080, 50), linewidth=1)

bgcolor(atAnyLevel ? color.new(#FFEB3B, 90) : na)

// ============================================================================
// SECTION 4: COMPONENT 3 - MOMENTUM HEALTH
// ============================================================================
// Sub-component A: Volume
volRatio = volume / ta.sma(volume, volPeriod)
volScore = volRatio >= 2.0 ? 100 : volRatio >= 1.5 ? 85 : 
           volRatio >= 1.3 ? 70 : volRatio >= 1.0 ? 50 : 25

// Sub-component B: RSI
rsi = ta.rsi(close, rsiLength)
rsiScore = isBullish ? 
           (rsi > 60 ? 100 : rsi > 50 ? 75 : rsi > 40 ? 50 : 25) :
           (rsi < 40 ? 100 : rsi < 50 ? 75 : rsi < 60 ? 50 : 25)

// Sub-component C: MACD
[macdLine, signalLine, histogram] = ta.macd(close, macdFast, macdSlow, macdSignal)
macdBullish = macdLine > signalLine
macdAccelerating = histogram > histogram[1]
macdScore = isBullish ?
            (macdBullish and macdAccelerating ? 100 : macdBullish ? 75 : 0) :
            (not macdBullish and not macdAccelerating ? 100 : not macdBullish ? 75 : 0)

// Composite
momentumHealth = (volScore * volWeight) + (rsiScore * rsiWeight) + (macdScore * macdWeight)
momentumState = momentumHealth >= 75 ? "STRONG" :
                momentumHealth >= 50 ? "HEALTHY" :
                momentumHealth >= 25 ? "WEAKENING" :
                "EXHAUSTED"

// ============================================================================
// SECTION 5: DECISION ENGINE (EXIT RULES)
// ============================================================================
// Rule 1: Trend break (CRITICAL)
trendBroken = ta.crossunder(close, supertrend) and barstate.isconfirmed

// Rule 2: Take profit (at resistance + weak momentum)
atResistanceLevel = atLevel1 and (close > srLevel1 * 0.98)
weakMomentum = momentumHealth < 50
takeProfitSignal = atResistanceLevel and weakMomentum and isBullish

// Rule 3: Momentum exhaustion
momentumExhausted = momentumHealth < 25

// Rule 4: Volume divergence
priceNewHigh = close > ta.highest(close[1], 20)
volumeDeclining = volume < ta.sma(volume, 5)
volumeDivergence = priceNewHigh and volumeDeclining and isBullish

// Decision summary
decision = trendBroken ? "EXIT NOW (Trend Broken)" :
           takeProfitSignal ? "TAKE PROFIT (Resistance + Weak)" :
           momentumExhausted ? "EXIT (Momentum Dead)" :
           volumeDivergence ? "CAUTION (Volume Diverging)" :
           momentumHealth >= 75 ? "HOLD (Strong Momentum)" :
           momentumHealth >= 50 ? "HOLD (Healthy)" :
           "WATCH (Weakening)"

// ============================================================================
// SECTION 6: VISUAL OUTPUTS
// ============================================================================
// Signal markers
if showSignals
    plotshape(trendBroken, "Trend Broken", shape.xcross, location.abovebar,
              color.new(#FF5252, 0), size=size.large, text="EXIT")
    plotshape(takeProfitSignal, "Take Profit", shape.diamond, location.abovebar,
              color.new(#FFEB3B, 0), size=size.normal, text="PROFIT")
    plotshape(momentumExhausted, "Weak", shape.circle, location.abovebar,
              color.new(#FF9800, 0), size=size.small, text="WEAK")

// Volume divergence label
if volumeDivergence
    label.new(bar_index, high * 1.02, "⚠️ VOLUME DIVERGENCE",
              color=color.new(#FFEB3B, 20), textcolor=color.black,
              style=label.style_label_down, size=size.normal)

// Info table (dashboard)
if showDashboard
    [... Table implementation from Section 5.2 ...]

// ============================================================================
// SECTION 7: ALERTS
// ============================================================================
if enableAlerts
    alertcondition(trendBroken, "CRITICAL: Trend Broken", 
                   "EXIT: {{ticker}} closed below trend at {{close}}")
    alertcondition(takeProfitSignal, "Profit Opportunity",
                   "PROFIT: {{ticker}} at resistance with weak momentum")
    alertcondition(momentumExhausted, "Momentum Warning",
                   "WARNING: {{ticker}} momentum exhausted")
    alertcondition(volumeDivergence, "Volume Divergence",
                   "CAUTION: {{ticker}} new high on declining volume")
```

---

## Testing & Validation Protocol

### 7.1 Unit Tests (Component-Level)

**Test 1: Trend Detection**
```
Scenario A: Clean uptrend (SPY daily, Jan-Mar 2023)
Expected: Green line, "UPTREND ↑", green background
Pass Criteria: No false flips in sustained trend (< 5 flips in 60 bars)

Scenario B: Clean downtrend (SPY daily, Sep-Oct 2022)
Expected: Red line, "DOWNTREND ↓", red background
Pass Criteria: No false flips in sustained trend (< 5 flips in 60 bars)

Scenario C: Ranging market (SPY daily, Jul-Aug 2023)
Expected: Multiple flips (acceptable)
Pass Criteria: Captures regime correctly (chart inspection)
```

**Test 2: S/R Proximity Detection**
```
Scenario A: Price at resistance (within 2%)
Expected: Yellow background, "At Resistance" in table
Pass Criteria: Activates when |close - srLevel1| / close < 0.02

Scenario B: Price between levels
Expected: No yellow background, "Between Levels" in table
Pass Criteria: Correct identification of neutral zone

Scenario C: Price at support (within 2%)
Expected: Yellow background, "At Support" in table
Pass Criteria: Activates when price approaches from above
```

**Test 3: Momentum Composite**
```
Scenario A: Strong momentum (all 3 components high)
Expected: Score 75-100, "STRONG", green color
Test values: volRatio=1.8, RSI=65, MACD bullish + accelerating
Calculation: (85*0.4) + (100*0.3) + (100*0.3) = 34 + 30 + 30 = 94

Scenario B: Weak momentum (all 3 components low)
Expected: Score 0-25, "EXHAUSTED", red color
Test values: volRatio=0.8, RSI=45 (in uptrend), MACD bearish
Calculation: (25*0.4) + (50*0.3) + (0*0.3) = 10 + 15 + 0 = 25
```

### 7.2 Integration Tests (Full System)

**Test 1: Exit Signal at Trend Break**
```
Setup: Long position in uptrend, price crosses below Supertrend
Expected:
  ✓ Red X marker appears on chart
  ✓ Table shows "EXIT NOW (Trend Broken)" in red
  ✓ Alert fires (if enabled)
  ✓ Background turns red

Pass Criteria: Signal appears on bar where close < supertrend (no lag)
```

**Test 2: Take Profit at Resistance**
```
Setup: Price within 2% of resistance, momentum score 45
Expected:
  ✓ Yellow diamond marker
  ✓ Yellow background (at S/R)
  ✓ Table shows "TAKE PROFIT (Resistance + Weak)"
  ✓ Alert fires

Pass Criteria: Signal only when BOTH conditions met (not just proximity)
```

**Test 3: No Signal in Neutral Zone**
```
Setup: Price between levels, momentum healthy (60), trend intact
Expected:
  ✓ No markers on chart
  ✓ Table shows "HOLD (Healthy)" in lime
  ✓ No alerts

Pass Criteria: Clean chart, clear hold message
```

### 7.3 Acceptance Criteria

**PASS Requirements:**
1. ✅ All 3 components calculate correctly (unit tests pass)
2. ✅ Decision engine executes rules in priority order
3. ✅ Visual outputs match specification (colors, shapes, positions)
4. ✅ Info table displays correct data in real-time
5. ✅ Alerts fire on correct conditions (no false triggers)
6. ✅ No repainting (all signals use `barstate.isconfirmed`)
7. ✅ Performance: Loads within 3 seconds on 5000-bar chart
8. ✅ Code quality: <300 lines, properly commented, maintainable

**FAIL Criteria (Any one triggers failure):**
- ❌ Repainting signals (changes historical markers)
- ❌ False trend breaks (>10% in ranging markets over 100 bars)
- ❌ Math errors in momentum composite (off by >5 points)
- ❌ Missing S/R proximity detection (doesn't trigger at levels)
- ❌ Visual clutter (too many overlapping markers)
- ❌ Performance issues (>5 second load time)

---

## Performance Expectations (Realistic Bounds)

### 8.1 Accuracy by Component

| Component | Expected Accuracy | Confidence Interval | Data Source |
|-----------|------------------|---------------------|-------------|
| Trend Detection | 70-75% | ±5% | Research citation [1] |
| S/R Level Holds | 65-75% | ±8% | Research citation [2] (ALPHA) |
| Momentum Composite | 60-70% | ±7% | Research citation [3] |
| **Combined Exit Decision** | **60-70%** | **±10%** | Ensemble correlation |

**Interpretation:**
- 60% = **6 out of 10 exit signals correct** (above 50% random)
- 70% = **7 out of 10 correct** (good for manual support tool)
- NOT 85-90% (unrealistic with OHLCV-only data)

### 8.2 Expected Performance by Market Regime

**[Citation: Market Regime Detection Research, Empirical Evidence]**

| Regime | Trend Accuracy | S/R Accuracy | Momentum Accuracy | Combined |
|--------|---------------|--------------|-------------------|----------|
| Strong Trend (ADX >40) | **80-85%** | 70-75% | 65-70% | **70-75%** |
| Moderate Trend (ADX 25-40) | 70-75% | 65-75% | 60-70% | **60-70%** |
| Ranging (ADX <25) | **50-60%** ⚠️ | 70-80% | 55-65% | **55-65%** |
| High Volatility | 60-70% | 60-70% | 50-60% | **50-60%** ⚠️ |

**Key Insights:**
1. **Best performance:** Strong trending markets (70-75% combined)
2. **Worst performance:** Ranging/volatile markets (50-60% combined)
3. **Limitation:** Framework designed for trends, struggles in chop

**Recommendation for Users:**
> "This framework works best in trending markets. In ranging conditions (ADX <25), 
> reduce position sizes by 50% and tighten stops. Consider not trading during 
> high volatility regimes (ATR >1.5× average)."

### 8.3 Win Rate vs. Profit Factor

**Realistic Trading Outcomes:**

Assuming user follows signals:
```
Win rate: 65% (baseline expectation)
Average winner: +8% (trend continuation captured)
Average loser: -4% (stopped at trend break)

Profit factor = (0.65 × 8%) / (0.35 × 4%) = 5.2% / 1.4% = 3.71

Expectancy per trade = (0.65 × 8%) - (0.35 × 4%) = 3.8%
```

**Over 20 trades per year:**
```
Expected return = 20 × 3.8% = 76% (gross)
Less transaction costs (2% per round trip) = 76% - 40% = 36% net
```

**Critical Caveat:**
> "These projections assume disciplined adherence to ALL signals, including 
> exits. Most traders struggle with discipline, reducing real-world performance 
> by 30-50% from theoretical maximum."

---

## Known Limitations & Edge Cases

### 9.1 Fundamental Constraints

**Limitation 1: OHLCV Data Ceiling**

**[Citation: Institutional Order Flow, Section Bottom Line]**
> "OHLCV data alone, detection accuracy peaks at 65-75% for multi-day patterns"

**Implication:** This framework **CANNOT exceed 70-75% accuracy** regardless of sophistication. True institutional flow detection requires tick data (72-85% with Lee-Ready algorithm).

**User Communication:**
> "This tool provides decision support, not perfect predictions. Expect 6-7 
> correct signals out of 10. Use your judgment as final arbiter."

---

**Limitation 2: Lag in Trend Detection**

**Issue:** Supertrend uses trailing stops, creating **2-4 bar lag** in trend change recognition.

**Consequence:** Price may move 3-6% against position before exit signal triggers.

**Mitigation:** User should monitor manually during volatile periods, not rely solely on automated signal.

---

**Limitation 3: S/R Level Dependency**

**Issue:** Framework assumes sr-algo5-ensemble indicator is connected and accurate. That indicator is **ALPHA status (unvalidated)**.

**Consequence:** If S/R levels are wrong (35-40% error rate), "take profit at resistance" signals will have high false positive rate.

**Mitigation:** User should validate S/R levels visually before trusting signals. Consider manual override capability.

---

**Limitation 4: Regime Transition Failures**

**Issue:** Framework optimized for trending markets. During regime shifts (trend → range or vice versa), accuracy drops to **50-55%** (coin flip).

**[Citation: Market Regime Detection, Critical Limitations]**
> "Regime detection lag averaging 2-4 periods means positions suffer adverse 
> conditions before adjustment."

**Mitigation:** Display regime warning in dashboard:
```
IF ADX < 25 OR ATR Ratio > 1.5 THEN
    Show label: "⚠️ RANGING/VOLATILE: Signals less reliable"
```

---

### 9.2 Edge Cases

**Edge Case 1: Gap Openings**
```
Scenario: Price gaps below Supertrend at market open
Issue: Trend break signal triggers, but gap may fill intraday
Recommendation: Wait for 1-bar confirmation before exit
```

**Edge Case 2: False Resistance Touch**
```
Scenario: Price wicks to resistance but closes below
Issue: Proximity detection triggers, but not a true test
Recommendation: Require close within 1% (not just wick touch)
```

**Edge Case 3: Low Volume Days**
```
Scenario: Holiday trading, volume 0.3× average
Issue: Momentum composite unreliable (volume score = 25)
Recommendation: Display warning "Low volume day - signals unreliable"
```

**Edge Case 4: Divergence Persistence**
```
Scenario: Volume divergence lasts 10+ bars without reversal
Issue: User receives repeated warnings, grows complacent
Recommendation: Limit warning to first detection, then clear after 5 bars
```

---

## Documentation Requirements

### 10.1 User Manual (Markdown)

**Required Sections:**
1. **Quick Start** (2 minutes to first chart)
2. **Component Explanation** (Trend, S/R, Momentum)
3. **Decision Rules** (When to exit)
4. **Parameter Tuning** (Asset-specific adjustments)
5. **Interpretation Guide** (What signals mean)
6. **Troubleshooting** (Common issues)
7. **Limitations** (What it can't do)

**Sample Quick Start:**
```markdown
## Quick Start (5 Steps)

1. Add "MKN Exit Framework" to your chart
2. Add "sr-algo5-ensemble" to same chart
3. Open MEF settings → S/R Integration
4. Map each "S/R Level" input to ensemble outputs:
   - S/R Level 1 → Top S/R Level 1 (from ensemble)
   - S/R Level 2 → Top S/R Level 2
   - S/R Level 3 → Top S/R Level 3
5. Enable alerts (optional)

Done! Framework is now tracking your position.
```

### 10.2 Code Comments (Inline)

**Required Comment Blocks:**
1. File header (indicator description, version, author)
2. Section headers (large ASCII art separators)
3. Complex calculations (formula explanation)
4. Parameter justifications (why these values)
5. Edge case handling (why this check exists)

**Sample Section Header:**
```pinescript
// ============================================================================
// SECTION 3: COMPONENT 2 - SUPPORT/RESISTANCE LEVELS
// ============================================================================
// 
// Integrates with external sr-algo5-ensemble indicator to identify key
// price levels where trend may reverse. Uses proximity detection (2% default)
// to trigger "take profit" signals when price approaches resistance.
//
// Research Foundation:
// - Volume Profile achieves 75-85% accuracy (S/R Research, Algorithm 1)
// - Ensemble methods improve robustness (S/R Research, Algorithm 5)
//
// Expected Accuracy: 65-75% (ALPHA status, unvalidated)
//
// Limitations:
// - Depends on ensemble indicator accuracy (external dependency)
// - Proximity threshold may need adjustment for different assets
// - Only shows top 3 levels (user must connect ensemble correctly)
// ============================================================================
```

### 10.3 Version History (CHANGELOG.md)

**Format:**
```markdown
# Changelog

## [1.0.0] - 2025-01-XX (Initial Release)

### Added
- Component 1: Supertrend trend detection
- Component 2: S/R level proximity detection (top 3)
- Component 3: Momentum health composite (Volume + RSI + MACD)
- Decision engine with 4 exit rules
- Visual dashboard (info table)
- Alert conditions (4 types)

### Accuracy Expectations
- Trend detection: 70-75%
- S/R holds: 65-75% (ALPHA, unvalidated)
- Momentum: 60-70%
- Combined: 60-70% (realistic for OHLCV)

### Known Issues
- Ranging markets: Accuracy drops to 50-60%
- Gap openings: May trigger false exits
- S/R dependency: Requires ensemble indicator connection

### Research Citations
- [1] Supertrend (Seban 2009): Algorithmic Trading Strategies doc
- [2] S/R Detection: S/R Algorithmic Research doc
- [3] Momentum Analysis: Algorithmic Trading Strategies doc
- [4] Performance Bounds: Institutional Order Flow doc
```

---

## Delivery Checklist for Pine Script Developer

**Before submitting code, verify:**

- [ ] **Indicator name:** "MKN Exit Framework" (exact)
- [ ] **Pine Script version:** v5 or v6 (specify which)
- [ ] **Line count:** <350 lines (target: ~300)
- [ ] **All components:** Trend, S/R, Momentum implemented
- [ ] **Decision engine:** 4 rules in priority order
- [ ] **Visual outputs:** Chart overlays + info table + alerts
- [ ] **No repainting:** All signals use `barstate.isconfirmed`
- [ ] **Performance:** <3 second load on 5000 bars
- [ ] **Comments:** Every section documented
- [ ] **Parameters:** All inputs with tooltips
- [ ] **Default values:** Match specification exactly
- [ ] **Asset-specific:** Auto-adjust multiplier by asset type
- [ ] **Testing:** Unit tests pass (trend, S/R, momentum)
- [ ] **Integration:** Full system test passes
- [ ] **Edge cases:** Gap handling, low volume warnings implemented
- [ ] **Documentation:** User manual + code comments + changelog
- [ ] **Limitations:** Clearly stated in comments and manual
- [ ] **Accuracy disclosure:** 60-70% realistic expectation noted

---

## Final Specification Summary

**What You're Building:**
A **decision support dashboard** (not trading algorithm) that answers:
1. Is trend intact? (Supertrend: Green = yes, Red = no)
2. Am I at a key level? (S/R proximity: Yellow highlight = yes)
3. Is momentum healthy? (Composite score: 75-100 = strong, 0-25 = weak)
4. Should I exit? (4 rules: Trend break, Resistance + weak, Exhausted, Divergence)

**What You're NOT Building:**
- ❌ Entry signal generator (user finds entries)
- ❌ High-accuracy predictor (85-90% is impossible with OHLCV)
- ❌ Automated trading system (user makes final decision)
- ❌ Complex squeeze/MTF/ensemble system (keep it simple)

**Expected Outcome:**
A **clean, visual, 300-line indicator** that helps manual investors **take profits** at the right time, with **60-70% accuracy** (better than guessing), delivered in **2 days** (16 hours) of focused development.

**Success Criteria:**
User opens chart, sees clear trend line, 3 S/R levels, momentum score, and decision ("HOLD" or "EXIT" or "TAKE PROFIT"). No ambiguity. No clutter. Actionable.
