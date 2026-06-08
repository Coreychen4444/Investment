---
tags:
  - trading
  - framework
aliases:
  - Kelly Position Sizing
  - 分数凯利仓位
  - Fractional Kelly
  - 凯利公式
---

# Kelly Position Sizing — 从后验概率到 size 的映射

> **状态**：active · 建立 2026-06-08 · 源：@RuujSs《The Math That Runs Every Hedge Fund》模块 2(分数凯利)整合
> **定位**：补 `bayesian-decision-model.md` 留的洞 —— 决策层说"仓位 = 概率加权的赌注",但没说**从概率 p 到 size 的映射公式**。本文 = 那个公式 + 生存约束。
> **优先级**：Kelly 给的是 **edge/odds 侧的 size 上限**;真实下单 size = `MIN(分数 Kelly 上限, concentration cap, ladder 预算)`。它是天花板之一,不是单独的开关。

---

## 0. 一句话

**有了后验胜率 p 和赔率 b,凯利公式给出"几何增长最优"的下注比例 f\*。但 f\* 是上限不是目标 —— 永远只用它的一半或四分之一,因为你的 p/b 都是带误差的估计,而过度下注会把正期望变成负的几何增长。**

---

## 1. 为什么需要它:生存数学(回撤不对称)

资金回撤是不对称的:
- 亏 50% → 要涨 **100%** 才回本
- 亏 75% → 要涨 **300%** 才回本

含义:**即使你有正期望(胜率 > 盈亏平衡),过度下注照样会让长期几何增长率变成负的。** 高波动本身会吃掉你的 edge(volatility drag)。

> 21 点例子(原文):胜率 60%、1:1 赔率。每把押 50% 资金,一赢一输后总资金缩水 25% —— 完全符合胜率分布,但仓位过大直接杀死正期望。凯利算出最优是押 20%。

这跟你已有的纪律是同一个家族,Kelly 只是给了它数学根:
- 三层仓位的 size 递减 · Iron Rule #2 的 pyramid 1/0.75/0.5/0.25 · concentration cap · humility anchor —— 全是"别把上限用满"的不同表达。

---

## 2. 公式

```
f* = (b·p − q) / b  =  (p·(b+1) − 1) / b

p = 后验胜率(你的 Bayesian 估计)
q = 1 − p
b = 净赔率(赢一次拿 b,输一次亏 1 单位的"at-risk")
```

特例 even-money(b = 1):`f* = 2p − 1`。
- p ≤ 盈亏平衡(b=1 时即 p ≤ 50%)→ **f\* ≤ 0 = 没有 edge,不下注**。

---

## 3. 永远用分数凯利(½ / ¼),不用 full Kelly

三个理由:
1. **参数有误差**:p 是你的主观后验,b 对正股还依赖 stop 假设。**高估 edge → 越过最优点 → 几何增长转负**(过度下注的惩罚 > 不足下注)。
2. **full Kelly 的波动极剧烈**:即使参数完全准确,full Kelly 的回撤也大到心理扛不住。
3. **真实对冲基金普遍用 half / quarter Kelly** —— 哲学是"用数学锁死仓位上限,确保连败时还在牌桌上"。

**默认 half-Kelly。** 陌生标的 / 相关性高 / p 估计没把握 → 降到 **quarter-Kelly**。

---

## 4. 关键:Kelly 输出的是 "at-risk",不是仓位名义额

凯利的 f 是**投入多少风险资本(会真正亏掉的那部分)**,不是仓位名义大小。换算:

```
at-risk $   = 分数f × bankroll
仓位名义 $  = at-risk $ / 单位最大亏损比例(loss_per_unit)
```

| 工具 | loss_per_unit | 含义 |
|---|---|---|
| **Long option / LEAPS** | **1.0** | 最坏亏 100% premium → 名义 = at-risk(你花的权利金就是风险) |
| **Defined-risk spread** | (宽度−credit)/宽度 | 最大亏损定义清晰 |
| **正股 + stop** | stop 距离 %(如 20%) | 名义 = at-risk / 0.20 = 5× at-risk(被打 stop 只亏 20%) |
| **裸正股无 stop** | — | **Kelly 不适用**(b 和 at-risk 都无法定义) |

> **洞见**:defined-risk 期权结构(long option / spread)比裸正股**更适合**套 Kelly —— 分母(最大亏损)是确定的。你的 book 以 LEAPS / spread 为主,正好是 Kelly 最干净的应用场景。

---

## 5. Half-Kelly 查找表(at-risk % of bankroll)

表内 = **half-Kelly 的 at-risk 比例**(已经砍半)。p = 后验胜率,b = 净赔率(赢:亏)。

| p \ b | 1:1 | 2:1 | 3:1 |
|---|---|---|---|
| **55%** | 5% | 16% | 20% |
| **65%** | 15% | 24% | 27% |
| **75%** | 25% | 31% | 33% |

读法:
- **高 p + 高 b 时 at-risk 比例会很大(>30%)** —— 这正是为什么第 6 条的 MIN(concentration cap) 几乎总会先 bind。Kelly 告诉你"边缘允许多大",concentration 告诉你"组合能承受多大",**取更紧的那个**。
- quarter-Kelly = 表内数字再砍半。
- 表是粗刻度;精确值用 `scripts/kelly_size.py`。

---

## 6. 绑定规则(怎么落成真实 size)

```
真实下单 size(名义)= MIN(
    分数Kelly 名义上限,        ← edge/odds 侧(本文)
    concentration cap 名义,    ← correlation/blow-up 侧(risk-capital-framework.md §3)
    本档 ladder 预算            ← 执行节奏(uncertainty-execution-system.md)
)
```

- Kelly 和 concentration 是**两个不同方向的天花板**:Kelly 防"下注太小浪费 edge / 太大杀几何增长";concentration 防"单点 blow-up / 相关性叠加"。**永远取最紧的。**
- Kelly 算出来比 concentration cap 还小 → 说明 edge 不够强,**别用 concentration 配额硬填到满**。

---

## 7. 相关性警告(接 chain_layers / 因子冗余)

**凯利假设每个赌注独立。** 你 book 里同一子驱动(内存 / 光互联 / 硅)的多个仓位**不独立** —— 它们会一起赢、一起输。

- ❌ 不能对 3 个同层(如光互联)仓位**各自**算 half-Kelly 再相加 → 实际等于对同一个赌注下了 ~3× Kelly = 严重过注。
- ✅ 同一子驱动层的多注 → 合并成**一个 Kelly 预算**,再在层内分配。
- 用 `scripts/chain_layers.py` 查当前 book 哪些层有 ≥2 个名字(同层堆叠),那一层套**联合 Kelly**,不是各自 Kelly。
- 这是 Kelly 与 `feedback_ai_circle_of_competence`(同层冗余)的接口:**冗余检测不是为了"减 AI",是为了"别对同一个赌注算多遍 Kelly"。**

---

## 8. 诚实边界(量化能 / 不能)

- **能**:把"概率加权的赌注"从口号变成一个 size 上限;给 fractional / 递减一个数学根;在 defined-risk 期权上分母干净。
- **不能**:
  - p 是**主观后验**(你 Bayesian 的估计,带误差)—— Kelly 不会让烂的概率估计变准,垃圾进垃圾出。
  - b 对正股需要先定 stop;stop 变,b 和 size 都变。
  - Kelly 优化的是**几何增长**,不优化心理舒适 / 路径 / 单笔后悔。短样本里 variance 仍主导。
  - 它是**上限**不是**信号** —— 不回答"该不该进"(那是 Bayesian + Pre-Trade Check),只回答"进的话最多多大"。

---

## 9. 工具:`scripts/kelly_size.py`

```
.venv/moomoo/bin/python3 scripts/kelly_size.py \
    --p 0.65 --b 2.5 --fraction 0.5 \
    --bankroll 50000 --loss-per-unit 1.0 \
    --cap-pct 25 [--json]

# 期权可用 --entry / --target 自动推 b(b = (target−entry)/entry):
.venv/moomoo/bin/python3 scripts/kelly_size.py \
    --p 0.6 --entry 2.47 --target 8.0 --bankroll 50000 --loss-per-unit 1.0
```

输出:f\*(full)、分数 f、at-risk $、仓位名义 $、binding 约束、edge 警告。

---

## Cross-refs
- `bayesian-decision-model.md` —— 提供 p(后验);本文是它"概率→size"的缺失环节
- `uncertainty-execution-system.md` —— 提供 ladder 节奏;Kelly 给单档 size 上限
- `risk-capital-framework.md` §3 —— concentration cap;与 Kelly 取 MIN
- `scripts/chain_layers.py` —— 同层冗余检测,决定哪些仓位套联合 Kelly(第 7 条)
- `.claude/rules/position-tiers.md` —— 三层 size 递减是 Kelly 的纪律表达

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[bayesian-decision-model|Bayesian 决策模型]]、[[uncertainty-execution-system|不确定性执行系统]]、[[risk-capital-framework|风控框架]]、[[trading-rules|Trading Rules]]
