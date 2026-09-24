---
tags:
  - polymarket
  - hub
aliases:
  - Polymarket Hub
---

# Polymarket Hub

> [!warning] 已归档（2026-09）
> 交易范围已收敛为美股正股与单腿 long call，不再有预测市场子系统。本目录只保留空框架与模板，不再维护；当前系统见 [[system-overview|系统总览]]。

## Purpose
Dedicated workspace for prediction market trading (e.g., esports, events).
Keep PM logic separate from US stocks logic.

## Sections
- `strategy/` — trading strategies, odds frameworks, iron rules
- `journal/` — trade journals (use template)
- `reviews/` — post-trade reviews (use template)
- `knowledge/` — domain knowledge (game strategy, team analysis, etc.)
- `templates/` — journal and review templates

## Templates
- [[trade-journal-template|Trade Journal Template]]
- [[review-template|Review Template]]

## Getting Started
1. Define your edge hypothesis in `strategy/`
2. Record each trade using the journal template
3. Review outcomes using the review template
4. Extract durable rules back into `strategy/`

---
> 📍 **Navigation**
> 上级：[[00_Index/Home|Dashboard]]
