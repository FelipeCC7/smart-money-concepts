# Swing Highs & Lows: In-Depth Analysis & Comparison

This document provides a comprehensive analysis of swing highs/lows implementations in the **Smart Money Concepts (SMC)** package and the **CCE Strategy** from `algotrading_development`.

---

## Smart Money Concepts (SMC) Approach

**Location:** `smartmoneyconcepts/smc.py:137-219`

### Definition

- **Swing High**: A candle whose high is the **highest** across `swing_length` candles before AND after
- **Swing Low**: A candle whose low is the **lowest** across `swing_length` candles before AND after

### Algorithm (3 Phases)

```
Phase 1: Rolling Window Detection
─────────────────────────────────
swing_length *= 2  # e.g., 50 → 100
- Shift data forward by swing_length/2
- Rolling max/min over swing_length window
- Mark: 1 = swing high, -1 = swing low, NaN = neither

Phase 2: Duplicate Removal (ensures alternation)
────────────────────────────────────────────────
- Two consecutive highs → keep HIGHEST
- Two consecutive lows → keep LOWEST
- Result: high-low-high-low pattern

Phase 3: Edge Correction
────────────────────────
- Adds artificial swings at first/last candles
- Maintains alternating pattern at boundaries
```

### Key Parameter

**`swing_length`** (default: 50)
- Doubled internally → actually uses 100-candle window
- Larger = fewer, more significant swings
- Smaller = more sensitive, more swings

### Dependent Indicators

| Indicator | Purpose |
|-----------|---------|
| `bos_choch()` | Break of Structure / Change of Character |
| `ob()` | Order Blocks |
| `liquidity()` | Liquidity pools (clustered swing levels) |
| `retracements()` | Fibonacci retracement calculations |

---

## CCE Strategy Approach

**Location:** `algotrading_development/00 - Runtime Version/Python Files/Trading/CalculationsPack.py:163-182`

### Definition

Candle-by-candle comparison with the previous bar:

```python
# legCounter() classifications:
UP    = high > prev_high AND low >= prev_low    # Bullish candle
DOWN  = high <= prev_high AND low < prev_low    # Bearish candle
OUTER = high > prev_high AND low < prev_low     # Expansion (engulfing)
INNER = high <= prev_high AND low >= prev_low   # Contraction (inside bar)
```

### Swing Extraction

The `reading_leg_map()` function extracts swings from leg classifications:
- `DOWN → UP` transition = **Swing Low** (bottom)
- `UP → DOWN` transition = **Swing High** (top)

---

## Key Differences

| Aspect | SMC Package | CCE Strategy |
|--------|-------------|--------------|
| **Detection Method** | Rolling window (look-ahead) | Sequential comparison |
| **Lookahead** | Yes (requires future data) | No (real-time capable) |
| **Parameter** | `swing_length` window size | None (fixed logic) |
| **Sensitivity** | Configurable via parameter | Fixed - every direction change |
| **Use Case** | Backtesting analysis | Live trading signals |
| **Output** | Binary (1/-1) + price level | Leg type + detailed metadata |

---

## Visual Comparison

```
Price Action:    ╱╲    ╱╲╱╲    ╱╲
                ╱  ╲  ╱    ╲  ╱  ╲
               ╱    ╲╱      ╲╱    ╲

SMC (swing_length=10):
Swings:         H         H       H
                    L         L

CCE (legCounter):
Swings:         H    H H    H    H
                  L    L  L    L
```

**SMC** filters out minor swings; **CCE** captures every reversal.

---

## How SMC Swings Flow Into Other Indicators

```
OHLC Data
    │
    ▼
swing_highs_lows(ohlc, swing_length=50)
    │
    ├──► bos_choch() → Detects market structure breaks
    │                   (when price closes beyond previous swing)
    │
    ├──► ob() → Identifies order blocks
    │           (last bearish candle before swing high break = bullish OB)
    │
    ├──► liquidity() → Finds liquidity pools
    │                   (multiple swing highs/lows within range_percent)
    │
    └──► retracements() → Calculates fib retracements between swings
```

---

## Integration Opportunity

The CCE strategy could benefit from SMC's `swing_length` parameter to **filter noise**:

```python
# Current CCE approach - captures all reversals
legs = legCounter(df)  # Very sensitive

# Potential enhancement - use SMC swings for structure
from smartmoneyconcepts import smc
structure_swings = smc.swing_highs_lows(df, swing_length=20)  # Major structure
micro_swings = smc.swing_highs_lows(df, swing_length=5)       # Minor structure
```

This would give you **multi-resolution swing analysis** - major structure for trend direction, minor for entry timing.

---

## File Locations Reference

### Smart Money Concepts

| File | Purpose | Lines |
|------|---------|-------|
| `smartmoneyconcepts/smc.py` | Main indicator implementation | 137-219 |
| `smartmoneyconcepts/__init__.py` | Package initialization | 1-4 |
| `tests/unit_tests.py` | Test cases | 43-144 |

### CCE Strategy (algotrading_development)

| Component | File Path | Lines |
|-----------|-----------|-------|
| legCounter (swing classification) | `00 - Runtime Version/Python Files/Trading/CalculationsPack.py` | 163-182 |
| reading_leg_map (swing extraction) | `00 - Runtime Version/Python Files/Trading/CalculationsPack.py` | 185-464 |
| FiboLevels (Fibonacci calc) | `00 - Runtime Version/Python Files/Trading/CalculationsPack.py` | 13-19 |
| High/Low ranges (vectorized) | `02 - Cloud Version/utilities/optimized_indicator_engine.py` | 222-242 |
| Fibonacci batch processing | `02 - Cloud Version/utilities/optimized_indicator_engine.py` | 244-279 |
| CCE strategy usage | `01 - Database Version/00 vsCode/CCE_Strategy_cmd.py` | 146-194 |

---

## Next Steps

1. **Side-by-side code comparison** - Deep dive into implementation details
2. **Integration plan** - How to incorporate SMC swings into CCE strategy
3. **Indicator deep-dives** - Detailed analysis of Order Blocks, BOS/CHoCH, Liquidity
