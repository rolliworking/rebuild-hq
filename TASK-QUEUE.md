# TASK-QUEUE.md — Qwen Discovery Tasks

> **For Qwen:** Pick the next task with status `pending`. Update status to `in-progress` when you start. To `complete` or `blocked` when you finish. Never skip the workflow.

**Queue status:** 15 pending, 0 in-progress, 0 complete, 0 blocked
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
| Q-006 | pending | P1 | Map Pickup Station + QBO bypass workflow | Q-001, Q-005 |
| Q-007 | pending | P2 | Inventory email send paths + template usage | Q-001 |
| Q-008 | pending | P1 | Close RolliTime ↔ RS contract gaps (audit follow-up) | — |
| Q-009 | pending | P1 | Map inspection photo storage + R2 access patterns | — |
| Q-010 | pending | P0 | Document cross-app authentication current state | — |
| Q-011 | pending | P0 | Audit component registration (unblocks Shop Floor) | Q-001 |
| Q-012 | pending | P2 | Identify RolliConnect duplicate-client root cause | Q-003 |
| Q-013 | pending | P1 | Map watch storage / safe workflow digital state | Q-001, Q-011 |
| Q-014 | pending | P2 | Inventory RolliWorking iPad UI + gestures | Q-002 |
| Q-015 | pending | P2 | Map intake workflow variants (estimate-first vs none) | Q-001 |

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
- `apps/rs/src/pages/intake/ReceivePackagesPage.tsx`
- `apps/rs/src/pages/intake/ClientWatchEntryPage.tsx`
- `apps/rs/supabase/migrations/` (for table definitions touched)
- `apps/rs/supabase/functions/` (any function called from the intake flow)
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
- Don't touch `apps/rt/` or `apps/rw/` (separate tasks)

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
- `apps/rw/src/` — UI and state logic
- `apps/rw/supabase/migrations/` — status enum and tables
- `apps/rw/supabase/functions/` — edge functions, especially anything `rs-*` or `rc-*`
- `apps/rs/supabase/functions/rw-*` — RS-side counterparts
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
- `apps/rc/src/` — inbox, portal, message threading
- `apps/rc/supabase/migrations/` — conversation, thread, token tables
- `apps/rc/supabase/functions/` — reply token handlers, portal magic links
- `apps/rs/supabase/functions/rc-*` — RS-side integration

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
- `apps/rs/src/pages/shop-floor/` (or wherever Shop Floor lives)
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
- `apps/rs/supabase/functions/qbo-*` (all 16 functions per the dossier)
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

### Q-006 — Map Pickup Station + QBO bypass workflow

**Status:** pending
**Priority:** P1
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001 (intake context), Q-005 (QBO context)

**Goal:** Document the full Pickup Station flow: QR scan, payment check, QBO bypass logic, intake-photo comparison, exit photo capture. Identify integration points for future IP cam / Nest snapshot capability per D-015.

**Why this matters:** Mike's chain-of-custody priority (D-015) hooks directly into the Pickup Station. The QBO bypass exists because QBO has payment registration latency — need to know exactly when and how the bypass is invoked.

**Files / folders in scope:**
- `apps/rs/src/pages/pickup/` (or wherever the Pickup Station UI lives)
- `apps/rs/supabase/functions/qbo-*` for the payment-check edge functions
- Compare to `documentation/workflows/MICHAEL-WORKFLOW.md` Section 3 (pickup section)

**Required output:** `documentation/discovery/Q-006-pickup-station.md` covering:
- Step-by-step flow from QR scan → completion
- Payment check logic (when does it call QBO vs when does the bypass kick in)
- Photo comparison logic (intake photos vs exit photos)
- Audit log writes
- Integration points where IP cam / Nest snapshot could hook in without rebuilding the flow

**Acceptance criteria:**
- QBO bypass code path is named with file path and line numbers
- The "known paid" decision logic is documented (how does it know?)
- Photo comparison flow is mapped end-to-end

**Out of scope:**
- IP cam integration design (just identify hook points)
- QBO sync redesign (covered in Q-005)

---

### Q-007 — Inventory email send paths + template usage (RS)

**Status:** pending
**Priority:** P2
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001

**Goal:** Catalog every email that RolliSuite sends, which template each uses, which trigger fires it, and which domain it sends from.

**Why this matters:** Vianna's email workflow has multiple pain points: noreply addresses getting replies, no CC/BCC into RC, manual update-email workflow, missing bulk send for "testing complete." Need to know what exists before redesigning the email layer.

**Files / folders in scope:**
- `apps/rs/supabase/functions/send-*-email`
- `apps/rs/src/pages/admin/email-templates/` (or wherever templates are managed)
- `message_templates` table content
- Compare to `documentation/workflows/VIANNA-WORKFLOW.md` Section 4 (pain points related to email)

**Required output:** `documentation/discovery/Q-007-email-send-paths.md` covering:
- Every edge function that sends email, with one-line purpose each
- Which template each pulls from
- Sender domain per template (noreply vs main)
- Trigger source per template (manual button, automated cron, workflow event)
- Identification of any hardcoded email content (violating the "always pull from message_templates" rule)

**Acceptance criteria:**
- All send-*-email functions inventoried
- Template-to-function mapping is complete
- Any hardcoded email strings are flagged with file path + line
- "Reply behavior" per template documented (does it accept replies? where do they go?)

**Out of scope:**
- Don't redesign email templates
- Don't propose bulk email infrastructure (that's a build task post-discovery)

---

### Q-008 — Close RolliTime ↔ RS contract gaps (audit follow-up)

**Status:** pending
**Priority:** P1
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Build on the prior Qwen audit (2026-06-07) that identified critical gaps in the RolliTime integration spec. Now with the workflow capture complete (VIANNA-WORKFLOW.md, MICHAEL-WORKFLOW.md), produce specific payload schemas for each of the four data flows.

**Why this matters:** The earlier audit was honest — the RT contract is intent-only, no payload schemas, no endpoint definitions. RT is in active development. Without these specifics, RT can't actually integrate with RS or RW even when the shared schema is ready.

**Files / folders in scope:**
- `documentation/ROLLITIME-INTEGRATION.md` (existing spec)
- `apps/rt/` (current build state)
- `apps/rw/` (target for RT→RW status push)
- `apps/rs/src/pages/inspection/` (where RT photos would surface)
- Compare to Mike's workflow re: testing-complete email and inspection flow

**Required output:** `documentation/discovery/Q-008-rt-rs-contract-gaps.md` covering:
- **RT → RS:** Exact payload schemas for test results write-back and dial-photo metadata API. Specific table/endpoint destinations. Auth model.
- **RS → RT:** Exact caliber/reference data fields RT needs. Pull mechanism (poll, webhook, Realtime subscription).
- **RT → RW:** Status push payload schema. Trigger event spec. Keying mechanism (ref-serial → which RW job).
- **RT ← RW:** Whether reverse path is needed (e.g. job cancelled). If yes, propose contract.
- For each gap: a specific recommendation, not just identification of the gap.

**Acceptance criteria:**
- Every gap from the 2026-06-07 audit either has a proposed contract or a specific escalated question
- Photo storage destination decision proposed (RT owns its own R2 bucket vs writes to RS's)
- Auth pattern proposed for cross-database writes
- The "RT cancelled mid-test" scenario has a proposed handling

**Out of scope:**
- Don't implement anything
- Don't redesign the broader RT architecture

---

### Q-009 — Map inspection photo storage + R2 access patterns

**Status:** pending
**Priority:** P1
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Document where inspection photos live, who writes them, who reads them, how they're organized today, and what changes are needed for the future shared dial-photo library (D-008 + RolliTime).

**Why this matters:** Photos are cross-cutting: RS captures inspection photos via Ipevo cam + microscope cam, RW staff need read access (currently blocked), RolliTime is building its own dial-photo library, future Authenticator app consumes the library. Galleria getting unwieldy per Vianna (30-50 watches × 10-40 photos = thousands per client).

**Files / folders in scope:**
- `apps/rs/supabase/functions/upload-photo`, `upload-client-photos`, `get-photo`
- `apps/rs/src/pages/inspection/` (capture and viewing)
- R2 bucket organization (paths, naming conventions, retention)
- `documentation/ROLLITIME-INTEGRATION.md` §6 (photo library spec)

**Required output:** `documentation/discovery/Q-009-photo-storage.md` covering:
- R2 bucket structure today (paths, naming)
- Photo upload paths (which functions, which storage prefix)
- Photo read paths (who can read, with what auth)
- The galleria UI organization (chronological? per-watch? per-job?)
- Why RW can't see photos today (auth gap or UI gap?)
- Proposed date-based folder structure for galleria
- Integration plan with RolliTime's separate photo library

**Acceptance criteria:**
- Current R2 path structure documented
- Auth model for photo access documented
- Specific changes needed to grant RW read access named
- Galleria reorganization plan proposed (without rebuilding the UI)

**Out of scope:**
- Don't build new photo viewers
- Don't migrate existing photos

---

### Q-010 — Document cross-app authentication current state

**Status:** pending
**Priority:** P0
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`

**Goal:** Map exactly how RS / RW / RC authenticate to each other today, including the M3KE JWT issue. Define the auth model the rebuild needs (per REBUILD-PREREQUISITES.md #3).

**Why this matters:** Foundational for the shared Supabase rebuild. Every other rebuild slice depends on knowing the auth pattern. Today's apps are in separate Supabase projects with ad-hoc cross-app auth. The rebuild needs a coherent model: service roles, RLS policies, edge function auth, cross-app reads/writes.

**Files / folders in scope:**
- `apps/rs/src/contexts/AuthContext.tsx`
- `apps/rs/src/integrations/supabase/client.ts`
- `apps/rs/supabase/functions/rw-*` (cross-app calls)
- `apps/rs/supabase/functions/rc-*` (cross-app calls)
- `apps/rw/` auth setup
- `apps/rc/` auth setup
- M3KE's auth approach (JWT-blocked issue per RS dossier)

**Required output:** `documentation/discovery/Q-010-cross-app-auth.md` covering:
- How each app authenticates a user session today
- How cross-app edge functions authenticate (shared secret? service role? user JWT?)
- The specific M3KE JWT issue (what's blocking it from reading RS data?)
- A proposed auth model for the rebuild:
  - User session pattern across apps (one login? per-app?)
  - Edge function auth pattern
  - AI module auth pattern (read-only role across operational schemas)
  - External program auth (RolliTime, Authenticator) — per-program service accounts

**Acceptance criteria:**
- Current auth pattern documented per app
- M3KE blocker root cause identified
- Rebuild auth proposal addresses every consumer type (apps, AI modules, external programs)
- RLS policy implications named

**Out of scope:**
- Don't implement the new auth model
- Don't change current auth

---

### Q-011 — Audit component registration (unblocks Shop Floor)

**Status:** pending
**Priority:** P0
**Estimated time:** 3-4 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001

**Goal:** Identify exactly where component registration happens (or fails to happen) in the intake flow. This is the root cause Mike named for the Shop Floor module's persistent failure.

**Why this matters:** Mike's quote: "components aren't registered so shop floor won't work." Until component registration is reliable, Shop Floor can't function, which means service-level completion can't function, which means a major rebuild theme is blocked.

**Files / folders in scope:**
- `apps/rs/src/pages/intake/` (Receive Watch flow from Q-001)
- Component-related tables (identified in Q-001 output)
- Any logic that creates rows in component tables during intake
- Any logic that *should* but doesn't (the 25-day component creation outage referenced in prior chats)

**Required output:** `documentation/discovery/Q-011-component-registration.md` covering:
- Every place component registration should occur (per the intended workflow)
- Every place it actually occurs in code (per Q-001 + further inspection)
- The specific gap (where it should happen but doesn't)
- The 25-day component creation outage post-mortem (what happened, what was fixed, what wasn't)
- A minimal change proposal to make component registration reliable

**Acceptance criteria:**
- Every intended registration point named with file/line evidence
- The gap is specific (not "registration is broken" — specifically, "row never created in `client_property_components` because trigger fires before X")
- Minimal-change proposal scoped to "fix what's broken" not "rebuild the whole thing"

**Out of scope:**
- Don't fix the bug yet (that's a build task)
- Don't redesign component tables

---

### Q-012 — Identify RolliConnect duplicate-client root cause

**Status:** pending
**Priority:** P2
**Estimated time:** 2 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-003

**Goal:** From Q-003's mapping, identify exactly why RC creates duplicate clients. Propose a specific fix.

**Why this matters:** Mike named duplicate clients as a real pain — conversations fragment across duplicate records. Mike wants merge/split + labels. Need to know why duplicates happen before deciding whether to fix matching logic or just add merge/split capability.

**Files / folders in scope:**
- Customer matching logic in RC (from Q-003)
- `shared.customers` (or RC's equivalent today) write paths
- Inbound email parsing logic

**Required output:** `documentation/discovery/Q-012-rc-duplicate-clients.md` covering:
- The current matching algorithm (email exact match? fuzzy? based on what fields?)
- The specific case(s) where duplicates get created
- Whether the root cause is matching logic or write logic
- Proposal: improve matching vs add merge/split UI vs both

**Acceptance criteria:**
- A specific test case is named ("when client emails from different address than estimate, duplicate is created because...")
- The fix recommendation is concrete

**Out of scope:**
- Don't build merge/split UI
- Don't deduplicate existing records

---

### Q-013 — Map watch storage / safe workflow digital state

**Status:** pending
**Priority:** P1
**Estimated time:** 2-3 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001, Q-011

**Goal:** Map what the software knows about physical watch storage (safe contents, watchmaker bins, polish room, departments) versus what only exists in human memory or on paper (3-color carbon work orders).

**Why this matters:** Mike's #1 operational risk: "Not knowing what's waiting in the safe. No trail or log for chain of custody." The chain-of-custody decision (D-015) hinges on what state already exists in code. Need to know the gap before deciding what surveillance + tracking infrastructure to add.

**Files / folders in scope:**
- `client_property` and related component tables
- Job status flow (Q-002 output) for "in safe" / "in workshop" / "in polish" states
- Department codes (W / B / P / PM) usage
- The handoff between Vianna's intake (Q-001) and watchmaker assignment (Q-002)

**Required output:** `documentation/discovery/Q-013-storage-workflow.md` covering:
- Every digital state the software tracks about physical location
- Every gap (where humans know something the system doesn't — bin assignment, current watchmaker's possession, polish room queue)
- The 3-color carbon work order data — what's captured digitally vs what only exists on paper
- A proposal for closing each gap with minimum invasive changes

**Acceptance criteria:**
- Digital vs paper inventory is exhaustive
- The current "safe audit" capability (or lack of) is documented
- Proposals for chain-of-custody hooks per D-015 are concrete

**Out of scope:**
- Don't design IP cam integration (D-015 is a future build)
- Don't propose new workflows that change physical operations

---

### Q-014 — Inventory RolliWorking iPad UI + gestures

**Status:** pending
**Priority:** P2
**Estimated time:** 2 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-002

**Goal:** Catalog every screen/page in RolliWorking, what it does on iPad today, what gestures or actions are required, and where the UI fails iPad usage patterns.

**Why this matters:** Foundation for the QR-driven bench workflow per D-016. The "scan ref-serial then scan process QR" pattern needs to know which screens are currently the friction points.

**Files / folders in scope:**
- `apps/rw/src/pages/` and `apps/rw/src/components/`
- iPad-specific styling (Tailwind classes, media queries)
- Touch event handlers (or lack of)
- Voice-to-text capability (currently absent per Mike's notes)

**Required output:** `documentation/discovery/Q-014-rw-ipad-ui.md` covering:
- Page-by-page inventory of RW screens
- Per page: typical user action, current gesture/tap count, friction notes
- Where the UI is web-view-only vs touch-optimized
- Where QR scanning could replace current navigation (per D-016)
- Recommended order of QR transformations (which screens to QR-ify first)

**Acceptance criteria:**
- Every page in RW is named with primary action
- Tap-count for common workflows is estimated (e.g. "marking a job in-testing currently takes 4 taps")
- QR replacement candidates are ranked by frequency-of-use

**Out of scope:**
- Don't redesign screens
- Don't build QR generators

---

### Q-015 — Map intake workflow variants (estimate-first vs none)

**Status:** pending
**Priority:** P2
**Estimated time:** 2 hours
**Skill:** `mapping-legacy-workflows`
**Depends on:** Q-001

**Goal:** Document the multiple intake patterns: (a) standard estimate-first (99% of cases), (b) shipment arrives with no prior estimate, (c) shipment arrives with multiple estimates on one label. Identify what code path exists for each, and where the gaps are.

**Why this matters:** The "estimate-first" assumption is baked into the current code (per Q-001 expected findings). The edge cases (no estimate, multi-estimate) currently rely on manual workarounds. Vianna sets aside; Mike creates estimate after the fact. Need a real process.

**Files / folders in scope:**
- `apps/rs/src/pages/intake/ReceivePackagesPage.tsx` (Q-001 will have mapped this)
- Shipping label scan logic
- Estimate creation flow (for backfill scenarios)

**Required output:** `documentation/discovery/Q-015-intake-variants.md` covering:
- Standard estimate-first flow: confirmed working
- Multi-estimate-per-label: current workaround documented, code support gap identified
- No-estimate-on-arrival: current workaround documented, code support gap identified
- Proposed minimal additions to support both edge cases without rebuilding intake

**Acceptance criteria:**
- All three patterns are named with frequency estimates
- Workarounds Vianna and Mike use are captured in code-touchable terms
- Proposed fixes are minimal-invasive

**Out of scope:**
- Don't redesign intake
- Don't change the estimate creation flow

---

## Backlog (after Q-015)

Future tasks that aren't yet queued:

- Q-016 — RolliCurator scaffolding spec (when ready to build)
- Q-017 — RolliShop greenfield architecture proposal
- Q-018 — Concierge role workflow design (when role is filled)
- Q-019 — PartsWiki standalone scope finalization
- Q-020 — Cross-app testing harness (e2e flows spanning RS + RW + RC)

---

_End of task queue. Append new tasks below the backlog section as they're identified._
