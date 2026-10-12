---
name: paw-mvp-completeness-guard
description: "Use after PAW MVP planning guards to check completeness, or in current-code assessment mode to identify gaps in a usable, safe, testable core journey."
---

# PAW MVP Completeness Guard

> **Execution Context**: Run this skill in a read-only assessment session. In
> planning mode, require passing scope and simplicity guards first. Return
> results to the caller. Do not modify planning artifacts, source files, or git
> state.

Verify that reduced scope has not produced a demo-only or unsafe product. This
skill decides **whether the minimal MVP is complete enough to ship and
validate**. It does not add optional product scope or speculative hardening.

## Assessment Mode

- `assessment_mode: plan` (default): Require the planning artifacts and prior
  passing guards, and use the existing Verdict and Completion Response.
- `assessment_mode: current-code` (explicit only): Read the
  [shared current-code contract](../paw-mvp-ident/references/current-code-assessment.md).
  Accept its in-memory context and earlier assessments regardless of status;
  no plan, prior `PASS`, or committed deferral records are required. Use the
  shared result shape with category `COMPLETENESS`.

Unknown modes return `BLOCKED` with the invalid value. Missing artifacts do not
implicitly select current-code mode. Sections labeled Planning apply only to
`assessment_mode: plan`.

## Planning Inputs

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

The planned or implemented core journey must cover:

- Entry point through observable successful outcome for the primary user
- Required state, data, and integration changes along that path
- Input validation and meaningful failure behavior at MVP boundaries
- Required authorization, privacy, and security controls
- Compatibility and migration handling when existing users or data are affected
- Tests that prove the core success path and material failure paths
- A usable invocation or interaction path without manual internal intervention
- Only the documentation needed to operate or evaluate the MVP

In planning mode, the deferred-work decision record is required project
documentation even when optional user-facing documentation or planning-docs
review is disabled. Current-code identification reports proposals; it does not
require or create these committed records.

Minimal scope never permits broken error handling, insecure behavior, omitted
validation, or tests that cannot prove the core outcome.

## Test Effectiveness

Read and apply the [shared test-value contract](references/test-value-contract.md).
In planning mode, check that the verification design names relevant risks,
observable assertions, and plausible defects those assertions should detect.
Accept justified reuse of existing coverage; require `REVISE` when required
behavior has no meaningful planned check or relies on vacuous assertions.
Do not demand a test-file edit, count, or coverage increase as a proxy.

In current-code mode, inspect the actual path and assertions, including mocked
boundaries, and report ineffective coverage with evidence in Findings and
Baseline Verification. Preserve current-code statuses and `NOT_RUN` handling;
do not execute tests or controlled faults, or require them merely to complete
the assessment. Both modes use existing output sections, not a new checklist
artifact.

## Review

Build a core-journey checklist from the stated outcome. In planning mode, trace
each step to a phase and success criterion. In current-code mode, trace it to
the actual implementation, entry points, tests, and documented behavior; use
prior assessments as context, not prerequisites or approved scope changes.
Look for:

- Missing links between user entry and final outcome
- Happy-path-only planning where foreseeable failures would invalidate use
- Required data or integration work missing from the plan or implementation
- Security or authorization deferred as post-MVP work
- Migration or backward compatibility obligations omitted from the narrow plan
- Tests that verify components but not the end-to-end MVP outcome
- Manual setup or intervention that makes the claimed user journey unusable
- Missing or unverifiable acceptance evidence for the MVP outcome

In planning mode, if prior guard results propose deferrals, verify they are
captured in the decision record with accurate status, rationale, MVP alternative,
evidence, and revisit trigger. Verify the index links to the correct work record. Return
`REVISE` for missing or misleading entries or broken index links. Do not let
required safety or core-outcome work become `DEFERRED`. Git persistence is
verified through `paw-mvp-defer`, not by modifying files in this guard.

In current-code mode, do not run verification commands, require a phase or plan,
or fail merely because tests were not run. Report evidenced gaps and required
checks using the shared contract. Flag any proposed deferral of core-outcome or
safety work as a completeness finding; return no deferred candidates for those
obligations. Use `NEEDS_CONTEXT` alignment rows for unresolved core-journey
coverage and identify assumptions when the goal is unknown.

Do not require secondary journeys, generalized resilience, scale beyond stated
needs, optional observability, or polish.

## Planning Verdict

- `PASS`: The narrow plan delivers a usable, safe, testable end-to-end MVP.
- `REVISE`: Required work for the accepted MVP is missing or unverifiable.
- `ESCALATE`: Minimum viability depends on a disputed quality, migration,
  security, or usability decision.
- `BLOCKED`: Required artifacts or prior passing results are unavailable.

## Planning Completion Response

In planning mode, return exactly:

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
