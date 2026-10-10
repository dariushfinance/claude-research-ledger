# Claude Research Ledger

Append-only, public ledger of two **simulated** paper-trading bots that act on a daily, rule-based equity research brief written by Claude. Displayed at [quantfoli.com/research](https://quantfoli.com/research).

**Paper trading only. No real money, no orders, no investment advice. Past performance is no indication of future results.**

## Rules
- **Claude Buy Zone (KZ):** buys every name the daily brief rates BUY ZONE (Top 3 class and expected value ≥ 5 %) that it does not already hold.
- **Claude Best-of-Day (BOD):** buys exactly one name per day: the highest-scoring Top 3 / Secondary name not already held (else the next best). Never Speculative or Avoid names.
- Each buy is 50 USD, fractional shares, buy & hold, no exits. Each portfolio has 10,000 USD virtual capital.
- Entry = **opening price on the brief date** (the brief is published before the US open, so no look-ahead).
- **Matched benchmark:** every fill buys 50 USD of SPY at the same open.
- Track record starts 2026-10-07. Days 07.–09.10.2026 were backfilled from the published briefs using the same rules.

## Files
- `data/paper-trades.csv`: events (`DECISION`, `FILL`, `NO_TRADE`), unique per (`trade_id`, `event`). A `DECISION` without a `FILL` is pending.
- `data/paper-snapshots.csv`: daily valuation per bot.

Rows are never edited or deleted. Corrections are made by appending new rows; the Git history is the audit trail.
