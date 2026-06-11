---
tags:
  - trading
  - framework
  - psychology
aliases:
  - Sell-fly vs Rebalance
  - 卖飞与再平衡
  - 卖飞判别框架
---

# Sell飞 vs Rebalance Framing（卖飞与再平衡的判别框架）

> Canonical（2026-06-11 自行为触发层迁入 strategy）。

Applied when: 评估"卖在 X，现在涨到 Y，少赚 Z%"的场景。

## 核心原则
**Sell飞 ≠ "卖了之后股票还在涨"**。资金不是真空，它去了别处。**真正的判断标准是"redirected capital 跑赢还是跑输被卖标的的同期涨幅"**。

## 框架
```
卖 ticker A @ $X → 同期 ticker A 涨到 $Y → "少赚" $(Y-X) × qty

但实际：
卖出获得的现金 → 去了哪里？
  ├─ A) 加仓 ticker B → 比较 B 同期涨幅 vs A 同期涨幅
  │     ├─ B 涨幅 ≥ A 涨幅 → 不是 sell飞，是成功 rebalance
  │     └─ B 涨幅 < A 涨幅 → 是机会成本损失（但仍要看仓位风险调整）
  ├─ B) 现金躺账户 → **真正的 sell飞**（机会成本 = 全额少赚）
  └─ C) 部分加仓 + 部分现金 → 混合，按比例算
```

## 错误的 framing（避免）
- ❌ "如果当时不卖现在多赚 $X" — 这是 hindsight magic，假设资金真空
- ❌ "我每次都卖飞" — 通常是 framing 错误，没追踪 redirected 资金的实际去向
- ❌ 用 sell飞情绪做下次决定 — 容易导致"不敢减仓 / 拿到 round-trip"

## 正确的 framing
- ✓ 卖出后跟踪 redirected capital 的实际去向 + 表现
- ✓ Rebalance 跑赢同期 = success，跑输 = 教训但不是 sell飞
- ✓ 现金真闲置 = sell飞（罕见情况，多数是 rebalance）

## 源案例
**2026-04-09 STOCK_N 高位卖出**:
- 误读："卖在高位 X，之后涨到 X+28%，少赚一截"
- 实际：套现资金 → 加仓 STOCK_Y + STOCK_Z → 同期分别涨 47% / 72%
- redirected 同期收益显著超过被卖标的同期涨幅
- **结论**：rebalance 跑赢，不是 sell飞，是成功 reallocation

**2026-06-01 STOCK_S 清仓（rotation 后次日 +34%）**:
- 资金当日 redirect 到 STOCK_D（次日 +7.6%）；catalyst（战略合作公告）除内幕外不可预测
- 真账 = redirected capital vs 被卖标的同期，不是孤立盯一只
- **结论**：process A 级 rebalance；+34% 是 variance 非失误；后悔 ROI=0

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[post-trade-scoring|事后评分准则]]、[[Mindset_Risk_Control|交易心法与风控]]
