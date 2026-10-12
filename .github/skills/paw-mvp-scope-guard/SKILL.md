---
name: paw-mvp-scope-guard
description: "Use first after PAW MVP planning to define the core outcome, classify work as KEEP, DEFER, or REMOVE, and report deferred candidates for committed documentation."
---

# PAW MVP Scope Guard

> **Execution Context**: Run this skill in a read-only subagent immediately
> after `paw-planning`. Return a structured verdict to the PAW-MVP agent. Do not
> modify planning artifacts, source files, or git state.

Determine whether every planned capability is necessary for the MVP's single
core user outcome. This gate decides **what belongs in the MVP**. It does not
judge architecture or verify implementation readiness.

## Required Inputs

Read these artifacts from `.paw/work/<work-id>/`:

- `WorkflowContext.md`
- `ImplementationPlan.md`
- `Spec.md` when present
- `CodeResearch.md`
- `WorkShaping.md` when present

If `ImplementationPlan.md` or `CodeResearch.md` is missing, return `BLOCKED`.
Do not invent missing requirements or evidence.

## Boundaries

Do:

- Identify the primary user or actor.
- State one observable end-to-end outcome the MVP must enable.
- Trace each material plan item to that outcome, an explicit acceptance
  criterion, or a non-negotiable quality constraint.
- Separate current MVP scope from valuable post-MVP work.

Do not:

- Evaluate implementation layers, dependencies, or phase design; the simplicity
  guard owns those concerns.
- Remove explicit requirements silently.
- Treat correctness, security, compatibility, migration safety, or validation
  as optional product scope.
- Expand the plan to make the product more complete for hypothetical users.

## Scope Classification

Classify each material plan item:

- `KEEP`: Required for the core outcome, an explicit acceptance criterion, or a
  non-negotiable quality constraint.
- `DEFER`: Valuable after validating the MVP, but unnecessary for the core
  outcome now.
- `REMOVE`: Unsupported by current requirements or verified constraints.

Look specifically for:

- Secondary personas or user journeys
- Optional workflows, modes, integrations, and configuration variants
- Administrative tooling and analytics
- Product polish beyond basic usability of the core journey
- Speculative scale or future requirements
- Phase candidates promoted without an explicit MVP need

Every `KEEP` item needs evidence. Any `DEFER` or `REMOVE` item still scheduled
for implementation requires `REVISE`, unless the user explicitly expanded MVP
scope.

## Deferred Candidate Reporting

For each actual `DEFER` candidate, report its rationale, chosen MVP alternative
when known, revisit trigger, and supporting evidence. Include candidates even
on non-`PASS` verdicts so replanning cannot lose the decision context.

Recommendations are proposals, not user-approved deferrals. Do not classify
unsupported `REMOVE` items as future commitments. The orchestrator owns writing
and committing records through `paw-mvp-defer`; this guard remains read-only.

## Verdict

- `PASS`: Every planned capability is required for the defined MVP.
- `REVISE`: Nonessential capabilities must be deferred or removed.
- `ESCALATE`: The core outcome is unclear, or explicit requirements conflict
  with the requested MVP boundary.
- `BLOCKED`: Required artifacts or evidence are unavailable or inconsistent.

## Completion Response

Return exactly:

```markdown
## MVP Scope Result: [PASS|REVISE|ESCALATE|BLOCKED]

### Core Outcome
- Primary user: [user or actor]
- End-to-end outcome: [observable result]

### Scope Classification
- KEEP: [required plan items]
- DEFER: [post-MVP items, or "None"]
- REMOVE: [unsupported items, or "None"]

### Findings
- [BLOCKING|ADVISORY] [ImplementationPlan.md section] — [finding and evidence]

### Required Scope Changes
- [Capability to defer, remove, or clarify, or "None"]

### Deferred Candidates
| Item | Why deferred | MVP alternative | Revisit when | Evidence |
| ---- | ------------ | --------------- | ------------ | -------- |
| [candidate] | [reason] | [smaller solution or "None"] | [trigger] | [source and rationale] |

### Recommendation
- Next activity: [paw-mvp-simplicity-guard|paw-planning|human decision|required artifact recovery]
- [One-sentence reason]
```

Use `BLOCKING` only for findings that determine a non-`PASS` verdict. Omit no
section. Write `- None` when a section has no items, including Deferred
Candidates; do not emit a placeholder table row.
