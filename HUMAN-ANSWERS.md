# HUMAN-ANSWERS.md — Answers to Agent Questions

> **For Michael:** When Qwen logs a question in `HUMAN-QUEUE.md`, answer it here. Reference the Q-ID. Agents check this file at session start to unblock work.
>
> **For Qwen:** Read this file first thing each session. Any new answers since your last run are decisions you can apply immediately to resume blocked tasks.

---

## Format

```
## A-YYYYMMDD-### — answers Q-YYYYMMDD-###
**Answered:** YYYY-MM-DD by Michael
**Decision:** [the actual decision, one or two sentences]
**Rationale:** [optional — why this is the right call]
**Action for agent:** [what the agent should do with this answer — e.g. "resume Q-001, apply this as a constraint"]
**Status:** active | superseded
```

---

## Answers

## A-20260628-001 — answers Q-20260626-001
**Answered:** 2026-06-28 by Michael
**Question:** Does the multi-estimate-per-package workflow exist in code?
**Decision:** Not implemented — but needed. Multi-estimate-per-tracking is a real recurring workflow that must be supported in the rebuild. Two patterns: (a) one client shipping multiple jobs on one tracking number, (b) two related clients (e.g., husband and wife) shipping together. The rebuild must support assigning multiple estimates to a single inbound package + backfill workflow when intake happens before estimate creation.
**Action for agent:** Add to rebuild scope: multi-estimate package assignment UI + backfill. Cross-reference with the categorization improvements in WISHLIST (separated client notes field, separated shipping/drop-off/pick-up field, client asset as separate line item type, duplicate-asset prevention).
**Status:** active

## A-20260628-002 — answers Q-20260626-002
**Answered:** 2026-06-28 by Michael
**Question:** Should received_bracelet / received_case be derived from itemType or from receivedComponents?
**Decision:** Derive from actual received components, not itemType. The mental model is that "received case + bracelet without movement" is a real (non-service) intake pattern. Component-to-watch matching also breaks the simple itemType assumption — sometimes a single intake contains a husband's watch head + wife's bracelet, or components from 3 different watches dropped by another watchmaker. The rebuild needs a multi-item entry pattern with auto-numbering (1 of 2, 2 of 2) so each piece can be tracked independently.
**Action for agent:** Treat received_bracelet / received_case as derived from explicit component selection, not itemType. Add to rebuild scope: "add another item" button with auto-numbering for multi-item intakes. Each item gets its own identifier even when grouped under one estimate.
**Status:** active

## A-20260628-003 — answers Q-20260626-003
**Answered:** 2026-06-28 by Michael
**Question:** Does Shop Floor get triggered from intake at all?
**Decision:** This is bigger than a Shop Floor trigger question. The canonical pattern is two-stage: Stage 1 (Drop-off + Shipping Receive pages) captures **possession** — what physically arrived. Stage 2 (Receive Watch page) is a **verification and inventory commit layer** — verifies Stage 1 matches reality, then commits to inventory. The two stages must be auditable against each other to prevent theft and provide a searchable log. Shop Floor's "components aren't registered" failure is downstream of this pattern not being enforced. See D-019.
**Action for agent:** Apply the two-stage pattern as foundational rebuild architecture. Drop-off + Shipping Receive = possession layer (source of truth for what arrived). Receive Watch = verification + inventory commit. Audit log must connect the two. This is a P0 schema-affecting decision — see D-019.
**Status:** active

## A-20260628-004 — answers Q-20260626-004
**Answered:** 2026-06-28 by Michael
**Question:** What minimum component data does Shop Floor need?
**Decision:** Component data varies by type. Watch case: serial number if readable (sometimes not). Bracelet: model number or name, sometimes date code. Other items: name only. Schema must accommodate optional identifiers per component type — not all components have all fields.
**Action for agent:** Design job_components table with: component_type (watch_case, bracelet, dial, bezel, clasp, etc.), name (required), serial_number (optional), model_number (optional), date_code (optional), notes (free text). Per-component-type validation rules can layer on top.
**Status:** active

## A-20260628-005 — answers Q-20260626-005
**Answered:** 2026-06-28 by Michael
**Question:** Is custody_status used by any downstream system?
**Decision:** Currently used only for sales order generation and label reprinting. "This is bad." The rebuild must provide a visual layer for client asset visibility plus an audit capability. custody_status becomes the foundation of the chain-of-custody system (D-015), not a deprecated field. Real-time visibility — what's in the safe, what's at which station, what's released — is a core operational need.
**Action for agent:** Treat custody_status as foundational, not legacy. Build the visual asset layer (D-015 dependency). Audit capability is a hard requirement, not nice-to-have.
**Status:** active

## A-20260628-006 — answers Q-004-1
**Answered:** 2026-06-28 (PM) by Michael
**Question:** What is the correct flow classification for markerless estimates?
**Decision:** The W/B/P/PM markers on estimates are SUGGESTIONS, not authoritative classifications. They indicate which room/department will be doing the work, to speed up the receive watch page workflow. If the markers are missing, the Receive Watch page input will backfill the classification. This confirms D-019: Receive Watch page is the source of truth, estimates are pre-fill hints only.
**Action for agent:** Treat estimate markers as UX speed optimization, not data of record. Receive Watch page determines actual flow classification. The current code that falls through to BAND_ONLY_FLOW when markers are missing is incorrect — it should defer to Receive Watch instead.
**Status:** active

## A-20260628-007 — answers Q-004-2
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Is the station_id fallback in dotStationFor acceptable, or should all rows be migrated?
**Decision:** Need a drag-and-drop GUI for Shop Floor station management. Staff must be able to manually move components between stations. Flow is typically right-to-left, but exceptions are real: client cancels/aborts → move to safe; client changes their mind → move from safe back into queue. Staff are non-developers; they understand a GUI, not code. The current implicit derivation logic is unacceptable as a long-term pattern.
**Action for agent:** Wishlist item created (W-37) for Shop Floor drag-and-drop GUI. For the rebuild, treat station_id as a first-class authoritative field, populated at intake AND mutable via GUI controls. Display layer derives from station_id, not from status+department heuristics.
**Status:** active

## A-20260628-008 — answers Q-004-3
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Should Shop Floor (or a shared service) create job_components at receive time rather than relying on rollisuite-intake?
**Decision:** Receive Watch page (RS) should be the creator of components, not rollisuite-intake on the RW side. The Receive Watch page allows the operator to select department per component, which is the data needed for routing. Component creation belongs at the verification + commit stage (Stage 2 of D-019), not at the RW intake hook.
**Action for agent:** Architectural decision pinned as D-020 (separate from D-019's general two-stage pattern). The component creation responsibility shifts from rollisuite-intake to Receive Watch. The rollisuite-intake edge function becomes a sync/notification mechanism, not a creator.
**Status:** active

## A-20260628-009 — answers Q-004-4
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Does the 3-color carbon work order data have any digital representation?
**Decision:** No. The paper 3-color carbon work orders have no relationship to the digital asset tracking system. They are operational tooling, not data.
**Action for agent:** Document as not digitized. Out of scope for D-015 (chain of custody) — paper forms stay paper.
**Status:** active

## A-20260628-010 — answers Q-005-A
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Do qbo_sync_log rows for sync_type='customer' reach success, or remain stuck at started?
**Decision:** Verified via direct SQL query: the cron has been firing daily but never completing for 30+ consecutive days. Zero success rows, zero failed rows, only stuck 'started' rows. Silent failure pattern. Push-on-fulfill has been silently carrying the system. Customer sync cron is non-functional and has been since at least 2026-01-12 (Lovable found 162 stuck rows from that date forward). Push-on-fulfill is canonical for customer sync. Future scale planning required: may have 100k+ customers eventually.
**Action for agent:** In the rebuild, treat push-on-fulfill as the canonical customer sync path. Do not preserve the current cron — kill or rewrite with chunking (500 rows per transaction), idempotent operations, visible monitoring, and explicit timeout handling. Architectural pattern: silent failure is now banned (D-020). Every async/cron/webhook/edge function must produce observable success/failure status.
**Status:** active

## A-20260628-011 — answers Q-005-B
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Was qbo_sync_log CHECK altered in production to allow sync_type='invoice'?
**Decision:** No — the production constraint was NOT altered. Lovable confirmed: the CHECK still only allows 'customer' and 'estimate', but qbo-invoice-sync line 443 inserts 'invoice'. The INSERT fails silently, the subsequent UPDATE matches 0 rows, the function returns 200. Zero telemetry for any invoice sync ever. Also: there's a related phantom column bug — qbo-invoice-sync line 797 writes to 'finished_at' but the real column is 'completed_at'. Both are code drift, not production patches.

Background context: Intuit changed payload requirements in April 2026 to support their CloudEvents migration. Lovable confirmed the April work was a PARTIAL CloudEvents migration scoped to qbo-payment-webhook — the only inbound webhook in the codebase. As a result, RolliSuite is already CloudEvents-compliant for the July 31, 2026 mandatory deadline. No CloudEvents action needed.
**Action for agent:** Two production fixes captured in documentation/operational-fixes/PENDING.md as PROD-FIX-001. CloudEvents urgency cleared.
**Status:** active

## A-20260628-012 — answers Q-005-C
**Answered:** 2026-06-28 (PM) by Michael
**Question:** What is the actual cron schedule in Supabase dashboard?
**Decision:** Deferred — Michael did not have visibility into the Supabase dashboard at answer time.
**Action for agent:** Re-ask in a future questions snapshot once Supabase dashboard access is reviewed.
**Status:** deferred

## A-20260628-013 — answers Q-005-D
**Answered:** 2026-06-28 (PM) by Michael
**Question:** How many received POs have qbo_bill_id or JE notes populated?
**Decision:** Deferred — Michael asked to be re-asked in a week.
**Action for agent:** Re-include this question in the next questions snapshot after 2026-07-05.
**Status:** deferred

## A-20260628-014 — answers Q-006-A
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Should drop-off intake photos (ClientWatchEntryPage) also appear in Pickup Station reference grid?
**Decision:** Yes. Pickup Station should display intake photos (from drop-off OR shipping receive — both sources) as "before" reference photos, alongside the items being released at pickup as "after" photos. The side-by-side comparison is critical to prevent mislabeling errors during return, which do happen.
**Action for agent:** Wishlist item created (W-38) for side-by-side before/after photo comparison at pickup. The rebuild must unify intake photo source per D-019 (Stage 1 = possession photos, applies to both drop-off and shipping receive).
**Status:** active

## A-20260628-015 — answers Q-006-B
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Should completePickup write custody release on client_property?
**Decision:** Yes — pickup completion should write a custody release on the client asset record. The release is a chain-of-custody event and must be captured.
**Action for agent:** Wire the pickup completion path to update custody_status (or the equivalent rebuild field) when a pickup session completes. This is a foundational hook for D-015 (chain of custody).
**Status:** active

## A-20260628-016 — answers Q-006-C
**Answered:** 2026-06-28 (PM) by Michael
**Question:** Why is pickup bypass open to all staff but ship bypass is restricted?
**Decision:** Intentional, with business rationale: In-person pickup involves taking payment at the counter before releasing the watch. Payment registration in QBO has latency, so if staff confirm payment was accepted, the bypass exists to avoid blocking the customer at $0 balance. Shipping is the opposite workflow — slower in nature (receive return address, pack, ship), so the bypass is restricted because there's time to verify balances properly.
**Action for agent:** Document as intentional operational policy. The rebuild's RBAC model should preserve this asymmetry: pickup bypass available to authenticated counter staff, shipping bypass restricted to managers.
**Status:** active

## A-20260629-001 — answers Q-005-C
**Answered:** 2026-06-29 by Michael (via pg_cron query)
**Question:** What is the actual cron schedule in Supabase dashboard?
**Decision:** Six active pg_cron jobs (UTC), sourced from direct pg_cron.job query:
1. `qbo-daily-sync` — daily 06:00 → `qbo-cron-sync` (customer sync)
2. `maintenance-hourly` — every hour on the hour → `maintenance-cron`
3. `qbo-invoice-sync-hourly` — every hour at :15 → `qbo-invoice-sync`
4. `rw-status-daily-5am` — daily 10:00 UTC (~05:00 CDT / 06:00 EDT) → `rw-status-cron`
5. `vault-storage-fee-daily` — daily 08:00 → `vault-storage-cron`
6. `daily-gold-price-update` — daily 12:00 → `gold-price-update`

Reframes prior assumptions:
- Q-005 initial guess said "single daily cron; invoice sync manual-triggered" — WRONG
- Invoice sync runs 24×/day via cron, but silently drops every log entry due to CHECK constraint bug (PROD-FIX-001)
- Two additional crons need D-020 audit trail: `maintenance-hourly` (unknown scope, 24×/day) and `vault-storage-fee-daily` (direct revenue impact if silent-fails)

**Action for agent:** Store the cron inventory in Q-005 discovery file. Amend PROD-FIX-001 rationale to reflect 24×/day invoice sync context. Add W-42 for D-020 audit of `maintenance-hourly` and `vault-storage-fee-daily` crons.
**Status:** active

## A-20260629-002 — answers Q-005-D
**Answered:** 2026-06-29 by Michael
**Question:** How many received POs have qbo_bill_id or JE notes populated?
**Decision:** Volume is 2–10 purchase orders per week. Push is manual after receiving items. Very low volume, not worth investment in cron-driven sync.
**Action for agent:** Confirm PO path stays as manual accounting bridge in the rebuild. No cron. Preserve the manual JE + Bill flow, but ensure both steps log to a durable audit surface per D-020 (Bill step failure after JE succeeds is a real risk — see Q-005 findings).
**Status:** active

## A-20260629-003 — answers Q-008-A
**Answered:** 2026-06-29 by Michael
**Question:** Should RT→RS test results write to legacy `timing_tests` or new `shared.timing_test_results`?
**Decision:** New shared table that all apps can read. Legacy RS `timing_tests` and `pressure_tests` stay read-only for historical jobs. The strategic use case: RT test data + inspection photos + service history feed into AI-generated warranty summary reports for clients. When a warranty issue arises or a client questions the quality of work, an AI report is generated pulling test data, photos, and job history to explain and document the work performed.
**Action for agent:** New shared schema table (e.g., `shared.timing_test_results` or `rt.timing_test_results` — see D-022-related work on canonical model). Legacy tables remain queryable for historical jobs. Create W-40 (AI warranty summary reports) as a downstream strategic feature.
**Status:** active

## A-20260629-004 — answers Q-008-B
**Answered:** 2026-06-29 by Michael
**Question:** Realtime vs poll for RS→RT reference sync?
**Decision:** Poll every 15 minutes. On-demand lookup at scan time as a secondary refresh path. Realtime not needed for launch.
**Action for agent:** Implement 15-minute delta poll pattern per the Q-008 discovery best-guess. Realtime deferred to future decision.
**Status:** active

## A-20260629-005 — answers Q-009-A
**Answered:** 2026-06-29 by Michael
**Question:** Migrate historical intake photos to R2 or keep Supabase Storage?
**Decision:** Standardize on R2 for new photos going forward. Keep old photos where they are (Supabase Storage). Build the `shared.intake_photos` index to bridge the two locations transparently. Migration of historical photos deferred indefinitely; retirement of Supabase Storage buckets happens naturally over time as photos age out.

Rationale: R2 has zero egress fees, essential for client-page photo galleries and future AI training data pipelines. Historical photos are low-view-frequency; migration cost isn't justified.
**Action for agent:** Formalized as D-021 (photo storage strategy). All new photo capture in the rebuild writes to R2. Index table is authoritative source for photo lookup regardless of physical storage location.
**Status:** active

## A-20260629-006 — answers Q-009-B
**Answered:** 2026-06-29 by Michael
**Question:** Should pickup before/after require paired photo counts (1:1)?
**Decision:** Human verification, not hard block. The Pickup Station UI must show the intake before photos alongside the release items, and the operator must click a confirmation prompt verifying that the items being returned match the intake photos. Soft workflow with mandatory acknowledgment.
**Action for agent:** W-38 implementation is a UI verification pattern: display before photos side-by-side with current release items, require operator to check "I verify these items match" before completing pickup. No count-based hard block. Log the operator's acknowledgment to the audit trail per D-020.
**Status:** active

## A-20260629-007 — answers Q-011-A
**Answered:** 2026-06-29 by Michael
**Question:** Should received_components drive job_components row creation at intake?
**Decision:** Yes, at the Receive Watch stage (not at shipping/dropoff, and not at rollisuite-intake edge function).

The two-stage intake model is more specific than originally captured in D-019:
- **Stage 1a — Shipping Receive page:** Captures package arrival data (data collection layer)
- **Stage 1b — Drop-off page:** Captures walk-in intake data (data collection layer)
- **Stage 2 — Receive Watch page:** The verification + inventory commit + component mapping layer. This is where staff view all upstream data (what was shipped/dropped-off, quantity of items, viewable estimate) and correctly enter/verify the components.

Missing UI feature: **variance notifier**. When received items don't match the estimate (more or fewer items than expected), staff currently have to go back, modify the estimate, and restart the receive process. Should be an inline notifier at the Receive Watch page that flags variance and offers a resolution path.

**Action for agent:** Amend D-019 to reflect the three-page structure (Shipping Receive + Drop-off + Receive Watch) and precise commit-point at Receive Watch. Component row creation happens at Stage 2. Create W-41 for the variance notifier feature.
**Status:** active

## A-20260629-008 — answers Q-011-B
**Answered:** 2026-06-29 by Michael
**Question:** Should RS intake save trigger RW push immediately?
**Decision:** Yes, but only after labels are printed. When labels are printed, RS pushes to RW with authority to overwrite any previous input for that watch (prevents duplicates from earlier partial pushes).
**Action for agent:** Preserve label-wizard as the canonical push trigger. Ensure the push is idempotent-with-overwrite semantics: incoming push replaces any prior RW state for that watch record. Log every push to audit trail per D-020.
**Status:** active

## A-20260629-009 — answers Q-013-A
**Answered:** 2026-06-29 by Michael
**Question:** Should safe bins be modeled as station_id values or a separate storage_slot entity?
**Decision:** Reuse the `station_id` namespace for safe bins (e.g., `safe_bin_A1`, `safe_bin_A2`). Uniformity across all physical locations (workshop stations, safe bins, polish room slots, etc.) simplifies the location model and enables W-37 drag-and-drop across all of them.
**Action for agent:** Extend the station_id namespace to include safe bins. Department metadata on each station distinguishes types (`safe_storage`, `watchmaker_bench`, `polish_room`, `receiving`, etc.). Bin ordering (Vianna sorts by due date) becomes a UI concern with sort keys on the station record.
**Status:** active

## A-20260629-010 — answers Q-013-B
**Answered:** 2026-06-29 by Michael
**Question:** Is RS `client_property` or RW `job_components` the canonical row for W-33 visual inventory?
**Decision:** Neither in isolation. A watch has one identity — its serial number. That identity never changes.

When a watch arrives, it's whole: **1 of 1** (one piece, one location).

When service requires disassembly, the count reflects how many pieces currently exist. A watch split into head, bracelet, and case for parallel work becomes **1 of 3, 2 of 3, 3 of 3** — three pieces, each potentially at a different station, all belonging to the same watch identity.

When work completes and everything reassembles, the count returns to **1 of 1**. Reunification.

The canonical spine for W-33 is the **watch record** (one row per unique serial number). Components are not independent things — they're pieces of one watch, temporarily separated during service. The X of Y notation captures the current assembly state at a glance.

Data model implications:
- Watch table has one row per unique serial number (the identity)
- Piece table has one row per piece currently in existence
- Every piece row references its parent watch via FK — no piece can exist without a watch
- Each piece has: piece_number, piece_total, piece_type (head, bracelet, case, whole watch), station_id, staff_id, moved_at
- When a watch is whole: one piece row (1 of 1, type "whole watch")
- When a watch is disassembled: multiple piece rows sharing the same watch_id
- When pieces reunify: rows collapse back to one (1 of 1)

UI implications for W-33: The dashboard shows watches, grouped by customer. Each watch card displays its current assembly state (1 of 1 · In safe, OR 1 of 3, 2 of 3, 3 of 3 · Distributed across stations). Expandable to see per-piece location and custody.

UI implications for W-37: Staff drag pieces between stations. Every drag is logged as piece X of Y moved from Station A to Station B for a specific watch_id. When all pieces of a watch land at the same station, the UI signals ready for reunification.

Chain-of-custody implications (D-015 / D-019 audit trail): Every piece movement produces an audit log row: watch_id, piece_id, from_station, to_station, staff_id, timestamp. Reunification is when all piece rows for a watch collapse back to one.

**Action for agent:** Formalize as D-022 (watch assembly state model). This is a foundational rebuild decision affecting SPEC-001 (two-stage intake), SPEC-005 (Shop Floor drag-drop), and any UI displaying watch inventory. The client_property vs job_components framing was based on a wrong assumption of competing identities; the correct model is one identity (watch) with N current-state pieces.
**Status:** active

## A-20260629-011 — noted for next snapshot: Q-005-C misfile
**Answered:** 2026-06-29 by Cursor Agent (procedural)
**Question:** Correction — the YOUR ANSWER for Q-005-C in the QUESTIONS-2026-06-29-am.docx doc contained the Lovable investigation content that was actually the answer to Q-005-B (already committed as A-20260628-011). The real answer to Q-005-C came separately via pg_cron query and is captured as A-20260629-001 above.
**Action for agent:** No action needed. This entry documents the paste error for audit clarity.
**Status:** noted

## A-20260630-001 — answers Q-007-A
**Answered:** 2026-06-30 by Michael
**Question:** Should customer email replies (currently sent to noreply@) automatically route into the customer's RolliConnect conversation thread?
**Decision:** Yes. Reply-token routing applies to ALL customer-facing emails from RolliSuite: shipping notifications, estimates, inspection notes, parts approvals. Every outgoing email carries a unique reply token in the Reply-To header. Replies land in the correct RC conversation for the correct client, automatically.

Additional scope surfaced by this answer: today Rolliworks has some clients who exist as email-only contacts (no RC client record). The rebuild needs a way to convert email-only clients into structured RC clients so the conversation history has a real client to attach to.
**Action for agent:** Wire every outbound email path (shipping, estimate, inspection, parts approval) with a per-send RC conversation token in Reply-To. Add W-46 for email-only-client-to-RC migration. RolliConnect becomes the destination for all customer email traffic in the rebuild.
**Status:** active

## A-20260630-002 — answers Q-007-B
**Answered:** 2026-06-30 by Michael
**Question:** Where should the one-button bulk "testing complete" email live — in RolliWorking (watchmaker workspace) or RolliSuite (template owner)?
**Decision:** Neither. Live on RolliConnect. Mike Michaels will own testing-complete marking going forward as part of his management role starting mid-July.

This is an ownership shift: testing-complete moves from watchmaker-action (RW) to management-action (RC-hosted UI). Aligns with the emerging pattern of RC as central client communication hub.
**Action for agent:** Bulk testing-complete email trigger lives on RC. RW notifies RC when testing status changes for a batch of jobs. Mike Michaels (or manager role per Q-010) reviews the batch and triggers send. RC handles template rendering, delivery, audit log, reply-token routing per A-20260630-001. Update Q-010 (cross-app auth) planning to reflect manager role reviewing testing-complete queue.
**Status:** active

## A-20260630-003 — answers Q-012-A
**Answered:** 2026-06-30 by Michael
**Question:** When an existing RC customer submits a new web form / portal request, should it join their existing thread or start a new one?
**Decision:** One client = one client (no duplicate client records). Within that single client's record, requests are organized in dated folders. Folder structure: conversations, requests, archive, answered.

Each new request creates a new folder named by request date. Reply-to-message buttons in the conversation UI route replies to the correct folder. Staff can move messages between folders manually when needed. This gives the operational separation of "one request, one folder" while preserving the single-client entity.

Pattern in plain terms:
- Client: John Smith (one record)
- Folder: "2026-05-14 request" (contains all messages for that specific service request)
- Folder: "2026-06-22 request" (separate service request, separate folder)
- Folder: "archive" (old resolved requests)
- Folder: "answered" (recently resolved)
- Reply button on a message routes reply into the same folder as the original message
**Action for agent:** RC data model supports one-client-many-conversations with folder-based organization. Folder is a first-class field on the conversation. Reply-token routing (A-20260630-001) resolves to the folder the original message lived in. Merge/split UI (per Q-012 findings) lets staff move messages between folders. Add to SPEC planning for RolliConnect rebuild.
**Status:** active

## A-20260630-004 — answers Q-014-A
**Answered:** 2026-06-30 by Michael
**Question:** Should Component Status page (older, mouse-only) merge into Shop Floor page (station map) with drag-drop per W-37?
**Decision:** Yes. Unify into a single map view with a search field. The map has two modes based on interaction:
- **Search mode (lookup):** operator types in the search field, map highlights component locations for matches. Component Status functionality preserved.
- **Assign mode:** operator clicks a clickable position on the map to bulk-assign components to that station. Shop Floor bulk-assignment functionality preserved.

Additional need surfaced: a separate tab that shows which jobs a specific tech has, with a way to generate a report. This is production reporting per tech — directly connects to the tech breadcrumb pattern (W-45) and closes the loop with the QBO product-code workaround being retired.
**Action for agent:** SPEC-005 (Shop Floor / iPad UI) unifies Component Status and Shop Floor into single map view with dual modes. Add search field as first-class UI control. Add "Jobs per Tech" tab with report generation. Production report queries the same `job_tech_involvements` join table that powers W-45. Both W-47 and W-45 share the same data plumbing.
**Status:** active

## A-20260630-005 — answers Q-015-A
**Answered:** 2026-06-30 by Michael
**Question:** When a package arrives with no estimate, should the system flag/escalate after N days?
**Decision:** Not primarily a stagnation problem — primarily a workflow gap. When a package arrives (or a client walks in) without a prior estimate, staff need a one-step way to: generate the next estimate number, look up or add the client, auto-create a memo line describing what was received, save the estimate, then proceed with the drop-off or shipping-receive flow.

Michael's current workaround: stop the intake flow, create the estimate with a memo line, save it, then re-enter the drop-off flow. The rebuild should collapse this into a single continuous flow.

Additional architectural signal: "Drop off and shipping receive modules are very similar and share a lot of features. Same goes for pickup and shipping modules." This validates D-019 two-stage intake pattern (drop-off + shipping-receive both = Stage 1 possession) AND extends it to the outbound side (pickup + shipping-outbound share similar structure).
**Action for agent:** Add W-48 for auto-generate-estimate at intake. Amend D-019 to note that outbound (pickup + shipping-outbound) follows a mirror two-stage pattern. Stagnation tracking still applies as a fallback safety net, but the primary fix is the workflow gap. Update SPEC-001 (two-stage intake) planning to include auto-generate estimate as part of Stage 1a and Stage 1b flow.
**Status:** active

## A-20260630-006 — answers SPEC-PREP-002
**Answered:** 2026-06-30 by Michael
**Question:** Migrate all historical data or use a cutoff date?
**Decision:** Migrate everything. Data older than 18 months = read-only import. Data within the last 18 months = editable import.

The 18-month cutoff applies uniformly to customer records, watch records, jobs, invoices, and any other historical operational data. Older records remain queryable and editable-in-view (for reference, warranty lookup, historical context) but not editable-in-place (no writes against the archived data).
**Action for agent:** Feeds SPEC-MIG-* planning per D-029. Migration workstream includes a `migration_read_only_flag` on migrated records where `job_completed_at` (or equivalent) is older than 18 months from cutover date. RS UI enforces the read-only flag for editing but shows the data. Full-text search and lookups work identically on read-only and editable data.
**Status:** active

## A-20260630-007 — answers SPEC-PREP-003
**Answered:** 2026-06-30 by Michael
**Question:** How firm is the February 2027 facility opening date?
**Decision:** Facility opening timeline is real, but a hidden critical path has emerged: **Lovable is breaking.** Some rebuild capabilities need to be operational sooner than the Labor Day 2026 cutover, not waiting for facility opening.

Michael explicitly stated: "We need some type of operational rs rc and rw soon. We will be simultaneously building a prototype of RolliTime as well for learning."

Sequencing implication:
- Rebuild slices ship AS SOON AS they can replace failing Lovable functionality, not on a preset schedule
- Cutover schedule remains a planning tool but individual cutovers may accelerate based on Lovable degradation
- RolliTime prototype ships 9/1/26 as a learning vehicle in parallel per SPEC-PREP-012
**Action for agent:** Update RISK-REGISTER-1-31-2027.md with "Lovable stability risk driving earlier partial cutovers" as a HIGH-severity risk. Update the cutover milestone schedule to note that dates are targets, not commitments — earlier cutover is acceptable when a rebuild slice can safely replace a degrading Lovable capability. PROD-FIX-001, PROD-FIX-002, PROD-FIX-003 remain in-scope as emergency stability patches per D-025. Consider adding an ongoing Lovable stability monitoring practice (weekly health check of the three Lovable apps) as an operational discipline.
**Status:** active

## A-20260630-008 — answers SPEC-PREP-004
**Answered:** 2026-06-30 by Michael
**Question:** When Michael shifts to home-based work mid-July, who covers day-to-day decisions and escalations?
**Decision:** Michael is not shifting to fully home-based. He works home-based on **Fridays only**. Combined with Saturday and Sunday, this creates 3-day work spans dedicated to rebuild/software work. Monday through Thursday, he remains on-site for shop operations, decisions, escalations, and staff support.

This significantly reduces the Transferability Test urgency around role-based approvals for the July timeframe. However, the underlying Transferability Test requirement remains valid architecturally: approvals in the software should route to a role (owner, manager, front_desk), not to a specific person (Michael, Vianna). Mike Michaels arriving end of July with expanded management role continues to reinforce role-based design.
**Action for agent:** No immediate coverage matrix required — Michael is on-site 4 days/week. Continue designing all approval workflows as role-based per Transferability Test (D-023). Update SCOPE-HOLD-1-31-2027.md working pattern note: 3-day rebuild work spans (Fri-Sat-Sun) at home, 4 days on-site. No coverage matrix artifact required for July. Q-010 (cross-app auth) planning continues to prioritize role-based permissions over name-based routing.
**Status:** active

## A-20260630-009 — answers SPEC-PREP-005
**Answered:** 2026-06-30 by Michael
**Question:** What must be live at facility opening 1/31/2027?
**Decision:** The 1/31/2027 IN scope is:

1. **Feature parity with current RS + RW + RC** — everything the Lovable apps do today, the rebuild does. Working better, more functional, no data silos.
2. **Canonical data model** — one client, one watch, one job, one custody chain across the ecosystem. No fragmented duplicates per D-028.
3. **Multi-entity operational model** — the software supports Rolliworks + RolliShop operating as two distinct entities within one facility. See D-030 for full scope. **NOT physical multi-location** — that's post-1/31 scope.
4. Plus all previously committed 1/31 items: two-stage intake (D-019), Shop Floor unified view with fast lane (W-44 + W-47), photo storage index (D-021), two-Supabase-project architecture (D-029), RGTime as staff master (D-026), chain-of-custody log (D-015), fresh Supabase for rebuild.
5. Plus new items from this cycle: client-facing self-service kiosk (W-49), voice memo to-do list (W-52), thin experimental Authenticator on inspection photos (W-53), Frigate NVR integration (W-51).

Michael's exact framing: "same functionality of current rs rw and rc but working better, more functional and no data silos. By 1/31 we need multi store ready."
**Action for agent:** Update SCOPE-HOLD-1-31-2027.md IN list with feature parity as baseline requirement + multi-entity operational model + all specific items listed. Every SPEC written between now and 1/31 must pass the Transferability Test AND the multi-entity test (can Rolliworks + RolliShop both use this feature under the same roof?).
**Status:** active

## A-20260630-010 — answers SPEC-PREP-006
**Answered:** 2026-06-30 by Michael
**Question:** What is explicitly OUT of scope for 1/31/2027?
**Decision:** OUT of scope for 1/31/2027:

- Physical multi-location expansion (second building)
- Customer-facing Authenticator V1 (deferred to Q2-Q4 2027 per POST-FACILITY-ROADMAP)
- RolliCurator wizard (deferred)
- PartsWiki (deferred)
- M3KE chatbot embedded in RolliSuite (deferred)
- Full historical photo migration (deferred per D-021)
- Paper carbon work order digitization (out of scope per Q-013)
- Full Jarvis AI capabilities (agentic, LLM-based, cross-app queries) — post-facility Q2-Q4 2027

IN scope for 1/31/2027 (from SPEC-PREP-005 + this answer):
- Multi-entity operational model per D-030 (Rolliworks + RolliShop, same facility)
- Basic Jarvis: room-to-room paging, whole-store paging, voice memo to-do list capture per D-023 amendment
- Thin experimental Authenticator: % authenticity score on inspection photos, internal-only, staff-feedback-captured per W-53
**Action for agent:** Update SCOPE-HOLD-1-31-2027.md OUT list with explicit items. Update POST-FACILITY-ROADMAP-Q2-Q4-2027.md to note thin Authenticator (W-53) precedes full Authenticator V1 — thin version collects training data during 2027 which then informs full V1. Update D-023 to reflect the voice memo addition to Jarvis MVP scope.
**Status:** active

## A-20260630-011 — answers SPEC-PREP-007
**Answered:** 2026-06-30 by Michael
**Question:** Should software station names mirror physical floor plan?
**Decision:** Yes — mirror physical floor plan. Software station IDs match the labels printed at physical stations in the new facility. Safe bin numbering matches physical safe slots (per A-20260629-009 already committed).
**Action for agent:** SPEC-002 canonical data model uses `station_id` values that match printed physical labels in the Feb 2027 facility. Floor plan naming convention must be finalized during facility buildout planning (Aug-Oct 2026) so software station IDs can be locked before SPEC-005 (Shop Floor) finalizes. Add facility buildout note to FACILITY-INTEGRATION-CHECKLIST.md.
**Status:** active

## A-20260630-012 — answers SPEC-PREP-008
**Answered:** 2026-06-30 by Michael
**Question:** Kiosk and iPad placement in the new facility?
**Decision:** Four distinct device categories with specified mounts:

1. **Front kiosk** — wall-mounted, client-facing. Client enters their info + job request (like filling out a website service request but via kiosk). See W-49.
2. **Watchmaker iPad** — mobile. Carried around by watchmakers as they move between bench and stations.
3. **Band room iPad** — mobile. Used for parts requests and band-department workflows.
4. **RGTime iPad** — wall-mounted. Dedicated kiosk for clock in/out per RGTime concept.

Additional inferred devices (from operational context, confirm during facility planning):
- Receiving desktop + label printer (still applies from prior discovery)
- Pickup station kiosk (client-facing at exit, TBD if mounted or shared with front kiosk)
- Concierge iPad (per D-024 fast lane, mobile or station-mounted TBD)
**Action for agent:** Update FACILITY-INTEGRATION-CHECKLIST.md with the four confirmed device categories + mount types. SPEC-001 (two-stage intake) design accommodates client-facing kiosk as intake entry point (Stage 0 pre-Stage 1). SPEC-005 (Shop Floor) design supports mobile watchmaker iPad. RGTime SPEC (when drafted) accommodates wall-mount UI. Network cabling plan per SPEC-PREP-011 accounts for these four device locations plus PoE for cameras.
**Status:** active

## A-20260630-013 — answers SPEC-PREP-009
**Answered:** 2026-06-30 by Michael
**Question:** How should the physical safe integrate with software?
**Decision:** Bulk-assign pattern:

- Optional: barcode on safe door for zone entry event
- Primary workflow: a folder containing all job QR code labels stays near the safe
- To bulk-assign jobs to safe: click "safe" button in Shop Floor (assign mode per W-47) → scan job labels one after another → all scanned jobs are assigned to safe in one bulk operation
- No per-slot scanning inside the safe. Physical safe bin location assumed (staff knows where they put it).

This aligns with W-37 bulk assignment pattern (click position on map = bulk assign) and W-47 (Shop Floor unified view with search + assign modes). Safe is one of the clickable positions on the Shop Floor map.
**Action for agent:** SPEC-005 Shop Floor implements "safe" as a clickable position on the map. Clicking "safe" enters bulk-scan mode. Each scanned job QR moves that job's custody to `in_safe_for_pickup` or equivalent state per D-024 state machine. Per-slot scanning inside safe deferred as a possible future enhancement (post-1/31). No custom hardware required — uses same iPad + job QR labels already in use.
**Status:** active

## A-20260630-014 — answers SPEC-PREP-010
**Answered:** 2026-06-30 by Michael
**Question:** IP camera positions and retention requirements?
**Decision:** Split-tier system with Frigate as unified NVR. Non-custody zones use RTSP-compatible Wi-Fi cameras (Reolink or Amcrest) for Nest-like install simplicity without cloud/subscription lock-in. Critical zones (safe, receiving, front desk, active benches) use PoE cameras for reliability. Both tiers feed the same Frigate system, which integrates with the rebuild's chain-of-custody log via MQTT/webhook. Cabling plan for Feb 2027 buildout only needs Cat6 runs to Tier 1 zones.

Full architecture spec captured in W-51.
**Action for agent:** Add W-51 (Frigate NVR + split-tier camera architecture) to WISHLIST.md. Update FACILITY-INTEGRATION-CHECKLIST.md with camera placement plan for Tier 1 (safe interior + exterior, receiving station, front desk, main entrance, rear exit, active work benches) and Tier 2 (general workshop, break room, hallways). Update SPEC-PREP-011 network plan with PoE runs for Tier 1 cameras. Retention policy: 30-90 days minimum per insurance requirements (confirm with insurer before final camera specification).
**Status:** active

## A-20260630-015 — answers SPEC-PREP-011
**Answered:** 2026-06-30 by Michael
**Question:** Network and power requirements at each station?
**Decision:** Hardwired Ethernet to benches and safe. PoE switch for cameras. Separate IoT VLAN for cameras and IoT devices.

Confirmed infrastructure baseline for Feb 2027 facility:
- Cat6 runs to every fixed station (benches, safe, RGTime kiosk, front kiosk, receiving desk, pickup)
- PoE switch (managed) for Tier 1 cameras and any PoE peripherals
- Business-class Wi-Fi (Ubiquiti UniFi or equivalent) for mobile iPads and Tier 2 Wi-Fi cameras
- Separate IoT VLAN for cameras isolated from staff/customer traffic
- UPS at safe area and network closet
- Guest Wi-Fi network isolated from operational network
**Action for agent:** Update FACILITY-INTEGRATION-CHECKLIST.md with confirmed network topology. IT install ownership: Michael + external IT contractor for cabling/switch install; Michael + Cursor for software configuration. Add to facility buildout schedule: network cabling must precede drywall.
**Status:** active

## A-20260630-016 — answers SPEC-PREP-012
**Answered:** 2026-06-30 by Michael
**Question:** Does the testing station move to the new facility day one, and must RolliTime integration be live for opening?
**Decision:** RolliTime prototype starts being used **9/1/2026** (Labor Day 2026 — cutover 1 milestone). Uses the same rebuild methodology: discovery → questions → SPEC → BUILD after some learning done in production use.

This means RolliTime enters "learning mode" 9/1/26 with the existing prototype. Real usage generates data + surfaces issues. Discovery task packet drafted for whichever session Michael queues it. Formal RolliTime rebuild SPEC follows after ~1-2 months of learning-mode usage.

RolliTime is confirmed IN scope for 1/31/2027 facility opening.
**Action for agent:** Update cutover milestone schedule: 9/1/2026 includes RolliTime prototype live in learning mode. Draft Q-RT-001 discovery task packet for later Cursor autonomous execution — investigates current RolliTime prototype code, actual usage patterns, and surfaces architectural questions. Add "RolliTime SPEC drafting" to the SPEC weekend rotation for late 2026 (specific weekend TBD based on learning-mode timing).
**Status:** active

## A-20260630-017 — answers SPEC-PREP-013
**Answered:** 2026-06-30 by Michael
**Question:** How much lead time do staff need to train on rebuilt flows? Should we run parallel old/new systems?
**Decision:** Training approach: **operating instructions drafted as a training manual**. No parallel run. Big-bang cutover per app slice. Michael is available on-site as a live resource for questions during rollout.

Michael's team learns as they go. Michael's management philosophy per prior sessions: team learns from pain and Michael is present to answer questions.

This aligns with cutover milestone schedule — each cutover ships as one event, no rehearsal period. Michael's 4-day on-site presence supports live Q&A during rollouts.
**Action for agent:** For each app slice cutover (Labor Day 2026, Oct, Nov, Dec, Jan), Cursor drafts a plain-language operating instructions manual as part of the cutover package. Manual covers: what the app does, how to do the common workflows (intake, pickup, assign, etc.), what to do if X happens, who to ask. No formal training curriculum. No parallel-run infrastructure required. Add "training manual" as a required deliverable in every cutover milestone package.
**Status:** active

---

_End of human answers file._
