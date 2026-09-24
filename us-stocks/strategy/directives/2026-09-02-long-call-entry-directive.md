---
tags:
  - trading
  - directive
  - options
aliases:
  - Long Call Entry Directive
  - 期权入场指令
---

> [!info] 裁决指令（directive）镜像
> 记录一条规则**何时、为何、凭什么证据**被采纳（后裁决优先）。原文住私有仓 `trade/strategy/proposals/`；「用户」= 系统的所有者与最终裁决人，「agent」= AI 研究/执行助理。ticker 已替换为 `STOCK_*` 占位符，文中 `quant/...` 路径为私有代码强制点。

# 2026-09-02 long call 入场 directive（用户 三裁，canonical）

> 语境：2026-09-01。8/13「期权要到 r2 才能进 …… 期权样本较少需要更高确定性」
> 立规三周后，用户 提出改口径。当轮逐字：
>
> **「现在需要提高下风险偏好 又或者说是适合 long call 的正确玩法。不再限制 gate2 才允许
> 期权入场 因为 gate2 通常意味着高波动率及低盈亏比 虽然胜率可能高一点 但并不适合 long call。
> long call 的目的就是要同时吃波动率和方向 所以最好是在低点布局 设定好更严格的止损条件即可
> 例如收缩到亏 10% 即刻立场」**
> **「但并不代表可以随意开仓 依然需要等见底条件」**
>
> agent 核数据后（下方 §数据）三条裁决：
> **① 「检查下 STOCK_E 553 是哪条线 其实 STOCK_E 在站上 553 的时候进场就非常对了 然后如果跌破 553
> 就果断离场 虽然是以 gate 作为退出条件 但这是含杠杆的期权 所以可以接受稍高的止损率」**
> **② 「硬门槛」（IV）**　**③ 「收到 20%」（S1）**

## 定版（代码真源 `quant/core/sizing.py` / `quant/decision/exit_guard.py`）

| # | 规则 | 值 | 强制点 |
|---|---|---|---|
| 1 | **入场资格** | `OPTION_MIN_GATE = 1`：见底条件 = 系统里第一个**给 cap 的** gate（收复线/止跌线 ×N 日 + RelVR ≥1.2，`structure_repair_veto` 放行），与正股入场资格**同一套**：cap>0 + 结构修复 + 卡未过期 + z1/z2 限价 | `entry_guard` 期权分支 / `option_plan.build` |
| 2 | **IV 硬门槛** | percentile **<40**（≥60 样本可信）/ 序列不足时绝对 ATM IV **<50%** 兜底 / 无数据不放行 | `sizing.long_call_iv_gate`，两个消费者同读；不过门 = 不选约、⛔ 原文进 reasons 与渲染 |
| 3 | **S1 权利金硬 stop** | −25% → **−20%**；`OPTION_ATRISK_LOSS_FRAC` 随 pin 0.20 | `exit_guard.PARAMS["opt_s1_stop"]` |
| 4 | **S5g gate 退出线** | 期权的正股退出线 = **授予当前 cap 的那道 gate 本身**，收盘 ×1 日跌破即清仓，收回重新武装；正股仍用 bfloor_buf（gate − 0.5N） | `sizing.granting_gate_level` → `exit_guard.option_loss_overlay` 🚨 `opt_gate_stop` → brief `stop_status` / 画线 `stop:gate` |
| 5 | **保费上限** | 本档新增 risk ÷ 0.20：gate1 **12.5%** / gate2 7.5% / gate3 5% NAV（仍不跟累计） | `sizing.option_premium_cap_frac` |
| 6 | **执行纪律** | 不在确认日盘中买 call；挂确认后的 z1/z2 回踩限价，与正股清单同拍。**2026-09-08 用户 补**（STOCK_E 560C 案复盘：「期权可以锚定正股回踩 z1 的价格 而千万千万不要追高 期权操作要尽量买在便宜位置」）：期权限价锚 = 正股 z1 回踩，**不取修复阶梯的确认日收盘**；成交时正股整根 60m bar 在 z1 上沿之上 = 追高 = 规则外评 C 起步 | `entry_guard` 期权分支：阶梯一律 zone 档 → `option_plan.build(anchor_note)`，所有非 `vehicle=stock` 的 PLAN 卡都出投影并落 `option_plans.json`（Today 组 + `oz1` 线）· `trade_history_sync` 期权 BUY 落库盖 `underlying_at_fill`（60m bar，拉不到退 1d）+ `price_flags` {`above_z1` / `below_z2` / `above_no_chase` / `in_trim_zone`}，判别式 `core.structure.option_fill_flags`，跨带归人判 · brief `INFRA:fill_price_breach`（此前期权不盖 = 「手册」） |

**STOCK_E 553 是哪条线**：8/10 建卡时的 `stop_level`（basis「LVN 554.4-560 之下的停靠区」），
8/18 收盘 543.8 破它 → 8/19 降级为观察层、553 成为修复 gate（结构层词表 = 止跌线），
8/26 收盘 576.14 站上 ×1 日 + RelVR 3.45 → 转移 `reclaim_553` cap 0.4 = 授予 cap 的 gate1；
现卡 z2 下沿 = 553「刚收复的止跌线本体」，stop 542.59 = 553 − 0.5N（正股地板）。
所以 用户 说的「站上 553 进场 / 跌破 553 离场」= 本 directive 第 1 条 + 第 4 条的逐字来源。
⚠️ STOCK_E 560C 实际成交是 8/26 13:37 ET 盘中（转移在当晚 postclose 才确认），当时卡上 cap 0；
规则仍以**收盘**确认为准（STOCK_O 7/28 盘中改状态事故），正确形状是 8/27 在 z1 561.9-572 挂限价。

## 数据（agent 当轮核对，用户 接受后改的两个前提）

- **gate2 不是高 IV，gate1 才是**：三对可对上 IV 记录的转移，STOCK_Z 149%→112% / STOCK_L 114→91 /
  STOCK_Y 93→67。Δ0.70、270 DTE call 在 gate1 买入、gate2 估值：正股 +4.4/+5.2/+8.8% 而 call
  −21/−10/−6%，vega crush 吃掉入场权利金 27/18/23%。「同时吃波动率和方向」在见底点上，
  吃到的是实现波动、付出的是隐含波动 ⇒ 第 2 条硬门槛的来源。
- **−10% 权利金 = 日常噪音**：按当前 ATM IV，−10% 权利金 ≈ 正股 0.40N（STOCK_Z）到 1.11N
  （STOCK_GO）。三张真实合约日线上任意入场日 20 日内被 −10% 打掉的概率 73/84/77%
  （STOCK_GO 390C / STOCK_E 560C / STOCK_A 220C），−25% 为 63/61/56%。42 张 Jan27 重放：S1 −20 whipsaw
  95%、−25 91%。STOCK_E 560C 实例：8/27 收 63.45 = −10.6% 而 STOCK_E 当日 −0.9%，9/1 以 60 卖出
  −15.5% 时 STOCK_E 收 578 高于 8/26 收盘 —— 正股没破任何线，call 亏在 gap 日 IV 回落与盘中
  成交价。STOCK_GO 390C 8/05 消息跳空收盘 −24.7%：止损线 ≠ 损失，sizing 不能用 0.10 当 loss/unit。
- **账户样本**：19 笔 long call/spread 合计净亏；按入场日在 60 日区间的位置分桶，
  低位/中段 6 笔仅 1 胜（贡献了几乎全部亏损），高位 8 笔 3 胜（STOCK_M ×2、STOCK_Y 价差，小幅净赚）。「低位 call」
  在本账户没有正样本；本 directive 是风险偏好选择，不是数据驱动。
- **真正的约束是 NAV**：池内 Δ0.7 270DTE 一张的保费是 12.5% NAV 上限的 1.2×（STOCK_IN）/ 1.9×（STOCK_Z）/
  2.2×（STOCK_A）/ 5.3×（STOCK_E）/ 14×（STOCK_L、STOCK_Y）—— 没有一张装得进去，现行任何 gate 规则在
  这个 NAV 上都只能靠书面例外执行。STOCK_A 220C 现仓：权利金 = 17.3% NAV，S1 −20 下
  at-risk = 3.5% NAV（gate1 risk 上限 2.5% 的 1.4×，T 5% 之内），delta 名义 =
  83% NAV（单票名义上限 40% 的 2×）。

## 未裁（引用时不能当规则用）

- **「1 张 + decisions 书面例外」**（镜像整股「0 股 → 1 股」条款，条件 at-risk = 权利金 × 0.20
  ≤ T 5% NAV，全书同时一个期权 campaign，该票不叠正股）—— 用户 未答，未裁前不存在，
  0 张的菜单 = 正股或 veto（scope 无 spread）。
- **绝对 IV 上限**：percentile 口径下 IV 88% 的票处于自身低分位会放行；要加是另一条裁决。
- 08-13 D2 翻案的依据是工具属性（gate2 时 delta 腿盈亏比已差）不是样本：样本至今只多
  STOCK_E 560C（−15.5%）与 STOCK_A 220C（持仓中）两笔，两笔在旧规则下都是规则外（gate1），
  简报已点名无授权标签，须补注解。

## 落点

`quant/core/sizing.py`（`OPTION_MIN_GATE` / `OPTION_ATRISK_LOSS_FRAC` / `option_premium_cap_frac` /
`granting_gate_level` / `LONG_CALL_IV_PCT_MAX` / `LONG_CALL_IV_ABS_MAX` / `long_call_iv_gate`）·
`quant/decision/exit_guard.py`（`opt_s1_stop` / `opt_gate_stop_days` / `option_loss_overlay` S5g）·
`quant/options/premium_proj.py`（`binding_stop(gate_level=)`）· `quant/options/option_plan.py`
（硬门槛前置）· `quant/decision/entry_guard.py`（`option_iv_gate` + 过门即 `option_plan.build`）·
`quant/sync/price_reminder_sync.py`（`stop:gate`）· `quant/report/portfolio_brief.py`（`stop_status`
gate）· 测试同名子目录 · `quant/research/RULINGS_LEDGER.md` 2026-09-02 行 ·
[[position-tiers]] / [[trading-discipline]] · [[leaps-call-template]]
§13.0/§13.3/§13.4 · [[options-strategy-framework]] §六。

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[leaps-call-template|LEAPS 手册]]、[[position-tiers|仓位与 sizing]]
