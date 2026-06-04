# Binary Catalyst Entry-Timing EV Framework

## 核心原则
> 对**基本面 intact + upside 无上限**的标的，"等完美价格 vs 现在买入"不是风格选择，是 EV 计算题。Target price 必须从 K 线 + ATR + cluster 推导，不允许直觉填空。

适用场景：48h 内有 binary catalyst（earnings / FDA / FOMC / 重大公告）+ 高 conviction thesis + LEAPS 或正股的多季度持有视角。

---

## 为什么要有这个框架：Process > Outcome

**核心信念**：交易目标是"**纪律执行完美**"，不是"实际价格完美"。

- **实际完美价**：今天 STOCK_Z 最低 $156，LEAPS 最低 $69 — 不可复用 noise，事后才能识别
- **纪律完美价**：用框架在 5/6 算出 target $78 → 这个 target 永远可以用同样方法重新算出，是 reusable 系统

### 双向对称失败模式（必须警惕）

| Trap | 触发情绪 | 短期诱惑 | 长期成本 |
|------|---------|---------|---------|
| **猜底成功** | "再砸一下我就抄底" | 抓到底了爽 | 下次同样赌底踏空 |
| **FOMO 追高成功** | "再不买就来不及" | 抓到拉升爽 | 下次同样追高深套 |

两者都是 EV-negative，只是 packaging 不同。**单笔成功的 dopamine 会 reinforce 错习惯**，下次失败时损失更大。

### Process-perfect 的承诺

- 不保证每笔抓最低/最高价
- **保证 N 笔聚合后接近最优**（D_chase × P_fill 公式把 single-trade variance 折算为期望成本，已经定价）
- 长期 EV 优势 > 任何 single-trade "如果当时..." 的 hindsight magic

### 事后复盘评分（用 process，不用结果）

| Process | 结果 | 评分 |
|---------|------|------|
| 通过 | 好 | A |
| 通过 | 坏 | A（运气，不影响评分）|
| 错 | 好 | **D（最危险，reinforce 错习惯）** |
| 错 | 坏 | C（提取教训）|

**详见**: `feedback_discipline_perfect_vs_actual_perfect.md`（根基级原则）

---

不适用场景：
- 无近端 catalyst 的纯位置建设 → 用 ladder framework（参考 `feedback_greedy_bid_trap.md`）
- Parabolic 末段（已是减仓区不是入场区）
- 短 DTE options（theta 主导，另套数学）

---

## 失败模式：直觉 target + 直觉 P_fill

零售 trader 在"等 vs 买"上最常见的失败：
- Target 凭"我觉得能跌到 $X"
- P_fill 反推用直觉估
- 整个 EV 沦为漂亮算术包装一个 wishful number
- 决策最终回到情绪（"急买怕被套" vs "等待怕踏空"）

**Rigorous 框架的本质**：把 2 个直觉变量（target + P_fill）通过 K 线 + ATR **绑定为 1 个数据驱动变量**，决策从"两个直觉打架"变成"一个客观计算"。

---

## Step 0: Implied Move 计算（市场对 binary 的预期波动定价）

Binary catalyst 入场前必跑。Anchor 选择因 use case 不同而不同。

### Step 0a — Implied move @ 决策时刻（pre-earnings entry decision）

适用：**正在做入场决策的此刻**（未到财报前最后 close）。

```
anchor = 决策时 spot price（不是任何收盘价）
straddle = ATM call mid + ATM put mid（覆盖 earnings 的最近 weekly 期权）
implied_move_pct = straddle / anchor
upper = anchor × (1 + implied_move_pct)
lower = anchor × (1 - implied_move_pct)
```

预测 period: 决策时刻 → 财报反应日 close（含 pre-earnings drift + 事件 reaction）

### Step 0b — Implied move @ pre-event close（post-earnings reaction prediction）

适用：**已到财报前最后 close，准备评估反应区间**。

```
anchor = 财报释放前最后一个 close
  - AMC 财报（4pm ET 释放）→ anchor = 当天 close
  - BMO 财报（开盘前释放）→ anchor = 前一天 close
straddle = ATM call mid + ATM put mid（财报次日 weekly expiry）
implied_move_pct = straddle / anchor
```

预测 period: anchor close → 财报反应日 close（仅事件 move，不含 pre-drift）

### 两 anchor 的差异（STOCK_Z 5/7 AMC 实证）

| 视角 | Anchor | 范围 | Actual ($145) |
|------|--------|------|--------------|
| 5/6 决策时（Step 0a）| $175.90 spot | [$148, $210] | 略破 lower bound (fat tail) |
| 5/7 close（Step 0b）| $157.55 close | [$138, $176] | 区间中段（符合预期）|

**5/6 → 5/7 之间 -11.75% pre-earnings drift 是两 anchor 数值差的来源**。

### 应用规则（4 项硬检查）

#### 1. Stop level stress test
```
your_stop < lower_bound  → 入场 OK（市场没定价穿透你 stop）
your_stop ≥ lower_bound  → 重新评估（市场定价 worst case 就触发你 stop）
your_stop > lower_bound × 1.05 → margin 太薄，size 减半或重选 strike
```

#### 2. Strike selection guard（DITM）
```
DITM strike < lower_bound × 0.95 → 即使 worst case 仍 ITM（安全）
DITM strike ∈ [lower_bound × 0.95, lower_bound] → 接受 worst case borderline 风险
DITM strike > lower_bound → strike 暴露在 worst case 区间，慎选或换 strike
```

#### 3. Size modifier
```
implied move < 8%  → standard size
implied move 8-15% → -25% size
implied move > 15% → -50% size or skip
```

#### 4. Anchor 选择规则
```
做入场决策 → Step 0a (anchor = 决策时 spot)
评估反应区间 / 决策已下挂等待财报 → Step 0b (anchor = pre-event close)
事后复盘 actual 是否 fat tail → Step 0b (用 actual vs Step 0b 区间比)
```

### Anti-pattern

❌ **"等股价跌了再赌财报" 作 entry gate**
- 这是 perfect-price-trap 的财报特化版（参考 `feedback_wait_for_pullback.md`）
- Drop 是 sizing 信息（drop quality diagnostic 后），不是 binary entry filter
- 选择性偏差：跳过"没跌但 thesis 强 + 财报 beat"的有效入场（如 STOCK_Z 5/6 if Q1 had cleanly beat）
- Hindsight 错误：pre-earnings 你不知道"thesis intact"，only confirmed post-fact

❌ **Anchor 混用**：用 Step 0a 算的 range 评判 Step 0b 的 actual move，反之亦然

✅ **Drop quality diagnostic（drop 作 sizing modifier，不是 entry gate）**:
```
财报前 5-7 日 stock 表现:
  - 没跌 / 涨 → standard size
  - 跌 5-10% on 板块 / positioning → standard size
  - 跌 10-20% on 板块 + 个股放量 → +20% size（washed extra）
  - 跌 5%+ on 个股 fundamental concern (downgrade / 客户流失) → -50% size 或 skip
  - 跌 20%+ regardless → 重新 verify thesis 而非仅 sizing 调整
```

### 工具

`scripts/implied_move.py {ticker} [{anchor_close_date}]` — 自动拉 ATM straddle + 计算 bounds + Stop/Strike/Size 检查。

---

## Step 1: 列候选 anchors（按距现价排序）

| Anchor 类型 | 数据来源 |
|------------|---------|
| **5d / 10d / 20d low** | `get_kline.py {ticker} --ktype 1d --num 30` |
| **VWAP**（突破日起算） | K 线 + volume |
| **Volume profile HVN**（高成交密集区） | 同 strike-triangulation 信号 2 方法 |
| **Prior consolidation top/bottom** | K 线结构识别 |
| **Round-number psych**（$100/$150/$200 etc） | 仅作 secondary 加权 |

⚠️ **过滤规则**：post-breakout 标的不取 breakout 之前的低点（STOCK_Z 4/22-4/28 的 $132-149 在 5/1 breakout 之后**不再是 reachable 锚点**，是历史区，必须排除）。

---

## Step 2: ATR 距离 → P_fill 概率表

```python
# 拉 ATR(20)
atr20 = sum(true_range[-20:]) / 20  # true_range = max(H-L, |H-prev_C|, |L-prev_C|)
distance_atr = (current_price - target_price) / atr20
```

| Distance (in ATR units) | P_fill (1-day window) |
|------------------------|----------------------|
| < 0.5× | 60-75%（近距 + 触发频繁） |
| 0.5-1.0× | 40-55% |
| 1.0-1.5× | 25-40% |
| 1.5-2.0× | 10-25% |
| > 2.0× | <10%（不可达，**不该选这个 target**） |

**N-day window 推广**：
```
P_fill_N ≈ 1 - (1 - P_fill_1)^N
```
（粗近似，假设每日独立；trending 名字偏低，mean-reverting 偏高）

---

## Step 3: 选最强 cluster within reachable zone

- **"最强"** = 多锚点收敛 / 多次触发 / 高 volume
- **2+ anchors 落在 $1-2 内** → 那个 cluster 就是 target，P_fill 上调
- **没有 cluster only single anchor** → 把那个单 anchor 当 target，但 P_fill 适当下调（structural confidence 不足）
- **超过 1.5× ATR 没有 anchor cluster** → 框架告诉你"该 target 不存在 reachable 路径，buy now 自动获胜"

---

## Step 4: Underlying target → Option target

### Linear approximation（DITM Δ stable 时足够）
```
target_option = current_option - (current_underlying - target_underlying) × Δ
```

### Option chain query（更准）
- Pre-catalyst（期间）：IV 通常 stable 或缓升，theta 1-2 天可忽略 → linear approx 足够
- Post-catalyst（期间）：IV crush 大，**必须**用 option chain 重算（输入 target underlying scenario + IV change 假设）

### 适用 Δ 区间
| Strike 类型 | Δ 区间 | Linear approx 误差 |
|------------|-------|-------------------|
| Deep ITM (Δ ≥ 0.75) | 0.75-0.95 | <5%，linear OK |
| ATM (Δ 0.45-0.55) | 0.45-0.55 | 5-15%，建议用 chain |
| OTM (Δ ≤ 0.30) | 0.10-0.30 | 15%+，必须用 chain |

---

## Step 5: 套 EV 公式

### 4 个输入

| 变量 | 定义 | 来源 |
|------|------|------|
| **P_fill** | 目标限价被打到的概率 | Step 2 计算 |
| **D_over** | 现价 - 目标价（确定的"被套"成本） | $/share × 100 (option) 或 × qty (stock) |
| **D_chase** | 踏空后被迫追的估价 - 目标价 | catalyst 期望 gap × Δ × 衰减系数 |
| **P_right** | Thesis 概率 | **不进决策公式**（两边都乘 P_right 抵消）；仅做 sanity gate — 若 < 50% 根本不该入场 |

### 公式
```
EV(等)    = (1 - P_fill) × D_chase    [踏空时被迫追的期望成本]
EV(现在买) = D_over                    [vs 假想低价的确定 overpay]

决策：取 EV 较小者
等价表述：D_over < (1 - P_fill) × D_chase → 现在买
```

---

## Worked Example: STOCK_Z 270115 C140 LEAPS, 2026-05-06

### Setup
STOCK_Z 5/7 AMC earnings binary，决策"现在买 vs 等限价"。
- Current LEAPS: $84.98 (mid $87.30)
- Current STOCK_Z: $178.54
- Δ 0.78
- ATR(20): $17.57

### Step 1-2: 候选 anchors

| Anchor | Underlying | 距离 | ATR 倍数 | P_fill (1-day) | 备注 |
|--------|-----------|-----|---------|----------------|------|
| 5/5 intraday low | $174.20 | $4.34 | 0.25× | ~65% | 浅触底，breakout retest |
| 5/6 intraday low | $169.24 | $9.30 | 0.53× | ~50% | 当日触底 |
| Pre-breakout top | ~$164 | $14.54 | 0.83× | ~35% | 5/1 breakout 起点 |
| 4/30 low | $149.73 | $28.81 | 1.64× | <20% | **breakout 之前，过滤掉** |
| 10d low (4/28) | $132.63 | $45.91 | 2.61× | <10% | **历史区，过滤掉** |

### Step 3: 选 cluster
$169-174 区间是 5/5 + 5/6 双触底 + breakout retest = 强 cluster。
**Target underlying = $170**（cluster 中点，保守取低端）

### Step 4: 映射 LEAPS
```
target_LEAPS = $84.98 - ($178.54 - $170) × 0.78
            = $84.98 - $6.66
            ≈ $78.32

→ rigorous target ≈ $78（不是 $80）
P_fill ≈ 55%（cluster 强 + 0.49× ATR）
```

### Step 5: EV 计算
```
D_over = $84.98 - $78 = $6.98 = $698 / 张
D_chase = STOCK_Z gap to $200 (beat scenario) × Δ 0.78 × 0.9 衰减
        = ($200 - $170) × 0.78 × 0.9
        ≈ $21 / share
        = $2,100 / 张 (worst case beat)
   实际期望（混合 beat / miss / flat）= $1,500 (60% × $2,100 + 40% × $0)
   
EV(等)    = (1 - 0.55) × $1,500 = $675
EV(现在买) = $698

$675 vs $698 → marginal，等略占优 ($23 边际)
```

### 事后 ground truth
5/7 盘前 sell-the-news：STOCK_Z 跌至 $161.31，LEAPS bid $73.60。
- 用 rigorous target $78 限价 → **实际会被 fill**
- Rigorous EV 算出"等略占优"事后被验证

### 对比 ad-hoc 直觉版本
原版（用户 5/6 intuition）：target $80, P_fill 25%（凭感觉估）
- EV(等) = 75% × $1,500 = $1,125
- EV(买) = $500
- 显示 buy now 2.25× 优势 → 决策方向相反

**结论变化**：直觉 target 太低（$80 ≈ STOCK_Z $158，超 1.6× ATR）导致 P_fill 估太低，扭曲决策。Rigorous 数据驱动 target 是 $78（STOCK_Z $170 cluster），与 wait 决策的 ground truth 一致。

---

## 关键 mental model: Asymmetric Pain

为什么这框架在 binary catalyst 前**系统性地偏向"现在买"或"接近 cluster 等"**：

```
Forward downside ≈ ±5-10% pullback     (bounded by stop)
Forward upside   ≈ ±20-50% on catalyst (unbounded by structure)

→ 非对称 → 必须付费换参与权
```

- **被套 pain 有底**：stop_level 决定下行 floor
- **踏空 pain 无顶**：winner 可能跑 30-50%，全部 forfeit

**Buy now 的本质**：用 D_over 这个 known cost 买掉 踏空 unknown cost 的 tail risk。

---

## 决策树（30 秒速决）

```
Q1: 48h 内有 binary catalyst？
├── No → 走 Ladder Framework（不在本框架范围）
└── Yes → ↓
    Q2: 走完 Step 1-3 推导 target
        Step 1: 列 anchors
        Step 2: 算 ATR 距离 → P_fill 表
        Step 3: 选最强 cluster within reachable zone
        ↓
    Q3: P_fill > 50% AND target 是 multi-anchor cluster？
    ├── Yes → 限价等待（target 在合理 reachable）
    └── No → ↓
        Q4: D_over < (1 - P_fill) × D_chase？
        ├── Yes → 现在买（买参与权）
        └── No  → 等待 / 重设更近 target
```

---

## Anti-patterns（违反即 flag）

1. **Target 凭直觉**："我觉得能跌到 $80" → 必须从 K 线 cluster 推导
2. **改低限价**："$85 → $80" = perfect-price trap (Rule #14)
3. **单档深跌挂单**：parabolic 标的只挂一档钉在 yesterday low 之下 = greedy-bid trap
4. **混淆"省钱"和"好交易"**：省 $500 的同时让出 $1,500 期望涨幅 = 数学上的 loss
5. **跳过 ATR 距离检查**：不验证 target 是否在 reachable zone (≤ 1.5× ATR)

---

## Cross-references

- `trade/us_stock/strategy/strike-triangulation.md` — 姊妹方法论（解决 strike 选择，本文件解决 entry timing）
- `feedback_wait_for_pullback.md` — 等回调 trap 的定性描述（本框架是其量化补充）
- `feedback_greedy_bid_trap.md` — 单档深跌挂单 anti-pattern
- `feedback_iv_adjacent_pairing.md` — 高 IV 名字的策略选择（与本框架并行考虑）
- `.claude/rules/trading-discipline.md` — Rule #14 perfect-price trap（本框架 Q4 的等价表述）

---

## 源案例

**2026-05-06 STOCK_Z Jan 2027 $140 LEAPS**：
- 原 $80 限价 SUBMITTED 11:42 ET，12:03 ET 改 $85 → 成交 $84.98
- 5/8 post-hoc EV 验证：rigorous target $78 (STOCK_Z $170 cluster) 比直觉 $80 更准确
- 5/7 ground truth：LEAPS bid $73.60 = $78 限价**实际会被 fill**
- 教训：target 直觉化导致 P_fill 误判，决策方向被扭曲；数据驱动后框架自洽

**框架 v2 诞生于 2026-05-08**，迭代过程：v1 接受直觉 target → 用户 指出"目标价需用 K 线计算" → v2 加入 4 步数据驱动 target derivation。
