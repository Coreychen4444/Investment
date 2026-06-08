# 不确定性下的理性加仓系统 (Uncertainty Execution System)

> Status: active · 建立 2026-06-06 · 源案例 STOCK_D 6/5 reactive add 复盘
> 决策层(Bayesian)与执行层的桥。**优先级:本系统是 `trading-discipline.md` Pre-Trade Quick Check 的 sizing 层补充,不替代铁律。**

---

## 0. 主轴(一句话)

**你永远不知道底在哪——别试。买入不是赌"这是底",是赌"thesis 成立 + 从这里、这个 size,即使再跌也是 +EV 且扛得住"。** 系统的工作 = 把每个决策做成 +EV + 可存活,然后用**证据**一步步替代**感觉**。

---

## 1. 三层架构(接 CLAUDE.md)

| 层 | 回答 | 工具 |
|---|---|---|
| 估值层 | 这东西值多少 | P = EPS × PE |
| 决策层 | 押不押 / 押的逻辑 | Bayesian / EV(概率加权,优先于任何单点 thesis) |
| **执行层 ← 本文档** | 决策层说"押"之后,**怎么押** | 资格 → 节奏 → 工具,三旋钮 |

执行层不替代上两层,是把"押"从**反射**变成**流程**。

---

## 2. 两个 keystone 认知(整套系统的地基)

### 2.1 买 sized 档 ≠ 赌底
脑中要杀掉的错误等式:`买 = 赌涨 = 赌这是底`。这就是 perfect-price trap 本体——一旦"买"必须等于"这是底",你要么不敢动,要么一动就满仓。

正确等式:
> **买一个 *sized* 档 = 赌"thesis 还成立,且从这里、这个 size,即使再跌也 +EV 且扛得住"。** 第一种要猜对底(不可能);第二种只要 +EV + 扛得住(可控)。

### 2.2 Q-A(资格)与 Q-B(size)是两根轴,不是一个开关
"可以碰"和"是不是底"回答的是**不同问题、控制不同旋钮**,所以不矛盾:

| | 回答 | 旋钮 |
|---|---|---|
| Q-A 宏观 vs 个股 | **要不要参与这赌局**(thesis 还成不成立) | GO / NO-GO + 总预算 |
| Q-B 阶段 | **怎么参与:多大注、留多少子弹** | 这一档占预算几分之一 |

合起来 = **"押,但假设你早了 → 先下一小档,把弹药和空间留在下面。"** 这就是 ladder。

---

## 3. 决策流水线(执行层引擎)

```
触发"想加仓"
   │
   ① 进场前定分母 ── 这标的总目标占比 / ladder 总预算(没有分母,"加仓"无意义)
   ▼
   ② Q-A 资格闸 ── 宏观砸 vs 个股坏?
   │     ├─ 个股 fundamental 坏 → 停(无论多超卖)
   │     └─ 宏观砸 + thesis intact → ladder 开,继续
   ▼
   ③ Q-B/Q-C 节奏 ── 什么阶段?这一注吃预算几分之一?
   │     • 半山腰/未超卖/收低位  → 极小探针, 大头留下面
   │     • 超卖+企稳/放量反转/背离 → 中等档
   │     • higher-low 确认        → 确认档(用"晚"换"稳")
   │     ⚠ size 跟证据走,不跟价格走;刀还在飞且无衰竭信号 → 守探针,不加大
   ▼
   ④ Q-D 工具 ── 标的 IV 贵贱?  高 → spread/等;低 → naked 可
   ▼
   下单(pre-planned 档,不是反射点击)
```

**核心约束:Q-A 的"可以碰"永远不能直接翻译成"满仓"。阶段(Q-B/C)决定这一注吃掉预算的几分之一。**

---

## 4. 四个问题 + 指标 + 阈值

指标按"回答哪个问题"组织。**固定一小组 + 事前定阈值**,不要堆 20 个事后挑(那是分析瘫痪 + cherry-pick)。

### Q-A:宏观恐慌 vs 个股崩? → 参与资格
**三问(2026-06-06 升级,源 X post 负 gamma 分析)**:① 宏观 vs 个股 ② 场内轮动 vs 全面出逃 ③ 机械/flow 驱动 vs 真基本面。

| 信号 | 阈值 | 能 / 不能 |
|---|---|---|
| VIX 绝对值 + 单日 spike% | <20 平静 / 20-25 警戒 / 30-40 恐慌 / >40 极端;单日 +30%↑ = 急跌重定价 | 市场天气;**不能**指单标的底 |
| VIX 期限结构(VIX vs VIX3M) | spot > 3月 = backwardation = 急性恐慌(常近短期底) | 比 VIX 绝对值更高质量 |
| 板块广度(同业 ETF/个股 change%) | 全板块同跌 = 宏观;独跌 = 个股问题 | 用 peers change% 对比 |
| **防御板块轮动**(XLV/XLP/XLU/XLE vs XLK/SMH) | 防御涨/抗跌 + cyclical 跌 = **场内轮动(健康, 可逢低)**;防御也跌 = **全面出逃(现金为王)** | `regime_score.py` 已实现;6/5 验证(防御逆涨=轮动) |
| **机械/flow 放大器** | 负 gamma 翻转 / CTA 触发线 / 强平踩踏 / IPO 抽血(SpaceX) / 回购静默期 / 指数 rebalance | 机械驱动 = **均值回归更快 + 对 thesis 信息量更低**;具体 gamma 点位需 GEX 数据源(当前无) |

→ 宏观 + thesis intact = 抄底 thesis 成立;个股 fundamental = 别加,先验证
→ **Bayesian 含义**:机械/flow 驱动的下跌,比普通宏观跌**更不该**移动"thesis 坏了"的后验(它是结构性资金流,不是对你标的的判断)。**但**机械踩踏("越跌越卖")可远超基本面合理位 → "机械=可逢低"只给 **GO 资格,不放宽 size**(size 仍归 Q-B/C)。

### Q-B:阶段 — 刚跌 / 半山腰 / climax? → 这一档多大
| 信号 | 读法 |
|---|---|
| RSI(14) | <30 超卖(反弹概率↑,≠底);**背离**(价新低 RSI 不新低)= 动能衰竭 |
| 距短均线 / 距高点 | 越深 = 均值回归压力↑,但"深"≠"到底" |
| K线结构 | 收当日**低位** = 卖方掌控(续跌);下跌后 hammer/吞没 = 可能转 |
| 连阴 + 加速 | 加速赶底 vs 阴跌 |

### Q-C:恐慌衰竭了吗(R:R 转好)? → 复用 `technical.detect_bottom_signals`(A/B/C)
- 放量 climax + 反转K线(放量**单独看是模糊的**,必须配价格确认)
- VIX backwardation / put-call 飙 / Fear&Greed 极端
- **higher-low 确认 = 唯一"确认转向"信号,但天然滞后**(用"晚"换"确定",放弃最低点)
- **右尾警惕(接 `endogenous-market-model.md` §4)**:跌不动 / 缩量 / 局部反弹 = 边际卖压衰减 → 继续看空是**赔率**问题不是方向问题;你是结构多头 → **别在 washout 底 panic-sell,机械砸点是 +EV 加仓**

### Q-D:vol 贵贱? → vehicle 【期权专属,杠杆最高】
- **标的自己的 IV Rank/Percentile**(不是 VIX):>80%ile = vol 贵 → spread / 别买 naked;<30%ile = 买 long premium 便宜
- VIX 是大盘天气,**标的 IV 才是这只票的恐慌温度**

---

## 5. 指标 = 证据,不是预言

- 每个指标把后验推一点;**共振 = 信心更集中,但永远到不了 100%。**
- 任何单一指标变成"VIX 飙了 = 到底"→ 它就是穿量化外衣的感觉,**比纯感觉更危险**(虚假精确感)。
- 价格下跌本身是新数据,通常把后验推向"我可能错了" → **越低越加大 = 反 Bayesian**(除非能归因宏观噪音)。

---

## 6. 护栏(踩到即停)

| 反模式 | 正解 |
|---|---|
| 用"是否到底 / 胜率"评判自己 | process 维度评分;胜率 ≠ EV(非对称押注低胜率也能 +EV) |
| 看红 → 反射点击 | 先跑流水线(资格→节奏→工具) |
| 越低越多(倒金字塔/Martingale) | size 跟证据(Q-C)走 + Iron Rule #2A 封顶;真金字塔 size **递减** 1/0.75/0.5/0.25 |
| 把"可以碰"读成"满仓" | 阶段(Q-B/C)决定吃预算几分之一 |
| 指标变新感觉 / cherry-pick | 事前定一小组 + 阈值;只用于 size/信心倾斜,不做 binary 买卖 |
| 用今天结果评昨天决策 | 后悔 ROI=0;只用当时信息 + process 评 |
| ladder = 降成本(误信) | 只有抄底降均价;追涨加仓**抬高**均价(且那是对的)。ladder 保证的是参与 + 优于单点 all-in + 双向封顶后悔 |

---

## 7. 诚实边界:量化能 / 不能

- **能**:分清宏观 vs 个股、判断阶段早晚、判 vol 贵贱、给 size 修正系数 → 降噪 + 提高每次决策合理性。
- **不能**:预测底、保证这次对。优化的是**很多次决策的 EV**,不是单次对错。capitulation 信号会假阳性(2022 多次假底);市场可以比你能撑得更久地不理性。ladder 存在就是因为"大致对、永不精确"。

---

## 8. 工具:`scripts/regime_score.py`

一条命令把 Q-A~Q-D 量化,输出 **regime 标签 + size 修正系数 + vehicle 判决**(**不输出买卖信号**)。复用 `technical.py` 全部指标。

```bash
.venv/moomoo/bin/python3 scripts/regime_score.py STOCK_D \
    --peers SMH,SOXX,STOCK_Y --option STOCK_D270115C60000 \
    --vix 21.5 --vix-chg 40 --budget-remaining 2
```

**数据缺口(诚实)**:
- RSI/ATR/MA/量/K线/见底评分 = OpenD 今天就能算 ✓
- VIX = OpenD 取不稳 → WebSearch 后用 `--vix` / `--vix-chg` 传入
- 期权 IV = 取的是 **spot IV** 不是 **IV Rank**(OpenD 无 IV 历史)。当前用 spot + 经验阈值粗判(脚本大声标注);**真百分位需逐日攒 IV** → 见 §10 TODO

### 8.1 STOCK_D 6/5 worked example(真实输出)
```
Q-A 资格 : 标的 -15.08% | 同业均 -10.97%(SMH -9.2/SOXX -10.4/STOCK_Y -13.3) | VIX 21.5 (+40%)
           → MACRO/SECTOR PANIC → GO(若 thesis intact)
Q-B 阶段 : RSI 52.2 | 收盘在日内 7% 分位 | 连阴 2 | 距20d高 -20.5%
           → MID-SLIDE 续跌(收当日低位 + 连阴)— 早,不是底
Q-C 衰竭 : 评分 C + 否决"放量破位(恐慌性下跌,非承接)" → 别指望是底
Q-D vol  : IV 88% = 高 → SPREAD / 等 IV 降;别买 naked
处方     : GO + MID-SLIDE + VOL-RICH → 这一档 ≈ 剩余预算 15% ≈ 0.3 张 + spread
```
**对照现实**:用户 实际 = 反射性 2 张 naked。系统处方 = 接近 0 张 / 顶多一个 spread。**系统没预测底,只告诉他"这不是底 + 用错 vehicle",足以把 size 做对。**

---

## 9. 反馈回路(这就是"越来越理性")

```
决策 → 结果 → process>outcome 复盘(A-D)→ 提炼规则 → 喂回流水线 → 下次更准
```
每次复盘**不修结果,修 process**。一笔 D 级交易的产出不是浮亏,是喂回系统的规则。

---

## 10. Ladder discipline 三处修正(并入本系统)
源:用户 6/6 从 STOCK_D reactive add 逆推 ladder 本质,80% 正确,修正 20%:
1. **"价格越低加仓越多 = 金字塔" 讲反了**。越低越多 = 倒金字塔/Martingale = 失败者陷阱。真金字塔 size 递减(Iron Rule #2B)。
2. **ladder ≠ 降成本**(见 §6 表末行)。
3. **三方向 EV 不对称**:抄底/买跌 = Iron Rule #2A(最多 1 次,接飞刀风险);追涨 = #2B(higher-low + 递减)。技术上"分批>all-in"三向都成立,但方向风险天差地别。

### TODO(让系统更完整)
- [ ] 逐日 log 各持仓标的 spot IV → 攒出 IV Rank,让 Q-D 用真百分位而非经验阈值
- [ ] VIX 自动取数源(替代手动 --vix)
- [ ] regime_score 接入 heartbeat / pre-trade,异动日自动跑

---

## Cross-refs
- `.claude/rules/trading-discipline.md`(Pre-Trade Quick Check + Iron Rule #2A/#2B + Post-Trade Rubric)
- `.claude/rules/macro-context-check.md`(Q-A 的规则化前身)
- `trade/us_stock/strategy/bayesian-decision-model.md`(决策层)
- `trade/us_stock/strategy/kelly-position-sizing.md`(本系统给 ladder 节奏,Kelly 给**单档 size 上限** = 分数凯利;每档 size = MIN(Kelly, concentration, 本档预算))
- `trade/us_stock/strategy/endogenous-market-model.md`(§4 跌不动/右尾 = Q-C 的市场结构解释;washout = 共识溢价压缩,别 panic-sell)
- `trade/us_stock/strategy/bottom-confirmation-signals.md`(Q-C 信号定义)
- `scripts/technical.py`(指标库) · `scripts/regime_score.py`(本系统的 CLI) · `scripts/kelly_size.py`(size 上限) · `scripts/chain_layers.py`(同层冗余 → 联合 Kelly)
- memory: `feedback_uncertainty_execution_system` · `feedback_bayesian_decision_model` · `feedback_embrace_uncertainty`
- journal 源案例: `holding/journals/options_journal.md` 2026-06-05 STOCK_D
