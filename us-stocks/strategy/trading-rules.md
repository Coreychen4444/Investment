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

Last major update: 2026-06-11 (master index 补全决策栈). 2026-04-16 absorbs event-day-trading-rules, opening-discipline, anti-fomo-rules. All source citations preserved.

## Core purpose
Protect against FOMO, early entry, one-size-fits-all actions, emotional trades, and confusing correct logic with correct timing.

## 决策栈索引（master index — 本文件是入口，不是全部）

**优先级声明：Bayesian 决策层高于任何单点 thesis 和本文件任何单条规则**（[[bayesian-decision-model|Bayesian 决策模型]] 六铁律）。

| 层 | 文件 | 角色 |
|----|------|------|
| 决策层 | `bayesian-decision-model.md` | 一切以概率为主：后验 + EV，6 铁律 |
| Sizing 层 | `kelly-position-sizing.md` + `risk-capital-framework.md` + 三层仓位管理 | 分数凯利上限 / capacity 与 cash% 压测 / 25% concentration cap |
| 结构层 | `endogenous-market-model.md` + `options/dealer-gamma-positioning.md` | 共识溢价/拥挤/反身性 + dealer gamma 路径 lens |
| 执行层 | `uncertainty-execution-system.md` + `two-stage-entry-rules.md` + `entry-timing-ev-framework.md` + `strike-triangulation.md` | 三旋钮加仓系统 / 两段式建仓 / binary catalyst EV / strike 四信号 |
| 技术层 | `bottom-confirmation-signals.md` + `technical-indicators-framework.md` | 见底信号量化 / 指标 SOP |
| 期权层 | `options/options-strategy-framework.md`（IV 闸门表 §六）+ `options/sell-put-rules.md` + `options/greeks-discipline.md` + `options/leaps-call-template.md` | 框架 / 卖 put / Greeks / LEAPS 手册 |
| 框架层 | `capital-deployment-while-waiting.md` + `sell-fly-vs-rebalance.md` + `natural-humility-anchor.md` + `post-trade-scoring.md` | 等待期部署 / 卖飞判别 / 谦逊锚 / 事后评分 |
| 退出层 | `trend-exit-system.md` | 赢家侧退出栈：利润棘轮（+50%保本 / +100%回吐上限 1/3）+ 趋势死亡线（3×ATR / 5d×0.92）+ 重进协议；宏观不豁免 |
| 事件层 | `event-risk-reduction-principle.md` + `pre-trade-checklist.md` | 事件前减脆弱性 / 完整 checklist |

---

## 止盈策略总表（2026-07-17 固化 — 全系统只有两族止盈 + 一个管辖前提）

> 设计总纲（[[trend-exit-system|趋势退出系统]]）：**卖飞和硬抗是同一旋钮的两端，单一止盈线无法同时最小化两者。**
> 所以两族方向相反、同时在场；其余看似止盈的条款（G-02/G-03/DTE 阶梯）是强制评估节点，不产生止盈动作。

**顺势侧（往上卖强度，防卖飞后悔）**：

| 条款 | 管什么仓 | 触发 | 硬度 |
|------|---------|------|------|
| G-01 阶梯 | 期权（lot 级） | mark 过 +100/+200/+300% → trim 30/30/40 | 硬提示，只减不清 |
| trim 区 | 正股（zone 体系） | 价格进入 trim 带 | **许可区不是指令**（"考虑比例减"，机器只保底） |
| target_exit | trading 层专属 | 入场预设目标价 | 硬，命中即平 |

**保护侧（往下卖弱度，防硬抗回吐；均为硬线，宏观不豁免，先破先执行）**：

| 条款 | 参照系 | 激活时点 | 触发 |
|------|-------|---------|------|
| 利润棘轮 | **你的成本与峰值**（护钱包） | 峰值浮盈 ≥+50% | mark 破地板（+50%→保本 / +100%→回吐上限 1/3 / DTE<90→1/4），两段式 |
| 趋势死亡线 | **市场结构**（护趋势） | 入场第 1 天（兼初始止损） | underlying **收盘** < 吊灯线（3×ATR20 / parabolic 5d×0.92），全平/减至 base |

分工速记：**期权多死于棘轮**（theta/IV 磨损 premium，underlying 未破线 mark 已穿地板——趋势线看不见 premium 流血）；**正股大赢单多终于趋势线**（浮盈 >9×ATR 后趋势线必在棘轮地板上方，数学边界：trail>floor ⟺ 浮盈>9ATR）。棘轮可盘中触发（巡逻 mark），趋势线只认收盘。

**管辖矩阵（哪类仓听谁的）**：

| 仓位 | 顺势侧 | 保护侧 |
|------|-------|-------|
| Base 正股（无杠杆） | 无 — 唯一卖出 = 大周期顶确认 / thesis 破坏（判断型） | **豁免**（价值配对特权，见 [[mindset-structure-pairing|心态×结构配对]]） |
| Core 正股 | trim 区（advisory） | 棘轮 + 趋势线 |
| Trading 正股 | target_exit（预设） | stop_loss + time_stop |
| 期权（全部） | G-01 阶梯 | 棘轮 + 趋势线 + DTE≤21+OTM |
| **margin 期间任何仓** | — | **全归退出监控，价值持有特权失效** |

止盈后的重进分两类（[[trend-exit-system|趋势退出系统]] §3）：**止盈退出且结构未破 → 随时可重进**（zone 重锚 + 新 zone1/突破回测 + higher-low，新 campaign 不继承递减链）；**规则触发的结构性退出 → 等 🔓 修复信号**。

_Source: 2026-07-17 全系统止盈梳理（全样本反事实回测立项日）；分工边界与"期权/正股死法差异"源自 6 个月全样本反事实（STOCK_M 期权棘轮先触发 vs STOCK_Y 正股趋势线在前）。_

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

4. Same-thesis adds follow _Iron Rule #2 v2_ (2026-04-22):
   - **Averaging down** (new price < avg cost): max 1 per thesis; must be in zone1/zone2; size ≤ 50% of original entry.
   - **Pyramiding up** (new price > avg cost): open-ended but disciplined — higher low confirmed, size ≤ 75% of previous entry, within concentration cap.
   - Never mix averaging down and pyramiding up in the same session.
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

6. Exits are for one of these reasons:
   - Thesis/event has been priced (success exit)
   - **Entry thesis falsified** — subsequent price action contradicts the reason you entered (the bounce never comes, the catalyst sells the news, support fails) → exit decisively, no "扛". See "Thesis-invalidation exit discipline" below.
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
    - Probe → confirmed upgrade is allowed; the upgrade add counts as a pyramiding entry under Iron Rule #2 v2 (higher low + decaying size).

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
    _Source: STOCK_X 0501 $11C +119% peak decayed to -33%, 2026-04-24_

31. For sell put rules, reference `options/sell-put-rules.md`.

---

## Circle of competence

32. **Whitelist (in-competence):** AI optics, semis (HBM/memory), foundry, AI networks, Physical AI.
    Outside whitelist (energy/nuclear/biotech/consumer/crypto/speculative small-cap):
    - Paper trade 1 month + read ≥ 2 analyst reports before real money
    - 3-day cooling period before any first real entry on a non-whitelist ticker
    _Source: STOCK_R call impulsive entry outside competence, 2026-04-24_

33. **Physical hesitation is a veto.**
    Order-entry mistakes, wrong limit parameters, three price edits, mis-clicks = 24h full stop.
    Body is more honest than brain; a hesitating hand is the subconscious rejecting the trade.
    _Source: STOCK_R mis-entered limit then re-submitted on FOMO, 2026-04-24_

---

## Catalyst vs technical

34. **Catalyst alone ≠ technical confirmation.**
    A news/event catalyst without confirming price action earns probe size only (≤ 30% of planned full size).
    Full size requires technical confirmation on the daily during the regular session.
    _Source: STOCK_Z after-hours $156.50 catalyst-only entry, 2026-04-21/24_

35. **After-hours / pre-market: no new positions.**
    AH and PM sessions have thin liquidity and unreliable prints. Only the following are allowed:
    - Defensive trim / stop-loss execution
    - Pre-planned limit orders already resting
    New long/short initiations must wait for the regular cash session.
    _Source: STOCK_Z $156.50 after-hours buy, chased zone upper bound, 2026-04-21/24_

36. **Zone stale + catalyst → refresh zone BEFORE entry.**
    Cannot enter based on "feel" using obsolete zones. See `.claude/rules/zone-maintenance.md`.

37. **"Falling knife" default no-go.**
    Definition: single-day |change| > 10% without a clear technical support level being defended.
    Required to enter: (a) next-session confirmation close in support, or (b) precise support touch with volume reversal + wick rejection.
    Emotional "buy the dip" on a -10%+ day without structure = default no.
    _Source: STOCK_Z -13.2% day into $156.50 after-hours catch, 2026-04-21/24_

38. **Trading lot entry position within a zone: mid or lower half only.**
    Upper-bound-of-zone entry is chasing, not accumulating.
    If price is at zone upper bound, wait for zone mid/lower touch before trading-lot entry.

---

## Narrative calibration

39. **Bear narrative discount rule.**
    After first-impression bearish signals (ATM dilution / insider selling / analyst downgrade / consensus target gap), hard-verify each source before sizing response. Verified strength is typically weaker than first impression.
    Verification toolchain (2026-06-11, broker API): insider-trade list (10b5-1 预设 vs discretionary), short interest / daily short volume (做空压力真值), analyst consensus / rating summary (评级与 target 现值, 不是 stale 报道) — WebSearch 只做补充, 不做主源.
    Limit placement must be anchored to **confirmed technical structure (support levels in the current regime)**, not to "how bad the narrative justifies how deep the entry."
    - Deep tranches = insurance against thesis-break scenarios
    - Main tranche must sit **within the current zone** unless thesis is clearly broken
    - Do NOT build exclusively deep-scenario ladders when narrative, on verification, does not meet "thesis broken" threshold
    - Missing a mean-reversion bounce by over-anchoring to bear narrative is the same mistake as missing a breakout by waiting for a perfect price — both cost real participation.
    _Source: STOCK_Z 2026-04-24 — original $135 single-tranche limit replaced with $128/$118 deep split based on bear narrative (ATM + insider + analyst sell + $53 target). On hard verification: insider was 10b5-1 pre-scheduled, analyst sell was B. Riley 2025/11 already reverted 2026/2/27 to Neutral, consensus target actually $82-101 not $53, ATM was 3/12 news not fresh. Stock gapped up $142 on 4/24 with $140 low, rallied to $153+ (+11%) — neither deep tranche triggered, mean-reversion bounce fully missed. Corrective: $118 moved to $140 (zone2 下沿, within current regime)._

---

## Five iron rules

1. **No confirmation, no size.** (Rule #2)
2. **Same-thesis adds per Iron Rule #2 v2 — averaging down max 1 (zone + half-size), pyramiding up disciplined (higher low + decaying size).** (Rule #4)
3. **Early strength on catalyst days is not confirmation.** (Rule #19) _Source: GTC day, 2026-03-17_
4. **Do not chase the prettiest price — protect the first tranche.** (Rules #14-16) _Source: perfect-price trap review, 2026-04-01_
5. **No trading in the first 30 minutes.** (Rule #17) _Source: opening-discipline, 2026-03_

## Four additional execution rules (2026-04-24)

6. **Short-dated option profit-taking is mandatory.** (Rule #30) _Source: STOCK_X 0501 $11C, 2026-04-24_
7. **Stay in circle of competence; hesitation = 24h stop.** (Rules #32-33) _Source: STOCK_R impulsive entry, 2026-04-24_
8. **Catalyst ≠ technical confirmation; no after-hours new positions; no falling knife; no zone upper-bound chase.** (Rules #34-38) _Source: STOCK_Z $156.50 after-hours catch, 2026-04-21/24_
9. **Bear narrative discount: verify before placing limits; main tranche stays within zone unless thesis is broken.** (Rule #39) _Source: STOCK_Z mean-reversion bounce missed due to over-deep split, 2026-04-24_

## Ladder discipline (2026-06-06)

10. **Buy a sized rung ≠ bet on the bottom.** Eligibility (macro-vs-idiosyncratic, Q-A) and size (phase, Q-B/C) are separate knobs — "可以碰" never means "full size." _Source: STOCK_D reactive add, 2026-06-05_
11. **Pre-define total size before adding; "越低越多" is an inverted pyramid (Martingale), not a pyramid.** Real pyramid size decreases (1/0.75/0.5/0.25); size follows evidence (exhaustion signals), not price. ladder ≠ cost-reduction. _Source: STOCK_D ladder corrections, 2026-06-05_
12. **Reactive add = process D, even if it lands in a preset zone (luck ≠ skill).** Run Q-A→Q-D (`regime_score.py`) before clicking. _Source: STOCK_D 6/5, see uncertainty-execution-system.md_

## Thesis-invalidation exit discipline (2026-06-24)

13. **Entry thesis falsified by subsequent price action = exit, decisively, no hesitation.** Every trade is entered for a stated reason; when the market moves against that reason, the thesis is dead — exit, do not "扛" (hold-and-hope). This is the single most important exit trigger for losers. _Source: two 2026-06 short-dated OTM call bounce/event bets (STOCK_RK index-inclusion bounce −$1,200; STOCK_AX oversold bounce −$740) — both correctly cut on thesis-break._
14. **Hesitation on a thesis-broken exit is the leak — pre-commit kills it.** Define the invalidation (price / event / time) at entry (Rule #18); when it triggers, execute mechanically without re-deliberating. Hesitation enters only when you re-open the decision inside a drawdown — so don't re-open it. (Bayesian #5 pre-commit.) _Source: same two cuts were correct but slow._
15. **Loss ≠ failure — but a timely stop scores the EXIT (A-grade), not the whole trade.** Trading is a probability game; a clean stop on a broken thesis is a successful exit, not a failed trade. This does NOT auto-grade the trade an A — entry edge is scored separately ([[post-trade-scoring|Post-Trade Scoring]]). A clean stop does not redeem a thin-edge entry; "I stopped out so it was a success" must not become a license for low-edge entries. _Source: both = B (exit A, entry thin: short-dated OTM calls betting a bounce/event)._

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
13. If adding/averaging: did I pre-define total size, and is this a sized rung (not a bet on the bottom)? Did I run Q-A→Q-D, or am I reacting to red?

---

## Reference files
- `bottom-confirmation-signals.md` — quantitative bottom signal scoring
- `pre-trade-checklist.md` — full pre-trade checklist
- `two-stage-entry-rules.md` — two-tranche entry framework
- `event-risk-reduction-principle.md` — event risk reduction full framework
- `options/options-strategy-framework.md` — options framework (Greeks, strategy tiers, hedging)
- `options/sell-put-rules.md` — sell put discipline and scoring
- `options/greeks-discipline.md` — operational Greeks rules (theta/delta/vega/gamma)
- `uncertainty-execution-system.md` — 不确定性下的加仓执行系统 (Q-A 资格 / Q-B·C 节奏 / Q-D 工具; `scripts/regime_score.py`)

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 框架连接：[[Position_Sizing|仓位管理]]、[[Stop_Loss_Strategy|止损与退出策略]]、[[Market_Sentiment|市场情绪]]、[[Mindset_Risk_Control|交易心法与风控]]
