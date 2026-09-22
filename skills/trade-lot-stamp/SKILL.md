---
name: trade-lot-stamp
description: Record a short timestamped archival log entry for each trading lot — an entry stamp when the position opens and an exit stamp when it closes — capturing market sentiment, underlying price and key levels, event catalyst and company news, options contract and greeks, exit price targets, stop, invalidation, and strategy, then grade process separately from outcome so the log is reviewable. Use for requests mentioning 交易日志、复盘、建仓、平仓、加仓、减仓、持仓记录、期权、期权日志、止损、目标价, trade log, trade journal, trade stamp, lot record, entry and exit log, options trade, position record, catalyst or event-driven trade, post-trade review, or amending, closing, or reviewing a previously stamped lot.
---

# Trade Lot Stamp

Press one short, dated record per trading lot: an **entry stamp** at open, an **exit stamp** at close, both in the same file. Optimized for options and for later 复盘.

Read [references/stamp-template.md](references/stamp-template.md) before writing any stamp.
Read [references/field-reference.md](references/field-reference.md) when a field's meaning, an options input, or a sentiment or catalyst classification is unclear.
Read [references/review-guide.md](references/review-guide.md) only for review mode.

This skill **documents** trades. It never recommends entering, exiting, sizing, or holding anything, and it never predicts a price.

## Modes

Determine the mode from the request before doing anything else.

| Mode | Trigger | Action |
| --- | --- | --- |
| `STAMP` | a new position was opened | create a new lot file with an entry stamp |
| `EXIT` | a position was closed or trimmed | append an exit stamp to the existing lot file |
| `AMEND` | plan changed while the lot is open (rolled, stop moved, target changed, added) | append a dated amendment line; never rewrite history |
| `REVIEW` | 复盘 over a period, a strategy, or a ticker | read the ledger and lot files; follow `references/review-guide.md` |

## Workflow

1. **Resolve the mode** and, for `EXIT`/`AMEND`/`REVIEW`, locate the existing lot file under `trades/` by ticker and open status. If more than one open lot matches, list them and ask which.
2. **Fix the timestamp honestly.** Every stamp carries a decision time with a timezone (use ET for US markets) and a recording tag:
   - `live` — written at or near the decision;
   - `same-day` — written later the same session;
   - `reconstructed` — written after the fact from memory or a broker statement.
   Mark `reconstructed` whenever the thesis is being written after the outcome is known. A reconstructed thesis is hindsight-contaminated and must be labeled so review does not trust it as a real-time prediction.
3. **Collect the facts.** Take whatever the user supplies — a broker fill, a screenshot, a sentence. Then fill gaps in this order:
   - derive what is derivable (cost basis, breakeven, DTE, max loss, R, percentages);
   - look up what is checkable and cite it (underlying price, VIX, IV rank, earnings date, a news headline with its source and date);
   - ask, in **one** batched question, only for what is neither derivable nor checkable — typically thesis, targets, stop, conviction, and position size.
   Never invent a number, a headline, a greek, or a price level. Leave a field as `—` rather than guess, and say which fields are blank.
4. **Keep it short.** The entry stamp is one screen. Thesis is one or two sentences and must be falsifiable. Every table row is a fact or a number, not prose.
5. **Write the file** at `trades/<YYYY-MM-DD>-<TICKER>-<strategy-slug>.md`, appending `-2`, `-3` for repeat same-day lots on the same ticker. Assign the lot ID `<TICKER>-<YYYYMMDD>-<n>`.
6. **Update the ledger** at `trades/_ledger.csv`, creating it with the header from `references/field-reference.md` if absent. One row per lot; the `EXIT` mode fills in the close columns of that same row. Partial exits add a child row with the parent lot ID.
7. **Report** the saved path, the lot ID, and any field left blank.

## Rules

### Entry stamp

- Record the **underlying** price at entry, not only the option price. Both are needed for 复盘.
- Classify the trade as `[EVENT-DRIVEN]` or `[SETUP-DRIVEN]` in the catalyst line. If event-driven, name the event, its date, and a source; if the date is scheduled and known (earnings, FDA, CPI, an investor day), record it — it drives the time stop.
- The plan table is mandatory and must state, at minimum: target 1, stop, and invalidation. A lot stamped without a stop is stamped with `Stop | — (NONE SET)` and flagged in the report, not silently left blank.
- Express risk in **R**: R is the planned max loss on the lot. Record max loss in both currency and percent of book, so outcomes are comparable across sizes.
- For options, always record contract, expiry, strike, right, quantity, net debit or credit, DTE, IV and IV rank, and delta. Record breakeven at the underlying. For multi-leg positions, list each leg, then the net.

### Exit stamp

- Record actual fills, not intended ones. Partial exits get their own dated exit block; the lot stays `OPEN` until the last contract is closed.
- Compute P&L in currency, percent, and **R multiple**. R multiple is the honest unit — a +$400 win on a 2R risk is worse than a +$300 win on a 0.5R risk.
- State the exit reason using one tag from the field reference (`target`, `stop`, `time-stop`, `invalidation`, `discretionary`, `assignment`, `expiry`) plus one clause of detail.

### 复盘 review block

- Grade **process** and **outcome** on separate lines, always. A winning trade taken off-plan is a process failure, and a losing trade taken on-plan is a process success. Never let the P&L set the process grade.
- Process grade answers only: was the plan followed, was the size right, was the stop honored, was the thesis actually tested?
- Close with exactly one transferable rule for next time, in one line. Not three.
- Keep the review factual. Write what happened and what deviated, not encouragement.

## Guardrails

- Never fabricate market data, news, greeks, or fills. Unknown is `—`.
- Never rewrite or delete a previous stamp. Corrections are appended, dated, and labeled `AMEND`.
- Never turn a `reconstructed` stamp into a `live` one.
- Never infer a thesis the user did not state; ask for it.
- Never let the review section grow past a screen, and never pad it with generic trading advice.
- Never emit a recommendation, a price forecast, or a "you should have" judgement.
- Match the user's language. If they write Chinese, write the prose and the review in Chinese and keep the field labels and tags as they appear in the template.
