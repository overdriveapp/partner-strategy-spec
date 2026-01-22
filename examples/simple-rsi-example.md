# Overdrive Strategy Logic Template
*Partner Strategy Specification (MVP Integration)*

---

## 1. Strategy Overview

**Strategy Name:**
Simple RSI Momentum

**Strategy Type:**
Momentum

**Market Type:**
- [x] Perpetuals
- [ ] Spot
- [ ] Other (please specify)

**Assets / Markets:**
- BTC-PERP

**Timeframe(s):**
1H (hourly candles)

**Objective:**
Enter momentum trades when RSI indicates strong directional conviction, using simple fixed risk parameters.

---

## 2. Core Strategy Concept

**Market Behavior Targeted:**
When RSI shows strong momentum (very high or very low readings), price often continues in that direction for several hours.

**Why This Edge Exists:**
Strong RSI readings indicate genuine momentum rather than just noise. Other traders see the same signal and pile in, creating self-fulfilling momentum.

**Performance Conditions:**
- **Best:** Trending markets with clear directional bias
- **Worst:** Choppy/sideways markets with no sustained momentum

---

## 3. Signal Generation Logic

### Step-by-step Logic

**Step 1: Calculate RSI**
- **Inputs:** 14-period RSI on 1H candles
- **Conditions:** Wait for candle close
- **Outputs:** RSI value (0-100)

**Step 2: Check RSI Thresholds**
- **Long Signal:** RSI > 65 (bullish momentum)
- **Short Signal:** RSI < 35 (bearish momentum)
- **Invalidation:** If RSI between 35-65, no signal

**Step 3: Emit Signal**
- Direction determined by RSI level
- Entry at current close price
- If RSI out of range → no signal

---

## 4. Entry & Execution Rules

**Trade Direction:**
- [x] Long
- [x] Short
- [x] Both

**Entry Logic:**
Market order at signal generation (hourly candle close)

**Execution Preferences:**
- Immediate execution
- Signal validity: 1 hour (next candle close)

---

## 5. Risk Management & Position Sizing

**Position Sizing Method:**
Fixed 10% of portfolio per trade

**Leverage:**
Fixed 3x leverage

**Stop Loss Logic:**
Fixed percentage: 2% from entry
- Long: stopLoss = entry × 0.98
- Short: stopLoss = entry × 1.02

**Take Profit Logic:**
Fixed 1:2 risk/reward ratio: 4% from entry
- Long: takeProfit = entry × 1.04
- Short: takeProfit = entry × 0.96

**Maximum Holding Period:**
12 hours (12 candles)

---

## 6. Confidence / Signal Quality (Optional but Recommended)

**Simplified Scoring:**
- RSI > 70 or < 30: High confidence (0.8)
- RSI 65-70 or 30-35: Medium confidence (0.6)

All signals above minimum threshold are emitted.

---

## 7. Signal Output (Conceptual)

**Example Signal:**
```json
{
  "asset": "BTC-PERP",
  "direction": "long",
  "entryPrice": 65000,
  "stopLossPrice": 63700,
  "takeProfitPrice": 67600,
  "leverage": 3,
  "confidence": 0.8,
  "timestamp": 1706000000000,
  "reason": "LONG momentum - RSI 72 indicates strong bullish momentum"
}
```

---

## 8. Data Requirements

**Required Data Sources:**
- OHLC price data (1H candles)
- Minimum 14 candles for RSI calculation

**Minimum Historical Lookback:**
- 20 candles recommended (14 minimum)

**Update Frequency:**
- Hourly (on candle close)

---

## 9. Assumptions & Constraints

**Assumptions:**
- Sufficient liquidity on BTC perpetuals
- 1H signals have enough time to develop
- Fixed 2%/4% stops/targets are reasonable for typical BTC volatility

**Known Limitations:**
- Simple fixed stops may be too tight in high volatility
- No trend filter, may enter counter-trend trades
- Single indicator means less confirmation

---

## 10. Notes for MVP Scope

**Why This Strategy is Simple:**
- Single indicator (RSI only)
- Fixed risk parameters (no calculations needed)
- No complex confidence scoring
- Easy to understand and validate

**MVP Implementation:**
- Fully defined logic
- No edge cases or special handling
- Standard RSI calculation (any TA library)

**Future Enhancements:**
- Add volume filter
- Dynamic stops based on ATR
- Trend filter to avoid counter-trend trades

---

### Quick Reference

| Parameter | Value |
|-----------|-------|
| Indicator | RSI(14) |
| Long Entry | RSI > 65 |
| Short Entry | RSI < 35 |
| Stop Loss | 2% fixed |
| Take Profit | 4% fixed |
| Leverage | 3x |
| Timeframe | 1H |
