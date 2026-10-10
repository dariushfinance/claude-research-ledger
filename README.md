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
- `data/paper-snapshots.csv`: one valuation row per bot per trading day. See the
  contract below.

Rows are never edited or deleted. Corrections are made by appending new rows; the Git history is the audit trail.

## `paper-snapshots.csv` contract

One row per (`date`, `bot`) per US trading day, appended after the US close.
This file is the **only** source of a daily return series: without it there is no
equity curve, no drawdown, no volatility and no Sharpe, and a missing day cannot
be reconstructed later without pulling prices from somewhere else.

Unlike the trade events, snapshots are **derived, not ledger**. Re-publishing a
row for a date that already exists overwrites it, which is the intended way to
fix a bad valuation. The key is (`date`, `bot`).

| Column | Meaning |
|---|---|
| `date` | `YYYY-MM-DD`, the US trading day the valuation is as of. |
| `bot` | `KZ` or `BOD`. |
| `trades` | Number of fills held at the close of that day. |
| `invested` | Cumulative USD committed: 50 x `trades`. |
| `cash` | **Leave empty.** There is no cash position; see the rules above. |
| `holdings_value` | Market value of the open positions at that day's close, USD. |
| `total_value` | Same as `holdings_value`. Kept for compatibility. |
| `pnl_pct` | `(holdings_value - invested) / invested * 100`. |
| `spy_value` | Value of the matched SPY shares at the same close, USD. |
| `spy_pnl_pct` | Same formula, against the SPY money committed. |
| `vs_spy_pp` | `pnl_pct - spy_pnl_pct`, in percentage points. |

Rules for writing a row:

- **A day with no fills still gets a row.** A gap in the series is
  indistinguishable from a flat day when the returns are computed, and the
  difference matters.
- **An empty cell is allowed and is stored as NULL, never as 0.** If a position
  cannot be priced that day, leave `holdings_value` and everything derived from
  it empty rather than valuing it at zero.
- **Only US trading days.** No rows on weekends or market holidays; a zero-return
  day that was not a trading day distorts an annualised figure.
- Before the first bot has any fill, `invested` is 0 and the percentage columns
  are empty, not 0.

Validate the file before trusting it: in the Quantfoli repo,
`npx tsx scripts/paper-dryrun.ts` parses both CSVs from this repo and prints
valid/rejected counts without writing anything.

## Two things that must not change

- **The `bot` codes `KZ` and `BOD`.** They are the key every stored row is
  filed under, in an append-only table that rejects updates. Renaming a code
  orphans every row already recorded and splits the track record in two. The
  display names above are free to change; the codes are not.
- **The `(trade_id, event)` pair.** It is the uniqueness key that makes
  re-importing the same file a no-op. Re-using a `trade_id` for a different
  trade silently hides the second one.
