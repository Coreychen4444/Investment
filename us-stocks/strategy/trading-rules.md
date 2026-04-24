---
tags:
  - trading
  - framework
aliases:
  - Trading Rules
  - 交易规则
  - Iron Rules
---

# Trading Rules

Last major update: 2026-04-16. Absorbs event-day-trading-rules, opening-discipline, anti-fomo-rules into one master file. All source citations preserved.

## Core purpose
Protect against FOMO, early entry, one-size-fits-all actions, emotional trades, and confusing correct logic with correct timing.

---

## Entry rules

1. Every trade must be labeled as one of:
   - **Confirmed** = clear confirmation signal (support hold, sector resonance, fundamental validation)
   - **Probe** = direction right but unconfirmed, light size only
   - **Early/risky** = unconfirmed and wants full size
   If you cannot label it clearly, do not trade.

2. No confirmation = no size.
   Unconfirmed trades are probe trades only.

3. Before entry, answer:
   - What type of trade is this?
   - What is the catalyst?
   - What proves me wrong?
   - Where do I reduce/exit if wrong?

4. Same thesis: maximum 2 entries total.
   No third add. _Iron Rule #2_
   Override requires explicit acknowledgment with reasoning documented in trade journal.

5. Confirmation signal checklist — at least 2 of:
   - Price holds key support / accumulation zone
   - Sector and index not simultaneously weakening
   - Not a single-headline spike; sustained buying
   - Opening 30-min settled, price behavior stable
   For quantitative bottom signals, reference `bottom-confirmation-signals.md`.
   For full indicator roles and SOP, reference `technical-indicators-framework.md`.

---

## Exit rules

6. Exits are for one of three reasons:
   - Thesis/event has been priced
   - Structure has weakened
   - Time window ended

7. Weak rebound is not proof of recovery.
   It may be a risk-reduction window.

8. If timing was wrong, reduce risk before arguing with the market.

---

## Position rules

9. Size must match certainty.
   High-beta names cannot be handled like core thesis names.

10. Separate core expressions from sentiment expressions.
    - Core / harder-expression beneficiaries → hold through noise
    - High-beta / sentiment-expression beneficiaries → tighter risk management
    _Source: subtheme hierarchy, GTC day review, 2026-03-17_

---

## Anti-FOMO rules

11. Confirmation matters more than pretty prices.
    A worse price for better confirmation is acceptable, not failure.
    _Source: catalyst day early entry review, 2026-03-17_

12. Missing part of a move is not failure.

13. The impossible trio does not exist:
    Buy the lowest + sell the highest + capture the full move.
    Attempting all three leads to early entries, delayed exits, fear-based actions.
    _Source: anti-fomo-rules, 2026-03-17_

14. Do not move a reasonable first buy limit lower just to chase a more perfect entry.
    _Source: perfect-price trap review, 2026-04-01_

15. If the first planned price is still valid, keep it and add a second lower tranche instead of replacing it.
    _Source: perfect-price trap review, 2026-04-01_

16. The first tranche exists to secure participation; the second tranche exists to exploit deeper weakness.

---

## Opening discipline

17. No trading in the first 30 minutes after the open (ET 09:30-10:00).
    Exception: explicit risk reduction (stop loss, position cut).
    _Source: opening-discipline, 2026-03_

18. Set the stop before the trade.
    Define the stop threshold in advance. Execute without hesitation if triggered.
    _Source: opening-discipline, 2026-03_

---

## Event-day rules

19. High-expectation catalyst days (GTC, earnings, keynotes): early strength is not confirmation.
    Opening strength, premarket momentum does not mean the market has finished pricing.
    _Source: GTC day early entry, 2026-03-17_

20. Wait for digestion before sizing high-beta subthemes.
    Do not size aggressively before the market has digested the content.

21. Thesis validation != same-day upside.
    A catalyst can strengthen long-term logic while triggering same-day sell-the-news in crowded subthemes.
    _Source: sell-the-news rule, 2026-03-17_

22. Before trading a catalyst day, ask:
    - Is this move driven by pre-event expectation or post-event digestion?
    - Am I buying the hardest expression or the highest-beta expression?
    - If the catalyst is merely "good, not shocking," does this stock still need to go up today?
    - What happens if the market validates the thesis but rotates out of the subtheme?
    _Source: event-day-trading-rules, 2026-03-17_

---

## Zone discipline

23. Zones are not eternal. When a stock structurally breaks out of its old range, update zones before trading — do not mechanically block a trade based on stale zones.
    - If the stock has moved decisively above old zones and the thesis supports the new level, update first, then trade.
    - Zone violations should still be flagged, but with the question: "Are the zones themselves stale?"
    _Source: zone staleness incident, 2026-04-14_

24. Zone checks are mandatory before entry:
    - In accumulation zone → proceed
    - In waiting zone → need stronger confirmation
    - Above no_chase_above → hard stop, do not chase
    - In trim zone wanting to buy → hard stop unless zones are explicitly outdated and being updated

---

## Event-risk reduction

25. When multiple risks stack (earnings + FOMC + macro), reduce portfolio fragility first.
    - Cut amplifiers (leveraged, high-beta) before core positions
    - Goal: "If worst case hits, I survive" — not predicting the bad scenario
    - Keep core position for upside participation
    Reference `event-risk-reduction-principle.md` for full framework.
    _Source: macro risk day de-risking, 2026-03-14; event-risk-reduction principle, 2026-03-18_

26. After reducing risk, do not immediately undo the defense.
    No selling puts or re-entering the same direction right after a defensive cut.

---

## Options discipline

27. Long calls / call spreads:
    - Time decay is real cost. Do not hold through expiry hoping for a miracle.
    - Set time-based exit: exit or roll at 50% of remaining time if position is underwater.
    - Probe → confirmed upgrade is allowed, but still counts toward max 2 entries per thesis.

28. Choose expiry with buffer:
    - Earnings play: expiry should cover earnings + at least 2-3 weeks of reaction time.
    - Do not buy weekly options for thesis trades — that is gambling, not a trade.

29. Spread mechanics:
    - Bull call spread caps upside but defines max loss. Appropriate for portfolio size constraints.
    - Know your breakeven, max profit, and time exit before entry.

30. **Short-dated option profit-taking is mandatory (not optional).**
    - DTE ≤ 14 option at **+50% → trim 30-50%**
    - DTE ≤ 14 option at **+100% → trim 50-70%**
    - DTE ≤ 7: **no naked hold** — must trim or close entirely
    - Earnings play: **24h before event, decide trim / roll / hold** (pick one, no deferral)
    - Theta curve does not negotiate; "miracle hold" is not a strategy.
    _Source: NOK 0501 $11C +119% peak decayed to -33%, 2026-04-24_

31. For sell put rules, reference `sell-put-rules.md`.

---

## Circle of competence

32. **Whitelist (in-competence):** AI optics, semis (HBM/memory), foundry, AI networks, Physical AI.
    Outside whitelist (energy/nuclear/biotech/consumer/crypto/speculative small-cap):
    - Paper trade 1 month + read ≥ 2 analyst reports before real money
    - 3-day cooling period before any first real entry on a non-whitelist ticker
    _Source: SMR call impulsive entry outside competence, 2026-04-24_

33. **Physical hesitation is a veto.**
    Order-entry mistakes, wrong limit parameters, three price edits, mis-clicks = 24h full stop.
    Body is more honest than brain; a hesitating hand is the subconscious rejecting the trade.
    _Source: SMR mis-entered limit then re-submitted on FOMO, 2026-04-24_

---

## Catalyst vs technical

34. **Catalyst alone ≠ technical confirmation.**
    A news/event catalyst without confirming price action earns probe size only (≤ 30% of planned full size).
    Full size requires technical confirmation on the daily during the regular session.
    _Source: AAOI after-hours $156.50 catalyst-only entry, 2026-04-21/24_

35. **After-hours / pre-market: no new positions.**
    AH and PM sessions have thin liquidity and unreliable prints. Only the following are allowed:
    - Defensive trim / stop-loss execution
    - Pre-planned limit orders already resting
    New long/short initiations must wait for the regular cash session.
    _Source: AAOI $156.50 after-hours buy, chased zone upper bound, 2026-04-21/24_

36. **Zone stale + catalyst → refresh zone BEFORE entry.**
    Cannot enter based on "feel" using obsolete zones.

37. **"Falling knife" default no-go.**
    Definition: single-day |change| > 10% without a clear technical support level being defended.
    Required to enter: (a) next-session confirmation close in support, or (b) precise support touch with volume reversal + wick rejection.
    Emotional "buy the dip" on a -10%+ day without structure = default no.
    _Source: AAOI -13.2% day into $156.50 after-hours catch, 2026-04-21/24_

38. **Trading lot entry position within a zone: mid or lower half only.**
    Upper-bound-of-zone entry is chasing, not accumulating.
    If price is at zone upper bound, wait for zone mid/lower touch before trading-lot entry.

---

## Narrative calibration

39. **Bear narrative discount rule.**
    After first-impression bearish signals (ATM dilution / insider selling / analyst downgrade / consensus target gap), hard-verify each source before sizing response. Verified strength is typically weaker than first impression.
    Limit placement must be anchored to **confirmed technical structure (support levels in the current regime)**, not to "how bad the narrative justifies how deep the entry."
    - Deep tranches = insurance against thesis-break scenarios
    - Main tranche must sit **within the current zone** unless thesis is clearly broken
    - Do NOT build exclusively deep-scenario ladders when narrative, on verification, does not meet "thesis broken" threshold
    - Missing a mean-reversion bounce by over-anchoring to bear narrative is the same mistake as missing a breakout by waiting for a perfect price — both cost real participation.
    _Source: AAOI 2026-04-24 — original $135 single-tranche limit replaced with $128/$118 deep split based on bear narrative (ATM + insider + analyst sell + $53 target). On hard verification: insider was 10b5-1 pre-scheduled, analyst sell was B. Riley 2025/11 already reverted 2026/2/27 to Neutral, consensus target actually $82-101 not $53, ATM was 3/12 news not fresh. Stock gapped up $142 on 4/24 with $140 low, rallied to $153+ (+11%) — neither deep tranche triggered, mean-reversion bounce fully missed. Corrective: $118 moved to $140 (zone2 下沿, within current regime)._

---

## Five iron rules

1. **No confirmation, no size.** (Rule #2)
2. **Max two entries per thesis.** (Rule #4)
3. **Early strength on catalyst days is not confirmation.** (Rule #19) _Source: GTC day, 2026-03-17_
4. **Do not chase the prettiest price — protect the first tranche.** (Rules #14-16) _Source: perfect-price trap review, 2026-04-01_
5. **No trading in the first 30 minutes.** (Rule #17) _Source: opening-discipline, 2026-03_

## Four additional execution rules (2026-04-24)

6. **Short-dated option profit-taking is mandatory.** (Rule #30) _Source: NOK 0501 $11C, 2026-04-24_
7. **Stay in circle of competence; hesitation = 24h stop.** (Rules #32-33) _Source: SMR impulsive entry, 2026-04-24_
8. **Catalyst ≠ technical confirmation; no after-hours new positions; no falling knife; no zone upper-bound chase.** (Rules #34-38) _Source: AAOI $156.50 after-hours catch, 2026-04-21/24_
9. **Bear narrative discount: verify before placing limits; main tranche stays within zone unless thesis is broken.** (Rule #39) _Source: AAOI mean-reversion bounce missed due to over-deep split, 2026-04-24_

---

## Quick reminder prompt

Before any trade, ask:
1. Is this confirmed, probe, or early/risky?
2. What proves me wrong? Where do I exit?
3. Am I in the right zone? Are my zones current?
4. Am I making a good trade, or chasing a pretty trade?
5. Am I preserving the first tranche, or lowering it for a perfect price?
6. Is there an event window I should respect?
7. Has the opening 30 minutes passed?
8. Is this ticker in my whitelist, or do I need a 3-day cooling period?
9. Did my hand hesitate on the order entry? If yes, stop for 24h.
10. Is this an after-hours initiation? If yes, wait for regular session.
11. If option, what's my profit-taking threshold at +50% / +100%?
12. Have I hard-verified the bearish narrative driving my deep-limit placement, or am I anchoring on first-impression fear?

---

## Reference files
- `bottom-confirmation-signals.md` — quantitative bottom signal scoring
- `pre-trade-checklist.md` — full pre-trade checklist
- `two-stage-entry-rules.md` — two-tranche entry framework
- `event-risk-reduction-principle.md` — event risk reduction full framework
- `sell-put-rules.md` — sell put discipline and scoring

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 框架连接：[[Position_Sizing|仓位管理]]、[[Stop_Loss_Strategy|止损与退出策略]]、[[Market_Sentiment|市场情绪]]、[[Mindset_Risk_Control|交易心法与风控]]
