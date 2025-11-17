# PineScript Library Refactoring - Detailed Checklist

**Start Date**: 2025-01-17
**Timeline**: 2 weeks (14 days)
**Approach**: All-In Refactor
**Target**: 9 core indicators
**Expected Savings**: 641 lines (17% reduction), 76% less duplication

---

## 📋 Phase 0: Safety Checkpoint (Day 0)

**Status**: ✅ Complete

- [x] Create this file (refactoring-plan-todo.md)
- [x] Git commit current state with detailed message
- [x] Create git tag: `v1.1-stable-pre-refactor`
- [x] Push tag to remote (if applicable)
- [x] Verify clean working directory
- [x] Document current line counts for comparison (3,671 lines total)

**Git Commit Message**:
```
Pre-refactor checkpoint: Save working state before library migration

Current state:
- 9 indicators functional and tested
- institutional-algo1: v6, all others: v5
- ~841 lines of duplication identified
- Ready for library refactoring

This checkpoint allows rollback if needed.
```

---

## 📚 Phase 1: Library Development (Days 1-3)

**Status**: ✅ Complete
**Estimated Time**: 8-12 hours
**Actual Time**: ~6 hours

### Library 1: lib_core_math.pine (86 lines)
- [x] Create file structure with v5 header
- [x] Add JSDoc description and version
- [x] Implement `safeDivide(numerator, denominator, defaultValue)`
- [x] Implement `safeRange(priceRange, close, minPercent)`
- [x] Implement `safeArrayGet(array, index, defaultValue)`
- [x] Add comprehensive JSDoc comments for all functions
- [x] Test locally in Pine Editor
- [x] Publish to TradingView as **Private** library
- [x] Record library URL: `redshad0ww/CoreMath/1`

### Library 2: lib_regime_detection.pine (107 lines)
- [x] Create file structure with v5 header
- [x] Define `RegimeData` type (name, atrRatio, multiplier, isHighVol, isLowVol, isNormalVol)
- [x] Implement `detectRegime(atrLength, atrLookback)` function
- [x] Add standard parameters: ATR length = 14, lookback = 50
- [x] Add thresholds: High vol = 1.3×, Low vol = 0.7×
- [x] Add multipliers: Low vol = 1.15, Normal = 1.0, High vol = 0.85
- [x] Add comprehensive JSDoc comments
- [x] Test locally in Pine Editor
- [x] Publish to TradingView as **Private** library
- [x] Record library URL: `redshad0ww/RegimeDetection/1`

### Library 3: lib_volume_analysis.pine (156 lines)
- [x] Create file structure with v5 header
- [x] Import lib_core_math as dependency
- [x] Define `VolumeMetrics` type (relativeVolume, volumeEfficiency, efficiencyRatio, volumeScore)
- [x] Implement `calculateVolumeMetrics(lookback)` function
- [x] Use `safeDivide()` from lib_core_math
- [x] Add comprehensive JSDoc comments
- [x] Test locally in Pine Editor
- [x] Publish to TradingView as **Private** library
- [x] Record library URL: `redshad0ww/VolumeAnalysis/1`

### Library 4: lib_trend_detection.pine (185 lines)
- [x] Create file structure with v5 header
- [x] Define `TrendData` type (isUptrend, isDowntrend, isRanging, adxValue, isTrending, isStrongTrend)
- [x] Implement `detectEMATrend(short, mid, long)` function
- [x] Implement `detectADXTrend(length, trendingThreshold)` function
- [x] Add comprehensive JSDoc comments
- [x] Test locally in Pine Editor
- [x] Publish to TradingView as **Private** library
- [x] Record library URL: `redshad0ww/TrendDetection/1`

### Library 5: lib_mtf_utils.pine (145 lines)
- [x] Create file structure with v5 header
- [x] Implement `getHigherTimeframe(baseTF)` function
- [x] Implement `getSecondHigherTimeframe(baseTF)` function
- [x] Implement `getThirdHigherTimeframe(baseTF)` function
- [x] Handle all timeframe types: minutes, hours, daily, weekly, monthly
- [x] Add comprehensive JSDoc comments
- [x] Test locally in Pine Editor
- [x] Publish to TradingView as **Private** library
- [x] Record library URL: `redshad0ww/MTFUtils/1`

### Phase 1 Completion
- [x] All 5 libraries published successfully (679 total lines)
- [x] All library URLs recorded above
- [x] Git commit: "feat: Create 5 shared libraries for code reuse"
- [x] Git push

---

## 🏢 Phase 2: Institutional Indicators Refactor (Days 4-6)

**Status**: 🟡 In Progress (2/4 complete)
**Estimated Time**: 12-16 hours

### Pre-Refactor: Version Standardization
- [x] Open institutional-algo1-volume-efficiency.pine
- [x] Change `//@version=6` to `//@version=5` (line 1)
- [x] Test in Pine Editor (verify no breaking changes)
- [x] Git commit: "chore: Downgrade institutional-algo1 from v6 to v5"

### Institutional-Algo1: Volume Efficiency
**Before**: 320 lines | **After**: 309 lines | **Saved**: 11 lines

- [x] Add library imports at top (CoreMath, VolumeAnalysis, TrendDetection)
- [x] Replace volume calculations with `volMetrics = vol_lib.calculateVolumeMetrics()`
- [x] Replace EMA trend detection with `trend_lib.detectEMATrend()`
- [x] Replace ADX detection with `trend_lib.detectADXTrend()`
- [x] Replace safe division calls with `math_lib.safeDivide()`
- [x] Update variable references to use library types
- [x] Test in Pine Editor - verify compilation
- [x] Compare calculations with pre-refactor version (spot check 10 bars)
- [x] Git commit: "refactor: Migrate institutional-algo1 to shared libraries"

### Institutional-Algo2: MTF Convergence
**Before**: 323 lines | **After**: 292 lines | **Saved**: 31 lines

- [x] Add library imports at top (CoreMath, MTFUtils, RegimeDetection)
- [x] Replace regime detection code with `regime_lib.detectRegime()`
- [x] Remove duplicate getHigherTimeframe() functions (33 lines)
- [x] Replace MTF timeframe detection with `mtf_lib.getHigherTimeframe()`
- [x] Replace safe math operations with `math_lib.safeDivide()`
- [x] Update variable references to use library types
- [x] Test in Pine Editor - verify compilation
- [x] Compare calculations with pre-refactor version
- [x] Git commit: "refactor: Migrate institutional-algo2 to shared libraries"

### Institutional-Algo3: Bayesian Regime
**Current**: 395 lines | **Estimated Lines Changed**: ~85

- [ ] Add library imports at top
- [ ] Replace regime detection code with library call
- [ ] Replace volume calculations with library call
- [ ] Replace ADX trend detection with `lib_trend_detection.detectADXTrend()`
- [ ] Replace safe math operations with library calls
- [ ] Update variable references to use library types
- [ ] Test in Pine Editor - verify compilation
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### Institutional-Algo4: Ensemble
**Current**: 473 lines | **Estimated Lines Changed**: ~120

- [ ] Add library imports at top
- [ ] Replace regime detection code with library call
- [ ] Replace volume calculations with library call
- [ ] Replace trend detection code with library calls
- [ ] Replace safe math operations with library calls
- [ ] Replace simplified algo1/2/3 code with library calls
- [ ] Update variable references to use library types
- [ ] Test in Pine Editor - verify compilation
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### Phase 2 Completion
- [ ] All 4 institutional indicators refactored successfully
- [ ] All indicators compile without errors
- [ ] Calculation parity verified (spot checks)
- [ ] Git commit: "refactor: Migrate institutional indicators to libraries"
- [ ] Git push

---

## 🎯 Phase 3: S/R Indicators Refactor (Days 7-9)

**Status**: ⚪ Pending
**Estimated Time**: 12-16 hours

### SR-Algo1: Volume Profile
**Current**: 529 lines | **Estimated Lines Changed**: ~70

- [ ] Add library imports at top (lib_core_math, lib_regime_detection, lib_volume_analysis)
- [ ] Replace regime detection code with library call
- [ ] Replace volume calculations with library call
- [ ] Replace safe division with library calls
- [ ] Update variable references to use library types
- [ ] **Verify POC/VAH/VAL exports unchanged** (sr-algo3 depends on these!)
- [ ] Test in Pine Editor - verify compilation
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### SR-Algo2: Statistical Peaks
**Current**: 520 lines | **Estimated Lines Changed**: ~90

- [ ] Add library imports at top
- [ ] Replace regime detection code with library call
- [ ] Replace volume calculations with library call (volume-weighted touches)
- [ ] Replace safe math operations with library calls
- [ ] Update variable references to use library types
- [ ] Test in Pine Editor - verify compilation
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### SR-Algo3: MTF Confluence
**Current**: 584 lines | **Estimated Lines Changed**: ~120

- [ ] Add library imports at top
- [ ] Replace regime detection code with library call
- [ ] Replace MTF timeframe detection with `lib_mtf_utils` functions
- [ ] Replace safe math operations with library calls
- [ ] **PRESERVE existing `input.source()` imports from SR-Algo1** (POC/VAH/VAL)
- [ ] Update variable references to use library types
- [ ] Test in Pine Editor - verify compilation
- [ ] Verify volume profile integration still works
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### SR-Algo4: Order Book
**Current**: 527 lines | **Estimated Lines Changed**: ~70

- [ ] Add library imports at top
- [ ] Replace regime detection code with library call
- [ ] Replace volume calculations with library call
- [ ] Replace safe math operations with library calls
- [ ] Update variable references to use library types
- [ ] Test in Pine Editor - verify compilation
- [ ] Compare calculations with pre-refactor version
- [ ] Document any breaking changes

### Phase 3 Completion
- [ ] All 4 S/R indicators refactored successfully
- [ ] All indicators compile without errors
- [ ] SR-Algo3 → SR-Algo1 data bridge still functional
- [ ] Calculation parity verified (spot checks)
- [ ] Git commit: "refactor: Migrate S/R indicators to libraries"
- [ ] Git push

---

## 🔄 Phase 4: SR-Ensemble Rebuild (Day 10)

**Status**: ⚪ Pending
**Estimated Time**: 6-8 hours

### Planning
- [ ] Review current sr-ensemble.pine (if exists)
- [ ] Document current functionality to preserve
- [ ] Design new architecture (libraries + data bridges)

### Implementation (~200 lines)
- [ ] Create new sr-ensemble.pine with v5 header
- [ ] Add all 5 library imports
- [ ] Add `input.source()` imports for SR-Algo1-4 data:
  - [ ] POC/VAH/VAL from SR-Algo1
  - [ ] Statistical level strength from SR-Algo2
  - [ ] MTF confluence strength from SR-Algo3
  - [ ] Order book rejection strength from SR-Algo4
- [ ] Implement regime detection via library call
- [ ] Implement weighted voting system:
  - [ ] Volume Profile: 30% weight
  - [ ] Statistical Peaks: 25% weight
  - [ ] MTF Confluence: 35% weight
  - [ ] Order Book: 10% weight
- [ ] Implement agreement scoring (4/4, 3/4, 2/4, 1/4)
- [ ] Add confidence levels (High/Medium/Low)
- [ ] Create visualization (S/R levels on chart)
- [ ] Add connection status table (verify all data bridges connected)
- [ ] Add tooltips with setup instructions

### Testing
- [ ] Test in Pine Editor - verify compilation
- [ ] Test with all 4 SR algos on same chart
- [ ] Verify data bridge connections work
- [ ] Verify connection status table shows correctly
- [ ] Test ensemble scoring logic
- [ ] Compare with old ensemble behavior (if applicable)

### Phase 4 Completion
- [ ] New sr-ensemble.pine functional
- [ ] All data bridges connected
- [ ] Delete old sr-ensemble.pine (if different)
- [ ] Git commit: "feat: Rebuild sr-ensemble with library architecture"
- [ ] Git push

**Old Lines**: ~594 (estimated)
**New Lines**: ~200
**Net Savings**: ~400 lines

---

## 🧪 Phase 5: Testing & Validation (Days 11-13)

**Status**: ⚪ Pending
**Estimated Time**: 12-16 hours

### Comprehensive Backtesting

**Test Matrix**: 3 Timeframes × 3 Assets = 9 Test Scenarios

#### Timeframes to Test
- [ ] 1 Hour (1H)
- [ ] 4 Hour (4H)
- [ ] Daily (D)

#### Assets to Test
- [ ] Stock: SPY (S&P 500 ETF)
- [ ] Crypto: BTC/USD
- [ ] Forex: EUR/USD

### Indicator Testing

#### Institutional-Algo1 (Volume Efficiency)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### Institutional-Algo2 (MTF Convergence)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### Institutional-Algo3 (Bayesian Regime)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### Institutional-Algo4 (Ensemble)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### SR-Algo1 (Volume Profile)
- [ ] 1H / SPY - No errors, calculations match, exports POC/VAH/VAL
- [ ] 4H / BTC - No errors, calculations match, exports POC/VAH/VAL
- [ ] D / EUR - No errors, calculations match, exports POC/VAH/VAL

#### SR-Algo2 (Statistical Peaks)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### SR-Algo3 (MTF Confluence)
- [ ] 1H / SPY - No errors, calculations match, imports from Algo1 work
- [ ] 4H / BTC - No errors, calculations match, imports from Algo1 work
- [ ] D / EUR - No errors, calculations match, imports from Algo1 work

#### SR-Algo4 (Order Book)
- [ ] 1H / SPY - No errors, calculations match
- [ ] 4H / BTC - No errors, calculations match
- [ ] D / EUR - No errors, calculations match

#### SR-Ensemble (Rebuilt)
- [ ] 1H / SPY - No errors, all data bridges connected, ensemble scoring works
- [ ] 4H / BTC - No errors, all data bridges connected, ensemble scoring works
- [ ] D / EUR - No errors, all data bridges connected, ensemble scoring works

### Calculation Parity Verification
- [ ] Create comparison table (pre vs post refactor)
- [ ] Sample 10 bars from each indicator
- [ ] Verify regime detection matches across all indicators
- [ ] Verify volume metrics match pre-refactor values
- [ ] Verify trend detection matches pre-refactor values
- [ ] **Expected Result**: 100% parity (no calculation changes)

### Performance Testing
- [ ] Measure execution time for each indicator (target: <500ms per bar)
- [ ] Check for NA errors on early bars (first 200 bars)
- [ ] Verify array size limits not exceeded
- [ ] Monitor memory usage (if possible)

### Breaking Changes Audit
- [ ] Document any input parameter name changes
- [ ] List any visual/output differences
- [ ] Identify alerts that need recreation
- [ ] Identify saved chart layouts that may break
- [ ] Create user migration checklist

### Phase 5 Completion
- [ ] All 9 indicators tested on 3 timeframes × 3 assets (81 total tests)
- [ ] Calculation parity verified
- [ ] Performance acceptable (<500ms)
- [ ] No NA errors found
- [ ] Breaking changes documented
- [ ] Git commit: "test: Validate refactored indicators"
- [ ] Git push

---

## 📝 Phase 6: Documentation & Final Commit (Day 14)

**Status**: ⚪ Pending
**Estimated Time**: 6-8 hours

### Update TECH-DEBT.md
- [ ] Mark regime detection as "✅ Refactored (lib_regime_detection.pine)"
- [ ] Mark volume calculations as "✅ Refactored (lib_volume_analysis.pine)"
- [ ] Mark MTF helpers as "✅ Refactored (lib_mtf_utils.pine)"
- [ ] Mark safe math as "✅ Refactored (lib_core_math.pine)"
- [ ] Mark trend detection as "✅ Refactored (lib_trend_detection.pine)"
- [ ] Update duplication statistics:
  - [ ] Before: 841 lines duplicated (23%)
  - [ ] After: ~200 lines duplicated (~7%)
  - [ ] Improvement: 641 lines saved (76% reduction in duplication)
- [ ] Update file counts:
  - [ ] Regime detection: 9 files → 1 library
  - [ ] Volume calculations: 7 files → 1 library
  - [ ] MTF helpers: 4 files → 1 library
  - [ ] Safe math: 9 files → 1 library
- [ ] Add library maintenance section
- [ ] Document library version management strategy

### Update CLAUDE.md
- [ ] Add new session entry: "2025-01-17 - Library Refactoring - COMPLETE ✅"
- [ ] Document refactoring summary:
  - [ ] 5 libraries created (~430 lines)
  - [ ] 9 indicators refactored
  - [ ] 641 lines saved (17% reduction)
  - [ ] 76% reduction in duplication
- [ ] Update "Key Project Files" section:
  - [ ] Add 5 library entries
  - [ ] Update indicator descriptions (now using libraries)
- [ ] Update "Important Decisions & Context" section:
  - [ ] Document library architecture decision
  - [ ] Document v5 standardization decision
  - [ ] Document private publishing decision
- [ ] Update "Quick Reference Links" table:
  - [ ] Add 5 library rows
  - [ ] Update status for all indicators (✅ Production + Libraries)

### Create MIGRATION-GUIDE.md
- [ ] Create new file: MIGRATION-GUIDE.md
- [ ] Document version change: v1.1 → v2.0
- [ ] List all breaking changes by indicator
- [ ] Document parameter name changes (if any)
- [ ] Provide alert recreation instructions
- [ ] Provide chart layout update instructions
- [ ] Include before/after screenshots (if applicable)
- [ ] Add troubleshooting section
- [ ] Add rollback instructions (revert to v1.1-stable-pre-refactor tag)

### Update refactoring-plan-todo.md
- [ ] Mark all phases as complete
- [ ] Add "Results" section at bottom:
  - [ ] Actual vs estimated time
  - [ ] Actual vs estimated line savings
  - [ ] Issues encountered and resolutions
  - [ ] Lessons learned
- [ ] Add completion date
- [ ] Archive as historical record (don't delete)

### Final Git Work
- [ ] Review all changes across all files
- [ ] Verify no debug code left in indicators
- [ ] Verify all TODOs resolved
- [ ] Git commit: "docs: Update documentation post-refactor"
- [ ] Create git tag: `v2.0-library-architecture`
- [ ] Git push (commits and tags)

### Phase 6 Completion
- [ ] TECH-DEBT.md updated
- [ ] CLAUDE.md updated
- [ ] MIGRATION-GUIDE.md created
- [ ] refactoring-plan-todo.md archived with results
- [ ] All commits pushed
- [ ] v2.0 tag created and pushed

---

## 📊 Final Metrics & Validation

### Code Statistics
- [ ] Count total lines before refactor: `____` lines
- [ ] Count total lines after refactor: `____` lines
- [ ] Calculate actual savings: `____` lines (`____`%)
- [ ] Compare to projected savings: 641 lines (17%)

### Duplication Statistics
- [ ] Count duplicated lines before: `____` lines (`____`%)
- [ ] Count duplicated lines after: `____` lines (`____`%)
- [ ] Calculate duplication reduction: `____`%
- [ ] Compare to projected reduction: 76%

### File Count
- [ ] Total files before: 9 indicators
- [ ] Total files after: 5 libraries + 9 indicators = 14 files
- [ ] Shared code files: 5 libraries (reusable)

### Library Adoption
- [ ] institutional-algo1 uses libraries: ✓
- [ ] institutional-algo2 uses libraries: ✓
- [ ] institutional-algo3 uses libraries: ✓
- [ ] institutional-algo4 uses libraries: ✓
- [ ] sr-algo1 uses libraries: ✓
- [ ] sr-algo2 uses libraries: ✓
- [ ] sr-algo3 uses libraries: ✓
- [ ] sr-algo4 uses libraries: ✓
- [ ] sr-ensemble uses libraries: ✓
- [ ] **Library Adoption Rate**: 100%

### Testing Coverage
- [ ] All 9 indicators tested: ✓
- [ ] All 3 timeframes covered: ✓
- [ ] All 3 asset types covered: ✓
- [ ] Total test scenarios: 81 (9 × 3 × 3)
- [ ] Passing tests: `____` / 81 (`____`%)

### Version Control
- [ ] Pre-refactor tag created: `v1.1-stable-pre-refactor`
- [ ] Post-refactor tag created: `v2.0-library-architecture`
- [ ] All commits include descriptive messages: ✓
- [ ] All commits pushed to remote: ✓

---

## 🎯 Success Criteria

- [x] All 5 libraries created and published
- [ ] All 9 indicators successfully refactored
- [ ] All indicators compile without errors
- [ ] Calculation parity verified (100% match)
- [ ] Performance acceptable (<500ms per bar)
- [ ] No NA errors on early bars
- [ ] Code reduction ≥15% (target: 17%)
- [ ] Duplication reduction ≥70% (target: 76%)
- [ ] All tests passing (≥95%)
- [ ] Documentation complete (TECH-DEBT, CLAUDE, MIGRATION-GUIDE)
- [ ] Git tags created (pre and post)
- [ ] Rollback path documented and tested

---

## 📌 Notes & Issues

**Issues Encountered**:
- _Record any blockers, bugs, or unexpected challenges here_

**Deviations from Plan**:
- _Record any changes to the original plan_

**Performance Notes**:
- _Record execution times, memory usage, or other performance observations_

**User Feedback**:
- _Record any user reports after deployment_

---

**Status Legend**:
- ⚪ Pending
- 🟡 In Progress
- ✅ Complete
- ❌ Failed / Blocked

**Completion Date**: `____________`
**Total Time**: `____` hours
