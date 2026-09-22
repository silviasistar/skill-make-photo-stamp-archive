# Stamp Templates

One file per lot. The entry stamp is written once; amendment, exit, and audit
blocks are appended below it, newest last. Nothing above is ever edited.

---

## 1. Worked example (options, event-driven)

```markdown
# NVDA · Long Call · 2026-09-22

**Lot** NVDA-20260922-1 · **Status** OPEN · **Recorded** live · 2026-09-22 10:34 ET

## Position

| | |
| --- | --- |
| Instrument | NVDA 2026-10-17 $180 C ×5 |
| Entry | $4.20 debit · 2026-09-22 10:34 ET |
| Cost / Risk | $2,100 · 1.4% of book · 1R = $1,050 |
| Underlying | $174.30 |
| Greeks | Δ 0.42 · DTE 25 · IV 48% (IVR 61) |
| Breakeven | $184.20 underlying at expiry |
| Risk gate | heat 3.1R / 4.0R · setup confirmed ✓ · regime `allowed` |

## Context

- **Regime** risk-on · SPY above 20d · VIX 14.2 · semis leading
- **Catalyst** [EVENT-DRIVEN] Q3 earnings 2026-11-19 after close, scheduled — company IR calendar, 2026-09-15
- **Levels** support $168 (20d MA) · resistance $182 (Aug 14 high)

## Thesis

Semis breaking out with the group; expect a move through the August high into
the pre-earnings run. Wrong if it closes back below the 20d.

## Plan

| | |
| --- | --- |
| Target 1 | $6.30 (+50%) — trim 3 of 5 |
| Target 2 | $8.40 (+100%) — trim remainder |
| Stop | $2.10 (−50% premium) |
| Time stop | 2026-10-03 (DTE 14) if underlying still below $178 |
| Invalidation | daily close below $168 |
| Max loss | $1,050 (0.7% of book) = 1R |

## Notes

- Conviction 3/5. Sized half normal because IVR is already elevated.
```

Appended at close:

```markdown
---

## Exit · 2026-09-30 14:12 ET · full

| | |
| --- | --- |
| Closed | $6.80 credit ×5 |
| P&L | +$1,300 · +61.9% · +1.24R |
| Hold | 8 days (17 DTE remaining) |
| Underlying | $182.10 |
| Reason | `discretionary` — T1 hit at $6.30, closed the full lot at $6.80 instead of trimming 3 of 5 |

## Audit

**Verdict** `WARN` · **Root cause** `execution` · **Outcome** win · +1.24R

| Axis | | Evidence |
| --- | --- | --- |
| Record | A | all planned and actual fields present |
| Thesis | A | falsifiable, invalidation named, recorded `live` before entry |
| Process | B | entry, size, stop, and regime gate all on plan |
| Risk | A | 1.4% of book, heat 3.1R inside the 4.0R limit |
| Execution | C | exited the full lot at T1; the written plan held 2 of 5 for T2 |

**Findings**

- `warning` `exit_vs_plan` — plan held 2 contracts for T2 at $8.40; all 5 were
  closed at $6.80. Evidence: Plan table vs exit block.

**Possible patterns**

- `premature_exit` (medium) — full close at T1 with no invalidation noted and no
  `AMEND` changing the plan. Reflection: what written rule did closing the
  runner satisfy?

**Operating rules proposed**

- Next 3 lots: once T1 fills, the runner comes off only at T2, the stop, or the
  time stop. — trigger `premature_exit` · expires 2026-10-31

`accept / modify / defer / log-only` (default: log-only)

## 复盘

- **Worked** waited for the 20d reclaim instead of chasing the first green day.
- **To fix** sold the runner on discomfort with an unrealized gain, not a signal.
- **Rule** T1 filling is not a reason to close the runner.
```

Note the grade: a **+1.24R winner** graded `C` on Execution and picked up a
behavior tag. That is the framework working as intended.

---

## 2. Entry stamp — blank

```markdown
# [TICKER] · [Strategy] · [YYYY-MM-DD]

**Lot** [TICKER-YYYYMMDD-n] · **Status** OPEN · **Recorded** [live | delayed | reconstructed] · fill [YYYY-MM-DD HH:MM ET] · recorded [YYYY-MM-DD HH:MM ET]

## Position

| | |
| --- | --- |
| Instrument | [shares ×N] or [TICKER YYYY-MM-DD $STRIKE C/P ×N] |
| Entry | [$price debit/credit] · [YYYY-MM-DD HH:MM TZ] |
| Cost / Risk | [$cost] · [%] of book · 1R = [$max loss] |
| Underlying | [$price at entry] |
| Greeks | [Δ] · DTE [n] · IV [%] (IVR [n]) |
| Breakeven | [$underlying at expiry] |
| Risk gate | heat [n]R / [limit]R · setup confirmed [✓ \| ✗ \| —] · regime `[allowed\|restrictive\|cash-only]` |

## Context

- **Regime** [risk-on / risk-off / chop] · [index vs key MA] · VIX [n] · [sector tone]
- **Catalyst** [[EVENT-DRIVEN] event + date + scheduled? + source] or [[SETUP-DRIVEN] setup name]
- **Levels** support [$] ([why]) · resistance [$] ([why])

## Thesis

[One or two sentences. What must happen, and what proves it wrong.]

## Plan

| | |
| --- | --- |
| Target 1 | [$] ([+%]) — [action] |
| Target 2 | [$] ([+%]) — [action] |
| Stop | [$ or −%] |
| Time stop | [date or DTE] [condition] |
| Invalidation | [what kills the thesis regardless of price] |
| Max loss | [$] ([%] of book) = 1R |

## Notes

- Conviction [n]/5. [Anything a reviewer would otherwise ask.]
```

Multi-leg positions replace the `Instrument` row with a leg table, then a net line:

```markdown
| Leg | Contract | Qty | Price |
| --- | --- | --- | --- |
| Long | [TICKER YYYY-MM-DD $K C/P] | [+n] | [$] |
| Short | [TICKER YYYY-MM-DD $K C/P] | [−n] | [$] |
| **Net** | [spread name, width $] | | **[$ debit/credit]** |
```

**Credit structures** (short vertical, iron condor, cash-secured put) replace the
`Cost / Risk` row, which assumes a debit. Money comes in, so there is no cost —
the risk is what the structure can lose, not what it cost:

```markdown
| Entry | [$price] credit · [YYYY-MM-DD HH:MM TZ] |
| Credit / Risk | +[$credit] received · [%] of book · 1R = [$max loss] |
| Max profit | [$credit] — [condition at expiry] |
| Max loss | [$width − credit, or assignment exposure] — [condition at expiry] |
| Return on risk | [credit ÷ max loss]% · credit is [n]% of width |
| Risk : reward | [n.n] : 1 [against \| for] |
```

For a short single-leg put also record `Assignment exposure | $[strike × 100 × n]`,
which is what the position obligates you to buy. That number, not the premium, is
what a single-leg limit is written against.

---

## 3. Exit stamp — blank

```markdown
---

## Exit · [YYYY-MM-DD HH:MM TZ] · [full | partial n of N]

| | |
| --- | --- |
| Closed | [$price] ×[n] |
| P&L | [±$] · [±%] · [±n.nnR] |
| Hold | [n days] ([n DTE remaining] or `expired`) |
| Underlying | [$price at exit] |
| Reason | `[target\|stop\|time-stop\|invalidation\|discretionary\|assignment\|expiry]` — [one clause] |
```

Partial exits append their own block and leave `Status` as `OPEN`. Only the final
exit flips the header to `**Status** CLOSED` and triggers the audit, which covers
the lot as a whole.

---

## 4. Audit — blank

Follow `audit-framework.md`. Omit any section with nothing in it; a clean lot's
audit is four lines, not this whole form.

```markdown
## Audit

**Verdict** `[OK\|WARN\|REVIEW_REQUIRED\|RULE_VIOLATION\|COOL_DOWN]` · **Root cause** `[cause]`[ (secondary: `[cause]`)] · **Outcome** [win\|loss\|scratch] · [±n.nnR]

| Axis | | Evidence |
| --- | --- | --- |
| Record | [A–F or —] | [what was present or missing] |
| Thesis | [A–F or —] | [evidence] |
| Process | [A–F or —] | [evidence] |
| Risk | [A–F or —] | [evidence] |
| Execution | [A–F or —] | [evidence] |

**Findings**

- `[info\|warning\|critical]` `[topic]` — [statement]. Evidence: [field comparison, quoted note, missing record, or AMEND date].

**Possible patterns**

- `[tag]` ([low\|medium\|high]) — [evidence]. Reflection: [one question]

**Operating rules proposed**

- [observable, time-boxed rule] — trigger `[tag or finding]` · expires [YYYY-MM-DD]

`accept / modify / defer / log-only` (default: log-only)

## 复盘

- **Worked** [one line]
- **To fix** [one line]
- **Rule** [one transferable line]
```

A `COOL_DOWN` verdict is stated on the first line of the audit and appended to
`trades/_operating-rules.md` the same day.

---

## 5. Amendment — blank

```markdown
---

## AMEND · [YYYY-MM-DD HH:MM TZ]

- **Changed** [field: old → new]
- **Why** [one line — the reason as of now, not with hindsight]
```

Use for rolls, added or reduced size, moved stops, and revised targets. Never
edit the original stamp; the gap between the first plan and the amendments is
itself the review material.

A **roll** always states whether the thesis is unchanged, revised, or gone. A roll
with no new thesis after the stop level is what `roll_to_avoid_loss` looks for,
and it can only be distinguished from a legitimate roll by what is written here
at the time.

---

## 6. `trades/_risk-plan.yaml` — written once

Account-level limits. The audit grades Risk against these; without the file,
Risk grades as `—` and no `RULE_VIOLATION` can be asserted.

Limits differ by instrument type, because the binding constraint differs. A
defined-risk spread is bounded by its max loss. A short single-leg put is bounded
by **assignment exposure** — what you are obligated to buy — which is usually far
larger than any loss figure and is the reason a trader picks a spread over a
naked put on a high-priced underlying.

```yaml
effective_from: 2026-09-22      # limits are NOT applied to lots opened earlier
book_value: 650000
currency: USD

limits:
  option_single_leg:
    max_assignment_exposure: 10000   # strike x 100 x contracts
  option_multi_leg:
    max_loss: 5000                   # width - credit, or net debit; = 1R
  stock_single_name:
    max_loss: 5000                   # = 1R

max_portfolio_heat: null        # total open risk across all lots
max_weekly_loss: null
max_consecutive_losses: null
regime_gate: null
require_setup_confirmation: null
require_stop_before_entry: null
```

`effective_from` is what keeps the audit honest about back-filled lots. A limit
written today is not a rule the trader broke last quarter: for a lot opened
before that date the audit records the comparison as an `info` observation and
never as a `RULE_VIOLATION`.

A limit left `null` is not zero and not unlimited — the audit grades the check it
governs as `—` and says which limit is missing. Fill them in as you decide them
rather than guessing now.

`1R` is read per instrument type from this file, so an R multiple means the same
thing across a stock lot and a spread lot.

## 7. `trades/_operating-rules.md` — append-only

```markdown
# Operating Rules

## Active

- [ ] [rule] — trigger `[tag]` · from [lot ID] · set [YYYY-MM-DD] · expires [YYYY-MM-DD]

## Expired

- [x] [rule] — set [YYYY-MM-DD] · expired [YYYY-MM-DD] · [held | broken at lot ID]
```

At most three active rules. `STAMP` reads this file before writing a new entry
stamp and flags any active rule the new lot appears to breach. Move a rule to
`Expired` on its date and record whether it held — a rule written three times and
broken three times is the finding.
