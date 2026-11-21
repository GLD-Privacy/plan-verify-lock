# Example: revising a Bill of Quantities without breaking it

Illustrative walkthrough. The figures and hashes are invented to show the method.

## Task
A landscape BOQ in `boq.xlsx`, at high stakes (priced). Units: U0 shared layout, styles and raw-hashed parts (logo image, chart); U1 Softscape; U2 Hardscape; U3 Irrigation (depends on U1); U4 Summary (formulas over U1-U3). Keys: the item ref (2.07), with fields named by column header (`desc`, `unit`, `rate`, `amount.formula`, `~amount`); summary lines are keyed by label. Editing tool: openpyxl failed stability check (a), because the chart part disappeared from U0. The tool chosen is direct edits to the XML parts. Recompute: `.pvl/recalc` (LibreOffice headless).

## Criteria (excerpt)
| ID | Unit | Criterion | Check | Evidence |
|---|---|---|---|---|
| C2.1 | U2 | Areas match paving takeoff within 1% | `python .pvl/chk/takeoff.py U2` | MACHINE |
| C2.2 | U2 | Every amount = qty x rate exactly | `python .pvl/chk/amounts.py U2` | MACHINE |
| C4.1 | U4 | Each bill total = sum of its bill; grand total = sum of bills (U4's only criterion) | `python .pvl/chk/summary.py` | MACHINE |

## Revision r1
Feedback: "The paving rate in U2 is wrong; it should be 18.50, not 15.80." Rows 1-20 are the r0 build. Base: raw/a5c3e810/, cited by row 20, the last r0 LOCK.

| # | rev | unit | ver | event | criterion | evidence | refs | note |
|---|---|---|---|---|---|---|---|---|
| 21 | r1 | U2 | 3f9a01c2 | FEEDBACK | - | HUMAN "The paving rate in U2 is wrong; it should be 18.50, not 15.80." (user) | - | factual claim |
| 22 | r1 | U2 | 3f9a01c2 | VERDICT | - | MACHINE `.pvl/extract U2 \| grep -P '^2\.07\t(desc\|unit\|rate)\t'` exit 0 "2.07 desc Interlock paving 80mm grey; 2.07 unit m2; 2.07 rate 15.80" + MACHINE `pdftotext quote.pdf - \| grep -in 'interlock paving'` exit 0 "41: Interlock paving 60mm grey m2 15.80; 42: Interlock paving 80mm grey m2 18.50" -> identity: 2.07 is 80mm grey m2 = quote line 42; support: 18.50 -> CONFIRMED | 21 | the 60mm line (15.80) is a different item |
| 23 | r1 | U2 | 3f9a01c2 | CRITERION | C2.3 | - | 22 | new: every rate = the quote line with the same desc and unit; C2.1 and C2.2 do not test rates |
| 24 | r1 | U2 | 3f9a01c2 | FAIL | C2.3 | MACHINE `python .pvl/chk/rates.py U2` exit 1 "item 2.07: 15.80 != 18.50 (quote line 42)" | 23 | - |
| 25 | r1 | U2 | 3f9a01c2 | UNLOCK | - | - | 22 | E |
| 26 | r1 | U4 | 91d0b7ee | UNLOCK | - | - | 25 | D: depends on U2 |
| 27 | r1 | U2 | - | EDIT | - | MACHINE `python .pvl/edits/27.py` exit 0 "item 2.07 rate 15.80 -> 18.50" | 22 | targets: 2.07/rate(1) |
| 28 | r1 | - | - | REGRESSION | - | MACHINE `.pvl/recalc && .pvl/regress` exit 0 "ok U2: replace 2.07/rate (2.07/rate); ok U2: replace 2.07/~amount (derived); ok U2: replace total/~amount (derived); ok U4: replace Hardscape/~value (derived); ok U4: replace Grand total/~value (derived)" (U0, U1, U3: no line changed) evidence/28.txt | 27 | - |
| 29 | r1 | U2 | 7c1e44d0 | PASS | C2.1 | MACHINE `python .pvl/chk/takeoff.py U2` exit 0 "max deviation 0.4%" | - | rerun at new ver |
| 30 | r1 | U2 | 7c1e44d0 | PASS | C2.2 | MACHINE `python .pvl/chk/amounts.py U2` exit 0 "0 mismatches" | - | rerun at new ver |
| 31 | r1 | U2 | 7c1e44d0 | PASS | C2.3 | MACHINE `python .pvl/chk/rates.py U2` exit 0 "0 mismatches" | 24 | 2.07 rate 15.80 -> 18.50 (row 27) |
| 32 | r1 | U2 | 7c1e44d0 | LOCK | - | snap/U2@7c1e44d0, raw/b41e09a7/ | 29,30,31 | all criteria PASS |
| 33 | r1 | U4 | 5be2a913 | PASS | C4.1 | MACHINE `python .pvl/chk/summary.py` exit 0 "4 totals reconcile" | - | recomputed from new U2 |
| 34 | r1 | U4 | 5be2a913 | LOCK | - | snap/U4@5be2a913, raw/b41e09a7/ | 33 | all criteria PASS |

Had item 2.07 been described as 60mm, the identity check would have matched quote line 41 at 15.80, which contradicts the claim. With the 80mm reading also tested and failing, the verdict would be REFUTED, shown to the user, with no edit.

Suppose edit 27 had also overwritten the Summary label, or put a constant in place of `=SUM(...)`. Then row 28 would read `REGRESSION U4: replace Hardscape/label` (or `.../formula`) with exit 1, because U4 is in D and may change only `~` lines. If it had rewritten every rate in U2, row 28 would read `REGRESSION U2: replace 2.01/rate` and so on, because only `2.07/rate` was declared. The agent restores from raw/a5c3e810/, replays a fixed step 27, appends REVERT, and reruns the check. A second exit 1 would mean BLOCKED and a question to the user.

Final gate: each file-mode check ran once, and all pass, including `cmp -s .pvl/ledger.md .pvl/ledger.seal` exit 0.

Report: `Verified: 4 criteria on U2, U4 (MACHINE); U0, U1, U3 unchanged line by line (regress). Changed: U2 item 2.07 rate 15.80 -> 18.50; U4 totals recalculated. Failed before passing: U2 (FAIL #24 -> PASS #31, rate not checked against quote; criterion C2.3 added). Not run / residual risk: none.`

## Revision r2 (scope added)
Feedback: "Add a landscape lighting bill." It is a preference (scope), so it is ACCEPTED (HUMAN), row 36. The working files hash to the base raw/b41e09a7/. The agent copies the extractor to `.pvl/extract.37`, adds U5 Lighting (locator: "Lighting sheet, item refs 5.01 to 5.99", empty for now) and makes U4 depend on U5. The stability check runs, then the equivalence check on raw/b41e09a7/: U0-U4 extract exactly as their latest snapshots, and U5 is empty (row 38, MACHINE, exit 0). Row 39 is the UNIT row, citing 36 and 38; row 40 is a FREEZE of U5 (empty, cites 39). U0 is unlocked for the new sheet's entry in the workbook (`targets: sheets/Lighting(1)`), U4 for its new SUM term (`targets: Lighting/formula(1) Grand total/formula(1)`), and U5 is built with `targets: 5.*/*`, a wildcard that may only create new keys. Without the UNIT row, every new line would have fallen into locked U0 and been flagged as a regression.

## What happens without the skill
The agent rewrites the whole BOQ to apply one rate. In the rewrite, a quantity in U1 drifts, an item unit changes from m2 to m, the chart is lost on save, and the summary no longer reconciles. The user finds out later.
