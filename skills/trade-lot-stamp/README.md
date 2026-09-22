# Trade Lot Stamp

A Claude skill that keeps a short, timestamped record of every trading lot and
audits the decision process when the lot closes. Built for options, built for
复盘.

One file per lot: an **entry stamp** when you open, an **exit stamp** when you
close, an **audit** graded across five axes, and dated **amendments** in between.
Nothing is ever overwritten.

It documents and audits trades. It does not recommend trades, predict prices, or
tell you what to buy.

## Install

The skill is a folder of Markdown files. Put it in one of these places:

```bash
# Personal — available in every project
mkdir -p ~/.claude/skills
cp -r skills/trade-lot-stamp ~/.claude/skills/

# Or project-scoped — available only inside one directory
mkdir -p ~/trading/.claude/skills
cp -r skills/trade-lot-stamp ~/trading/.claude/skills/
```

Don't have the repo locally:

```bash
git clone https://github.com/silviasistar/skill-make-photo-stamp-archive.git
cp -r skill-make-photo-stamp-archive/skills/trade-lot-stamp ~/.claude/skills/
```

For the claude.ai web or desktop app, zip the `trade-lot-stamp` folder and upload
it in Settings under skills/capabilities.

There is nothing to build, install, or run. Editing the skill means editing
Markdown in any text editor.

## Set up your log

The skill lives in `~/.claude/skills/`. **Your trades live somewhere else** —
wherever you want to keep them. The skill is the instruction manual; `trades/` is
your ledger.

```bash
mkdir -p ~/trading/trades
cd ~/trading
```

Write your account limits once, to `trades/_risk-plan.yaml`. The audit grades
risk against this file; without it, Risk grades as `—` and no rule violation can
be asserted.

```yaml
max_risk_per_trade_r: 1.0      # max 1R on any single lot
max_portfolio_heat_r: 4.0      # max summed open risk across all lots
max_weekly_loss_r: 3.0         # realized weekly loss that triggers cool-down
max_consecutive_losses: 4
regime_gate:
  allowed: [risk-on]
  restrictive: [chop]          # reduced size only
  cash_only: [risk-off]
require_setup_confirmation: true
require_stop_before_entry: true
```

Use your real numbers. These are placeholders, not recommendations.

## Use it

Start Claude Code from the directory holding `trades/`, then talk normally. The
skill triggers on what you say — there is no command to memorize.

```bash
cd ~/trading
claude
```

### Opening a position

> bought 5 NVDA Oct 17 180 calls at 4.20, underlying 174.30, IV 48 IVR 61,
> earnings Nov 19. target 6.30 then 8.40, stop 2.10, out by Oct 3 if it's
> still under 178

Claude fills in what it can derive (cost, breakeven, DTE, R, % of book), looks up
what is checkable, and asks once for whatever is left. It checks
`trades/_operating-rules.md` first and tells you if an active rule covers this
lot. Then it writes `trades/2026-09-22-NVDA-long-call.md` and adds a row to
`trades/_ledger.csv`.

### Closing

> closed the NVDA calls at 6.80

It appends the exit stamp, computes P&L in dollars, percent, and R, then audits
the lot: five axis grades, a root cause, a verdict, behavior tags with evidence,
and at most three time-boxed operating rules — which it proposes, never imposes.

### Changing a plan mid-trade

> rolled the NVDA calls out to Nov 21, same strike

Appends a dated `AMEND` block. The original stamp is never edited. A roll always
records whether the thesis is unchanged, revised, or gone — that one line is what
separates a real roll from refusing to take a loss.

### Reviewing

> 复盘 this month
> review my event-driven trades
> how did my long-premium lots do this quarter

Aggregates the ledger and the audits. Counts behavior tags across lots, checks
which operating rules held and which broke, and reports process separately from
P&L.

## What it writes

```
trades/
├── _risk-plan.yaml                      you write this once
├── _operating-rules.md                  rules from audits, append-only, max 3 active
├── _ledger.csv                          one row per lot, 43 columns
└── 2026-09-22-NVDA-long-call.md         entry stamp + amendments + exit + audit
```

## How it grades

Five axes, A–F, **none of which are affected by P&L**. A winning trade taken
off-plan grades as a process failure; a losing trade taken on-plan grades as a
process success.

| Axis | Question |
| --- | --- |
| Record | was the evidence complete enough to trust this audit? |
| Thesis | was the idea clear, falsifiable, matched to the setup? |
| Process | did actual behavior follow the written plan? |
| Risk | did actual risk stay inside the written limits? |
| Execution | were entry, stop, adds, trims, and exit consistent with plan? |

Record is graded first and caps confidence in the rest.

Verdicts run `OK` → `WARN` → `REVIEW_REQUIRED` → `RULE_VIOLATION` → `COOL_DOWN`.
The `COOL_DOWN` triggers are mechanical — two criticals on one lot, four
consecutive losses, a breached weekly limit — so the decision to stop is made by
the record rather than in the moment.

`randomness` is a first-class root cause. When the plan was followed and the
setup was valid, the audit says so instead of manufacturing a lesson.

Seven behavior tags are options-specific and are checked on every options lot:
`iv_blindness`, `theta_denial`, `catalyst_drift`, `gamma_week_hold`,
`roll_to_avoid_loss`, `assignment_surprise`, `spread_leg_out`.

## Files

Split deliberately. `SKILL.md` loads every time, so it stays short; the
references are read only when that mode needs them.

| File | Read when |
| --- | --- |
| `SKILL.md` | always — modes, workflow, rules, guardrails |
| `references/stamp-template.md` | writing any stamp |
| `references/audit-framework.md` | grading a closed lot, or aggregating grades |
| `references/field-reference.md` | a field, formula, or vocabulary is unclear |
| `references/review-guide.md` | period, strategy, or ticker review |

## Known limits

- **Grades are judgment, not arithmetic.** There is no scoring script. The same
  lot audited twice can come back `B` on one axis and `C` on another run. Trends
  in the tags and verdicts are more trustworthy than a month-over-month grade
  comparison. A deterministic scorer for the mechanical checks (R comparisons,
  heat against limits, cool-down triggers) would fix this and is not written yet.
- **Portfolio heat has to come from you.** Nothing connects to a broker, so heat
  at entry is whatever you report. Reconstructed heat is a guess, and a guessed
  gate makes every Risk grade downstream of it meaningless.
- **Market data is only as good as what you supply or what it can look up.** It
  will leave a field blank rather than invent a price, a greek, or a headline,
  and it tells you which fields it left blank.
- **`reconstructed` stamps are weaker evidence, by design.** A thesis written
  after the outcome is known is hindsight-contaminated; the audit caps its Thesis
  and Record grades and says so.

## Not financial advice

This is a record-keeping and process-review tool. It does not recommend entering,
exiting, sizing, or holding any position, does not forecast prices, and does not
offer therapy or diagnosis. Every trading decision and every risk remains yours.
