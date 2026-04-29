# Greeks Discipline (Operational Rules)

骨架文件，等 2026-04-29 期权复盘讨论后填充。

## 目的
把 `options-strategy-framework.md` 第二节的 Greeks 概念转成**可执行操作规则**：在哪个 Greek 触发什么动作、用什么阈值、避哪些坑。

正股纪律（zone、分类、Iron Rules）对期权同样适用，本文件只补「Greeks 引入的额外约束」。

## 计划填充章节（讨论时确定取舍）

### 1. Theta（时间价值）
- 买方 theta 损耗阈值（每日 theta / 期权价 比例 → 何时该 roll / 平仓）
- 卖方 theta 收割窗口（DTE 区间，何时开仓何时平）
- 周末 / 节假日 theta 双倍计算的执行规则
- 临近到期的 OTM 归零陷阱

### 2. Delta（方向暴露）
- 买 call/put 的 delta 选择标准（ATM vs ITM vs OTM 何时用）
- 等效股数换算（delta × 100 × contracts）与组合 beta 管理
- Delta drift（股价移动后 delta 变化）的再平衡阈值

### 3. Vega / IV（隐含波动率）
- 买方避开 IV 高点（earnings 前买 = 付溢价）
- 卖方利用 IV 高点（earnings 前卖 vs 事件后 IV crush）
- IV percentile / IV rank 参考阈值
- IV crush 估算方法（earnings 前后 IV 平均收缩比例）

### 4. Gamma（Delta 变化速度）
- 临近到期 ATM gamma 风险（pin risk）
- 卖方 gamma 风险窗口（最后一周避开 ATM 短仓）

### 5. 多腿组合（Spreads）
- Bull/Bear spread 何时用、何时不用
- Iron Condor / Butterfly 触发条件
- Spread 的 max loss / max gain / breakeven 速算

### 6. 期权专属反模式（待补）
- IV crush 误判为方向错误
- 时间换空间陷阱（"再等等就回来" = theta 烧钱）
- 卖 put 把防守变进攻
- 重仓单腿买方（leverage trap）

## 与正股规则的接口

**直接复用**：
- `trading-rules.md` 全部（entry/exit/zone/anti-FOMO/event-day/behavioral）
- `pre-trade-checklist.md` 主体
- `event-risk-reduction-principle.md`（事件前 IV 膨胀 + theta 双重作用）

**期权特有覆盖**（本文件补充）：
- 上述 6 个章节

## 源案例池（待整理）
- MU 7/17 440/500 spread（已平 / 持有，winner）
- NOK 5/15 11C theta miss（lesson #30+, 2026-04-24）
- SMR 6/18 20C out-of-competence（lesson #32+, 2026-04-24）
- AAOI 5/22 103P falling-knife（lesson #38+, 2026-04-24）
- AAOI 7/17 / MU / MRVL / SMR 当前持仓 → 滚动复盘
