# OOTM Strategy (Pine Script v6) — Session Sweeps & Breakouts + RR Projection

A feature-rich TradingView **strategy script** written in Pine Script v6 that detects and projects high-probability trades based on session high/low levels. 

The repository contains two script variants:
- **[EOM Script.pine](file:///Users/_freakzy_/Desktop/Trading%20VIew%20Indicator/Trading_View_Indicator/EOM%20Script.pine)**: Configured with an initial capital of **$100** (ideal for small account sizing or testing).
- **[EOM Strategy.pine](file:///Users/_freakzy_/Desktop/Trading%20VIew%20Indicator/Trading_View_Indicator/EOM%20Strategy.pine)**: Configured with an initial capital of **$3,000**.

Both scripts share identical trading logic, indicators, filtering mechanisms, and visual components.

---

## Key Features

- 🕒 **Session Tracking**: Monitored Sessions: **Asia / London / New York** (customizable, default Vienna/UTC+2 time). Displays session highs and lows with persistent, non-stacking dashed lines.
- 🔄 **Two Signal Modes**:
  - **Sweeps Mode** (Default): Detects when price trades beyond a finalized session level (wick break) but closes back inside.
  - **Breakouts Mode**: Detects when a candle close crosses completely out of a finalized session level.
- 📈 **Multi-Timeframe RSI Signals**: Plots standalone Multi-Timeframe (MTF) RSI BUY/SELL signals on the chart based on custom oversold (default `20`) and overbought (default `70`) thresholds.
- 🛡️ **Trend & Momentum Filters**:
  - **Trend filter** (optional, default ON): Restricts trades to the direction of the **EMA 200**.
  - **MA22 filter** (optional, default OFF): Restricts trades based on the relation to the **MA 22** (configurable as EMA or SMA).
- ⚖️ **Dynamic ATR Buffer Multiplier**: Automatically detects currency pairs and scales ATR buffer multipliers for high-volatility pairs:
  - **GBPJPY**: Scales buffer multiplier by `1.5x`
  - **GBP pairs**: Scales buffer multiplier by `1.2x`
  - **JPY pairs**: Scales buffer multiplier by `1.3x`
  - **Other pairs**: Uses the user-defined ATR buffer multiplier
- 📐 **Trade Projections**: Draws interactive Risk-to-Reward (RR) boxes or entry/SL/TP lines dynamically for the latest trade idea.
- 🤖 **Backtesting & Alert Automation**: Flat-only backtesting engine with configurable daily trade limits (default max `2` per day) and structured **JSON Alerts** for webhook/automation bridges.
- 📊 **Real-time Stats Table**: Displays a customizable HUD containing overall wins, losses, break-evens, win rates, and daily trade metrics.

---

## Technical Details

### Session Settings
By default, the script tracks:
- **Asia**: `0200-0800`
- **London**: `0900-1200`
- **New York**: `1530-1900`

Session lines update dynamically as the session develops. At the end of the Asia and London sessions, levels are finalized (`aH_final` / `aL_final` and `lH_final` / `lL_final`) to act as potential sweep or breakout areas:
- **Asia level trades**: Allowed during **London or New York** sessions.
- **London level trades**: Allowed during **New York** session only.

### Strategy Rules

#### 1. Sweeps Mode (Default)
- **Long Entry**: Price wicks below the finalized Asia/London session low and closes back above it.
- **Short Entry**: Price wicks above the finalized Asia/London session high and closes back below it.

#### 2. Breakouts Mode
- **Long Entry**: Price closes above the finalized Asia/London session high.
- **Short Entry**: Price closes below the finalized Asia/London session low.

### Risk Model

For each trade, the Stop Loss (SL) and Take Profit (TP) levels are derived dynamically:
- **ATR Calculation**: Uses a standard ATR length (default `14`).
- **Modern Buffer**: `ATR * ATR Buffer Multiplier` (scaled automatically for GBP/JPY pairs).
- **Long SL**: `Swept/Breakout Level - Buffer`
- **Short SL**: `Swept/Breakout Level + Buffer`
- **TP (Take Profit)**: Calculated using the user-defined Risk-to-Reward Ratio (RR) (default `3.0`).

### Filters
- **Trend Filter**: Longs allowed only when `close > EMA 200`. Shorts allowed only when `close < EMA 200`.
- **MA22 Filter**: Longs allowed only when `close > MA 22`. Shorts allowed only when `close < MA 22`.
- **Timeframe Filter**: Option to restrict signals strictly to the `15m` timeframe (`tf15mOnly`).

---

## Strategy Alerts (JSON Output)
The script triggers alerts on bar close with the following format:

**Long Alert Schema:**
```json
{
  "type": "signal",
  "side": "LONG",
  "symbol": "EURUSD",
  "tf": "15",
  "entry": "1.08500",
  "sl": "1.08200",
  "tp": "1.09400",
  "rr": "3.0",
  "ts": "1718367000000"
}
```

**Short Alert Schema:**
```json
{
  "type": "signal",
  "side": "SHORT",
  "symbol": "EURUSD",
  "tf": "15",
  "entry": "1.08500",
  "sl": "1.08800",
  "tp": "1.07600",
  "rr": "3.0",
  "ts": "1718367000000"
}
```

---

## Backtest Constraints
- **Flat-Only Execution**: The script operates flat-to-flat. No pyramiding or position building is permitted (`strategy.position_size == 0` check).
- **Daily Limits**: Rejects trades once the daily count reaches the user-specified maximum (e.g., `2` trades per day). Reset occurs daily at Vienna midnight.
