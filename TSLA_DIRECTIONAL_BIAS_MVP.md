# TSLA_DIRECTIONAL_BIAS_MVP.md — RedwoodVest Bot Fast-Track Spec

**Status:** Fast-track after Phase 0.2 Clean Boot + Phase 0.3 Real Market Data Foundation  
**User:** Maggie only  
**Primary symbol:** TSLA  
**Purpose:** Intraday decision support for deciding whether the market setup is currently more favorable to a bullish, bearish, or no-trade bias.

---

## 1. Product Decision

RedwoodVest Bot needs a dedicated **TSLA Directional Bias Engine** for Maggie's personal intraday workflow.

The engine must NOT issue guaranteed trade instructions.

It should classify the current setup as one of:

- `LONG BIAS`
- `SHORT BIAS`
- `NO TRADE / MIXED`

and provide:

- directional score
- confidence
- supporting evidence
- opposing evidence
- key trigger levels
- invalidation levels
- options context
- market context
- event risk

---

## 2. Why This Is Fast-Tracked

Maggie uses TSLA options and intraday price action to judge potential high/low areas, but the missing decision layer is direction.

This feature is therefore moved ahead of the broader Options roadmap as a focused TSLA-only MVP.

Do NOT pull the whole Phase 2 roadmap forward.

Only implement what is necessary for this one TSLA decision-support workflow.

---

## 3. Command

```text
/tsla_bias
```

Optional alias:

```text
/bias TSLA
```

The first version only needs to support TSLA.

---

## 4. Required Real Data Inputs

### A. Price Structure

Prefer real-time or near-real-time:

- current price
- previous close
- session open
- intraday high / low
- 1m candles
- 3m candles
- VWAP
- opening range high / low
- previous day high / low
- short-term EMA / trend structure if implemented deterministically

### B. Volume / Liquidity

- current volume
- relative volume
- intraday volume trend
- dollar volume if useful

### C. Options Positioning

Use only if the provider supplies real data:

- call volume
- put volume
- call/put volume ratio
- call open interest
- put open interest
- major call OI strikes
- major put OI strikes
- implied volatility
- IV skew if supported
- unusual options flow if supported
- call wall
- put wall
- Max Pain
- nearest relevant expiration
- gamma exposure / dealer positioning if a reliable provider is available

### D. Market Context

At minimum:

- QQQ direction
- SPY direction
- VIX direction or level
- Nasdaq / technology tone if available

### E. Event Risk

- TSLA earnings proximity
- major TSLA news
- major scheduled macro event
- major options expiration

---

## 5. No-Fake-Data Rule

If any required field is not available:

```text
資料暫無
```

Do not fill values with estimates.

Do not ask AI to infer missing options data.

Do not use mock data in production.

---

## 6. Directional Score

Create a deterministic score from:

```text
-100 = strongest bearish bias
0    = mixed / no edge
+100 = strongest bullish bias
```

Recommended component structure for MVP:

```text
Price Structure        30 points
Options Flow           25 points
OI / Gamma Structure   20 points
Volume / Momentum      15 points
Market Context         10 points
-------------------------------
Raw Directional Score 100 points
```

Apply a separate **risk/confidence adjustment** rather than allowing AI to alter the score.

The exact weights may be versioned and adjusted later.

Example:

```text
score_version = "tsla-bias-v1"
```

---

## 7. Example Deterministic Rules

These are implementation guidance, not final calibrated weights.

### Bullish evidence examples

- price above VWAP
- price holds above opening range high
- 1m and 3m structure both rising
- relative volume elevated
- call flow materially stronger than put flow
- important call strikes are being accepted above, not rejected
- put wall provides nearby support
- QQQ/SPY confirm direction

### Bearish evidence examples

- price below VWAP
- breakdown below opening range low
- 1m and 3m structure both falling
- elevated sell-side volume
- put flow materially stronger than call flow
- heavy call wall overhead
- support / put wall breaks
- QQQ/SPY confirm downside

### Mixed / no-trade examples

- price repeatedly crosses VWAP
- 1m and 3m disagree
- price is trapped between major call and put walls
- options flow conflicts with price structure
- relative volume is weak
- high-impact event is imminent
- critical data is missing

---

## 8. Confidence

Confidence must reflect:

- data completeness
- agreement between signals
- data freshness
- event risk
- provider quality

Suggested:

```text
High
Medium
Low
```

If critical options data is missing, do not output High confidence.

---

## 9. Trigger & Invalidation Logic

The Bot should not say:

```text
Buy TSLA now
Short TSLA now
```

Instead output conditional setups:

### Example bullish setup

```text
Bullish trigger:
Price holds above VWAP and reclaims opening-range high with volume confirmation.

Invalidation:
Price loses VWAP and breaks opening-range low.
```

### Example bearish setup

```text
Bearish trigger:
Price loses VWAP and opening-range low while put-side activity expands.

Invalidation:
Price reclaims VWAP and holds above the nearest resistance / call-wall zone.
```

Exact prices must come from live data.

---

## 10. High / Low Reference Zones

To help Maggie judge intraday highs and lows, compute / display:

- VWAP
- opening range high / low
- previous day high / low
- session high / low
- Max Pain
- call wall
- put wall
- major OI strikes
- expected move boundaries if available

Do not label any level as guaranteed high or guaranteed low.

Use labels such as:

- resistance zone
- support zone
- options magnet
- potential rejection zone
- potential support zone

---

## 11. Telegram Output

Recommended compact output:

```text
🎯 TSLA Directional Bias

Bias: LONG BIAS
Directional Score: +64 / 100
Confidence: Medium

Price: ...
VWAP: ...
OR High / Low: ...
Relative Volume: ...

OPTIONS
Max Pain: ...
Call Wall: ...
Put Wall: ...
Put/Call: ...
IV: ...

✅ Bullish evidence
• ...
• ...

⚠️ Bearish / opposing evidence
• ...
• ...

TRIGGER
• ...

INVALIDATION
• ...

KEY ZONES
Resistance: ...
Support: ...

Market context:
QQQ ...
SPY ...
VIX ...

Data time: ...
Sources: ...
```

---

## 12. Alert Mode

After the manual command is stable, add optional alerts:

```text
/tsla_bias_alert on
/tsla_bias_alert off
```

Possible alert conditions:

- bias changes LONG → SHORT
- bias changes SHORT → LONG
- score crosses +50
- score crosses -50
- Max Pain distance changes materially
- price crosses VWAP with options confirmation
- call wall / put wall materially changes
- confidence changes Low → High

Do not spam.

Use cooldown + material-change threshold.

---

## 13. AI Role

AI may:

- explain why the score is bullish / bearish / mixed
- summarize conflicting evidence
- translate structured metrics into readable language

AI may not:

- create the score
- create levels
- create option values
- invent a catalyst
- override deterministic calculations

---

## 14. Data Contract

AI receives a structured payload similar to:

```json
{
  "symbol": "TSLA",
  "score_version": "tsla-bias-v1",
  "directional_score": 64,
  "bias": "LONG_BIAS",
  "confidence": "medium",
  "price_structure": {},
  "volume": {},
  "options": {},
  "levels": {},
  "market_context": {},
  "events": [],
  "bullish_evidence": [],
  "bearish_evidence": [],
  "data_quality": {},
  "sources": [],
  "calculated_at": ""
}
```

---

## 15. Implementation Order

Do not implement this before the Bot can safely boot and fetch real market data.

Fast-track sequence:

```text
0.2 Clean Boot
↓
0.3 Real Market Data Foundation
↓
TSLA Directional Bias MVP
↓
Watchlist / broader Phase 1
↓
Full Options Intelligence later
```

This feature may reuse only the minimum options-provider work required for TSLA.

---

## 16. Acceptance Criteria

The feature is accepted only if:

- [ ] `/tsla_bias` returns real TSLA data
- [ ] score is deterministic
- [ ] no AI-generated market numbers
- [ ] missing options data lowers confidence
- [ ] price / options / context are timestamped
- [ ] outputs LONG BIAS / SHORT BIAS / NO TRADE
- [ ] shows bullish and bearish evidence
- [ ] shows trigger and invalidation conditions
- [ ] shows support / resistance reference zones
- [ ] Max Pain is real or unavailable
- [ ] no production mock data
- [ ] provider failure does not fabricate a result
- [ ] AI can be disabled and deterministic output still works
