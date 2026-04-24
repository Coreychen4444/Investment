---
tags:
  - review
  - trading
  - execution
aliases:
  - 2026-04-24 三条执行教训
date: 2026-04-24
tickers:
  - NOK
  - SMR
  - AAOI
---

# 2026-04-24 Three Execution Lessons Review

Self-initiated review covering three execution mistakes across the 4/21-4/24 window. Consolidated because all three trace back to the same underlying pattern: **emotion overriding a pre-defined rule**.

## Summary

Three lessons extracted, each spawning durable rules:

1. **NOK 0501 $11C** — held through +119% peak, decayed to -33% (theta + IV crush on earnings gap-up fade). Rule: mandatory short-dated option profit-taking.
2. **SMR call** — impulsive entry into out-of-competence sector after physical order-entry mistake. Rule: circle of competence + hesitation = 24h stop.
3. **AAOI $156.50** — after-hours chase on catalyst alone with stale zone and zone upper-bound entry. Rule: catalyst ≠ confirmation, no after-hours initiations, no falling knife, no upper-bound chase.

---

## Case 1: NOK 0501 $11C — Missed Profit-Taking

### Timeline
- **2026-04-14**: Entered 10 contracts @ $0.21 ($210 cost) as probe on NOK breakout, ahead of 4/23 Q1 earnings catalyst.
- **2026-04-22 close**: NOK $9.86 (-5% de-risking selloff); option ~$0.26.
- **2026-04-23 open (earnings day)**: NOK gapped up to $10.76 on strong Q1 beat (AI/Cloud +49%, Optical +20%, guidance raised). Option spiked to $0.46 intraday high = **+119% from entry, unrealized $250 profit**.
- **2026-04-23 intraday fade**: NOK sold-the-news + IV crush. Closed $10.175. Option decayed to $0.14 late in session = **-33% from entry, -$70 if closed**.
- **Net swing**: $320 round-trip wiped out by not trimming at peak.

### What Went Wrong
- No profit-taking threshold was defined before earnings. Should have been: at +50% trim 30-50%, at +100% trim 50-70%.
- Earnings-play decision (trim / roll / hold) was never made at T-24h. Position was "held for more upside" without explicit plan.
- Theta decay on DTE < 14 accelerates non-linearly. Gap-up fade + IV crush compounded.

### Root Cause
**Greed disguised as conviction.** "It's going higher" is the oldest earnings-play trap. With DTE ≤ 14, the asymmetry flips against the holder after the event.

### Rule Extracted
> **Rule #30**: Short-dated option profit-taking is mandatory (not optional).
> - DTE ≤ 14 at +50% → trim 30-50%
> - DTE ≤ 14 at +100% → trim 50-70%
> - DTE ≤ 7: no naked hold
> - Earnings play: decide trim / roll / hold 24h before event

### Forward Action
5/1 $11C DTE is now 7. Per new Rule #30, no naked hold allowed. Options: close at ~$0.14 to realize -$70, or roll to longer expiry if thesis remains. "Wait and see" is no longer permitted.

---

## Case 2: SMR Call — Out-of-Competence Impulsive Entry

### Timeline
- Impulsive entry into SMR (nuclear power small-cap) after seeing sector strength.
- Order mis-entered (wrong limit parameter) on first attempt.
- Re-submitted on second attempt, overriding the mis-entry.

### What Went Wrong
- **Outside circle of competence.** Nuclear/energy is not in Corey's active research universe. Whitelist: AI optics, semis (HBM/memory), foundry, AI networks, Physical AI.
- **Physical hesitation ignored.** A mis-entered limit is the subconscious rejecting the trade. Re-submitting overrode that signal with FOMO.
- **No preparation.** No paper trade, no analyst reports, no cooling period.

### Root Cause
**FOMO on a sector rotation.** Nuclear/energy was a hot theme in the quarter; seeing others profit triggered a desire to participate without any real edge.

### Rule Extracted
> **Rule #32**: Whitelist = AI optics, semis, foundry, AI networks, Physical AI. Non-whitelist requires paper trade 1 month + ≥ 2 analyst reports + 3-day cooling period.
>
> **Rule #33**: Physical hesitation is a veto. Order-entry mistakes, wrong parameters, three price edits, or mis-clicks = 24h full stop. Body is more honest than brain.

### Forward Action
No active SMR position. Rule enforcement from 2026-04-24 onward. If nuclear/energy interest returns, start with paper trading for 1 month minimum.

---

## Case 3: AAOI $156.50 — After-Hours Catalyst Chase

### Timeline
- **2026-04-21 ET session**: AAOI hit new ATH $173.41 intraday, then crashed to $149.68 low on Iran ceasefire fears (Trump earlier said no extension). Broad market also down (S&P -0.63%, Nasdaq -0.59%). Closed $150.57 (-13.2% from high).
- **2026-04-21 post-market**: Trump reversed — announced ceasefire extension. Futures rallied.
- **2026-04-21 ET 21:58 (SGT 4/22 09:58) after-hours**: Corey bought 10 shares @ $156.50 as "trading lot" on catalyst-resolution logic.
- **Problem at time of entry**: Zone was stale (old trim $148-160 from 4/13). New zone1 of $150-156 had not yet been computed. Entry at $156.50 was at upper bound of the (yet-to-be-written) new zone1.
- **2026-04-22 regular session**: AAOI opened $155.50, intraday dropped to $138.62 (-11.4% vs prev close intraday), closed $149.42. Trading lot underwater -4.5%.

### What Went Wrong
- **Catalyst alone, no technical confirmation.** Trump's extension was a real catalyst, but there was no price-action confirmation when the entry was made.
- **After-hours entry.** Thin liquidity, unreliable prints, no real price discovery.
- **Stale zone misused.** Old zone was obsolete; new zone not yet defined. Entry made on "feel" against an undefined structure.
- **Zone upper-bound entry.** Even when the new zone ($150-156) was later written, $156.50 was at the upper bound — chasing, not accumulating.
- **Classic falling knife.** The entry day had -13.2% intraday move without a defended technical support level; buying that on catalyst-only logic is textbook knife-catching.

### Root Cause
**Catalyst-resolution FOMO.** The logic "the bad news is over, this will rally" is emotionally compelling but not a technical setup. In after-hours, with no live cash-session price action, it was a pure gut call.

### Rules Extracted
> **Rule #34**: Catalyst alone ≠ technical confirmation. Probe size (≤ 30%) only until technical confirmation in regular session.
>
> **Rule #35**: After-hours / pre-market: no new positions. Only defensive trims or pre-planned resting limits.
>
> **Rule #36**: Zone stale + catalyst → refresh zone before entry. No "feel" entries on obsolete structure.
>
> **Rule #37**: "Falling knife" default no-go. Definition: single-day |change| > 10% without clear technical support. Required to enter: next-session confirmation close OR precise support touch with volume reversal.
>
> **Rule #38**: Trading lot entry position within a zone: mid or lower half only. Upper-bound = chase.

### Forward Action
Position is now managed per existing trading-lot rules (stop $144 close basis, target $170-175, time stop 5/7). Review is about the **entry quality**, not the ongoing management. The trade may still work out, but the process was poor.

---

## Meta-lesson

All three mistakes share one pattern: **a pre-defined rule existed (or should have existed), and emotion overrode it**.

- NOK: "take profit at +100%" rule existed implicitly, ignored for greed.
- SMR: "stay in competence" rule existed, ignored for FOMO.
- AAOI: "no after-hours initiations" / "no zone upper-bound chase" rule should have been explicit, wasn't written down, and catalyst-resolution emotion filled the vacuum.

The countermeasure is not more willpower. It is **making the rules explicit, written, and self-triggering**, so that violating them becomes a conscious decision rather than an automatic emotional response. This is why today's work is encoded as Rules #30, #32-33, #34-38 in `trading-rules.md`.

---

## Rules added to doctrine

- `trading-rules.md` #30 — Short-dated option profit-taking
- `trading-rules.md` #32 — Circle of competence whitelist + cooling period
- `trading-rules.md` #33 — Physical hesitation = 24h stop
- `trading-rules.md` #34 — Catalyst ≠ technical confirmation
- `trading-rules.md` #35 — No after-hours initiations
- `trading-rules.md` #36 — Refresh stale zone before entry
- `trading-rules.md` #37 — Falling knife default no-go
- `trading-rules.md` #38 — Trading lot entry position = zone mid/lower

Cross-linked in:
- Local memory: `trading_lessons.md`
- Trade journal: `trade_journal.md` 2026-04-24 entries

---

> 📍 Navigation
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关规则：[[trading-rules]]、[[pre-trade-checklist]]、[[sell-put-rules]]、[[two-stage-entry-rules]]
> 相关案例：[[2026-04-01-perfect-price-trap-review]]、[[2026-03-17-gtc-day-early-entry-review]]
