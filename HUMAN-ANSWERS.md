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

---

_End of human answers file._
