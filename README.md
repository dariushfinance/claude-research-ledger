# Claude Research Ledger

Append-only, public ledger of two **simulated** paper-trading bots that act on a daily, rule-based equity research brief written by Claude. Displayed at [quantfoli.com/research](https://quantfoli.com/research).

**Paper trading only. No real money, no orders, no investment advice. Past performance is no indication of future results.**

## Rules
- **Claude A+ Filter Setups (`KZ`):** buys every name that clears the brief's top filter — Top 3 class and expected value ≥ 5 % — and that it does not already hold.
- **Claude Best-of-Day (`BOD`):** buys exactly one name per day: the highest-scoring Top 3 / Secondary name not already held (else the next best). Never Speculative or Avoid names.
- Each buy is 50 USD, fractional shares, buy & hold, no exits. There is no starting capital and no cash position: the only money in the experiment is what has actually been committed, 50 USD at a time.
- Entry = **opening price on the brief date** (the brief is published before the US open, so no look-ahead).
- **Matched benchmark:** every fill buys 50 USD of SPY at the same open.
- Track record starts 2026-10-07. Days 07.–09.10.2026 were backfilled from the published briefs using the same rules.

## Files
- `data/paper-trades.csv`: events (`DECISION`, `FILL`, `NO_TRADE`), unique per (`trade_id`, `event`). A `DECISION` without a `FILL` is pending.
- `data/paper-snapshots.csv`: daily valuation per bot.

Rows are never edited or deleted. Corrections are made by appending new rows; the Git history is the audit trail.

## Two things that must not change

- **The `bot` codes `KZ` and `BOD`.** They are the key every stored row is
  filed under, in an append-only table that rejects updates. Renaming a code
  orphans every row already recorded and splits the track record in two. The
  display names above are free to change; the codes are not.
- **The `(trade_id, event)` pair.** It is the uniqueness key that makes
  re-importing the same file a no-op. Re-using a `trade_id` for a different
  trade silently hides the second one.
