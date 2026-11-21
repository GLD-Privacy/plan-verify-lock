# Ledger (append-only)

File mode: append rows only with `.pvl/append`, row on stdin, without its number (the script numbers it). Inline mode: end each reply with this table, every earlier row copied unchanged, new rows numbered on. Never edit, reorder or delete a row. A unit's state is its latest LOCK, FREEZE or UNLOCK row. Write `|` inside a cell as `\|`.
Events: PASS | FAIL | NOT-RUN | EDIT | LOCK | FREEZE | UNLOCK | REVERT | REGRESSION | BLOCKED | FEEDBACK | VERDICT (ACCEPTED / CONFIRMED / REFUTED / UNRESOLVED) | USER-DECISION | CRITERION | UNIT
Evidence: MACHINE `<command>` exit <n> "<output or evidence/<#>.txt>" | SOURCE <doc, page/line> "<verbatim>" | RENDER <file> `<tool command>` "<tool output>" | HUMAN "<user's words>"
EDIT rows: refs cite a VERDICT or USER-DECISION once the unit has been locked or frozen; note is `targets: <key/field>(<n>) ...` written before the step runs (n defaults to 1; wildcards only for new keys). REGRESSION rows: `.pvl/regress` exit code and output file. UNIT rows: cite the verdict and the equivalence check on the base.
A FAIL after a FAIL (same unit and criterion) cites the earlier FAIL. A PASS after FAILs cites every FAIL since the last PASS and states the change in note. A CONFIRMED verdict shows identity (same item, size, unit, edition) and support. LOCK cites the snapshot, the raw copy and a PASS for every criterion of the unit. FREEZE cites its reason (baseline, NOT-RUN rows, USER-DECISION or UNIT row). BLOCKED and UNRESOLVED stay open until a later row cites them and closes them.
Worked example: `examples/boq-revision.md`.

| # | rev | unit | ver | event | criterion | evidence | refs | note |
|---|---|---|---|---|---|---|---|---|
