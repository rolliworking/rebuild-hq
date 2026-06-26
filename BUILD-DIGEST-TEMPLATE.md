# BUILD-DIGEST-TEMPLATE.md — 72-Hour Build Summary

> **Purpose:** A single-page snapshot of what was built, decided, and blocked over the last 72 hours. Generated every 3 days. Scannable in under 2 minutes.
>
> **Filename convention:** `documentation/digests/DIGEST-YYYY-MM-DD.md` (use the date the digest covers UP TO).
>
> **Who generates this:** Opus / Claude at the end of every 72-hour window, reading the file system state. Can also be triggered manually anytime.

---

## How to generate

Opus reads these inputs to build the digest:

1. `documentation/builds/*/06-DELTA.md` — any DELTAs dated within the window
2. `DECISIONS-REGISTRY.md` — any decisions dated within the window
3. `documentation/discovery/` — any discovery files dated within the window
4. `HUMAN-QUEUE.md` — open questions
5. `TASK-QUEUE.md` — current status of all tasks
6. `BUILD-LOG`s in any in-progress slices

Output strictly follows the template below. **One page. Bullet points only. No prose.**

---

# BUILD DIGEST — [WINDOW START] to [WINDOW END]

**Generated:** [YYYY-MM-DD]
**Window:** 72 hours
**Status:** [GREEN — on track / YELLOW — friction / RED — blocked]

---

## Built and shipped (slices marked DELTA complete this window)

- **[slice-id]** — [one-line summary] — [commit SHA range]
- [...]

_(If none: "Nothing shipped this window.")_

---

## In progress (slices between SPEC and DELTA)

- **[slice-id]** — [stage: SPEC / PACKET / BUILD / REVIEW / ACCEPTANCE / DELTA] — [owner] — [days in current stage]
- [...]

_(Flag any slice stuck in one stage > 7 days.)_

---

## Decisions made

- **D-### — [title]** — [one-line decision]
- [...]

_(Reference D-### only. Don't restate full decision. If none: "No new decisions.")_

---

## Discovery completed

- **Q-### — [title]** — [output file path] — [key finding in one line]
- [...]

_(If none: "No new discovery this window.")_

---

## Open human questions (pending answers)

- **Q-YYYYMMDD-###** — [task it blocks] — [days open] — [one-line question]
- [...]

_(Flag any question open > 3 days.)_

---

## Blockers

- [blocker description] — [what's affected] — [needed to unblock]
- [...]

_(If none: "No blockers.")_

---

## Wishlist items closed this window

- [item] — [closed by slice-id]
- [...]

_(If none: "No wishlist items closed.")_

---

## Focus for next 72 hours

- [planned slice or discovery task]
- [planned slice or discovery task]
- [planned slice or discovery task]

_(Max 5 items. If empty, that's a planning gap.)_

---

## Risks worth surfacing

- [risk, one line] — [severity: low/medium/high]
- [...]

_(If none: "No new risks.")_

---

## Numbers

- Slices completed lifetime: **[N]**
- Slices in progress: **[N]**
- Decisions in registry: **D-001 through D-[NNN]**
- Discovery tasks complete: **[N] of [N]**
- Open human questions: **[N]** (oldest: [days])
- Open blockers: **[N]**

---

_End of digest. If this took longer than 2 minutes to read, something's wrong with the format._

---

## STOP — Format rules for Opus generating this

1. **One page. Hard limit.** If a section is long, trim items to the highest-impact 3-5 and note "see file X for full list."
2. **Bullets only. No prose.** Each bullet is one line.
3. **Reference, don't restate.** Use D-### / Q-### / slice-id to point at the source. Don't copy content.
4. **Flag don't fix.** Surface friction, don't try to solve it inside the digest.
5. **Honest status:** GREEN / YELLOW / RED at the top is the most-read line. Pick it deliberately.
6. **If no activity:** explicitly say "Nothing shipped" or "No new decisions." Empty sections look like data was missed.

---

_End of template._
