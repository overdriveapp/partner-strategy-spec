# Overdrive Strategy Logic Template
*Partner Strategy Specification (MVP Integration)*

---

## 1. Strategy Overview

**Strategy Name:**
Breakout-Squeeze Strategy

**Strategy Type:**
Breakout / Momentum

**Market Type:**
- [x] Perpetuals
- [ ] Spot
- [ ] Other (please specify)

**Assets / Markets:**
- SPX (S&P 500)
- DAX (German Index)
- BTC (Bitcoin)
- SOL (Solana)

**Timeframe(s):**
1D (Daily candles)

**Objective:**
Identify periods of low volatility (squeeze) followed by directional price breakouts. The strategy exploits the market tendency for compressed volatility regimes to precede explosive directional moves.

---

## 2. Core Strategy Concept

**Market Behavior Targeted:**
The strategy identifies volatility contraction (measured via Bollinger Band width) followed by directional expansion (breakout through Donchian Channel boundaries).

**Why This Edge Exists:**
Extended low-volatility regimes create a "coiled spring" effect where market participants accumulate positions during consolidation. When price breaks out of this range, it often leads to sustained directional moves as trapped traders exit and momentum traders enter.

**Performance Conditions:**
- **Best:** Trending markets following consolidation periods, high volume breakouts
- **Worst:** Choppy/ranging markets with frequent false breakouts, low liquidity environments

---

## 3. Signal Generation Logic

### Step-by-step Logic

**Step 1: Detect Volatility Squeeze**
- **Inputs:** 20-period Bollinger Bands (2 standard deviations)
- **Conditions:** Current BB width < 20-period average BB width
- **Outputs:** Squeeze detected (boolean), squeeze strength (0-1)
- **Invalidation:** If no squeeze detected, discard signal
- **Implementation Requirement:** Uses *previous* bar's BB width to prevent the breakout candle from affecting squeeze calculation (lookahead prevention)

**Step 2: Identify Breakout Direction**
- **Inputs:** 20-period Donchian Channel (highest high / lowest low)
- **Conditions:**
  - Long: Current close > Donchian high (20-day highest high)
  - Short: Current close < Donchian low (20-day lowest low)
- **Outputs:** Direction (long/short), breakout strength (% beyond channel)
- **Invalidation:** If price within Donchian channel, discard signal
- **Implementation Requirement:** Donchian channel is calculated from the previous 20 bars, excluding the current bar, to prevent the breakout candle from defining the channel it breaks (lookahead prevention)

**Step 3: Calculate Stop Loss**
- **Inputs:** EMA 20, current price
- **Conditions:** Stop must be on opposite side of entry with minimum 0.5% distance
- **Outputs:** Stop loss price
- **Logic:**
  - Long: Stop = max(EMA 20, entry - 0.5%)
  - Short: Stop = min(EMA 20, entry + 0.5%)

**Step 4: Calculate Take Profit**
- **Inputs:** Entry price, stop loss price
- **Conditions:** 1:3 risk/reward ratio
- **Outputs:** Take profit price
- **Logic:**
  - Risk = |entry - stop|
  - Reward = Risk × 3
  - Long TP = entry + reward
  - Short TP = entry - reward

**Step 5: Calculate Confidence Score**
- **Inputs:** Squeeze strength, breakout strength, volume ratio (current/20-day avg)
- **Conditions:** Minimum confidence threshold of 60%
- **Outputs:** Confidence score (0.5 - 0.95)
- **Scoring:**
  - Base: 50%
  - Squeeze bonus: +5% (mild) / +10% (moderate) / +15% (tight)
  - Breakout bonus: +5% (0.5%+) / +10% (1%+) / +15% (2%+)
  - Volume bonus: +10% (1.2x avg) / +15% (1.5x avg)
- **Invalidation:** If confidence < 60%, discard signal

**Step 6: Emit Signal**
- All conditions met → Generate signal with full parameters
- Any condition failed → No signal emitted

---

## 4. Entry & Execution Rules

**Trade Direction:**
- [x] Long
- [x] Short
- [x] Both

**Entry Logic:**
Market order on signal generation (daily close when breakout confirmed during squeeze)

**Execution Preferences:**
- Signal validity: 24 hours from generation
- Immediate execution preferred
- No price bounds (market entry at signal price)

---

## 5. Risk Management & Position Sizing

**Position Sizing Method:**
Fixed percentage allocation: 10% of portfolio per trade

**Leverage:**
Fixed 5x leverage

**Stop Loss Logic:**
Indicator-based: EMA 20 with minimum 0.5% distance from entry
- Long: Stop placed below EMA 20 (or entry - 0.5% if EMA is too close)
- Short: Stop placed above EMA 20 (or entry + 0.5% if EMA is too close)

**Take Profit Logic:**
Fixed R:R ratio of 1:3
- Take profit = entry ± (3 × risk distance)

**Maximum Holding Period:**
10 days (time-based exit if neither stop nor target hit)

---

## 6. Confidence / Signal Quality

**Scoring System:**
Composite score based on three factors:

| Factor | Threshold | Bonus |
|--------|-----------|-------|
| Squeeze Strength | >0% | +5% |
| Squeeze Strength | >15% | +10% |
| Squeeze Strength | >30% | +15% |
| Breakout Strength | >0.5% | +5% |
| Breakout Strength | >1% | +10% |
| Breakout Strength | >2% | +15% |
| Volume Ratio | >1.2x | +10% |
| Volume Ratio | >1.5x | +15% |

**Thresholds:**
- Minimum confidence to emit signal: 60%
- Maximum confidence cap: 95%

**Usage:**
- Signals sorted by confidence (highest first)
- User-facing transparency on signal quality
- Performance attribution by confidence tier

---

## 7. Signal Output (Conceptual)

Each signal includes:

```
{
  asset: "BTC" | "SOL" | "SPX" | "DAX"
  direction: "long" | "short"
  entryPrice: number (current close price)
  stopLossPrice: number (EMA 20 based)
  takeProfitPrice: number (1:3 R:R)
  leverage: 5
  confidence: 0.60 - 0.95
  timestamp: Unix timestamp (ms)
  reason: "LONG breakout during squeeze. Squeeze strength: X%, Breakout: Y%"

  // Detailed metrics
  squeeze: {
    bbWidth: number
    bbAvgWidth: number
    squeezeStrength: number
  }
  breakout: {
    donchianHigh: number
    donchianLow: number
    breakoutStrength: number
  }
}
```

---

## 8. Data Requirements

**Required Data Sources:**
- OHLCV price data (Open, High, Low, Close, Volume)
- Daily candles for all supported assets

**Minimum Historical Lookback:**
- 50 candles minimum for indicator calculation
- 2 years recommended for robust indicator values (especially EMA 200)

**Update Frequency:**
- Daily analysis (end of day)
- Signal generation once per day after market close

**External Data Sources:**
- Yahoo Finance API (or equivalent) for price feeds
- No on-chain data required for MVP

---

## 9. Assumptions & Constraints

**Liquidity Expectations:**
- Sufficient liquidity on perpetual DEX (Ostium on Arbitrum)
- Assets traded are major indices and top cryptocurrencies

**Execution Latency Tolerance:**
- Daily timeframe allows for relaxed execution (minutes, not seconds)
- Signals valid for 24 hours from generation

**Market Availability:**
- 24/7 availability for crypto assets (BTC, SOL)
- Index assets (SPX, DAX) subject to market hours for reference pricing

**Known Limitations:**
- False breakouts in choppy markets may generate losing signals
- Squeeze conditions may persist without breakout for extended periods
- Volume data quality varies by data source
- Strategy performs best in trending environments

---

## 10. Notes for MVP Scope

**MVP Implementation:**
- Strategy logic is fully defined and production-ready
- All indicator calculations are standard technical analysis
- Signal schema aligns with Overdrive's expected format

**Suggested Simplifications:**
- Start with BTC and SOL only (24/7 markets)
- Use fixed 5x leverage initially
- Time-based exits can be manual for MVP

**Future Iterations:**
- Dynamic leverage based on volatility
- Multiple timeframe confirmation
- Trailing stop implementation
- Partial position exits at intermediate targets
- Additional assets (ETH, major forex pairs)

---

### Technical Indicators Summary

| Indicator | Period | Usage |
|-----------|--------|-------|
| Bollinger Bands | 20, 2σ | Squeeze detection |
| Donchian Channel | 20 | Breakout identification |
| EMA | 20 | Stop loss placement |
| Volume SMA | 20 | Confidence scoring |

---
