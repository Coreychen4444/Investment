---
tags:
  - macro
  - concept
aliases:
  - 10年期美债
  - US Treasury
  - 无风险利率
---

# 10年期美债：全球资产定价之锚

## 全球资产的"定海神针" (The Anchor)
10年期美国国债收益率 (10-Year Treasury Yield) 被视为全球金融市场的 **无风险利率 (Risk-Free Rate, $R_f$)**。因为美国政府被认为不会违约，它是所有其他资产（股票、公司债、房地产）定价的基准。任何风险资产的回报率都必须在无风险利率的基础上加上 **风险溢价 (Risk Premium)**。

## DCF模型与估值 (The DCF Connection)
美债收益率通过 **现金流折现模型 (DCF)** 直接影响资产价格。公式如下：

$$
Valuation = \sum_{n=1}^{\infty} \frac{CF_n}{(1 + R_f + RiskPremium)^n}
$$

*   $CF_n$: 未来第 $n$ 年的预期现金流。
*   $R_f$: 无风险利率 (10年期美债收益率)。
*   **数学逻辑**: $R_f$ 位于分母。当美债收益率 ($R_f$) **上升**，分母变大，整个分数的数值（即估值）就会 **下降**。

## 估值杀手 (Valuation Killer)
这就是市场上常说的"杀估值"。
*   即使一家公司的基本面（分子端的 $CF_n$）没有变化，仅仅因为美联储加息导致 $R_f$ 上升，其理论估值也会大幅缩水。
*   对于**高成长股**而言，这种打击尤为致命，因为它们的大部分现金流都在遥远的未来，分母的复利效应会放大利率上升的负面影响。

## 久期概念 (Duration)
这就引入了债券市场中的 **久期 (Duration)** 概念，同样适用于股票：
*   **长久期资产 (Long Duration)**: 类似于科技成长股 (如 Tesla, Nvidia)。投资者买的是未来的增长，现金流主要集中在远期。利率敏感度极高。利率涨，跌幅大。
*   **短久期资产 (Short Duration)**: 类似于成熟价值股 (如 Coca-Cola, Walmart)。公司当下就有强劲的现金流和分红。利率敏感度较低。

## 重力隐喻 (The Gravity Metaphor)
沃伦·巴菲特曾做过一个精彩的比喻：
> **"利率之于资产价格，就像重力之于苹果。"** (Interest rates are to asset prices what gravity is to the apple.)

*   **低利率**: 重力微弱，资产价格可以漂浮在很高的位置（高估值泡沫）。
*   **高利率**: 重力增强，将所有资产价格猛烈地拉回地面。

---
> 📍 **Navigation**
> 上级：[[Fed_and_Liquidity|宏观视角：美联储与流动性]]
> 相关：[[DCF_Logic|DCF 估值逻辑]]、[[PE_Market_Expectation|PE 市场预期]]、[[Fed_Mechanics|美联储机制]]
