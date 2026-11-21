# plan-verify-lock

**Your AI gets it 90% right. You ask it to fix the other 10%. Now the 90% is broken.**

## The pain

You give an AI agent a multi-part task: a report, a cost estimate, a specification, a codebase. The first draft is mostly right, but one section has a mistake. You point it out. The agent fixes that section, and in the process quietly changes things that were already correct: a number drifts, a clause disappears, a unit changes, a working function breaks. You only notice later, so you check everything again after every edit. The more rounds of revision, the less you can trust the result.

This happens because most agents fix one part by **regenerating the whole thing**. They have **no written definition of "correct"** and **no regression check** after an edit.

## The fix

`plan-verify-lock` is a skill (a reusable instruction set) that makes the agent work like a disciplined project team:

1. **Plan first.** Define the goal, split the work into numbered units, write *checkable* acceptance criteria for each unit, map dependencies and risks, and lay out a flow with quality gates.
2. **Verify with evidence.** Each unit is checked for real (recalculated, cross-referenced, rendered, run), not just reread.
3. **Lock what is correct.** Verified units are recorded in a ledger with a snapshot.
4. **Fix surgically.** On feedback, the agent first checks whether the feedback is actually right. It then edits only the affected units and runs a regression check: every locked unit is diffed against its snapshot, and any unintended change is reverted.
5. **Deliver only through the final gate.** All criteria pass, the regression check is clean, and residual risks are stated.

Correct parts accumulate instead of eroding. Each revision round moves the work forward and does not reopen what was already done.

See [`examples/boq-revision.md`](examples/boq-revision.md) for a walkthrough.

## Install

**Claude (claude.ai / desktop / mobile):** zip the folder `skills/plan-verify-lock` and upload it under Settings > Capabilities > Skills. Steps can vary by plan; see Anthropic's help pages.

**Claude Code:**
```bash
git clone https://github.com/GLD-Privacy/plan-verify-lock
cp -r plan-verify-lock/skills/plan-verify-lock ~/.claude/skills/
```

**Other agents:** the skill is plain Markdown. Paste `SKILL.md` into your system prompt or custom instructions.

## Use it

It triggers on multi-part deliverables and on revision requests. You can also invoke it directly: *"Use plan-verify-lock for this."* It skips itself on trivial tasks so it does not add overhead where none is needed.

## Status and how you can help

This is an **early release**. The method draws on well-established engineering practice (acceptance criteria, version snapshots, regression testing), but we have not yet measured this skill across many users and tasks. That is where you come in:

- **Try it on a real task and file a [Field report](../../issues/new?template=field-report.md).** Tell us whether correct parts survived your revisions and where the skill fell short. These reports are how we measure whether the skill works.
- **Star the repo** if it saved you a revision round, so others can find it.
- **Open a pull request** with improvements or domain-specific variants (legal drafting, engineering calculations, data pipelines).

## Prior art

Planning and verification skills already exist for software development, for example [obra/superpowers](https://github.com/obra/superpowers). This skill focuses on **locking verified units and checking for regressions during revisions**, for any deliverable, not only code.

## License

MIT. Built by Global Landscape Designers (Muscat, Oman), from daily use of AI agents on engineering deliverables.
