# Technical Debt Tracker - PineScript Algorithms

## Purpose
Track code duplication and technical debt across PineScript indicators due to lack of library/import support in PineScript v5/v6.

## Status Key
- 🔴 **HIGH** - Critical duplication, high maintenance burden
- 🟡 **MEDIUM** - Moderate duplication, manageable
- 🟢 **LOW** - Minor duplication, low impact

---

## Duplicated Code Modules

### 1. Regime Detection Logic 🔴 HIGH

**Duplicated Across:**
- `institutional-algo2-mtf-convergence.pine` (lines 99-122)
- `institutional-algo3-bayesian-regime.pine` (lines 48-88)
- `institutional-algo4-ensemble.pine` (lines 198-289)
- **WILL DUPLICATE IN**: S/R Detection algorithms (all 3)

**Components:**
```pinescript
// ATR-based volatility regime
currentATR = ta.atr(atrLength)
avgATR = ta.sma(ta.atr(atrLength), atrLookback)
atrRatio = avgATR > 0 and not na(avgATR) ? currentATR / avgATR : 1.0

isHighVol = atrRatio > 1.3
isLowVol = atrRatio < 0.7
isNormalVol = not isHighVol and not isLowVol
```

**Why Duplicated:**
- No cross-indicator library sharing in PineScript
- Each indicator needs independent regime classification

**Maintenance Risk:**
- If we change regime thresholds (1.3, 0.7), must update in 6+ files
- If we add new regime states, must propagate to all files

**Mitigation:**
- Document standard regime thresholds here
- Use search/replace carefully when updating
- Test all indicators after regime logic changes

**Standard Regime Parameters:**
```pinescript
ATR_LENGTH = 14
ATR_LOOKBACK = 50  // or 100 for longer-term
HIGH_VOL_THRESHOLD = 1.3  // 30% above average
LOW_VOL_THRESHOLD = 0.7   // 30% below average
```

---

### 2. ADX-Based Trend Detection 🟡 MEDIUM

**Duplicated Across:**
- `institutional-algo1-volume-efficiency.pine` (lines 127-139)
- `institutional-algo3-bayesian-regime.pine` (lines 57-61)
- `institutional-algo4-ensemble.pine` (lines 275-277)
- **WILL DUPLICATE IN**: S/R Statistical Peak Detection

**Components:**
```pinescript
[diPlus, diMinus, adxValue] = ta.dmi(14, 14)
isTrending = adxValue > 25  // or 20, varies by algo
isRanging = adxValue < 20
```

**Variance:**
- Algo 1: uses `minADX` input (default 20)
- Algo 3: hardcoded 25 for trending, 20 for ranging
- Algo 4: uses 25

**Standard ADX Parameters:**
```pinescript
ADX_LENGTH = 14
ADX_TRENDING_THRESHOLD = 25
ADX_RANGING_THRESHOLD = 20
```

---

### 3. Volume Calculations & Thresholds 🟡 MEDIUM

**Duplicated Across:**
- `institutional-algo1-volume-efficiency.pine` (lines 32-35)
- `institutional-algo2-mtf-convergence.pine` (lines 86-89)
- `institutional-algo3-bayesian-regime.pine` (lines 146-147)
- `institutional-algo4-ensemble.pine` (lines 43-45, 186-189, 299-300)
- **WILL DUPLICATE IN**: All S/R algorithms

**Components:**
```pinescript
avgVolume = ta.sma(volume, volumeLookback)
relativeVolume = avgVolume > 0 and not na(avgVolume) ? volume / avgVolume : 1.0
```

**NA Protection Pattern:**
```pinescript
// ALWAYS use this pattern:
value = denominator > 0 and not na(denominator) ? numerator / denominator : defaultValue
```

**Standard Volume Parameters:**
```pinescript
VOLUME_LOOKBACK = 20
VOLUME_THRESHOLD_LOW = 1.3   // Minimum for detection
VOLUME_THRESHOLD_MED = 1.5   // Standard threshold
VOLUME_THRESHOLD_HIGH = 2.0  // High volume threshold
```

---

### 4. EMA Trend Detection 🟢 LOW

**Duplicated Across:**
- `institutional-algo1-volume-efficiency.pine` (lines 127-134)
- `institutional-algo4-ensemble.pine` (lines 99-104)
- **MAY DUPLICATE IN**: S/R Multi-Timeframe algorithm

**Components:**
```pinescript
ema20 = ta.ema(close, 20)
ema50 = ta.ema(close, 50)
ema200 = ta.ema(close, 200)

isUptrend = close > ema50 and ema20 > ema50 and ema50 > ema200
isDowntrend = close < ema50 and ema20 < ema50 and ema50 < ema200
isRanging = not isUptrend and not isDowntrend
```

**Standard EMA Periods:**
```pinescript
EMA_SHORT = 20
EMA_MID = 50
EMA_LONG = 200
```

---

### 5. Volume Efficiency Calculation 🟡 MEDIUM

**Duplicated Across:**
- `institutional-algo1-volume-efficiency.pine` (lines 46-55)
- `institutional-algo3-bayesian-regime.pine` (lines 150-154)
- `institutional-algo4-ensemble.pine` (lines 47-53, 303-306)
- **WILL DUPLICATE IN**: S/R Volume Profile algorithm

**Components:**
```pinescript
priceRange = high - low
safeRange = math.max(priceRange, close * 0.001)  // Prevent division by zero
volumeEfficiency = volume / safeRange
avgVolumeEfficiency = ta.sma(volumeEfficiency, lookback)
efficiencyRatio = avgVolumeEfficiency > 0 and not na(avgVolumeEfficiency) ?
                  volumeEfficiency / avgVolumeEfficiency : 1.0
```

**Critical Pattern:**
```pinescript
// ALWAYS protect against zero/tiny ranges:
safeRange = math.max(priceRange, close * 0.001)  // 0.1% minimum
// OR
safeRange = math.max(priceRange, close * 0.0001) // 0.01% minimum
```

---

### 6. Multi-Timeframe Helper Functions 🟢 LOW

**Duplicated Across:**
- `institutional-algo2-mtf-convergence.pine` (lines 47-78)
- `institutional-algo4-ensemble.pine` (lines 152-169)
- **WILL DUPLICATE IN**: S/R Multi-Timeframe Confluence

**Components:**
```pinescript
getHigherTimeframe(baseTF) =>
    tfInMinutes = timeframe.in_seconds(baseTF) / 60
    string result = ""
    if tfInMinutes <= 1
        result := "5"
    else if tfInMinutes <= 5
        result := "15"
    // ... etc
    result

getSecondHigherTimeframe(baseTF) =>
    firstHTF = getHigherTimeframe(baseTF)
    getHigherTimeframe(firstHTF)
```

**Standard Timeframe Ladder:**
```
1min → 5min → 15min → 60min → 240min → Daily → Weekly → Monthly
```

---

## Future S/R Algorithms - Expected Duplications

### S/R Algorithm 1: Volume Profile Detection
**Will Duplicate:**
- ✅ Regime detection (ATR-based)
- ✅ Volume calculations
- ✅ Volume efficiency calculations
- ✅ NA protection patterns

**New Unique Code:**
- Volume-at-price distribution (bins)
- POC/VAH/VAL calculations
- HVN/LVN detection

---

### S/R Algorithm 2: Statistical Peak/Trough Detection
**Will Duplicate:**
- ✅ Regime detection (ATR-based)
- ✅ ADX trend detection
- ✅ EMA trend detection
- ✅ Volume calculations

**New Unique Code:**
- Swing point detection (can use `ta.pivothigh()`/`ta.pivotlow()`)
- Clustering algorithm (DBSCAN-like)
- Temporal decay scoring
- Confluence detection (Fibonacci, MAs, psychological levels)

---

### S/R Algorithm 3: Multi-Timeframe Confluence
**Will Duplicate:**
- ✅ Multi-timeframe helper functions
- ✅ Regime detection
- ✅ Volume calculations

**New Unique Code:**
- Cross-timeframe level merging
- Timeframe-weighted scoring
- Confluence detection across timeframes

---

## Code Standards & Patterns

### NA Protection (CRITICAL)
```pinescript
// ALWAYS check for NA and zero before division:
result = denominator > 0 and not na(denominator) and not na(numerator) ?
         numerator / denominator :
         defaultValue
```

### Safe Division Pattern
```pinescript
// For price ranges, use minimum threshold:
safeRange = math.max(priceRange, close * 0.001)

// For ratios, check both NA and zero:
ratio = avg > 0 and not na(avg) ? current / avg : 1.0
```

### Array Bounds Checking
```pinescript
// ALWAYS check array size before accessing:
if array.size(myArray) > 0
    value = array.get(myArray, 0)
```

---

## Refactoring Strategy (Future)

### Option 1: PineScript Library (When/If Available)
- TradingView may add library support in future versions
- Would allow shared modules across indicators
- **Action**: Monitor PineScript updates for library features

### Option 2: External Code Generation
- Create Python/Node.js script that generates PineScript
- Single source of truth for shared logic
- Generate multiple indicator files
- **Complexity**: High, but maintainable

### Option 3: Manual Synchronization (Current Approach)
- Document all duplications here
- Use search/replace carefully
- Test all indicators after changes
- **Complexity**: Low, but error-prone

---

## Maintenance Checklist

When modifying shared logic:

- [ ] Identify which indicators use this logic (check sections above)
- [ ] Update code in ALL affected files
- [ ] Verify parameter values match across files (unless intentionally different)
- [ ] Test each indicator individually
- [ ] Update this document if logic changes
- [ ] Add git commit message noting cross-indicator change

---

## Change Log

### 2025-01-XX - Initial Tech Debt Documentation
- Documented existing duplications across 4 institutional algorithms
- Identified 6 major areas of code duplication
- Established code standards and NA protection patterns
- Created maintenance checklist

### [Future entries as we build S/R algorithms]

---

## Notes

- **PineScript Version**: Currently using v5 (some algos v6)
- **Max Security Calls**: 40 per indicator (relevant for MTF algorithms)
- **Max Execution Time**: 500ms per bar (impacts lookback periods)
- **Max Array Size**: No hard limit but performance degrades with very large arrays

---

## Quick Reference: Common Parameters

| Parameter | Standard Value | Variance Allowed | Notes |
|-----------|---------------|------------------|-------|
| ATR Length | 14 | 7-50 | Standard is 14 |
| ATR Lookback | 50-100 | 20-200 | 50 for short-term, 100 for long-term |
| High Vol Threshold | 1.3 | 1.2-1.5 | 30% above average ATR |
| Low Vol Threshold | 0.7 | 0.5-0.8 | 30% below average ATR |
| ADX Trending | 25 | 20-30 | 25 is standard |
| ADX Ranging | 20 | 15-25 | Below this = ranging |
| Volume Lookback | 20 | 10-50 | Standard SMA period |
| Volume Threshold | 1.5x | 1.3-2.0x | 1.5x = standard, 2.0x = high |
| Efficiency Threshold | 1.3x | 1.0-3.0x | Volume per price range ratio |
| EMA Short | 20 | 15-25 | Fast trend |
| EMA Mid | 50 | 40-60 | Medium trend |
| EMA Long | 200 | 150-250 | Long trend |

---

## Future Improvements

1. **Automated Testing**: Create test suite for regime detection logic
2. **Parameter Optimization**: Backtest standard parameters across multiple assets
3. **Documentation**: Add inline comments referencing this document
4. **Version Control**: Tag releases when major shared logic changes
5. **Deprecation Policy**: Document when removing old patterns
