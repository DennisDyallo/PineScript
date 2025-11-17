# PineScript Architecture Guide: Data Sharing & Code Reuse
**Version:** 1.0  
**Date:** November 17, 2025  
**Purpose:** Reference guide for AI LLM developers building modular PineScript indicator systems

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [The Two Mechanisms](#the-two-mechanisms)
3. [Decision Framework](#decision-framework)
4. [Architecture Patterns](#architecture-patterns)
5. [Implementation Guide: Libraries](#implementation-guide-libraries)
6. [Implementation Guide: Data Bridge](#implementation-guide-data-bridge)
7. [Hybrid Architecture](#hybrid-architecture)
8. [Best Practices](#best-practices)
9. [Common Pitfalls](#common-pitfalls)
10. [Testing & Validation](#testing--validation)

---

## Executive Summary

### The Problem
When building multiple related indicators in PineScript, you face three challenges:
1. **Code Duplication:** Same calculation logic repeated across indicators
2. **Data Sharing:** Need to combine results from multiple indicators
3. **Maintenance:** Bug fixes and updates must propagate to all indicators

### The Solution
PineScript provides two mechanisms that serve different purposes:

| Mechanism | Purpose | Use Case |
|-----------|---------|----------|
| **Library** | Share CODE (functions, types) | Reusable calculation logic you control |
| **Data Bridge** | Share DATA (calculated values) | Combine results from any indicator |

### Quick Decision Guide

**Use Library if:**
- ✅ You're writing all indicators from scratch
- ✅ Need version control for shared code
- ✅ Want professional code organization
- ✅ Performance matters (shared compiled code)

**Use Data Bridge if:**
- ✅ Combining third-party indicators
- ✅ Users need flexibility to choose data sources
- ✅ Don't want to publish library publicly
- ✅ Rapid prototyping of combinations

**Use Both if:**
- ✅ Building a professional indicator suite
- ✅ Want modular architecture with maximum flexibility

---

## The Two Mechanisms

### Library: Code Sharing

**What It Does:** Allows you to write reusable functions and types that other scripts can import

**Analogy:** Like a Python library or npm package

**How It Works:**
```pinescript
// ===== FILE: my_library.pine (published as library) =====
//@version=5
library("my_library")

export calculateSomething(simple int param) =>
    // Your calculation logic
    result = param * 2.0
    result

export type CustomData
    float value
    string label
    int count

// ===== FILE: my_indicator.pine (uses library) =====
//@version=5
indicator("My Indicator", overlay=true)

import username/my_library/1 as lib

result = lib.calculateSomething(20)
plot(result)
```

**Key Characteristics:**
- Must be published to TradingView (public, invite-only, or private)
- Version controlled (`/1`, `/2`, `/3`)
- Shares **compilation-ready code**
- Cannot share running indicator data
- Zero setup time (import in code)

---

### Data Bridge: Data Sharing via `input.source()`

**What It Does:** Allows one indicator to consume data values calculated by another indicator

**Analogy:** Like Unix pipes or message passing between processes

**How It Works:**
```pinescript
// ===== PRODUCER INDICATOR =====
//@version=5
indicator("Data Producer", overlay=true)

calculatedValue = ta.sma(close, 20)

// Export via invisible plot
plot(calculatedValue, "Exported_Value", 
    color=color.new(color.blue, 100), 
    display=display.none)  // Invisible but accessible

// ===== CONSUMER INDICATOR =====
//@version=5
indicator("Data Consumer", overlay=true)

// Import the calculated value
importedValue = input.source(close, "Select Data Source",
    tooltip="Choose 'Data Producer: Exported_Value' from dropdown")

// Use the imported value
plot(importedValue * 2, "Doubled Value")
```

**Key Characteristics:**
- Works with ANY indicator (yours or third-party)
- Requires manual UI setup (dropdown selection)
- Shares **live calculated values**
- No publishing required
- Each indicator calculates independently (no shared computation)

---

## Decision Framework

### Step-by-Step Decision Tree

```
START: Need to share something between indicators?
│
├─ Q1: Is it CALCULATION LOGIC or CALCULATED DATA?
│  │
│  ├─ LOGIC (functions, algorithms, types)
│  │  └─> Use LIBRARY
│  │
│  └─ DATA (results, values, signals)
│     └─> Continue to Q2
│
├─ Q2: Do you control ALL the indicators?
│  │
│  ├─ YES (you're writing everything)
│  │  └─> Use LIBRARY (share code) + optionally export data
│  │
│  └─ NO (combining with third-party indicators)
│     └─> Use DATA BRIDGE (only option)
│
├─ Q3: Do users need flexibility in data sources?
│  │
│  ├─ YES (users pick which indicator to combine)
│  │  └─> Use DATA BRIDGE
│  │
│  └─ NO (fixed architecture)
│     └─> Use LIBRARY
│
└─ Q4: Is performance critical?
   │
   ├─ YES (complex calculations on every bar)
   │  └─> Use LIBRARY (shared compiled code)
   │
   └─ NO (simple calculations or low-frequency updates)
      └─> Either approach works
```

### Comparison Matrix

| Criteria | Library | Data Bridge | Both |
|----------|---------|-------------|------|
| **Code Reuse** | ⭐⭐⭐⭐⭐ Perfect | ❌ None | ⭐⭐⭐⭐⭐ Perfect |
| **Data Sharing** | ❌ Functions only | ⭐⭐⭐⭐⭐ Real-time | ⭐⭐⭐⭐⭐ Best of both |
| **Third-Party Integration** | ❌ Can't import | ⭐⭐⭐⭐⭐ Any indicator | ⭐⭐⭐⭐⭐ Flexible |
| **Setup Time** | ⭐⭐⭐⭐⭐ Instant | ⭐⭐ Manual UI | ⭐⭐⭐ One-time UI setup |
| **Maintenance** | ⭐⭐⭐⭐⭐ Single source | ⭐ Update each | ⭐⭐⭐⭐ Library + exports |
| **Version Control** | ⭐⭐⭐⭐⭐ Built-in | ❌ None | ⭐⭐⭐⭐⭐ Library versioned |
| **Performance** | ⭐⭐⭐⭐ Shared code | ⭐⭐⭐ Independent calcs | ⭐⭐⭐⭐ Library optimized |
| **Flexibility** | ⭐⭐ Fixed functions | ⭐⭐⭐⭐⭐ User chooses | ⭐⭐⭐⭐⭐ Maximum |
| **Publishing Required** | ⚠️ Yes | ❌ No | ⚠️ Library only |
| **Learning Curve** | ⭐⭐⭐ Moderate | ⭐⭐⭐⭐ Easy | ⭐⭐ Steeper |

---

## Architecture Patterns

### Pattern 1: Pure Library (Code Reuse Only)

**Use When:** Building a suite of related indicators where you control all code

**Structure:**
```
project/
├── core_library.pine          (published library)
│   ├── Calculation functions
│   ├── Utility functions
│   └── Shared types
│
├── indicator_a.pine           (imports library)
├── indicator_b.pine           (imports library)
└── indicator_c.pine           (imports library)
```

**Advantages:**
- ✅ Zero code duplication
- ✅ Single source of truth for calculations
- ✅ Version control for shared code
- ✅ Best performance (shared compiled code)

**Disadvantages:**
- ❌ Must publish library (even if private)
- ❌ Cannot combine with third-party indicators
- ❌ Users can't mix and match data sources

**Code Volume Reduction:** 60-80%

---

### Pattern 2: Pure Data Bridge (Data Sharing Only)

**Use When:** Combining existing indicators or providing maximum user flexibility

**Structure:**
```
chart/
├── indicator_x.pine           (standalone, exports data)
├── indicator_y.pine           (standalone, exports data)
├── indicator_z.pine           (standalone, exports data)
└── ensemble_indicator.pine    (imports via input.source)
```

**Advantages:**
- ✅ Works with ANY indicator (third-party included)
- ✅ Users choose data sources flexibly
- ✅ No publishing required
- ✅ Rapid prototyping

**Disadvantages:**
- ❌ Code duplication if building from scratch
- ❌ Manual UI setup required
- ❌ No version control
- ❌ Independent calculations (potential performance hit)

**Setup Time:** 2-5 minutes per connection

---

### Pattern 3: Hybrid (Library + Data Bridge)

**Use When:** Building professional indicator suite with maximum flexibility

**Structure:**
```
project/
├── core_library.pine          (published library)
│   ├── Calculation functions
│   ├── Utility functions
│   └── Shared types
│
├── specialized_indicators/
│   ├── indicator_a.pine       (imports library, exports data)
│   ├── indicator_b.pine       (imports library, exports data)
│   └── indicator_c.pine       (imports library, exports data)
│
└── ensemble/
    └── master_indicator.pine  (imports library + data bridge)
```

**Advantages:**
- ✅ Zero code duplication (library)
- ✅ Can combine with third-party indicators (data bridge)
- ✅ Version control for core logic (library)
- ✅ User flexibility in data sources (data bridge)
- ✅ Best of both worlds

**Disadvantages:**
- ⚠️ More complex architecture
- ⚠️ Library must be published
- ⚠️ Requires understanding both mechanisms

**Code Volume Reduction:** 70-85% (library) + maximum flexibility (data bridge)

---

### Pattern 4: Data Bridge Hub (Centralized Export)

**Use When:** Many consumers need same data from multiple sources

**Structure:**
```
chart/
├── data_hub.pine              (imports from sources, exports combined)
│   ├── Imports from indicator_a
│   ├── Imports from indicator_b
│   ├── Combines/validates data
│   └── Exports unified data
│
├── consumer_1.pine            (imports from hub)
├── consumer_2.pine            (imports from hub)
└── consumer_3.pine            (imports from hub)
```

**Advantages:**
- ✅ Single point of connection management
- ✅ Data validation/transformation layer
- ✅ Easier to maintain connections
- ✅ Can add metadata (confidence scores, etc.)

**Disadvantages:**
- ❌ Extra indicator on chart
- ❌ Additional layer of indirection
- ❌ Hub becomes single point of failure

**Best For:** Complex systems with 5+ indicators

---

## Implementation Guide: Libraries

### Step 1: Create Library Structure

**File: `core_library.pine`**

```pinescript
//@version=5
// @description General-purpose library for [your indicator suite name]
// @version 1.0.0
library("library_name")

// ============================================================================
// EXPORTED FUNCTIONS
// ============================================================================

// @function Brief description of what this calculates
// @param paramName Description of parameter
// @returns Description of return value
export functionName(simple int param1, float param2 = 10.0) =>
    // Your calculation logic here
    result = param1 * param2
    result

// @function Another calculation function
// @param lookback Number of bars to analyze
// @returns Tuple of multiple values
export anotherFunction(simple int lookback = 50) =>
    value1 = ta.sma(close, lookback)
    value2 = ta.stdev(close, lookback)
    value3 = value1 + value2
    
    [value1, value2, value3]

// ============================================================================
// EXPORTED TYPES
// ============================================================================

// @type Description of this data structure
export type CustomDataType
    float price
    string category
    int count
    bool isValid

// @type Another data structure
export type ConfigType
    int lookback
    float threshold
    string mode

// ============================================================================
// UTILITY FUNCTIONS
// ============================================================================

// @function Helper function for internal use or export
export utilityFunction(float value, float threshold) =>
    value > threshold ? "HIGH" : value < -threshold ? "LOW" : "NORMAL"
```

### Step 2: Publish Library

**Publishing Checklist:**
1. ✅ All functions have `export` keyword
2. ✅ All functions have JSDoc comments (`// @function`, `// @param`, `// @returns`)
3. ✅ Library has description (`// @description`)
4. ✅ Version specified (`// @version`)
5. ✅ Code compiles without errors

**Publishing Steps:**
1. Open Pine Editor
2. Load library file
3. Click "Publish Script" button (top-right)
4. Select "Library" (not Indicator or Strategy)
5. Choose visibility:
   - **Public:** Anyone can use (shows in community scripts)
   - **Invite-Only:** Requires your approval
   - **Private:** Only you can use
6. Set category and tags
7. Click "Publish"

**Result:** Library accessible at `username/library_name/1`

---

### Step 3: Use Library in Indicators

**File: `my_indicator.pine`**

```pinescript
//@version=5
indicator("My Indicator", overlay=true)

// ===== IMPORT LIBRARY =====
// Format: import username/library_name/version as alias
import yourname/library_name/1 as lib

// Can also import specific version
// import yourname/library_name/2 as lib  // Use version 2

// ===== USE LIBRARY FUNCTIONS =====

// Simple function call
result = lib.functionName(20, 15.5)
plot(result, "Result")

// Function returning tuple
[val1, val2, val3] = lib.anotherFunction(50)
plot(val1, "Value 1", color.blue)
plot(val2, "Value 2", color.red)

// Using library types
config = lib.ConfigType.new(
    lookback = 100,
    threshold = 2.0,
    mode = "NORMAL"
)

data = lib.CustomDataType.new(
    price = close,
    category = lib.utilityFunction(close, 100.0),
    count = 1,
    isValid = true
)

// Access type fields
if data.isValid and data.category == "HIGH"
    label.new(bar_index, data.price, "High Signal")
```

### Step 4: Version Management

**When to Increment Version:**

**Major Version (1.0 → 2.0):**
- Breaking changes to function signatures
- Removed functions
- Changed return types

**Minor Version (1.0 → 1.1):**
- New functions added
- New optional parameters
- Bug fixes that change behavior

**Patch Version (1.0.0 → 1.0.1):**
- Bug fixes with no behavior change
- Documentation updates
- Performance optimizations

**Example:**
```pinescript
// Version 1
export calculate(int period) => ta.sma(close, period)

// Version 2 (breaking change - added required parameter)
export calculate(int period, string type) => 
    type == "SMA" ? ta.sma(close, period) : ta.ema(close, period)

// Better approach (non-breaking)
export calculate(int period, string type = "SMA") =>
    type == "SMA" ? ta.sma(close, period) : ta.ema(close, period)
```

**In indicators, specify version:**
```pinescript
import yourname/library_name/1 as lib_v1  // Use stable version 1
import yourname/library_name/2 as lib_v2  // Test new version 2
```

---

## Implementation Guide: Data Bridge

### Step 1: Producer Indicator (Export Data)

**Pattern: Export via Invisible Plots**

```pinescript
//@version=5
indicator("Data Producer", overlay=true)

// ===== CALCULATE YOUR VALUES =====
calculatedValue1 = ta.sma(close, 20)
calculatedValue2 = ta.stdev(close, 20)
calculatedValue3 = calculatedValue1 + (calculatedValue2 * 2)

// Additional metadata
confidence = calculatedValue2 < 10 ? 0.9 : 0.5
regime = calculatedValue2 > 20 ? "HIGH_VOL" : "LOW_VOL"

// Encode string as number for export (PineScript limitation)
regimeEncoded = regime == "HIGH_VOL" ? 2.0 : regime == "LOW_VOL" ? 1.0 : 0.0

// ===== EXPORT VIA INVISIBLE PLOTS =====
// Key: display=display.none makes it invisible but accessible
plot(calculatedValue1, title="Export_Value1", 
    color=color.new(color.blue, 100), 
    display=display.none)

plot(calculatedValue2, title="Export_Value2",
    color=color.new(color.red, 100),
    display=display.none)

plot(calculatedValue3, title="Export_Value3",
    color=color.new(color.green, 100),
    display=display.none)

plot(confidence, title="Export_Confidence",
    color=color.new(color.orange, 100),
    display=display.none)

plot(regimeEncoded, title="Export_Regime",
    color=color.new(color.purple, 100),
    display=display.none)

// ===== OPTIONAL: VISUALIZE SOME DATA =====
plot(calculatedValue1, title="SMA 20 (visible)", color=color.blue, linewidth=2)
plot(calculatedValue3, title="Upper Band (visible)", color=color.green)

// ===== OPTIONAL: INFO TABLE =====
var table info = table.new(position.bottom_right, 2, 3)
if barstate.islast
    table.cell(info, 0, 0, "Data Producer Active", bgcolor=color.new(color.blue, 70))
    table.merge_cells(info, 0, 0, 1, 0)
    
    table.cell(info, 0, 1, "Value1:")
    table.cell(info, 1, 1, str.tostring(calculatedValue1, format.mintick))
    
    table.cell(info, 0, 2, "Regime:")
    table.cell(info, 1, 2, regime, bgcolor=regimeEncoded > 1.5 ? color.red : color.green)
```

**Naming Convention for Exports:**
- Use prefix like `Export_` to distinguish from visual plots
- Use descriptive names: `Export_SMA20`, `Export_UpperBand`, `Export_Confidence`
- Document what each export represents in tooltips or table

---

### Step 2: Consumer Indicator (Import Data)

**Pattern: Input Sources with Defaults**

```pinescript
//@version=5
indicator("Data Consumer", overlay=true)

// ===== IMPORT DATA VIA INPUT.SOURCE =====
// Default to 'close' so indicator works without setup
importedValue1 = input.source(close, title="Value 1 Source",
    group="Data Sources",
    tooltip="Select 'Data Producer: Export_Value1' from dropdown")

importedValue2 = input.source(close, title="Value 2 Source",
    group="Data Sources",
    tooltip="Select 'Data Producer: Export_Value2'")

importedValue3 = input.source(close, title="Value 3 Source",
    group="Data Sources",
    tooltip="Select 'Data Producer: Export_Value3'")

importedConfidence = input.source(1.0, title="Confidence Source",
    group="Data Sources",
    tooltip="Select 'Data Producer: Export_Confidence'. Default=1.0 (100%)")

importedRegime = input.source(0.0, title="Regime Source",
    group="Data Sources",
    tooltip="Select 'Data Producer: Export_Regime'")

// ===== DECODE IMPORTED DATA =====
// Convert encoded regime back to string
regimeDecoded = importedRegime > 1.5 ? "HIGH_VOL" : 
                importedRegime > 0.5 ? "LOW_VOL" : 
                "UNKNOWN"

// ===== USE IMPORTED DATA =====
// Example: Calculate composite signal
compositeSignal = (importedValue1 + importedValue3) / 2

// Adjust by confidence
adjustedSignal = compositeSignal * importedConfidence

// Different behavior based on regime
threshold = regimeDecoded == "HIGH_VOL" ? 15.0 : 10.0

// ===== GENERATE SIGNALS =====
buySignal = ta.crossover(adjustedSignal, threshold)
sellSignal = ta.crossunder(adjustedSignal, threshold)

// ===== VISUALIZE =====
plot(compositeSignal, "Composite Signal", color.blue)
plot(adjustedSignal, "Adjusted Signal", color.orange)
hline(threshold, "Threshold", color.gray, linestyle=hline.style_dashed)

plotshape(buySignal, "Buy", shape.triangleup, location.belowbar, color.green, size=size.small)
plotshape(sellSignal, "Sell", shape.triangledown, location.abovebar, color.red, size=size.small)

// ===== CONNECTION STATUS TABLE =====
var table status = table.new(position.top_right, 2, 4)
if barstate.islast
    // Check if user connected data sources (heuristic: not equal to default)
    connected1 = importedValue1 != close
    connected2 = importedValue2 != close
    connected3 = importedValue3 != close
    
    table.cell(status, 0, 0, "Data Bridge Status", bgcolor=color.new(color.blue, 70))
    table.merge_cells(status, 0, 0, 1, 0)
    
    table.cell(status, 0, 1, "Value 1:")
    table.cell(status, 1, 1, connected1 ? "✓ Connected" : "⚠ Not Connected",
        bgcolor=connected1 ? color.green : color.red)
    
    table.cell(status, 0, 2, "Regime:")
    table.cell(status, 1, 2, regimeDecoded)
    
    table.cell(status, 0, 3, "Confidence:")
    table.cell(status, 1, 3, str.tostring(importedConfidence * 100, "#") + "%")
```

---

### Step 3: User Setup Instructions

**Document this in indicator description:**

```markdown
## Setup Instructions

This indicator requires data from "Data Producer" indicator.

### Step 1: Add Data Producer
1. Add indicator: "Data Producer" to your chart
2. Wait for it to load completely

### Step 2: Connect Data Sources
1. Open settings for "Data Consumer" (this indicator)
2. Go to "Data Sources" section
3. For each dropdown:
   - Click dropdown arrow
   - Scroll to find "Data Producer: Export_XXX"
   - Select the corresponding export
4. Click "OK"

### What to Select:
- **Value 1 Source**: Select "Data Producer: Export_Value1"
- **Value 2 Source**: Select "Data Producer: Export_Value2"
- **Value 3 Source**: Select "Data Producer: Export_Value3"
- **Confidence Source**: Select "Data Producer: Export_Confidence"
- **Regime Source**: Select "Data Producer: Export_Regime"

### Verification:
Check the "Data Bridge Status" table in the top-right corner.
All connections should show "✓ Connected".
```

---

### Step 4: Data Bridge Hub Pattern (Advanced)

**For complex systems with many indicators:**

**File: `data_hub.pine`**

```pinescript
//@version=5
indicator("Data Hub - Centralized Bridge", overlay=true)

// ===== IMPORT FROM MULTIPLE SOURCES =====
source1_value = input.source(close, "Source 1 Value", group="Imports")
source2_value = input.source(close, "Source 2 Value", group="Imports")
source3_value = input.source(close, "Source 3 Value", group="Imports")

// ===== VALIDATE & TRANSFORM DATA =====
// Check for stale or invalid data
isValid1 = source1_value != source1_value[1] or bar_index < 10  // Data changed or early bars
isValid2 = source2_value != source2_value[1] or bar_index < 10
isValid3 = source3_value != source3_value[1] or bar_index < 10

// Calculate confidence based on data validity
overallConfidence = (isValid1 ? 0.33 : 0.0) + 
                    (isValid2 ? 0.33 : 0.0) + 
                    (isValid3 ? 0.33 : 0.0)

// ===== COMBINE DATA =====
// Example: weighted average
combinedValue = (source1_value * 0.5) + (source2_value * 0.3) + (source3_value * 0.2)

// Example: consensus signal (all must agree)
bullish1 = source1_value > source1_value[1]
bullish2 = source2_value > source2_value[1]
bullish3 = source3_value > source3_value[1]

consensusBullish = bullish1 and bullish2 and bullish3 ? 1.0 : 0.0
consensusBearish = not bullish1 and not bullish2 and not bullish3 ? -1.0 : 0.0

// ===== EXPORT UNIFIED DATA =====
plot(combinedValue, "Hub_CombinedValue", display=display.none)
plot(overallConfidence, "Hub_Confidence", display=display.none)
plot(consensusBullish, "Hub_ConsensusBullish", display=display.none)
plot(consensusBearish, "Hub_ConsensusBearish", display=display.none)

// ===== STATUS DISPLAY =====
var table hub = table.new(position.bottom_left, 2, 5)
if barstate.islast
    table.cell(hub, 0, 0, "Data Hub Active", bgcolor=color.new(color.purple, 70))
    table.merge_cells(hub, 0, 0, 1, 0)
    
    table.cell(hub, 0, 1, "Sources Valid:")
    validCount = (isValid1 ? 1 : 0) + (isValid2 ? 1 : 0) + (isValid3 ? 1 : 0)
    table.cell(hub, 1, 1, str.tostring(validCount) + "/3",
        bgcolor=validCount == 3 ? color.green : color.orange)
    
    table.cell(hub, 0, 2, "Confidence:")
    table.cell(hub, 1, 2, str.tostring(overallConfidence * 100, "#") + "%")
    
    table.cell(hub, 0, 3, "Combined:")
    table.cell(hub, 1, 3, str.tostring(combinedValue, format.mintick))
    
    table.cell(hub, 0, 4, "Consensus:")
    consensusText = consensusBullish > 0 ? "BULLISH" : 
                    consensusBearish < 0 ? "BEARISH" : 
                    "NEUTRAL"
    table.cell(hub, 1, 4, consensusText,
        bgcolor=consensusBullish > 0 ? color.green : 
                consensusBearish < 0 ? color.red : 
                color.gray)
```

**Benefits of Hub Pattern:**
- ✅ Single connection point for downstream indicators
- ✅ Data validation and quality checks
- ✅ Transformation layer (normalization, weighting, consensus)
- ✅ Easier debugging (can log hub state)

---

## Hybrid Architecture

### Complete Example: Library + Data Bridge

**File 1: `core_library.pine` (Published Library)**

```pinescript
//@version=5
// @description Core calculation library
library("core_lib")

// @function Generic calculation that many indicators need
// @param data Input data series
// @param period Calculation period
// @returns Calculated result
export calculate(float data, simple int period) =>
    result = ta.sma(data, period)
    result

// @function Utility for strength scoring
// @param value Current value
// @param historical Historical average
// @returns Strength score 0-100
export calculateStrength(float value, float historical) =>
    ratio = historical != 0 ? value / historical : 1.0
    strength = math.min(100, math.max(0, ratio * 100))
    strength

// @type Shared data structure
export type Signal
    float value
    float strength
    string category
```

**File 2: `indicator_a.pine` (Specialized Indicator)**

```pinescript
//@version=5
indicator("Indicator A", overlay=true)

// Import library
import username/core_lib/1 as lib

// Settings
period = input.int(20, "Period")

// Use library function
calculatedValue = lib.calculate(close, period)
historicalAvg = ta.sma(calculatedValue, 50)
strength = lib.calculateStrength(calculatedValue, historicalAvg)

// Create library type instance
signal = lib.Signal.new(
    value = calculatedValue,
    strength = strength,
    category = strength > 70 ? "STRONG" : "WEAK"
)

// Visualize
plot(signal.value, "Value")

// Export for data bridge
plot(signal.value, "ExportA_Value", display=display.none)
plot(signal.strength, "ExportA_Strength", display=display.none)
```

**File 3: `indicator_b.pine` (Another Specialized Indicator)**

```pinescript
//@version=5
indicator("Indicator B", overlay=true)

// Import same library
import username/core_lib/1 as lib

// Different settings
period = input.int(50, "Period")

// Use same library function with different parameters
calculatedValue = lib.calculate(close, period)
historicalAvg = ta.sma(calculatedValue, 100)
strength = lib.calculateStrength(calculatedValue, historicalAvg)

// Create same library type
signal = lib.Signal.new(
    value = calculatedValue,
    strength = strength,
    category = strength > 80 ? "VERY_STRONG" : strength > 60 ? "STRONG" : "WEAK"
)

// Visualize
plot(signal.value, "Value")

// Export for data bridge
plot(signal.value, "ExportB_Value", display=display.none)
plot(signal.strength, "ExportB_Strength", display=display.none)
```

**File 4: `ensemble.pine` (Combines Everything)**

```pinescript
//@version=5
indicator("Ensemble", overlay=true)

// Import library for utility functions
import username/core_lib/1 as lib

// Import data from specialized indicators via data bridge
valueA = input.source(close, "Indicator A Value")
strengthA = input.source(50.0, "Indicator A Strength")

valueB = input.source(close, "Indicator B Value")
strengthB = input.source(50.0, "Indicator B Strength")

// Use library function to combine signals
combinedValue = lib.calculate((valueA + valueB) / 2, 10)

// Weight by strength
weightedValue = (valueA * strengthA + valueB * strengthB) / (strengthA + strengthB)

// Calculate ensemble strength using library function
ensembleStrength = lib.calculateStrength(weightedValue, ta.sma(weightedValue, 50))

// Create ensemble signal using library type
ensemble = lib.Signal.new(
    value = weightedValue,
    strength = ensembleStrength,
    category = ensembleStrength > 75 ? "STRONG" : "MODERATE"
)

// Visualize
plot(ensemble.value, "Ensemble Value", color.blue, linewidth=2)

// Display
var table info = table.new(position.top_right, 2, 3)
if barstate.islast
    table.cell(info, 0, 0, "Ensemble Status")
    table.merge_cells(info, 0, 0, 1, 0)
    
    table.cell(info, 0, 1, "Strength:")
    table.cell(info, 1, 1, str.tostring(ensemble.strength, "#"))
    
    table.cell(info, 0, 2, "Category:")
    table.cell(info, 1, 2, ensemble.category,
        bgcolor=ensemble.category == "STRONG" ? color.green : color.orange)
```

**Benefits of This Architecture:**
- ✅ `calculate()` function written once (library), used in 3 indicators
- ✅ `calculateStrength()` function written once, used in 3 indicators
- ✅ `Signal` type defined once, used in 3 indicators
- ✅ Indicators A and B export their results
- ✅ Ensemble combines results flexibly (users can swap indicators)
- ✅ Bug fix in library? Update library version, all indicators benefit

---

## Best Practices

### For Libraries

#### 1. Function Design

**✅ DO:**
```pinescript
// Clear, focused function with good defaults
export calculateMovingAverage(series float data, simple int period = 20, simple string type = "SMA") =>
    result = type == "EMA" ? ta.ema(data, period) : ta.sma(data, period)
    result
```

**❌ DON'T:**
```pinescript
// Vague, does too much, no defaults
export doStuff(float a, int b, int c, string d) =>
    // ... 100 lines of mixed logic ...
```

**Guidelines:**
- One function = one clear purpose
- Provide sensible defaults for all parameters
- Use `simple` type for parameters when possible (better performance)
- Return early for edge cases
- Maximum ~50 lines per function (extract helpers if longer)

#### 2. Documentation

**✅ DO:**
```pinescript
// @function Calculates momentum over specified period
// @param source Price source (typically close, high, or low)
// @param period Number of bars for momentum calculation
// @param smoothing Optional smoothing period (0 = no smoothing)
// @returns Momentum value normalized to 0-100 range
export calculateMomentum(series float source, simple int period, simple int smoothing = 0) =>
    // Implementation...
```

**❌ DON'T:**
```pinescript
// calc mom
export calcMom(float s, int p) =>
    // No docs, unclear parameters
```

**Guidelines:**
- Always use JSDoc style comments
- Describe what the function does (not how)
- Document all parameters with types
- Document return values
- Include units where applicable (%, points, etc.)

#### 3. Error Handling

**✅ DO:**
```pinescript
export safeDivide(float numerator, float denominator, float defaultValue = 0.0) =>
    result = denominator != 0 ? numerator / denominator : defaultValue
    result

export safeCalculate(series float data, simple int period) =>
    // Validate inputs
    validPeriod = math.max(1, period)  // Ensure positive
    validData = na(data) ? close : data  // Fallback if na
    
    result = ta.sma(validData, validPeriod)
    result
```

**❌ DON'T:**
```pinescript
export calculate(float data, int period) =>
    result = data / period  // Crashes if period = 0
    result
```

**Guidelines:**
- Validate all inputs
- Provide sensible defaults for edge cases
- Never let division by zero occur
- Handle `na` values explicitly
- Return meaningful values even on error

#### 4. Versioning Strategy

**Semantic Versioning:**
```
MAJOR.MINOR.PATCH

Example: 2.3.1
  │   │  │
  │   │  └─ Bug fixes, no behavior change
  │   └──── New features, backward compatible
  └──────── Breaking changes
```

**Version Comments:**
```pinescript
//@version=5
// @version 2.3.1
// @changelog
// v2.3.1 - Fixed divide-by-zero in calculateRatio()
// v2.3.0 - Added optional smoothing parameter to calculateMomentum()
// v2.0.0 - BREAKING: Changed return type of calculateStrength() from int to float
library("my_lib")
```

### For Data Bridge

#### 1. Naming Conventions

**✅ DO:**
```pinescript
// Clear, namespaced naming
plot(pocValue, "VP_POC", display=display.none)  // VP = Volume Profile
plot(resistance, "SR_Resistance1", display=display.none)  // SR = Support/Resistance
plot(regime, "REG_Regime", display=display.none)  // REG = Regime
plot(confidence, "META_Confidence", display=display.none)  // META = Metadata
```

**❌ DON'T:**
```pinescript
// Ambiguous, could conflict
plot(value, "Value", display=display.none)  // Which value?
plot(signal, "Signal", display=display.none)  // Which signal?
```

**Naming Pattern:**
```
[CATEGORY]_[SPECIFIC_NAME]

Categories:
- VP_ (Volume Profile)
- SR_ (Support/Resistance)
- REG_ (Regime)
- MA_ (Moving Average)
- IND_ (General Indicator)
- META_ (Metadata like confidence, regime, etc.)
```

#### 2. Metadata Export

**Always export metadata alongside data:**

```pinescript
// Export actual values
plot(calculatedValue, "IND_Value", display=display.none)

// Export quality indicators
plot(confidence, "IND_Confidence", display=display.none)  // 0.0-1.0
plot(barsActive, "IND_BarsActive", display=display.none)  // How long this signal has been valid
plot(isValid ? 1.0 : 0.0, "IND_IsValid", display=display.none)  // Binary flag

// Export contextual info
plot(regimeEncoded, "IND_Regime", display=display.none)  // 1=LOW, 2=NORMAL, 3=HIGH
```

**Why:** Consumers can make intelligent decisions based on data quality

#### 3. Default Values

**Always provide working defaults:**

```pinescript
// ✅ GOOD: Works without setup, improves with setup
importedValue = input.source(close, "External Value",
    tooltip="Default: close price. For best results, select external indicator value")

// Apply default fallback
effectiveValue = importedValue == close ? 
    ta.sma(close, 20) :  // Fallback calculation
    importedValue         // User-provided value

// ❌ BAD: Breaks without setup
importedValue = input.source(0, "External Value")  // 0 is rarely meaningful
```

#### 4. Connection Status Checking

**Heuristic for detecting connection:**

```pinescript
// Check if imported value differs from default
isConnected = importedValue != close  // Assuming default is 'close'

// Or check for reasonable range
isConnectedAndValid = importedValue != close and 
                      importedValue > 0 and 
                      not na(importedValue)

// Display status
var table status = table.new(position.top_right, 2, 2)
if barstate.islast
    table.cell(status, 0, 0, "Connection:")
    table.cell(status, 1, 0, isConnected ? "✓ Active" : "⚠ Default",
        bgcolor=isConnected ? color.green : color.orange)
```

### For Both

#### 1. Performance Optimization

**Heavy calculations once:**
```pinescript
// ✅ GOOD: Calculate once per bar
var float expensiveResult = na
if barstate.isfirst or barstate.islast
    expensiveResult := doExpensiveCalculation()

plot(expensiveResult)
```

**Avoid recalculation:**
```pinescript
// ❌ BAD: Calculates on every render
plot(doExpensiveCalculation())  // May run multiple times per bar
```

**Use simple types:**
```pinescript
// ✅ GOOD: simple = compile-time constant
export calculate(simple int period, simple string mode) =>
    // Faster execution
    
// ⚠️ OK but slower: series = runtime variable
export calculate(series int period, series string mode) =>
    // More flexible but slower
```

#### 2. Memory Management

**Limit array sizes:**
```pinescript
// ✅ GOOD: Fixed-size arrays with circular buffer
var maxSize = 100
var levels = array.new_float(maxSize, 0.0)

addLevel(level) =>
    if array.size(levels) >= maxSize
        array.shift(levels)  // Remove oldest
    array.push(levels, level)
```

**Clean up old data:**
```pinescript
// ✅ GOOD: Periodic cleanup
if bar_index % 100 == 0  // Every 100 bars
    cleanupOldData()
```

#### 3. User Experience

**Provide visual feedback:**
```pinescript
// Show indicator is active and working
var table status = table.new(position.bottom_right, 2, 2)
if barstate.islast
    table.cell(status, 0, 0, "Indicator Active", bgcolor=color.green)
    table.cell(status, 0, 1, "Last Calc:")
    table.cell(status, 1, 1, str.format("{0,time,HH:mm:ss}", time))
```

**Helpful tooltips:**
```pinescript
period = input.int(20, "Period",
    tooltip="Number of bars for calculation. Higher = smoother but slower to react. " +
            "Recommended: 20 for day trading, 50 for swing trading")
```

**Clear error messages:**
```pinescript
if invalidCondition
    label.new(bar_index, high, "⚠ Invalid settings: period must be > 0",
        style=label.style_label_down, color=color.red, textcolor=color.white)
```

---

## Common Pitfalls

### Library Pitfalls

#### 1. Breaking Changes Without Version Bump

**❌ PROBLEM:**
```pinescript
// Version 1
export calculate(int period) => ta.sma(close, period)

// Still version 1 (updated the same version)
export calculate(int period, string type) => 
    type == "SMA" ? ta.sma(close, period) : ta.ema(close, period)
    // BREAKS all indicators using version 1!
```

**✅ SOLUTION:**
```pinescript
// Version 1
export calculate(int period) => ta.sma(close, period)

// Version 2 (new version)
export calculate(int period, string type = "SMA") => 
    type == "SMA" ? ta.sma(close, period) : ta.ema(close, period)
    // Old indicators still work with v1, new ones can use v2
```

#### 2. Over-Generalization

**❌ PROBLEM:**
```pinescript
// Tries to do everything
export universalCalculator(float data, int param1, int param2, int param3, 
                          string mode, string type, string option) =>
    // 500 lines of if-else spaghetti
```

**✅ SOLUTION:**
```pinescript
// Focused functions
export calculateSMA(float data, int period) => ...
export calculateEMA(float data, int period) => ...
export calculateVWAP(float data, int period) => ...
```

#### 3. Stateful Functions (Usually Problematic)

**❌ PROBLEM:**
```pinescript
// Uses var inside function - dangerous
export statefulCalculation() =>
    var float state = 0.0
    state += close  // Accumulates forever!
    state
// Problem: State persists across all calls, causes unexpected behavior
```

**✅ SOLUTION:**
```pinescript
// Stateless - takes previous value as parameter
export statefulCalculation(float previousState) =>
    newState = previousState + close
    newState

// Or use series parameter
export statefulCalculation(series float state) =>
    result = state + close
    result
```

### Data Bridge Pitfalls

#### 1. Forgetting display=display.none

**❌ PROBLEM:**
```pinescript
plot(calculatedValue, "Export_Value")  // Shows on chart!
// Chart becomes cluttered with export lines
```

**✅ SOLUTION:**
```pinescript
plot(calculatedValue, "Export_Value", 
    color=color.new(color.blue, 100),  // Fully transparent
    display=display.none)  // Invisible

// Optionally also plot visible version
plot(calculatedValue, "Visual Value", color.blue, linewidth=2)
```

#### 2. Name Collisions

**❌ PROBLEM:**
```pinescript
// Indicator A
plot(value, "Value", display=display.none)

// Indicator B
plot(value, "Value", display=display.none)

// Consumer - which "Value" to select??
imported = input.source(close, "Value")  // Ambiguous!
```

**✅ SOLUTION:**
```pinescript
// Indicator A
plot(value, "IndicatorA_Value", display=display.none)

// Indicator B  
plot(value, "IndicatorB_Value", display=display.none)

// Consumer - clear distinction
importedA = input.source(close, "Indicator A Value",
    tooltip="Select 'IndicatorA_Value'")
importedB = input.source(close, "Indicator B Value",
    tooltip="Select 'IndicatorB_Value'")
```

#### 3. No Validation of Imported Data

**❌ PROBLEM:**
```pinescript
imported = input.source(close, "External Data")
result = 100 / imported  // Crashes if imported = 0!
```

**✅ SOLUTION:**
```pinescript
imported = input.source(close, "External Data")

// Validate before use
validImported = imported != 0 ? imported : close
result = 100 / validImported

// Or check for reasonable range
if imported < 0 or imported > 1000000 or na(imported)
    validImported := close  // Fallback
else
    validImported := imported
```

#### 4. Assuming Immediate Setup

**❌ PROBLEM:**
```pinescript
// Assumes user has connected data source
imported = input.source(close, "Must Connect!")
if imported == close
    runtime.error("Please connect data source!")  // Crashes user's indicator!
```

**✅ SOLUTION:**
```pinescript
// Works with defaults, better with setup
imported = input.source(close, "Optional External Data",
    tooltip="Default: close price. Connect external indicator for better accuracy")

// Detect connection and adapt
isConnected = imported != close
confidence = isConnected ? 1.0 : 0.5  // Lower confidence with defaults

// Or provide fallback calculation
effectiveValue = isConnected ? 
    imported : 
    ta.sma(close, 20)  // Fallback if not connected
```

### Architecture Pitfalls

#### 1. Wrong Tool for the Job

**❌ PROBLEM:**
```pinescript
// Using data bridge when library would be better
// Producer indicator A
calculate() =>
    result = close * 1.5
    result
output = calculate()
plot(output, "Export", display=display.none)

// Producer indicator B (duplicate code!)
calculate() =>
    result = close * 1.5
    result
output = calculate()
plot(output, "Export", display=display.none)
```

**✅ SOLUTION:**
```pinescript
// Library (shared code)
export calculate() =>
    result = close * 1.5
    result

// Indicator A
import lib/1 as lib
plot(lib.calculate())

// Indicator B
import lib/1 as lib
plot(lib.calculate())
```

#### 2. Circular Dependencies

**❌ PROBLEM:**
```pinescript
// Indicator A exports to B
// Indicator B exports to A
// Both try to import from each other = infinite loop!
```

**✅ SOLUTION:**
```pinescript
// Use hub pattern
// Indicator A exports to Hub
// Indicator B exports to Hub
// Hub combines and exports to both A and B
```

#### 3. Too Many Layers

**❌ PROBLEM:**
```pinescript
// Data Source → Hub1 → Hub2 → Hub3 → Consumer
// 5 indicators just to get one value!
```

**✅ SOLUTION:**
```pinescript
// Data Source → Consumer (direct)
// Or: Data Source → Hub → Consumer (max 3 indicators)
```

---

## Testing & Validation

### Library Testing

**Create Test Indicator:**

```pinescript
//@version=5
indicator("Library Test Suite", overlay=true)

import username/mylib/1 as lib

// ===== TEST 1: Basic Function Call =====
test1_result = lib.calculate(20)
test1_expected = ta.sma(close, 20)
test1_passed = math.abs(test1_result - test1_expected) < 0.01

// ===== TEST 2: Edge Cases =====
test2_result = lib.calculate(0)  // Should handle gracefully
test2_passed = not na(test2_result)

// ===== TEST 3: Type Creation =====
test3_obj = lib.CustomType.new(value=100.0, label="test")
test3_passed = test3_obj.value == 100.0

// ===== DISPLAY RESULTS =====
var table results = table.new(position.top_left, 3, 4)
if barstate.islast
    table.cell(results, 0, 0, "Test Suite")
    table.merge_cells(results, 0, 0, 2, 0)
    
    table.cell(results, 0, 1, "Test 1:")
    table.cell(results, 1, 1, "Basic Function")
    table.cell(results, 2, 1, test1_passed ? "✓ PASS" : "✗ FAIL",
        bgcolor=test1_passed ? color.green : color.red)
    
    table.cell(results, 0, 2, "Test 2:")
    table.cell(results, 1, 2, "Edge Cases")
    table.cell(results, 2, 2, test2_passed ? "✓ PASS" : "✗ FAIL",
        bgcolor=test2_passed ? color.green : color.red)
    
    table.cell(results, 0, 3, "Test 3:")
    table.cell(results, 1, 3, "Type Creation")
    table.cell(results, 2, 3, test3_passed ? "✓ PASS" : "✗ FAIL",
        bgcolor=test3_passed ? color.green : color.red)

// Alert if any tests fail
if not (test1_passed and test2_passed and test3_passed)
    runtime.error("Library tests failed!")
```

### Data Bridge Testing

**Test Connection Status:**

```pinescript
//@version=5
indicator("Data Bridge Test", overlay=true)

// Import
imported = input.source(close, "Test Import")

// Test 1: Connection detected
test1_passed = imported != close  // Heuristic: did user connect?

// Test 2: Valid data range
test2_passed = imported > 0 and imported < 1000000 and not na(imported)

// Test 3: Data updates (not static)
test3_passed = imported != imported[1] or bar_index < 10

// Display
var table results = table.new(position.top_right, 2, 4)
if barstate.islast
    table.cell(results, 0, 0, "Bridge Test")
    table.merge_cells(results, 0, 0, 1, 0)
    
    table.cell(results, 0, 1, "Connected:")
    table.cell(results, 1, 1, test1_passed ? "✓" : "✗",
        bgcolor=test1_passed ? color.green : color.orange)
    
    table.cell(results, 0, 2, "Valid Range:")
    table.cell(results, 1, 2, test2_passed ? "✓" : "✗",
        bgcolor=test2_passed ? color.green : color.red)
    
    table.cell(results, 0, 3, "Data Updates:")
    table.cell(results, 1, 3, test3_passed ? "✓" : "✗",
        bgcolor=test3_passed ? color.green : color.red)
```

### Performance Testing

**Measure Execution Time:**

```pinescript
//@version=5
indicator("Performance Test", overlay=true)

// Test expensive operation
startTime = timenow
for i = 0 to 1000
    _ = ta.sma(close, 20)  // Simulate work
endTime = timenow

executionMs = endTime - startTime

// Display
var label perf = na
label.delete(perf)
perf := label.new(bar_index, high, 
    "Execution: " + str.tostring(executionMs) + "ms",
    style=label.style_label_down,
    color=executionMs < 100 ? color.green : color.red)
```

### Validation Checklist

**Before Publishing:**

- [ ] All functions have JSDoc comments
- [ ] All edge cases handled (0, na, negative values)
- [ ] No division by zero
- [ ] No infinite loops
- [ ] Memory usage within limits
- [ ] Version number correct
- [ ] Changelog updated
- [ ] Test suite passes
- [ ] Performance acceptable (<100ms per bar)
- [ ] User documentation written
- [ ] Example indicators created

---

## Appendix: Complete Working Example

### Example: Moving Average Library System

**File 1: `ma_library.pine`**

```pinescript
//@version=5
// @description Moving average calculation library with multiple types
// @version 1.0.0
library("ma_lib")

// @function Calculates moving average of specified type
// @param source Data source
// @param period MA period
// @param maType Type: "SMA", "EMA", "WMA", "VWMA"
// @returns Calculated moving average
export calculateMA(series float source, simple int period, simple string maType = "SMA") =>
    result = switch maType
        "SMA" => ta.sma(source, period)
        "EMA" => ta.ema(source, period)
        "WMA" => ta.wma(source, period)
        "VWMA" => ta.vwma(source, period)
        => ta.sma(source, period)  // Default
    result

// @type MA configuration
export type MAConfig
    int period
    string maType
    color col

// @function Creates default MA config
// @returns Default configuration
export defaultConfig() =>
    MAConfig.new(period=20, maType="SMA", col=color.blue)
```

**File 2: `ma_fast.pine`**

```pinescript
//@version=5
indicator("MA Fast", overlay=true)

import username/ma_lib/1 as ma

// Settings
config = ma.MAConfig.new(
    period = input.int(10, "Period"),
    maType = input.string("EMA", "Type", options=["SMA", "EMA", "WMA", "VWMA"]),
    col = input.color(color.blue, "Color")
)

// Calculate using library
fastMA = ma.calculateMA(close, config.period, config.maType)

// Visualize
plot(fastMA, "Fast MA", config.col, linewidth=2)

// Export for data bridge
plot(fastMA, "Export_FastMA", color.new(config.col, 100), display=display.none)
```

**File 3: `ma_slow.pine`**

```pinescript
//@version=5
indicator("MA Slow", overlay=true)

import username/ma_lib/1 as ma

// Settings (same library, different parameters)
config = ma.MAConfig.new(
    period = input.int(50, "Period"),
    maType = input.string("SMA", "Type", options=["SMA", "EMA", "WMA", "VWMA"]),
    col = input.color(color.red, "Color")
)

// Calculate using library
slowMA = ma.calculateMA(close, config.period, config.maType)

// Visualize
plot(slowMA, "Slow MA", config.col, linewidth=2)

// Export for data bridge
plot(slowMA, "Export_SlowMA", color.new(config.col, 100), display=display.none)
```

**File 4: `ma_crossover.pine`**

```pinescript
//@version=5
indicator("MA Crossover Strategy", overlay=true)

// Import library for types
import username/ma_lib/1 as ma

// Import data via data bridge
fastMA = input.source(close, "Fast MA", 
    tooltip="Select 'MA Fast: Export_FastMA'")
slowMA = input.source(close, "Slow MA",
    tooltip="Select 'MA Slow: Export_SlowMA'")

// Detect crossovers
bullishCross = ta.crossover(fastMA, slowMA)
bearishCross = ta.crossunder(fastMA, slowMA)

// Visualize
plotshape(bullishCross, "Buy", shape.triangleup, location.belowbar, 
    color.green, size=size.small)
plotshape(bearishCross, "Sell", shape.triangledown, location.abovebar,
    color.red, size=size.small)

// Alerts
if bullishCross
    alert("Bullish crossover!", alert.freq_once_per_bar_close)
if bearishCross
    alert("Bearish crossover!", alert.freq_once_per_bar_close)
```

**Result:**
- ✅ MA calculation written once (library)
- ✅ Two specialized indicators reuse library code
- ✅ Strategy indicator combines their outputs
- ✅ Users can swap fast/slow MAs freely
- ✅ Total code: ~150 lines (vs. ~300+ without library)

---

## Conclusion

### Key Takeaways

1. **Library = Code Sharing**, Data Bridge = Data Sharing
2. Use **Library** when you control the code
3. Use **Data Bridge** when combining any indicators
4. Use **Both** for professional systems
5. Always document exports clearly
6. Provide defaults for data bridge inputs
7. Version libraries properly
8. Test before publishing

### Decision Summary

```
Need to share calculation logic? → Library
Need to combine indicator results? → Data Bridge
Building from scratch? → Library first, export data optionally
Combining third-party? → Data Bridge only
Professional suite? → Library + Data Bridge
```

### Next Steps

1. Identify what needs sharing (code vs. data)
2. Choose architecture pattern (Pure Library, Pure Bridge, Hybrid, Hub)
3. Create library if needed (publish as private initially)
4. Build specialized indicators using library
5. Export data via invisible plots
6. Create ensemble/consumer indicators
7. Test thoroughly
8. Document setup instructions
9. Iterate based on user feedback

---

**Document Version:** 1.0  
**Last Updated:** November 17, 2025  
**Maintained By:** Research Team  
**License:** Internal Use
