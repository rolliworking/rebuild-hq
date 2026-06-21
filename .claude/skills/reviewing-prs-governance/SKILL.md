# Skill: reviewing-prs-governance

## Purpose

Review pull requests and shipped commits against Rolliworks rebuild governance — scope, schema rules, branch policy, and evidence of verification. Catches drift before it merges to production.

A PR review is a gate: did the executor stay inside the task packet, follow canonical data rules, and prove the fix with observable evidence?

## When to invoke

- Reviewing a PR or commit before cherry-pick to `main`
- Verifying a Cursor or Cloud Agent handoff ("here's the SHA")
- Auditing whether a schema migration follows D-003 / D-010 / D-011
- Michael's review pass on any executor output
- Closing a task packet — confirming acceptance criteria were met

## When NOT to invoke

- Writing the implementation (that's the executor's job)
- Open-ended code exploration (use bulldozer research first)
- Drafting the original task packet (use `generating-task-packets`)
- Cutover or migration orchestration (use `orchestrating-cutover-tests`)

## Inputs

- The PR, commit SHA, or diff to review
- The source task packet (if one exists)
- Target repo and branch (`test` vs `main`)
- Any related decision IDs (D-###) or workflow docs

## Process

1. **Match scope to the packet.** List every changed file. Flag any file not in the packet's "Files to touch" list. Flag missing files the packet required.
2. **Check branch policy.** Work lands on `test` first. `main` only after regression verification. Cherry-pick individual commits — do not merge whole `test` branches wholesale.
3. **Verify the commit SHA.** "Build complete" without a reported SHA does not count as shipped. The SHA in the handoff must match the commit being reviewed.
4. **Run acceptance criteria.** Each criterion from the task packet must be checked against observable evidence — not "looks fine."
5. **Schema governance pass** (if DB touched). Apply mandatory checks from `modeling-canonical-data`: schema placement, single write owner, audit columns, no denormalized many-valued columns, FKs on stable codes, master lifecycle columns if applicable.
6. **Cross-app contract check** (if integration touched). Does the change match documented payload shapes in `documentation/integrations/`? Flag doc-vs-code divergence.
7. **Surgical fix compliance.** One concern per commit. No drive-by refactors, no unrelated test churn, no formatting sweeps.
8. **Record the verdict.** Approve, approve with notes, or block — with specific line-level or file-level reasons.

## Output structure

Every review must include:

1. **Verdict** — Approve / Approve with notes / Block
2. **Commit SHA reviewed** — exact hash
3. **Scope check** — files in packet vs files in diff; extras and omissions listed
4. **Acceptance criteria results** — numbered pass/fail per criterion with evidence
5. **Schema governance** — pass/fail/N/A with notes
6. **Branch recommendation** — stay on `test` / ready for cherry-pick to `main` / needs rework
7. **Blockers** — must-fix items before merge
8. **Notes** — non-blocking improvements deferred to future packets

## Anti-patterns

- ❌ Approving without running or verifying acceptance criteria
- ❌ Approving scope creep because "it's a small improvement"
- ❌ Merging `test` → `main` as a branch merge instead of cherry-picking verified commits
- ❌ Accepting "works on my machine" without the packet's stated test commands
- ❌ Skipping schema checks because "it's just one column"
- ❌ Reviewing intent instead of the actual diff

## Mandatory block conditions

Block the PR if any of these are true:

- Changed files outside the task packet scope (unless packet was amended and approved)
- No commit SHA provided in the handoff
- Acceptance criteria not demonstrated
- Schema change violates D-003 placement or D-010 canonical model rules
- Multiple unrelated concerns in one commit
- Hard delete on master rows without approved exception

## Output destination

- Review notes: inline on PR, or `documentation/reviews/<YYYY-MM-DD>-<short-title>.md` for significant audits

## Related skills

- `generating-task-packets` — defines the contract being reviewed
- `modeling-canonical-data` — schema rules to enforce
- `orchestrating-cutover-tests` — cutover reviews use this skill plus extra cutover checklist

## Related decisions

- D-005 (tool stack — who executes vs who reviews)
- D-009 (response format — reviews must be scannable in one screen)
- D-010 (canonical ecosystem data model)
- D-011 (living masters with promotion workflows)
