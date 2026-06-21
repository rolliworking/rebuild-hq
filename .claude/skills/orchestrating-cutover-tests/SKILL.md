# Skill: orchestrating-cutover-tests

## Purpose

Plan and execute cutover work — migrations, branch promotions, shared-database transitions, and production-affecting deploys — with rigor beyond a normal task packet. Cutovers fail silently; this skill forces pre-flight, execution, and post-flight verification with explicit rollback.

## When to invoke

- Promoting verified commits from `test` to `main` on any app repo
- Shared Supabase migration or schema cutover (D-003)
- Big-bang or read-only-freeze migration windows
- Dual-auth or dual-write transition periods (e.g. RC Bearer → x-api-key cutover)
- Any change where rollback must be executable within minutes, not hours

## When NOT to invoke

- Routine surgical fixes on `test` that don't touch production
- Documentation-only changes
- Greenfield module builds with no live users affected
- Standard PR review without cutover risk (use `reviewing-prs-governance`)

## Inputs

- The cutover plan: what changes, what systems, what window
- Rollback procedure (must exist before cutover starts)
- Regression fixtures (beacon estimates, known SOs, test accounts)
- Affected apps: RS, RW, RC, RT, or combinations
- On-call / operator availability during the window

## Process

1. **Classify cutover risk.** Low (config-only) / Medium (single-app deploy) / High (cross-app or DB migration). High cutovers require a written cutover brief, not just a task packet.
2. **Define the freeze boundary.** What stops changing during the window? Incoming writes, outbound pushes, scheduled jobs?
3. **List pre-flight checks.** Migrations applied? Edge functions deployed? Env vars set? Backups taken? Feature flags ready?
4. **Define the execution sequence.** Ordered steps with owner per step. No parallel steps that depend on each other unless explicitly safe.
5. **Define post-flight verification.** Exact queries, click paths, and API calls to confirm each system. Use beacon fixtures where available (EST 25530, BR-Tracking-Beacon).
6. **Define rollback triggers.** What observable failure triggers rollback? Who decides? How fast?
7. **Execute rollback drill.** Before high-risk cutovers, walk through rollback steps on paper or in staging — not improvised live.
8. **Record the cutover log.** Timestamp each step, result, and verifier name.

## Output structure

Every cutover brief must include:

### Header
```
CUTOVER BRIEF: <short title>
RISK: <Low / Medium / High>
WINDOW: <date/time range>
OPERATOR: <name>
ROLLBACK OWNER: <name>
```

### Body

1. **Goal** — what is true after cutover that isn't true now
2. **Systems affected** — RS / RW / RC / RT / Supabase / edge functions
3. **Pre-flight checklist** — numbered, each with pass/fail column
4. **Execution sequence** — numbered steps with owner and expected duration
5. **Post-flight verification** — numbered checks with exact commands or click paths
6. **Rollback procedure** — exact steps, tested or drill-confirmed
7. **Rollback triggers** — conditions that force rollback
8. **Regression fixtures** — which beacon estimates / SOs / accounts to verify
9. **Communication** — who gets notified before, during, and after
10. **Cutover log** — filled in during execution (timestamp, step, result, verifier)

## Anti-patterns

- ❌ Cutover without a written rollback procedure
- ❌ "We'll fix forward" as the rollback plan
- ❌ Merging whole `test` branches instead of cherry-picking verified commits
- ❌ Skipping post-flight verification because "deploy succeeded"
- ❌ Running high-risk cutovers without operator availability
- ❌ Changing multiple independent systems in one window without isolation plan
- ❌ Assuming Lovable "Applied" means production serves the change — verify the bundle

## Mandatory checks before any production cutover

- [ ] Task packet or cutover brief approved by Michael
- [ ] Rollback procedure written and drill-reviewed for High risk
- [ ] Pre-flight checklist complete
- [ ] Post-flight verification commands written (not improvised)
- [ ] Beacon or regression fixtures identified
- [ ] Cherry-pick targets identified (specific SHAs, not branch tips)
- [ ] Cutover log template ready

## Output destination

- Cutover briefs: `documentation/cutovers/<YYYY-MM-DD>-<short-title>.md`
- Execution logs: append to the same file under a `## Cutover log` section

## Related skills

- `generating-task-packets` — routine work uses packets; cutovers extend them with this skill
- `reviewing-prs-governance` — every commit cherry-picked to `main` must pass review first
- `mapping-legacy-workflows` — cutover verification often traces live workflows

## Related decisions

- D-003 (shared Supabase, separate apps — cutover target architecture)
- D-005 (tool stack — who executes cutover steps)
- D-008 (RolliTime contract — external system cutover boundaries)
