# Risk & Capital Framework — 资金 / capacity / 集中度

> Canonical home for capital-assessment、cash% reasonableness、concentration stress-test。
> 关系：与 `.claude/rules/position-tiers.md`（三层仓位 + Concentration Override Protocol）互补 —— position-tiers 定义 cap 与 override 协议，**本文定义"如何计算 capacity / cash% / 满 ladder 暴露"的方法**。
> Memory 源（recall 层，已降级为指针）：`feedback_buying_power` / `feedback_cash_pct_framework` / `feedback_concentration_full_ladder_check`。

## 1. Buying Power vs Cash —— 评估加仓 capacity
评估"有没有空间加仓"时，用**真实购买力**，不是单看 `cash` 字段。

- Capacity = broker `power`（margin-aware 购买力）+ off-broker near-cash（第二券商余额；场外 redeemable 货币基金）。**不是 `cash` 一项。**（具体账户标识不入策略文档）
- `risk_status` LEVEL1–5 = margin 风险等级标签，**不是"不能交易"信号**。LEVEL5 仍可加仓，只是占用更多 margin headroom。
- **加仓的真实约束通常不是现金**，而是：(a) 单票集中度 cap，(b) zone / no-chase 纪律，(c) 财报前风险窗口。说"不加"时必须点名真实约束（集中度 19% / zone1 上沿 / earnings 8 天），不能用"没现金"搪塞。
- **源案例**：2026-04-30 我用 `cash` $X,XXX + LEVEL5 误判"加仓空间≈0"；实际 power $X,XXX + off-broker ~$X,XXX ≈ $X,XXX deployable。

## 2. Cash% Reasonableness —— 压力测试，不是 benchmark
用户 问"X% cash 合理吗 / 现金充裕想 deploy"时，**不给通用 benchmark**，跑结构压力测试。Cash% 不是绝对值，是结构性风险的函数。

**Cash% 随这些因素 scale**：
- Margin tier（LEVEL 越高 = 有效杠杆越高 = 需更多 cash buffer）
- Theme 集中度（≥50% 单一 theme = cash 是唯一分散）
- 单票集中度（任何 >25%，尤其 override active = 需 cash 吸收个股冲击）
- 事件窗密度（数未来 4–12 周 earnings/expiry/macro，越多越需 buffer）
- Active override 数（每个消耗 risk budget，cash 补偿）

**压力测试（非 benchmark）**：估单日事件波动（5% earnings move × 17% 仓 ≈ $670 on $25k）+ 板块 -10% 系统性（期权放大）；cash 应 cover ≥ 1 个此量级事件。cash < 预期 stress = 不足，即使 % 看着正常。

**用户 典型杠杆集中结构的档位**：≤15% unsafe / **20% = floor（刚够）** / 25–30% healthy / 30%+ defensive（预期 macro shock 时）。

**侦测伪装成 deploy 的 FOMO**："购买力丰裕想买点什么 / 迫切 deploy / 尽快参与" = FOMO 信号（无论实际 cash%）。Reframe："若本周 -10% 系统性事件砸下来，cash 能 cover drawdown + 给再入场弹药吗？" 答"勉强" → cash 是 floor 不是新仓 buffer。
- **源案例**：2026-05-01 STOCK_AM，20% cash 被误读为"充裕"，实际是 70% AI infra 集中 + LEVEL5 + 11 周 4 个事件窗下的 floor。

## 3. Concentration Stress-Test at Full Ladder —— 按满 ladder 算，不是初始 fill
设 multi-tier ladder 时，挂 GTC 前必须按"每档都 fill"场景算 **delta 等价暴露**，不是看初始 fill cost（会严重低估）。

**公式（每 tier 累计）**：
```
单票 delta 等价值 = 现货_qty + Σ(tier_n_qty × tier_n_Δ × 100)
单票 exposure %  = (delta 等价值 × current_price) / total_assets_USD
```

**4 步硬检查（挂 multi-tier GTC 前）**：
1. 算满 ladder fill 后单票 delta 等价值
2. 占账户 % = 上式 × current_price / total_assets
3. >25% → 触发 override 检查（`position-tiers.md` Concentration Override Protocol）
4. 已有 active override → 缩 ladder 直至无双 override 风险（红线：**最多 1 个 active override**），或等其他 override 释放

**红线**：
- ❌ 单票 delta 暴露 >50%：无论 conviction 多强不允许
- ❌ 满 ladder 触 25%+ 暴露 + 已有 active override：必须缩 ladder 或撤 GTC
- ❌ 用"反正三档 parabolic 不会 fill"做 sizing 借口：GTC 挂着就有责任

- **源案例**：STOCK_X 270115 C15 ladder 2026-05-04 —— 看似 $4k=14% 安全；满 ladder 17 张 + 100 股 = 970 股等价 × $13 = $XX,XXX = **45.3%** + 与 STOCK_Y 33% override 冲突（双 override 违规）→ 撤三档 + 二档减半。
