---
name: paw-mvp-scope-guard
description: "Use after PAW MVP planning to guard scope, or in current-code assessment mode to identify MVP alignment and proposed deferrals without requiring a plan."
---

# PAW MVP Scope Guard

> **Execution Context**: Run this skill in a read-only assessment session. In
> planning mode, run immediately after `paw-planning`. Return results to the
> caller. Do not modify planning artifacts, source files, or git state.

Determine whether each capability is necessary for the MVP's single core user
outcome. This skill decides **what belongs in the MVP**. It does not judge
architecture or verify implementation readiness.

## Assessment Mode

- `assessment_mode: plan` (default): Enforce the existing planning gate and its
  artifact prerequisites, verdicts, and Completion Response.
- `assessment_mode: current-code` (explicit only): Read the
  [shared current-code contract](../paw-mvp-ident/references/current-code-assessment.md).
  Accept its in-memory context instead of planning artifacts and use its result
  shape with category `SCOPE`.

Unknown modes return `BLOCKED` with the invalid value. Missing artifacts do not
implicitly select current-code mode. Sections labeled Planning apply only to
`assessment_mode: plan`.

## Planning Inputs

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
- Trace each material capability to that outcome, an explicit acceptance
  criterion, or a non-negotiable quality constraint.
- Separate current MVP scope from valuable post-MVP work.

Do not:

- Evaluate implementation layers, dependencies, or phase design; the simplicity
  guard owns plan design and `paw-simplify` owns current-code simplification.
- Remove explicit requirements silently.
- Treat correctness, security, compatibility, migration safety, or validation
  as optional product scope.
- Expand the plan to make the product more complete for hypothetical users.

## Scope Classification

In planning mode, classify each material plan item:

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

Every `KEEP` item needs evidence. In planning mode, any `DEFER` or `REMOVE` item
still scheduled for implementation requires `REVISE`, unless the user explicitly
expanded MVP scope.

In current-code mode, apply the same core-outcome and quality criteria to
existing capabilities, using `KEEP`, `PROPOSE_DEFER`, or `NEEDS_CONTEXT`.
Outside-MVP supported features are proposed scope decisions, never approved
removals. Identify user journeys and consumers that would be affected; do not
infer that a feature is unused from its MVP classification. Unknown requirements
produce open questions rather than invented approved scope.

## Deferred Candidate Reporting

For each actual deferral candidate, report its rationale, chosen MVP alternative
when known, revisit trigger, and supporting evidence. Include candidates even
on non-`PASS` verdicts so replanning cannot lose the decision context.

Recommendations are proposals, not user-approved deferrals. Do not classify
unsupported `REMOVE` items as future commitments. The orchestrator owns writing
and committing records through `paw-mvp-defer` during an implementation workflow;
this guard remains read-only. In current-code mode, return proposals only and
do not require committed deferral records.

## Planning Verdict

- `PASS`: Every planned capability is required for the defined MVP.
- `REVISE`: Nonessential capabilities must be deferred or removed.
- `ESCALATE`: The core outcome is unclear, or explicit requirements conflict
  with the requested MVP boundary.
- `BLOCKED`: Required artifacts or evidence are unavailable or inconsistent.

## Planning Completion Response

In planning mode, return exactly:

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
