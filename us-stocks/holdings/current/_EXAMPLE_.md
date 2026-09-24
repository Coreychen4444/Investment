---
tags:
  - trading
  - template
aliases:
  - Holding Template
  - 标的卡模板
---

# _EXAMPLE_ — 标的卡模板

> 这是**示例**，替换成你自己的标的。字段与实盘系统的「卡片」一致：价格结构层是唯一的授权来源，thesis 层只作 [FYI]。画法与 staleness 规则见 [[zone-maintenance|Zone 维护与 gate 画法]]。

## Snapshot
- Ticker：EXAMPLE　｜　主题：Example Optoelectronics（高 beta 光通信）
- 持仓：空仓 / 股数 @ 均价 / 期权腿（空仓才可能走 🔁 抄底；持仓票加仓归 cap + Iron Rule #2 v2）
- 卡片日期：`anchored_on` YYYY-MM-DD　｜　`valid_until` YYYY-MM-DD

## 结构（授权层）
| 字段 | 值 | 证据（anchor_basis） |
|---|---|---|
| z1 | lo – hi | 近期浅回踩低点群 / pivot 簇 |
| z2 | lo – hi | 上一档整理区或缺口下沿 |
| trim | lo – hi | 许可区不是指令（normal：ATH × 1.05–1.07） |
| no_chase | 价 | 压在第一堵拒绝墙之下，不放在真空带里 |
| stop | 价 ×N 日收盘 | 结构位；距现价应落在 [0.7N, 2N]（N = ATR20） |

单调性自检：stop < z1 下沿，z1 上沿 ≤ no_chase ≤ trim 下沿。

## Regime（状态机）
- **current_regime**：cap = 0 / 40% NAV —— 原因（结构未修复 = VETO）
- **next gate**：价 —— 确认口径 = 收盘 ×2 + RelVR ≥ 1.1；证据 = 多维收敛（pivot 簇 / 供给簇上沿 / HVN-LVN 边界 / 旧 ATH / 破位起点）
- **两种模式**：🔁 抄底信号？（止跌低、信号日、有效至）｜🚀 离 gate 多远？（N 数）

## Thesis 框架（判断层，零授权）
- **假设**：EPS 由哪几条腿驱动
- **驱动**：机制 → 证据 → KPI（每条都可证伪）
- **空头与反驳**：空头最强的一句话、你的反驳、什么数据出现就算你错了
- **估值交叉验证**：市场隐含的增长是否与驱动相容（不写目标价）
- **下一催化剂**：事件 / 日期 / 基线 / 看点 —— 到期必须对账

## 退出（系统的钟）
- 卡片 stop 收盘确认 = 当日清仓；利润棘轮 / 趋势死亡线 / 期权 gate 线同样当日执行，**无 override**（见 [[trend-exit-system|趋势退出系统]]）

## Danger signs
- 反复修复失败：站上 gate 又收回
- 同主题里相对强弱持续落后
- 事件日 sell-the-news 后没有真正收回
- 你开始为「再看一天」找理由 —— 这本身就是信号
