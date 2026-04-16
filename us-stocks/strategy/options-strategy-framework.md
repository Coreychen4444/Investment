# 期权交易框架 v1

## 一、核心理念

> 期权是正股纪律的延伸工具，不是独立的赌场。

三条铁律：
1. **正股规则优先** — Zone、分类（Confirmed/Probe/Early）、Max 2 entries 对期权同样适用
2. **买方付权利金买权利，卖方收权利金扛义务** — 清楚自己站哪边
3. **策略复杂度匹配认知** — 不懂的策略不做，先用 Tier 1 建立肌肉记忆

---

## 二、Greeks 快速参考

不用算，理解对你的钱意味着什么：

| Greek | 含义 | 买方影响 | 卖方影响 |
|-------|------|---------|---------|
| **Delta** | 股价变 $1，期权变多少 | delta 0.3 = 股涨$1你赚$0.30 | 反向 |
| **Theta** | 每天时间价值损耗 | 敌人（每天亏） | 朋友（每天赚） |
| **Vega** | IV 变 1%，期权变多少 | 朋友（IV涨你赚） | 敌人（IV涨你亏） |
| **Gamma** | Delta 变化速度 | 临近到期 ATM 波动剧烈 | 临近到期 ATM 风险最大 |

关键认知：
- **Earnings 前**：IV 膨胀 → 买方的期权变贵（vega 推高），卖方承担更多风险
- **Earnings 后**：IV crush → 期权价格骤降，无论涨跌。买方被 crush 杀，卖方从 crush 赚
- **临近到期**：theta 加速衰减，OTM 期权归零速度加快

---

## 三、策略分级与详细场景

### Tier 1：核心策略（必须掌握，日常使用）

---

### 1. Long Call（买入看涨期权）

- **结构**：买入 call option
- **你花**：权利金（一次性，即最大亏损）
- **盈利条件**：股价 > strike + 权利金
- **最大盈利**：无限
- **最大亏损**：权利金

#### 适用场景
- 看涨某股票但不想全仓买入正股，用小资金参与
- 有明确催化剂 + 时间窗口（earnings、产品发布、政策落地）
- 两段式建仓中用 long call 代替正股 probe，错了只亏权利金
- 用小资金博取高倍回报（杠杆效应）

#### 对冲场景
- **对冲空仓/做空风险**：做空正股怕逼空，买 call 封顶最大亏损
- **对冲 trim 后踏空**：减仓后怕继续涨，买 call 保留上行参与权
- **对冲 covered call 的极端上行损失**：卖了 covered call 怕暴涨，再买更高 strike call 封顶踏空
- **对冲 sell put 未成交的踏空**：挂了 sell put 等接货但标的直接涨走 → 买 call 确保参与

**核心对冲逻辑：Long Call 对冲的是"错过上涨"的风险**

#### 不适合
- 没有催化剂的慢涨票（theta 每天吃你）
- IV 已经很高时买入（你在买贵的保险）
- 想收租/收权利金（买方是付钱的一方）

#### Strike 选择
- ATM（平值）：delta ~0.5，参与度高但贵
- OTM（虚值）：便宜但需要涨更多才赚，适合 probe
- 推荐：no_chase_above 附近或 trim zone 下沿作为 strike

#### Expiry 选择
- 催化剂日期 + 2-4 周缓冲（给反应时间）
- 最短不低于 2 周（太短 theta 杀太快）

#### 实战参考
STOCK_X $11C 5/1 @ $0.21 — 催化剂 earnings 临近，probe size

---

### 2. Long Put（买入看跌期权）

- **结构**：买入 put option
- **你花**：权利金
- **盈利条件**：股价 < strike - 权利金
- **最大盈利**：strike × 100 - 权利金（股票跌到 0）
- **最大亏损**：权利金

#### 适用场景
- **持仓保险（Protective Put）**：浮盈大 + 不想卖股票但怕回撤，锁住利润
- **事件前对冲**：earnings / FOMC 前不确定但想保持仓位
- **看空下注**：判断会跌但不想做空正股（做空有无限风险，long put 亏损固定）
- **替代止损单**：止损会在 flash crash 被扫，put 给你固定价格的卖出权
- **组合级别对冲**：买 QQQ/SPY put 保护整个组合 beta 暴露

#### 对冲场景
- **对冲持仓下行（protective put）**：浮盈大 + 事件临近 → 不想全卖但锁底
- **对冲事件风险**：earnings/FOMC 前不确定 → 买 put 保护，不用 trim 正股
- **对冲组合 beta 暴露**：整个组合偏多 → 买 QQQ/SOXX/SPY put 保护全组合
- **对冲 sell put 的 tail risk**：单独买更低 strike put → 实际上就变成了 bull put spread
- **对冲 covered call 的下行**：卖了 covered call 只收了小权利金 → 额外买 put 保护崩盘
- **event-risk-reduction 替代方案**：不减仓位但通过 put 降低 beta

**核心对冲逻辑：Long Put 对冲的是"已有仓位下跌"的风险**

#### 不适合
- 持仓太小不值得对冲（如持仓少于 100 股时买 1 张 put 覆盖 100 股 = 过度对冲）
- 股票已经暴跌后买（IV 暴涨，put 极贵）
- 长期持有（theta 持续消耗）

#### Strike 选择
- 保护性 put：accumulation zone1 上沿（保护浮盈的关键支撑）
- 方向性看空：当前价附近或略 OTM

---

### 3. Bull Call Spread（牛市看涨价差）

- **结构**：买低 strike call + 卖高 strike call（同 expiry）
- **你花**：净成本 = 买价 - 卖价
- **盈利条件**：股价 > 低 strike + 净成本
- **最大盈利**：两 strike 差 - 净成本
- **最大亏损**：净成本

#### 适用场景
- 看好方向 + 有明确上涨 target（高 strike = 你的 target）
- 想用杠杆但控制成本（比裸买 call 便宜 40-60%）
- IV 偏高时的看涨策略（卖的腿抵消部分 vega，IV 下降伤害更小）
- Earnings 方向性下注，成本低于裸买 call
- 对标的有价格上限判断（卖 call strike = 你认为的天花板）

#### 对冲场景
- **对冲 trim 后踏空（比 long call 便宜）**：减仓后想保留参与但不想花太多
- **对冲 long put 的方向判断错误**：买了 protective put 又怕其实会涨 → 加 bull call spread 对冲 put 成本
- **对冲 IV 高时买方的 vega 风险**：IV 高买裸 call 太贵 → spread 卖出腿抵消部分 vega

**核心对冲逻辑：Bull Call Spread 对冲的是"看涨但成本/IV 太高"的风险**

#### 不适合
- 看好但没有上限预期（纯趋势跟踪用 long call 更好）
- 标的期权流动性差、bid-ask 宽（两腿都吃 spread 成本）

#### Strike 选择
- 买入腿：accumulation zone 上沿或当前价附近
- 卖出腿：trim zone 上沿或 target price
- 宽度越大，成本越高但最大利润越大

#### 实战参考
STOCK_Y $440/$500 spread @ $18.40 — 风险 $1,840 / 最大回报 $4,160，R:R 1:2.26

---

### 4. Bull Put Spread（牛市看跌价差 / 卖 put spread）

- **结构**：卖高 strike put + 买低 strike put（同 expiry）
- **你收**：净权利金
- **盈利条件**：股价 > 高 strike（两腿都过期）
- **最大盈利**：净权利金
- **最大亏损**：两 strike 差 - 净权利金

#### 适用场景
- **替代裸卖 put 的默认选项** — 下行有保护，不怕 black swan
- 标的在 accumulation zone，愿意接货但想限制灾难损失
- IV 高时卖权利金吃 IV crush
- 看涨但不想花钱（credit strategy，开仓收钱）
- 降低保证金占用（比裸卖 put 少 50-70%）

#### 对冲场景
- **对冲裸卖 put 的 tail risk**：买入保护腿把 naked put → spread，封顶亏损
- **对冲"想建仓但怕买错"的成本**：正股建仓要全额资金，spread 先收权利金试探
- **对冲持仓不足的 theta 损耗**：等待建仓期间账户没有收益 → sell put spread 边等边收租

**核心对冲逻辑：Bull Put Spread 对冲的是"裸卖 put 的灾难风险"和"等待建仓期间的零收益"**

**规则：所有 sell put 都应默认用 spread 形式，除非你 100% 确定想接货且仓位合理。**

#### 不适合
- 真心想接货 100 股建仓（被 assign 后操作更复杂）
- 权利金太薄覆盖不了手续费

#### Strike 选择
- 卖出腿：accumulation zone1 上沿
- 买入腿（保护）：accumulation zone2 下沿或 invalidation 价格
- spread 宽度 = 你能接受的最大亏损 + 权利金

#### vs 裸卖 put 对比
| | 裸 Sell Put | Bull Put Spread |
|--|------------|----------------|
| 收入 | 高 | 低（一部分买了保护） |
| 最大亏损 | strike × 100 | spread 宽度 - 权利金 |
| 保证金 | 高 | 低 |
| 稀释/black swan | 可能炸账户 | 亏损固定 |
| **结论** | 只在 100% 想接货时用 | **默认应该用这个** |

---

### 5. Covered Call（备兑看涨期权）

- **结构**：持有 ≥100 股 + 卖 OTM call
- **你收**：权利金
- **盈利条件**：任何情况都收权利金；被叫走时赚 strike - 成本 + 权利金
- **最大盈利**：(strike - 成本 + 权利金) × 100
- **最大亏损**：股票跌到 0（和纯持股一样，权利金略微缓冲）
- **前提**：必须持有 ≥100 股整数

#### 适用场景
- 持仓在 trim zone，本来就愿意在高位卖出 → 卖 call = 自动 trim + 收租
- 震荡市增强持仓收益，横盘时正股不赚但权利金补收益
- 降低持仓成本（每月卖 call 收租 → 累积降低 avg cost）
- Earnings 后 IV crush 收割期（IV 暴跌后卖 call 的 theta decay 最快）

#### 对冲场景
- **对冲持仓的小幅下跌**：权利金提供下行缓冲
- **对冲震荡市的持仓成本**：横盘不赚钱 → 卖 call 收租补偿时间成本
- **对冲"该 trim 但不舍得卖"的心理**：设 trim zone 价格卖 call → 被叫走 = 强制纪律 trim
- **对冲 long put 的成本**：持仓 + 买 put + 卖 call = Collar，用 call 权利金覆盖 put 成本

**核心对冲逻辑：Covered Call 对冲的是"持仓不动的时间成本"和"不舍得 trim 的心理成本"**

#### 不适合
- 强烈看涨时（涨过 strike 利润被封顶，你会后悔）
- 持仓不满 100 股（不能 covered，变成 naked call = 无限风险）
- 股票被叫走你会心疼

#### Strike 选择
- trim zone 价格作为 strike（被叫走 = 在你本来想 trim 的价格卖出）
- no_chase_above 以上

---

### Tier 2：进阶策略（基础策略熟练后使用）

---

### 6. Protective Put（保护性看跌 — Long Put 的持仓对冲专用）

持有正股 + 买 put。和 Long Put 策略相同，但目的是对冲已有持仓而不是方向性做空。

#### 适用场景
- 浮盈大 + 事件临近 → 不想全卖但要锁底
- event-risk-reduction 原则的替代方案（不减仓位但降 beta）
- 比 trim 更灵活：事件过后 put 作废，仓位还在

#### 对冲场景
- **对冲单一持仓的黑天鹅**：标的有 tail risk（稀释、监管、做空报告）
- **对冲 event-risk 但不想减仓**：想保住仓位过 earnings
- **对冲 sell put 接货后的进一步下跌**：被 assign 了怕继续跌 → 立刻买 put 保护
- **替代止损单**：止损会被 flash crash 扫掉 → 买 put 锁定最低卖出价

**核心对冲逻辑：Protective Put 对冲的是"已有多头仓位的极端下行"**

#### Strike 选择
accumulation zone1 上沿（锁住 zone 以上的利润）

---

### 7. Wheel Strategy（轮转策略）

不是单一策略，是一个循环框架：Sell Put → 接货 → Sell Covered Call → 被叫走 → 回到 Sell Put

```
Phase 1: Sell Put（strike = accumulation zone）
   ├─ 不被 assign → 赚权利金 → 继续 Sell Put
   └─ 被 assign → 接货 100 股 → 进入 Phase 2

Phase 2: Sell Covered Call（strike = trim zone）
   ├─ 不被 call away → 赚权利金 → 继续卖 call
   └─ 被 call away → 赚价差 + 双重权利金 → 回到 Phase 1
```

#### 与 Zone 体系的映射
| Wheel 阶段 | 对应 Zone 框架 |
|-----------|------------|
| Sell put strike | accumulation zone |
| 接货 | 在 zone 内建仓 |
| Sell call strike | trim zone |
| 被叫走 | 自动 trim |

#### 对冲场景
- **对冲"想买但怕买贵"**：Sell put 在 zone 内等接货，涨了赚权利金也不亏
- **对冲"持仓震荡零收益"**：接货后卖 covered call，不涨也有权利金
- **对冲"trim 后完全离场"的踏空**：被 call away 后立刻进入 sell put phase，保留条件性参与
- **对冲正股交易的情绪消耗**：机械化循环 → 不用每次纠结买/卖时机

**核心对冲逻辑：Wheel 对冲的是"在 zone 之间反复交易的决策成本和踏空/套牢焦虑"**

#### 前提
标的必须是你长期愿意持有的（core thesis），不适合纯投机票。需要 100 股整数。

---

### 8. Covered Strangle（备兑宽跨 — Wheel 的并行版）

**结构**：持有 100 股 + 卖 OTM call（trim zone）+ 卖 OTM put（accumulation zone）

同时两边收租。被叫走 = trim；被 assign = 加仓。

#### 对冲场景
- **对冲 covered call 单边收租不够**：加卖 put 端双倍权利金
- **对冲"在 zone 内等建仓"的资金闲置**：卖 put 端等待接货期间也有收益
- **对冲单方向 covered call 的不对称**：两端都收租 → 不管涨跌都在赚时间价值

**核心对冲逻辑：Covered Strangle 对冲的是"单腿收租效率不够"**

#### vs Wheel 区别
| | Wheel | Covered Strangle |
|--|-------|-----------------|
| 节奏 | 交替 | 同时 |
| 权利金 | 一次一笔 | 双倍 |
| 风险 | 一端暴露 | 两端暴露 |
| 适合 | 入门 | Wheel 熟练后升级 |

---

### 9. Iron Condor（铁鹰 — 了解即可）

**结构**：Bull Put Spread + Bear Call Spread（同 expiry）

四条腿：买低 put + 卖中低 put + 卖中高 call + 买高 call

#### 对冲场景
- **对冲 short strangle 的无限风险**：想两端收租但不能承受无限亏损 → 加保护翅膀
- **对冲 IV 膨胀后的均值回归**：Earnings/事件前 IV 泡沫 → 卖 condor 赚 IV crush
- **对冲震荡市的零收益**：确信标的短期在区间内 → 两端收租

**核心对冲逻辑：Iron Condor 对冲的是"short strangle 的无限风险"和"震荡市无方向收益"**

**目前不推荐常用**：你是趋势/催化剂交易者，不是区间交易者。了解原理，遇到明确震荡市再考虑。

---

### Tier 3：高风险策略（了解原理，当前阶段不使用）

| 策略 | 结构 | 不用的原因 |
|------|------|----------|
| **Short Strangle** | 裸卖 OTM call + OTM put | 两端无限风险，需大账户 + Level 4/5 权限 |
| **Short Guts** | 裸卖 ITM call + ITM put | 和 strangle 等价但流动性更差、早期行权风险高 |
| **Naked Call** | 裸卖 call | 无限亏损，散户通常不批权限 |
| **Ratio Spread** | 不等量买卖 | 有裸腿风险，复杂度高 |
| **Butterfly** | 三腿 | 精准 pin 价格概率低，性价比差 |

---

## 四、每个策略的对冲核心总表

| 策略 | 对冲核心风险 |
|------|------------|
| **Long Call** | 错过上涨 / 空仓逼空 / trim 后踏空 |
| **Long Put** | 持仓下跌 / 事件风险 / 组合 beta |
| **Bull Call Spread** | 看涨但 IV 太高 / trim 后踏空（经济版） |
| **Bull Put Spread** | 裸卖 put 的 tail risk / 等待建仓的零收益 |
| **Covered Call** | 持仓时间成本 / 不舍得 trim 的心理 |
| **Protective Put** | 持仓极端下行 / 黑天鹅 / 替代止损 |
| **Wheel** | Zone 间反复交易的决策成本和情绪消耗 |
| **Covered Strangle** | 单腿收租效率不够 |
| **Iron Condor** | Short strangle 无限风险 / 震荡市零收益 |

---

## 五、决策树

```
交易意图
  │
  ├─ 看好，想建仓/加仓
  │    ├─ 标的在 accumulation zone
  │    │    ├─ 愿意接货 100 股 → Sell Put (cash-secured)
  │    │    └─ 想收租但控风险 → Bull Put Spread（默认选项）
  │    ├─ 有明确催化剂 + 时间窗口
  │    │    ├─ 高确信 → Bull Call Spread（成本可控，R:R 好）
  │    │    └─ 低确信 / probe → Long Call（权利金 = 最大亏损）
  │    └─ 超过 no_chase_above → 不做
  │
  ├─ 已有持仓，想增强/保护
  │    ├─ 持仓 ≥ 100 股 + 在 trim zone
  │    │    ├─ 愿意在 trim 价被叫走 → Covered Call
  │    │    └─ 想锁利润 + 零成本 → Collar（买 put + 卖 call）
  │    ├─ 持仓 ≥ 100 股 + 想双向收租 → Covered Strangle / Wheel
  │    ├─ 浮盈大 + 事件临近 → Protective Put
  │    └─ 组合 beta 过高 → 买 QQQ/SPY put 对冲
  │
  ├─ 看空 / 想对冲下行
  │    ├─ 对冲已有持仓 → Protective Put / Bear Put Spread
  │    ├─ 方向性做空 → Long Put（亏损固定）或 Bear Put Spread
  │    └─ 不确定但想买保险 → Long Put（小仓位）
  │
  └─ 事件博弈 (Earnings/FOMC/CPI)
       ├─ 有方向观点 → Bull/Bear Call/Put Spread
       ├─ 确定会大动但不知方向 → Long Straddle/Strangle（小仓）
       └─ 不确定 → 不做
```

---

## 六、Strike 与 Expiry 选择规则

### Strike 选择 — 映射 Zone 体系

| 策略 | Strike 选择 | Zone 映射 |
|------|-----------|----------|
| Sell Put / Bull Put Spread 卖出腿 | accumulation zone1 上沿 | zone1 |
| Sell Put / Bull Put Spread 保护腿 | accumulation zone2 下沿或 invalidation | zone2 / invalid_if |
| Covered Call | trim zone 内 | trim_zone |
| Long Call（催化剂 play） | no_chase_above 附近 | no_chase |
| Bull Call Spread 卖出腿 | trim zone 上沿或 target | trim_zone |
| Protective Put | accumulation zone1 上沿 | zone1 |

### Expiry 选择

| 场景 | 推荐 Expiry | 原因 |
|------|-----------|------|
| Earnings play | 事件日 + 2-3 周 | 给反应时间，覆盖 post-earnings drift |
| 收权利金（sell put/call） | 14-30 天 | theta 衰减最优区间 |
| 保护性 put | 覆盖事件窗口即可 | 不需要太长，保险费按天算 |
| 催化剂 long call | 催化剂 + 3-4 周缓冲 | 防止 timing 略偏 |
| Wheel / 持续收租 | 30-45 天滚动 | 每月到期后 roll 到下月 |

### 事件窗口内的 Expiry 特殊规则
- Earnings / FOMC / CPI 在窗口内 → 卖方策略 expiry 选在事件**之前**（避免事件 gap）
- 或者 expiry 跨过事件但用 **spread 限制损失**
- Long call/put 催化剂 play → expiry 必须**跨过**事件

---

## 七、仓位管理规则

### 单笔上限

| 策略类型 | 单笔风险上限 | 说明 |
|---------|-----------|------|
| Long call/put（probe） | 账户 1-2% | 权利金 = 最大亏损 |
| Long call/put（confirmed） | 账户 2-3% | |
| Bull/Bear spread | Max loss ≤ 账户 3-5% | spread 宽度 - 权利金 |
| Sell put (cash-secured) | 接货金额 ≤ 正股愿加仓位 | 被 assign = 实际建仓 |
| Covered call | 已持有的 100 股 | 不额外占资金 |

### 与正股合并计算
- **同一 thesis 的正股 + 期权合计最多 2 笔 entry**（Iron Rule #2）
- 卖 put = 条件性 entry，算 1 笔
- Long call 不算 entry（不增加持仓），但算 exposure
- Spread 的 max loss 计入该 ticker 的总风险暴露

### 组合级别限制
- 期权总风险（所有 max loss 之和）≤ 账户 10-15%
- 同一 ticker 期权 + 正股总暴露 ≤ 账户 25%
- 卖方总暴露（所有 short option 的潜在义务）需要保证金覆盖

---

## 八、期权 Pre-Trade Checklist

每笔期权交易前过一遍（与 `pre-trade-checklist.md` 配合使用）：

### 通用检查（所有期权策略）
- [ ] **分类**：Confirmed / Probe / Early-risky？
- [ ] **策略匹配**：这个意图用什么策略最合适？（查决策树）
- [ ] **Max loss 可接受吗？** 这笔全亏你不影响心态？
- [ ] **Expiry 覆盖催化剂吗？** 有足够时间窗口？
- [ ] **IV 水平**：当前 IV 偏高还是偏低？买方要低 IV，卖方要高 IV
- [ ] **流动性**：bid-ask spread < 10%？OI > 100？太差不做
- [ ] **事件检查**：窗口内有 earnings / FOMC / CPI 吗？
- [ ] **与正股 entry 合并计算**：算上这笔，同一 thesis 几笔了？

### 卖方额外检查（Sell Put / Covered Call / Spread 卖出端）
- [ ] **接货/被叫走你接受吗？** 被 assign 不难受？
- [ ] **Strike 在 zone 内吗？** Sell put → accumulation zone；Covered call → trim zone
- [ ] **有保护腿吗？** 默认用 spread，裸卖需要额外理由
- [ ] **会抵消防守动作吗？** 刚 trim/止损完就卖 put = undoing defense

### Sell Put 专项 5 问评分

详见 `sell-put-rules.md`。

| 项目 | 2分 | 1分 | 0分 |
|------|-----|-----|-----|
| Intent | 真想接货 | 一半接货一半收租 | 纯赚权利金 |
| Strike | 很舒服 | 勉强可接受 | 会后悔 |
| Expiry | 14-30天 | 7-14天 | 7天内 |
| Event risk | 平静期 | 一般 | 财报/FOMC/密集事件 |
| 抵消防守？ | 不抵消 | 部分抵消 | 完全抵消 |

- 7-8：健康，执行
- 5-6：可做，偏激进
- ≤4：赌博，不做

---

## 九、常见错误与行为检测

| 错误模式 | 信号 | 对策 |
|---------|------|------|
| 拿期权当彩票 | 买极 OTM 短期 call "博一把" | 问自己：这笔是 thesis 还是赌博？ |
| 卖保险给地震带 | 在高 beta + tail risk 的票上裸卖 put | 默认用 spread；确认 tail risk 可控才裸卖 |
| Earnings 前买贵期权 | IV 膨胀时买入 long call/put | 用 spread 对冲 vega；或等 IV crush 后再买 |
| 被 assign 后恐慌 | 卖 put 被 assign 不在计划内 | 卖之前就想好：被 assign = 在 zone 内建仓 |
| 期权当正股补仓 | 正股亏了想用 long call "翻本" | 这是 revenge trading，禁止 |
| 不计 max loss 就开仓 | "spread 只花 $200 嘛" | $200 是成本，max loss 可能是 $500。算清楚 |
| 到期前不管 | 忘记期权要到期 | 到期前 3-5 天必须决策：平仓 / roll / 让到期 |

---

## 十、策略进阶路线图

```
阶段 1（当前）: Long Call + Bull Call Spread + Bull Put Spread + Covered Call
   → 用 100 股整数仓练 Covered Call / Wheel
   → 所有 sell put 改用 spread
   → 做 2-3 个月，每笔记录在 trade journal

阶段 2（3个月后）: + Protective Put + Collar + Wheel 循环
   → 仓位到 100 股整数的票开始跑 Wheel
   → 事件前用 protective put 替代部分 trim

阶段 3（6个月后）: + Covered Strangle + Calendar Spread
   → Wheel 熟练后升级 Covered Strangle
   → 开始研究 IV term structure 和 calendar spread

不碰: Short Strangle / Naked Call / Ratio Spread / Short Guts
   → 除非账户规模翻 3 倍以上且有 Level 4+ 权限
```

---

## 相关文件

- `trading-rules.md` — 核心交易规则（master）
- `pre-trade-checklist.md` — 交易前完整检查清单（期权 checklist 配合使用）
- `sell-put-rules.md` — 卖 put 纪律（已纳入本框架 5 问评分）
- `two-stage-entry-rules.md` — 两段式建仓（期权中 Long Call = 第一段 probe）
- `event-risk-reduction-principle.md` — 事件前减仓（Protective Put 为替代方案）
- `position_zones.json` — Zone 数据（Strike 选择的直接参考）
