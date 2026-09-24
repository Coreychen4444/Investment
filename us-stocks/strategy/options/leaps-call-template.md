---
tags:
  - trading
  - options
  - framework
aliases:
  - LEAPS Call Template
  - LEAPS 操作手册
  - LEAPS
---
# LEAPS Call 操作手册

## 核心原则
> LEAPS = 用资本效率换 vega 暴露。它不是"更长的 long call"，是另一种 Greeks profile 的工具。

LEAPS（Long-term Equity Anticipation Securities）= **DTE > 365 天的 long call/put**。本文件聚焦 long LEAPS call。

### 框架定位（先过上两层，再用本手册）
本手册是**执行层**，坐在两个更高层之下。选 LEAPS 之前必须先过：
1. **决策层 — Bayesian 概率模型**（[[bayesian-decision-model]]）：该不该押、押多大，用后验分布 + EV 决定，不是看 thesis 顺不顺眼。本手册只回答"押了之后用什么 vehicle / strike / DTE / expiry"，**不回答"该不该押"**。
2. **Vehicle 选择 — IV regime 闸门**（memory `options_vehicle_selection.md` §IV-pairing）：IV > 70%ile **倾向** sell premium（spread / sold put）而非买 long LEAPS。先确认 LEAPS 是对的 vehicle，再进本手册选 strike。

> 把本手册当孤立的 LEAPS 选择器 = 丢掉概率层和 vehicle 层 = 系统性偏 OTM + 在高 IV 乱买 long call。这正是外部框架（如纯 GPT-discussion 版）最容易缺的两层。

### 标的质量默认倾斜（2026-07-21，用户 设定）
LEAPS 标的**默认优先大市值 + 已盈利 + 现金流充足**的公司（STOCK_A / STOCK_Q / STOCK_F class）。不是保守偏好，是 LEAPS 结构决定的：
1. **Vega 税**：mega-cap IV 常态 30-45%；小市值/未盈利标的 IV 80-105%（STOCK_O 105% / STOCK_G 87% 实测）→ 同样的 thesis，一半权利金在付 vega，IV crush 单独可打掉 30-50%
2. **路径风险**：LEAPS 是 12-24 月 marathon，赢的前提是路径上不被 -40% 级 drawdown 逼出局；盈利 + FCF 自带地板（回购/盈利支撑），路径波动结构性更低。STOCK_X LEAPS 7/2 强平 -$2,295 = 路径杀死方向的实证
3. **流动性**：深链 OI + 窄 spread，Entry Checklist 的 bid-ask/OI/volume 三项在 mega-cap 上天然达标
- **非硬 veto**："尽量优先" = 质量差/高 IV 标的 LEAPS 仍可做，但必须过 IV 闸门 override 4 条（[[options-strategy-framework]] §六）+ 自动减 size + 偏 DITM
- **质量倾斜 ≠ 豁免配对规则**：STOCK_A LEAPS 仍是期权 book = 趋势配对，退出权归 exit_guard（[[mindset-structure-pairing]]），大市值不改变这一条

---

## 一、与短期 Long Call 的本质差异

### Greeks 对照（DTE 30 vs DTE 540）

| Greek | 短期 Call (DTE 30, ATM) | LEAPS Call (DTE 540, ATM) | 差异含义 |
|-------|------------------------|--------------------------|---------|
| Theta | 高（~$0.05-0.10/day） | 低（~$0.005-0.015/day） | LEAPS 时间是温柔的敌人 |
| Vega | 中（~$0.05-0.10 / 1% IV） | **极高**（~$0.30-0.80 / 1% IV） | LEAPS 第一大风险源 |
| Gamma | 高（临近到期 ATM 爆炸） | 低（DTE 长 gamma 平缓） | LEAPS Delta 反应慢、稳定 |
| Delta | 同 moneyness 类似 | 同 moneyness 类似 | 差异不大 |

### Mental model
- 短期 Long Call = 高 gamma + 高 theta 的**方向 sprint**
- LEAPS Call = 高 vega + 低 theta 的**方向 marathon + IV 押注**

### 上涨路径的 Greeks 演化（轻度 OTM LEAPS 被正股拉起时，2026-06-08 back-port）
买入时 Δ0.4 的轻度 OTM LEAPS，若正股一路上涨，期权性质会**分三段质变**：

| 阶段 | Δ 变化 | Gamma | Vega | 含义 |
|------|--------|-------|------|------|
| **OTM → ATM** | 0.40 → 0.55 | 上升至峰值 | 高 | **最肥的 convexity 段**：正股涨 + Δ 升 + gamma 加速 +（若 IV expansion）vega 助攻 → call 涨得最猛 |
| **ATM → ITM** | 0.55 → 0.80 | 开始下降 | 开始下降 | 仍赚，但最强凸性已过 ATM，加速度递减 |
| **深度 ITM** | → 接近 1 | 明显下降 | 下降 | 从爆发仓**质变为类正股仓**，后续走势 ≈ 100 股正股 |

**操作含义**：轻度 OTM 涨成深度 ITM 后，**原始交易逻辑已经变了**——它不再是高 convexity 押注，而是低杠杆类正股仓。此时强制重估：继续持有 / G-01 锁利 / roll up / 转正股。最肥的 OTM→ATM→ITM 段吃完 ≈ 该兑现的信号。

> ⚠️ **LEAPS 专属 caveat（外部框架普遍缺）**：这条 convexity 路径默认 move 发生时 **DTE 仍长**。若正股大涨发生在 12 月 LEAPS 的第 11 个月，gamma 早已随 DTE 缩短衰减，"最肥的一段"被时间吃掉。这是 §11.4「strike 和 DTE 不能赌不同的事」的镜像：**convexity 需要 DTE 配合，OTM + 短 DTE = 两头不靠**。

---

## 二、三种核心用法

### 用法 1：Stock Replacement（资本高效正股替代）

**结构**：DITM LEAPS call，delta 0.75-0.85，DTE 12-24 月

**逻辑**：
- 用期权代替正股，资本占用 25-40%
- 1 张 delta 0.80 = 控制 80 股的 directional exposure
- 解锁的 60-75% 资本可做其他事（更多仓位 / sell put 收租 / 现金缓冲）

**适用**：
- 资本受限（你目前 LEVEL5 杠杆已近顶）
- thesis 持续 1+ 年
- 标的 IV percentile < 60

**风险**：
- ⚠️ Vega 风险（IV 暴跌 30% → 1 张蒸发 $200-500）
- ⚠️ 无股息（持股有股息，LEAPS 没有）
- ⚠️ 远月流动性差（bid-ask 通常 5-10%，比正股贵）

### 用法 2：Long-Term Thesis Expression（长期方向押注）

**结构**：ATM 或 slight OTM LEAPS call，delta 0.45-0.60

**逻辑**：
- 长期叙事确定但 timing 模糊（Physical AI / HBM cycle / 6G / quantum）
- 时间换空间，不被短期波动洗出
- 不希望 base 仓波动情绪干扰判断

**适用**：
- 多季度才 play out 的故事
- 资本量小但想押大方向
- thesis 强但 catalyst timing 不确定

**风险**：
- ⚠️ Thesis 实现但慢于预期 → IV 不动 + theta 慢慢吃
- ⚠️ Thesis 实现但通过 sideways → DITM 稳但 ATM 难赚

### 用法 3：对冲用法 — Long-term Insurance Against 踏空

**结构**：OTM LEAPS call，delta 0.20-0.35

**逻辑**：
- 已 trim 某 core 仓 → 怕错过结构性 melt-up
- 比短期 long call 便宜，保留长期参与权
- 对"trim 后焦虑"型交易者特别有效

**适用**：
- 减仓后仍看好长期 thesis（"减是为了 zone discipline，不是 thesis 破"）
- 大盘/板块顶部争议时不想完全离场

**风险**：
- ⚠️ OTM LEAPS 仍可能归零（不便宜的保险）
- ⚠️ "保险心理"导致对其他正股仓位过度自信

---

## 三、Strike 选择决策树

```
LEAPS 目的？
  │
  ├─ 替代 100 股正股 (stock replacement)
  │     → DITM, delta 0.75-0.85, strike 在标的现价 -15% 到 -25%
  │
  ├─ 长期 thesis 方向押注
  │     → ATM 或 slight OTM, delta 0.45-0.60
  │     → strike 在标的现价 ±5% 到 +10%
  │
  ├─ 高赔率 lottery / 长期保险
  │     → OTM, delta 0.20-0.35
  │     → strike 在 trim_zone 上沿或更高
  │
  └─ 不知道 → 不做（LEAPS 错配比短期错配代价大）
```

### Strike 与 Zone 体系映射

| 用法 | Strike 锚定 | 对应 zone |
|------|-----------|----------|
| Stock replacement | accumulation zone1 下沿 / breakeven 接近 zone1 上沿 | zone1 内成本最低 |
| Thesis expression | accumulation zone1 上沿 / no_chase 之间 | 偏 ATM |
| Long-term lottery | trim_zone 上沿或更高 | OTM 高赔率 |

---

## 四、Expiry 选择规则

LEAPS 期权链通常只有 **Jan 系列**（每年 1 月第三个周五）。

| Expiry | DTE 区间 | 用途 | 流动性 |
|--------|---------|------|--------|
| Jan +1 年 | 270-365 | 准 LEAPS（看 12 月内 catalyst） | 较好 |
| **Jan +2 年** | **540-635** | **LEAPS sweet spot**（vega/theta 比最优） | 较好 |
| Jan +3 年 | 900-1000 | 远月（bid-ask 15-25%） | 差 |

**默认**：Jan +2 年（最佳 vega/theta 比 + 流动性 + 跨多个财报 catalyst）

**不选 Jan +3 年的原因**：
- bid-ask spread 太宽（开仓平仓成本高）
- 多 12 个月 = 多付 30-40% premium 但 delta 提升微弱（边际收益递减）
- 退出成本高

**Jan +1 年（准 LEAPS）适用场景**：
- 看好 12 个月内某 catalyst
- 不适合纯 stock replacement（DTE 不够长）

---

## 五、Position Management 生命周期

### Entry Checklist（LEAPS 专项）

| 检查项 | 标准 | 不达标怎么办 |
|--------|------|--------------|
| IV percentile | < 60%（canonical 闸门表：[[options-strategy-framework]] §六） | ≥60 默认不入；4 条 override 全满足才放行（parabolic AI + uncapped chain + LEAPS DTE + size 减档），且偏 DITM |
| Bid-ask spread | < 8% of mid | 换个 strike 或 expiry |
| Open Interest | > 50 contracts | 流动性不足，弃 |
| Volume (近 5 日均) | > 5/day | 流动性不足，弃 |
| Underlying thesis | ≥ 12 月可信 | 短期 long call 替代 |
| Underlying 质量 | 大市值 + 已盈利 + FCF 充足优先（核心原则§标的质量倾斜） | 质量差/高 IV 标的：override 4 条 + 减 size + 偏 DITM |
| Position sizing | MIN(分数 Kelly, 25% concentration cap, ladder 预算)——[[kelly-position-sizing]]；LEAPS independent book 不进 stock pyramid 链但受同 ticker cap | 减 size 或弃 |

### 监控节奏（不像短期那样敏感）

| 时间 | 关注 |
|------|------|
| 每周 | IV percentile 变化、underlying vs strike 距离 |
| 每月 | DTE 倒数、distance to ATH、thesis 验证进度 |
| 每季度 | Thesis 阶段性验证（财报、guidance、catalyst） |
| **DTE 90** | **强制重审**：roll out vs close vs hold |
| **DTE 30** | 自动退化为短期期权，G-02 启动 |

### Exit Triggers

```
Underlying close > strike + 1.5 × premium → 进入 trim 评估区
G-01 触发 (+100% unrealized) → 强制 partial exit
   ├─ DITM (Tier C, DTE > 90 + Δ > 0.7) → floor 20%
   ├─ ATM (Tier B, DTE 30-90 或 Δ 0.3-0.7) → floor 30%
   └─ OTM (Tier A, DTE < 30 或 Δ < 0.3) → floor 50%
DTE = 30 → G-01/G-02 高强度规则启动
DTE = 90 → 决策点：roll out (Jan +3 年) / close / hold
Underlying close < invalid_if → 全平
Underlying close < zone1 下沿 + IV 飙升 → 评估 vega 收割（IV crush 后 reload）
```

### Roll Out 决策（DTE 90 时）

**Roll 条件**（同时满足才 roll）：
1. Thesis 仍完好
2. Underlying 仍在 zone1 上方 / 在 entry strike 附近或上方
3. Roll 后新 LEAPS 的 Greeks 重置（theta 降回 ~$0.005）值得多付 premium
4. Roll cost ≤ 当前 LEAPS 价值的 30%

**不 roll 条件**：
1. Thesis 已退化（财报 miss / 关键客户流失）
2. Underlying 进入 invalid_if 区间
3. 你只是为了"避免承认错误"

---

## 六、LEAPS 独有 Anti-Patterns

| 反模式 | 信号 | 危害 | 对策 |
|--------|------|------|------|
| 当短期期权管理 | 每天盯盘看 P&L 焦虑 | 对 daily noise 过度反应 | LEAPS 月度 review，不日度 |
| IV 高位 entry | 在 earnings 前 / 大事件前买 LEAPS | 事件后 IV crush 吃 30-50% premium | 等 IV percentile < 50 |
| 滚动续命 | DTE < 90 时 roll Jan +3 年续命 | 越滚成本越高，thesis 早就不成立 | DTE 90 必须基于 thesis 重审 |
| DITM 当股票一样砍仓 | underlying 跌 5% 立刻平 | DITM 不是底部托住的股票 | DITM 严守 invalid_if，不太早砍 |
| OTM 过度 lottery | 同一 ticker 买多张极 OTM LEAPS | 累积成本 = 一笔正经 ATM | 要么 ATM 要么不做 |
| 多 LEAPS 集中度叠加 | 多个 ticker 买 LEAPS | 单笔小，组合 vega 暴露巨大 | LEAPS 总成本 ≤ 账户 15% |

---

## 七、Pre-Trade Checklist（LEAPS 专项）

每笔 LEAPS 入场前过一遍。

### 通用检查（与 framework checklist 配合）
- [ ] 分类：Confirmed / Probe / Early-risky？
- [ ] 用法：Stock Replacement / Thesis Expression / Hedge / Lottery？
- [ ] Max loss = premium，全亏你不影响心态？
- [ ] Strike 在 zone 内吗？
- [ ] Expiry：Jan +1 / Jan +2 / Jan +3 哪个？为什么？

### LEAPS 专项 5 问
1. **Thesis 持续度**：这个 thesis 12 个月后还成立吗？2 年呢？
2. **IV 时机**：当前 IV percentile < 60% 吗？（≥60 默认不入，除非 §六闸门表 4 条 override 全满足）
3. **资本机会成本**：花这笔 premium 而不买正股，因为什么？（资本受限 / 分散需求 / 杠杆需求）
4. **流动性**：bid-ask < 8%、OI > 50、daily volume > 5？
5. **退出预案**：DTE 90 决策点你计划怎么走？write 下来

### 入场前必算两个数（2026-06-08 back-port）
checklist 不只勾选——这两个数必须算出来写下：

1. **有效杠杆** = `Δ × 正股现价 ÷ Call 权利金`
   - 例：正股 $100、Δ0.55、premium $25 → 0.55×100÷25 = **2.2×**（正股涨 1%，call 理论涨 ~2.2%）
   - 动态值，随股价 / Δ / IV / DTE 变化。**判断**：ATM LEAPS 若只有 ~1.5× 却扛着归零 + IV + theta 三重期权风险 → 不如直接买正股；轻度 OTM 有 3–5× 且 thesis 强才值得做
2. **Break-even**（到期口径）= `Strike + 权利金`
   - 算完接 Bayesian 那一问：**"我的时间窗口内，正股到达并超过 break-even 的后验概率 × 赔率 = EV 正不正？"** 答不上 = 直觉，不是投资
   - 注：这是**到期** break-even（保守）。LEAPS 基本提前平，提前 exit 因保留剩余时间价值，实际 break-even 更低

### 赔率三情景（入场前填，叠在评分卡之上）
不只算单点目标——三个情景一起看，这才是"方向对、涨幅不够也可能亏"的 LEAPS 本质：

| 情景 | 正股到价 | Call 到期内在值 | 判断 |
|------|---------|----------------|------|
| **目标兑现** | 你的乐观 PT | max(PT − strike, 0) | 上行收益率够不够补归零风险 |
| **只涨一半** | (现价 + PT) / 2 | max(· − strike, 0) | **关键诊断**：半程还赚吗？OTM 半程常仍亏 |
| **横盘 3–6 月** | ≈ 现价 | 仅剩时间价值（损耗） | theta 扛得住吗？（LEAPS 温柔但非零） |

> "只涨一半仍亏" = strike 选太远，你在赌幅度不是赌方向 → 往 ATM/DITM 收，或承认这是 lottery 仓（按 §九 trading-tier 小 size + target/stop/time）。

### 评分卡

| 项目 | 2分 | 1分 | 0分 |
|------|-----|-----|-----|
| Thesis 持续度 | 强（多季度催化） | 一般 | 短期 catalyst（用短期 call 不是 LEAPS） |
| IV 时机 | percentile < 30 | 30-60 | > 60（贵的保险；override 入场计 0 分如实扣） |
| 资本理由 | 明确（杠杆/分散/资本受限） | 一般 | 没想清楚 |
| 流动性 | 优（< 5% bid-ask, OI > 200） | 中 | 差（> 10% bid-ask 或 OI < 50） |
| 退出预案 | 写清楚 | 模糊 | 没想 |

- 8-10：健康，执行
- 5-7：可做，偏激进
- ≤ 4：不做

---

## 八、与 G-01/G-02/G-03 的接口

| 规则 | LEAPS 适用 | 说明 |
|------|----------|------|
| **G-01** (+100% partial exit) | ✓ 适用 | DITM 走 Tier C (floor 20%)；ATM 走 Tier B (floor 30%)；OTM 走 Tier A (floor 50%) |
| **G-02** (短 DTE theta 复核) | DTE > 30 不适用，DTE ≤ 30 适用 | LEAPS 进入最后 30 天等于短期期权 |
| **G-03** (财报前 5 日强制三选一) | 部分适用 | LEAPS 财报前 IV crush 影响小（vega 占 premium 比例较小），> 9 月 DTE 可考虑 hold through，但仍需评估 |

---

## 九、与 Position Tiers 映射

| Tier | LEAPS 类型 |
|------|-----------|
| **Base** | Stock replacement DITM LEAPS（thesis 5+ 年）— 锁死管理，不 trim 信号 |
| **Core** | Thesis expression ATM/OTM LEAPS — 按 zone 信号管理 |
| **Trading** | 准 LEAPS（Jan +1 年）的事件押注 — 有 target/stop/time |

---

## 十、第一笔 LEAPS 的推荐姿势

**强烈推荐第一笔走 Stock Replacement**：

**理由**：
1. 你执行 mechanism 是 one-shot lump-sum，stock replacement 的"买入持有"匹配
2. DITM vega/premium ratio 低，心理压力小
3. Delta 0.80+ 体感接近持股，不容易被波动吓出
4. 最坏 "亏 25-40% premium 但跑赢卖股票"

**具体配置建议**（保守首选模板；实际第一批 LEAPS 走了 §11 的 aggressive 路径，见下方源案例）：
- 标的：base 仓 thesis 最强的多年 cycle 名
- Strike：现价 -15% 到 -20%（DITM Δ 0.80-0.85）
- Expiry：Jan +2 年
- Size：MIN(分数 Kelly, concentration cap, ladder 预算)，不写死 %（2026-06-11 去掉过时的账户金额）

**避免第一笔做的事**：
- 不做 OTM LEAPS lottery（赔率好但归零率高）
- 不在 IV percentile > 60 时入
- 不在 LEVEL5 满杠杆时叠加（先 trim 现有再开新）

---

## 十一、Strike / Expiry 精化 + 替代用法（2026-06-04 从 memory back-port）

### 11.1 Strike 锚分类：DITM=支撑 / OTM=阻力（别混用）
选 strike 前先分 intent，再用**对应方向**的 4 信号 anchor 集（配合 [[strike-triangulation]]）：

| Intent | Strike | Δ | 4-signal anchor 集 |
|--------|--------|---|-------------------|
| **DITM** stock replacement | spot −15~25% | 0.75–0.85 | parabolic invalid_if floor + breakout volume base + 高 OI 机构锚 + 多日 close support pivot |
| **OTM** leveraged thesis | spot +5~30% | 0.40–0.65 | parabolic trim_zone center + 分析师 PT 中点 + Option OI magnet + fib 1.618 延伸 |

**严禁**：用 support 信号锚 OTM strike，或用 resistance 信号锚 DITM strike —— 方向相反，混用得 0–1/4 收敛。OTM call 的 strike 是你 **take profit / 被 cap 收益**的位置，不是你 defend 的位置。
源：STOCK_X 270115 $15C（2026-05-04），$15 = trim_zone 中点 + 分析师 PT + 链上最高 OI + fib 1.618 四信号 resistance 收敛。

### 11.2 Expiry：单张 PnL vs 等资金 PnL 反向（Jan +1 vs Jan +2）
section 四的 Jan+2 默认是"单张 PnL"视角。**等资金视角相反**：
- **单张 PnL**：Jan +2 几乎全胜（time value 保留多）
- **等资金 PnL**：标的 hit target 时 Jan +1 大胜（同样钱买 1.5–1.7× 张数 = 杠杆放大）
- **Crossover**：标的需到 **strike + 25–30%** 才让等资金 Jan +1 反超

选 Jan +1（短月）必须**显式接受 3 个隐性成本**：① 必须 12 月内主动 exit（失去后续 thesis 验证参与）；② 中途 30% 回调因 DTE 缩短崩得更快（路径风险）；③ 标的真大涨你已落袋，Jan +2 还能继续吃。
→ **高 conviction + 明确 6–12m catalyst window → Jan +1（杠杆胜）；不押 timing / 长期 replacement → Jan +2（容错胜）。**
源：STOCK_X 270115 vs 280121 $15C —— 横盘场景 Jan+2 容错 2×，target 场景 Jan+1 等资金多赚 $1.2k–4.8k。

### 11.3 现货 no-chase 锁死 → LEAPS ladder 替代（不追现货）
现货 thesis 完好但价格已穿 no_chase_above（parabolic）→ 不追现货（perfect-price chase, Rule #14），用 LEAPS ladder 表达 core-tier delta。
- **Capital efficiency**：1 张 Δ0.55 = 55 股等价，资本占用 ~34%（$247 vs 55 股 $717）。
- **Trigger（4 全满足）**：① 现货已持仓 ② thesis 完好（catalyst + 分析师确认）③ 价 > no_chase_above 持续 ≥1 日 ④ parabolic-mode confirmed。
- **Action**：3-tier LEAPS ladder（per Greedy-Bid Trap 锚）+ OTM strike（per 11.1）+ tier=core + sizing 受满-ladder 25% 约束（[[risk-capital-framework]] §3）。
- **不允许**：现货 thesis 受损（该 stop/trim 不该加杠杆）／已有多个同向 LEAPS（vega 集中失控）／LEVEL5 + 已有 active override（等释放）。
源：STOCK_X 5/4，现货 100 股在 waiting zone 顶不能加 → LEAPS 首档 5 张 @ $2.47 = capital-efficient 的 core 升级。

### 11.4 确定性 decomposition → DTE/strike 必须匹配兑现窗口
听到"确定性高"先拆三段，别把一段的确定性借给另一段：
1. **EPS 兑现**：盈利真在增长？常**已发生 + 已涨过一波**（已定价）
2. **PE re-rating**：倍数在扩张？**大空间的真正来源**，常**才刚开始**（大头在前，需时间）
3. **时机/路径**：方向确定 ≠ 路径确定 ≠ 入场时机对

**确定性高 ≠ 大空间**：高确定性常已被定价（已涨）；大空间要的是未被定价的超预期或未完成的 re-rating。
→ **DTE 必须匹配 thesis 兑现窗口**：re-rating 早期 = 大头在 1–2 年后 = 需长 DTE。**strike 和 DTE 不能赌不同的事**：OTM=赌大空间（需时间发酵）+ 短 DTE=赌短期 → 两头不靠。
源：2026-05-31 STOCK_D/STOCK_K —— EPS 已兑现（+900%/+127%），PE re-rating 才开始（6x，大头在 2027）→ 选 STOCK_D 2027-06 而非 2026-12。

### 11.5 Strike 收益量化：crossover vs 目标价 + IV 调整（2026-06-09）
**"越 OTM → delta 越低 → 杠杆越高 → 收益越可观" 是 IV + 目标价的条件命题，不是无条件真。** 入场前用 `quant/options/strike_compare.py` 算，别凭直觉判断收益。
> ⚠️ 本节 = **Mode A：hold-to-expiry / 持有期 IV 下行**（STOCK_D washout 型）。**提前平仓 + IV 上行**（催化剂 ramp 型）下更 OTM 反而会赢 —— 见 §11.6。先判断你属于哪个 mode 再用本节。

**到期 crossover**（同方向同到期两 strike，K₁<K₂，premium p₁>p₂）：`S* = (p₂·K₁ − p₁·K₂) / (p₂ − p₁)`
- 标的到期 **> S*** → 更 OTM 的 K₂ 收益才反超 K₁
- 目标价 **< S*** → 低 strike K₁ 收益更高（OTM 的杠杆没激活，因为这次 move 不够大）

**为什么高 IV 杀 OTM 杠杆**：OTM 加杠杆的前提是 OTM strike 便宜到补偿放弃的 intrinsic。IV 越高 → OTM 的 extrinsic 越贵 → 同样的 premium 差换不回足够 intrinsic → **crossover 被推高、远离你的目标价**。两条推论：
- **washout / 事件恐慌入场 = IV 峰值 = 买 OTM 最差时机** → 反而该往 DITM / 低 strike 靠（多拿 intrinsic、少付被吹高的 extrinsic）。与 §11.1（DITM=支撑）、memory `options_vehicle_selection.md` §IV-pairing（高 IV 倾向 sell premium / 低 strike）同向。
- 涨到目标的**路径**通常 IV 回落 → 高 vega 的 OTM **双重受损**（到期内在值低 + 路径上 vega 流血）。`strike_compare.py --exit-dte --exit-iv` 用 BS 量化这一段。

**入场前必跑**：`python3 quant/options/strike_compare.py --strikes K1:p1 K2:p2 --targets ... --entry-iv <IV>`。先确认目标价 > crossover，再谈"往 OTM 推"；目标价 < crossover 还选 OTM = 赌一个连你自己目标价都没覆盖的更大空间。

**与 §11.4 接口**：§11.4 管**时间维度**（strike/DTE 匹配兑现窗口）；§11.5 管**空间 + 波动率维度**（strike 收益 = crossover + IV 的函数）。两者合起来 = strike 选择的完整量化。

源：2026-06-09 STOCK_D washout 后选 strike。用户 直觉"目标 $100 该买 $70C（更 OTM）而非 $60C"。实测（周五入场价 $60C $13.90 / $70C $11.10）：crossover = **$109.64**，目标 $100 在其下 → **$60C 收益更高（2.88x vs 2.70x）**，$70C 要 STOCK_D > $110 才反超。错因：周五 IV 88%（washout 峰值）把 $70C 的 extrinsic 吹贵（只便宜 $2.80 却在 $100 少拿 $10 intrinsic）。**高 conviction 长期 thesis 也不该无脑往 OTM 推，除非目标 > crossover 且 IV 不在峰值。**

### 11.6 Mode B — 提前平仓：gamma + IV 方向（at-expiry crossover 在这里失效）（2026-06-09 STOCK_M 补强）
§11.5 的 crossover **只管 hold-to-expiry**。**催化剂 / 动量交易多数是提前平仓** —— 这时时间价值（gamma + vega）主导，crossover 会给出**相反的错答案**。

§11.5 没算的两股力：
- **Gamma**：大涨把 OTM 从深 OTM 拉向 ATM，delta 飙升 → OTM 的 % 收益被放大。**移动够大，OTM 杠杆才激活**（温和移动则低 strike 赢）。
- **Vega / IV 方向**：**IV 升**（财报/催化剂前 ramp）→ 高 vega 的 OTM 被吹大 → **利好 OTM**；**IV 降**（washout 峰值后 crush，= §11.5）→ 利好低 strike。

**STOCK_M 源案例（STOCK_D 的镜像，必须成对记）**：5/11 入场 $150C $51.10 / $200C $32.22，标的 $170.84 → 5/26 $208.26（+21.9%，财报前 IV 峰值）。
- **同一标的价 $208，strike 排名完全相反**：
  - 持到到期：$150C **+14%** / $200C **−75%**（$200C 只剩 $8 内在值）→ at-expiry crossover $285，$150C 完胜
  - 5/26 提前平：$150C **+61%** / $200C **+82%** → **$200C 赢**
- 差异 100% 是时间价值（gamma + 财报前 IV ramp）。持到 5/27（财报后 IV crush）→ 收窄到 +45% vs +59%（IV 那部分吐回，但大涨的 gamma 收益留住 → $200C 仍赢）。

**完整规则（合并 §11.5 + §11.6）—— 选 strike 前先答三问：**
1. **持到到期 还是 提前平？** 到期 → Mode A crossover；提前平 → Mode B（看下面两条）。
2. **移动多大？** 温和 → 低 strike；大涨（冲过 crossover 区）→ OTM 的 gamma 才激活。
3. **持有期 IV 升还是降？** 催化剂前 ramp（升）→ 利好 OTM；washout 峰值后（降）→ 利好低 strike。

STOCK_D 低 strike **只是略胜（margin 薄、依赖假设）**：at-expiry $60 赢；提前平则看 exit IV —— IV 真 crush 到 ~50% + 接近到期 → $60 赢，IV 只软到 60% + 时间还多 → **$70 反超**（脚本 `--exit-iv 60 --exit-dte 195` 实测 $70 3.18x > $60 3.10x）。STOCK_M 高 strike 赢 = 提前平 + 大涨 + IV 升，三问都指向 OTM。**教训：连"低 strike 赢"的 STOCK_D 都是薄 margin 的 regime call —— 没有便携结论，每次绑定"持到到期 vs 提前平 + 预期 exit IV"跑脚本，别背规则。****"越 OTM 越赚" 在 Mode B 大涨 + IV 升下成立；在 Mode A / 温和移动 / IV 降下不成立 —— 没有通用答案，只有 regime。** 跑 `strike_compare.py --exit-dte --exit-iv --entry-iv`，脚本会在两 mode 排名相反时报警。

**实操推论（2026-06-09 STOCK_D $60→$70 三情景验证；前提：总是提前平仓 = 永远 Mode B）**：同一 $60→$70 move、同 Dec26 $60C/$70C、同提前平，strike 赢家完全由 IV 方向决定 —— IV 升 100% → $70C +67% vs $60C +51%（OTM 大胜）；IV 平 85% → $70C +44% vs $60C +36%（OTM 小胜）；**IV crush 到 55% → $70C −4% vs $60C +5%（OTM 亏钱，即使方向全对）**。
- **两个旋钮分开，别焊死**：**先验概率定 size**（越高赌越大，Kelly）；**IV 方向定 strike**（低/升 IV → OTM；峰值 IV → 低 strike/DITM）。"先验越高越买 OTM" 把两旋钮焊成一个 —— 错。
- **washout 抄底的特别陷阱**："非基本面 washout → 高先验 → 买 OTM" 里，washout 本身 = 峰值 IV 入场，反弹常伴 IV crush —— **给你高先验的那个事件，正是制造 IV 逆风的同一个事件**。washout 买 OTM 踩在 OTM 最吃亏的 regime（拿 STOCK_M 的 IV-ramp 成功套 washout 的 IV-crush = 方向反了）→ washout 抄底反偏低 strike/DITM。

**OTM LEAPS 的两轴证伪（持仓纪律，2026-06-09）**：正股无限期，只看"基本面证伪"就够；**LEAPS 有 DTE，会死于时间，即使 thesis 没坏**。所以 OTM LEAPS 的退出是**两条轴，不是一条**：
1. **Thesis 被破坏**（基本面坏）→ 退出。
2. **Thesis 完好 + 按时兑现**（价格推进、re-rating 在发生）→ 持有 / 滚动。
3. **Thesis 完好但 stalled**（没坏，但 re-rating 没在 DTE 窗口内发生，价格原地踏步）→ **软退出**。它没"破坏"，但 theta + DTE 在清算它 —— 轻度 OTM LEAPS 就是这样"你对了、只是太早"地悄悄归零。"thesis 没坏就一直 roll" = §六 `滚动续命` 反模式（越滚越贵）。

→ **每张 OTM LEAPS 入场即配一个进度 checkpoint**："re-rating 必须在 {日期 / 财报} 之前显现"，到点没动静 = 软退出，哪怕 thesis 还没坏。即 OTM LEAPS 的证伪含"时间到了还没穿越 K"，不只是"基本面坏了"。接 §11.4（DTE 必须匹配兑现窗口）。

## 十二、退出粒度 → 张数/Strike 选择的显式输入（2026-07-17 回测立项）

**问题**：G-01 阶梯（+100/+200/+300 → trim 30/30/40）和棘轮 stage1（破锁利地板卖 50%）的分批设计在 **qty=1 的仓位上全部退化为全平**——单张仓无法 partial，地板一破就是二选一（全平 or 全扛）。2026-01→07 的期权仓大多 1-2 张，分批退出栈实际从未按设计运行过（半年回测中 G-01 仅在 STOCK_X 11C ×10 与 STOCK_D 60C ×2 上有机会触发）。

**规则**：同等 premium 预算下，**≥2 张较低价合约优先于 1 张贵合约**（需要更高名义敞口时用 spread 而不是加价钱买单张深度合约）。"能不能分批退"是 strike/expiry 选择的显式输入，与 §三 决策树、§11.6 crossover 分析并列。例外：DITM 替代正股用法（§二 用法1）单张可接受——它的退出本来就是一次性的（正股逻辑）。

**下单前自问**：这个仓位触发 G-01 +100% 时，我能只卖 30% 吗？"不能" → 换结构，或接受"地板破 = 全平"并写进 entry 注解。

## 十三、Strike 唯一主锚 = delta 带 + DTE 比率规则（2026-08-01 定稿，STOCK_GO 7/31 案例日全链评审）

> 来源：2026-08-01 评审（canon 审计 + STOCK_GO 实盘期权链 + 红队攻击），用户 批准落实。
> 起因是「strike 定在下一个 confirmed gate」提案——方向感对（OTM 锚阻力），但被否，本节是替代定稿。

### 13.0 资格闸：期权 vehicle = confirmed-only（2026-08-10 用户「这个必须」）

> **2026-09-02 定版（后裁决优先；canonical = [[2026-09-02-long-call-entry-directive]]）**
> 本节下方 08-10/08-13 文本只读留存。现行资格闸三条，代码真源 `quant/core/sizing.py`：
> 1. **gate1 起可期权**（`OPTION_MIN_GATE=1`）——用户：「不再限制 gate2 才允许期权入场 ……
>    long call 的目的就是要同时吃波动率和方向 所以最好是在低点布局 …… 但并不代表可以随意
>    开仓 依然需要等见底条件」。见底条件 = 系统里第一个**给 cap 的** gate（收复线/止跌线
>    ×N 日 + RelVR ≥1.2，`structure_repair_veto` 放行），与正股入场资格**同一套**，挂 z1/z2
>    限价、不在确认日盘中追（STOCK_E 560C 8/26 案：gap 日盘中 71 买入、次日 −10.6% 而正股 −0.9%）。
> 2. **IV 硬门槛**（`long_call_iv_gate`）：percentile <40（≥60 样本可信）/ 序列不足时 HV20 的 252 日
>    分位代理（2026-09-11，同阈值 <40）/ 两个分位都没有才退绝对 ATM IV <50% 兜底 / 无数据不放行。数据：三对可对上 IV 记录的 gate1→gate2 转移 IV 全部回落（STOCK_Z
>    149→112 / STOCK_L 114→91 / STOCK_Y 93→67），Δ0.7 270DTE call 在正股 +4.4~8.8% 时仍 −6~−21%
>    ——见底点恰是 IV 峰值，「同时吃波动率」在高 IV 票上是反的。高 IV 票 gate1 一律正股。
> 3. **退出**：S1 −25→**−20%**（§13.4）+ **S5g gate 退出线** = 授予 cap 的 gate 本身 ×1 日收盘
>    （`granting_gate_level`）——用户：「跌破 553 就果断离场 虽然是以 gate 作为退出条件 但这是
>    含杠杆的期权 所以可以接受稍高的止损率」；正股仍用 bfloor_buf（gate − 0.5N）。
>
> **2026-09-11 补：第二入场类 `reversal_probe`**（用户 三裁，canonical [[2026-09-11-reversal-probe-directive]]）：
> 不等 gate，反转阳线打破下跌结构（`core.reversal`）即出合约计划，只开 R1 域（距 60 日高回撤 10–20%），IVP <40
> 用 HV20 分位代理，止损 = 支撑收盘 ×1 **无 S1**，cap = gate1 2.5% NAV ÷ 支撑处保费投影亏损（不是固定 12.5%），
> 0 张 = veto。本节以下 gate 类条款对该类不适用之处以 directive 为准。
>
> 保费上限随 |S1| 换分母：gate1 **12.5%** / gate2 7.5% / gate3 5% NAV（本档新增 risk ÷ 0.20，
> 仍不跟累计）。⚠️ 小账户下这个上限常常不到一张：池内 Δ0.7 270DTE 单张保费约为上限的 1.2×（STOCK_IN）/
> 2.2×（STOCK_A）/ 5.3×（STOCK_E）——0 张的菜单 = 正股或 veto，「1 张书面例外」未裁。
> 账户自身样本（19 笔 long call/spread，合计净亏）：60 日区间低位/中段入场 6 笔仅 1 胜，
> 高位 8 笔 3 胜（STOCK_M ×2、STOCK_Y 价差）——「低位 call」在本账户没有正样本，本裁决是风险偏好选择。

**期权 vehicle 的入场资格 = entry_guard GO 且 classification = confirmed。probe / early_risky 只能正股。**
机制：probe 的语义是"方向未确认轻仓试探"，用带杠杆+时间衰减的 vehicle 做试探，亏损形态与试探意图
不匹配（试探错了应该小亏，不该 -41.5%）。源案例：STOCK_GO 270115 390C，2026-08-05 `early_risky` +
`above_no_chase` 入场 $35，4 个交易日 spot −6.2% 时 premium **−41.5%**。
代码强制点：`entry_guard.evaluate_ticker` 期权分支前置闸（非 confirmed → 期权计划不生成 + ⛔ 播报，
渲染层同拍不给"手动跑 option_plan"话术）；测试 + 变异验证在案（`tests/decision/test_entry_guard.py`）。
另：2026-08-10 用户 定期权**管理口径 = 权利金**（zone/stop/trim 直接对 premium 画线；盈利侧照搬正股
退出栈 + underlying regime trim 第 5 线，premium 基取 max、underlying 基并行 OR；趋势死亡线保留
underlying 口径）。亏损侧 S1–S4 参数见 `quant/research/options_exit_overlay_design.md`（预注册）。

### 13.1 Strike：delta 带唯一主锚，结构位退出 long 腿决策

**原理**：趋势系统由 exit_guard 触发退出、从不持有到期 → strike 的唯一真实功能 = 在波动率曲面上选 (Δ, Γ, vega, θ) 组合点。"价格到不到某个位"与已实现 P&L 无关。把 strike 交给结构位距离 = 三重错误：①moneyness 变成画卡日期的随机函数（STOCK_O 实测同方法 gate 198→234→198）；②反向选择——gate 近（阻力贴脸、setup 最弱）拿最高 delta，gate 远（跑道干净、setup 最强）拿彩票 delta，攻击性系统性配错方向；③违反 §11.5「没有便携结论」铁律。

| 参数 | 取值 | 依据 |
|---|---|---|
| 主带 | **Δ0.60–0.80**（intrinsic ≥40% premium） | 本账户退出执行弱 + 确认日入场时点 → extrinsic 必须压低当缓冲 |
| 放宽档 | Δ0.55–0.65，仅当 IV percentile < 30-40 | canon 轻度 OTM 档（§二 用法2）保留但加 IV 前置 |
| 绝对禁区 | **Δ < 0.40** | OTM lottery ban 的量化版 |
| ATH 情形 | **零特判** | delta 带不依赖上方结构存在——"ATH 时 strike 没法锚"这个问题本身证明 strike 从来不该锚结构 |

**结构位（gate/HVN/trim）在期权上的三个合法角色**：thesis 验证、underlying 退出线、**debit spread 短腿定位**。gate 出现在 long 腿 strike 决策里即违规。§11.1 的 OTM 阻力锚集降级为带内 tie-break 参考，不再是独立锚。

**伪锚黑名单**（2026-08-01 红队五方案评审）：OI call wall（pin 力 ∝ gamma，只在 0-30 DTE 有效，锚 12 个月合约 = 时间尺度错两个量级，且 券商 OI 无 dealer 方向）；固定 % OTM（scale-variant：同一 10% 在 σ60% 票近 ATM、σ25% 票是 lottery，违反全系统 ATR 归一原则）。ATR 倍数投影仅作 chain 数据不可用时的 fallback（k 必须按 σ√T 校准到等效 delta 带）。

### 13.2 DTE：比率规则（绝对下限 270 的数学本体）

**T_entry ≥ 3× 赢家路径持有期**——不是中位持有期。实测（2026-08-01 事件流，期权 18 笔 round trip）：中位 23d 但亏家 1-14d 就被砍（theta 无关紧要），赢家 40-66d 且被 7/16 清账截断；正股赢家 campaign 76-188d（STOCK_N/STOCK_Z/STOCK_L/STOCK_Y）。**期权久期为赢家路径设计，不为平均交易设计。**

- 按当前实测赢家 → T ≥ **270**（canonical 下限的来源，不是魔法数）
- 按设计目标（棘轮让 winner 跑数月）→ **365-540（Jan+2）才是对齐档**
- **退出地板：剩余 DTE ≤ 90 无条件 close-or-roll** = §13.4 S4（2026-08-10 数据裁：本节 8/1 初稿写 120，合成 1967 campaigns S4-90 mean +14.1% vs S4-120 +13.1%、p10 持平 → 90 弱优，用户 终裁随 overlay 接线，代码真源 `exit_guard.PARAMS["opt_s4_dte_floor"]`）。T<120 后月燃烧 >15% 的物理不变——120→90 段是 roll 决策带不是安逸持有带，§5 的 DTE 90 强制重审同拍；OTM 残仓提前至 ≤150
- **DITM 例外**：Δ ≥ 0.70 且 extrinsic ≤ 25% premium 时 floor 可放宽至 ~180-200（extrinsic 只有 ATM 的 1/3，DTE 弹性大增）
- **IV 分支**：入场 IV pct 低 → 上探 Jan+2（vega 是资产，flat-IV 下成本单调降）；确认日 IV 偏高 → 270-365 平台或直接 DITM 化让 DTE 降维
- **执行周期（2026-08-11 用户 修订：移除「最近 Jan 周期」限制）**：实际执行 = 满足下限的**最近可用周期，不限 Jan**（代码 `entry_guard` → `option_plan.build(target_dte=270, min_dte=270)`，`pick_expiry` 下限单独成参——纯最近会选中更近但破下限的到期日）。流动性交给 §13.3 硬门槛把关（spread ≤5% / OI ≥200 / bid>0）；「Jan 周期 OI 厚 3-10×」（STOCK_GO 7/31 实拉）降级为最近周期流动性不过关时的上探参考，不再是执行约束。旧规则下 270 下限实际恒被执行成 Jan+2（写作当日最近合格 Jan = ~525d），下限首次真实 binding
- roll = 新 campaign，重过 IV 条件与 sizing（"DTE>180 就算 LEAPS"的私有定义否决——180 是定义边界不是操作点）

### 13.3 期权版整股约束（share_ladder 镜像）

```
contracts = floor(cap ÷ (ask × 100))
  ≥1 → 正常
   0 → 降级 debit spread 或正股，或 veto
```

> **2026-08-11 scope 限定（用户 再确认「只用单腿 long call」）**：上行 spread 降级项在
> 现行 scope（全书无卖方腿，option_strategy_scope 2026-08-07/08-11）下**不可用**——
> 0 张的实际菜单 = **正股或 veto**。cap 口径同日定：保费上限 = min(R1 轮预算 2.5%/0.25
> = 10% NAV, campaign at-risk 5%/0.25 = 20% NAV)，R1 恒 binding（risk_allocation §2b）。
> **2026-09-02 口径**：保费上限 = min(gate 本档新增 risk ÷ 0.20 = gate1 **12.5% NAV**，
> at-risk 5%/0.20 = 25% NAV)，gate1 恒 binding；小账户下常不足一张 → 0 张 = 正股或 veto。

**严禁用降 delta 装 cap**——cap 压力是 OTM lottery 从 sizing 后门溜进来的主通道。delta-notional（Δ×100×S）计入 25% 单票集中度 cap，红线 50%；小账户下同时 active 的 LEAPS campaign ≤ 1。源数据（STOCK_GO 2026-07-31 实拉）：Jun27 370C 单张保费 = 4× confirmed cap，delta-notional = 87% NAV——**该 NAV 下 $300+ 票的 naked LEAPS 不存在**，菜单只有正股或 defined-risk spread。

流动性硬门槛收紧（LEAPS 腿）：spread ≤ **5%** of mid（option_plan 现行 15% 对 LEAPS 太松）、OI ≥ 200、bid > 0。GO 后允许 1-3 日限价，不抢确认日（确认日 IV 常处抬升后）。

### 13.4 期权亏损侧退出栈（✅ 2026-08-10 定版接线，用户 终裁）

现有告警栈（棘轮/G-01/趋势死亡线）**全在盈利侧或盯 underlying**——棘轮武装前的裸窗口零覆盖。
构造性盲区（2026-08-01 立项原文）：OTM 横盘 + IV 回落 → premium -64% 零告警;活案例 STOCK_GO
2026-08-05:spot −6.2% 时 premium **−41.5%**,underlying stop 慢一个量级。

**定版参数**(代码真源 `exit_guard.PARAMS`,heartbeat 每 6h 巡逻,告警原文推 TG):

| 线 | 值 | 语义 |
|---|---|---|
| **S1 premium 硬 stop** | **−20%**（2026-09-02 用户「收到 20%」；08-10 定版 −25。42 张重放：p10 −27.5 vs −32.3 好、mean +24.8 vs +32.6 差、whipsaw 95% vs 91%，知情换尾部） | mark ≤ 加权成本×0.80 → 清仓。收复 +2% 重新武装 |
| **S2 时间 stop** | **(20, 25)** | 入场后 d20 underlying 未收上入场 close+1N → 减半(单张不可拆降级提示);d25 → 清仓 |
| **S4 DTE 地板** | **90** | ≤90 无条件 close-or-roll(roll = 新 campaign 重过 IV+sizing) |
| S3 IV×横盘 | advisory | IV 自入场 −8pts 且未 +1N → [FYI] 行(真实案例 STOCK_Z200C −51.9%→−5.3%,样本 2 不裁) |
| S5 underlying stop | 保留 | 与 S1–S4 为 OR 双防线 |
| **S5g gate 退出线** | **授予 cap 的 gate 本身 ×1 日**（2026-09-02，STOCK_E 553 案） | underlying 收盘跌破 → 清仓，不等正股 bfloor_buf；收回 gate 之上重新武装；`sizing.granting_gate_level` → `exit_guard` 🚨 `opt_gate_stop` → brief/画线 `stop:gate` |

盈利侧照正股退出栈:premium 基(棘轮保本/回吐上限/G-01)同基取 max,underlying 基(趋势死亡线/
regime trim)并行 OR(用户 2026-08-10)。

**回测链**(三证据体,判据与全量数字见 `quant/research/options_exit_overlay_design.md` +
`synthetic_longcall_design.md` + `tmp/backtest/` 三份报告):17 笔实盘回放(切片组合全正,
损失省 61%,STOCK_X 强平案 S2 拦截)· 合成 1967 campaigns(S1 whipsaw 62-69%)· **真实 42 笔
Jan27 重放(whipsaw 91-100%,STOCK_Z +399% 被 S1-30 d4 砍成 −31% 案)**。S1=−25 为 用户
知情保守裁决——回测窗 2026-03 末起为 AI 牛市,whipsaw 读数被「什么都涨回来」抬高不可外推,
紧档 mean 代价读数同被牛市偏置污染,保守取紧。**+12 月复测窗(含更多 regime)复核 whipsaw。**

**联动**:at-risk 期权系数放宽 保费全额 → **保费 × 0.25**(锚 S1,pins 测试钉相等; **2026-09-02 随 S1 → 0.20**, 跳空日止损线 ≠ 损失的低估已知——STOCK_GO 390C 8/05 收盘 −24.7%;
`sizing.OPTION_ATRISK_LOSS_FRAC`,用户 2026-08-10 知情裁决;5% cap 下单笔保费上限 ≈ 20% NAV)。放宽前提 = S1 会被执行——首笔 S1 触发未当日执行 = 前提证伪,复核系数。

## 与期权框架的关系

本文件为 LEAPS Call 单一策略的操作手册。完整期权策略体系（决策树、其他策略）详见 [[options-strategy-framework]]。Greeks 操作规则（G-01/02/03）详见 [[greeks-discipline]]。

**框架定位**：LEAPS 是 Long Call 的 DTE 365+ 变种，不是独立的 Tier 1 策略，但因 Greeks profile 完全不同 + 适用场景明确，单独建文件管理。

---

## 源案例（2026-06-11 回填——完整 entry 级记录见 期权日志）

- **STOCK_X 270115 $15C ×16**（5/4-5/15 三档 ladder）：no-chase 锁死现货 → LEAPS 替代；IV 71% 走 4 条 override；Jan+1 timing bet 显式声明（§11.2 等资金对比的活案例）
- **STOCK_Z 270115→270617 $140C**（5/6 入场 + 5/8 roll）：DITM Δ0.78 四信号 4/4；roll = 时间 arbitrage（per-day extension cost < theta drag，§11.2 crossover 实算）；independent book 概念诞生处
- **STOCK_M 260821→270115 $150C**（4/27 入场 + 5/11 roll）：时间错配修正（6 月爆发 thesis vs DTE 102）；EV +$613 favor roll；§11.5 Mode A/B 源头
- **STOCK_IN 270115 $120C**（6/1）：错杀 dip 单张 single-shot，6/3 promote core Tier1
- **STOCK_D Dec26 $70C / Jan27 $60C**（6/1 + 6/5）：probe ladder 首档 A 级 vs reactive 加仓 D 级——同一标的两种 process 的对照组
- 反例参考：STOCK_R 6/18 20C deep OTM（非 LEAPS 短月赌幅度 → humility anchor → force close -$780），见 [[natural-humility-anchor]]

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[options-strategy-framework|期权交易框架]]、[[greeks-discipline|Greeks Discipline]]
