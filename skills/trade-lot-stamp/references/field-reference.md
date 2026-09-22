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
| Return on risk | credit ÷ max loss — what the premium pays for the risk taken |
| Credit as % of width | net credit ÷ width — the same number before the credit is netted out |
| 1R | planned max loss on the lot, in currency |
| R multiple | realized P&L ÷ 1R |
| % of book | lot risk ÷ total account value |
| Hold | exit date − entry date, in calendar days |
| Credit-structure P&L % | realized P&L ÷ credit received (percent of max profit captured) |
| Debit-structure P&L % | realized P&L ÷ net debit paid |

A credit structure has two defensible percentages and they differ by a lot: P&L
over credit received (percent of max profit captured) and P&L over risk. The
ledger's `pnl_pct` always holds the first, because it is the one that is
comparable across credit lots; the second is `r_multiple`, which is already a
column. Closing a $17 credit spread at $7 is 58.8% of max profit and +0.12R —
both true, and the gap between them is the point of recording each.

**Return on risk is the field a credit seller actually screens on.** Absolute
premium is not comparable across structures — $17 on a $100-wide spread and $12
on a $50-wide are 20.5% and 31.6% return on risk. Record it on every credit lot;
a review cannot compare credit trades without it, and its reciprocal (risk:reward
"4.9 : 1 against") carries the same information in the direction nobody thinks in.

A low return on risk is the arithmetic signature of a far-out-of-the-money strike:
high probability of keeping the credit, little paid for taking the risk. A high one
means the short strike sits close to the money. Neither is good or bad on its own —
record the number and let the review find out which one the trader actually wins on.

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

## Axis grades

The same A–F scale grades each of the five audit axes in
`audit-framework.md`. Grade the decisions that were controllable; the P&L is
recorded separately and never feeds a grade.

| Grade | Meaning |
| --- | --- |
| A | fully consistent with the written plan and the risk plan |
| B | one deviation, and it was reasoned and recorded at the time |
| C | one material deviation, unreasoned — or a plan too vague to deviate from |
| D | a limit was exceeded, or a stop was moved against the position |
| F | no plan, no stop, or risk far beyond the stated limit |
| — | no evidence in the record; not a guess, not a zero |

Read per axis: `D` on Risk means a risk limit was breached; `D` on Execution
means the fills broke the plan; `D` on Record means the lot is barely
auditable. A `—` on Record is impossible — an empty record grades `F`.

## Verdicts and root causes

Defined in `audit-framework.md`. Ledger values, verbatim:

- `verdict` — `OK`, `WARN`, `REVIEW_REQUIRED`, `RULE_VIOLATION`, `COOL_DOWN`
- `root_cause` — `thesis_quality`, `execution`, `risk_sizing`,
  `market_environment`, `rule_violation`, `randomness`, `unknown`

## Ledger

`trades/_ledger.csv`, one row per lot, header exactly:

```csv
lot_id,parent_lot_id,status,ticker,instrument,strategy,driver,catalyst,recorded,entry_ts,entry_price,underlying_in,qty,cost,risk_r,pct_book,heat_r,setup_confirmed,regime_gate,dte_in,iv_in,ivr_in,delta_in,target1,stop,exit_ts,exit_price,underlying_out,exit_reason,pnl,pnl_pct,r_multiple,realized_risk_r,hold_days,verdict,root_cause,grade_record,grade_thesis,grade_process,grade_risk,grade_execution,cycle_id,linked_lot,link_type,credit_pct_width,return_on_risk,findings,tags,outcome
```

- `parent_lot_id` is empty on the parent row and carries the parent's lot ID on
  a partial-exit child row.
- `driver` is `event` or `setup`; `catalyst` is a short slug (`q3-earnings`,
  `20d-breakout`).
- `recorded` is `live`, `same-day`, or `reconstructed`.
- Timestamps are `YYYY-MM-DD HH:MM TZ`. Money is plain numbers, no symbols or
  thousands separators. Unknown is empty, never `0`.
- `heat_r` is total open risk across all lots at the moment of entry, this lot
  included; `regime_gate` is `allowed`, `restrictive`, or `cash-only`.
- `realized_risk_r` is what the lot actually risked, which exceeds `risk_r` when
  a stop gapped through. `size_creep` compares these two, not `risk_r` alone.
- `cycle_id` groups the lots of one wheel cycle (`<TICKER>-WHEEL-<n>`). Review
  reports the cycle, not its legs — see `audit-framework.md`.
- `linked_lot` names the other lot ID when an exercise or assignment moves value
  between two lots; `link_type` is `hedge_of`, `hedged_by`, `assigned_into`, or
  `incidental`. Review must read linked lots as a pair — the option's payoff sits
  in the other lot's price, so ranking either one alone is wrong.
- `credit_pct_width` and `return_on_risk` are percentages, credit lots only;
  both empty on a debit lot.
- `findings` and `tags` are space-separated lists of finding topics and behavior
  tags from `audit-framework.md`, each empty when none fired. Both use the
  controlled vocabularies so a review can count them across lots.
- Grade columns hold a single letter or `-` for an ungraded axis.
- Quote any field containing a comma.

## Risk gate fields

Recorded on the entry stamp in one line. These three are what let the audit tell
a rule breach apart from bad luck; without them Risk and Process grade as `—`.

- **Assignment exposure** (short single-leg options only) — strike × 100 ×
  contracts, which is what assignment obligates you to buy or sell. It is
  unrelated to the premium and is usually much larger than any loss figure, so a
  single-leg limit is written against this number. It is the field that explains
  choosing a spread over a naked put on a high-priced underlying.
- **Portfolio heat** — total open risk in R across every open lot, counting this
  one, at the moment of entry. Compare against `max_portfolio_heat_r`. This is
  the field that catches the third correlated position that looked fine alone.
- **Setup confirmed** — `✓` if the entry conditions written in the strategy were
  met before the fill, `✗` if the entry front-ran them, `—` if the strategy has
  no confirmation step. `✗` is the primary evidence for `fomo_entry`.
- **Regime gate** — `allowed`, `restrictive`, or `cash-only`, resolved from the
  regime tone against `regime_gate` in `_risk-plan.yaml`. Opening at
  `cash-only`, or at full size under `restrictive`, is a `critical` finding.

Record these at entry, not at exit. Reconstructed heat is a guess, and a guessed
gate makes every Risk grade downstream of it meaningless.
