---
name: plan-verify-lock
description: Use for any multi-part deliverable (document, BOQ, spec, report, design, code) and whenever revising one after feedback: plan with acceptance criteria, verify, lock correct parts, fix without regressions.
---

# Plan - Verify - Lock

Purpose: stop the failure mode in which fixing one part of a deliverable silently degrades parts that were already correct. Method: an explicit plan with testable acceptance criteria, verification before delivery, a ledger of locked (verified) units, and regression checks on every revision.

## 0. Scale the process to the task
- Trivial (one short answer, one small edit): skip this skill.
- Medium (several parts, one deliverable): compact plan (5-10 lines), ledger, regression check.
- High-stakes (money, contracts, safety, client-facing): full plan, risk register, independent verification pass.

## 1. Plan (before producing anything)
Write `plan.md` in the working directory (template: `templates/plan.md`):
1. **Goal**: one sentence stating the final deliverable and who uses it.
2. **Units**: break the deliverable into units with stable IDs (U1, U2...), e.g. sections, BOQ bills, drawings, functions.
3. **Acceptance criteria per unit**: each must be checkable, e.g. "quantities match takeoff within 1%", "every clause cites a current standard", "totals recompute exactly". Vague criteria such as "looks good" are not allowed.
4. **Dependencies**: which units feed which. A change upstream invalidates downstream locks.
5. **Risks**: top 3-5 ways this could go wrong, each with its check.
6. **Flow with gates**:
```
PLAN -> BUILD unit -> VERIFY unit --fail--> fix unit (max 3 tries, then flag)
                         |pass
                         v
                       LOCK unit -> next unit -> ... -> FINAL CHECK (all criteria + regression) -> DELIVER
```

## 2. Build and verify each unit
- Verify with real checks, not by rereading: recompute arithmetic in code, cross-check against source documents, render or open the file, run the code.
- Record the evidence in the ledger. A unit without evidence is not verified.

## 3. Lock
Keep `ledger.md` (template: `templates/ledger.md`):

| Unit | Status | Criteria checked | Evidence | Snapshot |
|---|---|---|---|---|
| U1 | LOCKED | qty, units, totals | recalc script OK | snapshots/U1_v1 |

- Save a snapshot of every locked unit (a copy of the section or file, or a hash).
- Status values: DRAFT, VERIFIED, LOCKED, UNLOCKED(reason).

## 4. Revise after feedback (the core rule)
1. **Verify the feedback first.** Check the claimed error against evidence.
   - Confirmed: proceed.
   - Refuted: keep the output and show the evidence.
   - Unclear: ask.
   - Never accept a correction blindly. Never dismiss one without evidence. Refuting needs stronger evidence than accepting.
2. **Scope it.** Name the affected unit IDs. Everything else is out of scope.
3. **Edit surgically.** Change only the affected units, using targeted edits (find/replace, diff-level edits). Never regenerate the whole deliverable to fix a part.
4. **Regression check.** After the edit, compare every LOCKED unit against its snapshot (diff) and rerun its criteria checks.
   - A locked unit changed unintentionally: revert that change.
   - A downstream dependent of an edited unit: re-verify it, since its lock is invalidated.
5. **Unlock only with a reason.** If a correct fix genuinely requires changing a locked unit, mark it UNLOCKED(reason), state this to the user, re-verify, and re-lock. For high-stakes work, ask before unlocking.

## 5. Final gate before delivery
- All units LOCKED, all criteria passed with evidence, regression check clean.
- Report to the user in one or two lines: what was verified, what was changed in this revision, and any residual risk or unverifiable item.
- If a check could not be run, say so explicitly. Do not imply it passed.

## Anti-patterns
- Rewriting the whole document or file to fix one paragraph or line.
- Claiming "fixed" without rerunning the checks.
- Silent edits to locked units, including formatting or wording drift.
- Acceptance criteria written after the work to match what was produced.
