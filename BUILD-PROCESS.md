# BUILD-PROCESS.md — How Builds Get Documented

> **Purpose:** Every meaningful build slice (a module, a contract, a schema migration) produces a defined documentation trail. This file is the chain. The skills referenced here are the *how* — this file is the *what* and *when*.

---

## The build chain

Every build slice flows through six stages, each producing one artifact. The artifacts compound: they become the institutional memory of *why* and *how*, not just *what*.

```
1. SPEC          → what we agreed to build
2. PACKET        → bounded brief handed to executor
3. BUILD-LOG     → what actually got built (commits, deviations)
4. REVIEW        → governance check before merge
5. ACCEPTANCE    → how we confirmed it works
6. DELTA         → changes to canonical model / glossary / contracts
```

If a slice skips any of these, the rebuild loses memory. Six months later you'll have working code and no idea why.

---

## Stage 1 — SPEC

**Artifact:** `documentation/builds/<slice-id>/01-SPEC.md`

**What it is:** The agreed-upon design for the slice. Schema shape, business behavior, acceptance criteria at the goal level (not yet decomposed).

**Produced by:** Opus / Claude in a planning chat, using the `modeling-canonical-data` skill when schema is involved.

**Triggers:** A decision in DECISIONS-REGISTRY needs implementation, OR a discovery output identifies work to do.

**Entry criteria:**
- Discovery for the affected area is complete (see `documentation/discovery/`)
- Relevant decisions are recorded (D-### references in registry)
- Workflow truth is documented (see `documentation/workflows/`)

**Exit criteria:**
- Target language identified per D-017 (TypeScript/React for RS/RW/RC slices; Python/FastAPI for RT and AI/data modules)
- Schema (if any) follows D-010, D-011, D-013
- Cross-app contracts are defined
- Open questions are resolved or logged to `HUMAN-QUEUE.md`

---

## Stage 2 — PACKET

**Artifact:** `documentation/builds/<slice-id>/02-PACKET.md`

**What it is:** A bounded implementation brief that an executor (Cursor, Cloud Agent, human dev) can work from without coming back to ask questions every 20 minutes.

**Produced by:** Opus / Claude using the `generating-task-packets` skill.

**Triggers:** SPEC is complete and approved.

**Entry criteria:**
- SPEC exists and is signed off
- The right executor for the slice is identified (Cursor interactive / Cloud Agent / Emergent / human dev)

**Exit criteria:**
- Goal is one sentence
- Non-goals are explicit
- Files to touch are listed path-by-path
- Acceptance criteria are testable
- Rollback procedure is defined
- Commit message format is stated

**Hand-off:** The packet is copied or referenced into the executor's chat. Execution starts.

---

## Stage 3 — BUILD-LOG

**Artifact:** `documentation/builds/<slice-id>/03-BUILD-LOG.md`

**What it is:** The record of what actually got built. Commit SHAs, diffs from the spec, problems encountered, decisions made mid-build that didn't escape to the registry.

**Produced by:** The executor (Cursor / Cloud Agent / dev), at the end of their work.

**Triggers:** Build complete. Per the standing rule from `CLAUDE.md`, "build complete" without a commit SHA does not count.

**Entry criteria:**
- All commits are pushed
- Branch is named per convention (typically `test/<slice-id>`)
- Tests defined in the packet are run

**Exit criteria:**
- Commit SHA(s) recorded
- Deviations from SPEC are listed and justified (or flagged as scope changes needing re-spec)
- Any decisions made mid-build are surfaced for capture into the registry

**Format:**

```
## Slice <slice-id> Build Log

**Branch:** test/<slice-id>
**Commits:** <sha1>, <sha2>, ...
**Executor:** Cursor / Cloud Agent / <name>
**Start:** YYYY-MM-DD
**End:** YYYY-MM-DD

### What was built
[bulleted list per the packet's file-by-file scope]

### Deviations from SPEC
[anything that differs; for each, say whether it's a justified adaptation or a scope change that needs re-spec]

### Decisions made mid-build
[any decisions that bypassed the registry; flagged for retroactive capture]

### Tests run
[output or summary of acceptance tests from the packet]

### Notes for review
[anything the reviewer should pay attention to]
```

---

## Stage 4 — REVIEW

**Artifact:** `documentation/builds/<slice-id>/04-REVIEW.md`

**What it is:** Governance review of the PR against architecture rules. Catches drift before merge.

**Produced by:** Opus / Claude using the `reviewing-prs-governance` skill, or a human reviewer.

**Triggers:** PR opened from the slice's branch.

**Entry criteria:**
- PR exists and is linked
- BUILD-LOG references the same commit SHAs as the PR
- Tests from the packet have run

**Exit criteria:**
- All seven checks from the skill are completed:
  - architecture / auth / schema ownership / naming / migration safety / auditability / tests
- Blockers, required changes, and nits are listed separately
- Reviewer signs off or sends back

**Hand-off:** If approved, the slice moves to ACCEPTANCE. If not, return to executor with the review attached.

---

## Stage 5 — ACCEPTANCE

**Artifact:** `documentation/builds/<slice-id>/05-ACCEPTANCE.md`

**What it is:** Confirmation that the slice actually works in the way the business needed. Not just "tests pass" — *Vianna's intake flow now produces the expected database state, including the new `caliber_id` FK*.

**Produced by:** Michael, with Vianna or whoever uses the affected workflow.

**Triggers:** Slice is merged to main (post-review) and deployed somewhere testable.

**Entry criteria:**
- Code is deployed to staging or test environment
- Test data is available
- The human verifier knows what they're checking

**Exit criteria:**
- The packet's acceptance criteria are confirmed in practice
- Any production-only edge cases are tested or explicitly deferred
- Sign-off is recorded with a date and name

**Format:**

```
## Acceptance — Slice <slice-id>

**Verified by:** <name>
**Date:** YYYY-MM-DD
**Environment:** staging / production

### Acceptance criteria (from packet)
- [ ] criterion 1 — observed / not observed
- [ ] criterion 2 — observed / not observed
[etc.]

### Edge cases checked
[bulleted list]

### Issues found
[any, or "none"]

### Sign-off
Confirmed.
```

---

## Stage 6 — DELTA

**Artifact:** `documentation/builds/<slice-id>/06-DELTA.md`

**What it is:** Everything that changed in the broader documentation as a result of this slice. Glossary updates, new decisions, schema changes, contract updates. This is the artifact that keeps the rebuild's institutional memory in sync.

**Produced by:** Opus / Claude in a closing chat for the slice.

**Triggers:** ACCEPTANCE is signed off.

**Entry criteria:**
- All earlier artifacts (SPEC through ACCEPTANCE) are complete
- Reviewer knows what was promised vs what was delivered

**Exit criteria:**
- New decisions are appended to `DECISIONS-REGISTRY.md` (with D-### numbers)
- New glossary terms are added to `GLOSSARY.md`
- Schema changes are reflected in any relevant schema docs
- Cross-app contracts are updated (e.g. `integrations/<system>-INTEGRATION.md`)
- Workflow docs are updated if behavior changed (`workflows/*.md`)
- Wishlist items completed by this slice are checked off
- This DELTA file lists every doc that was touched, with links

**Format:**

```
## Delta — Slice <slice-id>

### Decisions added
- D-### — [title]

### Glossary updates
- [term added / changed / removed]

### Schema docs updated
- [file path] — [what changed]

### Contracts updated
- [file path] — [what changed]

### Workflow docs updated
- [file path] — [what changed]

### Wishlist items closed
- [item] — [where it lived in WISHLIST.md]

### Other affected docs
- [file path] — [why]
```

---

## Slice naming convention

```
<slice-id> = <area>-<seq>
```

Examples:
- `intake-001` — first intake slice
- `intake-002` — second intake slice
- `shop-floor-001` — first Shop Floor slice
- `qbo-001` — first QBO sync slice
- `rt-bridge-001` — first RolliTime ↔ RS bridge slice

Sequence is per-area, not global. This keeps related work clustered.

---

## Roles

| Role | Stages they own |
|---|---|
| Opus / Claude | SPEC, PACKET, REVIEW, DELTA |
| Qwen | Discovery (feeds SPEC), but does not produce SPECs |
| Cursor / Cloud Agent / Dev | BUILD-LOG |
| Michael | ACCEPTANCE, plus all approvals between stages |
| Vianna / Watchmakers | ACCEPTANCE participation when their workflow is affected |

---

## When the chain doesn't apply

Some work doesn't need the full chain. Use judgment:

- **Pure docs work** (updating GLOSSARY, adding a wishlist item) — direct commit, no chain
- **Discovery work** (Qwen reading code) — goes through `discovery/`, not `builds/`
- **Hotfixes** (production bug, no time for chain) — fix first, then back-fill SPEC + BUILD-LOG + DELTA after
- **Exploration / prototypes** (trying something to see if it works) — keep on a feature branch, no chain unless promoted to a real slice

---

## When the chain saves you

Six months from now you'll want to ask:
- Why does the intake schema look like this?
- Who decided the `received_case` field should always be 1?
- When did we add the multi-estimate-per-package handling?
- What did Vianna actually approve in the intake rebuild?
- Why did we change the contract between RS and RW for status updates?

If every slice produced its six artifacts, all of those questions have answers in `documentation/builds/`. If slices skipped artifacts, you're guessing.

Each artifact takes 5-30 minutes to produce. The whole chain per slice is ~2 hours of documentation overhead for what's typically days or weeks of build work. That's a low tax for keeping the rebuild explainable.

---

## How this fits with the skills

| Stage | Skill |
|---|---|
| SPEC | `modeling-canonical-data` (schema), `mapping-legacy-workflows` (if rebuilding behavior) |
| PACKET | `generating-task-packets` |
| BUILD-LOG | (no skill — executor's own output) |
| REVIEW | `reviewing-prs-governance` |
| ACCEPTANCE | (no skill — human-driven) |
| DELTA | (no skill — synthesis across all prior artifacts) |

Skills are *how* to do each step. This document is *what* to produce when. The skills tell Claude/Cursor *how* to write a SPEC; this document tells them *when* a SPEC is required and where it goes.

---

## Folder layout

```
documentation/
├── builds/
│   ├── intake-001/
│   │   ├── 01-SPEC.md
│   │   ├── 02-PACKET.md
│   │   ├── 03-BUILD-LOG.md
│   │   ├── 04-REVIEW.md
│   │   ├── 05-ACCEPTANCE.md
│   │   └── 06-DELTA.md
│   ├── intake-002/
│   │   └── ...
│   └── shop-floor-001/
│       └── ...
├── discovery/
│   └── Q-001-receive-watch-module.md
├── workflows/
│   ├── VIANNA-WORKFLOW.md
│   └── MICHAEL-WORKFLOW.md
└── DECISIONS-REGISTRY.md (etc.)
```

---

_End of build process. Update this file if the chain evolves. New stages or new skills get reflected here._
