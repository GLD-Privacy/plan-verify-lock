# Example: revising a Bill of Quantities without breaking it

Illustrative walkthrough. The figures are invented to show the method.

## Task
Landscape BOQ with three bills: U1 Softscape, U2 Hardscape, U3 Irrigation. U4 Summary depends on U1-U3.

## Plan (excerpt)
| ID | Unit | Acceptance criteria | Depends on |
|---|---|---|---|
| U1 | Softscape | Quantities match planting takeoff; every item has a unit; amounts = qty x rate exactly | - |
| U2 | Hardscape | Areas match paving takeoff within 1% | - |
| U3 | Irrigation | Pipe lengths match layout; emitter count = plant count | U1 |
| U4 | Summary | Equals sum of U1-U3 | U1, U2, U3 |

## Revision r1
Feedback: "The paving rate in U2 is wrong; it should be 18.50, not 15.80."

1. **Verify the feedback.** Checked against the supplier quote: confirmed.
2. **Scope it.** U2 is edited. U4 depends on U2, so its lock is invalidated. U1 and U3 stay locked.
3. **Edit surgically.** One cell changes in U2. Nothing else is regenerated.
4. **Regression check.** Diffing U1 and U3 against their snapshots shows no change. U2 and U4 are recomputed, and the totals reconcile.
5. **Report.** "Paving rate corrected in U2 and summary updated. U1 and U3 are unchanged (diff clean)."

## What happens without the skill
The agent rewrites the whole BOQ to apply one rate. In the rewrite, a quantity in U1 drifts, an item unit changes from m2 to m, and the summary no longer reconciles. The user finds out later.
