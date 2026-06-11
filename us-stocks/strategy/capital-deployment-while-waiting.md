---
tags:
  - trading
  - framework
  - options
aliases:
  - Capital Deployment While Waiting
  - 等待期资金部署
---

# Capital Deployment While Waiting（等待期资金部署）

> Canonical（2026-06-11 自行为触发层迁入 strategy）。触发条件与响应话术留在触发层规则，本文件是完整方法论。

Applied when: 等某笔大 entry（LEAPS / 重仓 probe / catalyst 入场）时账户有闲置 cash，且当前 IV 在偏高区间 (percentile ≥ 60%，IV 闸门表见 `options/options-strategy-framework.md` §六)。

## 核心原则
**不让 cash 闲置等。等本身有机会成本 + 你的 thesis（IV 回归）正好可以主动卖给市场。**

## 同 thesis 反方向 vehicle

| 你的等待方向 | IV 状态 | 建议 active vehicle |
|------------|--------|-------------------|
| 等 long call/LEAPS 入场（看涨） | IV ≥ 60%ile | **Bull put spread** on 同标的（同看涨 thesis，但收 premium 卖方友好） |
| 等 short call 入场（看空） | IV ≥ 60%ile | Bear call spread on 同标的 |
| 等大跌后 buy stock | IV ≥ 60%ile | Sell put spread at zone1 lower |

## 4 信号场景对比

sell put spread while waiting for LEAPS, all 4 outcomes vs holding cash:

| 场景 | Cash 等 | Spread 行动 |
|------|--------|-----------|
| IV 回归 + 标的微涨 | 0% (cash 不动) | 收 50-70% premium，LEAPS 仍可 entry（更贵但有现金流补） |
| IV 回归 + 标的盘整 | 0% | spread 大幅收割 theta + vega |
| IV 回归 + 标的大涨 | 0% | spread max profit + LEAPS 涨更贵但你 cash flow 充裕 |
| IV 回归 + 标的暴跌（板块砸盘日） | 0% | spread max loss 但**触发 zone1 加仓机会 + IV crush 后 LEAPS 便宜** |

**4 个场景都比 hold cash 好**。这就是 sell premium 的非对称美。

## 执行约束
1. Spread 占用 margin ≤ 计划 LEAPS 预算（不超支等待）
2. Strike 必须过 strike-triangulation 检验（≥ 3 信号，见 `strike-triangulation.md`）
3. 单笔 sell put 评分 ≥ 7/8（见 `options/sell-put-rules.md`）且不触发抵消防守 veto
4. 受 concentration override protocol 约束（三层仓位管理规则）

## 源案例
**2026-04-30 STOCK_Y spread while waiting for LEAPS**：
- 等 IV 回归后开 STOCK_Y 远月深度 spread（LEAPS 替代）
- 此期间 deploy 3× STOCK_Y 近月 bull put spread 收 IV peak premium
- 同方向 thesis（押 IV 回归 + 标的不破支撑）
- 4 场景 EV 都 > hold cash

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[options-strategy-framework|期权框架]]、[[strike-triangulation|Strike 三角验证]]
