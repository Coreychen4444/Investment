---
tags:
  - trading
  - review
aliases:
  - DRAM Reactive Add Review
  - Ladder Discipline Corrections
---

# Review — DRAM Reactive Add & Ladder Discipline Corrections

## Summary
2026-06-05 美股 -4% 板块杀跌中,反射性加仓 DRAM Jan 2027 $60C × 2 @ $15.50(naked,IV 88%)。恰好落在预设 zone2,但属**运气位置,非纪律执行 → process D**。复盘从这笔逆推出 ladder 本质 + 四处修正,固化为「不确定性下的加仓执行系统」。

DRAM = Roundhill Memory ETF,SK Hynix HBM re-rating proxy(HXSCL OTC ADR 0 成交,ETF 为唯一可执行 vehicle)。

## Timeline
- **6/1**: probe 2× Dec26 $70C @ $15(计划内 ladder 首档)
- **6/3** 顶 $69.71 → **6/4** $65.70(-5.7%)→ **6/5** open $60.82 → low $55.38 → close $55.79(**-15.1%**,量 77.7M = 2.2x 均量 = capitulation)
- **6/5**: 反射性加 2× Jan27 $60C @ $15.50(**未参考 ladder budget**,IV 88%)
- **宏观**: Nasdaq -4.18%、VIX +40% 到 21.5、强 NFP(+17.2万)+ AVGO 未上调 AI 指引 → 半导体板块连杀 = **macro/sector 驱动,非 DRAM 个股 fundamental 破裂**

## What went wrong (root mistake)
- **用错尺子自评**:"是否到底" + "胜率" 都是 outcome / 不可知维度;正确是 **process 维度**。"是否到底"是范畴错误——永远不可能知道,ladder 存在就是为了不需要知道。
- **反射性进场**(看红就点),跳过 budget + 宏观验证 → 即使落 zone,process = D(不是 A;碰巧落 zone 是运气)。
- **IV 88% 买 naked long premium** = 在 vol 尖峰买 vega,vehicle 错(该用 spread / 等 IV 降)。

## What went right
- thesis intact(macro 杀跌非个股坏),stop $48 明确,没乱平。
- vehicle **strike/DTE 选择**合理(标的跌后降 strike $70→$60 拿回 delta + 延 DTE Dec26→Jan27)。
- 事后**诚实复盘**(承认 reactive,不粉饰),并自发逆推出 ladder 哲学。

## Rules extracted — 加仓体系四修正
1. **倒金字塔陷阱**:"价格越低加仓越多" = 倒金字塔 / Martingale = 失败者陷阱(最重的钱压在最不确定价位)。真金字塔 size **递减**;size 跟**证据**(衰竭信号)走,不跟价格走。
2. **ladder ≠ 降成本**:只有抄底降均价;追涨加仓**抬高**均价(且那是对的)。ladder 保证的是「先参与 + 优于单点 all-in + 双向封顶后悔」。
3. **进场前定分母**:先定总目标占比 / ladder 预算。没有分母,"加仓"无意义。纪律不是"有计划",是"**点之前先看计划**"。
4. **反射加仓 = D**:看红反射性加仓,即使恰落预设 zone,process 仍是 D 不是 A。

## System produced
「**不确定性下的加仓执行系统**」: 买 sized 档 ≠ 赌底;**Q-A 资格 / Q-B·C 节奏 / Q-D 工具** 三旋钮分离(不矛盾);指标 = 证据非预言。详见 [[uncertainty-execution-system]](`../strategy/uncertainty-execution-system.md`),工具 `scripts/regime_score.py`。

### 如果 6/5 跑这套系统会输出什么
- **Q-A**: 标的 -15% + 同业 SMH -9.2/SOXX -10.4/MU -13.3 + VIX +40% → **MACRO PANIC → GO**(thesis intact)
- **Q-B**: RSI 52(未超卖)+ 收当日 7% 分位(低位)+ 连阴 2 → **半山腰续跌,非底**
- **Q-C**: 见底评分 **C** + 否决"放量破位(恐慌非承接)" → 别指望是底
- **Q-D**: IV **88%** → vol 贵 → **SPREAD / 等,别买 naked**
- **处方**: GO + 半山腰 + vol 贵 → 这一档 ≈ 剩余预算 15% ≈ **接近 0 张 / 一个 spread**,而非 2 张 naked

**系统没预测底,只告诉"这不是底 + 用错 vehicle",就足以把 size 做对。** 这就是"用量化逐步替代凭感觉"的实证。

## Cross-refs
- `../strategy/uncertainty-execution-system.md`(执行层主文档)
- `../strategy/bayesian-decision-model.md`(决策层)
- `../strategy/trading-rules.md`(Ladder discipline 规则 10-12)
