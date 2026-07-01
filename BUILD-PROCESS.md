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
- **Transferability Test** completed per D-023 — all five results Pass, or documented waiver in `documentation/TRANSFERABILITY-WAIVERS.md` (see `documentation/TRANSFERABILITY-TEST.md`)

**Transferability gate (D-023):** At the end of `01-SPEC.md`, include the Transferability Test Results table. A SPEC does not lock until every test is Pass or waived. During BUILD, re-verify the implementation matches what the SPEC promised. Waivers expire within 12 months and are reviewed quarterly.

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

## Question snapshot cadence

Question snapshots are the human-facing surface of the autonomous workflow. Qwen logs questions to `HUMAN-QUEUE.md` at the rebuild-hq root during its work. Periodically these get exported to dated Word documents in Dropbox for offline, cross-device review.

### When snapshots get generated

A new Word doc gets created when any of these fire:

- **Every 72 hours** — alongside the build digest (see BUILD-DIGEST-TEMPLATE.md)
- **Volume trigger** — when HUMAN-QUEUE.md has 10+ open questions
- **Planning trigger** — before any Opus planning session, so the operator arrives with answers ready
- **Pre-run trigger** — before any Qwen autonomous run lasting 4+ hours, so Qwen has fresh answers to work from

### Where they get saved

Word doc snapshots live in Dropbox, not the git repo:

```
G:\Dropbox\__AI\emergent-hq-human-queue\
```

Filename pattern: `QUESTIONS-YYYY-MM-DD.docx`

This allows cross-device review and answering without git pull/push overhead. The markdown source of truth (HUMAN-QUEUE.md, HUMAN-ANSWERS.md) remains in the rebuild-hq git repo — Dropbox is for the Word snapshots only.

See D-018 (amended) and `documentation/queue-exports/README.md` for the full workflow.

### Who generates them

Opus or Cursor produces the Word doc by:
1. Reading HUMAN-QUEUE.md (the live source)
2. Reading every "Open questions" section in `documentation/discovery/`
3. Consolidating into a Word doc with the structure shown in `documentation/queue-exports/README.md`
4. Saving as `G:\Dropbox\__AI\emergent-hq-human-queue\QUESTIONS-YYYY-MM-DD.docx` (not committed to git)

### Where they fit in the build chain

This sits **orthogonal to the 6 build stages** (SPEC → PACKET → BUILD-LOG → REVIEW → ACCEPTANCE → DELTA). Questions can surface at any stage; snapshots happen on cadence regardless of which stage is active. They are an ambient ritual that keeps the autonomous workflow unblocked without forcing the operator to scroll markdown.

### How answers return

1. Operator answers questions in the Word doc on phone, tablet, or anywhere (Dropbox auto-syncs)
2. Saves the answered Word doc in the same Dropbox folder
3. Cursor reads the answered doc from Dropbox and updates `HUMAN-ANSWERS.md` accordingly
4. Cursor commits and pushes (markdown only)
5. Qwen reads `HUMAN-ANSWERS.md` at session start and applies the answers to blocked work

This closes the loop without requiring the operator to ever open the raw markdown HUMAN-QUEUE.md.

### See also

- `D-018` (amended) in `DECISIONS-REGISTRY.md` — the discipline as a decision record
- `documentation/queue-exports/README.md` — workflow and Cursor prompts

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
├── TRANSFERABILITY-TEST.md
├── TRANSFERABILITY-WAIVERS.md
└── DECISIONS-REGISTRY.md (etc.)
```

---

_End of build process. Update this file if the chain evolves. New stages or new skills get reflected here._
