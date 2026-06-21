# Skill: mapping-legacy-workflows

## Purpose

Convert existing app behavior (code, UI, business habits) into structured workflow documentation. Captures operational truth before any rebuild work touches the code.

## When to invoke

- Documenting current-state of RolliSuite, RolliWorking, RolliConnect, or RolliTime before changing it
- Capturing a workflow from a domain expert (Vianna, watchmaker supervisor, kiosk operator)
- Writing CONTEXT_DOSSIER files
- Before drafting any task packet that touches existing functionality

## When NOT to invoke

- Greenfield modules with no prior implementation
- Pure schema design (use `modeling-canonical-data`)
- PR review (use `reviewing-prs-governance`)

## Inputs

- A target app, module, or workflow to document
- Access to one of: source code, live app, screen recording, or a domain expert interview transcript

## Process

1. **Identify entry points.** What triggers this workflow? UI action, scheduled job, webhook, API call.
2. **Trace data flows.** What tables get read? What gets written? What gets transformed?
3. **Document side effects.** Emails sent, edge functions called, status changes propagated, files uploaded.
4. **Capture human steps.** Where does a person make a decision or take a manual action? Name the role.
5. **Note workarounds.** Where do people work around the system? Those are bugs masquerading as features.
6. **Identify edge cases.** What happens when input is missing, invalid, or unexpected?
7. **Flag silos.** Where does data go in but never come out? Where is it lost vs. siloed?

## Output structure

Every workflow doc must include these sections, in order:

1. **Workflow name** and one-line purpose
2. **Trigger** — what starts this
3. **Actors** — who or what touches this (humans, apps, edge functions)
4. **Inputs** — fields, files, scans, anything entering the workflow
5. **Steps** — numbered sequence with side effects per step
6. **Outputs** — what gets created, updated, sent, or stored
7. **Cross-app touches** — which other apps receive or send data, by what mechanism
8. **Edge cases & failure modes** — what goes wrong, and what happens when it does
9. **Workarounds observed** — manual fixes, undocumented shortcuts, "just don't do that" rules
10. **Open questions** — explicit gaps that need a human or further research to resolve

## Anti-patterns

- ❌ Documenting intent instead of actual behavior. ("It's supposed to send an email" — does it?)
- ❌ Skipping the workaround section. That's where the real knowledge lives.
- ❌ Assuming the documented contract matches the implemented contract. They diverge — verify.
- ❌ Documenting code without capturing the human steps around it.
- ❌ Stopping at "happy path." Edge cases are where the rebuild will trip.

## Output destination

- `documentation/workflows/<app>-<workflow-name>.md`
- Cross-reference from CONTEXT_DOSSIER and DAILY-WORKFLOW-MAP

## Related skills

- `modeling-canonical-data` — for turning workflow data into schema decisions
- `generating-task-packets` — for turning workflow gaps into implementation tasks

## Related decisions

- D-008 (RolliTime contract — workflow mapping revealed gaps Qwen audit confirmed)
- D-010 (canonical ecosystem model)
- D-011 (living masters with promotion workflows)
