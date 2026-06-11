# Greeks Discipline (Operational Rules)

期权交易中由 Greeks 引入的额外约束。正股纪律（zone、分类、Iron Rules）对期权同样适用，本文件只补「Greeks 特有」的执行规则。

每条规则附**源案例**，规则 = 教训的固化，不是教科书条款。

---

## Rule G-01: Long 期权 +100% 浮盈强制 floor 锁定式 partial exit

### 触发条件
任何 long call / long put 的 `unrealized_gain ≥ 100% × cost_total`

### 强制 4 步

**Step 1 — 评估风险档位**（决定 floor 比例）

| 档位 | 判定条件 | floor 比例 |
|---|---|---|
| A 高风险 | DTE < 30 **或** abs(Δ) < 0.30 | floor ≥ **50% × cost_total** |
| B 中风险 | DTE 30-90 **且** abs(Δ) 0.30-0.70 | floor ≥ **30% × cost_total** |
| C 低风险 | DTE > 90 **且** abs(Δ) > 0.70 | floor ≥ **20% × cost_total** |

绝对下限：`floor_pnl_usd ≥ $100`（避免小仓位锁太少导致规则失效）。

**Step 2 — 计算最小 partial exit 数量**

```
required_realized = cost_total + floor_pnl_usd
min_exit_qty = ⌈required_realized / (current_price × 100)⌉
```

**`current_price` 的精确定义（避免与 fill price 混淆）**：

指 **G-01 评估/执行那一刻的市场价 (mark-to-market)** — 不是 cost、不是限价单价格、不是事后查到的 fill price。

- 触发瞬间，current_price 必然 ≥ 2 × cost_per_share（这就是 +100% 浮盈的定义）
- 越晚执行 G-01，current_price 越高 → min_exit_qty 越少 → runner 越多
- 当你的 sell limit 价位**恰好等于** +100% 触发价时，fill price 数字 = current_price 数字（巧合，不是定义）。当 sell limit > 触发价时，fill 时的 current_price = 你的 limit 价格，按 fill 时重算 min_exit_qty
- **错误用法**：用 cost × 2 作为 current_price 来反推 min_exit_qty。这只在触发瞬间正好执行时成立

**示例**（cost $0.62/张 × 10 张 = $620，Tier A floor $310 → required_realized $930）：

| 执行时机 | current_price | min_exit_qty | Realized | Runner 张数 |
|---------|--------------|--------------|----------|-------------|
| $1.24 触发瞬间 | $1.24 | ⌈$930/$124⌉ = **8** | $992 | 2 |
| $1.50 时（延后） | $1.50 | ⌈$930/$150⌉ = **7** | $1,050 | 3 |
| $1.68 时（更晚） | $1.68 | ⌈$930/$168⌉ = **6** | $1,008 | 4 |
| $2.00 时（深 ITM） | $2.00 | ⌈$930/$200⌉ = **5** | $1,000 | 5 |

延后执行的代价是"晚锁 floor"（中间下跌风险全担），收益是"runner 张数更多"。**纪律上 G-01 强制在触发瞬间评估**，不鼓励延后博更高 current_price。

**Step 3 — 最坏情况书面验证**

在 `trade/us_stock/holding/journals/options_journal.md` 当前 entry 写：

> "若剩余 (total_qty − min_exit_qty) 张归零，净 PnL = realized − cost_total = +$X (+Y%)"

没写完 = 不下单。

**Step 4 — 心理框架切换**

平仓后，剩余仓位重新定位为「市场送的免费筹码」：
- 归零是 baseline，不是损失
- 不再设新 stop loss
- 不为剩余仓位回撤焦虑

这个心理切换比规则本身重要。否则会出现"剩余仓位回撤 50% 我又焦虑"——但 floor 早就锁了。

### Floor 上限（防贪）
`floor_pnl_usd ≤ unrealized_gain_usd`。
floor ≥ 浮盈 = 等价全平，不是 partial exit。partial exit 的本意是「放一部分继续跑」。

### 例外（必须书面记录）
1. **Thesis 升级**：新催化剂支持显著更高目标。journal 列出新催化剂 + 新目标价 + 旧目标价
2. **结构性头寸**：spread / 多腿组合按整体 PnL 判断，不按单腿

### 源案例
**STOCK_X 5/01 11C，2026-04-10 至 2026-04-24**
- cost_total = 10 张 × $0.21 × 100 = **$210**
- 4/13 收盘 $0.47 (+124%) — 触发 G-01
- 风险档位：A（8 DTE + OTM）→ floor = max(50% × $210, $100) = **$105**
- required_realized = $315
- min_exit_qty = ⌈$315 / ($0.47 × 100)⌉ = **7 张**
- **应执行**：平 7 张 → realized $329 → 锁定 +$119（+57%），剩 3 张让其跑
- **实际**：0 张平仓，14 天后全平 @ $0.22，净 +$10
- 损失机会成本：
  - vs floor 少赚 $109
  - vs floor + 剩余持有到 4/28 收盘 $0.48 少赚 $253

_Source: STOCK_X 5/01 11C +124% peak decayed to +5% exit, 2026-04-13 至 2026-04-24_

---

## Rule G-02: 短 DTE (<30 天) Long 期权 daily theta budget

### 触发条件
任何 long call / long put 持仓且 DTE < 30

### 强制规则

**每日检查**：当日估算 theta 损耗 ÷ 期权价 ≥ **5%** → 该笔进入「每日复核」状态

**每日复核必须满足以下之一才允许继续持有**：
1. 新催化剂（earnings 临近、技术突破、板块共振、宏观事件）
2. Thesis 成立 + **明确短期价格目标**（X 日内到 Y 价位）
3. 处于已确认的上行 trendline / 突破回测后续涨

全部不满足 → **默认 partial exit 至 DTE > 30 或全平**。

### 计算示例
STOCK_X 5/01 11C 18 DTE 时：
- 期权价 ~$0.20，估算 daily theta ~$0.02
- $0.02 / $0.20 = **10% > 5% 阈值** → 复核状态
- 11 个交易日不复核 → 净 theta 损耗 ~$0.22（≈ 期权价 110%）

### 与 G-01 的关系
G-01 触发于 +100% 浮盈，G-02 触发于 DTE < 30。两者**独立判断、可叠加**：
- 如果同时触发，先按 G-01 partial exit 锁 floor，剩余仓位再按 G-02 每日复核

### 源案例
**STOCK_X 5/01 11C 18 DTE → 0 DTE，2026-04-10 至 2026-04-24**
- 持有 11 个交易日，每日 theta 占期权价 5-15% 区间
- 未启动每日复核机制 → 净 theta 损耗 ~$0.22

_Source: STOCK_X 5/01 11C theta-only loss ~$0.22 over 11 trading days, 2026-04-10 至 2026-04-24_

---

## Rule G-03: Long 期权进财报前 5 个交易日强制三选一

### 触发条件
持有 long call / long put，标的财报日距今 ≤ 5 个交易日

### 强制三选一（不可什么都不做）

**选项 A — 全平**（推荐）
财报前 IV 已膨胀到峰值，期权价含高 vega 溢价。落袋。
- 适用：DTE 短（<14）、当前已 +50% 以上浮盈、thesis 不强
- 数学：vega 红利已实现，下行空间是 vega + theta + 方向三重风险

**选项 B — 半平 + Roll 到远月**
- 卖近月（捕获 IV 峰值）
- 买远月相同 strike 或邻近 strike
- 净效果：vega 暴露降低、theta 加速延后、保留方向暴露
- 适用：thesis 强、相信财报后还有持续行情
- 必须明示：「我赌财报后股价持续走 X 周，目标 Y 价位」

**选项 C — 保持但下调 thesis 信心阈值**
- 必须 journal 书面承担：「接受财报后 IV crush + 方向风险双重打击」
- 必须给方向胜率估值：「我对方向有 X% 信心，敢承担 Y% 概率归零」
- 仓位 ≤ 你愿意完全损失的金额
- 适用：极端高信念 + 已是小仓位 probe

### 禁止
- 「再观察一下」
- 「财报后再决定」
- 「先持有看看走势」
- 任何不做明确选择 = 默认违规

### IV crush 量级参考（用来选 A 还是 B）
- 财报前 ATM IV 通常被打到 IV percentile 80%+
- 财报后 IV crush 25-50%（中位数 40%）
- 对 ATM 期权：IV crush 单项可吃掉 30-60% 期权价
- 要靠 delta 补回，需要正股变动 ≥ **1.5× market-implied 1-σ move**（通常 6-9%）
- 实际市场预期 1-σ move 通常已价入 4-6%，所以需要 8-12% 的实际 move 才能 net 赚

### 源案例
**STOCK_X 5/01 11C 财报 2026-04-23**
- 财报前 5 日（4/16）期权 $0.38（+81%）— 此时应触发 G-03
- 财报前 1 日（4/22）期权 $0.27（已回吐到 +29%）
- 财报当日（4/23）盘中触底 $0.09，收 $0.16（-24%）
- 单日故事：股价 +4.8%，期权 -41%
- IV crush 单项侵蚀 ~$0.20-$0.30（占财报前期权价 70%+）

_Source: STOCK_X 5/01 11C earnings 2026-04-23 stock +4.8% but option -41% intraday low to -24% close_

---

## 计划填充章节（待后续逐步补全）

### Greeks 概念深入（操作层补充）
- Theta 加速曲线（DTE 90/60/30/14/7 节点的相对损耗速度）
- Vega 在不同 DTE 的相对幅度
- Gamma 在 ATM 临近到期的爆炸性 + pin risk
- Delta 的等效股数转换（组合 beta 管理）

### 多腿组合操作规则（✅ 已落地——STOCK_Y 440/500 复盘完成）
正文在 `options-strategy-framework.md` §11.1（多腿先识别组合再算合并 PnL + spread 76.7% max 早平 EV 决策）。本节不再重复维护。

### IV percentile / IV rank 应用规则（阈值跟随 canonical 闸门表 §六）
- 卖方择时：IV percentile ≥ 60 优先 sell premium（capital-deployment 触发线）
- 买方择时：IV percentile < 30 买方友好；≥60 需 4 条 override
- IV term structure（远月低 IV、近月高 IV → 买远卖近的 calendar）
- 数据源：每日记录 IV 历史自建 percentile（攒满 60 天前用外部源手查并标注）

### 期权专属反模式
- IV crush 误判为方向错误
- 时间换空间陷阱（"再等等就回来" = theta 烧钱）
- 卖 put 防守变进攻
- 重仓单腿买方（leverage trap）

---

## 与正股规则的接口

**直接复用**（不重复在本文件）：
- `../trading-rules.md` 全部 entry/exit/zone/anti-FOMO/event-day/behavioral
- `../pre-trade-checklist.md` 主体
- `../event-risk-reduction-principle.md`（事件前 IV + theta 双重作用，G-03 是其期权专项实现）
- `../bottom-confirmation-signals.md`（技术信号同样适用于期权 timing）

**期权特有覆盖**（本文件）：
- G-01: +100% 浮盈强制 floor partial exit
- G-02: 短 DTE daily theta budget
- G-03: 财报前 5 日强制三选一

---

## 引用案例索引

- **STOCK_X 5/01 11C** (2026-04-10 至 2026-04-24): G-01, G-02, G-03 三规则共同源案例
- **STOCK_Y 440/500 spread** ✅: 复盘完成（+$2,760, 23 天捕 76.7% max）→ 多腿规则已入 options-framework §11.1
- **STOCK_Z deep-OTM short put** ✅: 复盘完成（+$685, 收割 77% + pre-earnings de-risk）→ G-02×earnings 联动已入 sell-put-rules 评分卡补充触发
- **STOCK_R 短月 deep OTM call** ✅: 复盘归入 `../natural-humility-anchor.md` 源案例（force close -$780）
