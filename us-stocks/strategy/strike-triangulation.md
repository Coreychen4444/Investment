# Strike Triangulation 方法论

## 核心原则
> Strike 决策不是单点 input，是 4 个独立信号的收敛。Zone 是 baseline 不是终点。

适用场景：sell put / sell call strike 选择、bull put spread / bear call spread 双 leg 构造、long call/put strike、Zone1/Zone2/Trim/Invalid 阈值锚定。

---

## 四个独立信号

### 信号 1: Zone 框架 (baseline)
来源：`state/positions.json` 的 `tickers["<T>"].agent.zones`（active）或 `state/watchlist.json` 同位置（候选）
- 当前 zone1 / zone2 / trim_zone / no_chase / invalid_if 的位置
- 提供 strike 的"应该在哪个区间"的初始判断

### 信号 2: Volume Profile (近 25 日)
来源：`get_kline.py US.{ticker} --ktype 1d --num 30`
方法：
```python
# 按 10-dollar (或 10% of price for high-priced names) bucket 累积成交量
buckets = {}
for day in last_25_days:
    avg_price = (open + high + low + close) / 4
    bucket = int(avg_price // 10) * 10
    buckets[bucket] += volume
```
解读：
- 单 bucket > 60M = 强支撑/阻力
- 集中分布 (top 2 buckets > 50% total) = 价格记忆点高浓度区
- 离散分布 = 当前价格区域无明显锚定

### 信号 3: Option Open Interest (OI) 集群
来源：`get_stock_quote.py` 各 strike option codes 取 `open_interest` 字段
方法：
- 看目标 expiry (5/29, 6/18, 7/17 ...) 的 put OI 横向对比
- 找出"OI 比邻近 strike 高 ≥ 3x"的 strike → 机构级 OI cluster

解读：
- Put OI MAX strike = 机构 conditional 仓位最重的位置（无论是 short put 愿意接货还是 long put 对冲，都说明该 strike 是定价 floor）
- 多个 expiry 共同显示 OI cluster 在某 strike → 高 conviction

### 信号 4: K-line 关键技术位
方法：人工识别（或脚本提取）
- 突破日 close（breakout origin）
- 突破后回测 low（breakout retest）
- Panic 单日低点（recent flush low）
- Gap fill 区
- 多日 close cluster（consolidation 中枢）

---

## 收敛评分

| 信号收敛数 | conviction 等级 | 操作 |
|----------|----------------|------|
| 4-5 信号 | 高 conviction | 可加大 size，spread buy leg 锚定此处 |
| 3 信号 | 标准 conviction | 标准 size |
| 2 信号 | 低 conviction | 减半 size 或弃 |
| 0-1 信号 | 单点判断 | 不下单，等更多信号收敛 |

---

## 应用矩阵

### Sell Put Spread 构造 (Bull Put Spread)
- **Sell leg (高 strike)**：在 zone2 中段 (signal 2-3 weight)，**舒适 buffer + 收 premium**
- **Buy leg (低 strike)**：**在 4-5 信号收敛的最强支撑点**，作为 protective floor
- **核心逻辑**：sell 不在最强支撑（避免 max 被 assign 风险），buy AT 最强支撑（如果跌穿 sell strike，protective put 在最强支撑处自动 kick in）

### Sell Call Spread / Covered Call
- **Sell leg**：trim_zone 上沿 (signal 1) + Call OI cluster (signal 3) + ATH + extension (signal 4)
- **Buy leg**（如果是 spread）：no_chase_above 上方
- 类似 bull put 但反向

### Long Call Strike (含 LEAPS)
- **DITM (stock replacement)**：strike 在 zone1 下沿，OI 不必高（不需要 institutional 验证 floor）
- **ATM (thesis expression)**：strike 在 zone1 上沿/no_chase 之间
- **OTM (high赔率/lottery)**：strike 在 trim_zone 上沿或更高 + 该 strike 有 OI（确保流动性）

### Zone1/Zone2 Anchor 设定
4 个信号 EQUAL weight。**任何只用 1 信号锚定的 zone 都是 weak zone**，必须再补 1-2 个信号验证。

---

## 数据获取脚本（脚手架）

```bash
# Volume profile
.venv/moomoo/bin/python3 .claude/skills/openapi/scripts/quote/get_kline.py US.{TICKER} --ktype 1d --num 30 --json

# Option OI for an expiry's put chain
.venv/moomoo/bin/python3 .claude/skills/openapi/scripts/quote/get_option_chain.py US.{TICKER} --start {EXPIRY} --end {EXPIRY} --json
# Then iterate strikes:
.venv/moomoo/bin/python3 .claude/skills/openapi/scripts/quote/get_stock_quote.py US.{TICKER}{YYMMDD}P{STRIKE}000 --json
# Pull `open_interest` field

# Snapshots for spreads
.venv/moomoo/bin/python3 .claude/skills/openapi/scripts/quote/get_snapshot.py {OPTION_CODES} --json
```

---

## 源案例

### STOCK_Y $450 strike (2026-04-30 multi-signal convergence)
- Underlying: $525 mid
- 信号 1 (Zone): $450 在 zone2 (455-470) 外的深档支撑
- 信号 2 (Volume): $450-460 bucket = 106M (近 25 日第 3 大)
- 信号 3 (OI): STOCK_Y 6/18 P450 = **7,603 contracts** (chain MAX，~$340M 名义暴露)
- 信号 4 (K-line): 4/14-4/17 close cluster $456-465 + breakout origin
- 收敛: 4-5 信号
- **Action**: sell 6/18 460/450 spread, buy leg 锚定此 5-signal 收敛点

### STOCK_F $360 strike (2026-04-30)
- Underlying: $397 mid
- 信号 1 (Zone): zone2 (365-380) 下沿之外 + 等于 invalid_if 阈值
- 信号 2 (Volume): $360-370 bucket = 83M (近 25 日 MAX)
- 信号 3 (OI): STOCK_F 6/18 P360 = 2,831 contracts (高位但非 MAX，$350 是 MAX 4,573 但下穿 invalid_if)
- 信号 4 (K-line): 4/16 panic 单日 LOW $360.55
- 收敛: 5 信号
- **Action**: sell 6/18 370/360 spread (sell zone2 mid + buy at 5-signal convergence)

---

## Anti-patterns

| 反模式 | 危害 | 对策 |
|--------|------|------|
| 仅用 zone 选 strike | 错过机构级 OI 信号 / volume 真实支撑 | 必须查 OI + volume profile |
| 看到 OI MAX 就 sell at that strike | sell at strongest support = max assignment risk | OI MAX = buy leg 锚定，不是 sell leg |
| 用单日 K-line 锚定 zone | 缺乏 volume 验证 = 假支撑 | 至少叠加 volume + OI |
| 不查 OI 在多个 expiry 的分布 | 误把短期 hedge cluster 当结构性支撑 | 看 5/29 + 6/18 + 7/17 OI 是否同向 |
| 在 invalid_if 下方 sell put | 即使 OI 大也不算 thesis 支撑（institutional tail-risk hedge） | OI 信号必须落在 invalid_if 之上才纳入收敛 |

---

## 与其他规则接口

- `zone-maintenance.md` step 4 momentum diagnosis 是设 zone 时的 mode 选择
- 设 zone1/zone2 阈值时，**4 信号收敛是 mandatory**（不只是 ATH retrace）
- `sell-put-rules.md` strike comfort 检查在三角验证 PASS 后再做
- `options-strategy-framework.md` 决策树 + 本文件配合使用

---

## 工具速查

完整 Python script (运行后输出 4 信号汇总) 待补 — 现阶段 ad-hoc 跑各组件命令。
