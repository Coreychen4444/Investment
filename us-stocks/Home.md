---
tags:
  - trading
  - hub
aliases:
  - US Stocks Hub
  - 美股工作区
---

# US Stocks Hub

> ⚓ **宁可错过，不可失序；失去节奏，终会成为市场的猎物。**

一个真实美股交易系统的公开脱敏镜像：正股 + 单腿 long call，零杠杆；**技术面授权、概率决策、机械退出**。最近一次全量同步：2026-09-24。

> [!tip] 从这里开始
> **[[system-overview|📐 当前交易系统总览]]** —— 分工四格、决策栈、两种入场模式、sizing 六臂、退出栈、每日节奏、演进时间线，一页看完。

## Strategy canonical（方法论正文）
入口 = [[trading-rules|Trading Rules]] 顶部的 master index（决策栈索引）。

| 层 | 文件 |
|---|---|
| 决策层 | [[bayesian-decision-model\|Bayesian 决策模型]] · [[post-trade-scoring\|事后评分准则]] |
| Sizing 层 | [[kelly-position-sizing\|分数凯利]] · [[risk-capital-framework\|资金 capacity 与压测]] · [[position-tiers\|仓位分层与 sizing]] |
| 结构层 | [[endogenous-market-model\|内生市场论]] · [[dealer-gamma-positioning\|Dealer Gamma]] |
| 执行层 | [[uncertainty-execution-system\|不确定性执行系统]] · [[two-stage-entry-rules\|两段式建仓]] · [[zone-touch-confirmation\|Zone 触碰判读]] · [[entry-timing-ev-framework\|入场时机 EV]] · [[strike-triangulation\|Strike 三角验证]] · [[pre-trade-checklist\|交易前清单]] |
| 技术层 | [[bottom-confirmation-signals\|底部确认信号]] · [[technical-indicators-framework\|技术指标 SOP]] · [[2026-09-12-two-mode-directive\|两种模式]] |
| 期权层 | [[options-strategy-framework\|期权框架]] · [[greeks-discipline\|Greeks 纪律]] · [[leaps-call-template\|LEAPS 手册]] · [[options-journal-template\|期权复盘模板]] |
| 框架层 | [[sell-fly-vs-rebalance\|卖飞 vs 再平衡]] · [[natural-humility-anchor\|谦逊锚]] · [[mindset-structure-pairing\|心态×结构配对]] · [[event-risk-reduction-principle\|事件前减仓]] · [[quarterly-review-template\|季度复盘模板]] |
| 退出层 | [[trend-exit-system\|趋势退出系统]] |

**已退役（只读留档）**：[[sell-put-rules|卖 put 纪律]]、[[capital-deployment-while-waiting|等待期资金部署]]（2026-09-12 起两种模式之外不做卖方）。

## Directives（裁决指令：规则何时、为何被采纳）
- [[2026-08-04-technical-first-directive|2026-08-04 技术面优先 v3]] —— 入场授权只看价格结构
- [[2026-08-18-entry-delegation-directive|2026-08-18 入场委托]] —— 人选股，系统出清单，人执行
- [[2026-08-31-intel-consolidation-directive|2026-08-31 情报层收拢]] —— 归因纪律：「触发源未知」是合法输出
- [[2026-09-02-long-call-entry-directive|2026-09-02 long call 入场]] —— gate1 起可期权 + IV 门 + z1 回踩锚
- [[2026-09-12-two-mode-directive|2026-09-12 两种模式]] —— 只做抄底与趋势确认（现行）
- [[2026-09-11-reversal-probe-directive|2026-09-11 反转探针]] —— 已被两种模式取代，只读

## 行为触发规则（always-on）
- [[trading-discipline|交易纪律触发器]] —— 第零问 + 三问快检 + 行为模式检测
- [[position-tiers|仓位分层与 sizing]] —— cap 二值、gate 预算、六臂、期权保费上限
- [[zone-maintenance|Zone 维护与 gate 画法]] —— staleness 触发、gate 检查表
- [[macro-context-check|宏观归因检查]] —— 异动先分个股 / 板块 / 宏观驱动

## Methods（判断层方法，零交易授权）
- [[fundamental-frame|基本面框架]] · [[market-structure-analysis|市场结构分析]]
- [[macro-inference-contract|宏观推导合同]] · [[macro-sensor-map|宏观观测选择]] · [[rate-to-equity-transmission-framework|利率到估值传导]]

## Research
- [[research-verdicts|研究裁决账本]] —— 预注册回测：采纳了什么、否掉了什么、为什么

## Knowledge
- [[market-traffic-signal-phase-theory|市场红绿灯相位理论]]
- [[photonics-supply-chain|Photonics Supply Chain]]

## Reviews（脱敏真实案例）
- [[2026-03-14-macro-risk-day-de-risking-review|2026-03-14 宏观风险日减仓]]
- [[2026-03-17-gtc-day-early-entry-review|2026-03-17 催化剂日过早入场]]
- [[2026-04-01-perfect-price-trap-review|2026-04-01 完美价格陷阱]]
- [[2026-04-24-three-execution-lessons-review|2026-04-24 三个执行教训]]
- [[2026-06-05-reactive-add-ladder-review|2026-06-05 Reactive 加仓与 ladder 校准]]
- [[2026-07-15-semiannual-execution-gap-review|2026-07-15 半年复盘：执行缺口]]
- [[2026-08-22-stop-override-retirement-review|2026-08-22 止损 override 退役]]

## Holdings（模板）
- [[us-stocks/holdings/current/README|Current Holdings]] —— 标的卡模板见 `_EXAMPLE_.md`
- [[us-stocks/holdings/watchlist/README|Watchlist]]

## Directory Structure
- `system-overview.md` = 系统地图
- `strategy/` = 方法论 canonical；`strategy/options/` 期权；`strategy/directives/` 裁决指令；`strategy/rules/` 行为触发规则
- `methods/` = 宏观 / 基本面判断方法（零授权）
- `research/` = 回测与研究裁决
- `knowledge/` = 主题知识
- `reviews/` = 正式复盘（文件名不含真实 ticker）
- `holdings/` = 标的笔记模板

---
> 📍 **Navigation**
> 上级：[[00_Index/Home|Dashboard]]
> 相关：[[system-overview|系统总览]]、[[Investment_First_Principles|投资第一性原理]]、[[Mindset_Risk_Control|交易心法与风控]]
