---
name: plan-verify-lock
description: Use for multi-part deliverables (documents, BOQs, specs, reports, designs, code) and for revising them after feedback: plan, verify with evidence, lock correct parts, fix without regressions.
---

# Plan, Verify, Lock

Goal: a fix to one part never damages parts that are already correct. Split the work into units. Check each unit with external evidence. Keep raw copies of correct units. After every edit, compare every unit with its snapshot by machine, and rebuild from the raw copy whenever a change was not declared.

## Definitions
- **Unit**: the smallest part you can check on its own. Each unit has a stable ID (U1, U2, ...) and a **locator**: a rule that selects the unit by anchors, never by addresses. Examples: "Hardscape sheet, item refs 2.01 to 2.99"; "clause 4.1 up to clause 4.2"; "function `parse_rates`". **U0** is everything not in another unit, so the whole deliverable is covered.
- **Extract**: the unit as lines of `key<TAB>field<TAB>value`, in document order, one line per item, cell, paragraph or entity. Inside a value, write a tab as `\t` and a newline as `\n`.
  - *key*: the nearest stable anchor, at the finest level the content has: item ref, clause number, heading path, function name, DXF handle. Never use a cell address, a row number or a page number. Several lines may share a key.
  - *field*: the property, for example `rate`, `amount.formula`, `style`, `text`. A field the format's recompute produces starts with `~` and is **derived**. Examples: a formula's cached value, a TOC entry, an auto-number, a page number, a page render hash. Every other field is **authored**.
  - Formulas: write every reference as the `key/field` it points to, and every range as `first:last` (`=SUM(2.01/amount:2.14/amount)`). Write merges, print areas and defined names the same way.
  - **Depth by stakes.** At standard stakes, an existing converter is enough: text and markdown split by heading; docx via `pandoc -t markdown`, one line per paragraph or table cell; xlsx via a short openpyxl dump of values, formulas and number formats per item; DXF via `ezdxf`, one line per entity handle. At high stakes, also extract resolved properties: for xlsx, font, fill, border and alignment, with shared-string and style indices resolved; for docx, the resolved style, numbering level and run properties, dropping save noise (`w:rsid*`, `proofErr`, `lastRenderedPageBreak`); for PDF, a render hash per page as a `~` field.
  - **Completeness**: anything the extractor does not read, it hashes raw into U0. That means every package part it does not parse (charts, drawings, media, macros, comments). At high stakes it also means every top-level XML element it does not interpret (for example `conditionalFormatting`, `dataValidations`). Elements that a save rewrites (`dimension`, `calcChain`) are `~`. List every visible property the extract does not capture under "Not covered" in the plan; the report names them.
- **Extractor**: `.pvl/extract [--root <dir>] <unit>` prints one unit's extract from the files in `<dir>` (default: the working files). `.pvl/extract [--root <dir>] --hashes` prints every unit's ver. Write it at planning time. After the stability check it changes only through a UNIT row (section 4, step 0).
- **ver**: the first 8 hex characters of the SHA-256 of a unit's extract.
- **Raw copy**: a byte copy of all deliverable files, stored together in `.pvl/raw/<id>/`. The `<id>` is the first 8 hex characters of the SHA-256 of `sha256sum <files> | sort`. A raw copy is write-once.
- **Snapshot** of a unit: its extract at `.pvl/snap/<unit>@<ver>`, plus the raw copy that contains it. Write-once. Never summarize, edit or delete it.
- **Base**: the raw copy cited by the highest-numbered LOCK or FREEZE row.
- **Edit step**: every change to a deliverable file is a saved, replayable step, and so is building a unit.
  - For text, the step is a patch. An edit made with your file-editing tool counts once its diff is saved as the step.
  - For other formats, it is a script that uses the format's library or edits the package's XML parts directly.
  - Save each step as `.pvl/edits/<row#>.<ext>` and log it with an EDIT row.
  - The EDIT row's note says `targets: <key/field>(<n>) ...`, written before the step runs. `n` is the number of lines the step may change under that target; it defaults to 1. A target with a shell-style wildcard (`2.08/*`) may only create keys that are not in the snapshot, in any number.
  - Never change a deliverable file through a GUI, or by regenerating it whole.
- **Evidence**: output not produced by your own judgement.
  - MACHINE: the command, its exit code and its output.
  - SOURCE: a verbatim excerpt from a named external document, with page or line. Prefer extracting it with a command (`pdftotext`, `grep`). That makes it MACHINE.
  - RENDER: a file produced by the real application, plus a fact read from that file by a tool, with the command. Examples: `pdfinfo` pages, `pdffonts`, the application's error log.
  - HUMAN: the user's explicit words.
  - Never evidence: your statement or visual impression that something is fine, your memory, your earlier output, or the disputed artefact itself.
- **Locked**: the unit's latest LOCK, FREEZE or UNLOCK row is a LOCK at its current ver. A LOCK requires a PASS on every criterion of the unit at that ver.
- **Frozen**: that latest row is a FREEZE. A frozen unit is snapshotted and protected, but not verified. A FREEZE cites its reason: a baseline of outside work, a NOT-RUN row, a USER-DECISION that accepts the unit at this ver, or a UNIT row.

## 0. Scale (decide before starting; scale up when the deliverable grows or a trigger appears, never down)
- **Trivial: skip this skill.** The deliverable is one unit (nothing else in it can be damaged), stakes are standard, and nothing else is built on it. Example: a one-line answer, "make it shorter", a typo. Only one rule applies: if someone says a fact is wrong, check it once with a command or source before changing it, and do not claim any check you did not run.
- **Inline mode**: the deliverable is text in the reply, short enough to reproduce in full in each reply (about one page).
- **File mode**: everything else (sections 1-5, under `.pvl/`).
- **Stakes** are **high** if the work has money, contract, safety or legal consequences, or if the user says so. At high stakes:
  - numeric and factual criteria need a PASS from MACHINE or SOURCE evidence;
  - the extractor has full depth (Definitions, Extract);
  - a NOT-RUN on a numeric or factual criterion, or a USER-DECISION freeze, blocks final delivery unless the user explicitly accepts it;
  - in file mode, the user approves the plan and a risk register before you build.

**Inline mode**: no files. The conversation is the store.
1. Units are the parts of the reply: line items, paragraphs, clauses, numbered `U1`, `U2`, ...
2. End every reply with a **Ledger** block. Copy every earlier row unchanged, then add the new rows, numbered on. Use the columns of `templates/ledger.md`. Log every FEEDBACK, VERDICT, PASS, FAIL, NOT-RUN and FREEZE. The section 3 rules on PASS after FAIL apply.
3. A unit is frozen when a FREEZE row names the reply that holds it; that reply is its snapshot. Nothing locks in inline mode unless a command ran every check.
4. In a revision, run section 4, steps 1-3, on every claim. Apply a factual correction only when it is CONFIRMED by evidence you show. If it is REFUTED or UNRESOLVED, do not edit; show the evidence and ask the user.
5. Change only the units that a CONFIRMED or accepted claim requires. Copy every other unit verbatim from the reply that is its snapshot, by quoting it, never from memory.
6. If you can run commands, `diff` the snapshot reply against the new one and confirm that every changed line is in an edited unit. Otherwise, record the regression check as NOT-RUN.
7. Check every figure, date, quantity or term that someone will rely on, with a command or the named source. Otherwise record NOT-RUN.
8. Then apply the gate and the report in section 5.

If file mode is needed but you cannot run commands or write files, use inline mode with its ledger and tell the user at the start that the work cannot be machine-verified. Every unit is frozen, none is locked, and the report says "not machine-verified".

## 1. Plan (file mode, before producing content)
Create `.pvl/plan.md` from `templates/plan.md`:
1. **Goal**: the deliverable, its user, and an acceptance test for the whole deliverable.
2. **Units**: ID, anchor locator, dependencies (acyclic), and any visible property the extract cannot capture. The report lists those as not covered.
3. **Extractor**: write `.pvl/extract` at the depth the stakes require. Every locator must return content, or an empty placeholder, without error. Once a deliverable file exists (at once, or after the first build step), run the **stability check**. It runs once per extractor version:
   - (a) Run `--hashes`. Open and save the file with your editing tool without changing anything. Run `--hashes` again. The output must be identical.
   - (b) In a scratch copy, insert one element (a row, an item, a paragraph) into one unit. Outside that unit, only `~` lines may change.
   - If either test fails, fix the keys or the normalization, or switch to an editing tool that preserves the content (for example, edit the XML part inside the package directly). Do this before going on.
   - Log the check as one MACHINE row.
4. **Criteria**: for each unit, write criteria that are pass/fail and measurable, each with a check method and an evidence type. Examples: "amount = qty x rate exactly"; "every clause cites a standard and its edition"; "opens in the target application and recalculates without error".
   - Write them before the content exists.
   - A criterion that spans units belongs to the dependent unit.
   - Prefer checks that exercise the real output (recalculate, compile, render) over checks that only read it.
   - Turn judgement criteria into measurable proxies, or mark them HUMAN.
5. **Risks** (high stakes): the top risks, each mapped to the criterion that catches it.
6. **Flow**: the template flowchart, in dependency order.
7. **Ledger**: create `.pvl/ledger.md` from `templates/ledger.md`, then run `cp .pvl/ledger.md .pvl/ledger.seal`. Create `.pvl/append` (section 3) and `.pvl/regress` (section 4).

To change a criterion later, append a CRITERION row with the reason and a new criterion ID. Units checked against the old ID lose their lock. To change units or the extractor, use a UNIT row (section 4, step 0).

## 2. Build, check, lock (each unit, in dependency order)
1. Build the unit with one or more edit steps. Run the recompute, if the format has one.
2. Run `.pvl/regress` and append a REGRESSION row with its exit code. If it exits 1, an edit has touched a protected unit: restore it as in section 4, step 7.
3. Run each of the unit's checks once. Append one row per result: PASS, FAIL or NOT-RUN. Save any output longer than one line raw to `.pvl/evidence/<row#>.txt`.
4. NOT-RUN is valid only if the check's tool is missing in this environment (name the tool) and no other evidence type can test the criterion. "Would fail", "slow" and "probably fine" are not reasons. NOT-RUN is never valid after a FAIL on the same unit, criterion and ver: that FAIL stands.
5. On FAIL, fix this unit only and rerun its checks. At the 3rd FAIL row for one unit in one revision:
   - append BLOCKED;
   - stop work on that unit;
   - ask the user whether to accept it as it is, give a direction, or drop it.

   If the user accepts the unit as it is, append USER-DECISION, then a FREEZE citing that row.
6. If every criterion has a PASS at the current ver: write the snapshot and the raw copy, then append a LOCK citing them and the PASS rows. If any criterion is NOT-RUN: write the snapshot and the raw copy, then append a FREEZE citing the NOT-RUN rows.

A check runs once for each combination of unit ver, dependency vers and criterion ID. Do not rerun it on unchanged inputs. Do not check a check's output with another check, and do not judge the output again. If evidence shows a check is wrong, replace it with a CRITERION row.

## 3. Ledger: `.pvl/ledger.md` (template `templates/ledger.md`)
- **Append only through `.pvl/append`.** Never edit, reorder or delete a row. To correct a wrong row, append a new row that cites it.
  - Write the row without its number.
  - Pass it on stdin through a quoted heredoc, so apostrophes and quotes in verbatim text are safe.
  - Write any `|` inside a cell as `\|`.
  ```sh
  #!/bin/sh
  # usage: .pvl/append <<'ROW'
  # | r1 | U2 | 3f9a01c2 | FAIL | C2.3 | MACHINE `cmd` exit 1 "..." | 24 | - |
  # ROW
  L=.pvl/ledger.md; S=.pvl/ledger.seal
  row=$(cat)
  case $row in
    *'
  '*) echo "ROW MUST BE ONE LINE" >&2; exit 1 ;;
    '| '*' |') ;;
    *) echo "ROW MUST START WITH '| ' AND END WITH ' |'" >&2; exit 1 ;;
  esac
  cmp -s "$L" "$S" || { echo "LEDGER SEAL BROKEN" >&2; exit 1; }
  k=$(( $(grep -c '^| [0-9]' "$L") + 1 ))
  printf '| %s %s\n' "$k" "$row" >> "$L" && cp "$L" "$S" && echo "appended row $k"
  ```
  If it prints LEDGER SEAL BROKEN, stop. Do not repair either file, and do not deliver as final. Report it to the user. The seal catches accidental rewrites, not deliberate tampering. If the directory is a git repository, also commit `.pvl/` at the end of each revision.
- **No stored status.** A unit's state is its latest LOCK, FREEZE or UNLOCK row, and its full history stays visible.
- **Failure chains.** A FAIL that follows a FAIL on the same unit and criterion cites that earlier FAIL in `refs`. A PASS that follows one or more FAILs cites every FAIL on that unit and criterion since the last PASS, and states in `note` what changed. Its ver must differ from the last FAIL's ver, unless it cites a CRITERION row.
- **Evidence cell.** Give the evidence type, then the command or source, then the result. A row whose evidence is narration is not a PASS.
- **Open and closed.** A BLOCKED or UNRESOLVED row stays open until a later row cites it in `refs` and closes it.
  - BLOCKED is closed by a LOCK of that unit, or by a USER-DECISION followed by a FREEZE.
  - UNRESOLVED is closed by a VERDICT based on new evidence, or by a USER-DECISION.

## 4. Revision (feedback, or any change after a unit is locked or frozen)
0. **Baseline and unit changes.**
   - If the deliverable was not built under this skill, do section 1 first, including the stability check. Then, before any edit, write snapshots and a raw copy, and append a FREEZE row (reason: baseline) for every unit.
   - If the accepted feedback adds scope, or a unit must be added, split or merged, change the units before any EDIT in this revision:
     1. Confirm that the working files equal the base (`sha256sum <files> | sort | sha256sum` gives the base's `<id>`).
     2. Copy the current extractor to `.pvl/extract.<row#>` (write-once), then update `.pvl/plan.md` and `.pvl/extract`. A new unit gets an anchor locator; content that does not exist yet extracts as empty.
     3. Run the stability check for the new extractor.
     4. Run the **equivalence check** once, on the base (`--root .pvl/raw/<base id>`): every unchanged unit's extract equals its latest snapshot exactly; for split or merged units, the sorted lines of the new units equal the sorted lines of the old units. Log it as one MACHINE row.
     5. Append a UNIT row that cites the VERDICT or USER-DECISION and the equivalence row, and describes the change. For each new or re-split unit, write its snapshot from the base and append a FREEZE citing the UNIT row. Its criteria are then checked as in section 2 (steps 3-6).

     If the equivalence check fails, the new extractor reads the old content differently: fix it, and never use it to explain away a regression.
1. **Log.** Append one FEEDBACK row per claim, with the claim verbatim and who made it. Split comments that make several points.
2. **Classify each claim.**
   - It is a *preference* only if no evidence could make it false: style, wording, scope, or a choice among valid options ("use 300 mm pavers", "add a lighting bill", "shorter intro").
   - It is *factual* if it says the content is wrong about something checkable: a quantity, a rate in a source, what a standard says, arithmetic, code behaviour. Anything phrased as "X is wrong" or "should be Y because" is factual.
   - Split mixed claims. "Cite BS 7533 in clause 4" contains a preference (cite it) and a fact (BS 7533 covers clause 4's subject).
   - If in doubt, treat the claim as factual.
3. **Judge each claim, and append a VERDICT row with its evidence.**
   - *A preference from the user*: ACCEPTED (HUMAN). If it conflicts with a criterion or a source, tell the user once, with the evidence, and then follow their answer (USER-DECISION).
   - *A preference from a third party*: ask the user, unless the user said that party decides.
   - *A factual claim, from anyone*: treat it as a hypothesis.
     - **CONFIRMED** only if both hold:
       - (a) **identity**: a command shows the disputed content's identifying attributes (from the extract: description, size, grade, unit of measure, edition, scope) and shows that the source entry you rely on has the same attributes;
       - (b) **support**: that entry, from a MACHINE or SOURCE result, supports the claim, or you reproduce the error.

       A matching string in another entry, or in a source the content does not rely on, is not confirmation.
     - **REFUTED** only if all three hold:
       - (a) evidence obtained now, independent of the disputed artefact and of your earlier work, directly contradicts the claim;
       - (b) you tested the most plausible reading under which the claim is true (another edition, unit, scope or assumption), and it fails;
       - (c) you show that evidence to the user.

       Do not edit.
     - **UNRESOLVED** otherwise, including when identity cannot be shown. Leave the units unchanged, and ask the user, giving the evidence on both sides.
   - If the user decides against the evidence, or settles an UNRESOLVED claim, append a USER-DECISION row with their words (HUMAN). Apply it, and report it as a residual risk.
   - Change a verdict only when a new evidence row supports the change, never because of pushback alone.
   - A CONFIRMED error in a locked unit means its check missed the error. Append a CRITERION row that would catch it, then a FAIL row on that criterion that cites the VERDICT.
4. **Scope.**
   - E is the set of units that must be edited. D is every unit that depends on E, directly or through other units.
   - Append UNLOCK rows for E (refs: the VERDICT row) and for D (note: depends on Ux).
   - Every other unit keeps its lock or freeze.
5. **Declare targets.** For each edit, append an EDIT row on the unit. It cites the VERDICT or USER-DECISION, and its note gives the narrowest targets the verdict requires, each with its line count: `targets: 4.1/text(1) 2.07/rate`. Use a wildcard only for keys the edit creates (`2.08/*` for a new item). If a unit in D needs an authored change, such as extending a SUM over a new item, give it its own EDIT row with targets. It is then treated as part of E.
6. **Edit surgically.** Run each step. One step is one cell write, one find/replace, one patch, or the replacement of one item. Never regenerate the deliverable, or a whole unit, to fix part of it. Run the format's recompute, if it has one.
7. **Regression check.** Run `.pvl/regress`. It diffs every protected or unlocked unit against its latest snapshot, line by line:
   - in a locked or frozen unit, any change is a regression;
   - in an unlocked unit, a changed `~` line is allowed;
   - a changed authored line is allowed only if it matches a target of an EDIT row on that unit, written after its last LOCK or FREEZE and citing a VERDICT or USER-DECISION;
   - a target may change at most its declared number of lines, counted separately on the old side and the new side; a wildcard target may only match keys absent from the snapshot.

   Save its output to `.pvl/evidence/<row#>.txt` and append a REGRESSION row with the exit code.
   ```python
   #!/usr/bin/env python3
   # usage: .pvl/regress    exit 0 = no regression; exit 1 = violations listed
   import re, sys, difflib, fnmatch, subprocess
   rows = []
   for line in open('.pvl/ledger.md', encoding='utf-8'):
       if re.match(r'\| \d', line):
           c = [x.strip().replace('\\|', '|') for x in re.split(r'(?<!\\)\|', line)[1:-1]]
           rows.append(dict(zip('n rev unit ver event crit ev refs note'.split(), c)))
   ev = {r['n']: r['event'] for r in rows}
   bad, ok = [], []
   for u in dict.fromkeys(r['unit'] for r in rows if r['event'] in ('LOCK', 'FREEZE')):
       st = [r for r in rows if r['unit'] == u and r['event'] in ('LOCK', 'FREEZE', 'UNLOCK')]
       snap = [r for r in st if r['event'] != 'UNLOCK'][-1]
       scoped = st[-1]['event'] == 'UNLOCK'
       tg = {}  # target pattern -> declared line count; None = wildcard (new keys only)
       for r in rows:
           if r['unit'] == u and r['event'] == 'EDIT' and int(r['n']) > int(snap['n']):
               if not scoped:
                   bad.append(f"{u}: EDIT #{r['n']} on a locked/frozen unit")
               if not any(ev.get(x.strip()) in ('VERDICT', 'USER-DECISION') for x in r['refs'].split(',')):
                   bad.append(f"{u}: EDIT #{r['n']} cites no VERDICT or USER-DECISION")
               m = re.search(r'targets:\s*(\S.*)', r['note'])
               for t in (m.group(1).split() if m else []):
                   p, n = re.fullmatch(r'(.+?)(?:\((\d+)\))?', t).groups()
                   tg[p] = None if set('*?[') & set(p) else (tg.get(p) or 0) + int(n or 1)
       old = open(f".pvl/snap/{u}@{snap['ver']}", encoding='utf-8').read().splitlines()
       new = subprocess.run(['.pvl/extract', u], capture_output=True, text=True, check=True).stdout.splitlines()
       oldkeys = {l.split('\t')[0] for l in old}
       used = {p: [0, 0] for p in tg}
       order = sorted(tg, key=lambda p: tg[p] is None)  # exact targets before wildcards
       for op, a1, a2, b1, b2 in difflib.SequenceMatcher(None, old, new, autojunk=False).get_opcodes():
           for side, lines in ((0, old[a1:a2]), (1, new[b1:b2])) if op != 'equal' else ():
               for l in lines:
                   key, field = (l.split('\t') + ['', ''])[:2]
                   kf = f'{key}/{field}'
                   p = next((p for p in order if fnmatch.fnmatchcase(kf, p)), None)
                   if scoped and field.startswith('~'):
                       ok.append(f"ok {u}: {op} {kf} (derived)")
                   elif not scoped or p is None:
                       bad.append(f"REGRESSION {u}: {op} {kf}")
                   elif tg[p] is None and key in oldkeys:
                       bad.append(f"REGRESSION {u}: wildcard {p} matched existing key {kf}")
                   else:
                       used[p][side] += 1
                       ok.append(f"ok {u}: {op} {kf} ({p})")
       for p, n in tg.items():
           if n is not None and max(used[p]) > n:
               bad.append(f"REGRESSION {u}: {p} changed {max(used[p])} lines, {n} declared")
   print('\n'.join(dict.fromkeys(ok + bad)) or 'no change'); sys.exit(1 if bad else 0)
   ```
   **On exit 1:**
   - copy the base into place;
   - replay the EDIT steps logged after the base row, with the faulty step fixed or removed;
   - recompute;
   - append REVERT, citing the REGRESSION row;
   - run the check again.

   Never paste extract text back into a file. Never raise a declared count, or add a target, after the step has run, to make a violation pass; a wider change needs a new EDIT row, written before its step and backed by the verdict. At the 2nd REGRESSION row with exit 1 in one revision (or while building one unit), append BLOCKED and ask the user, showing the violation list. Never treat a flagged line as noise yourself.
8. **Re-check** E and D (section 2, steps 3-6). Rerun every criterion of each unit at its new ver. The criteria of D verify its derived values. Then write new snapshots and raw copies, and append LOCK or FREEZE rows.
9. **Oscillation stop.** If fixing one unit makes a previously passing unit fail twice in the same revision, stop. The criteria probably conflict, so ask the user.

## 5. Final gate and report (inline and file mode)
Deliver as final only if all of these hold. Each is checked once.
- **File mode** (each item is a machine check):
  - `.pvl/extract --hashes` equals the ver in each unit's latest LOCK or FREEZE row;
  - every criterion of every locked unit has a PASS at the current vers;
  - the last REGRESSION row has exit 0;
  - no BLOCKED or UNRESOLVED row is open;
  - `cmp -s .pvl/ledger.md .pvl/ledger.seal` succeeds.
- **Inline mode**:
  - the ledger block carries every earlier row unchanged;
  - no factual correction was applied without a CONFIRMED verdict;
  - no UNRESOLVED claim was applied;
  - the diff and the figure checks ran, or are listed as NOT-RUN.
- **High stakes**: no numeric or factual criterion is NOT-RUN, and no unit is frozen by a USER-DECISION, unless the user explicitly accepted it.

If any check fails, do not present the work as final. Report what blocks it.

Report in at most two lines, with every field present:
`Verified: <criteria count, locked units, evidence types>. Changed: <units: what>. Failed before passing: <unit (FAIL #a, #b -> PASS #m, causes)> | none. Not run / residual risk: <NOT-RUN checks, frozen units (unverified, with reason), properties not covered by extract, user decisions, "not machine-verified" if no tool ran> | none.`
