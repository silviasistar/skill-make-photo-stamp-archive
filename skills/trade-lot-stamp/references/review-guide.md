# 复盘 Review Guide

For `REVIEW` mode only. The stamps are the raw material; this is how to read them
back. Review reports the record — it does not advise on future positions.

## Scope

Ask for, or infer from the request, exactly one scope: a period (week, month,
quarter), a strategy, a ticker, or a driver (`event` vs `setup`). Mixing scopes
in one report hides the pattern.

Read `trades/_ledger.csv` first for the numbers. Open individual lot files only
for the ones the numbers single out — the worst R, the best R, every `D`/`F`
process grade, and every `discretionary` exit.

## Metrics

Compute from closed lots in scope. State the sample size before the numbers, and
say plainly when it is too small to mean anything — under roughly 20 closed lots,
report the numbers as a description of what happened, not as an edge.

| Metric | Formula |
| --- | --- |
| Win rate | wins ÷ closed lots |
| Average win / average loss | mean R of winners, mean R of losers |
| Expectancy | mean R across all closed lots |
| Profit factor | gross profit ÷ gross loss |
| Adherence rate | lots with Process graded A or B ÷ closed lots |
| Verdict mix | share by `OK` / `WARN` / `REVIEW_REQUIRED` / `RULE_VIOLATION` / `COOL_DOWN` |
| Randomness share | lots with root cause `randomness` ÷ losing lots |
| Plan-exit share | exits tagged `target`/`stop`/`time-stop` ÷ closed lots |
| Max drawdown in R | largest peak-to-trough run of cumulative R |
| Hold skew | mean hold of winners vs mean hold of losers |

Then cut expectancy by: strategy, driver, day of week, DTE bucket
(`0–7`, `8–21`, `22–45`, `45+`), IV rank bucket at entry (`<30`, `30–60`, `>60`),
`recorded` tag, root cause, and verdict. Report only the cuts with enough lots
to show anything.

Watch the **randomness share**. A trader who never concludes `randomness` is
over-fitting lessons to noise; one who concludes it on most losses is using it
to avoid looking at the process. Neither is a number to target — it is a
question to raise when the split looks lopsided.

## Cross-lot patterns

Per-lot behavior tags are assigned by the audit at close, per
`audit-framework.md`. **Review does not re-derive them** — it counts them,
which is the whole point: one `premature_exit` is an anecdote, six is the
finding. Report a tag only with the lot IDs behind it and its cost in summed R.

Then test what only becomes visible across lots:

- **Process drift** — mean axis grades falling over the period.
- **Verdict drift** — the `OK`/`WARN` share shrinking week over week.
- **Stop migration** — `AMEND` blocks that moved a stop against the position.
- **Cut winners, hold losers** — winners' mean hold well below losers'.
- **Size inversion** — largest `pct_book` on the lowest-conviction or
  worst-graded lots.
- **Heat blindness** — losses clustered in lots opened above 75% of the heat
  limit, which no single lot's audit can see.
- **Gate erosion** — a rising share of lots opened under `restrictive` or
  `cash-only`.
- **Discretionary leak** — `discretionary` exits underperforming plan exits in
  mean R.
- **Rule recidivism** — rules in `_operating-rules.md` that expired `broken`,
  especially a rule set more than once. A rule written three times and broken
  three times is the most important line in the report.
- **Hindsight inflation** — `reconstructed` lots grading better than `live`
  ones. This measures the log's honesty, not the trading. Report it first when
  it appears, because it discredits everything else in the report.

Cool-down check: recount the `COOL_DOWN` triggers in `audit-framework.md` across
the period. A trigger that fired without a cool-down being taken is a finding
on its own, and it outranks any P&L result in the report.

## Report shape

Keep it to one page.

```markdown
# 复盘 · [scope] · [date range]

**Sample** [n] closed lots ([n] wins / [n] losses / [n] scratch) · [n] open

## Numbers
[the metrics table, plus the two or three cuts that actually separated]

## What the record shows
- [finding] — [lot IDs]
- [finding] — [lot IDs]

## Process
Adherence [n]% · plan-exits [n]% · Process grades A[n] B[n] C[n] D[n] F[n]
Verdicts OK[n] WARN[n] REVIEW[n] VIOLATION[n] COOL_DOWN[n]
Mean axis grades: record [x] · thesis [x] · process [x] · risk [x] · execution [x]
[the recurring deviation, with its lot IDs]

## Tags
[tag counts with lot IDs and summed R cost, highest cost first]

## Rules
[from `_operating-rules.md`: which expired held, which expired broken, which are
still active. Then the one-line Rule from each lot's 复盘, duplicates merged,
repeats first with a count. Propose at most three rules to carry forward, and
end with the decision gate.]

`accept / modify / defer / log-only` (default: log-only)
```

## Discipline

- Separate process from outcome everywhere. A profitable period with a falling
  adherence rate is a warning, and the report says so.
- Attribute results to what the record supports. Luck is a permissible conclusion.
- Quote the lot file's own words when reporting a deviation; do not re-characterize.
- Exclude open lots from every metric. Mark-to-market is not a result.
- No advice, no forecasts, no encouragement, no moralizing about losses or
  broken rules. Report what happened, in objective rule language.
- Read `audit-framework.md` before grading or re-grading anything. Review
  aggregates audits; it does not overturn them. A lot whose audit looks wrong
  is re-run through `AUDIT` mode, which appends a dated re-grade rather than
  editing the original.
