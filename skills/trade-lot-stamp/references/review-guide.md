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
| Adherence rate | lots graded A or B ÷ closed lots |
| Plan-exit share | exits tagged `target`/`stop`/`time-stop` ÷ closed lots |
| Max drawdown in R | largest peak-to-trough run of cumulative R |
| Hold skew | mean hold of winners vs mean hold of losers |

Then cut expectancy by: strategy, driver, day of week, DTE bucket
(`0–7`, `8–21`, `22–45`, `45+`), IV rank bucket at entry (`<30`, `30–60`, `>60`),
and `recorded` tag. Report only the cuts with enough lots to show anything.

## Patterns to test for

Check each against the data and report it only with the lot IDs that support it.
Never assert a pattern from a single trade.

- **Process drift** — adherence rate falling across the period.
- **Stop migration** — `AMEND` blocks that moved a stop against the position.
- **Cut winners, hold losers** — winners' mean hold well below losers'.
- **Size inversion** — largest `pct_book` on the lowest-conviction or worst-graded lots.
- **Discretionary leak** — `discretionary` exits underperforming plan exits in mean R.
- **IV blindness** — long-premium lots clustered in high IVR, or short-premium in low.
- **Theta denial** — losses concentrated in low-DTE lots held past the time stop.
- **Catalyst drift** — event-driven lots held past their event with no new thesis.
- **Revenge sizing** — a lot opened within a session of a loss at above-average size.
- **Hindsight inflation** — `reconstructed` lots grading better on process than `live` ones.
  This measures the log's honesty, not the trading. Report it first when it shows up.

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
Adherence [n]% · plan-exits [n]% · grades A[n] B[n] C[n] D[n] F[n]
[the recurring deviation, with its lot IDs]

## Rules carried forward
[collect the one-line Rule from each lot's 复盘; merge duplicates; list the
repeats first, with a count — a rule written three times and broken three times
is the finding]
```

## Discipline

- Separate process from outcome everywhere. A profitable period with a falling
  adherence rate is a warning, and the report says so.
- Attribute results to what the record supports. Luck is a permissible conclusion.
- Quote the lot file's own words when reporting a deviation; do not re-characterize.
- Exclude open lots from every metric. Mark-to-market is not a result.
- No advice, no forecasts, no encouragement. Report what happened.
