---
tags:
  - strategy
  - sector
aliases:
  - AI 板块
  - AI Logic
  - Physical AI
---

# AI 板块：从基建到应用 (Artificial Intelligence)

## 核心 Thesis

AI 是 2024-2030 最大的产业主题，但不是所有环节同时兑现。按确定性排序投资。

### Phase 1: 基础设施 — "卖铲子"阶段（确定性最高）

- **GPU/加速器**: NVIDIA 主导，AMD/自研芯片（Google TPU, Amazon Trainium）追赶
- **HBM（高带宽内存）**: AI 芯片的瓶颈从算力转向内存带宽，SK Hynix/Samsung/Micron 受益
- **网络**: 大规模集群需要高速互联（InfiniBand → Ethernet 演进），Broadcom/Arista/Cisco
- **电力**: 数据中心耗电指数级增长，详见 [[Energy_Logic|能源板块]]
- **核心特征**: 需求已被 hyperscaler capex 验证，收入可见度高，但估值也高

### Phase 1.5: 中间层 — 连接基础设施与应用

- **光互联/光通信**: 数据中心内部和之间的高速数据传输（Coherent/Lumentum/II-VI/Fabrinet）
- **冷却系统**: 高密度 GPU 集群的散热瓶颈（液冷 > 风冷），Vertiv/CoolIT/Schneider
- **数据传输/存储**: 训练数据的搬运和存储基础设施
- **核心特征**: 受益于 Phase 1 的规模扩张，但不直接暴露于 AI 模型性能风险

### Phase 2: 应用 — "淘金热"阶段（待验证，风险更高）

- **AI Agents**: 自主执行任务的智能体，替代部分人力工作流
- **垂直 SaaS**: 行业专用 AI 解决方案（医疗、法律、金融、制造）
- **Workflow Automation**: 企业内部流程的 AI 化改造
- **核心特征**: TAM 巨大但商业模式未验证，赢家难以预判，估值泡沫风险高
- **投资策略**: 等待收入确认信号（ARR 增速 > 50%，客户留存 > 90%），不预判赢家

### 资本开支周期 (Capex Cycle)

- 当前处于 capex 加速期：MSFT/GOOG/AMZN/META 2025 capex 指引合计 > $250B
- **关键转折**: capex 增速放缓 ≠ capex 下降，但市场会提前反应预期变化
- ROI 验证窗口：2026-2027 是关键——如果 AI 相关收入未能匹配 capex 增速，削减风险上升
- 历史参照：2000 年电信 capex 泡沫，过度建设→产能过剩→资产减值

### Phase 之间的投资节奏

```
时间     Phase 1 (基础设施)    Phase 1.5 (中间层)    Phase 2 (应用)
2024     ██████████ 高峰       ████████ 加速          ██ 萌芽
2025     ████████ 持续         ██████████ 高峰        ████ 增长
2026     ██████ 见顶?          ████████ 持续          ██████ 验证
2027+    ████ 成熟/下行?       ██████ 成熟            ████████ 分化
```

- Phase 1 的高点可能先到（GPU 供给追上需求），此时 Phase 1.5 和 Phase 2 仍在加速
- **不要同时重仓三个 Phase** — 根据各自节奏分配权重
- 每季度财报后重新评估各 Phase 所处位置

---

## Bull Case / Bear Case

### Bull Case (AI capex 持续加速)

- **Hyperscaler 资本开支**: 2025-2027 年复合增速 30-40%，训练+推理双引擎驱动
- **企业渗透率仍低**: AI 在企业工作流中的渗透率 < 10%，大量增量空间
- **Scaling Laws 仍有效**: 更大模型 + 更多数据 + 更多算力 = 更强能力，尚未触顶
- **推理需求爆发**: 从训练为主转向推理为主，推理算力需求可能 10x 于训练
- **新应用不断涌现**: Agents、多模态、具身智能（Physical AI）打开新 TAM

### Bear Case (需警惕的风险)

- **Scaling Laws 放缓**: 数据/算力的边际收益递减，模型能力提升遇瓶颈
- **Capex ROI 不达预期**: Hyperscaler 发现 AI 投资回报不及预期，开始削减资本开支
- **监管风险**: AI 安全法规可能限制模型训练和部署
- **开源竞争**: 开源模型（Llama, DeepSeek）压缩闭源公司的定价权和护城河
- **集中度风险**: NVIDIA 一家独大→任何 NVIDIA 的负面都会拖累整个板块
- **估值泡沫**: AI 板块整体估值偏高，一旦增速不及预期，回调幅度可能 30-50%

### Thesis 失效信号 (Kill Switch)

以下任一信号出现，需立即 reassess 仓位（不是立刻卖，但要认真评估）：

- 连续两个季度 hyperscaler capex 指引下调（不是增速放缓，是绝对值下调）
- NVIDIA 数据中心收入 QoQ 首次下滑（不含季节性因素）
- 主要 AI 实验室公开承认 Scaling Laws 遇到根本性瓶颈
- 多家企业客户 AI 项目大规模缩减/暂停（不是个别案例）

---

## 关键观察指标 (Key Metrics to Track)

| 指标 | 数据来源 | 频率 | 意义 |
|------|----------|------|------|
| Hyperscaler capex 指引 | MSFT/GOOG/AMZN/META 财报 | 每季度 | Phase 1 需求的直接验证 |
| GPU 交付周期 | 行业渠道/NVDA 财报 | 每季度 | 供需紧张程度 |
| AI 相关收入占比 | 各公司 AI revenue disclosure | 每季度 | ROI 验证进度 |
| 数据中心用电增速 | 公用事业公司报告/EIA | 每季度 | 部署规模的物理验证 |
| HBM 出货量/价格 | SK Hynix/Micron 财报 | 每季度 | 内存瓶颈缓解程度 |
| 光模块出货量 | Coherent/Lumentum 财报 | 每季度 | 集群互联规模 |
| 企业 AI 采用率调查 | Gartner/McKinsey/各公司 | 年度 | Phase 2 落地进度 |

**领先指标 vs 滞后指标**:
- 领先: GPU 交付周期缩短（需求可能见顶）、capex 指引措辞变化、AI 初创融资额变化
- 同步: 光模块出货量（反映当前部署进度）、数据中心电力申请
- 滞后: AI 收入占比（确认已发生的趋势）、企业 AI 采用率调查

**跟踪节奏**: 每季度财报季（1/4/7/10月）集中更新上述指标，非财报期关注事件驱动（产品发布、政策变化、技术突破）

---

## 估值框架 (Valuation Framework)

### 硬件/基础设施 (Phase 1)

- **核心指标**: PEG < 1.5（Forward PE / EPS 增速）
- **辅助**: EV/EBITDA 相对历史区间，gross margin 趋势
- **注意**: 硬件公司的周期性——peak earnings 时 PE 看起来便宜但可能是陷阱
- **参考**: NVDA 2024 forward PE ~30x @ 100%+ 增速 → PEG < 0.3（极度便宜或增速不可持续）

### 光通信/中间层 (Phase 1.5)

- **核心指标**: EV/EBITDA + 产能利用率
- **辅助**: 订单积压 (backlog) 增速，客户集中度
- **注意**: 光通信周期性强，产能扩张后可能面临价格战
- **关键**: 区分 AI 相关收入 vs 传统电信收入的占比变化

### 软件/应用 (Phase 2)

- **核心指标**: Rule of 40（Revenue Growth % + FCF Margin % > 40）
- **辅助**: ARR 增速，Net Dollar Retention > 120%，Gross Margin > 70%
- **注意**: 大部分 AI 应用公司仍在 pre-profit 阶段，用 EV/ARR 估值
- **警惕**: "AI wrapper" 公司护城河低，可能被平台方（MSFT/GOOG）整合

---

## 风险评分与仓位建议

```
风险项                | 级别    | 影响
Scaling Laws 放缓    | 高      | 整个 thesis 动摇，Phase 1+2 同时受损
Capex ROI 不达预期   | 中-高   | Phase 1 营收见顶，Phase 2 延后
竞争/开源替代         | 中      | 压缩应用层定价权，但利好基础设施层
能源约束              | 低-中   | 延缓部署速度，但利好能源板块
单一客户集中          | 中      | NVDA 供应商链条风险高度相关
地缘政治/出口管制     | 中      | 影响中国相关需求和供应链
```

### 仓位建议

| 阶段 | 角色 | 仓位范围 | 理由 |
|------|------|----------|------|
| Phase 1 基础设施 | Core | 5-10% per name | 需求已验证，确定性高，但估值需等回调 |
| Phase 1.5 中间层 | Core/Satellite | 3-5% per name | 受益明确但周期性强，择时重要 |
| Phase 2 应用 | Satellite/Probe | 2-3% per name | 不确定性高，轻仓试探，等收入确认 |

### 操作原则

- **不追高**: AI 板块波动大，回调 20-30% 是常态，耐心等 accumulation zone
- **分阶段建仓**: 用两档建仓法（详见 [[two-stage-entry-rules]]），不一次性重仓
- **Phase 1 → Phase 2 轮动**: 当 capex 增速放缓信号出现，Phase 1 开始减仓，观察 Phase 2 赢家浮现
- **跟踪 thesis**: 每季度财报后更新判断，thesis 动摇时果断减仓
- **AI 板块总仓位上限**: 不超过组合的 40%，单一 Phase 不超过 20%
- **相关性管理**: Phase 1 标的之间高度相关（同涨同跌），不要误以为"分散"了

---

## 与其他板块的联动

- **能源板块 [[Energy_Logic]]**: AI capex ↑ → 数据中心用电 ↑ → 能源板块受益。两者正相关，但能源的下行风险更小
- **航天板块 [[Aerospace_Logic]]**: AI + 卫星数据 = 对地观测分析；AI + 国防 = 自主系统。间接联动
- **宏观**: 利率上行 → 高估值 AI 股承压；衰退 → 企业 AI 采购延后但 hyperscaler capex 可能不减

---

## 当前持仓与 Thesis 映射

> 持仓详情见 `portfolio_latest.csv`，此处仅做 thesis 分类参考。
> 每次复盘时更新此映射。

| Thesis 阶段 | 关注方向 | 确定性 |
|-------------|----------|--------|
| Phase 1 | GPU, HBM, AI 芯片 | 高 |
| Phase 1.5 | 光模块, 冷却, 网络 | 中-高 |
| Phase 2 | AI SaaS, Agents | 低-中 |

---
> 📍 **Navigation**
> 上级：[[2026_Investment_Plan|年度投资计划]]
> 相关：[[Energy_Logic|能源板块]]、[[Tech_Metrics|科技指标]]
