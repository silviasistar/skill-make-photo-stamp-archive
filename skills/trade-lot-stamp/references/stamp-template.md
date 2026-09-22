# Stamp Templates

One file per lot. The entry stamp is written once; exit and amendment blocks are appended below it, newest last.

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

## Context

- **Regime** risk-on · SPY above 20d · VIX 14.2 · semis leading
- **Catalyst** [EVENT-DRIVEN] Q3 earnings 2026-11-19 after close — company IR calendar, 2026-09-15
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
| Reason | `target` — T1 hit at $6.30, took the rest into strength at $6.80 |

## 复盘

- **Plan vs actual** T1 hit as planned, but exited the full lot at T1 instead of trimming 3 of 5. Left the T2 runner on the table.
- **Process** B — entry, size, and stop were on plan; the exit was not the exit I wrote down.
- **Outcome** win · +1.24R
- **Worked** waiting for the 20d reclaim before entering instead of chasing the first green day.
- **To fix** sold the runner out of discomfort with an unrealized gain, not on any written signal.
- **Rule** when T1 hits, the runner comes off only on T2, the stop, or the time stop — nothing else.
```

---

## 2. Entry stamp — blank

```markdown
# [TICKER] · [Strategy] · [YYYY-MM-DD]

**Lot** [TICKER-YYYYMMDD-n] · **Status** OPEN · **Recorded** [live | same-day | reconstructed] · [YYYY-MM-DD HH:MM TZ]

## Position

| | |
| --- | --- |
| Instrument | [shares ×N] or [TICKER YYYY-MM-DD $STRIKE C/P ×N] |
| Entry | [$price debit/credit] · [YYYY-MM-DD HH:MM TZ] |
| Cost / Risk | [$cost] · [%] of book · 1R = [$max loss] |
| Underlying | [$price at entry] |
| Greeks | [Δ] · DTE [n] · IV [%] (IVR [n]) |
| Breakeven | [$underlying at expiry] |

## Context

- **Regime** [risk-on / risk-off / chop] · [index vs key MA] · VIX [n] · [sector tone]
- **Catalyst** [[EVENT-DRIVEN] event + date + source] or [[SETUP-DRIVEN] setup name]
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

## 复盘

- **Plan vs actual** [where execution matched the written plan and where it did not]
- **Process** [A–F] — [plan followed? size right? stop honored? thesis actually tested?]
- **Outcome** [win / loss / scratch] · [±n.nnR]
- **Worked** [one line]
- **To fix** [one line]
- **Rule** [one transferable line]
```

Partial exits append their own block and leave `Status` as `OPEN`. Only the final
exit flips the header to `**Status** CLOSED` and writes the 复盘 section, which
covers the lot as a whole.

---

## 4. Amendment — blank

```markdown
---

## AMEND · [YYYY-MM-DD HH:MM TZ]

- **Changed** [field: old → new]
- **Why** [one line — the reason as of now, not with hindsight]
```

Use for rolls, added or reduced size, moved stops, and revised targets. Never
edit the original stamp; the gap between the first plan and the amendments is
itself the review material.
