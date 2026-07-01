# Rolliworks Rebuild HQ — Agent Doctrine

This file orients Claude (and human operators) working from the `rebuild-hq` repo. Read this before any rebuild task.

## One-line goal

Build an integrated watch-service-center suite where fixing operational bottlenecks captures valuable byproducts and reusing that data improves the next workflow. Full vision: `documentation/NORTH-STAR.md`.

## Repo layout

`rebuild-hq` is a **standalone git repo**. It holds agent skills and doctrine only. App codebases and domain documentation live in **separate repos**, typically cloned as sibling folders on disk. They are referenced here for orientation — not contained in this repo.

```
rebuild-hq/                     ← THIS REPO (rolliworking/rebuild-hq)
├── CLAUDE.md
├── .claude/skills/             ← Phase 1 skills (invoke by name)
└── .cursor/rules/              ← Cursor agent parity

Sibling repos (separate git remotes — not tracked by rebuild-hq):
├── apps/rs/                    ← RolliSuite (RS)
├── apps/rw/                    ← RolliWorking (RW)
├── apps/rc/                    ← RolliConnect (RC)
├── apps/rt/                    ← RolliTime (RT)
├── apps/aa/                    ← Authenticator (AA)
├── documentation/              ← domain docs + all decision records
└── (Jarvis)                    ← planned app; repo TBD
```

## App roster

| Abbr | App | Status |
|------|-----|--------|
| RS | RolliSuite | Live — ERP / client files |
| RW | RolliWorking | Live — watchmaker workflow |
| RC | RolliConnect | Live — customer portal / CRM |
| RT | RolliTime | Live — accuracy / power-reserve testing |
| AA | Authenticator | In ecosystem — genuine vs counterfeit verification |
| — | Jarvis | Planned — not yet built |

## Language strategy (D-017)

Customer-facing operational apps (**RS, RW, RC**) stay **TypeScript / React**. AI, ML, data, and computer-vision modules (**RT, M3KE, Authenticator, Jarvis, RolliCurator**) are built in **Python (FastAPI)**. All stacks connect via shared PostgreSQL (Supabase) and language-agnostic API contracts — not shared-codebase imports. See `documentation/DECISIONS-REGISTRY.md` D-017 for the full assignment table.

## Tool stack (D-005)

| Tool | Role |
|------|------|
| Existing apps + SQL + docs repo | Source of truth for behavior and domain |
| Opus / Claude | Planner, spec author, reviewer |
| Cursor (interactive) | Supervised builder — surgical fixes |
| Cursor Cloud Agents | Bounded maintenance only |
| Emergent | Module rebuilder once spec is stable |
| Local Qwen / gbrain | Documentation drafting and indexing — not code generation |
| Lovable | Product memory helper — not primary builder |

## Skill routing — which skill to invoke

| Situation | Skill |
|-----------|-------|
| Document how an existing workflow actually works | `mapping-legacy-workflows` |
| Design or change a database table | `modeling-canonical-data` |
| Hand bounded work to an executor | `generating-task-packets` |
| Review a PR, commit, or handoff SHA | `reviewing-prs-governance` |
| Cutover, migration, or test→main promotion | `orchestrating-cutover-tests` |

Skills live in `.claude/skills/<name>/SKILL.md`. Read the full skill before starting.

## Operating rules

1. **Surgical fix only** — touch only files listed in the task packet.
2. **One commit per packet** — report the SHA on completion.
3. **Cherry-pick, don't merge** — promote individual verified commits from `test` to `main`; never merge whole `test` branches.
4. **"Build complete" without SHA does not count as shipped.**
5. **Ground truth over inference** — verify DB content and live behavior; don't speculate.
6. **Bulldozer before fix** on cross-app bugs — map the full pipeline before proposing fixes.
7. **False summits cost trust** — don't claim "found it" until every gap in the pipeline is accounted for.
8. **Max 1–2 questions per turn** (D-009) — don't create decision backlog.

## Ambiguity handling — plain-language questions

When you cannot decide from code alone, **do not guess**. Log the question to `HUMAN-QUEUE.md` and continue with the next task.

**Plain-language discipline applies to Cursor Agent, Opus, and every coding agent** during discovery, SPEC, and BUILD work:

- Use business terms (customer, staff, watch, invoice) — not engineer terms (row, upsert, JWT, CHECK constraint)
- Do not use variable names, function names, or table names in the question text itself
- Do not front-load file paths — explain what happens operationally first; put paths in parentheses at the end if needed
- Define acronyms on first use
- Frame questions around operational impact: "Should X happen when Y?"

See **QWEN.md "Plain-language rules"** section for the full guidance, format template, and bad→good examples. Apply the same discipline whether you're Cursor Agent, Opus, or any other coding agent.

Discovery output files (`documentation/discovery/Q-###-*.md`) must also use plain language in their "Open questions" sections. Questions in engineer-language must be rewritten before the next Word snapshot export (D-018).

---

## Where to look first

1. This file (`CLAUDE.md`)
2. Relevant skill in `.claude/skills/`
3. `documentation/handoff/current.md`
4. `documentation/DECISIONS-REGISTRY.md` (all decisions, including D-014+)
5. Relevant CONTEXT_DOSSIER or workflow doc
6. Only then — app source code in the relevant sibling repo

## Branch conventions (app repos)

- Work on `test` branch in RS, RW, RC, RT, AA
- Merge to `main` only after regression verification
- Cursor + localhost is the dev environment

## Output destinations (documentation repo)

| Artifact | Path |
|----------|------|
| Workflow docs | `documentation/workflows/<app>-<workflow-name>.md` |
| Schema proposals | `documentation/schema/<entity>.md` |
| Task packets | `documentation/task-packets/<YYYY-MM-DD>-<short-title>.md` |
| Cutover briefs | `documentation/cutovers/<YYYY-MM-DD>-<short-title>.md` |
| PR reviews | `documentation/reviews/<YYYY-MM-DD>-<short-title>.md` |

## Decisions

All decision records (D-001 through D-017): `documentation/DECISIONS-REGISTRY.md`

## Anti-patterns

- Documenting intent instead of actual behavior
- Scope creep beyond the task packet
- Architectural decisions inside executor sessions (those happen upstream)
- Storing the same fact in two schemas (drift incoming)
- Hard deletes on master rows
