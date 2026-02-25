# OOTM Strategy (Pine Script v6) — Session Sweeps + RR Projection

A TradingView **strategy script** that:
- Tracks **Asia / London / New York** session highs & lows (UTC+2 inputs)
- Detects **liquidity sweeps** of finalized session levels
- Applies optional **EMA200 trend filter** and **MA22 filter**
- Draws a **trade projection** (RR box or lines) for the current idea
- Places **backtest orders** (flat-only) with **max trades per day**
- Emits **JSON alerts** for automation / execution bridges
- Shows a small **stats table** (wins/losses/BE + daily counts)

> Script name: `OOTM strategy`  
> Pine version: `//@version=6`

---

## Core Idea

### Sessions
The script defines three sessions (configurable):
- **Asia**: `0200-0800`
- **London**: `0900-1200`
- **New York**: `1530-1900`

During each session it continuously updates the session high/low and draws **one persistent high line + one persistent low line** (no stacking).

At the **end** of Asia/London, it stores a finalized level:
- `aH_final / aL_final`
- `lH_final / lL_final`

### Sweep Conditions
A sweep is defined as:
- Price **trades beyond** a finalized level (wick breaks)
- Candle **closes back inside** (close re-enters)
- Confirmed bar (`barstate.isconfirmed`)

Sweeps are only allowed in specific windows:
- **Asia sweep entries**: allowed during **London or NY**, but **not during Asia**
- **London sweep entries**: allowed during **NY**, but **not during London**

Signals:
- **Long idea**: sweep **below** (Asia low or London low) and close back above
- **Short idea**: sweep **above** (Asia high or London high) and close back below

---

## Filters

Two optional filters:
- **Trend filter** (default ON): `close > EMA200` for longs, `close < EMA200` for shorts
- **MA22 filter** (default OFF): `close > MA22` for longs, `close < MA22` for shorts  
  MA22 can be EMA or SMA via input.

Plots:
- `MA22`
- `EMA Trend` (EMA200)

---

## Risk Model & Projection

Risk is derived from the swept level plus an ATR buffer:
- `ATR(atrLen)` * `atrBufMult`

Stops:
- Long SL: `sweptLow - buf`
- Short SL: `sweptHigh + buf`

Take profit uses RR:
- Long TP: `entry + (entry - SL) * rr`
- Short TP: `entry - (SL - entry) * rr`

Visualization:
- Either a clean **RR box** (`useRRBox=true`)  
- Or three lines (entry/SL/TP)

Only **one active projection** exists at a time; new ideas delete the old drawing objects.

---

## Strategy Orders (Backtest)

Controlled by:
- `enableOrders` (default ON)
- `maxTradesPerDay` (default 2)
- Flat-only execution (`strategy.position_size == 0`)

Triggering:
- Uses **edge trigger** so it only fires on the first bar a signal appears:
  - `longTrig = longOnly and not longOnly[1]`
  - `shortTrig = shortOnly and not shortOnly[1]`

Orders:
- `strategy.entry("L", strategy.long)` and `strategy.exit("L-exit", stop=slLong, limit=tpLong)`
- `strategy.entry("S", strategy.short)` and `strategy.exit("S-exit", stop=slShort, limit=tpShort)`

No pyramiding, no flips, no overlapping positions.
