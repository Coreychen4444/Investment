---
tags:
  - core-logic
  - concept
aliases:
  - PE
  - 市盈率
  - Price Earnings
---

# PE：市场情绪的温度计 (Market Expectation)

**定义**: PE (Price / Earnings) = 股价 / 每股收益。它代表了按照当前的盈利水平，你需要多少年才能回本。

## 核心比喻：房地产投资
- 想象一套房子价格 **10万 (Price)**。
- 每年租金净收入 **5000 (Earnings)**。
- 这里的 **PE = 20倍**。
- 如果利率下降，大家觉得5000的租金很值钱，愿意出 **15万** 买这套房，此时 **PE = 30倍**。房子还是那套房子，但市场给出的估值变了。

## PE 扩张与收缩的驱动力 (Drivers)
PE 的变化往往比 EPS 更剧烈，被称为“估值倍数的扩张与收缩”。

1. **增长率预期 (Growth Rate)**
   - 增长越快，市场愿意给的 PE 越高（为未来买单）。
   - 预期一旦落空，杀估值最惨烈。

2. **利率 (Interest Rates)**
   - 利率是资产价格的地心引力。
   - **美联储 (Fed)** 提高利率时，无风险收益率上升，资金流出股市，PE 受到压制（收缩）。

3. **市场情绪 (Sentiment)**
   - **贪婪 (Greed)**: 大众觉得股票只会涨，给予极高 PE。
   - **恐惧 (Fear)**: 大众恐慌抛售，即便好公司也只给极低 PE。

### 底层公式
这三个驱动力可以用一个公式统一理解：

$$PE = \frac{1}{r - g}$$

其中 $r$ = 折现率（受利率 + 风险偏好影响），$g$ = 增长率预期。

- 利率上升 → $r$ 增大 → PE 下降（驱动力 2）
- 增长预期提升 → $g$ 增大 → PE 上升（驱动力 1）
- 市场恐慌 → $r$ 中的风险溢价飙升 → PE 暴跌（驱动力 3）

> 详见 [[Undervaluation_First_Principles|低估的第一性原理]] 中的完整推导。

---
> 📍 **Navigation**
> 上级：[[Investment_First_Principles|投资第一性原理]]
> 相关：[[Fed_Mechanics|美联储机制]]、[[Ten_Year_Treasury|10年期美债]]、[[Market_Sentiment|市场情绪]]、[[Davis_Cycle|戴维斯双击]]
