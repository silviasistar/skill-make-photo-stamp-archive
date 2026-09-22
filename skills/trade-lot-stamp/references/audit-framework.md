# Audit Framework

How a closed lot is graded. Applies to `EXIT` (audit runs at close) and `AUDIT`
(re-run on an already-closed lot). Read with `field-reference.md`.

The audit grades the decision process, not the money. **A clean-process loss is
acceptable. A profitable rule violation is still a process failure.** The P&L
never touches an axis grade.

## Five axes

Grade each A–F using `field-reference.md`'s scale. Grade only what the record
supports; an axis with no evidence is `—`, never a guess.

| Axis | Question | Evidence |
| --- | --- | --- |
| Thesis | Was the idea clear, falsifiable, and matched to the setup? | thesis, invalidation, catalyst, `Recorded` tag |
| Process | Did actual behavior follow the written plan? | entry timing, setup confirmation, regime gate, stop plan |
| Risk | Did actual risk stay inside the written limits? | 1R vs realized risk, portfolio heat, weekly loss, `_risk-plan.yaml` |
| Execution | Were entry, stop, adds, trims, and exit consistent with plan? | fills, `AMEND` blocks, partial exits, exit reason tag |
| Record | Was the evidence complete enough to trust this audit? | blank fields, `Recorded` tag, missing stop or plan |

**Record is graded first.** A low Record grade caps how much the other four mean:
below `C`, report the findings but state that the audit is evidence-limited, and
never raise a behavior tag above `low` confidence.

## Root cause

Exactly one primary, plus optional secondaries.

| Cause | Use when |
| --- | --- |
| `thesis_quality` | the plan was followed, but the idea itself was weak or unfalsifiable |
| `execution` | the setup or entry conditions were not met, or fills deviated from plan |
| `risk_sizing` | realized risk exceeded the written limit |
| `market_environment` | the regime gate was ignored, or conditions invalidated the structure |
| `rule_violation` | a stated rule was explicitly broken |
| `as_designed` | plan followed, structure behaved as intended, result matched the written thesis |
| `randomness` | plan followed, setup valid, market simply moved — for or against |
| `unknown` | the record cannot support a call |

`as_designed` is the only non-failure cause, and it is deliberately hard to earn:
it requires a thesis written **before** the outcome was known. A winning lot with
no recorded thesis is `unknown`, not `as_designed` — you cannot claim the plan
worked without having written the plan. Where the thesis is `reconstructed` but
the structure and the outcome are independently verifiable from the fills, record
`as_designed` and say in the evidence that the attribution rests on those, not on
recalled intent.

`randomness` is a legitimate and expected conclusion. Reach for it whenever the
plan was followed and the setup was valid — do not manufacture a lesson from a
trade that did not have one. Inventing a cause for a random outcome is the most
common way a review log becomes useless.

Prefer the cause the user supplied if they stated one. Otherwise derive:

1. realized risk > written 1R → `risk_sizing`
2. setup unconfirmed or stop moved unplanned → `execution`
3. regime gate ignored → `market_environment`
4. plan followed, thesis written before the outcome, result matched it → `as_designed`
5. plan followed, setup valid, no limit breached, but no pre-written thesis to
   match the result against → `randomness`
6. none of the above → `unknown`

## Verdict

| Verdict | Condition |
| --- | --- |
| `OK` | record adequate, no material finding, no behavior tag |
| `WARN` | one minor finding, or incomplete record |
| `REVIEW_REQUIRED` | a real process, risk, or execution finding, short of a stated-rule breach |
| `RULE_VIOLATION` | a rule in `_risk-plan.yaml` or the lot's own plan was broken |
| `COOL_DOWN` | escalation — see triggers below |

### COOL_DOWN triggers

Any one of:

- two or more `critical` findings on one lot;
- a `revenge_trade` tag plus two consecutive losing lots;
- four consecutive losing lots;
- the weekly loss limit in `_risk-plan.yaml` breached;
- two `RULE_VIOLATION` verdicts inside one week.

`COOL_DOWN` writes a dated entry to `trades/_operating-rules.md` and is reported
at the top of the audit. Its value is that it is mechanical: the decision to
stop is made by the record, at a moment when judgement is least reliable.

## Back-filled lots

A limit in `_risk-plan.yaml` binds only lots opened on or after its
`effective_from` date. For an earlier lot, compare anyway and record the result
as an `info` finding that names both numbers and states the limit was not in
force at the time. Never grade it `D` on Risk and never call it a
`RULE_VIOLATION`: a rule written today was not available to be followed last
quarter, and convicting the record of it teaches nothing and makes the log feel
like a trap. The comparison still belongs in the audit — it is how a trader
learns that past sizing sat outside what they now consider acceptable.

## Finding severity

- `info` — observation, no rule concern.
- `warning` — possible issue, or evidence too thin to call.
- `critical` — an explicit limit or written rule was breached.

## Corrections to recorded facts

Append-only protects the history of *decisions*. A wrong number is not a decision
and must not be left to mislead: a reader scanning the Position table sees the
error, not the correction appended eighty lines below.

For a data-entry error — a misheard fill, a transposed strike, a wrong timestamp —
append a dated `CORRECTION` block giving the wrong value, the right value, and
what it changes, **and** annotate the original field in place with a pointer,
leaving the wrong value visible:

```markdown
| Entry | $5.00 debit ⚠️ see CORRECTION 2026-09-22 | 
```

The original value stays; nothing is deleted or overwritten. This is the only
sanctioned in-place edit, and it applies to facts alone — never to a thesis, a
reason, a grade, or anything that records what was decided or believed at the
time.

Then re-grade. A corrected fact usually moves more than its own lot: any audit or
review that cited it has to be revisited, and a cross-lot narrative built partly
on the wrong number may not survive at all. Say so plainly when it does not. **A
review that quietly keeps its conclusion after losing the evidence for it is worse
than one that never ran.**

## Linked lots

An option exercised or assigned against stock you hold does not pay out in its
own P&L — the payoff lands in the **other lot's entry or exit price**. A long put
exercised against 100 shares shows as the full debit lost on the option lot and a
better exit price on the stock lot. Both numbers are right and adding them is not
double counting, but a reviewer scanning the ledger sees a losing option and
misses that it was the profitable half of the pair.

Record the relationship with `hedge_of` (or `hedged_by`) naming the other lot ID,
and audit such a lot **on the pair**, not standalone:

- state the option lot's own P&L, which is what it cost;
- state what the linked lot's price would have been without it, as a bounded
  counterfactual with its reference point named (a close, an after-hours print,
  a next-day open) — never an open-ended "could have been";
- grade the pair on whether the protection was placed before the exposure became
  unavoidable, not on which line shows red.

A lot that is linked only in hindsight is still recorded, with the linkage marked
`incidental`: the protection was real, the intent is unproven. The distinction
matters because an incidental hedge cannot be relied on to repeat.

## Finding topics

A finding names one topic from this list, so findings stay countable across
lots. Anything that fits none of these is `other` plus a short phrase.

| Topic | Axis | Fires when |
| --- | --- | --- |
| `pre_entry_thesis` | Thesis | no thesis recorded before entry, or `Recorded` is `reconstructed` |
| `invalidation_missing` | Thesis | no invalidation written, or it is not falsifiable |
| `setup_confirmation` | Process | entry preceded the strategy's confirmation step |
| `regime_gate` | Process | opened under `restrictive` at full size, or under `cash-only` |
| `stop_missing` | Process | no stop written before entry |
| `stop_change_rule` | Execution | a stop moved against the position with no pre-written rule |
| `position_size` | Risk | realized risk exceeded the written 1R or the per-trade limit |
| `portfolio_heat` | Risk | heat at entry exceeded `max_portfolio_heat_r` |
| `weekly_loss` | Risk | the week's realized loss breached `max_weekly_loss_r` |
| `correlation_cluster` | Risk | the lot concentrated exposure already held elsewhere |
| `entry_vs_plan` | Execution | the fill deviated materially from the written trigger |
| `exit_vs_plan` | Execution | the exit deviated from the written target, stop, or trim plan |
| `add_trim_unplanned` | Execution | size changed with no rule and no `AMEND` reason |
| `record_incomplete` | Record | a field the audit needed was blank |
| `other` | any | anything else, with a short phrase |

Severity comes from the next section, not from the topic: `position_size` is
`warning` when realized risk exceeded plan by up to 25% and `critical` beyond
that; `portfolio_heat` is `warning` when near the limit and `critical` past it;
a topic with evidence too thin to call is `warning`, never `critical`.

## Evidence standard

Every finding and every tag cites one of: a field comparison
(`realized risk 1.8R vs 1R written`), a quoted phrase from the lot's own notes, a
named missing record, or an `AMEND` block by date. No citation, no finding. When
evidence is unclear, write `unclear` — never round it up to a conclusion.

## Behavior tags

Hypotheses about repeated behavior, not psychological claims. Always
`possible pattern`, always with evidence, a confidence (`low`/`medium`/`high`),
and one reflection question. Never infer personality or mental state. Never
moralize. A tag with thin evidence is `low`, or is omitted.

### General

| Tag | Evidence | Reflection question |
| --- | --- | --- |
| `fomo_entry` | entry before confirmation; chased above the written trigger | What did you know before entry that waiting would not have told you? |
| `revenge_trade` | opened within a session of a loss; size above average after a loss | Would you have taken this if the prior trade had won? |
| `premature_exit` | `discretionary` exit before target or stop, no invalidation noted | What written rule did the exit satisfy? |
| `overconfidence_after_winner` | risk above plan following a winner | Did the prior win change your size or your quality threshold? |
| `stop_moved` | `AMEND` moved the stop against the position, unplanned | Was the new stop part of the plan before entry? |
| `size_creep` | realized risk > written 1R, explicitly compared | What would this have looked like at the planned size? |
| `breakeven_exit_only` | the only exit order placed sits at or above the entry price, with no stop below it | What price were you willing to sell at if it did not come back? |
| `hesitation` | planned entry missed, then entered late or not at all | What would have made the trigger automatic? |
| `exposure_stacking` | a second same-direction lot opened minutes to hours after the first, while the first is under water and with no written plan covering the addition | Was the second position planned before the first went against you, or decided while it was moving? |
| `rule_drift` | three or more minor deviations across recent lots | Which single rule is worth enforcing next week? |
| `unknown_size_discipline` | 1R or realized risk missing | Record both next lot so this becomes checkable. |
| `hindsight_inflation` | `reconstructed` lots grading better than `live` ones | Is the log recording decisions, or outcomes? |
| `no_pattern_detected` | clean record, no supporting evidence | — |

### Options-specific

These are the ones an equity-shaped review misses. Most options damage is here.

| Tag | Evidence | Reflection question |
| --- | --- | --- |
| `iv_blindness` | long premium opened at IVR > 60, or short premium at IVR < 30 | Were you paid for the volatility you bought or sold? |
| `theta_denial` | held past the written time stop with DTE < 21 | What was the thesis for the days after the time stop? |
| `catalyst_drift` | event-driven lot held past its event with no new written thesis | The event happened — what is the thesis now? |
| `gamma_week_hold` | held into expiry week, **or opened at 0–2 DTE**, without a written plan for it | What was the plan for the last days, written when? |
| `roll_to_avoid_loss` | `AMEND` rolled out or down with no new thesis, after the stop level | Is this a new position, or the old one not taken off? |
| `assignment_surprise` | `assignment` exit with no assignment plan in the stamp | Was assignment an accepted outcome or an accident? |
| `spread_leg_out` | one leg closed discretionarily, breaking the structure | What did legging out do to the position's max loss? |

`breakeven_exit_only` describes an order, not a feeling, and the order is on the
record: an exit at the entry price is neither a target nor a stop. It can fill
only if the position is made whole, so the outcomes it permits are scratch, or
maximum loss, with nothing in between. Read it alongside the entry: a position
that had no stop when it was opened has not acquired one by having a
breakeven limit placed on it later.

On a 0–2 DTE contract the tag carries more weight. Theta means the underlying has
to move further with every hour just to hold the option's price flat, so a
breakeven limit becomes progressively harder to reach as the session runs. On
those lots the order is close to no exit at all, and the position is effectively
unmanaged from the moment it is placed.

Size decides the damage, not the pattern. The same order on a larger position
converts the same adverse move into the same total loss.

`exposure_stacking` is not the same as a planned scale-in. Adding into weakness
on a schedule written before entry is a strategy; adding because the first lot is
moving against you is a decision made under pressure. The fills look identical,
and only a plan written beforehand separates them. Record the tag on evidence of
timing and absence of plan, never on an assumption about motive.

`roll_to_avoid_loss` deserves the closest reading. A roll that keeps a thesis
alive and a roll that refuses to realize a loss look identical in the fills and
are distinguished only by whether a new thesis was written at the time. That is
why `AMEND` blocks require a reason and are never backdated.

### Text patterns

Boolean and numeric evidence above is primary and is what most tags rest on.
Note text is a secondary signal. Match case-insensitively:

- `fomo_entry` — `didn't want to miss`, `fomo`, `chase`, `moving fast`, `不想错过`, `追`
- `revenge_trade` — `make it back`, `win it back`, `revenge`, `赚回来`, `扳回`
- `premature_exit` — `got scared`, `took it off early`, `couldn't hold`, `拿不住`, `怕了`
- `overconfidence_after_winner` — `easy money`, `can't lose`, `sure thing`, `稳了`, `躺赚`
- `hesitation` — `hesitated`, `waited too long`, `犹豫`, `没敢`

A text match alone is `low` confidence. Text plus a field or flag is `medium`
or `high`. Never tag on text alone when the fields contradict it.

## Operating rules

An audit that finds something produces temporary guardrails, appended to
`trades/_operating-rules.md`. Good rules are observable, time-boxed, tied to a
named finding, and simple enough to follow before the next entry.

- "Next two lots: max risk 0.5R." — trigger `size_creep`
- "No entry without a written invalidation." — trigger `fomo_entry`
- "Review-only for one session." — trigger `COOL_DOWN`
- "Any stop change written before entry or not at all." — trigger `stop_moved`

Cap at three active rules. More than three is not a plan.

Every audit that proposes rules ends with a decision gate — the rules are
proposed, never imposed:

```text
accept / modify / defer / log-only     (default: log-only)
```

`STAMP` mode reads `_operating-rules.md` before writing a new entry stamp and
flags in the report any active rule the new lot appears to breach. This is the
only thing that makes the loop real: rules that are written but never read at
entry time change nothing.

## Boundary

The audit reports what the record shows. It does not recommend entering,
exiting, sizing, or holding anything, does not forecast, does not offer therapy
or diagnosis, and does not shame the user for a loss or a broken rule. Rule
language stays objective: "realized risk was 1.8R against a written 1.0R" — not
"this was reckless".
