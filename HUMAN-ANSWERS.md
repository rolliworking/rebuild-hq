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

---

_End of human answers file._
