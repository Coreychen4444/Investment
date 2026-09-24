---
tags:
  - trading
  - directive
  - options
  - archived
aliases:
  - Reversal Probe Directive (superseded)
  - 反转探针指令（已被两种模式取代）
---

> [!info] 裁决指令（directive）镜像
> 记录一条规则**何时、为何、凭什么证据**被采纳（后裁决优先）。原文住私有仓 `trade/strategy/proposals/`；「用户」= 系统的所有者与最终裁决人，「agent」= AI 研究/执行助理。ticker 已替换为 `STOCK_*` 占位符，文中 `quant/...` 路径为私有代码强制点。

# 2026-09-11 反转探针入场类 directive（用户 三裁，canonical）

> 语境：2026-09-10。同日上午 用户 先提「DTE >150 ATM long call、持有约 1 个月」spec，
> agent 探索定价后（`quant/research/reversal_probe_design.md` §2）再提改规。当轮逐字：
>
> **「我们的期权策略不需要等 gate 只在标的跌到足够便宜即可尝试进场 就是空头力量衰竭的时候
> 有一根阳线打破下跌结构即可尝试进场 严格设置好止损 这套策略本身就是需要在上升初期 所以不能
> 等待过多 不然优势就没了 我们设置好更浅的止损 接受更高的止损率 一旦跌破关键支撑立即离场
> 操作逻辑和正股不同 所以以前制定的期权策略可能需要推翻」**
> **「而且我认为不应该只看 iv 绝对值 要看 ivp 不同标的 iv 绝对值本身就不一样」**
>
> agent 核数据后（§数据）建议「不推翻，加第二入场类」，用户 三裁：
> **「要这个类 ivp 按你推荐 保费按照 gate1 正常取 但是止损放在 gate 倒推算出 cap」**

## 定版（代码真源 `quant/core/reversal.py` / `quant/core/sizing.py` / `quant/decision/entry_guard.py`）

| # | 规则 | 值 | 强制点 |
|---|---|---|---|
| 1 | **入场类** | `reversal_probe` = gate 类之外的**第二入场类**，只给单腿 long call；正股 gate 授权（cap / z1 / 转移机 / 9/02 directive 全部条款）一字不动。正股卡 cap 0 / STALE / 结构 VETO 时探针照样评估 | `entry_guard.evaluate_ticker.done()` → `reversal_probe_plan`；`schema.ENTRY_CLASSES` |
| 2 | **信号** | 近 5 日内出现 20 日新低 ∧ 反转日收阳 ∧ 收盘 > 前 5 日最高（吃掉最后一个 lower high）；**关键支撑 = 前 5 日 + 反转日的波段低点**（结构层词表 = 止跌线）= 本类的 **gate**；信号后任一收盘 < 支撑 = 作废不复活 | `core.reversal.signal_at / detect`，参数 `reversal.PARAMS`（改 = 换判别式，必重跑 `research/reversal_probe_backtest.py` 并进 ledger） |
| 3 | **域** | **只开 R1** = 距 60 日高回撤 [10%, 20%)；R2 深跌（≥20%）与 <10% 只播不授权 | `signal_at["in_domain"]`；渲染 `OUT_OF_DOMAIN` |
| 4 | **IVP 门** | percentile **<40**（同 `LONG_CALL_IV_PCT_MAX`，不放宽）；IV 序列 <60 样本时用 **HV20 的 252 日分位代理**（≥120 样本）；**绝对 IV 兜底对本类不算**（要分位口径）。代理对 gate 类同样生效（顺序 IV pct → HV pct → 绝对 IV → 无数据不放行，basis 必须播出） | `sizing.long_call_iv_gate(hv_pct=)` · `book.hv20_percentile_of` · `entry_guard.option_iv_gate` · `option_plan.build(hv_pct=)` |
| 5 | **限价锚 / 有效期** | 阶梯 = min(信号收盘, 最新收盘) / lower high 回测 / lower high 与支撑中点，全在支撑上；**不锚 z1**（z1 是 gate 类的锚）；有效 **3 个交易日**（XNYS 日历），过期 = 错过不追（座右铭） | `reversal.ladder` · `reversal.PARAMS["valid_days"]` · `market_sessions.add_sessions` · 落 `option_plans.json` 带 `valid_until` |
| 6 | **止损** | 本类 gate = 支撑，**正股收盘 ×1 跌破即清仓**；**无 S1**（保费百分比线关）；S2 (20,25) / S4 90 / S3 advisory / 棘轮 / 趋势死亡线照旧。持仓卡写 `agent.entry_class=reversal_probe` + `zones.stop_level=支撑`（basis underlying） | `exit_guard.evaluate` 期权分支按 `entry_class` 分流 → `option_loss_overlay(gate_level=卡 stop, s1_enabled=False)` 🚨 `opt_gate_stop`「反转支撑」；写卡 `update_position_zones.py --entry-class reversal_probe --stop-level <支撑> --stop-level-basis underlying` |
| 7 | **cap（倒推）** | 风险预算 = gate1 本档 **2.5% NAV**（`gate_step_frac(1)`）；**cap = 2.5% NAV ÷ 正股跌到支撑时保费的投影亏损比例**（BS 同 IV / DTE−fill_days）；再与组合层臂（单票 delta 名义 40% − 敞口 / 冲击预算 / gross / 购买力）折保费取小；**0 张 = veto**（无正股腿，「1 张书面例外」仍未裁） | `sizing.reversal_probe_premium_cap` · `option_plan.loss_frac_at_stop / apply_premium_cap` · `entry_guard.reversal_probe_plan` |
| 8 | **授权 / 盖章** | 授权住 **`option_plans.json`**（`entry_class=reversal_probe` + `valid_until` + `cap_frac` + `signal_close`），不住正股卡 → Today 组只进合约（正股不进）+ `oz1` 线不看正股白名单；成交落库盖 `auth {cap_pct_nav, gate_no:1, entry_class}`，过期成交 = 无授权归人判；正股价位盖 `underlying_at_fill(anchor=signal_close)` + flag **`above_signal_close`**（整根 60m bar 在信号收盘之上 = 追高 → `INFRA:fill_price_breach`） | `entry_guard.write_option_plans` · `today_group_sync.desired_members` · `price_reminder_sync.desired_lines` · `sizing.derive_auth(option_plans=)` · `trade_history_sync` 期权分支 · `schema.PRICE_FLAGS` |
| 9 | **合约** | DTE 下限 270 / Δ 带同 §13.1（IVP <30 放宽 0.55）—— 探索定价 ATM/150 尾部更差，本类不另开 ATM | `entry_guard.RP_TARGET_DTE` · `option_plan.pick_strike(eff_pct)` |

## 数据（探索定价 2026-09-11，非预注册；全表 `tmp/backtest/reversal_probe_report.md`，设计稿 §3）

- **反转 vs 突破**（合成 165 票 2024-01→2026-07，IV 逐日，DTE270 Δ0.70，支撑 stop 无 S1）：
  突破 G 1967 笔 胜率 43% mean +18.2% | R1 369 笔 40% / +12.3% [CI90 +6.8, +19.0] | R0 500 笔 38% / +8.9% |
  **R2 148 笔 32% / +0.4% [−7.8, +11.2]**。正股同信号同序（G +3.9 / R1 +2.0 / R2 +0.8）。
  → 「等太久优势就没了」不成立：退出栈让赢家从两种入场都跑得出来，反转入场多吃的是失败反转；
  「推翻」被数据否，改为**加类**。
- **IVP 是本类的选择器**：R1 按 HV20 分位分桶 <40：胜率 **62% mean +47.9% [+31.1, +63.7] 中位 +24.5%**；
  40–60：37% / +10.7%；60–80：43% / +9.6%；**≥80：23% / −11.5% [−17.7, −4.9]**。反转信号日分位中位 67%，
  <40 只占 24% —— 门拦掉的 3/4 正是亏钱的 3/4。⇒ 阈值不放宽，代理用 HV 分位。
- **更浅止损更差**：支撑改反转阳线低点（0.7N）：R1 mean +12.3 → +7.4%，止损率 15% → 58%（whipsaw 79%）。
  ⇒ 止损 = 波段低点，不是阳线低点。
- **真实 Jan27 合约重放**（2026-01→07，19 笔）：胜率 21%、中位 −17%、平均亏 −23%、平均赢 +263%（STOCK_M +609% /
  STOCK_AK +396%），mean +37.5%；S1−20 会打掉 10/19 ⇒ 彩票形状，期望靠右尾。
- **账户自身**：60 日区间位置 <55% 入场 7 笔仅 STOCK_Y 3/27 一胜（小仓），合计净亏；≥65% 入场 14 笔 6 胜。
  这 7 笔多是接飞刀无确认，不等于本类的「一根阳线确认」，但「买便宜」在本账户没有正样本 —— 本类是风险偏好选择。
- **cap 换算**（Δ0.70/270，IV 40%）：支撑 2N → 投影亏损 −15% → 倒推 cap 16.9% NAV（组合层 40%
  名义臂折保费先咬）；3N → −21% → 11.7%；4N → −28% → 9.0%。

## 首日 live 读数（2026-09-10 ET 收盘卡，接线当轮 `entry_guard --tickers STOCK_B`）

STOCK_B 9/09 收阳 521.10 吃掉 lower high 512.36，支撑 440.5（3.2N），距 60 日高 −10.9% → R1；IVP 门 HV20 分位 16.2 ✓
（basis hv_pct，ATM IV 51% 在旧绝对口径下会被拦）；选约 270617 C470 Δ0.68 mid $122；支撑处投影亏损 39% →
倒推 cap **6.4% NAV**，一张保费 = 12.7× cap → **0 张 = VETO**。STOCK_G 9/08 R1 但 HV 分位 68.7 ⛔；
STOCK_Z / STOCK_AM / STOCK_W 9/08 反转但 R2 域外。**NAV 仍是第一约束**：$500 票在本类下装不下，与 gate 类同病。

## 未裁 / 已知局限（引用时不能当规则用）

- **「1 张书面例外」仍未裁**：0 张 = veto。STOCK_A 9/01 被接受一次未立通则（见 [[2026-09-02-long-call-entry-directive]]）。
- **持仓票不走探针**：只对 watchlist 候选卡（`is_candidate`）评估；持仓正股上加 call 仍走 gate 类。
- **组 / 画线滞后**：`option_plans.json` 由 briefing 的 entry 阶段写，Today 组与 oz1 线在下一轮 heartbeat（≤6h）接上；
  简报 §4 文本当轮即见（与 gate 类选约同一滞后）。
- **跳空低估**：倒推 cap 用的是收盘投影，跳空日止损线 ≠ 损失（STOCK_GO 390C 8/05 −24.7%）。
- **探索定价非预注册**：判据未先锁；复测条件 = live 本类完结 ≥10 笔或 +12 月，届时判据先锁（设计稿 §5）。
- IVP 代理是 HV 分位不是 IV 分位：溢价结构变化（如财报前 IV 抬升而 HV 未动）会漏判，basis 播出即可审计。

## 落点

`quant/core/reversal.py`（新）· `quant/core/technical.py`（`hv` / `hv_percentile`）· `quant/core/book.py`
（`hv20_percentile_of`）· `quant/core/market_sessions.py`（`add_sessions`）· `quant/core/sizing.py`
（`long_call_iv_gate(hv_pct=)` / `REVERSAL_PROBE_RISK_GATE` / `reversal_probe_premium_cap` / `derive_auth(option_plans=)`）·
`quant/core/schema.py`（`ENTRY_CLASSES` / `above_signal_close`）· `quant/options/option_plan.py`（`build(hv_pct=)` /
`loss_frac_at_stop` / `apply_premium_cap`）· `quant/decision/entry_guard.py`（`reversal_probe_plan` / `done()` 钩子 /
render 🔁 / `write_option_plans` / `log_decisions`）· `quant/decision/exit_guard.py`（`entry_class` 分流，
`option_loss_overlay(gate_label=, s1_enabled=)`）· `quant/sync/{today_group_sync,price_reminder_sync,trade_history_sync}.py` ·
`quant/report/portfolio_brief.py`（at-risk 按支撑投影）· `.claude/skills/trading-analysis/scripts/update_position_zones.py`
（`--entry-class`）· `quant/research/reversal_probe_backtest.py` + `reversal_probe_design.md` · `quant/research/dof_audit.py`
（8+1+5 个键登记）· 测试 core/options/decision/sync/research 各一组 · `quant/research/RULINGS_LEDGER.md` 2026-09-11 行 ·
[[position-tiers]] / [[trading-discipline]] · `CLAUDE.md` #6 · [[leaps-call-template]] §13.0。

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[2026-09-12-two-mode-directive|两种模式]]
