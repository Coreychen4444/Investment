# Bayesian Decision Model — 投资底层模型

> **状态**：底层模型（foundational），2026-06-04 由 用户 确立，优先级高于任何单点 thesis。
> **关系**：是 `feedback_embrace_uncertainty` 哲学（memory）的操作引擎；与 P=EPS×PE（估值层）互补 —— 本模型 = 决策层。CLAUDE.md「Core Decision Model」是精简指针，**本文是 canonical 详述**。

## 核心原则
一切投资决策以概率为主。每个仓位 = 一个概率加权的赌注。判断标准是**后验分布 + EV**，不是 narrative conviction，不是价格位置，不是"感觉"。

决策 = 持续的贝叶斯更新循环：
1. 建立 prior（入场时对远期结果分布的估计）
2. 新证据到达 → 评估 likelihood → 更新 posterior
3. posterior + 机会成本 → 决定 sizing / 持有 / 轮动
4. 结果出来 → 按 **process** 复盘（不按 outcome），校准下次估计质量

## 6 条操作铁律

### 1. 后验是 agent-relative，不是 market-relative
"市场已知 / 已 priced in" 只回答**定价问题**（标的还会不会动 / 有没有 mispricing edge），**不回答配置问题**（我这笔资本该放哪）。
- 仓位 sizing、机会成本、要不要轮动 → 只看**你当前对远期分布的最佳估计**，与"市场是否已 price in"无关。
- 一条对市场是旧闻、对你是新信息的证据，照样合法移动**你的**后验。
- **源案例（6/4 STOCK_Z）**：我用"市场已知所以别更新"去答配置问题 → 错。个人后验与机会成本和"市场已知"无关。

### 2. 只在 information update 上更新，不在 salience 上
- **Information update**：某个具体事实改变了某个具体结果的概率（真学习）。
- **Emotion / salience update**：已知风险被讲得更生动 / 一根大红 K / 一篇有说服力的文章（近因偏差）。
- **自检**：说得出"哪个数字改了哪个具体概率" = 真更新，调仓有据；只是一股不安 = salience，不动。
- **源案例**：STOCK_Z 看空研报对 用户 是信息型（客户集中度 / FCF 量级他没纳入过）；但 -12% 板块恐慌 K = salience，不更新。
- **增量信息检验（new info vs already-counted，2026-06-08 加）**：即使是真 information（不是 salience），也要问"它相对我**已经计入**的证据，增量解释力是多少"。3 篇都指向同一个 driver 的研报 = **1 个信号，不是 3 个**；相关证据重复计数 → 后验过度更新 → 假自信（穿量化外衣的近因偏差）。正交化检验在 research 库执行（`knowledge/research/README.md` 第一原则 #5）。源：@RuujSs 量化框架模块 4（边际信息 / 正交化）。

### 3. 更新前先验证事实
posterior update 只在**前提为真**时合法。先用 OpenD / WebSearch 核实，不在 misread 上调仓。
- **源案例**：用户 读研报得"STOCK_Z laser 还在 patent 阶段未验证" → 实际 STOCK_Z 已量产自制 laser（收入证明），未验证的只是某个 BH 工艺专利。**capability 已证 / moat 未证才是真赌点**。基于错误前提的"更新"会污染整条决策链。

### 4. 机会成本 / 相对 EV
持仓不是孤立的，它与所有替代品竞争同一块资本。
- **轮动条件**：`E[替代] − E[现仓] > 摩擦`，或轮动改善组合方差 / 集中度（降相关性）。
- 沉没成本无关，只看**远期 EV**。
- 负现金 / 满仓时，"投进最好板块"实为强制 swap（两条腿都付摩擦）→ 决策是"边际这块资本最优用途"，不是"有没有闲钱"。
- **源案例**：STOCK_Z LEAPS 20.4% / STOCK_Z 总 24.7% / optics（STOCK_Z+STOCK_M）56% / 负现金 → 减仓客观依据是集中度 + 相对 EV，独立于研报对错。

### 5. 概率触发要 pre-commit
把"我会在 X 时 / X 价卖"变成**硬触发（日期 + 价格）**，否则 intention 会 decay 成"再看一天"，被动钉在你最担心的 binary 上。
- 时间触发 + 价格 fallback 双写，用既有 zone（stop / no-chase / trim / G-01）做 fallback 锚。
- 与 `feedback_post_incident_attention_hijack`（plan decay）同 bug。
- **源案例**：STOCK_Z "财报前 sell the news" → 落成 7/28 前无条件平 + 跌破 $130 提前出 + 冲 $250 触 G-01。

### 6. Process > outcome
用**决策时的概率估计质量 + EV** 评分，不用结果。
- process 对 + 结果坏 = A（运气）；process 错 + 结果好 = D（variance ≠ skill，下次同赌会败）。
- 完整 rubric：`.claude/rules/trading-discipline.md` Post-Trade Scoring Rubric。
- 根基：`feedback_discipline_perfect_vs_actual_perfect`（"过度追求对已知的完美贴近，恰恰是对未知的背离"）。

## 与现有框架的关系
- **P=EPS×PE**：估值层（标的值多少）；本模型：决策层（不确定下怎么下注）。两层都跑。
- **entry-timing-ev-framework.md**：binary catalyst 入场是本模型的具体应用（target 从 K 线 / ATR / cluster 推导 = 量化 likelihood）。
- **position-tiers.md / concentration**：机会成本铁律（#4）的组合层约束。
- **trading-discipline.md Post-Trade Rubric**：铁律 #6 的执行细则。
- **kelly-position-sizing.md**：本模型给出后验概率 p；Kelly 把 p → size 上限，是"概率→下注比例"的缺失环节。真实 size = `MIN(分数Kelly, concentration, ladder)`。"仓位 = 概率加权的赌注"这句话的执行公式。
- **endogenous-market-model.md**：市场结构层。**共识溢价压缩 / 机械踩踏 = 结构驱动的波动，对个股 EPS×PE thesis 信息量最低 → 不该移动后验**（铁律 #2 的市场结构版）。短期幅度看结构，中长期方向才回基本面。

## 源对话
2026-06-04 STOCK_Z LEAPS 讨论（研报合理性核查 → 是否调整 LEAPS）。本模型 4 个 refinement（#1 agent-relative / #2 info-vs-salience / #3 verify-before-update / #4–5 机会成本 + pre-commit）全部从该次对话逼出，非教科书移植。
