# TASK-QUEUE.md — Qwen Discovery Tasks

> **For Qwen:** Pick the next task with status `pending`. Update status to `in-progress` when you start. To `complete` or `blocked` when you finish. Never skip the workflow.

**Queue status:** 5 pending, 0 in-progress, 0 complete, 0 blocked
**Last updated:** 2026-06-25

---

## How this queue works

Each task is a bounded unit of discovery work, scoped to fit in 2-4 hours of Qwen run-time. Tasks are independent unless a `depends_on` field is set. Pick any pending task with no unsatisfied dependencies.

**Status values:** `pending`, `in-progress`, `complete`, `blocked`

**Priority order:** P0 (do first) > P1 > P2 > P3

---

## Active queue

| ID | Status | Priority | Title | Depends on |
|---|---|---|---|---|
| Q-001 | pending | P0 | Map the Receive Watch module (RS) | — |
| Q-002 | pending | P0 | Map the Work Queue state machine (RW) | — |
| Q-003 | pending | P1 | Map the RolliConnect inbox + portal | — |
| Q-004 | pending | P1 | Audit the Shop Floor module (RS) | Q-001 (component registration) |
| Q-005 | pending | P2 | Map QBO sync paths (customer / invoice / PO) | — |

---

## Task packets

---

### Q-001 — Map the Receive Watch module (RS)

**Status:** pending
**Priority:** P0
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Produce a structured map of the Receive Watch module in RolliSuite — every input field, every database table written, every edge function called, every cross-app effect. The output is the ground truth for rebuilding intake against the canonical model.

**Why this first:** Receive Watch is the entry point for every watch in the shop. Earlier discovery showed the 4 fields entered at intake (ref-serial, department, components, caliber) land across 5 tables, with a broken FK to `calibers`. The Shop Floor failure is downstream of this. Fixing the intake data model is foundational.

**Files / folders in scope:**
- `apps/rollisuite/src/pages/intake/ReceivePackagesPage.tsx`
- `apps/rollisuite/src/pages/intake/ClientWatchEntryPage.tsx`
- `apps/rollisuite/supabase/migrations/` (for table definitions touched)
- `apps/rollisuite/supabase/functions/` (any function called from the intake flow)
- Compare findings to `documentation/workflows/VIANNA-WORKFLOW.md` and `documentation/workflows/MICHAEL-WORKFLOW.md`

**Required output:** `documentation/discovery/Q-001-receive-watch-module.md` following the `mapping-legacy-workflows` skill structure.

**Acceptance criteria:**
- Every field entered at intake is named with source UI label, code variable name, table.column destination, and data type
- Every cross-app effect (edge function call, status push to RW, email send) is listed with payload shape
- Every workaround or hard-coded value (per prior audit: `received_case = 1`) is flagged
- Comparison to the Vianna+Michael workflow docs identifies behavior gaps
- Ambiguities are logged to HUMAN-QUEUE, not guessed

**Out of scope:**
- Don't propose schema changes (that's a separate decision)
- Don't redesign the module
- Don't touch `rollitime` or `rolliworking` (separate tasks)

**Blocker examples (escalate to HUMAN-QUEUE):**
- "Is the `received_case = 1` hardcode intentional or a bug?"
- "Does the multi-estimate-per-package workflow exist in code at all, or only as a wished-for feature?"

---

### Q-002 — Map the Work Queue state machine (RW)

**Status:** pending
**Priority:** P0
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Produce a complete state machine for RolliWorking jobs: every status, every transition, who/what can trigger each transition, what side effects fire on transition (emails, status pushes back to RS, audit log writes).

**Why this first:** Mike's workflow doc named several statuses (in-testing, waiting-parts-approval, downgrade, ready-for-inspection, ready-to-ship). Need to confirm what actually exists in code vs. what's expected. The QR-driven bench workflow (D-016) depends on knowing this state machine completely.

**Files / folders in scope:**
- `apps/rolliworking/src/` — UI and state logic
- `apps/rolliworking/supabase/migrations/` — status enum and tables
- `apps/rolliworking/supabase/functions/` — edge functions, especially anything `rs-*` or `rc-*`
- `apps/rollisuite/supabase/functions/rw-*` — RS-side counterparts
- Compare to `documentation/workflows/MICHAEL-WORKFLOW.md` Section 2

**Required output:** `documentation/discovery/Q-002-work-queue-state-machine.md` with:
- Enum of all statuses
- Diagram (text/markdown table) of transitions
- For each transition: trigger, who can do it, side effects, downstream consumers
- Parts request flow start-to-finish
- Gaps where the documented workflow doesn't exist in code

**Acceptance criteria:**
- No status missed from the database enum
- Every transition has a documented trigger
- Cross-app side effects (RS notifications, RC updates) listed with payload shape
- Parts request flow fully mapped from WM submission to client approval

**Out of scope:**
- Don't redesign the state machine
- Don't propose new statuses
- iPad UI gripes are out of scope here (captured separately in workflow docs)

---

### Q-003 — Map the RolliConnect inbox + portal

**Status:** pending
**Priority:** P1
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Document the RolliConnect message inbox, customer portal, reply token system, and the duplicate-client problem.

**Why:** Mike flagged that RC creates duplicate clients and there's no merge/split. Also that inspection photos can't be sent from RS to RC. Need to know what exists today before designing what's needed.

**Files / folders in scope:**
- `apps/rolliconnect/src/` — inbox, portal, message threading
- `apps/rolliconnect/supabase/migrations/` — conversation, thread, token tables
- `apps/rolliconnect/supabase/functions/` — reply token handlers, portal magic links
- `apps/rollisuite/supabase/functions/rc-*` — RS-side integration

**Required output:** `documentation/discovery/Q-003-rolliconnect-inbox-portal.md` covering:
- Inbox architecture (conversations, threads, messages, replies)
- Reply token strategy (lifetime, scope, HMAC keys)
- Portal magic-link flow
- Customer matching logic (how RC decides if an inbound email matches an existing client)
- Where duplicate clients are created and why

**Acceptance criteria:**
- The customer-matching logic is documented with code references
- The reply token + HMAC strategy is explained
- The duplicate-client root cause is identified (or flagged as unclear)

**Out of scope:**
- Don't design merge/split UI
- Don't propose token strategy changes

---

### Q-004 — Audit the Shop Floor module (RS)

**Status:** pending
**Priority:** P1
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001 (need component registration mapped first)

**Goal:** Determine exactly why Shop Floor doesn't work, what it was supposed to do, and what would need to change to make it function as designed.

**Why:** Mike named Shop Floor as a critical broken module. Service-level completion ("in safe awaiting component") is designed into Shop Floor but the failure mode is "components aren't registered so Shop Floor won't work." Need to confirm this and map the gap.

**Files / folders in scope:**
- `apps/rollisuite/src/pages/shop-floor/` (or wherever Shop Floor lives)
- Component registration tables (identified in Q-001)
- Any docs about Shop Floor in `documentation/` (Mike mentioned separate docs exist — search for them)

**Required output:** `documentation/discovery/Q-004-shop-floor-audit.md` with:
- What Shop Floor was supposed to do (from code + comments + any existing docs)
- What it actually does today (from running the code mentally and inspecting state)
- Specific failure points (root causes, not symptoms)
- Component registration gap from Q-001 confirmed or refuted as the root cause
- What would need to change to make it work

**Acceptance criteria:**
- Root cause(s) named with code/data evidence
- Recommended fix scope (small / medium / large) with justification
- Any pre-existing Shop Floor docs are linked

**Out of scope:**
- Don't rebuild Shop Floor
- Don't propose new statuses (that's part of Q-002)

---

### Q-005 — Map QBO sync paths (customer / invoice / PO)

**Status:** pending
**Priority:** P2
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Document the three QBO sync paths separately: customer sync (suspect), invoice sync (healthy), PO sync (unconfirmed). Confirm current status of each.

**Why:** Per RS_CONTEXT_DOSSIER §0.1, customer sync is broken (cron started but no terminal status), invoice sync works fine. PO sync is low-volume and unconfirmed. Rebuild needs to know which paths to preserve, which to fix, which to redesign.

**Files / folders in scope:**
- `apps/rollisuite/supabase/functions/qbo-*` (all 16 functions per the dossier)
- Cron job configurations
- QBO token storage / refresh logic
- Any audit logs of recent sync runs

**Required output:** `documentation/discovery/Q-005-qbo-sync-paths.md` with three sub-sections:
- Customer sync: code path, current failure point, evidence
- Invoice sync: code path, why it works
- PO sync: code path, low-volume reality
- Recommendation: which paths the rebuild keeps as-is vs. rewrites

**Acceptance criteria:**
- All 16 qbo-* edge functions inventoried with one-line purpose each
- Customer sync failure point identified or flagged as still-unclear
- Sync run audit trail (if any) summarized

**Out of scope:**
- Don't fix QBO sync
- Don't redesign the QBO contract

---

## Backlog (not yet queued)

Future tasks that will be added once the first batch is complete:

- Q-006 — Map the Pickup Station flow + QBO bypass
- Q-007 — Inventory all email send paths (RS) and template usage
- Q-008 — RolliTime ↔ RS contract gap analysis (close Qwen audit gaps from earlier session)
- Q-009 — Inspection photo storage (R2 bucket) — where photos live, who reads, who writes
- Q-010 — Cross-app authentication patterns (current state) — input to D-014 prereq #3

---

_End of task queue. Append new tasks below the backlog section as they're identified._
