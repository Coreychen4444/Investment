# Investment Knowledge Base

[中文版本](README_CN.md)

A principle-driven investment framework built with [Obsidian](https://obsidian.md/), plus the sanitized mirror of a real, running US-equity trading system. The foundation is the stock price formula:

$$P = EPS \times PE$$

- **EPS (Earnings Per Share)**: Business fundamentals and intrinsic growth (micro perspective)
- **PE (Price/Earnings)**: Market expectations and liquidity (macro + sentiment)

The trading side is built on a different premise: **entry authorization comes from price structure only; decisions are probabilistic; exits are mechanical.** Fundamentals, macro and valuation inform the prior — they never grant a trade.

> **Motto** — *Better to miss a trade than to lose your rhythm; the trader who loses rhythm ends up as the market's prey.*

## Structure

```
00_Index/            → Dashboard and navigation
10_Core_Logic/       → First principles: Davis Cycle, EPS, PE, earnings surprise
20_Macro/            → Fed policy, treasury yields, monetary history, gold
30_Micro_Analysis/   → Moats, Porter's Five Forces, financial health, tech metrics
40_Valuation/        → PE Band, DCF, relative valuation, metric selection
50_Psychology_Risk/  → Sentiment, position sizing, stop-loss, exit strategy
60_Strategy/         → Sector thesis and annual investment plan (example)
us-stocks/           → The live trading system (sanitized mirror)
  ├─ system-overview.md   one-page map of the whole system
  ├─ strategy/            methodology canonical (+ options/, directives/, rules/)
  ├─ methods/             macro & fundamental methods (zero trading authority)
  ├─ research/            pre-registered backtests: what was adopted, what was rejected
  ├─ reviews/             anonymized real case studies
  └─ knowledge/           thematic essays
polymarket/          → Archived template workspace (not maintained)
```

## Highlights

- **System Overview** — division of labor (human picks names / system plans entries / system's clock owns exits / human executes), the decision stack, sizing, exit stack and daily cadence on one page
- **Two entry modes only** — 🔁 bottom-fishing on seller exhaustion (stop = the reversal low) and 🚀 trend confirmation via a gate-transition state machine; everything else is off the table
- **Cards as state machines** — every name carries zones, a binary regime cap {0, 40% NAV} and the next gate; gates are *discovered* where the market repeatedly made decisions, not picked
- **Six-arm sizing** — regime cap, gate risk budget (2.5 / 4 / 5% NAV), at-risk cap, shock budget, gross cap and buying power; the tightest arm wins and is always reported
- **Mechanical exit stack, no overrides** — card stop, profit ratchet, chandelier trend line, option ladder; "watch one more day" was tried three times, paid every time, and abolished
- **Research verdicts** — pre-registered, often out-of-sample tests; many intuitive ideas (key-level breakouts, extra bottom "evidence", tighter volume thresholds) were rejected by the data
- **Post-trade reviews** — real, anonymized cases; process is graded separately from outcome (wrong process + good outcome = D, the most dangerous grade)

## How to Use

1. Open in [Obsidian](https://obsidian.md/) for the full experience (bidirectional links, Mermaid diagrams, LaTeX)
2. Start at `00_Index/Home.md` for the knowledge framework, or `us-stocks/system-overview.md` for the trading system
3. Use `us-stocks/holdings/current/_EXAMPLE_.md` as a template for your own position cards
4. Use `us-stocks/strategy/trading-rules.md` (master index) as your pre-trade reference

## Anonymization

Tickers are replaced with `STOCK_*` placeholders, account-level figures are removed or expressed as ratios, and company-identifying catalysts are generalized. Names such as `quant/...` refer to code enforcement points in the author's private repository — they are kept to show where a rule is enforced ("a rule that isn't in code doesn't exist").

## Philosophy

> Process over outcome. Discipline compounds. Build the system, fight the randomness.

## License

[CC BY-SA 4.0](LICENSE) — share and adapt with attribution.

## Disclaimer

This is a personal learning framework, not financial advice. All example positions, prices, and reviews are anonymized. Do your own research and consult qualified professionals before making investment decisions.
