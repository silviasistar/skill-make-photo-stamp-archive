# Field Reference

What each field means, how to derive it, and the controlled vocabularies. Leave
any field that is neither supplied, derivable, nor checkable as `—`.

## Derived values — compute, never ask

| Value | Formula |
| --- | --- |
| Option cost | net debit × 100 × contracts |
| DTE | expiry date − entry date, in calendar days |
| Long-call breakeven | strike + net debit |
| Long-put breakeven | strike − net debit |
| Debit-spread breakeven | long strike ± net debit (call: +, put: −) |
| Credit-spread breakeven | short strike ∓ net credit (call: +, put: −) |
| Debit-spread max profit | (width × 100 × contracts) − cost |
| Credit-spread max loss | (width × 100 × contracts) − credit |
| 1R | planned max loss on the lot, in currency |
| R multiple | realized P&L ÷ 1R |
| % of book | lot risk ÷ total account value |
| Hold | exit date − entry date, in calendar days |

For a long option with no stop below the premium, 1R is the full premium paid.
If a stop is set at −50% of premium, 1R is half the premium — and the exit stamp
must note if the stop gapped through, because realized risk then exceeded 1R.

## Position fields

- **Instrument** — shares, or `TICKER YYYY-MM-DD $STRIKE C/P ×N`. Always the full
  contract; `NVDA calls` is not a record.
- **Entry** — net fill price and the fill timestamp with timezone. Use the fill,
  not the mid or the limit that was working.
- **Underlying** — the underlying's price at the moment of the option fill. This is
  the single most-skipped field and the one review needs most: without it there is
  no way to tell later whether the thesis on the stock was right and the structure
  was wrong.
- **Greeks** — delta of the position, DTE, implied volatility, and IV rank or IV
  percentile. Record IV rank whenever the strategy is long or short volatility.
  Note theta per day when DTE is under 21.

## Regime / sentiment line

Record what was observable at entry, in this order. One clause each, no forecast.

1. **Tone** — `risk-on`, `risk-off`, or `chop`.
2. **Index** — an index versus a reference level (e.g. `SPY above 20d`, `QQQ below 50d`).
3. **Volatility** — VIX level, and its direction if it moved notably that session.
4. **Group** — the ticker's sector or peer group, leading or lagging.

Optional when they informed the decision: breadth, put/call ratio, term structure
(contango/backwardation), the dollar, rates, or a scheduled macro print. Record a
sentiment input only if it actually entered the decision — a log of everything is
a log of nothing.

## Catalyst classification

Every lot is exactly one of:

- **`[EVENT-DRIVEN]`** — the thesis depends on a specific event. Record the event,
  its date, whether the date is scheduled or unscheduled, and the source with its
  date. Scheduled events: earnings, guidance, investor day, FDA PDUFA, court date,
  index rebalance, lockup expiry, CPI/FOMC/NFP. Unscheduled: M&A, analyst action,
  product news, executive change, litigation, regulatory action, macro shock.
- **`[SETUP-DRIVEN]`** — the thesis depends on price, structure, or flow. Name the
  setup: breakout, pullback, mean reversion, trend continuation, range fade, gap
  fill, volatility expansion, volatility crush, earnings IV sale, delta hedge flow.

Cite news with source and date. Never paraphrase a headline into a claim the
source did not make, and never record a rumor as a fact — tag it `unconfirmed`.

## Exit reason tags

| Tag | Means |
| --- | --- |
| `target` | a written price target was hit |
| `stop` | the written stop was hit |
| `time-stop` | the written date or DTE limit was reached |
| `invalidation` | the thesis broke on something other than price |
| `discretionary` | closed for a reason not in the written plan |
| `assignment` | assigned or exercised |
| `expiry` | expired, worthless or in the money |

`discretionary` is not a failing grade by itself, but it always requires a
sentence in **Plan vs actual** explaining what was seen that the plan did not
anticipate. Most process leaks show up here.

## Process grades

| Grade | Meaning |
| --- | --- |
| A | entry, size, stop, and exit all matched the written plan |
| B | one deviation, and it was reasoned and recorded |
| C | one material deviation, unreasoned — or a plan too vague to deviate from |
| D | stop moved against the position, or size exceeded the written risk |
| F | no plan, no stop, or risk taken far beyond the stated limit |

Grade the decisions that were controllable. The outcome is graded separately and
never feeds into this column.

## Ledger

`trades/_ledger.csv`, one row per lot, header exactly:

```csv
lot_id,parent_lot_id,status,ticker,instrument,strategy,driver,catalyst,recorded,entry_ts,entry_price,underlying_in,qty,cost,risk_r,pct_book,dte_in,iv_in,ivr_in,delta_in,target1,stop,exit_ts,exit_price,underlying_out,exit_reason,pnl,pnl_pct,r_multiple,hold_days,process_grade,outcome
```

- `parent_lot_id` is empty on the parent row and carries the parent's lot ID on
  a partial-exit child row.
- `driver` is `event` or `setup`; `catalyst` is a short slug (`q3-earnings`,
  `20d-breakout`).
- `recorded` is `live`, `same-day`, or `reconstructed`.
- Timestamps are `YYYY-MM-DD HH:MM TZ`. Money is plain numbers, no symbols or
  thousands separators. Unknown is empty, never `0`.
- Quote any field containing a comma.
