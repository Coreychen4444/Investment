---
tags:
  - trading
  - framework
  - exit
  - risk-management
aliases:
  - Trend Exit System
  - 赢家侧退出栈
  - 利润棘轮
  - Profit Ratchet
---

# Trend Exit System — 赢家侧退出栈（利润棘轮 + 趋势死亡线）

> Created 2026-07-04。源案例：STOCK_X LEAPS episode C（2026-07-02 margin 强平 -$2,295，G-01 +100% 在 6/2 客观触发却零执行）+ STOCK_X 短期 $11C episode C（2026-04-24，+119%→flat）。**同标的同错误间隔六周 = pattern 不是 variance。**

## 定位

**打的问题**：输家侧止损纪律已修好（thesis-invalidation 果断离场）；出血点在**赢家管理**——两次 round trip 都是盈利单。核心发现：规则不缺（G-01 白纸黑字写在每个 lot 的 exit_trigger），缺的是**触发监控自动化 + 触发后执行**。执行率 0% 的系统等于没有系统。

**设计原则**：卖飞和硬抗是同一旋钮的两端，单一止盈线无法同时最小化两者。本系统用**分批 + 只升不降的地板 + 重进协议**改变 payoff 结构：每种错误各付小保费，不做二选一。

**与既有栈的关系**（完整退出栈 = 5 件）：

| 层 | 规则 | 打什么 | 来源 |
|----|------|--------|------|
| G-01 阶梯 | +100/+200/+300% → trim 30/30/40 | 卖飞（let winner run，只减不清） | 既有 |
| **利润棘轮** | 本文 §1 | 硬抗（利润回吐上限） | 新增 |
| **趋势死亡线** | 本文 §2 | 硬抗（趋势结束强制离场） | 新增 |
| invalid_if / thesis stop | 破位或基本面证伪 | 输家硬抗 | 既有 |
| DTE 阶梯 | 强制评估节点 | 期权时间衰减 | 既有 |

## §1 利润棘轮（giveback floor）— 硬线，无豁免

按 position 级追踪 high-water mark（HWM），地板只升不降：

| 档位 | 触发 | 地板 |
|------|------|------|
| 保本档 | 峰值浮盈曾 ≥ **+50%** | floor = 成本价。**+50% 的赢单永远不允许变亏单** |
| 锁利档 | 峰值浮盈曾 ≥ **+100%** | floor = 成本 + 峰值利润 × 2/3（**回吐上限 = 峰值利润的 1/3**） |
| DTE 收紧 | 期权 DTE < 90 | 回吐上限收紧至 1/4 |

**触发动作（两段式软着陆）**：
- 破锁利档地板 → **stage1**：今日内卖 50%（期权整张取整；单张仓 = 全平），剩余地板抬至保本
- **stage2**：剩余部分破保本 → 全平
- 破保本档地板（峰值只到 +50-99%）→ 一段式全平
- 触发同时 **no-add lock ON**：该标的禁一切加仓，直到结构修复（close > 触发时刻的高水位收盘）。这条机械杀死"地板破了还逆势加仓"的失败模式
- **新 leg 重新武装**：fire 后 HWM 再创高 >1% → fired 标志复位，地板随新 HWM 抬高

## §2 趋势死亡线（chandelier trail）— 硬线，无豁免

在**正股日线收盘**上计算（期权仓看 underlying），线只升不降：

- **normal 模式**：trail = 布防以来最高收盘 − **3 × ATR(20, Wilder)**
- **parabolic 模式**：trail = trailing **5d high × 0.92**（与 parabolic zone 锚一致，≈ 2.5×ATR）
- 收盘破线 → 平剩余 / 至少减至 base 层；同时 no-add lock ON
- **不越价约束**：trail 不允许抬升到现价之上（止损线只能从价格下方拖尾）。暴跌日公式值越过 close → 本轮不抬线，用旧线判破位
- 崩盘中段首次布防：历史破位不追溯告警，回落到 ATR fresh-start 线

**ATR 是什么**：Average True Range = 该标的"一天正常呼吸的幅度"。TR = max(H−L, |H−昨收|, |L−昨收|)——后两项抓跳空（隔夜 gap 是持有人的真实波动）。3×ATR 的回落 = 三天呼吸量单向堆叠，统计上不太可能是噪音 = 趋势大概率死亡。相比固定百分比止损，ATR 自适应波动 regime：狂暴期线自动放宽（不被牛市震荡洗下车），安静期自动收紧。

**道氏结构标签**（佐证层，机械计算不目测）：5-bar fractal pivot → 最近两个 swing high/low 比较 → HH/LH + HL/LL。**HH+HL = 趋势活着；LH = 病危通知；LL = 死亡确认。** 结构标签比趋势线敏感但噪音多 → 定位为佐证 + 加仓资格 gate（结构 down 不给加仓），硬触发平仓仍由棘轮和趋势线负责。

## §3 重进协议 — 卖飞的真正解药

卖出 ≠ 离婚。**卖飞只在"卖了之后拒绝重进 + 资金闲置"时成立**（见 [[sell-fly-vs-rebalance|卖飞判别]]）。
- 结构修复定义：收盘突破触发离场的那个 lower high（= 新 HH）→ no-add lock 自动解除
- 修复后按**新 tranche** 走正常 entry 纪律重进（ladder / higher-low / zone 检查全套适用）
- Round trip 差价 = 为利润保护付的保费，不是错误
- **双向 trap 防护**：本系统的教训不是"涨了就跑"。G-01 分批设计保留 let-winner-run 尾部；棘轮只在回吐超限时动手。把教训内化成"早止盈"，下次就是主升浪 +100% 踏空——同一枚硬币的另一面

## §4 宏观裁决

宏观/机械踩踏 lens（负 gamma、CTA、强平）只作用于 **(a) thesis 后验更新**（不因 flush 判 thesis 死）和 **(b) 重进速度**（机械踩踏修复快 → 可以更快重进）。**它不豁免棘轮和趋势死亡线。** 地板保护的是已积累利润和资金久期，与下跌"原因"无关；"机械踩踏会弹回来"是统计倾向不是担保——源案例那次就没弹回来。想赌修复，用重进协议赌，不是用不卖赌。

## §5 Margin guard — 资金久期 ≥ thesis 久期

源案例第三层失败：margin 随时可 call，thesis 要等下季 earnings。**"基本面没坏就拿着"的入场券是无杠杆资金。** 红灯（购买力仅剩 1.2% 市值）在数据库躺了一周无人消费 → 最高风险等级被迫平仓，卖出时点的选择权交给了 broker。

- power/market_val < 5% → WARN（本周内 delever 或停止一切加仓）
- power/market_val < 2% 或券商风险等级进入危险区 → CRITICAL（今日 delever，按 conviction 排序自己选，不要等 broker 替你选）
- 账户数据 >3 天未更新 → 监控盲飞本身要报警
- cash < 0 期间：timing-sensitive LEAPS 的"拿到 catalyst"假设默认不成立

## §6 执行军规 — 从 0% 执行率来的半壁江山

1. **告警即行动**：告警文本自带预写行动指令（卖几张、地板在哪），当日收盘前执行，或书面 override 写进 journal（override 留痕，月度数次数；≥2 次/月 = 系统性问题）
2. **触发时刻的你不可信任**：两次实证，峰值亢奋日人会说"才刚开始"。能预挂 GTC stop 就预挂（正股安全；期权链薄默认告警制）
3. 参数改动必须记录 §8 并说明理由——防"这次不一样"式软化

## §7 自动化

监控脚本每轮巡逻（6h）自动跑：HWM/地板/趋势线/G-01 档位/结构标签/margin 全量检查，状态文件地板与 HWM 只升不降，fired 标志一次性防重复告警。盘前简报输出**每标的开盘操作手册**（8 级优先级：margin > 棘轮 stage2 > stage1 > 趋势线破 > invalid_if > no-add lock > zone 加仓 > 持有；宏观事件日 CPI/NFP/FOMC 自动降级加仓动作，减仓不延期）。

## §8 参数记录

| 日期 | 参数 | 值 | 理由 |
|------|------|-----|------|
| 2026-07-04 | 棘轮 | +50%→保本 / +100%→回吐 1/3 / DTE<90→1/4 | 两案回测最优 |
| 2026-07-04 | 趋势线 | 3×ATR20 normal / 5d×0.92 parabolic | 教科书 chandelier；与 zone 锚一致 |
| 2026-07-04 | margin | warn 5% / crit 2% | 红灯位 1.2% 校准 |

## §9 源案例回测（参数非拟合，教科书默认值）

STOCK_X Jan-2027 OTM LEAPS，16 张 avg $2.541，峰值 mark $5.25（+107%）：

| 规则 | 触发点 | 结果 |
|------|--------|------|
| 棘轮锁利档（floor ≈ $4.35） | 峰值后首个大跌日（mark 从 4.93 砸穿） | 16 张 @ ~4.3-4.4 → **+$2,800** |
| ATR 趋势线 | 同一天收盘破线 | 两条独立规则同日收敛 |
| 结构层（反弹确认 LH → trim；前低跌破 LL → 清） | 其后 1-3 周 | **+$1,400-1,800** |
| 仅保本档（最弱参数） | mark 回落破 avg cost | **≈ $0** |
| 实际（无系统） | margin 强平 | **-$2,295** |

且棘轮触发 → no-add lock → 峰值回落途中的 lower-high 加仓（实际 -$780 = 34% of loss）机械不会发生。**-$2,295 里约 $2,000 是执行失败的定价，市场只欠 $300。**

## 引用根基
[[post-trade-scoring|事后评分]]（process > outcome）· [[sell-fly-vs-rebalance|卖飞判别]]（重进协议依据）· [[bayesian-decision-model|Bayesian 决策模型]] 铁律 #5（pre-commit 防 decay——本系统是该铁律的机械化）

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[post-trade-scoring|事后评分]]、[[sell-fly-vs-rebalance|卖飞判别]]、[[uncertainty-execution-system|不确定性执行系统]]、[[Mindset_Risk_Control|交易心法与风控]]
