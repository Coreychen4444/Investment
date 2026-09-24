---
tags:
  - trading
  - framework
  - exit
  - risk-management
aliases:
  - Trend Exit System
  - 赢家侧退出栈
  - 利润棘轮
  - Profit Ratchet
---
# Trend Exit System — 赢家侧退出栈（利润棘轮 + 趋势死亡线）

Created: 2026-07-04. Source cases: STOCK_X270115C15000 episode C (2026-07-02 margin 强平 -$2,295,
G-01 +100% 6/2 触发未执行) + STOCK_X260501C11000 episode C (2026-04-24, +119%→flat, Rule #30 源案例)。
两案同标的同错误间隔 6 周 = pattern 不是 variance。

## 定位

**打的问题**:输家侧止损纪律已修好(STOCK_RK/STOCK_AX 6/24 thesis-invalidation 干净离场);出血点在
**赢家管理**——两次 round trip 都是盈利单,合计回吐数千美元浮盈。核心发现:规则不缺(G-01 白纸黑字
在每个 lot 的 exit_trigger),缺的是**触发监控自动化 + 触发后执行**。执行率 0% 的系统等于没有系统。

**设计原则**:卖飞和硬抗是同一旋钮的两端,单一止盈线无法同时最小化两者。本系统用
**分批 + 只升不降的地板 + 重进协议**改变 payoff 结构:每种错误各付小保费,不做二选一。

**与既有栈的关系**(完整退出栈 = 5 件):

| 层 | 规则 | 打什么 | 来源 |
|----|------|--------|------|
| G-01 阶梯 | +100/+200/+300% → trim 30/30/40 | 卖飞(let winner run, 只减不清) | 既有,不动 |
| **利润棘轮** | 本文件 §1 | 硬抗(利润回吐上限) | 新增 |
| **趋势死亡线** | 本文件 §2 | 硬抗(趋势结束强制离场) | 新增 |
| invalid_if / thesis stop | 破位或基本面证伪 | 输家硬抗 | 既有 |
| DTE 阶梯 (G-02/G-03) | 强制评估节点 | 期权时间衰减 | 既有 |

## §1 利润棘轮(giveback floor)— 硬线,无豁免

按 **position 级**(broker avg_cost vs mark;期权用 option mark)追踪 high-water mark(HWM),
地板只升不降:

| 档位 | 触发 | 地板 |
|------|------|------|
| 保本档 | 峰值浮盈曾 ≥ **+50%** | floor = 成本价。**+50% 的赢单永远不允许变亏单** |
| 锁利档 | 峰值浮盈曾 ≥ **+100%** | floor = 成本 + 峰值利润 × 2/3(**回吐上限 = 峰值利润的 1/3**) |
| DTE 收紧 | 期权 DTE < 90 | 回吐上限收紧至 1/4 |

**触发动作(两段式软着陆)**:
- **破锁利档地板 → stage1**:今日内卖 50%(期权整张取整;单张仓 = 全平),剩余地板抬至保本。
- **stage2**:剩余部分破保本 → 全平。
- **破保本档地板**(峰值只到 +50-99% 区间)→ 一段式全平(赢单不变亏单)。
- 触发同时 **no-add lock ON**:该标的禁一切加仓,直到结构修复(close > 触发时刻的高水位收盘)。
  这条机械杀死 STOCK_X 6/15 型 lower-high 加仓(地板破了还加仓 = 直接拦)。
- **新 leg 重新武装**:fire 后 HWM 再创高 >1% → fired 标志复位,地板随新 HWM 抬高,可再次触发。

## §2 趋势死亡线(chandelier trail)— 硬线,无豁免

在**正股日线收盘**上计算(期权仓看 underlying),线只升不降:

- **normal 模式**:trail = 布防以来最高收盘 − **3 × ATR(20, Wilder)**
- **parabolic 模式**(zone methodology = parabolic_mode_spirit):trail = trailing **5d high × 0.92**
  (与 zone 体系锚一致,数学上 ≈ 2.5×ATR)
- **收盘破线 → 平剩余 / 至少减至 base 层**;同时 no-add lock ON。
- **不越价约束**:trail 不允许抬升到现价之上(止损线只能从价格下方拖尾)。暴跌日公式值越过
  close(5d high 嵌旧峰 / 单日 >3ATR)→ 本轮不抬线,用旧线判破位。
- **崩盘中段首次布防**:历史破位不追溯告警,回落到 ATR fresh-start 线(从布防日高水位起算)。

## §2b 止损锚三家族 — 分工与校准(2026-07-15 整合, 用户 提出)

三种止损方法论与本系统的映射。工具 `quant/decision/level_map.py <T> --stops [--entry-price --entry-date]`
一次输出全部数字(与 exit_guard 同源 technical.atr, 数字一致):

1. **结构位止损/止盈(S/R)** — 一直是主锚:zones.stop_level / invalid_if / trim_zone 全是结构位。
   2026-07-15 起结构位推导数据化 = level_map(volume HVN + pivot 聚类 + OI 墙 + 交互史),不再手绘。
   止盈侧 trim_zone 对齐 HVN/pivot 供给带(源案例 STOCK_Z 130→141→152→160 阶梯)。
2. **海龟 2N(ATR20)** — 新采纳,角色 = **sanity band + sizing 交叉验证**,不替代结构位:
   - **停损距离体检**:结构 stop 距现价应落在 **[0.7N, 2N]**。<0.7N = 噪音区必被随机扫掉
     (日波 8-15% 的票尤甚);>2N = 太远,按 2N 缩 size 或找更近结构位。level_map --stops 自动判。
   - **N-unit sizing**:unit = (0.5-1% NAV) / 2N。真实 size = MIN(分数 Kelly, concentration cap,
     ladder budget, **N-unit**) — kelly-position-sizing 的第 4 个 min() 项。
     > **2026-08-10 risk_allocation 修订（用户 三点锁定）**：confirmed campaign 的 unit
     > 改走**轮 schedule 2.5/1.5/1.0% NAV**（= T5% × 50/30/20，三轮封顶；**超出本带宽**，
     > 知情选择），probe 仍在带内 0.75%。真源 `quant/core/sizing.ROUND_RISK_FRACS`，
     > 设计 `quant/research/risk_allocation_design.md`。本行带宽保留为 probe 口径与历史记载。
   - **不采纳**海龟 +0.5N 等量金字塔 cadence — 与 Iron Rule #2B(size 递减 + higher-low)冲突,#2B 优先。
     采纳其一条精神:每次 pyramid add 后全仓 stop 上移(zone refresh 显式化)。
3. **吊灯止损(chandelier)** — **即 §2 趋势死亡线,系统 2026-07-04 起已实现**(3×ATR20 锚布防后
   最高收盘,只升不降,exit_guard.py 每 heartbeat 自动跑)。与教科书差异:教科书 22 期 ATR 锚盘中
   高点,我们 ATR20 锚收盘(抗假影线),维持现参数。**双线取先触发**:结构 stop 与吊灯谁先破谁作数。
   (显示合同 2026-08-10:播报层各基折 max 只显示一个值+来源 card|lock|trail——**执行不变**,两线照跑,先触发者仍作数;折进 max 的只是显示。)

**突破/破位确认通用标定**:"站上/跌破 X"必须用**收盘**;单日收盘越过幅度 <1N 视为未确认,
2 连收或越幅 ≥1N 才算(源案例 2026-07-15: STOCK_Z 128 四拒带 / STOCK_IN 110-112 十三拒带)。

### 布防口径 — armed_date / seed_close(2026-07-25 对齐)

**从入场日布防,seed 取入场当日收盘**(`exit_guard.campaign_entry()`),与
`analytics/exit_rules_backtest.py` 同模型。

此前生产是"exit_guard 第一次看到这个仓位那天布防,seed 取布防前一日收盘",
和回测的 `armed_date = entry_date` 是**两个不同的模型** —— 实盘跑的那条 trail
不是被验证过的那条:

| | armed_date | seed_close | trail |
|---|---|---|---|
| 旧生产 | 首次巡逻见到那天 | 布防前一日收盘 990.21 | 750.84 |
| 回测/现生产 | 入场日 2026-07-24 | 入场当日收盘 920.95 | 680.70 |

_(STOCK_Y 2026-07-24 @925 实例。990.21 是**建仓前一天**的收盘,这笔仓位从没见过那个价。)_

生产那条更**紧**不是更松(HWM 越高线越高),所以不是"保护变少",是实盘跑的规则
和 7/17 那批退出栈回测数字(错配税)描述的行为不是同一个。对齐 = 回到
已验证模型,不是引入新参数,故不需要新回测。

**存量 guard 按棘轮处理**:只升不降。STOCK_Y 对齐后公式值 680.70 < 已挂 750.84 →
保留 750.84;STOCK_O 对齐后 148.50 > 147.30 → 抬到 148.50。

取不到 events/kline 时退回旧口径(`armed_basis: first_seen`),缺一条事件不能
导致布不了防。**armed_date 用 ET 交易日历日**(`lib.timeutil.et_today()`),
不用 `date.today()` —— 后者是 SGT,会把 armed_date 戳到 ET 的非交易日上
(2026-07-25 实测:STOCK_Y 戳成 ET 周六,`since` 直接筛空)。

### ATR 期数口径(2026-07-25 定,用户)

系统里一直有两个 ATR 在跑,分工其实成立,但从没写下来,于是漏出来过:

| 期数 | 用途 | 具体位置 |
|------|------|---------|
| **ATR20** | **算钱** — 决定 stop 距离和 size | `technical.atr` 默认值 / exit_guard 3×ATR trail / entry_guard N-unit sizing / level_map `n_atr20` / `exit_rules_backtest` / regime_transition 观察级推导 |
| **ATR14** | **读盘** — 分档、容差、单日异动判别 | level_map 头行与 volume profile 桶宽 / pivot 聚类容差 / `regime_score.py` / zone-touch A·B 的 `1.5×ATR14%` |

**硬规则**:
1. **算钱路径一律 ATR20**。不是因为 20 比 14"更好",是因为 **sizing 用的 N 必须与 stop 距离用的 N
   是同一个数** —— `unit = 0.75%NAV / 2N`,两边取不同期数则风险预算本身就是错的。且退出栈已在
   ATR20 上回测过(§9 + `exit_rules_backtest`),换期数 = 改退出参数 = 必须重跑回测。
2. **调用点显式传 period**,不吃默认值。
3. **卡片 prose 禁止裸写 "ATR"** —— 必须带期数;凡涉及 stop / 2N / size 的推导只能引 ATR20。

**源漏洞**:2026-07-24 STOCK_Y 卡写 "stop 800 收盘(结构位先于 2N **758**)",758 = 925 − 2×83.6(ATR14);
而 entry_guard 拿去算 size 的是 ATR20 80.07 → 2N = **765**。两个"2N"不是同一个数,
差 $7 不致命,但任何人按卡片复核 size 都会得到跟系统不一样的答案。2026-07-25 已改卡并统一口径。

## §3 重进协议 — 卖飞的真正解药

卖出 ≠ 离婚。**卖飞只在"卖了之后拒绝重进 + 资金闲置"时成立**(sell-fly-vs-rebalance)。

**先分退出类型,重进 gate 不同(2026-07-17 明确,用户 提问 STOCK_Y 止盈重进案)**:
- **A 结构性退出**(棘轮/趋势线/止损触发)→ 结构破了,重进 gate = 下述修复信号(🔓 机械盯)。
- **B 止盈退出且结构未破**(G-01 trim / trim_zone / target 到价)→ **无"修复"概念,随时可重进**,
  gate = 正常入场纪律全套:zone 先重锚(breakout stale → parabolic 锚),重进点 = 新 zone1
  浅回踩/突破回测 + higher-low 确认(没确认 = 最多 probe cap);全平后重进 = **新 campaign**,
  size 不继承 #2B 递减链(fresh Kelly/cap),但 25% 集中度与 no_chase 照管。上次卖价与重进
  无关 — "卖在 X 不肯 Y 买回"= hindsight anchor,才是真卖飞。
- 结构修复定义:收盘突破触发 trim/exit 的那个 lower high(= 新 HH)→ no-add lock 自动解除。
- 修复后按**新 tranche** 走正常 entry 纪律重进(ladder / Iron Rule #2B / zone 检查全套适用)。
- **机械化(2026-07-17)**:exit_guard `reentry_watch` — 全平离场且结构破坏的标的持续盯收盘,
  close > 退出时高水位 → **🔓 一次性告警**(90d 过期;窗口必须 >68d = STOCK_Y 案例 2/4 洗出→4/13 修复);
  解锁后由 `quant/decision/entry_guard.py` 按 watchlist 卡出 ladder 计划。回测立项依据:STOCK_Y 被 2/4
  趋势线洗出后无重进信号 = 全部出场变体中最大单项反事实损失(~-$5.2k)。**只提醒不硬拦**——
  严格 lock(拦重进直到修复)已被回测否决,见 §8。
- Round trip 的差价 = 为利润保护付的保费,不是错误。
- 双向 trap 防护(philosophy_core):本系统的教训**不是**"涨了就跑"。G-01 的 30/30/40 分批
  设计保留 let-winner-run 尾部;棘轮只在回吐超限时动手。若把教训内化成"早止盈",下次就是
  STOCK_Z +100% 踏空主升——同一枚硬币的另一面。

## §4 宏观裁决 — 与 macro-context-check 的冲突消解

macro-context-check 说"机械踩踏 ≠ 技术破位"。裁决:**宏观 lens 只作用于 (a) thesis 后验更新
(不因 flush 判 thesis 死) 和 (b) 重进速度(机械踩踏修复快 → 可以更快重进)。它不豁免棘轮和
趋势死亡线。** 地板保护的是已积累利润和资金久期,与下跌"原因"无关;"机械踩踏会弹回来"是
统计倾向不是担保——STOCK_X 6/5 就没弹回来。想赌修复,用重进协议赌,不是用不卖赌。

## §5 Margin guard — 资金久期 ≥ thesis 久期

STOCK_X 案第三层失败:margin 随时可 call,thesis 要等 7/24 catalyst。"基本面没坏就拿着"的
入场券是无杠杆资金。6/26 购买力红灯在 nav_history 躺了一周无人消费 → 7/2 LEVEL6 强平。

- **power/market_val < 5% → WARN**(本周内 delever 或停止一切加仓)
- **power/market_val < 2% 或 risk_status ≥ LEVEL4 → CRITICAL**(今日 delever,按 conviction
  排序自己选,不要等 broker 替你选)
- nav 数据 >3 天未更新 → INFRA 告警(监控盲飞本身要报警)
- cash < 0 期间:timing-sensitive LEAPS 的"拿到 catalyst"假设默认不成立;加仓先看 power buffer。

## §6 执行军规 — 从 0% 执行率来的半壁江山

1. **告警即行动**:告警文本自带预写行动指令(卖几张、地板抬到哪),当日收盘前执行
   (盘后可成交即当晚——STOCK_W 8/19 18:03 ET 是范本)。
   **卡片 stop 按其 `stop_level_days` 口径收盘破位确认(含 `invalid_if` 同线)= 当日清仓,
   无 override 路径**(用户 2026-08-22 裁:「stop 破了 ×1 立即清仓退出,不能再心存侥幸」)。
   不执行 = 规则外,评 C 起步,brief 的 `exit_overdue` 逐日点名到动作为止。
   证据:override 路径三次使用全部付费——STOCK_Z 8/20「看一天」-$98(8/19 盘后 124.90 可出 -$454,
   8/21 盘后公司宣布第二轮 ATM 增发后 120 出 -$552:ATM 砸的正是 override 买回来的那一天)/
   STOCK_IN 8/20「看一天」-$76(-$204 → -$280)/ STOCK_GO 8/12→8/17 迟到 3 日 -$725;准时执行的
   STOCK_W 8/19 零附加税;7/17 回测「破位确认缓冲」变体(净负)早已否决。
   **棘轮地板 / 趋势死亡线 / G-01 同样无 override 路径**(用户 2026-08-22 同日补裁:
   「一并退役,棘轮趋势线也不留 override」)。旧「书面 override 写进 journal,月度 review 数次数;
   ≥2 次/月 = 系统性问题」条款整条废止——从 STOCK_X 4/24·6/2(棘轮 +100% 未执行 → 7/2 强平 -$2,295)
   到 STOCK_Z/STOCK_IN 8/20,override 没有一次是等到月度 review 才显形的系统性问题,每一次都是当期损失。
   退出栈唯一合法的「不卖」= 告警判据本身未成立(收盘口径、×N 日),不是理由更好。
2. **触发时刻的你不可信任**:4/24 与 6/2 两次证实,ATH 亢奋日人会说"才刚开始"。地板价能挂
   GTC stop 单就预挂(正股安全;期权链薄,stop-limit 触发不成交风险高 → 默认告警制,
   流动性好的链可个案预挂)。
3. 参数改动必须记录在本文件 §8 并说明理由,且 **2026-07-17 起必须附
   `quant/research/exit_rules_backtest.py` 反事实对比数字**(base vs variant 三世界终值
   + maxDD)——防止"这次不一样"式软化。两个"直觉上更稳"的改法(破位确认缓冲、严格重进门)
   都已被该回测否决,教训:先跑数字再动参数。

## §7 自动化实现

- **`quant/decision/exit_guard.py`**:HWM/地板/趋势线/G-01 档位/margin 全量检查。
  State: `state/exit_guards.json`(script-owned,地板与 HWM 只升不降,fired 标志一次性)。
  `--arm` 初始布防(HWM 从 positions.json git 历史回填,现有赢家地板从历史峰值算)。
- **portfolio_brief.py 集成**:每轮 heartbeat(6h)/overnight 自动跑;`Exit guards:` 段每轮
  可见(6/26 教训:margin 状态必须常显,不是只在破阈值时);one-shot 告警(棘轮/G-01)由
  fired 标志自去重,persistent(趋势线/margin)走既有 24h 边沿触发。
- **--read-only 不落状态**:session 内查看不消耗 fired 标志与告警冷却。
- **画线推送 `quant/sync/price_reminder_sync.py`**(2026-08-07 用户 裁决"下方三者取 max"):
  下方线 = max(卡片 stop, 棘轮地板, 趋势死亡线) 挂 券商 服务端价格提醒,note
  `stop:card|lock|trail` = binding 来源(🔒=lock / 📉=trail);期权 basis 分开——
  地板(期权 mark 基)挂合约 code、trail(underlying 基)挂正股 code;G-01 下一档挂合约
  note `g01`。**只读 exit_guards.json 已算好的值**(单一真源,不越价/armed 口径不重算);
  guards 由 brief 阶段刷新而 reminders 排其前 → 画线滞后 ≤6h,方向 = 偏低 = 报警偏晚,
  触发权威仍是本模块每轮真线;提醒 tick 触发 = 预警,执行口径仍是收盘。
- 新仓自动布防(下一轮巡逻即武装);平仓自动撤防——**结构破坏中离场的移入 `reentry_watch`**
  (state/exit_guards.json,盯收盘收复退出时高水位 → 🔓 one-shot,90d 过期;§3 机械化)。
- **入口侧对称物 `quant/decision/entry_guard.py`**(2026-07-17):watchlist 候选每日出 verdict
  (PLAN/VETO/STALE/NEEDS_CARD — 2026-08-10 移除 GO/WAIT,挂的是 zone 限价单,市价位置
  降信息行)+ ladder 价位 + 分类 size cap × N-unit + 止损单行,
  PLAN 变动才落 decisions.jsonl;overnight-briefing §入场执行计划消费其输出,用户 手动挂单。
- 数据依赖:kline 缓存(overnight 每日更新)→ 趋势线是**收盘口径**,盘中不看。

## §8 参数记录

| 日期 | 参数 | 值 | 理由 |
|------|------|-----|------|
| 2026-07-04 | 棘轮 | +50%→保本 / +100%→回吐上限 1/3 / DTE<90→1/4 | STOCK_X+11C 回测最优;用户 确认 |
| 2026-07-04 | 趋势线 | 3×ATR20 normal / 5d×0.92 parabolic | 教科书 chandelier;与 zone 锚一致 |
| 2026-07-04 | margin | warn 5% / crit 2% / LEVEL4+ | 6/26 red flag 位 1.2% 校准 |
| 2026-07-17 | 全栈回测基线 | 实际 = 峰值浮盈全数回吐至 ≈打平 → 规则全栈保住峰值 ~27% / 期权-only(正股价值持有豁免)保住 ~52%;maxDD 收窄约 2/3 | exit_rules_backtest 首跑(2026-01-20→07-16, 142 事件);STOCK_M/STOCK_D/STOCK_IN 为非母案例独立复现 |
| 2026-07-17 | 破位确认缓冲(<1N 首日不执行) | **否决** | 回测净负:真破位晚一日出场的代价 > 避掉的假破位;§2b 确认标定只留在结构位判断,不延伸到死亡线 |
| 2026-07-17 | 严格重进门(lock 存活到修复) | **否决** | 回测净负:被拦的重进净正贡献;重进用 🔓 告警提醒,不硬拦 |
| 2026-07-17 | reentry_watch 过期窗 | 90d | 必须覆盖 STOCK_Y 案例 68d(2/4 洗出→4/13 修复) |

## §9 STOCK_X 回测证据(参数非拟合,教科书默认值)

| 规则 | 触发点 | 结果 |
|------|--------|------|
| 棘轮锁利档(floor ≈ $4.35) | 6/5(mark 从 4.93 砸穿) | 16 张 @ ~4.3-4.4 → **+$2,800** |
| ATR 趋势线(16.85 − 3ATR ≈ 14.8-15.0) | 6/5 收 14.38 破线 | 同日收敛,同上 |
| 结构层(LH 15.07 → trim;6/25 破 13.18 → 清) | 6/16 / 6/25 | **+$1,400-1,800** |
| 仅保本档(最弱参数) | 6/18 mark 破 avg | **≈ $0** |
| 实际(无系统) | 7/2 margin 强平 | **-$2,295** |

且棘轮 6/5-6/12 触发 → no-add lock → 6/15 Tier4 加仓(-$780)机械不会发生。

## 引用
- [[post-trade-scoring]](process>outcome)/ [[sell-fly-vs-rebalance]](重进协议依据)
- [[bayesian-decision-model]] 铁律 #5(pre-commit 防 decay)——本系统是该铁律的机械化
- [[macro-context-check]](§4 裁决对象)/ [[zone-maintenance]](parabolic 锚)
- scores.jsonl 2026-07-02 STOCK_X episode C / 2026-06-15 Tier4 entry C / 2026-04-24 11C C

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[post-trade-scoring|事后评分]]、[[sell-fly-vs-rebalance|卖飞判别]]、[[uncertainty-execution-system|不确定性执行系统]]、[[Mindset_Risk_Control|交易心法与风控]]
