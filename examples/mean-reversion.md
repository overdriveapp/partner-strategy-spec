# Overdrive Strategy Logic Template
*Partner Strategy Specification (MVP Integration)*

---

## 1. Strategy Overview

**Strategy Name:**
RSI-Bollinger Mean Reversion

**Strategy Type:**
Mean reversion

**Market Type:**
- [x] Perpetuals
- [ ] Spot
- [ ] Other (please specify)

**Assets / Markets:**
- BTC-PERP (primary)
- ETH-PERP (secondary)

**Timeframe(s):**
4H (primary signal generation)

**Objective:**
Capture price reversals when markets reach extreme overbought or oversold conditions, betting on price returning to its statistical mean.

---

## 2. Core Strategy Concept

The strategy identifies when price has deviated significantly from its statistical mean and bets on a reversion back toward equilibrium.

**Market behavior targeted:**
- Crypto markets frequently overshoot in both directions due to high volatility and emotional trading
- After extreme moves, price tends to revert toward moving averages
- RSI extremes combined with Bollinger Band touches provide high-probability reversal setups

**Why this edge should exist:**
- Retail traders chase momentum at extremes, creating temporary mispricings
- Funding rates at extremes create economic pressure for price to revert
- Liquidity imbalances at extremes attract mean reversion traders

**Best conditions:**
- Range-bound or choppy markets
- High RSI readings (>75) or low RSI readings (<25)
- Price touching or exceeding Bollinger Bands

**Worst conditions:**
- Strong trending markets with sustained momentum
- News-driven moves with fundamental shifts
- Low liquidity / weekend markets

---

## 3. Signal Generation Logic

### Step-by-step Logic

**Step 1: Calculate Indicators**
- Inputs: Last 50 candles (4H timeframe)
- Calculate: RSI(14), Bollinger Bands (20-period SMA, 2 standard deviations)
- Outputs: rsi, bbUpper, bbLower, bbMiddle (SMA20), bbWidth

**Step 2: Detect Extreme RSI Condition**
- Condition A (Oversold): RSI < 25
- Condition B (Overbought): RSI > 75
- If neither condition met → no signal, exit early
- Output: extremeType ('oversold' | 'overbought')

**Step 3: Confirm with Bollinger Band Touch**
- For oversold: price must be at or below bbLower (close <= bbLower * 1.005)
- For overbought: price must be at or above bbUpper (close >= bbUpper * 0.995)
- If BB confirmation fails → no signal, exit early
- Output: bbConfirmed (boolean)

**Step 4: Check Trend Filter (Invalidation)**
- Calculate EMA50 for trend context
- For oversold/long entry: price should not be more than 15% below EMA50 (avoid catching falling knives)
- For overbought/short entry: price should not be more than 15% above EMA50
- If trend filter fails → no signal, exit early

**Step 5: Determine Trade Direction**
- Oversold + BB confirmation → direction = 'long'
- Overbought + BB confirmation → direction = 'short'

**Step 6: Calculate Signal Quality / Confidence**
- RSI extremity: how far beyond threshold (e.g., RSI 20 is more extreme than RSI 24)
- BB penetration depth: how far beyond the band
- Combined into confidence score (0-1)

**Step 7: Emit Signal or Discard**
- If confidence >= 0.5 → emit signal
- If confidence < 0.5 → discard (weak setup)

---

## 4. Entry & Execution Rules

**Trade Direction:**
- [x] Long
- [x] Short
- [x] Both

**Entry Logic:**
- Market order on signal confirmation
- Entry price = current close price at signal generation

**Execution Preferences:**
- Immediate execution preferred
- Price tolerance: within 0.3% of signal price
- Signal validity: 4 hours (one candle period)

> _Note: For MVP, execution may be simplified or abstracted by Overdrive._

---

## 5. Risk Management & Position Sizing

**Position Sizing Method:**
Fixed percentage of account (managed by user settings in Overdrive)

**Leverage:**
5x (fixed for MVP)

**Stop Loss Logic:**
- Long trades: stopLoss = entryPrice - (2.5 × ATR14)
- Short trades: stopLoss = entryPrice + (2.5 × ATR14)
- Minimum stop distance: 1.5% from entry
- Maximum stop distance: 5% from entry (cap for extreme volatility)

**Take Profit Logic:**
- Primary target: BB middle line (SMA20) for 1:1.5 R:R minimum
- If R:R to middle band < 1.5, extend target to opposite band for better R:R
- Partial exit: 50% at BB middle, 50% trails with EMA9

**Maximum Holding Period:**
10 candles (40 hours on 4H timeframe) - time-based exit if neither stop nor target hit

---

## 6. Confidence / Signal Quality (Optional but Recommended)

**Scoring System (0-1 scale):**

```
rsiScore = (thresholdDistance / 25) capped at 0.4
  - For oversold: thresholdDistance = 25 - RSI (e.g., RSI 15 → score = 10/25 = 0.4)
  - For overbought: thresholdDistance = RSI - 75 (e.g., RSI 85 → score = 10/25 = 0.4)

bbScore = (bbPenetration / 0.02) capped at 0.3
  - bbPenetration = how far price exceeded the band as % of price

volumeScore = min(volumeRatio / 2, 0.3)
  - Higher volume at extremes = more conviction

confidence = rsiScore + bbScore + volumeScore
```

**Minimum Threshold:**
- Confidence >= 0.5 required to emit signal
- Higher confidence = priority for execution if multiple signals

---

## 7. Signal Output (Conceptual)

Each signal should conceptually include:
- **asset**: "BTC-PERP" or "ETH-PERP"
- **direction**: "long" | "short"
- **entryPrice**: current close price
- **stopLossPrice**: calculated using ATR method
- **takeProfitPrice**: BB middle line or extended target
- **leverage**: 5
- **confidence**: 0.5 - 1.0 scale
- **timestamp**: Unix timestamp in ms
- **reason**: Human-readable explanation

**Example Signal:**
```json
{
  "direction": "long",
  "asset": "BTC-PERP",
  "assetSymbol": "BTC",
  "entryPrice": 62500,
  "stopLossPrice": 61250,
  "takeProfitPrice": 64500,
  "leverage": 5,
  "confidence": 0.72,
  "timestamp": 1706000000000,
  "reason": "LONG mean reversion. RSI: 22 (oversold), price at lower BB. Target: SMA20.",
  "indicators": {
    "rsi": 22,
    "bbLower": 62600,
    "bbMiddle": 64500,
    "bbUpper": 66400,
    "atr14": 500
  }
}
```

_(Exact schema handled by Overdrive.)_

---

## 8. Data Requirements

**Required data sources:**
- OHLCV price data (Open, High, Low, Close, Volume)
- Minimum 50 candles for indicator calculation

**Minimum historical lookback:**
- 50 periods (200 hours / ~8 days on 4H timeframe)
- RSI requires 14 periods
- Bollinger Bands require 20 periods
- ATR requires 14 periods
- EMA50 requires 50 periods

**Update frequency:**
- Signal generation: Every 4 hours (on candle close)
- Price monitoring: Real-time for execution

---

## 9. Assumptions & Constraints

**Assumptions:**
- Markets exhibit mean-reverting behavior at extremes
- RSI and BB are reliable indicators of short-term exhaustion
- Sufficient liquidity exists at major support/resistance levels
- Funding rates don't dominate trade economics at 5x leverage

**Liquidity expectations:**
- BTC: >$100M daily volume on exchange
- ETH: >$50M daily volume on exchange

**Execution latency tolerance:**
- Signal valid for up to 4 hours
- Entry within 0.3% of signal price acceptable

**Known limitations:**
- Will underperform in strong trending markets
- Multiple consecutive losses possible during trend breakouts
- Requires discipline to hold through drawdowns
- Not suitable for news-driven / event markets

---

## 10. Notes for MVP Scope

**MVP Implementation:**
- Start with BTC-PERP only (single asset simplifies testing)
- Fixed 5x leverage (user cannot override initially)
- Basic confidence scoring (can be refined post-MVP)
- Time-based exit as safety mechanism

**Post-MVP Enhancements:**
- Add ETH-PERP and other liquid perpetuals
- Dynamic leverage based on confidence
- Trailing stop optimization
- Funding rate integration (avoid shorts when funding highly negative)
- Multi-timeframe confirmation (4H signal + 1H entry timing)

**Implementation Notes:**
- Indicator calculations should match standard TA library implementations
- RSI uses Wilder's smoothing method (exponential)
- Bollinger Bands use simple moving average with 2 standard deviations
- ATR uses true range with 14-period smoothing
