---
name: trade-lot-stamp
description: Record a short timestamped archival log entry for each trading lot — an entry stamp when the position opens and an exit stamp when it closes — capturing market sentiment, underlying price and key levels, event catalyst and company news, options contract and greeks, exit price targets, stop, and strategy, then audit the closed lot across thesis, process, risk, execution, and record quality to produce a verdict, a root cause, behavior tags, and time-boxed operating rules. Use for requests mentioning 交易日志、复盘、建仓、平仓、加仓、减仓、持仓记录、期权、期权日志、止损、目标价、交易审计、流程复盘, trade log, trade journal, trade stamp, lot record, entry and exit log, options trade, position record, catalyst or event-driven trade, post-trade review, trade audit, process adherence, risk discipline, rule violation, cool down, or amending, closing, auditing, or reviewing a previously stamped lot.
---

# Trade Lot Stamp

Press one short, dated record per trading lot: an **entry stamp** at open, an
**exit stamp** at close, both in the same file. Audit the lot's process when it
closes. Optimized for options and for 复盘.

Read [references/stamp-template.md](references/stamp-template.md) before writing any stamp.
Read [references/audit-framework.md](references/audit-framework.md) before grading a closed lot (`EXIT`, `AUDIT`) or aggregating grades (`REVIEW`).
Read [references/field-reference.md](references/field-reference.md) when a field's meaning, an options input, or a sentiment or catalyst classification is unclear.
Read [references/review-guide.md](references/review-guide.md) for period, strategy, or ticker review.

This skill **documents and audits** trades. It never recommends entering,
exiting, sizing, or holding anything, never predicts a price, and never offers
therapy, diagnosis, or judgement of the user.

## Modes

Determine the mode from the request before doing anything else.

| Mode | Trigger | Action |
| --- | --- | --- |
| `STAMP` | a new position was opened | check active operating rules, then create a lot file with an entry stamp |
| `EXIT` | a position was closed or trimmed | append an exit stamp, then audit the lot if fully closed |
| `AMEND` | plan changed while the lot is open (rolled, stop moved, target changed, added) | append a dated amendment block; never rewrite history |
| `AUDIT` | a closed lot needs grading or re-grading | run the audit on an existing lot file |
| `REVIEW` | 复盘 over a period, strategy, ticker, or driver | aggregate the ledger and the audits |

## Files

| Path | Role |
| --- | --- |
| `trades/<YYYY-MM-DD>-<TICKER>-<strategy>.md` | one lot: entry stamp, amendments, exit stamps, audit |
| `trades/_ledger.csv` | one row per lot, for aggregation |
| `trades/_risk-plan.yaml` | account-level limits, written once by the user |
| `trades/_operating-rules.md` | append-only, dated, time-boxed rules produced by audits |

## Workflow

1. **Resolve the mode**, then for `EXIT`/`AMEND`/`AUDIT` **find the lot**. Weeks
   usually pass between the entry and the exit, and a later session remembers
   nothing — the file is the only memory, so look it up rather than relying on
   conversation:
   - `grep -l "^# <TICKER> " trades/*.md`, then read the `**Status**` line of
     each hit;
   - exactly one `OPEN` match → append to it;
   - several `OPEN` matches → list them with lot ID, entry date, and structure,
     and ask which. Never guess, and never merge two lots into one file;
   - only `CLOSED` matches → say so and ask. A closed lot never silently takes
     another exit block; the user may mean a new position on the same ticker;
   - no match at all → the entry was never stamped. Say so, and offer to write a
     `reconstructed` entry stamp first so the lot has a plan to be audited
     against. Do not write a bare exit with no entry.
   - no ticker given ("closed it at 7") → ask. A fresh session has no antecedent
     for "it".
2. **Fix the timestamps honestly — there are two of them.** Every stamp carries
   a **fill time** and a **record time**, both in ET for US markets. Convert the
   trader's local clock to ET first; a 23:19 reading is 11:19 ET from Beijing and
   23:19 ET from New York, and the tag differs.
   The recording tag is set by the **gap between them**, with one cap:
   - `live` — recorded **within 24 hours of the fill**, and while the position
     was still unresolved;
   - `delayed` — more than 24 hours after the fill, but still written before the
     outcome was known;
   - `reconstructed` — written after the outcome was known, or from a broker
     statement.
   The 24-hour window is deliberately generous: a trader cannot stop and write
   during the session, and a tag nobody can earn tells a review nothing.
   **The cap is what the tag exists for.** A record made after the position
   resolved is never `live`, however few hours have passed — a 0 DTE contract
   written up six hours later is written up after it expired, and calling that
   `live` would let the tag certify exactly the records it was built to catch.
   Both timestamps go on the stamp in ET so the tag can be re-derived; it is
   derived, not asserted, and a correction to a timezone or either timestamp
   changes it.

3. **Collect the facts.** Take whatever the user supplies — a fill, a
   screenshot, a sentence. Then fill gaps in this order:
   - derive what is derivable (cost basis, breakeven, DTE, max loss, R, percentages);
   - look up what is checkable and cite it (underlying price, VIX, IV rank, earnings date, a headline with source and date);
   - ask, in **one** batched question, only for what is neither derivable nor checkable — typically thesis, targets, stop, conviction, and size.
   Never invent a number, a headline, a greek, or a price level. Leave a field
   as `—` rather than guess, and say which fields are blank.
4. **Keep it short.** The entry stamp is one screen. Thesis is one or two
   sentences and must be falsifiable. Every table row is a fact or a number.
5. **Write or append**, then **update the ledger** at `trades/_ledger.csv`,
   creating it with the header from `references/field-reference.md` if absent.
6. **Report** the saved path, the lot ID, the verdict when one was produced, and
   every field left blank.

## Rules

### Entry stamp (`STAMP`)

- Read `trades/_operating-rules.md` first. If an active rule appears to cover
  this lot — a risk cap, a required record, a cool-down — say so in the report
  **before** writing the stamp, quote the rule and its expiry, and note whether
  the lot as described breaches it. Never block, never nag, never repeat it
  twice. Rules that are written but not read at entry time change nothing.
- Record the **underlying** price at entry, not only the option price. Both are
  needed for 复盘.
- Record the **risk gate** in one line: portfolio heat at entry against the limit
  in `_risk-plan.yaml`, whether the setup was confirmed, and the regime gate
  (`allowed` / `restrictive` / `cash-only`). These three are what make the audit
  able to separate a rule breach from bad luck; without them Risk and Process
  grade as `—`.
- Classify the trade `[EVENT-DRIVEN]` or `[SETUP-DRIVEN]`. If event-driven, name
  the event, its date, whether the date is scheduled, and a source with its date.
- The plan table is mandatory: target 1, stop, and invalidation at minimum. A lot
  without a stop is stamped `Stop | — (NONE SET)` and flagged in the report.
- Express risk in **R** — R is the planned max loss on the lot. Record max loss in
  currency and in percent of book so outcomes compare across sizes.
- For options record contract, expiry, strike, right, quantity, net debit or
  credit, DTE, IV and IV rank, delta, and breakeven at the underlying. Multi-leg
  positions list each leg, then the net.

### Exit stamp (`EXIT`)

- Record actual fills, not intended ones. Partial exits get their own dated
  block; the lot stays `OPEN` until the last contract is closed.
- Compute P&L in currency, percent, and **R multiple**. R multiple is the honest
  unit — +$400 on 2R risk is worse than +$300 on 0.5R.
- Note when a stop gapped through: realized risk then exceeded 1R, which is a
  Risk finding about the structure, not about discipline.
- Tag the exit reason (`target`, `stop`, `time-stop`, `invalidation`,
  `discretionary`, `assignment`, `expiry`) plus one clause of detail.
- On full close, run the audit.

### Audit (`EXIT` on full close, or `AUDIT`)

Follow `references/audit-framework.md`. In short:

- Grade **Record first**, then Thesis, Process, Risk, Execution, A–F. An axis with
  no evidence is `—`. Below `C` on Record, say the audit is evidence-limited and
  hold every behavior tag to `low` confidence.
- Assign one primary root cause. **`randomness` is a legitimate verdict** — when
  the plan was followed and the setup was valid, do not manufacture a lesson.
- Assign a verdict: `OK`, `WARN`, `REVIEW_REQUIRED`, `RULE_VIOLATION`, `COOL_DOWN`.
- Tag behavior only with cited evidence, a confidence, and one reflection
  question. Check the options tags — `iv_blindness`, `theta_denial`,
  `catalyst_drift`, `gamma_week_hold`, `roll_to_avoid_loss`,
  `assignment_surprise`, `spread_leg_out` — on every options lot.
- Propose at most three time-boxed operating rules, append accepted ones to
  `trades/_operating-rules.md`, and end with the decision gate
  (`accept / modify / defer / log-only`, default `log-only`). Rules are proposed,
  never imposed.
- The P&L never touches an axis grade. A winning lot taken off-plan grades as a
  process failure; a losing lot taken on-plan grades as a process success.

## Guardrails

- Never fabricate market data, news, greeks, or fills. Unknown is `—`.
- Never rewrite or delete a stamp, an amendment, or an audit. Corrections are
  appended, dated, and labeled.
- Never backdate an `AMEND` reason, and never turn a `reconstructed` stamp into
  a `live` one.
- Never infer a thesis the user did not state; ask for it.
- Never raise a behavior tag without cited evidence, and never assert a pattern
  from a single lot.
- Never infer personality, mental state, or intent. Tags are `possible pattern`.
- Never let the P&L set a process grade, in either direction.
- Never let an audit run past a screen, and never pad it with generic advice.
- Never emit a recommendation, a price forecast, a "you should have", or any
  moralizing about a loss or a broken rule. Objective rule language only.
- Match the user's language. Keep field labels, tags, verdicts, and root-cause
  names in English as written in the references so the ledger stays sortable.
