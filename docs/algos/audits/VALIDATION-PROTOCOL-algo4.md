# Empirical Validation Protocol: Algorithm 4 Order Book Reconstruction
**Version:** 2.1
**Date:** 2025-01-17
**Status:** Ready for execution
**Auditor Grade:** A (93/100) - Approved for validation phase

---

## Executive Summary

Following the secondary audit approval, this protocol outlines **empirical testing** to validate the claimed 70-78% accuracy for Algorithm 4 (Order Book Reconstruction) levels with 3+ rejections.

**Objective:** Measure actual hold rates across multiple asset classes and compare to theoretical estimates.

**Timeline:** 2-3 weeks
- Week 1: Futures validation (ES, NQ, RTY, GC, CL)
- Week 2: Cross-asset validation (stocks, crypto, forex)
- Week 3: Parameter sensitivity and ensemble integration

---

## PHASE 1: FUTURES VALIDATION (HIGH PRIORITY)

### Why Futures First?
1. **Highest data quality** - CME/CBOT tick-level accuracy
2. **Institutional participation** - 60-70% professional traders
3. **24-hour markets** - Eliminates gap-bar edge cases
4. **High liquidity** - True price discovery (not spoofing)
5. **Standardized contracts** - Consistent behavior across time

### Assets to Test

| Symbol | Name | Why Selected | Expected ATR | Timeframe |
|--------|------|--------------|--------------|-----------|
| **ES** | E-mini S&P 500 | Most liquid equity future, high institutional | 1.5-2.5% | 15m, 1H, 4H |
| **NQ** | E-mini Nasdaq 100 | Tech-heavy, high vol, good for algo testing | 2.0-3.5% | 15m, 1H, 4H |
| **RTY** | E-mini Russell 2000 | Small-cap proxy, different dynamics | 2.5-4.0% | 1H, 4H |
| **GC** | Gold Futures | Safe-haven, institutional hedging | 1.0-2.0% | 1H, 4H, D |
| **CL** | Crude Oil | Commodity, geopolitical sensitivity | 3.0-5.0% | 1H, 4H |

### Test Protocol (Per Asset)

#### Step 1: Historical Level Collection
```
1. Load chart with 6 months historical data
2. Apply Algorithm 4 with standard settings:
   - Lookback: 200 bars
   - Min rejections: 3 (quality filter)
   - Min strength: 60
   - ATR clustering: ENABLED
   - Time decay: ENABLED (50-bar half-life)
3. Record all levels that appear with strength ≥60
4. Target: 50-100 levels per asset
```

#### Step 2: Forward Performance Tracking
For each level identified, track:

**A) Hold Event Definition:**
```
Level holds IF:
- Price tests within 1% of level (touches zone)
- AND does NOT close >2% beyond level within next 10 bars
- AND bounces back within rejection zone

Example:
- Resistance at $5,000
- Price reaches $4,950 (within 1%)
- Does NOT close above $5,100 (2% breach)
- Returns below $5,020 within 10 bars
→ HOLD ✅
```

**B) Break Event Definition:**
```
Level breaks IF:
- Price closes >2% beyond level
- AND sustains for 3+ consecutive bars
- AND does not return to level within 20 bars

Example:
- Resistance at $5,000
- Price closes at $5,110, $5,125, $5,140 (3 bars >2%)
- Does not return to $4,980-$5,020 within 20 bars
→ CLEAN BREAK ❌
```

**C) Data Collection:**
For each level, record:
```csv
Symbol,Timeframe,Level,Strength,Rejections,FirstTest,Outcome,Duration,BreakSize
ES,1H,4525.50,72,4,2024-12-15T14:00,HOLD,18,N/A
ES,1H,4580.25,68,3,2024-12-16T09:00,BREAK,8,2.3%
NQ,15m,16250.00,81,5,2024-12-15T10:30,HOLD,42,N/A
```

#### Step 3: Statistical Analysis
Calculate for each asset + timeframe:

```
holdRate = holds / (holds + breaks)
avgHoldDuration = mean(durations for holds)
avgBreakSize = mean(break percentages)
strengthCorrelation = correlation(strength, holdRate)
```

**Target Benchmarks:**
- Overall hold rate: **70-78%** (claim)
- Strength ≥70: **75-82%**
- Strength 60-70: **68-75%**
- Strength <60: Not tested (filtered out)

**Pass Criteria:**
- ✅ PASS: Overall hold rate 68-78% (within ±2% of claim)
- ⚠️ CONDITIONAL: 65-68% or 78-82% (investigate parameters)
- ❌ FAIL: <65% or >85% (fundamental issue)

---

## PHASE 2: CROSS-ASSET VALIDATION

### Asset Selection (Diversity Test)

**Stocks (Low-Med Volatility):**
- AAPL (mega-cap tech, ATR ~2%)
- MSFT (mega-cap tech, ATR ~2%)
- JPM (financials, ATR ~2.5%)

**Crypto (High Volatility):**
- BTCUSD (flagship, ATR ~5%)
- ETHUSD (alt leader, ATR ~6%)

**Forex (24-hour, macro-driven):**
- EURUSD (most liquid pair, ATR ~0.8%)
- GBPUSD (Brexit/BoE sensitivity, ATR ~1.0%)

### Expected Variance by Asset Class

| Asset Class | Expected Hold Rate | Reasoning |
|-------------|-------------------|-----------|
| **Futures** | 72-78% | Highest institutional participation |
| **Stocks** | 70-76% | Good institutional, some retail noise |
| **Crypto** | 65-72% | High volatility, mixed participants |
| **Forex** | 68-75% | Macro-driven, but high algo trading |

**Why differences?**
- **Crypto:** More retail, emotional trading, gaps on weekends
- **Forex:** Fundamental shifts (central banks) can invalidate levels quickly
- **Stocks:** Earnings gaps, news events create breakouts

---

## PHASE 3: PARAMETER SENSITIVITY ANALYSIS

### Variables to Test

#### A) Time Decay Half-Life
**Current:** 50 bars (exponential decay e^(-bars/50))

**Test range:**
```
decayHalfLife = [20, 35, 50, 65, 80]

For each asset:
- Run validation with each half-life value
- Measure hold rate variance
- Calculate optimal value per timeframe
```

**Hypothesis:**
- **Intraday (15m-1H):** Optimal = 30-35 bars (fast turnover)
- **Swing (4H-D):** Optimal = 50-60 bars (position holding)
- **Position (D-W):** Optimal = 70-80 bars (longer-term defense)

**Decision Criteria:**
- If variance >5%: **Parameter is critical** → make user-adjustable
- If variance <2%: Current default (50) is robust

---

#### B) Distance Scale (Gaussian Decay)
**Current (v2.1):** ATR-adaptive `distanceScale = max(0.05, min(0.30, atrPercent × 3.0))`

**Test multipliers:**
```
scaleMultipliers = [2.0, 2.5, 3.0, 3.5, 4.0]
// Current: 3.0 means ATR×3 = distance scale

For each asset:
- Vary multiplier (keeping ATR-adaptive logic)
- Measure strength distribution changes
- Calculate correlation with hold rate
```

**Hypothesis:**
- **Low vol assets (stocks):** Multiplier = 2.0-2.5 (tighter relevance)
- **High vol assets (crypto):** Multiplier = 3.5-4.0 (wider relevance)

**Decision Criteria:**
- If correlation >0.7: **Scale is working correctly**
- If correlation <0.5: Distance decay not predictive

---

#### C) Component Weights (Strength Formula)
**Current:**
```pinescript
volumeScore × 0.35 + rejectionScore × 0.40 + wickScore × 0.25
```

**Test permutations:**
```
weights = [
    [0.30, 0.40, 0.30],  // More wick emphasis
    [0.35, 0.40, 0.25],  // Current (balanced)
    [0.40, 0.35, 0.25],  // More volume emphasis
    [0.30, 0.45, 0.25],  // More rejection emphasis
]

For each weight set:
- Recalculate all level strengths
- Measure hold rate by strength decile
- Calculate AUC (area under ROC curve)
```

**Optimal weight selection:**
- Maximize AUC (strength → hold rate predictiveness)
- Ensure monotonic relationship (higher strength = higher hold rate)

**Decision Criteria:**
- If AUC difference <0.05: **Current weights are near-optimal**
- If AUC improves >0.10: **Update default weights**

---

## PHASE 4: CROSS-ALGORITHM VALIDATION

### Overlap Analysis with Algorithm 1 (Volume Profile)

**Objective:** Measure how often Algo 4 levels align with Algo 1 POC/VAH/VAL

**Protocol:**
```
1. For each Algo 4 level (strength ≥60, 3+ rejections):
   - Measure distance to nearest POC/VAH/VAL from Algo 1
   - If <1% away: Mark as "CONFIRMED"
   - If >3% away: Mark as "UNIQUE"

2. Calculate:
   confirmationRate = confirmed / total
   uniqueRate = unique / total

3. Compare performance:
   confirmedHoldRate = holds(confirmed) / tests(confirmed)
   uniqueHoldRate = holds(unique) / tests(unique)
```

**Target Benchmarks:**
- **Confirmation rate:** 55-70% (sweet spot)
  - <50%: Algo 4 detecting noise (too different)
  - >75%: Algo 4 redundant (just use Algo 1)
- **Confirmed hold rate:** Should be 5-10% higher than unique
- **Unique hold rate:** Should still be ≥65% (Algo 4 adding value)

**Decision Criteria:**
- ✅ PASS: 60-70% confirmation, unique hold rate ≥65%
- ⚠️ INVESTIGATE: <55% or >75% confirmation
- ❌ FAIL: Unique hold rate <60% (no incremental value)

---

## PHASE 5: REGIME-SPECIFIC ANALYSIS

### Market Regime Classification

**Using ADX (Average Directional Index):**
```pinescript
adx = ta.adx(14)

regime = adx < 20 ? "RANGING" :
         adx < 40 ? "TRENDING" :
         "STRONG_TREND"
```

### Expected Performance by Regime

| Regime | ADX Range | Expected Hold Rate | Reasoning |
|--------|-----------|-------------------|-----------|
| **Ranging** | <20 | **72-78%** | Best case - levels define boundaries |
| **Trending** | 20-40 | **65-72%** | Moderate - levels slow momentum |
| **Strong Trend** | >40 | **55-65%** | Worst case - breakouts likely |

**Protocol:**
```
1. For each level test event:
   - Calculate ADX at time of test
   - Classify regime
   - Track outcome

2. Separate analysis by regime:
   rangingHoldRate = holds(ranging) / tests(ranging)
   trendingHoldRate = holds(trending) / tests(trending)
   strongTrendHoldRate = holds(strongTrend) / tests(strongTrend)
```

**Decision Criteria:**
- ✅ If ranging <70%: **Core algorithm problem** (should work best here)
- ✅ If trending <60%: **Need stronger regime filtering**
- ✅ If strong trend <55%: **Expected** (use as invalidation signal)

**Recommendation Enhancement:**
```pinescript
// Add regime warning to strength display
if adx > 40 and finalStrength < 70:
    // Reduce displayed strength or add warning
    displayStrength = finalStrength × 0.8
    tooltip += "\n⚠️ Strong trend - level vulnerable"
```

---

## VALIDATION DATA COLLECTION TEMPLATE

### CSV Schema
```csv
# Level Identification
Symbol,Timeframe,DetectionDate,Price,Strength,Rejections,LevelType

# Test Events
Symbol,Level,TestDate,TestPrice,PriceDistancePct,ADX,Regime,Volume,Outcome,HoldDuration,BreakSize

# Cross-Algorithm
Symbol,Level,Algo1Distance,Algo1Type,Algo3Distance,Algo3Confirmed,EnsembleAgreement

# Performance Metrics (aggregated)
Symbol,Timeframe,TotalLevels,TotalTests,Holds,Breaks,HoldRate,AvgStrength,ConfirmationRate
```

### Example Rows
```csv
# Identification
ES,1H,2024-12-15,4525.50,72,4,Resistance

# Test Event
ES,4525.50,2024-12-16T14:00,4518.25,0.16%,28,TRENDING,850K,HOLD,18,N/A

# Cross-Algorithm
ES,4525.50,0.8%,POC,1.2%,YES,3/4

# Metrics
ES,1H,52,87,63,24,72.4%,68.5,64.2%
```

---

## SUCCESS CRITERIA SUMMARY

### ✅ VALIDATION PASSES IF:

**Overall Performance:**
- Hold rate for 3+ rejections: **68-78%** (within ±2% margin)
- Strength correlation with hold rate: **r >0.60** (predictive)
- False positive rate (clean breaks): **<25%**

**Cross-Asset Consistency:**
- All asset classes within **±5%** of predicted range
- Futures: 72-78%, Stocks: 70-76%, Crypto: 65-72%

**Cross-Algorithm Validation:**
- Algo 1 confirmation rate: **55-70%**
- Unique level hold rate: **≥65%**
- Confirmed level hold rate: **≥70%**

**Regime Performance:**
- Ranging: **≥70%**, Trending: **≥65%**, Strong Trend: **≥55%**

**Parameter Stability:**
- Decay half-life variance: **<5%**
- Distance scale correlation: **>0.7**
- Weight optimization AUC: **<0.10 improvement** (current near-optimal)

---

### ⚠️ CONDITIONAL APPROVAL IF:

- Hold rate: **65-68%** or **78-82%** (investigate edge cases)
- One asset class underperforms by >10% (asset-specific issue)
- Parameter variance >5% (needs adaptive tuning)

**Actions:**
- Review edge case handling
- Consider asset-specific parameter sets
- Test alternative weight combinations

---

### ❌ VALIDATION FAILS IF:

- Overall hold rate: **<65%** (fundamental flaw)
- Strength correlation: **<0.50** (not predictive)
- Unique level hold rate: **<60%** (no incremental value)
- Ranging regime: **<70%** (should work best here)

**Actions:**
- Revisit volume weighting formula (test quadratic)
- Reexamine stop cascade detection logic
- Consider reverting to simpler strength formula
- Increase minimum rejections to 4-5

---

## TIMELINE & MILESTONES

### Week 1: Futures Deep Dive
- **Day 1-2:** ES + NQ validation (most liquid)
- **Day 3:** RTY validation (different dynamics)
- **Day 4-5:** GC + CL validation (commodities)
- **Deliverable:** Futures validation report with 250-500 test events

### Week 2: Cross-Asset Expansion
- **Day 1-2:** Stocks (AAPL, MSFT, JPM)
- **Day 3:** Crypto (BTC, ETH)
- **Day 4:** Forex (EURUSD, GBPUSD)
- **Day 5:** Cross-algorithm overlap analysis
- **Deliverable:** Multi-asset comparison report

### Week 3: Optimization & Integration
- **Day 1-2:** Parameter sensitivity testing
- **Day 3:** Regime-specific analysis
- **Day 4:** Ensemble weight recommendations
- **Day 5:** Final validation report + production go/no-go

---

## REPORTING FORMAT

### Daily Progress Update
```markdown
## ES Futures - 1H Timeframe Validation

**Data Range:** 2024-06-01 to 2024-12-15 (6 months)
**Levels Identified:** 63
**Test Events:** 94
**Results:**
- Holds: 68 (72.3%)
- Breaks: 26 (27.7%)
- Avg Hold Duration: 14.2 bars
- Strength Correlation: 0.68

**Regime Breakdown:**
- Ranging (ADX <20): 78.5% hold rate (33 tests)
- Trending (ADX 20-40): 71.2% hold rate (48 tests)
- Strong Trend (ADX >40): 53.8% hold rate (13 tests)

**Status:** ✅ PASS (within target range)
```

### Final Validation Report Structure
```markdown
1. Executive Summary
   - Overall hold rate: X%
   - Pass/Fail/Conditional
   - Key findings

2. Asset-by-Asset Results
   - Table with all metrics
   - Outliers explained

3. Parameter Sensitivity
   - Optimal ranges identified
   - Recommended adjustments

4. Cross-Algorithm Analysis
   - Confirmation rates
   - Unique value quantified

5. Production Recommendations
   - Ensemble weights
   - Usage guidelines
   - Known limitations

6. Appendix: Raw Data
   - CSV files attached
```

---

## NEXT STEPS AFTER VALIDATION

### If PASS (≥68% hold rate):
1. ✅ Update ensemble weights (20-25% for Algo 4)
2. ✅ Document validated accuracy in indicator header
3. ✅ Create user guide with realistic expectations
4. ✅ Begin production deployment (paper trading first)

### If CONDITIONAL (65-68% or 78-82%):
1. ⚠️ Investigate edge cases causing variance
2. ⚠️ Test alternative parameter sets
3. ⚠️ Re-run validation on problematic assets
4. ⚠️ Conservative ensemble weight (15-20%)

### If FAIL (<65% hold rate):
1. ❌ Revisit theoretical assumptions
2. ❌ Test alternative formulas (quadratic volume, etc.)
3. ❌ Consider limiting to specific asset classes
4. ❌ Hold production release until fixes implemented

---

## RESOURCE REQUIREMENTS

**Data:**
- TradingView Premium (for historical data export) OR
- Manual chart review (100+ hours) OR
- Automated screenshot parsing

**Time:**
- Week 1: 20-25 hours (futures testing)
- Week 2: 15-20 hours (cross-asset)
- Week 3: 10-15 hours (optimization)
- **Total: 45-60 hours**

**Personnel:**
- 1 analyst for data collection
- 1 developer for parameter testing
- 1 reviewer for quality control

---

**Protocol Status:** READY FOR EXECUTION
**Approved By:** Development Team
**Date:** 2025-01-17
**Next Action:** Begin Phase 1 (ES/NQ validation)
