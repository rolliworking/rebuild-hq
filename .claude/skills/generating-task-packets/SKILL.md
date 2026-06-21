# Skill: generating-task-packets

## Purpose

Convert approved decisions, specs, or workflows into bounded implementation briefs that an executor (Cursor, Emergent, Cloud Agent, human developer) can work from without coming back to ask questions every 20 minutes.

A task packet is a contract: input fully specified, output measurable, scope explicitly bounded.

## When to invoke

- Handing work to Cursor for surgical fixes
- Spawning a Cloud Agent for bounded maintenance work
- Defining a module rebuild slice for Emergent
- Onboarding a new developer to a discrete task
- Whenever Michael's review hours are the bottleneck

## When NOT to invoke

- Open-ended exploration (the spec isn't done yet)
- Pure documentation work (use `mapping-legacy-workflows`)
- Schema design (use `modeling-canonical-data`)
- Decisions not yet approved (don't pre-pack work)

## Inputs

- An approved decision, spec, workflow document, or bug report
- Target executor (Cursor / Cloud Agent / Emergent / Human)
- Target repo and branch

## Process

1. **State the goal in one sentence.** What is true at the end of this task that isn't true now?
2. **Define non-goals explicitly.** What this task does NOT include. Refactoring? Tests for unrelated code? Updating other modules? Each non-goal is a chance to prevent scope creep.
3. **List exactly which files will be touched.** Path-by-path. New files separately.
4. **State acceptance criteria.** Concrete, testable conditions. Not "works correctly" — specific behavior.
5. **State test commands.** How the executor verifies their own work before declaring done. Include exact commands or click paths.
6. **List blocker questions.** Things that would force the executor to stop and ask. Pre-answer them, or mark "stop here if X is true."
7. **State the rollback procedure.** How to undo if something breaks. Git revert? Specific SQL? Manual?
8. **State the commit message format.** Force a SHA-trackable handoff.

## Output structure

Every task packet must have these sections, in order:

### Header
```
TASK PACKET: <short title>
EXECUTOR: <Cursor / Cloud Agent / Emergent / Human>
REPO: <repo name>
BRANCH: <target branch>
PRIORITY: <P0 hotfix / P1 / P2 / P3>
ESTIMATED EFFORT: <hours>
```

### Body

1. **Goal** — one sentence
2. **Non-goals** — bulleted list, explicit "this task does NOT do X"
3. **Files to touch** — bulleted list with paths, marked as `[modify]`, `[create]`, or `[delete]`
4. **Acceptance criteria** — numbered list of testable conditions
5. **Test commands** — exact shell commands or click paths to verify
6. **Blocker questions** — pre-answered, or "stop here and ask if X"
7. **Rollback procedure** — exact steps to undo
8. **Commit message** — exact format expected
9. **Cross-references** — link to source decision, spec, or workflow doc

## Anti-patterns

- ❌ "Make it work" goals. Specific behavior only.
- ❌ Missing non-goals. They're the most important section.
- ❌ Open-ended file lists ("touch whatever's needed"). Forces ad-hoc decisions.
- ❌ "Add tests" without specifying which tests. Tests must be enumerated.
- ❌ "Works correctly" acceptance criteria. Must be observable.
- ❌ Combining multiple unrelated changes in one packet.
- ❌ Asking the executor to make architectural decisions. Those happen upstream.

## Critical operational rules

These come from prior session learnings and must be embedded in every packet:

- "Surgical fix only — touch only the listed files"
- One commit per packet
- Commit SHA must be reported on completion
- Cherry-pick individual commits from `test` to `main` rather than merging the whole branch
- "Build complete" without commit SHA does not count as shipped

## Output destination

- `documentation/task-packets/<YYYY-MM-DD>-<short-title>.md`
- Or paste directly into the executor's chat with a copy archived to the docs folder

## Related skills

- `modeling-canonical-data` — produces the schema specs that feed packets
- `reviewing-prs-governance` — reviews the output of executed packets
- `orchestrating-cutover-tests` — packets for cutover work follow this skill, but with extra rigor

## Related decisions

- D-005 (tool stack — defines who executes what)
- D-009 (response format — packets must be readable in one screen)
