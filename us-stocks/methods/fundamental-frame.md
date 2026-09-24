---
tags:
  - methods
  - fundamental
aliases:
  - Fundamental Frame
  - 基本面框架
---

> [!info] 方法规格镜像
> 原文住私有仓 `agents/methods/`，是 AI 研究角色共用的方法规格。宏观 / 基本面结论在本系统**零交易授权**：只作方向先验与风险环境，不产生买卖、仓位或退出规则。

# 基本面框架（`agent.thesis.frame`）

判断层，**零授权效力**：不改 zone / gate / cap / stop，不给入场时机，不出评级和目标价。形状与替换规则的判别式 = `quant/core/thesis_frame.py`（schema、写卡 writer、提交边界共用）；本文件是写法规格，`/trading-analysis` Step 1 与 technical 角色共用这一份。

骨架取自卖方个股研报（用户 2026-09-15 贴的半导体周报个股段）：**假设 → 驱动 → 空头与反驳 → 估值交叉验证 → 下一催化剂预案**。三件不取：

- **评级与「好的入场点」**：入场只看价格结构（技术面优先 v3）。「跌了 X% 所以值得买」在本系统是抄底判别式与 gate 转移机的事，不是基本面结论。
- **目标价**：估值层 2026-09-08 起 publication withheld（`quant/valuation/model.py` 的 `publication_policy`），且零授权。
- **「强劲 / 一般 / 偏弱」式评级词**：换成可证伪的驱动与条件。

## 槽

| 槽 | 写什么 | 不许 |
|---|---|---|
| `as_of` | 本 frame 所引最新基本面证据的日期 | 用重画日冒充证据日 |
| `hypothesis` | 一句话工作假设：增长或利润由哪一两个驱动决定。`thesis.summary` 由它派生 | 形容词、价位、时机 |
| `drivers`（1–3） | `name`；`mechanism` = 这个驱动怎么变成收入或利润；`evidence` = 事实 + 出处；`kpi` = 下一次披露里读得到的指标 | 没有 KPI 的「长期看好」 |
| `bear_case` | `claim` = 空头的**最强**版本；`rebuttal` = 反驳与出处；`falsifier` = 可观测的证伪条件，出现即 `thesis.status=broken` | 稻草人空头；把证伪条件写进 `invalid_if` |
| `valuation` | 当前价格**要求**什么（`agent.valuation` 的要求 CAGR / 目标利润率，带 as_of、标「诊断口径」），与驱动是否相容；可加透明的自上而下校验（份额 × 市场规模） | 公允价当目标价；「低估所以买」 |
| `catalyst` | 下一个检验驱动的事件：`event`；`date` = 事件日（财报取 `trade/market/earnings_calendar.json` 的 `pub_day`）；`baseline` = 上次指引 / 长期模型 / 一致预期；`watch` 1–5 条 = 这次看什么（来自 KPI 与证伪条件） | 事件发生后才补看点 |
| `sources` | 出处：`company_profile` 日期、情报归档文件名、财报期、用户 提供的材料（提供者 + 日期） | 无出处的数字 |
| `review`（可选） | 上一版 `catalyst` 到事件日后必带：`event` / `date` 沿用上一版；`outcomes` 逐条 `{watch, result}`，`watch` 逐字抄上一版 | 删掉拿不到结果的看点 |

任何槽证据不足写「待核实」（可带原因），不省略、不编造。`catalyst.date` 未知同样写「待核实」。

## 证据（只读）

卡片 `agent.company_profile`（券商网关 财报机器区 + 商业模式 / 融资人工区）、`agent.valuation` 镜像（as_of 早于最新财报期 = 过期，写「待核实」）、`trade/intelligence/reports/` 归档、`trade/market/earnings_calendar.json`、用户 在对话里提供的研报。**不自行 WebSearch / X**，情报采集归 `tasks/intelligence/`。同一来源的几个数字不是互相独立的佐证。

## 什么时候写

- **新卡**：画卡时一并写。
- **财报或公司重大事件之后**：必须重写并带 `review`。事件日到了但结果还拿不到，本次就别传 frame（价位带照常写）。
- **旧散文卡被重画价位带**：一并迁入（用户 2026-09-15：存量卡不立即迁，重画时逐张替换）。旧 thesis 里仍成立的基本面论点放进对应槽；价位、真空、量腿之类的结构注记不迁（它们属于 `anchor_basis` / `reasoning_summary`）；旧 `invalid_if` / `key_risk` 里的基本面证伪条件与风险移进 `bear_case`，`invalid_if` 只留价格结构。迁移不是重新研究：没有新事实时 `as_of` 取旧论点所依据事实的日期。
- **卡上已有 frame 的纯结构重画**：不传，frame 原样保留。

## thesis.status

`falsifier` 出现 → `broken`；驱动 KPI 恶化但未证伪 → `weakening`；上一版关键看点结果「待核实」→ `needs_reverification`；其余 `intact`。status 仍只是 [FYI] 播报，不进授权。

## 机器强制

- 形状：槽齐全且不夹带；文本非空；`drivers` 1–3；`watch` 1–5 且不重复；日期 YYYY-MM-DD（`catalyst.date` 可「待核实」）。
- `thesis.summary` = `hypothesis`：`--thesis` 与 `--thesis-frame` 同传拒绝；卡上已有 frame 后 `--thesis` 拒绝。
- `as_of` 不晚于今天（ET）；新 `catalyst.date` 不早于今天。
- 上一版 `catalyst.date` ≤ 今天 → 新 frame 须带 `review` 且覆盖上一版每条 `watch`，否则整次写入拒绝。与 2026-09-12 拒擦 stop 同形：预注册看点静默消失比写得松更危险。

## 写法

`update_position_zones.py --thesis-frame '<JSON>'`（受控提交里是同名参数，值为一个 JSON 字符串）。示意（虚构公司，数字仅作演示）：

```json
{"as_of": "2026-08-20",
 "hypothesis": "EPS 由云业务需求与 AI 订阅席位两条腿驱动",
 "drivers": [
  {"name": "云业务", "mechanism": "AI 负载上云 → 云收入与合同积压", "evidence": "上季云收入 +40%，合同积压环比 +8%（财报）", "kpi": "云业务增速；合同积压单季净增"},
  {"name": "AI 订阅", "mechanism": "付费席位 → 办公套件 ARPU", "evidence": "付费席位环比 +50%（管理层披露）", "kpi": "AI 付费席位"}],
 "bear_case": {"claim": "capex 下修被读成减支，PE 修复有一部分建立在误读上", "rebuttal": "口径变更（租赁分类 / 使用年限）不是减支，下一财年 capex 同比增（管理层表述）", "falsifier": "云业务增速 < 35% 或合同积压单季净增转负"},
 "valuation": "诊断口径：要求年 2 起 CAGR +13%，低于云业务增速，相容；估值层不发布公允价",
 "catalyst": {"event": "下一季财报", "date": "待核实", "baseline": "待核实", "watch": ["云业务增速是否 ≥ 35%", "合同积压单季净增", "capex 口径"]},
 "sources": ["公司财报", "管理层电话会", "估值层诊断"]}
```

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[Business_Quality_Moats|商业模式与护城河]]、[[Valuation_Framework|估值体系]]
