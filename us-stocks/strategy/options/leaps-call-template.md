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
1. **决策层 — Bayesian 概率模型**（`../bayesian-decision-model.md`）：该不该押、押多大，用后验分布 + EV 决定，不是看 thesis 顺不顺眼。本手册只回答"押了之后用什么 vehicle / strike / DTE / expiry"，**不回答"该不该押"**。
2. **Vehicle 选择 — IV regime 闸门**（memory `feedback_iv_adjacent_pairing`）：IV > 70%ile **倾向** sell premium（spread / sold put）而非买 long LEAPS。先确认 LEAPS 是对的 vehicle，再进本手册选 strike。

> 把本手册当孤立的 LEAPS 选择器 = 丢掉概率层和 vehicle 层 = 系统性偏 OTM + 在高 IV 乱买 long call。这正是外部框架（如纯 GPT-discussion 版）最容易缺的两层。

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
| IV percentile | < 60%（canonical 闸门表：`options-strategy-framework.md` §六） | ≥60 默认不入；4 条 override 全满足才放行（parabolic AI + uncapped chain + LEAPS DTE + size 减档），且偏 DITM |
| Bid-ask spread | < 8% of mid | 换个 strike 或 expiry |
| Open Interest | > 50 contracts | 流动性不足，弃 |
| Volume (近 5 日均) | > 5/day | 流动性不足，弃 |
| Underlying thesis | ≥ 12 月可信 | 短期 long call 替代 |
| Position sizing | MIN(分数 Kelly, 25% concentration cap, ladder 预算)——`kelly-position-sizing.md`；LEAPS independent book 不进 stock pyramid 链但受同 ticker cap | 减 size 或弃 |

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
选 strike 前先分 intent，再用**对应方向**的 4 信号 anchor 集（配合 `../strike-triangulation.md`）：

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
- **Action**：3-tier LEAPS ladder（per Greedy-Bid Trap 锚）+ OTM strike（per 11.1）+ tier=core + sizing 受满-ladder 25% 约束（`../risk-capital-framework.md` §3）。
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
**"越 OTM → delta 越低 → 杠杆越高 → 收益越可观" 是 IV + 目标价的条件命题，不是无条件真。** 入场前用量化脚本（crossover + BS 情景）算，别凭直觉判断收益。
> ⚠️ 本节 = **Mode A：hold-to-expiry / 持有期 IV 下行**。**提前平仓 + IV 上行**（催化剂 ramp 型）下更 OTM 反而会赢 —— 见 §11.6。先判断属于哪个 mode 再用本节。

**到期 crossover**（同方向同到期两 strike，K₁<K₂，premium p₁>p₂）：`S* = (p₂·K₁ − p₁·K₂) / (p₂ − p₁)`
- 标的到期 **> S*** → 更 OTM 的 K₂ 收益才反超 K₁；目标价 **< S*** → 低 strike K₁ 更高（OTM 杠杆没激活，move 不够大）

**为什么高 IV 杀 OTM 杠杆**：OTM 加杠杆的前提是 OTM strike 便宜到补偿放弃的 intrinsic。IV 越高 → OTM extrinsic 越贵 → 同样 premium 差换不回足够 intrinsic → **crossover 被推高、远离目标价**。两条推论：
- **washout / 事件恐慌入场 = IV 峰值 = 买 OTM 最差时机** → 反而该往 DITM / 低 strike 靠（多拿 intrinsic、少付被吹高的 extrinsic）。与 §11.1（DITM=支撑）同向。
- 涨到目标的**路径**若 IV 回落 → 高 vega 的 OTM 双重受损（到期内在值低 + 路径 vega 流血）。

**入场前必跑量化脚本**：填 strikes + premiums + 目标价 + 入场 IV，先确认目标价 > crossover，再谈"往 OTM 推"；目标价 < crossover 还选 OTM = 赌一个连目标价都没覆盖的更大空间。
**与 §11.4 接口**：§11.4 管时间维度（strike/DTE 匹配兑现窗口）；§11.5 管空间 + 波动率维度（strike 收益 = crossover + IV 的函数）。

源：2026-06-09 STOCK_D washout 后选 strike，$60C vs $70C，目标 $100。crossover ≈ $109.6 > 目标 → at-expiry $60C 收益更高（2.88x vs 2.70x），$70C 要标的 > $110 才反超。错因：washout 峰值 IV（~88%）把 $70C 的 extrinsic 吹贵。**高 conviction 长期 thesis 也不该无脑往 OTM 推，除非目标 > crossover 且 IV 不在峰值。**

### 11.6 Mode B — 提前平仓：gamma + IV 方向（at-expiry crossover 在这里失效）（2026-06-09 补强）
§11.5 的 crossover **只管 hold-to-expiry**。**催化剂 / 动量交易多数是提前平仓** —— 这时时间价值（gamma + vega）主导，crossover 会给出**相反的错答案**。

§11.5 没算的两股力：
- **Gamma**：大涨把 OTM 从深 OTM 拉向 ATM，delta 飙升 → OTM % 收益被放大。**移动够大，OTM 杠杆才激活**（温和移动则低 strike 赢）。
- **Vega / IV 方向**：**IV 升**（催化剂前 ramp）→ 高 vega 的 OTM 被吹大 → 利好 OTM；**IV 降**（washout 峰值后 crush）→ 利好低 strike。

**镜像源案例（必须成对记）**：STOCK_M $150C vs $200C，提前平。**同一标的价，strike 排名完全相反**：持到到期 $150C +14% / $200C −75%（at-expiry crossover 说低 strike 完胜）；提前平 $150C +61% / $200C **+82%**（高 strike 赢）。差异 100% 是时间价值（gamma + 催化剂前 IV ramp）。对照 STOCK_D washout（IV crush）则低 strike 赢 —— 互为镜像。

**完整规则 —— 选 strike 前先答三问：**
1. **持到到期 还是 提前平？** 到期 → Mode A crossover；提前平 → Mode B。
2. **移动多大？** 温和 → 低 strike；大涨（冲过 crossover 区）→ OTM gamma 才激活。
3. **持有期 IV 升还是降？** 催化剂前 ramp（升）→ OTM；washout 峰值后（降）→ 低 strike。

**实操推论（三情景验证；前提：总是提前平仓 = 永远 Mode B）**：同一 move（如 $60→$70）、同两 strike、同提前平，赢家完全由 IV 方向决定 —— IV 升 → 高 strike 大胜；IV 平 → 高 strike 小胜；IV crush → 低 strike 赢（高 strike 甚至亏钱，即使方向全对）。
- **两个旋钮分开，别焊死**：**先验概率定 size**（越高赌越大，Kelly）；**IV 方向定 strike**（低/升 IV → OTM；峰值 IV → 低 strike/DITM）。"先验越高越买 OTM" 把两旋钮焊成一个 —— 错。
- **washout 抄底的特别陷阱**："非基本面 washout → 高先验 → 买 OTM" 里，washout 本身 = 峰值 IV 入场，反弹常伴 IV crush —— **给你高先验的那个事件，正是制造 IV 逆风的同一个事件** → washout 抄底反偏低 strike/DITM。

**OTM LEAPS 的两轴证伪（持仓纪律）**：正股无限期，只看"基本面证伪"够；**LEAPS 有 DTE，会死于时间，即使 thesis 没坏**。退出是**两条轴**：① thesis 被破坏 → 退出；② thesis 完好 + 按时兑现 → 持有/滚动；③ **thesis 完好但 stalled（re-rating 没在 DTE 窗口内发生）→ 软退出**（theta + DTE 在清算它，"你对了只是太早"）。"thesis 没坏就一直 roll" = §六 滚动续命反模式。
→ **每张 OTM LEAPS 入场即配进度 checkpoint**："re-rating 必须在 {日期 / 财报} 之前显现"，到点没动静 = 软退出。即 OTM LEAPS 的证伪含"时间到了还没穿越 K"，不只是"基本面坏了"。接 §11.4。

## 与期权框架的关系

本文件为 LEAPS Call 单一策略的操作手册。完整期权策略体系（决策树、其他策略）详见 `options-strategy-framework.md`。Greeks 操作规则（G-01/02/03）详见 `greeks-discipline.md`。

**框架定位**：LEAPS 是 Long Call 的 DTE 365+ 变种，不是独立的 Tier 1 策略，但因 Greeks profile 完全不同 + 适用场景明确，单独建文件管理。

---

## 源案例（2026-06-11 回填）

- **STOCK_X 远月 $15C ×16**（三档 ladder）：no-chase 锁死现货 → LEAPS 替代；IV 71% 走 4 条 override；Jan+1 timing bet 显式声明（§11.2 等资金对比的活案例）
- **STOCK_Z 远月 $140C**（入场 + roll 延 expiry）：DITM Δ0.78 四信号 4/4；roll = 时间 arbitrage（per-day extension cost < theta drag，§11.2 crossover 实算）；independent book 概念诞生处
- **STOCK_M 8/21→远月 $150C roll**：时间错配修正（6 月爆发 thesis vs DTE 102）；EV 量化 favor roll；§11.5 Mode A/B 源头
- **STOCK_IN 远月 $120C**：错杀 dip 单张 single-shot，后 promote core Tier1
- **STOCK_D 两档 ladder**：probe 首档 A 级 vs reactive 加仓 D 级——同一标的两种 process 的对照组
- 反例参考：STOCK_R 短月 deep OTM call（非 LEAPS 赌幅度 → humility anchor → force close -$780），见 `../natural-humility-anchor.md`

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[options-strategy-framework|期权交易框架]]、[[greeks-discipline|Greeks Discipline]]
