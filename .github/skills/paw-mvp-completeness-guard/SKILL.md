---
name: paw-mvp-completeness-guard
description: "Use after PAW MVP scope and simplicity guards pass to ensure the minimal plan still delivers a usable, safe, testable end-to-end product slice."
---

# PAW MVP Completeness Guard

> **Execution Context**: Run this skill in a read-only subagent after
> `paw-mvp-scope-guard` and `paw-mvp-simplicity-guard` both return `PASS`.
> Return a structured verdict to the PAW-MVP agent. Do not modify planning
> artifacts, source files, or git state.

Verify that simplification has not produced a demo-only or unsafe plan. This
gate decides **whether the minimal MVP is complete enough to ship and validate**.
It does not add optional product scope or speculative hardening.

## Required Inputs

Read these artifacts from `.paw/work/<work-id>/`:

- `WorkflowContext.md`
- `ImplementationPlan.md`
- `CodeResearch.md`
- `Spec.md` when present
- The latest passing scope and simplicity results supplied by the orchestrator

If the plan, code research, or either passing result is unavailable, return
`BLOCKED`.

Also read `docs/mvp-deferred/README.md` and the active `<work-id>.md` when
present. These live outside the PAW artifact directory. The orchestrator
supplies both paths; `paw-mvp-defer` owns their writes and commits.

## Minimum Complete Slice

The plan must cover:

- Entry point through observable successful outcome for the primary user
- Required state, data, and integration changes along that path
- Input validation and meaningful failure behavior at MVP boundaries
- Required authorization, privacy, and security controls
- Compatibility and migration handling when existing users or data are affected
- Tests that prove the core success path and material failure paths
- A usable invocation or interaction path without manual internal intervention
- Only the documentation needed to operate or evaluate the MVP

The deferred-work decision record is required project documentation, even when
optional user-facing documentation or planning-docs review is disabled.

Minimal scope never permits broken error handling, insecure behavior, omitted
validation, or tests that cannot prove the core outcome.

## Review

Build a core-journey checklist from the accepted scope and trace each step to a
plan phase and success criterion. Look for:

- Missing links between user entry and final outcome
- Happy-path-only planning where foreseeable failures would invalidate use
- Required data or integration work hidden outside the implementation phases
- Security or authorization deferred as post-MVP work
- Migration or backward compatibility obligations omitted from the narrow plan
- Tests that verify components but not the end-to-end MVP outcome
- Manual setup or intervention that makes the claimed user journey unusable
- Success criteria that cannot objectively show the MVP works

If prior guard results propose deferrals, verify they are captured in the
decision record with accurate status, rationale, MVP alternative, evidence, and
revisit trigger. Verify the index links to the correct work record. Return
`REVISE` for missing or misleading entries or broken index links. Do not let
required safety or core-outcome work become `DEFERRED`. Git persistence is
verified through `paw-mvp-defer`, not by modifying files in this guard.

Do not require secondary journeys, generalized resilience, scale beyond stated
needs, optional observability, or polish.

## Verdict

- `PASS`: The narrow plan delivers a usable, safe, testable end-to-end MVP.
- `REVISE`: Required work for the accepted MVP is missing or unverifiable.
- `ESCALATE`: Minimum viability depends on a disputed quality, migration,
  security, or usability decision.
- `BLOCKED`: Required artifacts or prior passing results are unavailable.

## Completion Response

Return exactly:

```markdown
## MVP Completeness Result: [PASS|REVISE|ESCALATE|BLOCKED]

### Core Journey
- Entry point: [where the primary user starts]
- Success outcome: [observable completion]
- Planned path: [phases/components covering the journey]

### Required Quality Coverage
- Validation and failures: [coverage]
- Security and authorization: [coverage]
- Compatibility and migration: [coverage or "Not applicable"]
- Verification: [tests and success criteria]
- Deferred documentation: [decision-record coverage, or "No deferrals"]

### Findings
- [BLOCKING|ADVISORY] [ImplementationPlan.md section] — [finding and evidence]

### Missing MVP Work
- [Required work or verification to add, or "None"]

### Recommendation
- Next activity: [paw-plan-review|paw-planning|human decision|required artifact recovery]
- [One-sentence reason]
```

Use `BLOCKING` only for findings that determine a non-`PASS` verdict. Omit no
section. Write `- None` when a section has no items.
