---
name: paw-mvp-review-guard
description: "Use before acting on implementation-review findings, final-review findings, or PR comments in PAW-MVP to triage required fixes, simplifications, proposed deferrals, and changes needing a decision or replanning."
---

# PAW MVP Review Feedback Guard

> **Execution Context**: Run in a read-only subagent before review-driven
> changes. Return a disposition for every feedback item to PAW-MVP. Do not edit
> files, run tests, commit, reply to comments, or resolve threads.

Assess the underlying problem separately from the reviewer's suggested
solution. A required bug fix does not automatically justify a new framework.
This is feedback triage, not a replacement for implementation verification or
the three planning gates.

## Inputs

Receive from the orchestrator:

- Proven execution checkout, Work ID, current phase or originating review/PR.
- Core outcome, accepted requirements and preservation constraints from
  `WorkflowContext.md`, `ImplementationPlan.md`, `Spec.md` when present, and
  `CodeResearch.md`; latest planning verdicts and explicit user decisions.
- Current implementation and relevant tests/docs, revision and dirty-state
  snapshot, plan version, and available verification results.
- All feedback in the batch: automated findings (including advisory notes),
  or unresolved PR review threads and actionable conversation comments, with
  stable IDs/URLs, full discussion, original severity/blocking status, and
  follow-up changes. Preserve IDs when grouping duplicates.
- Existing dispositions and the active deferred-work record when present.

Missing required context, incomplete comment retrieval, or an unproven checkout
returns `BLOCKED`; never substitute an empty successful batch. Missing tests or
failed checks are evidence to assess, not proof that the feedback is optional.
Do not invent current passing verification from historical results.

## Shared Criteria

Read and reuse these criteria without executing their planning/current-code
workflows or fabricating passing gate results:

- [Scope boundaries and core-outcome criteria](../paw-mvp-scope-guard/SKILL.md#boundaries).
- [Simplicity principles](../paw-mvp-simplicity-guard/SKILL.md#simplicity-principles).
- [Minimum complete slice](../paw-mvp-completeness-guard/SKILL.md#minimum-complete-slice).
- [Proportionate and effective testing](../paw-mvp-completeness-guard/references/test-value-contract.md).

This skill owns feedback dispositions; those skills retain ownership of their
existing assessment modes and verdicts.

## Triage

For every item, establish whether the reported problem exists in current code,
what requirement or quality obligation it affects, and the smallest sufficient
response. Cite plan/requirement and code/test evidence. Reviewer severity,
consensus, a small diff, or an "easy win" label is not scope approval.

| Disposition | Use when | Response |
| ----------- | -------- | -------- |
| `FIX` | An evidenced correctness, security, compatibility, migration, acceptance, or required verification/documentation gap can be corrected within the accepted design. | Specify the smallest complete fix and checks proving it. |
| `SIMPLIFY` | A bounded change demonstrably reduces complexity while preserving accepted behavior and contracts. | Identify the reduction, affected consumers, preservation evidence, and verification. |
| `PROPOSE_DEFER` | The request adds optional behavior, abstractions, configurability, speculative hardening, or polish not needed by the MVP. | Explain why it is optional, the current MVP alternative, and a concrete revisit trigger. No implementation authorization. |
| `REPLAN` | An already authorized change requires material plan/design revision. | State the approved decision and affected plan sections; rerun planning and all gates before implementation. |
| `ESCALATE` | A material scope/architecture/operational trade-off lacks user approval, requirements conflict, or a proposed deferral is disputed. | Ask for a specific decision with evidence and a recommended smaller alternative. |
| `NO_CHANGE` | The item is already addressed, duplicated, or demonstrably inapplicable. | Cite the fix and verification, canonical item ID, or disproving evidence; never silently drop it. |
| `BLOCKED` | Evidence needed to assess an item cannot be obtained. | Identify what must be recovered; do not guess a disposition. |

- `FIX` and `SIMPLIFY` must fit accepted scope and design. New public contracts,
  dependencies/services, cross-cutting redesigns, or additional product journeys
  are material unless already explicitly covered by the approved plan. Route
  material changes to `ESCALATE` until approved, then `REPLAN`.
- Never defer a required quality obligation just because the suggested solution
  is oversized. Split mixed feedback into linked subitems: retain the required
  fix and separately assess the optional extension. If even the smallest safe
  fix needs a material change, escalate/replan; do not implement a partial fix.
- Missing references alone do not justify removing code, tests, or supported
  features. Consumer uncertainty requires evidence recovery or a user decision,
  not a `SIMPLIFY` authorization.
- Explicit user approval may expand scope, but does not waive correctness,
  security, preservation obligations, or the planning gates.
- Do not use `NO_CHANGE` or `PROPOSE_DEFER` to overrule an unresolved reviewer
  blocker. Preserve the original verdict and identify the needed reassessment
  or agreement. Conflicting recommendations remain visible.

For test-related feedback, use the shared test-value contract in the existing
Evidence and rationale and Required verification fields. Name the missing risk
coverage and defect-sensitive assertion before authorizing `FIX`; use
`SIMPLIFY` for evidenced redundant machinery only with preserved protection.
Demonstrated existing coverage can justify `NO_CHANGE`, while genuinely
optional testing may be `PROPOSE_DEFER`. A passing suite or coverage percentage
alone proves neither sufficiency nor redundancy. Vacuous coverage of required
behavior needs correction, not deferral. Unknown sufficiency requires evidence
recovery, not invented test additions. Inspect recorded evidence only; this
guard does not run tests or fault experiments.

## Batch Verdict

Return `BLOCKED` if any item or required input is blocked; otherwise `ESCALATE`
if a decision is needed; otherwise `REPLAN` if a plan revision is needed;
otherwise `READY`. Preserve all rows and proposed deferrals on every verdict.

`READY` means triage is complete, not that the code passed review or every
comment is resolved. Only `FIX` and `SIMPLIFY` rows authorize bounded rework.
Optional proposals still require documentation and an explicit disposition
message. A verified empty batch may return `READY` with no items.

## Completion Response

```markdown
## MVP Review Feedback Result: [READY|REPLAN|ESCALATE|BLOCKED]

### Context
- Work/phase/review: [identity]
- Assessed state: [revision, dirty state, plan version, feedback batch]
- Core outcome: [accepted observable outcome]

### Dispositions
| Feedback IDs | Original severity/blocker | Disposition | Evidence and rationale | Smallest response | Required verification |
| ------------ | ------------------------- | ----------- | ---------------------- | ----------------- | --------------------- |
| [IDs/URLs] | [original status] | [disposition] | [requirement and code/test references] | [bounded response or decision] | [checks or None with reason] |

### Coverage
- Received IDs: [all source IDs, or None]
- Accounted-for IDs: [all source IDs, including grouped/split mappings, or None]
- Unresolved reviewer blockers: [IDs and required reassessment/agreement, or None]

### Deferred Candidates
| Feedback IDs | Item | Why optional | MVP alternative | Revisit when | Evidence |
| ------------ | ---- | ------------ | --------------- | ------------ | -------- |
| [IDs] | [candidate] | [reason] | [smaller solution] | [trigger] | [durable rationale/references] |

### Decisions and Blockers
- [Specific question, approved decision requiring replanning, recovery need, or None]

### Recommendation
- Next activity: [bounded rework|document/reply and continue review|paw-planning|human decision|evidence recovery]
- [One-sentence reason]
```

Omit no section. Use `- None` instead of an empty/placeholder table. Missing
feedback coverage is `BLOCKED`, never implicit approval.
