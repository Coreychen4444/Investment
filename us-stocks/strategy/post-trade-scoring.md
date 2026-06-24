---
tags:
  - trading
  - framework
  - psychology
aliases:
  - Post-Trade Scoring
  - 事后评分准则
  - Process over Outcome
---

# Post-Trade Scoring Rubric（事后评分准则）

> Canonical（2026-06-11 自行为触发层迁入 strategy）。评分应同时落结构化文件（不要 prose-only），统计用绩效报告的 Process × Outcome 矩阵 + D 级赢单检测。

> 所有交易事后复盘**必须**用 process 维度评分，不用结果维度。

## 触发条件
任何交易完成后的复盘讨论：
- 平仓 / 止损 / 止盈触发后
- "这笔交易怎么样" / "复盘下昨天的 X" 类查询
- "如果当时..." / "应该 / 不应该..." 类 hindsight 表达
- 周/月 portfolio review

## 评分矩阵

| Process | 结果 | 评分 | 处理 |
|---------|------|------|------|
| 通过 | 好 | **A** | 标记为 reusable success，复用模板 |
| 通过 | 坏 | **A** | "运气问题，process 没错；下次仍然这么做" |
| 错 | 好 | **D 🚨** | "这次成功是 variance，不是 skill。下次同样赌会失败。立即 flag warning" |
| 错 | 坏 | **C** | 提取 process 教训 → 形成新规则或更新现有规则 |

## Process 通过的判断标准

**入场决策类**（buy / sell / sell put / spread）：
- 通过 Pre-Trade Quick Check 4 问（Q1 分类 / Q2 错了怎么认 / Q3 zone / Q4 research 因子）
- Strike 选择走 4 信号 triangulation（如适用）
- Binary catalyst 入场走 EV 框架（如适用）
- Size 符合 tier + Iron Rule #2 v2 + fractional Kelly 上限
- 无 perfect-price trap / greedy-bid trap / FOMO chase / reactive add

**退出决策类**（trim / stop / close）：
- Trigger 是预设的 stop / target / time / G-01/G-02/G-03，不是情绪
- 触发后执行无延迟（"再观察一天" = 不通过）
- 无 anchor / sunk-cost framing

## Anti-patterns（评分时立即 flag）

1. **Hindsight magic**: "如果当时挂 $X 就抓到底了" → 用今天的 ground truth 评判昨天的决策 = 完全不公平。决策只能用**当时已知信息 + process** 评判。
2. **Outcome bias**: "这次赚了所以决定是对的" → process 错也可能赚，**process 错 + 结果好 = D 级 不是 A 级**。
3. **省钱 framing**: "省了 $X entry cost" → 转换为 process 维度："你用了什么数据推导那个 target？" 答不上 = 直觉，过程 fail。
4. **猜对底/顶庆祝**: "我说会跌到 $X 吧！" → "variance ≠ skill，下次同样直觉会失败"。
5. **"我止损了所以这笔成功"**: 及时止损是 **EXIT 步骤**的 A 级 process，但**不自动**让整笔交易变 A —— entry edge 仍单独评分。否则"及时止损=成功"沦为低 edge 入场的免罪符（comfort-trap 镜像：用"我守了纪律"掩盖入场 process 漏洞）。一笔交易可以 = 好 exit (A) + 薄 entry → 整笔 **B**。亏损≠失败为真，但它说的是"process 对的亏损不该后悔"，不是"止了损的交易入场就免检"。

## 双向对称 trap（永远成对警惕）

| Trap | 短期诱惑 | 长期成本 |
|------|---------|---------|
| **猜底成功** | 抓到底了爽 | 下次赌底踏空 |
| **FOMO 追高成功** | 抓到拉升爽 | 下次追高深套 |

两者都是放弃 process 换 dopamine。**唯一的解 = 不论哪次成功，都坚持 process 复盘**（即使是 A 级，也要写下"为什么 process 通过"以确认 reusable）。

## 源案例（process ≠ outcome 的实证）
- **2026-06-05 STOCK_D reactive add = D**：恰落 zone2 = 运气位置非计划触发；若最终盈利 = variance 非 skill
- **2026-06-10 STOCK_IN 末日 put = D**：buy-put≠reduce-risk 结构性错误 + reactive 付峰值 IV
- **2026-06-01 STOCK_S 清仓 = A（后 +34%）**：process 对 + variance 坏，下次仍这么卖
- **2026-06 STOCK_RK = B**：指数纳入类事件 bounce（短期 OTM call，−$1,200）；纳入日 sell-the-news 破 stop，纪律平仓 = exit 对，但 entry edge 薄（事件型 catalyst sell-the-news 是已知 front-run 模式）→ 整笔 B 非 A（exit≠整笔，见 anti-pattern #5）
- **2026-06 STOCK_AX = B**：超跌博反弹 probe（短期 OTM call，−$740）；次日无反弹即止损 = exit 对，但 entry 接飞刀（当日大跌中买 ~26% OTM）+ 深 OTM 短期 call 差 vehicle → 整笔 B

## 引用根基原则
**Rule foundation**: 交易哲学核心（"后悔无价值，复盘有价值"）+ [[bayesian-decision-model|Bayesian 决策模型]] 铁律 #6（process > outcome）。

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[bayesian-decision-model|Bayesian 决策模型]]、[[sell-fly-vs-rebalance|卖飞判别]]、[[Mindset_Risk_Control|交易心法与风控]]
