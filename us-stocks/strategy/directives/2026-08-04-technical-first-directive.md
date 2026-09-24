---
tags:
  - trading
  - directive
  - technical
aliases:
  - Technical-First Directive v3
  - 技术面优先 v3
---

> [!info] 裁决指令（directive）镜像
> 记录一条规则**何时、为何、凭什么证据**被采纳（后裁决优先）。原文住私有仓 `trade/strategy/proposals/`；「用户」= 系统的所有者与最终裁决人，「agent」= AI 研究/执行助理。ticker 已替换为 `STOCK_*` 占位符，文中 `quant/...` 路径为私有代码强制点。

# 技术面优先 v3 — 用户 Directive 记录 + 分阶段实施计划

> 2026-09-07 路径迁移：本文历史 ai_news 数据引用现映射到 `trade/intelligence/`；任务归 `tasks/intelligence/`，方法归 `agents/methods/`。详见 `docs/intelligence-architecture.md`；既有交易权限不变，历史原文保留。

> **2026-08-31 宏观相位补充（后裁决优先）**：本 directive 的“技术面单独决定 gate/regime/zone/cap、消息不直接授权”继续有效；“宏观仅是给 用户 的散文 FYI”被替换。`ai_news/macro/current.json` 现在是技术 agent 可读的方向先验和风控 agent 可读的风险环境，但它没有 hard VETO 或任何交易状态写权限，不能代替价格确认，也不能直接改变 gate、zone、cap、stop 或订单。

- **记录时间**: 2026-08-04 03:18 SGT（美股 8/3 周一盘中 ET 15:18，收盘前 42 分钟）
- **背景**: 8/3 大涨日（QQQ +1.7%，STOCK_O +15.6%，STOCK_T 财报后累计 +21%）四小时复盘对话。对话中确认的系统缺陷：231 变更未送达、STOCK_T print 前无卡（stub）、"等 print 却不吃 print" 逻辑洞（stale no_chase 僵尸拦截）。
- **状态**: directive 已记录生效（意图层）；文件/代码/卡片手术 = 8/4 白天部署窗口执行；不可逆项逐条签核。

---

## `1 用户 Directive（2026-08-04 02:30-03:00 SGT，原意忠实转录）

1. gate / regime / zone 的价格层制定**十分合理，保留**。
2. 财报、thesis 等非结构化数据的 agent 判断质量差，"一看越多越犹豫、越犯错"。
3. **决定：现阶段系统量化仅关注技术面。基本面与消息面（财报、thesis、客户订单等）不参与 gate / regime / zone 在价格层面的制定与限制，仅作为附加消息提醒（advisory / FYI）。**
4. 连同本轮对话全部修改点，全面优化系统。

## `2 改什么 / 不改什么

**移入 advisory（不再拦截/降级）**：
- 两腿模型的 thesis 腿（confirmed 升级改为纯结构单腿：收盘 > 动态 lsh ×2 + 量比）
- 财报 print−7 clamp（`max_classification` 财报类约束）→ FYI 横幅行
- "thesis 待验证 → zone 触碰不给资格" 的回踩资格限制 → 按结构资格判定
- 研报因子卡 verified/unverified 对仓位的加权 → 纯提醒

**一件不动（全部为价格层，即"刹车系统"）**：
- 分类 cap（probe/confirmed）与整股约束、集中度 25%/50% + Override 协议
- exit_guard 全栈：利润棘轮 / 趋势死亡线 / G-01 / bfloor_buf 地板
- no_chase / zone / gate ×2 确认 / breakout-stale 重画 / ladder 权重与重锚
- decisions.jsonl / annotate_event 落库纪律；30 分钟规则；Iron Rule #2

## `3 零分歧证明（8/3-8/4，新旧宪法逐票对比）

| 票 | 两套规则的授权 | 首个分歧场景 |
|---|---|---|
| STOCK_O | 均为可加 0 股（结构腿 220 < 230.3，×2 第 0 天；probe 满员） | 8/6 收盘（若 231×2 于 print 前成立：新规给 confirmed 5%，旧规锁至 8/12） |
| STOCK_T / STOCK_Q | 均为明晚 ×2 → 5% cap（本来就是纯结构授权） | 无 |
| STOCK_A / STOCK_M | 均为等 gate ×2；价格在 zone 上方，触碰资格无实义 | 回踩发生时（新规：结构资格；旧规：thesis 限制） |
| STOCK_Z / STOCK_L | 均被**结构** gate 挡（103/120.5、716 全部未成立 ×2） | STOCK_Z 8/6 print 前结构上无法分歧；STOCK_L 粒度 0 股 ≈ 无 |

**结论：今晚（8/3 收盘）简报在两套宪法下逐行相同，仅财报行措辞由"拦截理由"改为"FYI"。“立刻改否则简报错误”的前提不成立；真实的首个分歧日 = 8/6。**

## `4 分阶段实施

### 已完成（今晚，仅渲染层 + 记录，零状态手术）
- [x] 本 memo 落库
- [x] 04:22 收盘检查 cron 按新渲染规则重建（财报/thesis 行 FYI 前缀 + 机械报告两宪法分歧清单）
- [x] 不动 state/rules/code：部署窗口 ≠ 行情窗口（与"未完成的 bar 不是收盘"同一纪律）

### 8/4 白天部署（用户 2026-08-04 03:30 SGT 指令"直接进行白天部署，所有工作完成先"——即时执行，diff 留档供事后 review）
1. [x] **①probe cap 1% → 2%**（`quant/core/sizing.py` CAP_FRACS；粒度修复，STOCK_T 模拟中最大单一杠杆）— 完成 04-04 03:32，pytest 472 绿（3 个预算测试改为从 CAP_FRACS 推导）
   - **①-补丁（同日 SGT 晚间，用户 盘前问"probe 不是 2% 了吗"抓出）**：regime 层两处漏网——`regime_transition.py` reclaim 生成器硬编码 `cap_pct_nav: 0.01` → 改读 `CAP_FRACS["probe"]`（单源）；四张已画修复卡（STOCK_IN/STOCK_L/STOCK_M/STOCK_Z）`next_regime.cap_pct_nav` 0.01→0.02 官方脚本写回（min() 只收紧语义下旧值会把授权按回 1%）；第 4 个钉 0.01 的预算断言改推导。pytest 472 绿。赶在当晚收盘 ×2 成立、`--apply` 把旧值烧进 current_regime 之前。
2. [x] ~~**②confirmed 5% 里程碑升档机制**——文档化默认值（数值 用户 可再调）：连续 **4 周**零授权违规 **且** 窗口内全部 🔒📉🎯 告警当日执行或书面 override → confirmed 5%→**7%**；再 4 周 → **8%** 封顶；任一违规重置计时并回落一档。生效方式 = 届时手改 `CAP_FRACS["confirmed"]` + 本 memo 记录，无需新代码~~

   > 🚫 **SUPERSEDED 2026-08-11**（用户 批）。8/8 冲击预算定版把 `CAP_FRACS["confirmed"]`
   > 从 5% 改成 **0.40 = `NOTIONAL_CAP`**（单票名义天花板），"5%→7%→8%" 升的是一个
   > **已经不存在的数**——照着执行会把单票天花板从 40% 砍到 7%。这条留在文件里
   > 三天没被作废，正是本仓典型的旧记忆地雷（ledger A5 同型）。
   >
   > **机制本身不废**（4 周零违规 → 放宽，执行率激励是 8/4 定的，值钱的是机制不是那三个数）。
   > 新载体 = **两个都挂**（用户 2026-08-11 A5）：
   > - `ROUND_RISK_FRACS` 的 **T = 5% NAV 顶格**（每个 campaign 总共冒多少险；
   >   与 `AT_RISK_CAP_FRAC` 同一个数，改要同改，相等由 pins 测试钉）
   > - `CAP_FRACS["probe"]` = **2%**（试探档粒度）
   >
   > `CAP_FRACS["confirmed"]` **不再是激励旋钮**——它是单票名义天花板，归冲击预算框架管。
3. [x] **③财报 clamp → advisory**（六卡 max_classification 已移除：STOCK_O/STOCK_L/STOCK_Z/STOCK_M/STOCK_A/STOCK_GO；STOCK_K premium gate 保留；官方脚本 --max-classification none）
4. [x] **④两腿 → 结构单腿**（entry_guard：thesis 降级/gate_only thesis 半/event_day 降级 全部改 [FYI]；结构降级判别式保留；docstring 更新）
5. [x] **⑤动态 lsh 读数 + 下移条件渲染上卡**（231 事件的修复）— entry_guard 侧原有（"修复第一信号"行）；position card 结构行新增 `动态 lsh $X (修复线权威读数…卡上 gate=静态保守锚)`（portfolio_brief 873 段，report 测试 57 绿）
6. [x] **⑥卡片变更播报通道**——v1 = 任务 prompt 强制（briefing 顶部 git-diff 式变更区块 + 改卡当场 tg_notify）；代码级 diff 渲染器留待后续
7. [x] **⑦verified 事件 day-1 协议**——v1 = 定时任务点名 + zone-maintenance v3 注（own-print event-stale 重画 SLA=当日，stale no_chase 无否决权）；day-1 ladder（60m 首根 → ½ confirmed → 首收盘补足，stop=day-1 低点/gap 下沿）待预注册回测后接线 entry_guard。首个实战窗口 STOCK_Z 8/6
8. [x] **⑧pre-print 建卡 SLA**——v1 = briefing prompt 强制（NEEDS_CARD + 已知 print ≤3 交易日 → 红色待办置顶）
9. [x] 卡片手术：约束移除完成（六卡）。**STOCK_O 纯技术面重画完成**（8/3 收盘 bar + level_map：z1 197-204.6 / z2 187-196 / trim 226.8-234.4 / no_chase 222 / stop 175.1 regime 地板不动 / next 231 保留；probe cap 2% → 回踩 z1 可加 1 股，预算余 ≈$259）。STOCK_T/STOCK_Q 卡当日（8/3 晨）已锚 + 突破转移 1/2 armed → **免重画**（转移+pending_repaint 机制接手）。遗留：STOCK_O current_regime 块补写待下次 /trading-analysis（CLI 无该字段入口，不做裸 JSON 手术）；五张 watchlist 卡的旧 clamp prose 随自然重画淘汰
10. [x] 规则文档同步：trading-discipline / position-tiers / zone-maintenance / bayesian-decision-model / CLAUDE.md 全部加 v3 批注（历史保留）
11. [x] pytest 472 全绿（3 个预算测试改由 CAP_FRACS 推导）；commit A 已提交
12. [x] **情报层接入**：两个任务 prompt 已加情报层区块（`A 宏观含 USDJPY / `B 个股一行式 / 零解读纪律），已 cp 同步 live SKILL.md（03:37）

## `7 情报层 v1（用户 2026-08-04 授权 agent 先定版式，观察 1-2 周后再决定是否定模版）

**定位**：纯 FYI，附于 overnight briefing 与 heartbeat 对应位置；不参与任何授权判定；判断权归 用户。

**A. 宏观（优先级 1）**
- 事件日历：未来 5 个交易日 CPI/PPI/NFP/FOMC/重要讲话 + 持仓与 watchlist 已定档财报日（日期取官方源：BLS/Fed/公司 IR）
- 隔夜读数：大盘/期货、10Y + DXY、VIX、油价（地缘标尺）、**USDJPY**（2026-08 起列入——日美协同干预进行中，carry unwind 观察项）
- 突发：关税/地缘/Fed 要员，只报主流媒体过夜仍在头条追踪级别

**B. 个股（优先级 2）**：持仓 + watchlist 全票
- 每票 ≤1 行；**无新闻 = 不出现**（信噪比第一，空行胜过凑数）
- 标签：[财报] / [订单·客户] / [评级] / [公司事件] / [板块 read-through]（如 STOCK_G print 对 STOCK_O）
- 来源分级照旧标注：官宣 > Reuters/Bloomberg > 财经媒体 > 传闻（"报道≠官宣"保留为**标签**，不再是 gate）

**C. 格式纪律（"拉取有效信息"的操作定义）**
- 一条 = 一行事实 +（来源，日期）；**不附解读**、不写"这可能意味着"
- 边沿去重：已报且无新进展不重复（对齐 heartbeat 边沿触发哲学）
- 拉取工具：WebSearch + 既有 `macro_fetch.py`（VIX/FRED）/ `x_fetch.py`；实现进 tasks prompt（`4.12）

**D. 评估**：跑至 ~8/15（覆盖 STOCK_Z 8/6 / STOCK_L 8/11 / STOCK_O 8/12 三个 print 周），用户 验收后决定是否升级为正式信息流模版。

### 并行验证（不 gate 上线，结果出来按 process>outcome 复盘）
- structure-only vs 两腿（修复路径 episode 池，gate_backtest2 基建）
- 财报 clamp on/off 前向分桶（earnings_gate 复测预注册的提前局部版）
- day-1 协议（own-print beat 格子，PEAD 口径）

## `5 Agent 保留意见（记录后即服从执行）

1. **时点**：修宪发生于 +15% 日盘中 02:30、8/3 纠错后第 2 个交易日——与 pre-commit 铁律的适用条件同形。记录在案，供未来复盘对照，不阻塞执行。
2. **日历事实 carve-out — 已裁决（用户 2026-08-04 03:25 SGT）：不采纳，全 advisory。** 原文："不会再存在什么 print 前 clamp 的问题，结构该怎么样就是怎么样。" print−7 clamp（含 用户 7/31 自定 fiat）随 v3 废止；财报日期只进情报层日历，不进任何授权判定。agent 建议记录在案，服从执行。
3. **被说服的部分（诚实记录）**：本系统教义"不进代码的规则 = 不存在"，thesis 层是唯一结构上无法全部进代码的层——按系统自己的哲学它就是最弱环节，8/3 夜四个洞全部出在此缝，价格层零伤。刹车全在价格层，本 directive 不拆刹车。

## `6 部署完成记录（2026-08-04 04:15 SGT）+ 遗留清单

- [x] `2 范围（用户 03:25 消息确认：clamp 全废、情报层分工）
- [x] `4 全部 12 项部署完成（用户 03:30 指令"直接进行白天部署"）
- [x] `5.2 carve-out：用户 裁决不采纳，全 advisory
- [x] commit A `4a091e7`（code/rules/tasks/tests/memo）+ commit B/C（state + 渲染收尾）
- [x] 收盘检查已推 Telegram（04:10，v3 渲染）；STOCK_O 重画完成
- [x] **全卡刷新（用户 04:25 指令）**：STOCK_GO 修复梯退役→常规 zone 卡 + R_突破_379.5（342 切换已确认吸收）；STOCK_L 重锚为自洽转移前卡 + next_regime 真实分布（716 计数 1/2 保留，652/683 退役）；STOCK_Z/STOCK_IN v3 prose 清洗；STOCK_M/STOCK_A/STOCK_Y/STOCK_T/STOCK_Q 审计后免动（reason 纯结构 + 梯子计数中，中途重锚=作弊）；STOCK_K/STOCK_AK 已移出 券商 分组不评估
- **遗留（白天/后续）**：① STOCK_O current_regime 块补写；② SOXX/SMH stub 建卡（要参与先 /trading-analysis）；③ `4 并行验证三回测预注册（不 gate 上线）；④ day-1 ladder 接线（回测后）；⑤ 情报层 v1 效果观察至 ~8/15 → 用户 决定是否定正式模版
- **8/4+ 议程（用户 提出，待讨论）**：⑥ **情报源渠道补充**——X 平台等（现状：`quant/data/x_fetch.py` 已存在于数据层、研报因子卡历史上多来自 X KOL；待定 = 频道/KOL 白名单、进 briefing `B 的筛选纪律、与"来源分级"如何衔接）；⑦ **TradingView 接入评估**——更全技术数据与 K 线（评估框架：TV 无官方行情 API，路径 = 非官方库/付费 webhook；先盘点现有栈覆盖面 [券商网关 日线+60m/level_map 四信号/Yahoo 指数] 的真实缺口再决定，60m 量价下沉是已记档需求但 券商网关 本身有 60m kline）
- **⑧ 盘后转移简报（2026-08-05 用户 指令，同日落地）**：盘后+夜盘允许交易 → 转移/重画/提醒线同步必须发生在**盘后**（SGT 04:10 cron `10 4 * * 2-6` = 夏令 ET 16:10 收盘后 10min；2026-08-05 与 gate-ladder-report 换序【postclose 前置——转移在第 1 步 pipeline ~2min 落地，ladder `20 4` 渲染转移后世界】；冬令 postclose→`10 5` / ladder→`20 5`，忘改有盘中自检【pipeline 中止 + TG 告警，rc=3 入测试】；registry 已备份），并在简报中说明**达标条件**（线+口径+两个达标收盘的日期与收盘价）与**转移过程**（旧→新 regime + 新授权 + 整股股数）。实现 = pipeline 新 `postclose` 模式（klines→regime→reminders，lean 合同入测试）+ `tasks/claude/postclose-transition.md` 任务（说明渲染/当日重画 SLA/重画后二次同步提醒线/夜盘纪律尾行）+ Desktop 注册表登记。briefing 的 regime 步保留为幂等兜底。
- **⑥⑦ 决议（2026-08-05 用户 拍板）**：⑥ **bird 全面弃用**（非官方通道断无告警 = 静默失效源；trading-analysis SKILL.md 6 处引用已清）→ **`quant/data/x_search.py` 上线**（官方 /2/tweets/search/recent，现有 token 实测 200；主动搜索像 WebSearch 一样用，深读单条仍走 x_fetch；计费 pay-per-use 按条，max 默认 10 上限 25，窗口 7 天；产出只进情报层 [FYI] 零解读）；T1 白名单 timeline polling **待实证**（门槛：X 独有且早于 WebSearch 的可用情报 ≥2 条/周 × 2 周，再谈付费轮询）。⑦ **TV 不接**（用户 条件：gate/regime/zone 所用指标能准确实时获取即不接——盘点满足：`technical.py` sma/ema/rsi/macd/bollinger/atr/swing/背离 + `level_map` VP/HVN/pivot/OI + 券商网关 1m-60m-日线与实时 snapshot；**适配系统而非系统向下兼容 = 实时读数只喂情报/监控层，授权计数只认收盘 bar**，kline_cache 丢弃未完成 bar 的既有闸门即此约束）；替代立项待排期：(a) gate/stop 价位 → 券商 `set_price_reminder` 自动同步（补 6h heartbeat 间隔内即时性）；(b) 60m 量价下沉进 level_map / catalyst_check（数据已够，纯代码）。

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[2026-09-12-two-mode-directive|两种模式]]
