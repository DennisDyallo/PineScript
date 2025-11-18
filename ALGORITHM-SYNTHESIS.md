# Algorithm Inventory & Synthesis
**Research-Grade Summary for External Collaboration**

---

## Overview

**11 Indicators:** 5 S/R algorithms + 4 institutional algorithms + 2 trading strategy indicators
**6 Libraries:** Shared utilities (published to TradingView)
**Total LOC:** ~4,500+ lines PineScript v5/v6

---

## 1. Support/Resistance Algorithms (WHERE to trade)

### sr-algo1-volume-profile.pine (380 LOC)
**Method:** Volume-at-price distribution analysis
**Technique:** Divides price range into 30-60 bins, calculates volume distribution
**Outputs:** POC (Point of Control), VAH/VAL (Value Area), HVN/LVN zones
**Target Accuracy:** 75-85% (research estimate)

**Strengths:**
- Volume-weighted price levels (institutional footprints)
- Dynamic bin sizing adapts to volatility
- Exports POC/VAH/VAL via `plot()` for cross-indicator use

**Limitations:**
- Fixed 500-bar lookback (memory constraint)
- No multi-timeframe aggregation
- POC can shift on ranging markets

**Optimization Potential:**
- Adaptive lookback (regime-dependent: 200 bars low vol, 1000 bars high vol)
- Session-based profiling (RTH vs ETH for futures)
- Integration with Order Flow Imbalance (algo4 wick rejection)

---

### sr-algo2-statistical-peaks.pine (355 LOC) - v1.1
**Method:** DBSCAN clustering of swing points with temporal decay
**Technique:** Pivot detection → cluster formation → confluence scoring
**Outputs:** Clustered S/R levels with strength 0-100
**Target Accuracy:** 70-80% daily, 65-75% intraday

**Strengths:**
- Volume-weighted touch counts (institutional absorption)
- Multi-magnitude psychological levels ($100, $250, $500, $1000)
- ATR-relative epsilon (auto-adapts to volatility)
- Temporal decay (0.9942^bars, 120-bar half-life)

**Limitations:**
- Pivot detection lag (10-bar confirmation = 40 hours on 4H)
- Clustering requires ≥2 touches (MinClusterSize=2)
- No ML validation (PineScript constraint)

**Optimization Potential:**
- Replace pivots with fractals (2-bar patterns, faster detection)
- Exponential distance decay option (current uses floor at 0.5)
- Cross-timeframe cluster merging (validate with algo3)

**Known Issues (FIXED in v1.1):**
- ✅ Temporal decay corrected (0.992 → 0.9942)
- ✅ Volume weighting added (sqrt-dampened)
- ✅ Confluence changed to multiplicative (was additive)

---

### sr-algo3-mtf-confluence.pine (405 LOC) - v1.1
**Method:** Multi-timeframe swing point convergence
**Technique:** Auto-detect 3 TFs → merge within tolerance → weight by TF count
**Outputs:** Confluence levels with TF-weighted strength
**Target Accuracy:** 80-90% (3-TF), 65-75% (2-TF), 55-65% (1-TF)
**Realistic Weighted:** 65-70% (OHLCV ceiling is 70-75%)

**Strengths:**
- Auto-timeframe selection (5min→30min→4H or D→W→M)
- Touch tracking on EVERY bar (not just current)
- Rejection analysis with next-bar confirmation
- Volume profile integration via `input.source()`
- Real-time performance tracking (touches, rejections, age)

**Limitations:**
- Pivot lag compounds across 3 TFs (192 hours on 1H→Daily, 56 days Daily→Weekly)
- Cannot backtest historically (TradingView security model)
- Multi-TF error compounding (70% × 70% × 70% = 34% for 3-TF)
- Regime dependency without ML (-5% to -10% penalty)

**Optimization Potential:**
- Fractal detection (5-bar patterns, ~30% faster than pivots)
- Basic volume validation (require high-volume confirmation)
- Distance filtering with exponential decay (relevance weighting)
- Spatial hashing for merge performance (O(n³) → O(n log n))

**Known Issues (FIXED in v1.1):**
- ✅ Prominence asymmetric window corrected
- ✅ Merge type separation (S vs R levels)
- ✅ Touch tracking rewritten (runs every bar, not just last)
- ✅ Volume profile integration added (POC/VAH/VAL confluence)

---

### sr-algo4-order-book.pine (345 LOC)
**Method:** Wick rejection analysis (order book proxy)
**Technique:** Wick-to-body ratio → estimate absorbed volume
**Outputs:** Rejection zones with estimated order volume
**Target Accuracy:** 65-75% (experimental/supplementary)

**Strengths:**
- No order book data required (retail-accessible)
- Detects institutional absorption zones
- Complements volume profile (order flow context)

**Limitations:**
- Wick size doesn't always = absorption (can be low liquidity)
- No true order book data (OHLCV proxy only)
- Lower accuracy than other algos (designed as supplementary)

**Optimization Potential:**
- Combine with algo1 volume bins (rejection at HVN = stronger)
- Filter by relative volume (require 1.5x avg volume for validity)
- Multi-bar rejection sequences (3+ bars rejecting same level)

---

### sr-algo5-ensemble.pine (475 LOC) - ALPHA v1.1
**Method:** Weighted voting from all 4 S/R algorithms
**Technique:** Merge within 0.2% tolerance → weighted average → agreement bonus
**Outputs:** Ensemble strength 0-100, agreement 1/4 to 4/4
**Target Accuracy:** 70-78% paper, 65-73% live (with 5-7% friction penalty)

**Weighting:**
- **DEFAULT (v1.1):** Equal weights 25% each (DeMiguel 2009 - safest without data)
- **OPTIONAL:** Original VP(35%) MTF(30%) STAT(20%) OB(15%) - UNVALIDATED

**Strengths:**
- Combines all detection methods (diversity)
- Agreement bonuses (4/4: +15%, 3/4: +10%, 2/4: +5%)
- Toggle for equal vs original weights (user experimentation)
- Contains simplified versions of all 4 algorithms (no imports needed)

**Limitations:**
- **ALPHA status:** ZERO empirical testing (theory only)
- Weight contradiction: MTF highest accuracy (80-90%) but middle weight (30%) in original
- Highest maintenance burden (duplicates all 4 algorithm cores)
- Cannot import from other indicators (PineScript security model)

**Optimization Potential:**
- Empirical weight optimization (60-day forward testing → Breiman log-odds)
- Regime-filtered confidence (show "High Vol - Use Caution" labels)
- Lag-adjusted weighting (penalize MTF for 10-bar pivot lag)
- Automated performance tracking (CSV export for validation)

**Known Issues (FIXED in v1.1):**
- ✅ Formula bug (removed `* 100` causing 0-10,000 range saturation)
- ✅ ALPHA status disclosure (was claimed "production-ready")
- ✅ Equal weighting default (DeMiguel 2009 research-backed)
- ✅ Friction-adjusted accuracy (paper 70-78%, live 65-73%)

**Recommended Next Steps:**
1. Execute 60-day forward test on SPY daily
2. Calculate empirical accuracy per algorithm
3. Apply Breiman log-odds for optimal weights
4. Upgrade to BETA after validation

---

## 2. Institutional Activity Algorithms (WHEN to trade)

### institutional-algo1-volume-efficiency.pine
**Method:** Volume efficiency + price absorption
**Technique:** Volume/Range ratio → detect accumulation/distribution
**Outputs:** Pattern type (ACCUM/DISTR/MIXED), strength 0-100

**Strengths:**
- 5 pattern types (graduated trend filtering)
- Regime-adaptive scoring
- Clear accumulation vs distribution classification

**Limitations:**
- Lagging (relies on completed bars)
- Trend filter can miss early reversals

---

### institutional-algo2-mtf-convergence.pine
**Method:** Multi-timeframe volume convergence
**Technique:** Detect volume spikes across 3 timeframes
**Outputs:** Campaign type (Building/Mature), confidence

**Strengths:**
- Detects institutional campaigns early
- Building vs mature classification
- Adaptive thresholds by regime

**Limitations:**
- Requires volume data (not all assets)
- Higher TF lag compounds detection delay

---

### institutional-algo3-bayesian-regime.pine
**Method:** Bayesian probabilistic classification
**Technique:** Prior probabilities + regime-specific likelihoods
**Outputs:** 5-state regime classification, probability scores

**Strengths:**
- Intrabar analysis for confirmation
- Probabilistic output (not binary)
- Regime-specific likelihood functions

**Limitations:**
- Computationally intensive
- Requires parameter tuning per asset

---

### institutional-algo4-ensemble.pine
**Method:** Weighted voting from 3 institutional algorithms
**Weights:** Algo1(25%), Algo2(35%), Algo3(40%)
**Outputs:** Consensus pattern, agreement 1/3 to 3/3

**Strengths:**
- Pattern consensus voting
- Agreement-based confidence
- Customizable weights

**Limitations:**
- Contains duplicated code from all 3 algorithms
- Highest maintenance burden

---

## 3. Trading Strategy Indicators (MKN Series)

### mkn-gaps.pine (165 LOC) - PineScript v6
**Method:** Body-only gap detection and tracking
**Technique:** Detects gaps between candle bodies (open/close ranges), ignoring wicks
**Outputs:** Gap boxes with partial/full close tracking, alerts
**Philosophy:** "Where price committed" vs temporary wick probes

**Strengths:**
- Body-only logic finds MORE gaps than traditional high/low detection
- Matches visual analysis when wicks disabled
- Partial vs full gap closing options
- Dynamic gap tracking with configurable limits (max 500 boxes)
- ATR-relative minimum gap size (30% of 14-bar ATR by default)
- Fading for closed gaps (visual clarity)
- Alert system (new gap, gap closed)

**Limitations:**
- No gap quality scoring (all gaps treated equally)
- No timeframe analysis (doesn't distinguish intraday vs overnight gaps)
- No volume integration (gap + volume spike = stronger)
- Box limit may truncate older gaps on high-volatility charts

**Optimization Potential:**
- Gap quality scoring (volume, timeframe, % size relative to ATR)
- Session-based gaps (RTH vs ETH vs overnight for futures/stocks)
- Volume confirmation (require volume spike on gap-up/down)
- Integration with sr-algo1 (gaps at HVN = institutional interest)
- Fill probability estimation (% of gaps filled within N bars)
- Multi-timeframe gap confluence (daily gap + 4H gap = stronger)

**Use Cases:**
- Gap fade strategies (trade into gap fill)
- Breakaway gap identification (unfilled gaps = trend continuation)
- Exhaustion gap detection (late-trend gaps that fill quickly)
- Overnight risk assessment (gap size = volatility measure)

**Notable Implementation:**
- Uses PineScript v6 (newer than other indicators)
- Clean type system (`Gap` and `AlertInfo` types)
- Efficient box management (deletes oldest when limit exceeded)
- Body-only logic: `bodyTop = math.max(open, close)`, `bodyBottom = math.min(open, close)`

---

### mkn-hma-bands.pine (181 LOC) - PineScript v5
**Method:** Hull Moving Average bands with volume-confirmed exits
**Technique:** Fast HMA (55) + Slow HMA (89) + Bollinger Bands around Fast HMA
**Outputs:** Buy/sell signals when price crosses bands with volume confirmation
**Philosophy:** Exit strategy indicator (not entry), volume validation essential

**Strengths:**
- Hull MA = reduced lag vs traditional MA (sqrt weighting)
- Dual HMA system (55/89) provides trend bias
- Bollinger Bands around Fast HMA = volatility-adaptive zones
- Volume spike confirmation (150% minimum, 200% strong)
- Band expansion/contraction detection (trending vs ranging)
- Visual hierarchy: gradient fills, bar coloring, signal shapes
- Real-time info table (bias, volume ratio, band state, price position)
- Yellow bar coloring for first entry into HMA band with volume (momentum shift detection)

**Limitations:**
- No regime filtering (High Vol = less reliable, not disclosed)
- Fixed HMA periods (55/89) may not suit all assets/timeframes
- Volume spike thresholds (1.5x, 2.0x) are asset-agnostic
- No integration with S/R levels (exits at arbitrary bands vs key levels)
- Lagging (HMA still lags despite sqrt weighting)

**Optimization Potential:**
- Regime-adaptive parameters (High Vol: wider bands, higher volume threshold)
- Asset-specific HMA periods (crypto 24/7 vs stock RTH)
- Volume spike normalization by regime (1.5x in Low Vol = 2.5x in High Vol equivalent)
- Integration with sr-algo1/2/3 (exit at band + S/R confluence)
- Multi-timeframe HMA alignment (require 2/3 TFs bullish/bearish for signal)
- Position sizing based on band width (wider = reduce size)
- Divergence detection (price new high but HMA lagging = weakening trend)

**Use Cases:**
- Exit strategy for mean reversion trades (price at band + volume = take profit)
- Trend bias confirmation (HMA 55 > HMA 89 = bullish bias)
- Momentum shift detection (first bar in HMA band with volume = early reversal)
- Volatility regime classification (expanding bands = trending, contracting = ranging)

**Notable Implementation:**
- Custom HMA function: `f_hma(src, length)` with WMA composition
- Band expansion detection: `isExpanding = bandWidth > ta.sma(bandWidth, 20)`
- Volume ratio: `volRatio = volume / ta.sma(volume, 20)`
- Fancy gradient colors (#00D9FF cyan, #B57EDC lavender, #00FFA3 mint green)
- First bar in band detection: `isFirstBarInBand = insideHMABand and wasOutsideHMABand`
- Info table with real-time metrics

**Signal Logic:**
- **Sell Signal:** `close > upperBand AND volRatio >= 1.5x`
- **Buy Signal:** `close < lowerBand AND volRatio >= 1.5x`
- **Strong Sell/Buy:** Same but `volRatio >= 2.0x`
- **HMA Crossovers:** Fast crosses Slow (trend change, not exit)

**Integration Opportunity:**
This indicator SHOULD be combined with sr-algo5-ensemble:
- MKN-HMA-Bands signals exit timing
- sr-algo5-ensemble provides WHERE (S/R levels)
- Combined strategy: Exit at HMA band + S/R confluence + volume spike = high-probability exit

---

## 4. Shared Libraries (Published to TradingView)

### lib_core_math.pine
**Provides:** Safe division, safe range, array bounds checking
**Key Functions:**
- `safeDivide(num, denom, default)` - NA protection
- `safeRange(range, close, minPercent)` - Prevent division by tiny ranges
- `safeArrayGet(arr, index, default)` - Bounds checking

**Used By:** All 11 indicators
**Optimization:** None needed (fundamental utilities)

---

### lib_regime_detection.pine
**Provides:** ATR-based volatility regime classification
**Key Functions:**
- `detectRegime(atrLen, lookback, thresholds)` - Returns regime data
- Output: name, atrRatio, multiplier, boolean flags

**Standard Parameters:**
- ATR Length: 14
- High Vol Threshold: 1.3 (30% above average)
- Low Vol Threshold: 0.7 (30% below average)
- Multipliers: Low Vol +20%, High Vol -20%

**Used By:** 9 indicators (S/R + Institutional, not MKN series which use simpler approaches)
**Optimization:** Consider regime-specific parameter sets (e.g., 10 ATR for crypto, 20 for equities)

---

### lib_volume_analysis.pine
**Provides:** Volume metrics calculations
**Key Functions:**
- `calculateVolumeMetrics(lookback)` - Returns VolumeMetrics type
- Fields: relativeVolume, volumeEfficiency, efficiencyRatio, volumeScore

**Used By:** 7 indicators (all institutional + sr-algo2, sr-algo4)
**Optimization:** Add volume profile binning (move from algo1 to library for reuse)

---

### lib_level_utils.pine
**Provides:** Distance calculations, touch detection, rejection analysis
**Key Functions:**
- `calculateDistance(levelPrice, currentPrice)` - % distance
- `isTouching(level, high, low, tolerance)` - Touch detection
- `analyzeRejections(level, bars, tolerance)` - Rejection strength

**Used By:** All 5 S/R algorithms
**Optimization:** Add multi-bar rejection sequences, volume-weighted rejection scoring

---

### lib_mtf_utils.pine
**Provides:** Automatic higher timeframe calculation
**Key Functions:**
- `getHigherTimeframe(baseTF)` - Standard progression (5→15→60→240→D→W)
- `getHigherTimeframeWide(baseTF)` - Wider spacing (5→30→240→D→W)

**Used By:** sr-algo3, sr-algo5, institutional-algo2
**Optimization:** Add custom TF ladder (e.g., crypto 24/7 vs stock RTH)

---

### lib_trend_detection.pine
**Provides:** Trend classification (uptrend, downtrend, ranging)
**Key Functions:**
- `detectTrend(maShort, maLong, atrMultiplier)` - Returns trend state

**Used By:** institutional-algo1
**Optimization:** Add multi-timeframe trend alignment (trend on 3 TFs = stronger)

---

## 5. Key Limitations Across All Algorithms

### PineScript Platform Constraints:
1. **No historical backtesting:** Cannot validate past performance (security model)
2. **No ML capabilities:** Cannot implement Algorithm 5 (ML classifier)
3. **No library imports in indicators:** Must duplicate code (TECH-DEBT.md tracks this)
4. **500-bar memory limit:** Restricts lookback periods
5. **Array/label limits:** max_lines_count=100, max_labels_count=30

### Data Limitations:
1. **OHLCV only:** No tick data, order book, or bid/ask spread
2. **Accuracy ceiling:** 70-75% realistic max for OHLCV-only methods
3. **Friction penalties:** Live trading 5-7% lower than paper (slippage, spreads)
4. **Regime dependency:** -5% to -10% without ML regime adaptation

### Detection Lag:
1. **Pivot detection:** 10-bar confirmation (40 hours on 4H chart)
2. **Multi-TF lag:** Compounds across timeframes (192 hours 1H→Daily)
3. **Regime detection:** 7-14 bars to confirm volatility shift

---

## 6. Optimization Opportunities

### High-Impact (Estimated +10-15% Accuracy):
1. **Fractal detection for S/R:** Replace pivots (10-bar lag → 5-bar lag)
2. **Empirical ensemble weights:** 60-day forward test → Breiman log-odds
3. **Volume profile in algo2/3/4:** Cross-indicator `input.source()` integration
4. **Adaptive lookbacks:** Regime-dependent (200-1000 bars)

### Medium-Impact (Estimated +5-10% Accuracy):
1. **Multi-bar rejection sequences:** 3+ bars rejecting level (vs single bar)
2. **Exponential distance decay:** Replace linear penalty in algo2
3. **Session-based volume profiling:** RTH vs ETH for futures
4. **Spatial hashing for merges:** O(n³) → O(n log n) in algo5

### Low-Impact (Estimated +2-5% Accuracy):
1. **Regime-filtered labels:** "High Vol - Use Caution" visual cues
2. **Multi-TF trend alignment:** Require 2/3 TFs in same trend
3. **Custom TF ladders:** Crypto 24/7 vs stock RTH-specific progressions
4. **Automated performance tracking:** CSV export for empirical validation

---

## 7. Reusability Matrix

| Component | Currently Used By | Reusable For | Modification Needed |
|-----------|------------------|--------------|---------------------|
| **Regime Detection** | 9 indicators (S/R + Inst) | Any indicator | Asset-specific thresholds |
| **Volume Analysis** | 8 indicators (7 S/R+Inst, 1 MKN) | All strategies | Add volume profile binning |
| **Safe Math** | All 11 indicators | All future work | None (stable) |
| **MTF Utils** | 3 indicators | Cross-TF analysis | Custom TF ladders |
| **Level Utils** | 5 S/R algos | Price level work | Multi-bar sequences |
| **Trend Detection** | 1 indicator | Directional filters | MTF trend alignment |

### Duplication Hotspots (TECH-DEBT.md):
- **Regime detection:** Duplicated in 9 files (standard params enforce consistency)
- **Volume calcs:** Duplicated in 7 files (library exists but early indicators predate it)
- **Ensemble cores:** algo4 & algo5 contain full algorithm logic (highest maintenance burden)

---

## 8. Research Collaboration Recommendations

### For S/R Algorithm Validation:
1. **Forward Testing Protocol:** 60-day SPY daily with CSV logging
2. **Metrics to Track:** Touch accuracy, rejection accuracy, friction penalty
3. **Baseline Comparison:** Random levels, simple MA S/R, pivot points

### For Institutional Detection:
1. **Pattern Validation:** Accumulation/distribution vs subsequent price movement
2. **Lead Time Analysis:** How many bars before breakout/breakdown
3. **False Positive Rate:** Signals without follow-through

### For Ensemble Optimization:
1. **Weight Optimization:** Breiman log-odds, DeMiguel equal weights, grid search
2. **Agreement Threshold:** Test 2/4 vs 3/4 minimum for signal validity
3. **Regime Filtering:** Compare accuracy in High/Low/Normal vol regimes

### For Library Enhancements:
1. **Volume Profile Binning:** Move from algo1 to lib_volume_analysis
2. **Multi-Bar Patterns:** Add to lib_level_utils (rejection sequences)
3. **Fractal Detection:** Add to new lib_pattern_detection (replace pivots)

### For MKN Trading Strategy Integration:
1. **Gap Quality Scoring:** Validate if gaps at HVN levels (sr-algo1) fill slower/faster
2. **HMA-Band + S/R Confluence:** Test exit accuracy when HMA band aligns with S/R level
3. **Volume Integration:** Compare MKN volume thresholds (1.5x/2.0x) to lib_volume_analysis metrics
4. **Session Analysis:** Test overnight gaps vs intraday gaps (futures/stocks)

---

## 9. Critical Issues for External Review

### sr-algo5-ensemble.pine (ALPHA):
- **Status:** Unvalidated (claimed "production" in v1.0, downgraded to ALPHA in v1.1)
- **Weight Justification:** Original weights (35/30/20/15) have NO research citation
- **Contradiction:** MTF highest accuracy (80-90%) but middle weight (30%)
- **Recommendation:** Use equal weights (DEFAULT) until empirical validation

### sr-algo2-statistical-peaks.pine:
- **v1.0 Bugs:** Temporal decay wrong (0.992 vs 0.9942), confluence additive vs multiplicative
- **v1.1 Status:** All critical bugs fixed, production-ready (8/10 quality)
- **Remaining Risk:** Pivot lag (10 bars) causes 40-hour delay on 4H charts

### sr-algo3-mtf-confluence.pine:
- **Accuracy Claims:** Downgraded from 80-90% → 65-70% (realistic OHLCV ceiling)
- **Lag Issue:** Multi-TF pivot lag = 192 hours (1H→Daily), 56 days (Daily→Weekly)
- **Validation Gap:** Cannot backtest historically (TradingView constraint)

### All S/R & Institutional Indicators:
- **No Empirical Testing:** All accuracy claims are research-based estimates, not validated
- **Friction Penalty:** Live trading 5-7% lower than paper (must disclose to users)
- **Code Duplication:** 9 files with regime detection, 7 with volume calcs (maintenance burden)

### MKN Trading Indicators:
- **mkn-gaps.pine:** Production-ready (gap detection is binary, no accuracy metrics)
- **mkn-hma-bands.pine:** Production-ready but could benefit from regime filtering
- **Integration Opportunity:** Should be tested in combination with sr-algo5-ensemble
- **No Critical Issues:** Strategy tools vs detectors (different validation requirements)

---

## 10. Immediate Action Items for Production Use

**Before Live Trading:**
1. Execute 60-day forward test (SPY daily, log all signals)
2. Calculate empirical accuracy per algorithm
3. Measure friction penalty on real broker (compare paper vs live fills)
4. Update ensemble weights based on actual performance

**Documentation Updates:**
1. Add "ALPHA" status to sr-algo5 until validated
2. Disclose pivot lag in sr-algo2/3 documentation
3. Separate paper vs live accuracy estimates
4. Add regime-specific performance warnings

**Code Improvements:**
1. Implement fractal detection (reduce pivot lag)
2. Add volume profile integration to algo2/3/4 via `input.source()`
3. Refactor ensemble cores (reduce duplication burden)
4. Add automated performance tracking (CSV export)

**For MKN Indicators:**
1. Add regime filtering to mkn-hma-bands (High Vol = wider bands, higher volume threshold)
2. Add gap quality scoring to mkn-gaps (volume, timeframe, ATR-relative size)
3. Create combined strategy: MKN-HMA-Bands + sr-algo5-ensemble for exit optimization
4. Test session-based gap analysis (overnight vs intraday for futures/stocks)

---

## Summary Statistics

| Metric | Value |
|--------|-------|
| **Total Indicators** | 11 (5 S/R + 4 Institutional + 2 MKN Trading) |
| **Total Libraries** | 6 (shared utilities) |
| **Lines of Code** | ~4,500+ PineScript v5/v6 |
| **Claimed Accuracy** | 65-90% for S/R (unvalidated) |
| **Realistic Accuracy** | 60-75% (OHLCV ceiling for S/R) |
| **Live Trading Penalty** | -5% to -7% (friction) |
| **Code Quality** | 6-8/10 (bugs fixed in v1.1) |
| **Validation Status** | 0/10 (ALPHA - no testing) |
| **Maintenance Burden** | High (9 regime dupes, 2 ensemble cores) |
| **MKN Indicators** | No accuracy claims (strategy tools, not detectors) |

**Production Readiness:**
- S/R Algorithms: Code 8/10, Validation 0/10 → Requires 60-day testing
- MKN Indicators: Production-ready (exit/gap tools, no accuracy metrics required)

---

## Contact & Next Steps

**For collaboration on:**
- Empirical validation protocols
- Weight optimization research
- Friction penalty quantification
- ML-based regime classification (outside PineScript)

**Repository:** PineScript project (TradingView platform)
**Documentation:** See `docs/algos/` for detailed algorithm specifications
**Audit Trail:** See `CLAUDE.md` for full development history and audit responses
