# Quarterly Strategy Review Template

手动触发，每季度末（3/6/9/12月）或宏观环境显著变化时执行。

## Steps

1. **读取全部策略文件**：
   - trading-rules.md
   - pre-trade-checklist.md
   - two-stage-entry-rules.md
   - bottom-confirmation-signals.md
   - event-risk-reduction-principle.md
   - sell-put-rules.md

2. **读取本季度交易日志**：
   - Your trade journal (maintain separately)
   - 统计本季度交易笔数、纪律评分分布（A/B/C/D）

3. **规则分类**：
   - **永恒纪律**：任何市场环境都适用（position sizing, stop rules, FOMO protection, confirmation > price）
   - **当前环境**：与当前市场状态绑定（Physical AI World 主题权重、specific zone 逻辑、beta 管理策略）

4. **环境适用性检查**（针对"当前环境"类规则）：
   - 市场 regime 是否已变？（Bull→Bear, 低波→高波, risk-on→risk-off）
   - 持仓组合是否已显著变化，使某些规则不再适用？
   - 是否有新的行为模式尚未被规则捕获？

5. **提议更新**：
   - 新增规则（来自本季度新教训）
   - 归档规则（环境已变、不再适用）
   - 锐化规则（表述模糊、执行时容易绕过）

6. **执行更新**（经确认后）：
   - 更新 trading-rules.md
   - 同步到 investment repo
   - Commit: `strategy: quarterly review Q[N] 2026`

## Output
- 变更摘要（新增/归档/锐化）
- 本季度纪律评分趋势
- 下季度重点关注的行为模式
