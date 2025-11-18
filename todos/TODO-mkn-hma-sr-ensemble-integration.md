# TODO: MKN HMA Bands + S/R Ensemble Integration

**Goal:** Combine exit timing (HMA bands) with exit location (S/R levels) for high-probability exits

**Priority:** High - Strategic integration opportunity
**Status:** Not Started
**Estimated Effort:** 2-3 sessions

---

## Background

**Current State:**
- `mkn-hma-bands.pine` signals exits when price crosses bands with volume confirmation
- `sr-algo5-ensemble.pine` identifies S/R levels with strength 0-100
- **No integration:** HMA exits at arbitrary band levels (not key S/R zones)

**Problem:**
Exit signals may occur far from S/R levels, reducing effectiveness:
- Exit at upper band but resistance 5% away = early exit (missed profit)
- Exit at lower band but support 3% away = late exit (gave back gains)

**Opportunity:**
Combined exits at **HMA band + S/R confluence + volume spike** = optimal timing + location

---

## Implementation Options

### **Option A: Modify mkn-hma-bands.pine (Standalone Enhancement)**

Add S/R level inputs via `input.source()`:
```pinescript
// Import S/R levels from ensemble
srLevel1 = input.source(close, "S/R Level 1", group="S/R Integration")
srLevel2 = input.source(close, "S/R Level 2", group="S/R Integration")
srLevel3 = input.source(close, "S/R Level 3", group="S/R Integration")
// ... up to 10 levels

// Check proximity to S/R when at band
atSRConfluence = false
tolerance = input.float(1.5, "S/R Confluence Tolerance (%)", minval=0.5, maxval=5.0)
for i = 0 to 9
    levelPrice = array.get(srLevels, i)
    distance = math.abs(close - levelPrice) / levelPrice * 100
    if distance <= tolerance
        atSRConfluence := true
        break

// Enhanced signals
sellSignalEnhanced = sellSignal and atSRConfluence
buySignalEnhanced = buySignal and atSRConfluence
```

**Pros:**
- Minimal code duplication
- Uses existing sr-algo5-ensemble via `input.source()`
- Backward compatible (S/R integration optional)

**Cons:**
- User must manually map 10 S/R levels from ensemble (tedious)
- No access to S/R strength scores (only price levels)
- Cannot filter by S/R agreement (4/4 vs 2/4)

---

### **Option B: Create New Indicator (mkn-exit-strategy.pine)**

Combine full logic from both indicators:
```pinescript
indicator("MKN: Exit Strategy (HMA + S/R)", overlay=true)

// Include HMA calculation
hmaFast = f_hma(close, 55)
hmaSlow = f_hma(close, 89)
upperBand = hmaFast + ta.stdev(close, 55) * 2.0
lowerBand = hmaFast - ta.stdev(close, 55) * 2.0

// Include S/R detection (simplified from sr-algo5-ensemble)
// OR import via request.security() from published ensemble

// Enhanced exit logic
exitSignal = (atBand and volumeSpike and atSRLevel and srStrength > 70)
```

**Pros:**
- Full control over exit conditions
- Access to S/R strength and agreement scores
- Can optimize thresholds holistically

**Cons:**
- Code duplication (copies HMA logic)
- Must maintain separately from mkn-hma-bands
- Heavier computational load

---

### **Option C: Create Library + Both Standalone Indicators Use It**

Extract HMA calculation to `lib_hma_utils.pine`:
```pinescript
//@version=5
library("HMAUtils")

export f_hma(simple float src, simple int length) =>
    halfLen = length / 2
    sqrtLen = math.round(math.sqrt(length))
    wma1 = ta.wma(src, halfLen)
    wma2 = ta.wma(src, length)
    raw = 2 * wma1 - wma2
    ta.wma(raw, sqrtLen)

export calculateBands(float hmaValue, int hmaLength, float multiplier) =>
    stdDev = ta.stdev(close, hmaLength) * multiplier
    [hmaValue + stdDev, hmaValue - stdDev]
```

Then both indicators import it:
```pinescript
// In mkn-hma-bands.pine
import yourname/HMAUtils/1 as hma_lib
hmaFast = hma_lib.f_hma(close, 55)

// In new mkn-exit-strategy.pine
import yourname/HMAUtils/1 as hma_lib
hmaFast = hma_lib.f_hma(close, 55)
```

**Pros:**
- Single source of truth for HMA logic
- Both indicators stay lightweight
- Easier maintenance (fix once, affects both)

**Cons:**
- Requires publishing new library to TradingView
- More upfront work

---

## Recommended Approach: **Option A → Option C**

**Phase 1: Quick Win (Option A)**
1. Modify mkn-hma-bands.pine to add S/R confluence check
2. Use `input.source()` to import 10 levels from sr-algo5-ensemble
3. Add enhanced signal types (normal vs enhanced)
4. Test on 60-day SPY daily

**Phase 2: Proper Refactor (Option C)**
1. Extract HMA logic to lib_hma_utils.pine
2. Publish library
3. Create mkn-exit-strategy.pine with full S/R integration
4. Refactor mkn-hma-bands.pine to use library

---

## Task Breakdown

### **Phase 1: S/R Confluence Integration (Quick Win)**

**File:** `mkn-hma-bands.pine`

- [ ] **Task 1.1:** Add S/R input section (10 levels via `input.source()`)
  - Lines to modify: After line 26 (after display inputs)
  - Add group: "S/R Integration"
  - 10 inputs: `srLevel1` through `srLevel10`

- [ ] **Task 1.2:** Add confluence tolerance input
  - Default: 1.5% (band within 1.5% of S/R level = confluence)
  - Range: 0.5% to 5.0%

- [ ] **Task 1.3:** Implement confluence detection function
  - Function: `checkSRConfluence(price, srLevels, tolerance) => bool`
  - Loop through 10 levels
  - Return true if any level within tolerance

- [ ] **Task 1.4:** Create enhanced signal variables
  - `sellSignalEnhanced = sellSignal and atSRConfluence`
  - `buySignalEnhanced = buySignal and atSRConfluence`
  - `sellSignalStrongEnhanced = sellSignalStrong and atSRConfluence`
  - `buySignalStrongEnhanced = buySignalStrong and atSRConfluence`

- [ ] **Task 1.5:** Add new plot shapes for enhanced signals
  - Star shape for enhanced signals (vs triangle for normal)
  - Brighter colors (#00FF00 for buy, #FF0000 for sell)
  - Larger size (`size.large` vs `size.normal`)

- [ ] **Task 1.6:** Update info table
  - Add row: "S/R Confluence: YES/NO"
  - Color: Green if YES, Gray if NO

- [ ] **Task 1.7:** Add alert conditions
  - `alertcondition(sellSignalEnhanced, "Enhanced Sell", "HMA + S/R + Volume")`
  - `alertcondition(buySignalEnhanced, "Enhanced Buy", "HMA + S/R + Volume")`

- [ ] **Task 1.8:** Update documentation
  - Add comment section explaining S/R integration
  - Explain how to connect to sr-algo5-ensemble
  - Document expected improvement (+10-15% exit accuracy)

**Estimated Lines:** +80 lines (180 LOC → 260 LOC)

---

### **Phase 2: Extract HMA Logic to Library**

**New File:** `lib_hma_utils.pine`

- [ ] **Task 2.1:** Create library structure
  - `//@version=5`
  - `library("HMAUtils")`
  - Add description and version

- [ ] **Task 2.2:** Export HMA calculation function
  - `export f_hma(float src, int length) => float`
  - Copy from mkn-hma-bands.pine:31-37

- [ ] **Task 2.3:** Export band calculation function
  - `export calculateBands(float hmaValue, int length, float multiplier) => [float, float]`
  - Returns `[upperBand, lowerBand]`

- [ ] **Task 2.4:** Export band state detection
  - `export isBandExpanding(float bandWidth, int lookback) => bool`
  - Returns true if bandWidth > SMA(bandWidth, lookback)

- [ ] **Task 2.5:** Add type definition for HMA data
  - `export type HMAData`
  - Fields: `fast, slow, upperBand, lowerBand, isExpanding`

- [ ] **Task 2.6:** Export all-in-one calculation function
  - `export calculateHMA(float src, int fastLen, int slowLen, float stdMult) => HMAData`
  - Convenience wrapper

- [ ] **Task 2.7:** Publish library to TradingView
  - Ensure username is correct
  - Add tags: "hma", "bands", "moving-average"
  - Version: 1.0.0

**Estimated Lines:** ~120 lines (new library)

---

### **Phase 3: Refactor mkn-hma-bands.pine to Use Library**

**File:** `mkn-hma-bands.pine`

- [ ] **Task 3.1:** Add library import
  - Line 2 (after indicator declaration)
  - `import yourname/HMAUtils/1 as hma_lib`

- [ ] **Task 3.2:** Replace inline HMA function with library call
  - Remove lines 31-37 (f_hma function)
  - Replace: `hmaFastLine = f_hma(close, hmaFast)`
  - With: `hmaFastLine = hma_lib.f_hma(close, hmaFast)`

- [ ] **Task 3.3:** Replace band calculation with library call
  - Replace lines 44-47
  - With: `[upperBand, lowerBand] = hma_lib.calculateBands(hmaFastLine, hmaFast, stdDevMult)`

- [ ] **Task 3.4:** Replace band expansion check
  - Replace line 51
  - With: `isExpanding = hma_lib.isBandExpanding(bandWidth, 20)`

- [ ] **Task 3.5:** Test indicator still works
  - Load on SPY daily chart
  - Verify HMA lines match previous version
  - Verify signals match previous version

**Estimated Lines Removed:** -30 lines (260 LOC → 230 LOC)

---

### **Phase 4: Create Integrated Exit Strategy Indicator**

**New File:** `mkn-exit-strategy.pine`

- [ ] **Task 4.1:** Create indicator scaffold
  - Copy structure from mkn-hma-bands.pine
  - Rename to "MKN: Exit Strategy (HMA + S/R)"
  - Add overlay=true, max_lines_count=50

- [ ] **Task 4.2:** Import libraries
  - `import yourname/HMAUtils/1 as hma_lib`
  - `import redshad0ww/CoreMath/3 as math_lib`
  - `import redshad0ww/VolumeAnalysis/3 as vol_lib`

- [ ] **Task 4.3:** Add inputs for both HMA and S/R
  - HMA parameters (fast, slow, multiplier)
  - Volume parameters (period, min spike, strong spike)
  - S/R parameters (min strength, min agreement, tolerance)

- [ ] **Task 4.4:** Calculate HMA bands
  - Use `hma_lib.f_hma()` for fast and slow
  - Use `hma_lib.calculateBands()` for upper/lower

- [ ] **Task 4.5:** Import S/R levels via `input.source()`
  - 10 level inputs
  - 10 strength inputs (optional, if available)

- [ ] **Task 4.6:** Calculate volume metrics
  - Use `vol_lib.calculateVolumeMetrics()`
  - Extract `volRatio`

- [ ] **Task 4.7:** Implement confluence detection
  - Function: `findNearestSR(price, srLevels, tolerance) => [int, float]`
  - Returns: [index of nearest level, distance %]

- [ ] **Task 4.8:** Create tiered exit signals
  - **Tier 3 (Highest):** HMA band + S/R strength >80 + volume >2.0x
  - **Tier 2 (Medium):** HMA band + S/R strength >60 + volume >1.5x
  - **Tier 1 (Basic):** HMA band + volume >1.5x (no S/R)

- [ ] **Task 4.9:** Add visualization
  - Plot HMA lines (same colors as mkn-hma-bands)
  - Plot S/R levels as dashed lines
  - Different shapes for each tier (star, triangle, circle)
  - Color intensity based on tier (brighter = higher)

- [ ] **Task 4.10:** Add comprehensive info table
  - Current tier
  - Nearest S/R distance
  - Volume ratio
  - Band state
  - Bias (bullish/bearish/neutral)

- [ ] **Task 4.11:** Add alert conditions for all tiers
  - 6 alerts: Tier 1/2/3 Buy, Tier 1/2/3 Sell

- [ ] **Task 4.12:** Add documentation section
  - Explain tier system
  - Explain how to connect to sr-algo5-ensemble
  - Example usage scenarios

**Estimated Lines:** ~350 lines (new indicator)

---

## Testing Protocol

### **Phase 1 Testing (Quick Win):**
1. Load mkn-hma-bands.pine on SPY daily chart
2. Add sr-algo5-ensemble.pine to same chart
3. Connect each S/R level plot to corresponding input
4. Verify confluence detection works (check distance calculation)
5. Verify enhanced signals appear only at S/R confluence
6. Count signals: Normal vs Enhanced (expect 30-50% enhancement rate)

### **Phase 2 Testing (Library):**
1. Publish lib_hma_utils to TradingView
2. Load refactored mkn-hma-bands
3. Side-by-side comparison: Old vs New (should be identical)
4. Check performance (execution time should be same or better)

### **Phase 4 Testing (Exit Strategy):**
1. Load mkn-exit-strategy.pine on SPY daily
2. Add sr-algo5-ensemble.pine
3. Connect S/R levels
4. Track exit signals by tier (Tier 3 should be rarest, highest accuracy)
5. Compare to standalone mkn-hma-bands (should have fewer but better signals)

**Success Metrics:**
- Phase 1: 30-50% of signals now "enhanced" (at S/R confluence)
- Phase 4: Tier 3 signals 50% fewer but 15-20% more accurate (if backtestable)

---

## Expected Improvements

**Before Integration:**
- HMA exits at arbitrary band levels
- Exit accuracy: ~60-70% (mean reversion assumption)
- No awareness of key S/R zones

**After Integration (Phase 1):**
- HMA exits prioritize S/R confluence
- Expected accuracy: **70-80%** (+10-15% improvement)
- Fewer signals but higher quality

**After Integration (Phase 4):**
- Tiered system separates high/medium/low confidence
- Tier 3 expected accuracy: **75-85%**
- Tier 2 expected accuracy: **70-80%**
- Tier 1 expected accuracy: **65-75%** (baseline)

---

## Dependencies

**Required:**
- sr-algo5-ensemble.pine (must be on same chart)
- Understanding of `input.source()` for cross-indicator data

**Optional (Phase 2):**
- TradingView account with library publishing rights

**Libraries to Use:**
- lib_core_math.pine (safe math operations)
- lib_volume_analysis.pine (volume metrics)
- lib_hma_utils.pine (Phase 2+, new library)

---

## Risks & Mitigation

**Risk 1: `input.source()` requires manual mapping (tedious)**
- Mitigation: Provide clear documentation with screenshots
- Future: Request TradingView add array input.source() feature

**Risk 2: S/R levels may be far from HMA bands (low confluence rate)**
- Mitigation: Use wider tolerance (2-3%) for initial testing
- Monitor confluence rate (should be >30%, else too strict)

**Risk 3: Tier system may be too complex for users**
- Mitigation: Default to "Auto" mode (use highest tier available)
- Add toggle: "Show All Tiers" vs "Show Best Only"

**Risk 4: Performance degradation (checking 10 S/R levels every bar)**
- Mitigation: Use early break in loop when match found
- Worst case: +20ms per bar (acceptable for TradingView)

---

## Next Steps After Completion

1. **Backtest Validation:**
   - If TradingView adds backtesting for indicators, run 60-day test
   - Compare Tier 3 accuracy vs Tier 1 vs standalone HMA

2. **Parameter Optimization:**
   - Optimize tolerance (0.5% to 3%)
   - Optimize S/R strength threshold (60 to 90)
   - Optimize volume thresholds by regime

3. **Regime Integration:**
   - Add lib_regime_detection for volatility filtering
   - High Vol: wider tolerance, higher volume threshold
   - Low Vol: tighter tolerance, lower volume threshold

4. **Multi-Timeframe Version:**
   - Require HMA band on 2/3 timeframes for signal
   - Require S/R level on 2/3 timeframes for confluence

---

## Files to Create/Modify

**Phase 1:**
- ✏️ Modify: `mkn-hma-bands.pine` (+80 lines)

**Phase 2:**
- ✨ Create: `lib_hma_utils.pine` (120 lines)
- ✏️ Modify: `mkn-hma-bands.pine` (-30 lines refactor)

**Phase 4:**
- ✨ Create: `mkn-exit-strategy.pine` (350 lines)

**Documentation:**
- ✏️ Update: `ALGORITHM-SYNTHESIS.md` (add Phase 1/4 results)
- ✨ Create: `docs/indicators/mkn-exit-strategy.md` (usage guide)

---

## Completion Criteria

- [ ] Phase 1 complete: mkn-hma-bands has S/R integration
- [ ] Phase 2 complete: lib_hma_utils published and used
- [ ] Phase 4 complete: mkn-exit-strategy works with tiered signals
- [ ] Testing complete: Side-by-side comparison shows improvement
- [ ] Documentation updated: All changes documented in synthesis

**Target Date:** TBD (estimate 2-3 sessions based on complexity)
