# Options Trade Journal Template

期权专属复盘模板，正股使用 `../trade_journal.md` 模板。

每个期权 entry 记录到 `trade/us_stock/holding/trade_history/options_journal.md`，格式如下：

---

## YYYY-MM-DD — TICKER YYMMDD STRIKE C/P (qty)

### 入场快照
- **下单时间**：YYYY-MM-DD HH:MM SGT (= ET HH:MM)
- **正股价**：$___
- **期权价**：$___（mid / 实际成交）
- **DTE**：___ 天
- **Delta**：___（绝对值）
- **Theta**：$___ / day（每张合约美元损耗）
- **Vega**：$___ / 1% IV
- **IV**：___%（IV percentile / rank: ___%）
- **Breakeven**：$___（正股需到这个位）
- **Max loss**：$___（买方 = 权利金；卖方 = strike × 100 - 权利金）
- **Max gain**：$___（spread 类）

### 决策逻辑
- **分类**：Confirmed / Probe / Early-risky
- **仓位层级**：base / core / trading（期权 99% 是 trading 仓）
- **Catalyst**：___（earnings? FOMC? 板块 rotation? 技术突破?）
- **Why options 而不是正股**：（杠杆？资金效率？事件 binary？时间窗口?）
- **Why this strike**：（ATM vs OTM 选择理由）
- **Why this expiry**：（避 earnings? 给 thesis 多少时间?）
- **是否过 pre-trade-checklist**：是 / 否（哪些没过）

### 退出条件（必填）
- **目标**：正股达 $___ / 期权达 $___ → 平仓
- **止损**：正股跌破 $___ / 期权损失 $___ → 平仓
- **时间止损**：DTE < ___ 平仓（避 theta 加速 + gamma 风险）
- **Thesis 失效**：___（具体事件 / 价格条件）

### 退出记录
- **平仓时间**：YYYY-MM-DD HH:MM SGT
- **正股价**：$___
- **期权价**：$___
- **持有天数**：___ 天
- **PnL**：$___（USD 绝对值）
- **触发哪个退出条件**：target / stop / time / thesis
- **如果未按计划退出，原因**：___

### 复盘（事后填）
- **判断对错**：
  - 方向对错？___
  - 时间对错？___
  - IV 对错？___（事件后 IV crush 是否吃掉收益？）
- **Greeks 教训**：（哪个 Greek 的影响超出预期？）
- **可改进点**：___
- **规则更新**（若适用）：写入 `greeks-discipline.md` 或 `trading-rules.md` 第 N 条

---

## 引用（路径相对于 `trade/us_stock/holding/trade_history/options_journal.md`）
- Greeks 操作规则：`../../strategy/options/greeks-discipline.md`
- 期权框架：`../../strategy/options/options-strategy-framework.md`
- 卖 put 纪律：`../../strategy/options/sell-put-rules.md`
