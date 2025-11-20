# MKN Super - Build Progress Tracker

## Project Overview

**Indicator Name**: MKN Super (formerly MKN Exit Framework)
**Purpose**: Complete entry + exit trading decision support system
**Target LOC**: 350-400 lines
**Estimated Time**: 21.5 hours
**Status**: 🚧 IN PROGRESS

---

## Design Decisions (User-Confirmed)

✅ **Entry Threshold**: Moderate (≥65) - Creates 15-point neutral zone
✅ **Exit Threshold**: Conservative (<50) - Early capital preservation
✅ **Ranging Filter**: Show signals with "⚠️ RANGING" warning labels
✅ **Multiple Entries**: Display "BUY+++" for confluence (multiple confirmations)
✅ **Indicator Name**: "MKN Super" (not "MKN Exit Framework")

---

## Specification Documents

**Primary Spec**: `/docs/algos/trends/mkn-super.md` (exit framework)
**Addendum**: `/docs/algos/trends/mkn-super-addendum.md` (entry additions)

---

## Phase 1: FOUNDATION (3 hours)

### Phase 1.1: Project Setup ✅ COMPLETE
**Time**: 30 minutes
**Status**: ✅ Done (2025-01-20)

**Deliverables**:
- ✅ Created `mkn-super.pine` with Pine Script v5 header
- ✅ Created `mkn-super-build.md` progress tracker
- ✅ Defined all input parameters (6 groups, 20 parameters total)

**Input Parameters Implemented**:
- Group 1: Trend Detection (2 params)
  - `atrLength` = 10 (range: 5-20)
  - `multiplier` = auto (2.5/3.0/3.5 by asset type)
- Group 2: S/R Levels (4 params)
  - `srLevel1/2/3` = input.source(close)
  - `tolerance` = 2.0% (range: 0.5-5.0%)
- Group 3: Momentum Health (8 params)
  - `volPeriod` = 20 (range: 10-50)
  - `rsiLength` = 14 (range: 7-21)
  - `macdFast/Slow/Signal` = 12/26/9
  - `volWeight/rsiWeight/macdWeight` = 0.4/0.3/0.3
- Group 4: Decision Thresholds (3 params)
  - `exitMomentumThreshold` = 50
  - `entryMomentumThreshold` = 65
  - `adxThreshold` = 20
- Group 5: Display (5 params)
  - `showSRLevels/Dashboard/Signals/RangingWarnings` = true
  - `enableAlerts` = false
- Group 6: Position State (1 param)
  - `inPosition` = false

**Verification**:
- ✅ Indicator loads in TradingView without errors
- ✅ All 20 parameters visible in settings panel
- ✅ Tooltips provide clear guidance
- ✅ Asset-specific multiplier auto-detects correctly

**Issues**: None

---

### Phase 1.2: Trend Detection (Supertrend) ✅ COMPLETE
**Time**: 1 hour
**Status**: ✅ Done (2025-01-20)

**Tasks**:
- ✅ Implement Supertrend using `ta.supertrend(multiplier, atrLength)`
- ✅ Calculate `isBullish` and `isBearish` states
- ✅ Plot trend line (3px width, green/red color)
- ✅ Add background fill (95% transparency)

**Implementation**:
```pinescript
[supertrend, direction] = ta.supertrend(multiplier, atrLength)
isBullish = direction < 0  // -1 = bullish
isBearish = direction > 0  // 1 = bearish
plot(supertrend, color=trendColor, linewidth=3)
bgcolor(bgColor)  // 95% transparent green/red
```

**Verification Criteria**:
- ✅ Green line (#00E676) displays in uptrend, red line (#FF5252) in downtrend
- ✅ Asset multiplier auto-adjusts (crypto=3.5, stocks=2.5, futures=3.0)
- ✅ Background color changes with trend (95% transparency)
- ✅ No repainting (ta.supertrend uses confirmed bars by default)

**Spec Reference**: mkn-super.md lines 83-181
**Code Lines**: 179-230 in mkn-super.pine

---

### Phase 1.3: S/R Level Integration
**Time**: 1.5 hours
**Status**: ⏳ PENDING

**Tasks**:
- [ ] Implement proximity detection formulas
- [ ] Calculate `atLevel1`, `atLevel2`, `atLevel3`, `atAnyLevel`
- [ ] Determine level type (RESISTANCE/SUPPORT/BETWEEN)
- [ ] Plot 3 S/R levels (white/gray, different styles)
- [ ] Add proximity background (yellow when at level)
- [ ] Add S/R labels at right edge of chart

**Verification Criteria**:
- [ ] Yellow background triggers when price within 2% of level
- [ ] Level 1: White solid line (2px)
- [ ] Level 2: Gray dashed line (1.5px)
- [ ] Level 3: Gray dotted line (1px)
- [ ] Labels show "S/R #1", "S/R #2", "S/R #3"
- [ ] Manual connection to sr-algo5-ensemble works

**Spec Reference**: mkn-super.md lines 184-274

---

## Phase 2: MOMENTUM COMPOSITE (2.5 hours)

### Phase 2.1: Volume Health
**Status**: ⏳ PENDING

**Implementation**:
```pinescript
volRatio = volume / ta.sma(volume, volPeriod)

volScore = volRatio >= 2.0 ? 100 :  // Extreme (Tier 3)
           volRatio >= 1.5 ? 85  :  // High (Tier 2)
           volRatio >= 1.3 ? 70  :  // Elevated (Tier 1)
           volRatio >= 1.0 ? 50  :  // Normal (Tier 0)
           25                        // Below average
```

**Test Cases**:
- [ ] volRatio = 2.5 → volScore = 100 ✓
- [ ] volRatio = 1.6 → volScore = 85 ✓
- [ ] volRatio = 1.4 → volScore = 70 ✓
- [ ] volRatio = 1.1 → volScore = 50 ✓
- [ ] volRatio = 0.8 → volScore = 25 ✓

**Spec Reference**: mkn-super.md lines 283-300

---

### Phase 2.2: RSI Health
**Status**: ⏳ PENDING

**Implementation**: Trend-adjusted scoring
- Uptrend: >60=100, >50=75, >40=50, else 25
- Downtrend: <40=100, <50=75, <60=50, else 25

**Test Cases**:
- [ ] Uptrend + RSI=65 → rsiScore = 100 ✓
- [ ] Uptrend + RSI=55 → rsiScore = 75 ✓
- [ ] Downtrend + RSI=35 → rsiScore = 100 ✓
- [ ] Downtrend + RSI=65 → rsiScore = 25 ✓

**Spec Reference**: mkn-super.md lines 301-323

---

### Phase 2.3: MACD Health
**Status**: ⏳ PENDING

**Implementation**: Acceleration check
- Uptrend: bullish+accel=100, bullish=75, else 0
- Downtrend: bearish+accel=100, bearish=75, else 0

**Test Cases**:
- [ ] Uptrend + MACD bullish + accelerating → macdScore = 100 ✓
- [ ] Uptrend + MACD bullish (not accel) → macdScore = 75 ✓
- [ ] Uptrend + MACD bearish → macdScore = 0 ✓

**Spec Reference**: mkn-super.md lines 325-349

---

### Phase 2.4: Momentum Composite
**Status**: ⏳ PENDING

**Formula**:
```
momentumHealth = (volScore × 0.4) + (rsiScore × 0.3) + (macdScore × 0.3)
```

**Classification**:
- STRONG: ≥75
- HEALTHY: ≥50
- WEAKENING: ≥25
- EXHAUSTED: <25

**Test Case** (from spec):
- volScore=85, rsiScore=100, macdScore=100
- Expected: (85×0.4) + (100×0.3) + (100×0.3) = 34 + 30 + 30 = **94** ✓

**Spec Reference**: mkn-super.md lines 351-360

---

## Phase 3: EXIT DECISION ENGINE (2 hours)

### Exit Rule 1: Trend Break (CRITICAL)
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
trendBroken = ta.crossunder(close, supertrend) and barstate.isconfirmed
```

**Output**:
- Red X marker (`shape.xcross`, `size.large`) above bar
- Label: "EXIT"
- Table: "EXIT NOW (Trend Broken)" [RED]
- Alert: "CRITICAL: Trend line broken"

**Expected Accuracy**: 70-75%

**Spec Reference**: mkn-super.md lines 442-463

---

### Exit Rule 2: Take Profit at Resistance
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
atResistanceLevel = atLevel1 and (close > srLevel1 * 0.98)
weakMomentum = momentumHealth < exitMomentumThreshold  // <50
takeProfitSignal = atResistanceLevel and weakMomentum and isBullish
```

**Output**:
- Yellow diamond marker (`shape.diamond`, `size.normal`) above bar
- Label: "PROFIT"
- Table: "TAKE PROFIT (Resistance + Weak)" [YELLOW]

**Expected Accuracy**: 60-70%

**Spec Reference**: mkn-super.md lines 466-491

---

### Exit Rule 3: Momentum Exhaustion
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
momentumExhausted = momentumHealth < 25
```

**Output**:
- Orange circle marker (`shape.circle`, `size.small`) above bar
- Label: "WEAK"
- Table: "EXIT (Momentum Dead)" [ORANGE]

**Expected Accuracy**: 55-65%

**Spec Reference**: mkn-super.md lines 494-516

---

### Exit Rule 4: Volume Divergence
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
priceNewHigh = close > ta.highest(close[1], 20)
volumeDeclining = volume < ta.sma(volume, 5)
volumeDivergence = priceNewHigh and volumeDeclining and isBullish
```

**Output**:
- Yellow label "⚠️ VOLUME DIVERGENCE"
- Table: "CAUTION (Volume Diverging)" [YELLOW]

**Expected Accuracy**: 50-60%

**Spec Reference**: mkn-super.md lines 519-544

---

## Phase 4: ENTRY DECISION ENGINE (2 hours)

### Entry Rule 1: Trend Confirmation
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
trendEstablished = ta.crossover(close, supertrend) and barstate.isconfirmed
```

**Output**:
- Green triangle up (`shape.triangleup`, `size.normal`) below bar
- Label: "ENTRY"
- Table: "ENTRY (Trend Confirmed)" [GREEN]

**Expected Accuracy**: 70-75%

**Spec Reference**: mkn-super-addendum.md lines 6-28

---

### Entry Rule 2: Support + Strong Momentum (BEST)
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
atSupportLevel = atLevel1 and (close < srLevel1 * 1.02)
strongMomentum = momentumHealth >= entryMomentumThreshold  // ≥65
buySignal = atSupportLevel and strongMomentum and isBullish
```

**Output**:
- Bright green diamond (`shape.diamond`, `size.normal`) below bar
- Label: "BUY+"
- Table: "BUY (Support + Strong)" [BRIGHT GREEN]

**Expected Accuracy**: 65-75%

**Spec Reference**: mkn-super-addendum.md lines 31-57

---

### Entry Rule 3: Momentum Building
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
momentumBuilding = momentumHealth >= entryMomentumThreshold and
                   momentumHealth > momentumHealth[1] and
                   not atAnyLevel
earlyEntry = momentumBuilding and isBullish
```

**Output**:
- Lime circle (`shape.circle`, `size.small`) below bar
- Label: "MOM+"
- Table: "WATCH (Momentum Building)" [LIME]

**Expected Accuracy**: 60-70%

**Spec Reference**: mkn-super-addendum.md lines 60-87

---

### Entry Rule 4: Volume Confirmation (MULTIPLIER)
**Status**: ⏳ PENDING

**Trigger**:
```pinescript
volumeConfirmation = volRatio >= 1.5 and close > close[1] and isBullish
```

**Output**:
- Green "✓ VOL" label below bar (when combined with other entry)
- Enhances other signals to create "BUY++" or "BUY+++"

**Spec Reference**: mkn-super-addendum.md lines 90-113

---

### Confluence Tracking (BUY+++)
**Status**: ⏳ PENDING

**Logic**:
- Count how many entry rules triggered on same bar
- 1 signal: "BUY" or "ENTRY"
- 2 signals: "BUY++"
- 3+ signals: "BUY+++"

**Spec Reference**: mkn-super-addendum.md lines 115-134

---

## Phase 5: ADX RANGING DETECTION (1 hour)

**Status**: ⏳ PENDING

**Implementation**:
- Calculate ADX(14) using standard formula
- Define ranging: ADX < adxThreshold (default: 20)
- Add "⚠️ RANGING: Less reliable" labels to signals

**Verification**:
- [ ] ADX calculation matches TradingView built-in
- [ ] Warning appears when ADX <20
- [ ] Warning disappears in trending conditions (ADX >25)

**Spec Reference**: mkn-super-addendum.md lines 136-165, 269-288

---

## Phase 6: VISUAL OUTPUTS (3 hours)

### Chart Markers - Exit Signals
**Status**: ⏳ PENDING

- [ ] Red X (trend break)
- [ ] Yellow diamond (take profit)
- [ ] Orange circle (weak momentum)
- [ ] Yellow label (volume divergence)

### Chart Markers - Entry Signals
**Status**: ⏳ PENDING

- [ ] Green triangle (trend confirmation)
- [ ] Bright green diamond (support + strong)
- [ ] Lime circle (momentum building)
- [ ] Green "✓ VOL" label (volume confirmation)

### Info Table Dashboard (10 rows)
**Status**: ⏳ PENDING

**Structure**:
```
Row 0: Header "MKN SUPER" (merged cells)
Row 1: Trend status (UPTREND ↑ / DOWNTREND ↓)
Row 2: Position relative to trend (% above/below line)
Row 3: Nearest S/R level (type + distance)
Row 4: Momentum composite (score + state)
Row 5: ├─ Volume (ratio + score)
Row 6: ├─ RSI (value + score)
Row 7: └─ MACD (state + score)
Row 8: Exit Decision
Row 9: Entry Signal
```

**Spec Reference**: mkn-super.md lines 624-725, mkn-super-addendum.md lines 167-203

---

## Phase 7: ALERTS (1 hour)

**Status**: ⏳ PENDING

### Exit Alerts (4)
- [ ] Alert 1: "CRITICAL: Trend Broken"
- [ ] Alert 2: "Profit Opportunity"
- [ ] Alert 3: "Momentum Warning"
- [ ] Alert 4: "Volume Divergence"

### Entry Alerts (3)
- [ ] Alert 5: "BUY: Trend Confirmed"
- [ ] Alert 6: "BUY: Support + Strong"
- [ ] Alert 7: "Entry Opportunity: Momentum Building"

**Spec Reference**: mkn-super.md lines 729-755, mkn-super-addendum.md lines 207-224

---

## Phase 8: ADVANCED FEATURES (2 hours)

**Status**: ⏳ PENDING

### Position State Toggle
- [ ] Implement `inPosition` checkbox logic
- [ ] Conditional emphasis (exits if true, entries if false)

### Neutral Zone (15-point buffer)
- [ ] Exit signals: momentum <50
- [ ] Entry signals: momentum ≥65
- [ ] Display "WAIT (Neutral Zone)" when 50-64

### Edge Cases
- [ ] Gap opening detection (delay confirmation)
- [ ] Low volume warning (volume <0.5× average)
- [ ] Prevent simultaneous entry+exit on same bar

**Spec Reference**: mkn-super-addendum.md lines 228-268, 269-298

---

## Phase 9: TESTING & VALIDATION (3 hours)

**Status**: ⏳ PENDING

### Unit Tests - Components
- [ ] Trend detection (SPY uptrend/downtrend/ranging)
- [ ] S/R proximity (at resistance, at support, between)
- [ ] Momentum composite (math verification)

### Integration Tests - Exit Signals
- [ ] Trend break exit (signal + marker + alert)
- [ ] Take profit at resistance (confluence required)
- [ ] Momentum exhaustion warning
- [ ] Volume divergence detection

### Integration Tests - Entry Signals
- [ ] Trend confirmation entry
- [ ] Support + strong entry ("BUY++")
- [ ] Momentum building entry
- [ ] Confluence "BUY+++" display
- [ ] Ranging warning label (ADX <20)

---

## Phase 10: DOCUMENTATION & POLISH (2 hours)

**Status**: ⏳ PENDING

### Code Comments
- [ ] File header with complete description
- [ ] Section separators (ASCII art)
- [ ] Formula documentation
- [ ] Design decision notes

### User Documentation
- [ ] Create `docs/algos/trends/mkn-super-guide.md`
- [ ] Quick start (5 steps)
- [ ] Parameter tuning
- [ ] Interpretation guide
- [ ] Troubleshooting

### Changelog
- [ ] Create `CHANGELOG-mkn-super.md`
- [ ] v1.0 features
- [ ] Design decisions
- [ ] Known limitations
- [ ] Accuracy expectations

---

## Progress Summary

**Phases Complete**: 1 / 10
**Hours Invested**: 0.5 / 21.5
**Current LOC**: 230 / 380 target
**Next Milestone**: Complete Phase 1 (Foundation)

---

## Issues & Resolutions

### Issue #1: Indicator Name Change
**Date**: 2025-01-20
**Description**: User requested name change from "MKN Exit Framework" to "MKN Super"
**Resolution**: Updated all references in code and documentation ✅

---

## Spec Verification Checklist

After each phase, verify against:
- [ ] mkn-super.md (original exit framework spec)
- [ ] mkn-super-addendum.md (entry additions spec)
- [ ] All formulas match specification exactly
- [ ] All visual outputs match color/shape/position requirements
- [ ] All parameters use correct defaults and ranges
- [ ] All accuracy expectations documented honestly

---

**Last Updated**: 2025-01-20 - Phase 1.1 Complete
