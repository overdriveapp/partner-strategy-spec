# Overdrive Strategy Logic Template
*Partner Strategy Specification (MVP Integration)*

---

## 1. Strategy Overview

**Strategy Name:**
(Descriptive name)

**Strategy Type:**
(e.g. momentum, mean reversion, breakout, arbitrage, hybrid)

**Market Type:**
- [ ] Perpetuals
- [ ] Spot
- [ ] Other (please specify)

**Assets / Markets:**
(List symbols, pairs, or market identifiers)

**Timeframe(s):**
(e.g. 1D, 4H, 1H, mixed)

**Objective:**
(High-level description of what the strategy is designed to exploit)

---

## 2. Core Strategy Concept

Provide a concise explanation of:
- the market behavior you are targeting
- why this edge should exist
- under what conditions the strategy performs best / worst

> Example:
> “The strategy identifies volatility contraction followed by directional expansion, based on the assumption that extended low-volatility regimes precede strong trend moves.”

---

## 3. Signal Generation Logic

Describe the **logical steps** required to generate a signal.
This does **not** need to be code, but should be precise.

### Step-by-step Logic
For each step, specify:
- inputs
- conditions
- outputs
- invalidation criteria (when no signal is produced)

**Suggested structure:**
1. Detect condition A
2. Confirm condition B
3. Determine trade direction
4. Validate signal quality
5. Emit signal or discard

You may use:
- technical indicators
- on-chain metrics
- funding / basis data
- external signals
- rule-based logic

---

## 4. Entry & Execution Rules

**Trade Direction:**
- [ ] Long
- [ ] Short
- [ ] Both

**Entry Logic:**
(e.g. market on signal, limit near X, conditional)

**Execution Preferences (if any):**
(e.g. immediate execution, price bounds, time validity)

> _Note: For MVP, execution may be simplified or abstracted by Overdrive._

---

## 5. Risk Management & Position Sizing

**Position Sizing Method:**
(e.g. fixed %, volatility-based, units-based)

**Leverage:**
(fixed or dynamic)

**Stop Loss Logic:**
(e.g. indicator-based, % based, invalidation-based)

**Take Profit Logic:**
(e.g. fixed R:R, trailing, partial exits)

**Maximum Holding Period (if applicable):**

---

## 6. Confidence / Signal Quality (Optional but Recommended)

If applicable, describe how signal quality or confidence is assessed:
- scoring system
- filters
- thresholds

This may be used for:
- ranking signals
- user-facing transparency
- performance attribution

---

## 7. Signal Output (Conceptual)

Each signal should conceptually include:
- asset / market
- direction
- entry reference
- stop reference
- target reference (if any)
- leverage
- confidence (if applicable)
- timestamp
- brief human-readable explanation

_(Exact schema handled by Overdrive.)_

---

## 8. Data Requirements

Please list:
- required data sources (price, volume, funding, etc.)
- minimum historical lookback
- update frequency expectations

---

## 9. Assumptions & Constraints

List any assumptions the strategy relies on:
- liquidity expectations
- execution latency tolerance
- market availability
- known limitations

---

## 10. Notes for MVP Scope

This initial integration is intended as an **MVP**:
- strategy logic should be well-defined but pragmatic
- Overdrive will assess feasibility and propose implementation tradeoffs
- more advanced features can be iterated post-MVP

---

## ✅ Pre-Submission Checklist

Before sending this strategy to Overdrive, please verify:

**Essential Information:**
- [ ] Strategy name and type are clearly defined
- [ ] Market type and assets are specified
- [ ] Timeframe(s) are indicated

**Strategy Logic:**
- [ ] Core concept is explained (why this edge should exist)
- [ ] Signal generation has step-by-step logic
- [ ] Entry/exit rules are unambiguous
- [ ] All indicator parameters include specific numbers (periods, thresholds, multipliers)

**Risk Management:**
- [ ] Stop loss logic is defined
- [ ] Take profit or exit logic is specified
- [ ] Position sizing method is stated
- [ ] Leverage amount is specified

**Data & Constraints:**
- [ ] Required data sources are listed
- [ ] Minimum lookback period is specified
- [ ] Known limitations or assumptions are documented

**Quality Check:**
- [ ] I've reviewed at least one example strategy
- [ ] All sections are filled out (not just placeholders)
- [ ] Logic can be understood by someone unfamiliar with my strategy

---

### How to Use This Template

- Fill out each section as clearly as possible
- The more precise the logic, the faster implementation can proceed
- Overdrive will handle translation into production execution where feasible
