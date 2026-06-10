---
tags:
  - trading
  - options
  - framework
aliases:
  - Dealer Gamma Positioning
  - 做市商Gamma定位
  - Gamma结构与短期路径
---

# Dealer Gamma Positioning — 期权结构如何影响正股短期路径

> Applied when: 解读单日/短期异动路径(pin / squeeze / air-pocket)、OPEX 周、key strike 邻近、判断"破位 vs 机械"时。

## 定位(先划边界,最重要)
- 本文是**解读短期路径的 lens,不是 sizing / entry trigger**。基本面 + zones / Bayesian / Kelly 决定 size;gamma 结构只解释"为什么价格在某些位被压、被吸、突破加速、或瀑布"。
- 一句话支柱:**基本面决定大方向,期权结构决定短期路径**(= `../endogenous-market-model.md` "短期波动=结构" 的微观机制)。
- 本文是宏观/flow 异动框架的**"正 gamma 半"**(那里已建"负 gamma 放大器 / 机械踩踏半")。

## 一、机制(30 秒)
做市商卖期权给你 → 不想赌方向 → delta hedge。**Gamma = delta 变化的速度 = 对冲调整的速度与方向。**

| 环境 | dealer | 对冲行为 | 市场效果 |
|---|---|---|---|
| **正 gamma** | long gamma | 涨了卖 / 跌了买 | **减震器**:压波动、均值回归、易 pin 在 strike |
| **负 gamma** | short gamma | 涨了买 / 跌了卖 | **放大器**:放波动、趋势加速、squeeze / 瀑布 |

## 二、四个结构位
- **Call wall**:上方 call OI / gamma 集中的 strike。未破 = **阻力**(dealer 卖正股压制 → pinning);**强势突破 + dealer short gamma + call 续买 → 变 squeeze 燃料**。
- **Put wall**:下方 put OI / gamma 集中的 strike。守住 = **支撑**(dealer 买正股托);**跌破 + 负 gamma / 恐慌 → air-pocket 瀑布**。
- **Gamma flip**:dealer gamma 由正转负的价格带。上方 = 减震器模式,下方 = 加速器模式。
- **OPEX**:到期前 gamma 约束最强(易 pin);到期后约束消失,方向易释放 →「OPEX 前压住,后突破 / 失守」。

## 三、数据现实(硬约束 — 别造假精度)
- **多数散户无 GEX / dealer-gamma 数据源**(SpotGamma / Menthor 类,付费)。所以 wall / flip 是**概念,不是精确点位**。
- **唯一普遍可得 = 券商每 strike OI**(粗代理):
  - 粗略 call wall ≈ 现价**上方最大 call OI** 的 strike
  - 粗略 put wall ≈ 现价**下方最大 put OI** 的 strike
  - **看不到 dealer 方向 → 算不出真 GEX / 真 flip**。flip 只能**定性**(见第四节)。
- **看 wall 要看最近周 / 月链**,不是你的长期 LEAPS 到期月 —— 长期 LEAPS 合约几乎**不贡献**近端 pin 结构,真正 pin / squeeze 的是前周 / 前月链。
- OI 粗代理的限制(= 第六节误区 3/4):OI 大 ≠ 一定有效(可能已被 spread 对冲 / 买卖开仓方向不明 / DTE 太远 / 离现价太远);GEX 是模型估算,非真相。

## 四、单票动量名的结构偏向
单票高动量名(尤其散户/机构狂买 call 的成长股)→ dealer short call → **结构性偏负 gamma / 放大器**。这解释这类票的暴力波动,也是为什么 **"机械下杀 ≠ thesis 坏"** 对它们尤其成立。对照:大盘指数更常处正 gamma → 更稳、更易均值回归。

## 五、LEAPS 系统用法(只取有用的薄片)
1. **路径 ≠ thesis**:LEAPS 标的上方有大 call wall 被短期压制 / 跌破 put wall 瀑布 / 负 gamma 下破均线 —— **先归因结构,别当 thesis 坏**(接 flow 异动框架 + "MA 破位 caveat":负 gamma 破均线 ≠ 技术破位 + Bayesian 低信息量,见 `../bayesian-decision-model.md`)。
2. **突破 call wall(基本面没坏)= 路径顺风**:dealer 对冲买盘助推,对轻度 OTM LEAPS 有利 —— **温和确认信号,不是加仓 trigger**。
3. **OPEX 周对持仓名提高警惕**:pin 或释放都可能。知道这周是 OPEX,异动先打个结构问号再下结论。
4. **绝不放宽 size**:同 flow 规则,结构只影响参与 / 解读,**不放宽仓位大小**(size 仍由阶段 / 衰竭信号定)。

## 六、四个误区(别忘)
1. **call wall 一定阻力** → 不一定;突破后是燃料(看 dealer 正负 + 是否强突 + call 续买)。
2. **put wall 一定支撑** → 守住才是;跌破 = 加速点。
3. **OI 大一定有用** → 还要看 dealer short gamma 真伪 / 买卖开仓 / 是否已被 spread 对冲 / DTE / 离现价多远。
4. **gamma 数据 = 真相** → 是模型估算,辅助工具,不单独做交易依据。

## 相关文件
- `../endogenous-market-model.md` — 内生市场论(短期路径=结构 / VIX=变化速度 / 拥挤);本文是其 order-flow 微观机制
- `../bayesian-decision-model.md` — 机械移动 = 低信息量,不动后验
- `options-strategy-framework.md` — Greeks / 对冲 / Protective Put

---
> 📍 **Navigation**
> 上级：[[us-stocks/Home|US Stocks Hub]]
> 相关：[[endogenous-market-model|内生市场论]]、[[options-strategy-framework|期权交易框架]]、[[bayesian-decision-model|Bayesian决策模型]]
