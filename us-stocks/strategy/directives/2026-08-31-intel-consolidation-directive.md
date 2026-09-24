---
tags:
  - trading
  - directive
  - macro
aliases:
  - Intelligence Consolidation Directive
  - 情报层收拢
---

> [!info] 裁决指令（directive）镜像
> 记录一条规则**何时、为何、凭什么证据**被采纳（后裁决优先）。原文住私有仓 `trade/strategy/proposals/`；「用户」= 系统的所有者与最终裁决人，「agent」= AI 研究/执行助理。ticker 已替换为 `STOCK_*` 占位符，文中 `quant/...` 路径为私有代码强制点。

# 2026-08-31 情报层收拢 Directive（STOCK_IN Consolidation）

> 2026-09-07 路径迁移：本文历史 ai_news 数据引用现映射到 `trade/intelligence/`；任务归 `tasks/intelligence/`，方法归 `agents/methods/`。详见 `docs/intelligence-architecture.md`；既有交易权限不变，历史原文保留。

> Status: **live**（当日生效，当日接线完毕）
> 裁决人：用户（2026-08-31）
> 执行：本系统（本文档 = canonical；CLAUDE.md Core Decision Model 段有摘要）

## 用户 原话（逐字）

> 「请移除所有除了 ai_news 目录的情报层 后面这些情报仅由 ai_news 负责（即另外一个 agent 负责获取）」
>
> 「即后续 claude 不再负责宏观部分 仅专注技术分析和风控」

## 一句话

**全系统唯一情报通道 = `ai_news/`**（独立的 Claude 定时任务 `ai-news-radar`，每日 05:30/20:30 SGT
收集归档并自行推 TG——2026-09-03 由 Codex 迁回，见文末追记）。本仓（本系统 交易系统）**只做技术分析 + 风控**：
不再拉取、播报、转述任何新闻/宏观/社媒情报；需要消息面 context 时**只读** radar 归档
（`ai_news/reports/*/*.telegram.md`），归档没有 = 触发源未知（合法输出，不编造）。

## 与既有 directive 的关系

- **v3 技术面优先（2026-08-04）**：不变。v3 把情报降为 [FYI] 不驱动授权；本 directive 把
  「谁去取 [FYI]」也移出本仓——授权层一个字没动。
- **入场委托（2026-08-18）**：不变。分工四格照旧，简报 `4 挂单清单照旧。
- v3 的两个 sizing/地图侧例外**保留**（它们是风控不是情报）：财报反应日 T+0 缩仓
  （`core/earnings.py`）+ print 后重画 SLA（`INFRA:card_stale_earnings`）——数据源是本地
  `earnings_calendar`，不是新闻。

## 退役清单（本次全部落地）

| 项 | 处置 |
|---|---|
| X 通道 `quant/data/x_search.py` / `x_fetch.py` | **删除**（含 `tests/data/test_x_search.py`、DoF 注册表条目、module-coverage 名单项） |
| 三认可 X 源（@financialjuice 2026-08-06 / @SemiAnalysis_ 2026-08-06 / @dnystedt 2026-08-18） | 退役出本仓工具箱；评审史与 8/5 STOCK_GO 归因源案例以本文档为归档。速报时效职能归 radar |
| heartbeat「重大突发」检查 + 📰 情报段（产业轨 / 周一回看窗 / WebSearch 遍历义务） | 整段退役（tasks/claude/heartbeat.md 4 段 → 3 段；`.claude/rules/heartbeat.md` 检查项 3 墓碑） |
| briefing `A 宏观数据日历（WebSearch）+ 隔夜读数行（`macro_fetch --live` 简报职务）+ 🗣 叙事行 | 退役；`A 瘦身为 💵 财报（本地）+ 📊 指数收盘（kline） |
| briefing `B 情报段（轨① 个股新闻 / 轨② 产业主题） | 整段退役；轨②事件类型清单与源案例（Gemini 3.7 Flash 8/13、STOCK_A CPO 8/17）归档于此，设计遗产供 radar 侧参考 |
| 🌐 宏观观察行（2026-08-25 立，仅活 6 天） | 整链删除：`portfolio_brief.macro_lines()` / pipeline `klines_macro` 步 / `kline_cache` 宏观三票 TNX·DXY·BTC + `YH_UTC_DATED` UTC 守卫 + DXY 17:05 特例 |
| weekly review「下周情报」WebSearch 日历 | 换本地 `earnings_calendar`；宏观日历不采集 |
| 研报决策因子库 `trade/knowledge/research/` | **冻结为只读归档**（README/INDEX 带 🧊 横幅）；Pre-Trade Quick Check **Q4 退役 → 三问**；CLAUDE.md session behavior 的 INDEX 查阅义务删除。历史卡不删——git 里的知识不销毁，只摘活性 |
| trading-analysis skill Step 3 新闻分析师 + Step 4 情绪分析师 | 合并为「Step 3 情报回顾」：只读 radar 归档，[FYI]，无归档命中 = 「情报层：无输入」。六步 → 五步（`six_step.md` 文件名沿用） |
| `card_autopaint` 子 agent prompt 的 x_search/WebSearch 指令 | 删除；财报日期改用本地 `earnings_calendar` |
| [[macro-context-check]] 的 X/WebSearch 工具段 | 重写：tape 定时点 → radar 归档 → 触发源未知；**判断逻辑（个股/板块/宏观/机械 flow 分层）原样保留** |

## 保留清单（自有量化数据 = 技术/风控，不是情报获取）

| 项 | 为什么留 |
|---|---|
| `earnings_calendar.py`（财报日历） | 喂 T+0 缩仓（sizing 例外）+ 重画 SLA + day-1 点名——风控与地图维护 |
| 指数/板块 kline（NDX/SPX/SOX/VIX + QQQ/SPY/SMH/SOXX） | 归因的「大盘+板块」腿（macro-context-check 检查清单 1）+ `index_risk` 大盘 -2% 告警 + weekly 基准 |
| VIX via `macro_fetch.py`（FRED） | `regime_score` 的量化输入（加仓执行三旋钮）。`--live` CLI 保留为工具，简报职务已摘 |
| `tape_check.py` / `catalyst_check.py` | 纯量价机械层；catalyst_check 的「触发源未知」正是本 directive 下的常态出口 |
| `ainews_push.py`（雷达自推 / 事件 runner 直推 / `ai-news-push` 扫尾） | 唯一情报通道的投递侧：零交易语义，原文搬运。2026-09-03 起采集侧也是 Claude 任务——「Claude 不参与」自此指**交易任务**不采集不解读，不再指 vendor |

## 归因新流程（macro-context-check 重写后）

1. **tape 定时点**（60m K 线钉住异动时段）
2. **大盘/板块**（自有 kline：全板块 vs 个股独立）
3. **事件叙事**：只读 `ai_news/reports/` 归档
4. 三条都对不上 → **触发源未知**（catalyst_check 合法输出）；补新闻的职责归 radar，不归本仓

## 防复活棘轮

- `tests/data/test_kline_cache.py::test_macro_observation_tickers_stay_retired`
  （TNX/DXY/BTC 不得回 `INDEX_YH`；`YH_UTC_DATED` 不得复活）
- `tests/ops/test_pipeline.py::test_intel_layer_stages_stay_retired`（`klines_macro` 步不得复活）
- `tests/test_repo_references.py`（tasks/ live 副本对账 + 路径引用——x_* 残留引用会红）

## 追加（同日）：雷达目录转正入仓

用户：「ai_news 允许正式进入 本系统 并且放到合适的目录下吧 加进 git」。落地：

- 旧雷达目录 → **`ai_news/`**（repo 根，可见目录；内部 `*` gitignore 删除）
- 版本化：`PROMPT.md` / `README.md` / `reports/` 归档；`state/` 运行时去重不入 git
- 新报告的提交：`commit_state.sh` PATHS 加入 `ai_news`（briefing/weekly 每日顺路扫入；
  雷达自己照旧不碰 git，硬边界不变）
- 雷达收集 runner 已直接绑定 `ai_news/`；旧兼容入口已删除，不再支持旧路径

## 已知代价（知情选择）

- **归因时效**：radar 每日两收（05:30/20:30 SGT），盘中突发事件在下一次收集前查不到叙事
  ——那几个小时的答案就是「触发源未知」。tape/板块判定不受影响（自有数据实时）。
- **10Y/DXY/BTC/USDJPY/WTI/Gold 读数**从此不在任何简报出现；用户 从 radar 报告或行情 App 看。
- **宏观数据日历（CPI/NFP/FOMC 时刻表）**不再进简报——v3 之后它本来就只是 [FYI] 不拦任何动作。
- 研报因子卡停止新增：新研报的消化归 radar 报告；用户 对话里贴的研报照常可用（他提供 ≠ 本仓拉取）。

## 后续裁决：宏观相位成为系统只读先验

用户（2026-08-31）追加裁决：“宏观是方向先验和风险环境，不直接交易；新指令替代旧的宏观 FYI 限制；采用固定运行加重大事件复评；三个时间尺度；结构化历史入 Git。”

这条只替换本文“宏观仅为散文 FYI”的部分，不撤销情报层收拢：

- 采集与分析仍只有 `ai_news/` 负责；交易本体不自行 WebSearch 或重复拉宏观新闻。
- `ai_news/macro/current.json` 是最新成功且通过 schema 的状态；技术与风控 agent 只读消费。
- 技术 agent 取得方向先验和相位 context，但价格结构仍独立确认；宏观不能绕过 gate/zone。
- 风控 agent 取得 `supportive/neutral/adverse/crisis` 和事件尾部路径，但仓位/cap 映射仍由风控规则拥有。
- 宏观 agent 的 hard VETO、订单、持仓、watchlist、zone、gate、cap 写权限恒为 false。
- 固定 05:30/20:30 SGT 维护 event window、1–10 个交易日、1–3 个月三个尺度；Tier-1 事件事前 plan、事后 result 分开且 plan 不可覆盖。
- 当前状态、append-only history、事件 plan/result 和原始 snapshot 纳入 Git；`ai_news/state/` 仍只放 ignored 的 staging/claim。

因此上方“宏观数据日历不再进简报”仍成立（交易简报不恢复日历采集），但不再等于“宏观没有系统输入”：新输入由 radar 生成，交易系统只读。

## 追记 2026-09-02：投递侧归回 Claude `ai-news-push`

用户：「codex 生成完后发送不了 tg，重新接手该定时任务」。08-31 定的「Codex 同轮直投」实测两种失败形状：

1. **定时 shell 无 DNS**——Codex Scheduled 的执行环境解析不了 `api.telegram.org`，联网升级被安全审查拒绝
   （8/31–9/1 JOLTS/ISM 事件：plan 侥幸送达，result 从未送达）。
2. **插件 sha pin**——改走 `ai-news-telegram@personal` 插件后能发了，但插件把本仓 `tg_notify.py` /
   `paths.py` / `ainews_push.py` / `io_json.py` 四个文件的 sha pin 死；09-02 估值层改 `paths.py`、
   座右铭改 `tg_notify.py`，当晚 20:48 premarket 报告落地没出门。**别人的安全 pin 让自己的代码改动
   静默杀掉通道**，且 pin 失配没有任何出口会喊。

裁决：**投递归回 Claude `ai-news-push`**（05:40/20:40 SGT，`quant/ops/ainews_push.py --wait 480`，
真源 `tasks/claude/ai-news-push.md`）。情报层收拢**不变**——Claude 仍不采集、不解读、不转述，投递 = 原文搬运。
Codex 固定轮不再发送（归档 + 宏观成功即推进 `last-run.json`）；事件 runner 的插件发送降为可选快路径
（失败 = `TELEGRAM_HANDOFF` 不算本轮失败，Claude 下一槽按 sha 扫尾）。去重真源 = 主仓
`tmp/ainews-push-state.json`，两条路共用，永不双发。Codex 侧两个 automation 的 prompt 真源
`ai_news/CODEX_AUTOMATION.md`，**改 repo 不会自动改 Codex，要在 ChatGPT 桌面端手动粘贴**。

## 追记 2026-09-03：采集侧也迁回 Claude（Codex Scheduled 全部退役）

用户：「我对 codex 的定时任务彻底累了 各种限制还不能联网 请将 codex 的定时任务全面迁移至 Claude」。
08-31 到 09-03 三天里 Codex 侧实测的形状：定时 shell 无 DNS、插件 sha pin 一改代码即失配、外发被安全审查
反复拒绝、单任务最快每小时一次（事件 runner 得拆两个）、prompt 只在 Codex 注册表里改 repo 不会跟着改。

裁决：**本 directive 的分层原则不变，vendor 换掉。** 「Claude 不再负责宏观」自此读作
「**交易任务**（heartbeat / briefing / postclose / ladder / chart-read / weekly）不采集、不解读、不转述」；
情报只住 `ai_news/` 目录与两个专用 Claude 任务：`ai-news-radar`（固定轮，`tasks/claude/ai-news-radar.md`，
唯一允许 WebSearch/WebFetch 采集情报的任务）与 `macro-event-recheck`（Tier-1 事件，
`tasks/claude/macro-event-recheck.md`，一个任务盖住原来两个）。两者归档后自己推 TG，`ai-news-push` 退居扫尾员 +
dead-man（07:00 / 22:00）。归因新流程、退役符号棘轮、`macro/current.json` 的只读契约一个字不变。
迁移账：`ai_news/CODEX_AUTOMATION.md`。

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[macro-context-check|宏观归因检查]]、[[macro-inference-contract|宏观推导合同]]
