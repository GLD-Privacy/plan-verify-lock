# Plan        mode: file        stakes: <standard|high>        revision: r0

**Goal:** <deliverable, who uses it>
**Whole-deliverable acceptance test:** <one checkable statement>
**Files:** <paths>   **Editing tool:** <patch | openpyxl | python-docx | direct XML part edit | ...>
**Extractor:** `.pvl/extract [--root <dir>] <unit> | --hashes`   **Depth:** <standard: converter used | high: full>   **Recompute:** <command | none>
**Stability check:** (a) save round-trip <row #>   (b) structural probe in <unit> <row #>
**Derived (`~`) fields:** <e.g. cached formula values, TOC, page numbers, dimension>
**Raw-hashed in U0 (not interpreted):** <parts and elements, e.g. xl/charts/*, xl/media/*, conditionalFormatting>
**Unit changes:** <UNIT row # : what changed; old extractor kept at .pvl/extract.<row#>> | none

## Units (build in this order; dependencies must be acyclic)
| ID | Unit | Locator (by anchor, never address) | Key (anchor) | Depends on | Not covered by extract |
|---|---|---|---|---|---|
| U0 | Everything not in another unit | <rule> | <part / element name> | - | <property / none> |
| U1 | | | | - | |
| U2 | | | | U1 | |

## Criteria (written before building; change only by a CRITERION row in the ledger)
| ID | Unit | Criterion (pass/fail, measurable) | Check (command / source / render+tool / user) | Evidence type |
|---|---|---|---|---|
| C1.1 | U1 | | | MACHINE |

## Risks (required at high stakes)
| Risk | Impact | Caught by criterion |
|---|---|---|
| | | |

## Flow
```
PLAN -> STABILITY CHECK (round-trip + structural probe) --(high stakes: user approves)-->
for each unit in dependency order:
  BUILD (EDIT steps) -> RECOMPUTE -> REGRESS --exit 1--> restore from base, REVERT (2nd: BLOCKED)
  -> CHECK --FAIL, <3 fails--> FIX unit -> CHECK
          --FAIL, 3rd-------> BLOCKED -> ask user --accept--> USER-DECISION -> FREEZE
          --all PASS--------> SNAPSHOT + RAW COPY + LOCK -> next unit
          --any NOT-RUN-----> SNAPSHOT + RAW COPY + FREEZE (unverified) -> next unit
ALL UNITS -> FINAL GATE (hashes = latest LOCK/FREEZE; locked criteria PASS; last REGRESS exit 0;
             nothing open; high stakes: no unaccepted NOT-RUN or freeze; seal)
             --pass--> DELIVER + 2-line report   --fail--> back to the failing unit's stage

REVISION: [BASELINE: plan + stability check + snapshots + raw copy + FREEZE, if outside work]
  -> FEEDBACK -> CLASSIFY
  preference (user) -> ACCEPTED (conflict? ask once -> USER-DECISION)
  factual -> JUDGE --CONFIRMED (identity + support)--> CRITERION + FAIL
                   --REFUTED----> show evidence, no edit; user may decide (USER-DECISION)
                   --UNRESOLVED-> ask user; no edit until VERDICT or USER-DECISION
  [scope added / unit split: files = base -> new extractor -> stability + EQUIVALENCE on base
                             -> UNIT row -> snapshots + FREEZE for new units]
  -> SCOPE E + dependents D -> UNLOCK E, D
  -> EDIT rows (targets with counts, cite verdict) -> steps on E only -> RECOMPUTE
  -> REGRESS: locked/frozen = snapshot; unlocked: ~ lines free, authored lines only in targets,
              within declared counts; wildcards only on new keys
       exit 1 -> base + kept steps + recompute, REVERT, rerun (2nd exit 1: BLOCKED)
  -> CHECK all criteria of E, D -> SNAPSHOT + RAW COPY + LOCK/FREEZE -> FINAL GATE
```
