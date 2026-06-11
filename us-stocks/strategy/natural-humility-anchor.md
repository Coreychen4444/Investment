---
tags:
  - trading
  - framework
  - psychology
aliases:
  - Natural Humility Anchor
  - 自然谦逊锚
  - 深亏持仓规则
---

# Natural Humility Anchor（自然谦逊锚）

> Canonical（2026-06-11 自行为触发层迁入 strategy）。

Applied when: 评估是否要主动止损一个深亏 position（亏损 ≥ 30% from cost basis）。

## 核心原则
深亏 position 自然形成的 humility anchor 效应是 **side effect**，不是 entry justification。本规则定义"何时可以 NOT 主动卖深亏"，但**严禁用 anchor 价值为任何买入 / 加仓辩护**。

## 适用范围（必须全部满足）
- 已经在 base / core / trading 任一层的 position
- 实际亏损 ≥ **30%** from cost basis
- Underlying thesis 尚未被正式证伪
- 持有 ≥ 24 小时（防 day-trade 滥用）

## 默认动作
**可以不主动卖**。让它充当 humility anchor，提醒"我可以错"。

## 5% Cap (cost basis)
所有 anchor positions 的 **cost basis** 合计 ≤ portfolio total cost basis × 5%。
突破 cap → 从 conviction 最弱的 anchor 开始 trim 直到合规。

## 强制 override（必须卖，anchor 价值 forfeit）
1. **Thesis formally invalidated** — earnings miss / 关键指标 confirmed miss / 关键客户流失 / 技术面 confirmed 破位
2. **5% cap 突破** — trim 弱者直到合规
3. **Options 三档 DTE 处理**：
   - DTE > 60 OR LEAPS → 标准 anchor handling
   - DTE 21-60 + OTM + bid < $0.10 → force close（marginal psych value < friction）
   - DTE ≤ 21 + OTM → force close（gamma 加速区独立规则；复核窗口与 G-02 的 DTE<30 监测衔接，见 `options/greeks-discipline.md`）
   - DTE ≤ 7 任何状态 → force close
4. **Stocks**：无 time-based 强制；只看 thesis 和 5% cap
5. **Position 价值 → 0** — 留着的 marginal psych value < transaction friction

## 防 creep 红线
1. ❌ **不能 retro-label** — 不能因为某 position 跌深了"宣布"它是 anchor。Anchor 是 result（自然形成），不是 designation（事后宣布）
2. ❌ **不能用 anchor 价值 justify 任何买入 / 加仓**（含相关标的：anchor 某票 ≠ permission to buy 其杠杆 ETF）
3. ❌ **不豁免 Iron Rule #2A** — averaging down 限制照常生效
4. ❌ **不能 roll options 给 anchor "续命"** — 到期 expire 或 force close，不延期
5. ❌ **同时 ≥ 3 个 anchors = 警讯** — 提示账户风格在恶化（不是单纯 anchor 问题），需主动 reduce

## 与其他规则优先级
Iron Rule #2A、G-02、thesis invalidation 都是 **higher priority** than anchor framework。Anchor 不创建任何这些规则的例外。

## 源案例
**2026-04-30 STOCK_R 深 OTM call**：
- 持仓 10 张 @ cost basis $0.86 = $860
- Status: -66%, DTE 49, deep OTM，thesis intact
- 4-check：loss ≥ 30% ✓ / thesis intact ✓ / 持有 ≥ 24h ✓ / cost basis 在 5% cap 内
- 决策：符合 natural humility anchor，可 hold
- 后续：按 force-close 规则于 DTE 收窄后平仓 @ $0.08，realized -$780
- 禁止：用该 anchor 身份 justify 加仓同票或买对应杠杆 ETF

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[trading-rules|Trading Rules]]、[[greeks-discipline|Greeks 纪律]]、[[Mindset_Risk_Control|交易心法与风控]]
