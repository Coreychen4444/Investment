# LEAPS Call 操作手册

## 核心原则
> LEAPS = 用资本效率换 vega 暴露。它不是"更长的 long call"，是另一种 Greeks profile 的工具。

LEAPS（Long-term Equity Anticipation Securities）= **DTE > 365 天的 long call/put**。本文件聚焦 long LEAPS call。

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
| IV percentile | < 50% | 等 IV 回归再入 |
| Bid-ask spread | < 8% of mid | 换个 strike 或 expiry |
| Open Interest | > 50 contracts | 流动性不足，弃 |
| Volume (近 5 日均) | > 5/day | 流动性不足，弃 |
| Underlying thesis | ≥ 12 月可信 | 短期 long call 替代 |
| Position sizing | 单笔 ≤ 账户 5% | 减 size 或弃 |

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
2. **IV 时机**：当前 IV percentile < 50% 吗？（> 60 不要入）
3. **资本机会成本**：花这笔 premium 而不买正股，因为什么？（资本受限 / 分散需求 / 杠杆需求）
4. **流动性**：bid-ask < 8%、OI > 50、daily volume > 5？
5. **退出预案**：DTE 90 决策点你计划怎么走？write 下来

### 评分卡

| 项目 | 2分 | 1分 | 0分 |
|------|-----|-----|-----|
| Thesis 持续度 | 强（多季度催化） | 一般 | 短期 catalyst（用短期 call 不是 LEAPS） |
| IV 时机 | percentile < 30 | 30-50 | > 50（贵的保险） |
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

**具体配置建议**（首次）：
- 标的：base 仓 thesis 最强（你现状下推荐 STOCK_Y 或 STOCK_F — 多年 cycle）
- Strike：现价 -15% 到 -20%（DITM Δ 0.80-0.85）
- Expiry：Jan +2 年
- Size：账户 4-6%（$1,000-$1,500）

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

## 与期权框架的关系

本文件为 LEAPS Call 单一策略的操作手册。完整期权策略体系（决策树、其他策略）详见 `options-strategy-framework.md`。Greeks 操作规则（G-01/02/03）详见 `greeks-discipline.md`。

**框架定位**：LEAPS 是 Long Call 的 DTE 365+ 变种，不是独立的 Tier 1 策略，但因 Greeks profile 完全不同 + 适用场景明确，单独建文件管理。

---

## 源案例（待填）

- 待第一笔 LEAPS 实际开仓后，按 `options-journal-template.md` 格式回填
- 参考案例（非 LEAPS 但邻近）：STOCK_M 8/21 C150 (DTE 116, Δ ~0.65) — Thesis Expression 准 LEAPS 用法
