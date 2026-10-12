---
name: paw-mvp-simplicity-guard
description: "Use after the PAW MVP scope guard passes to minimize architecture, dependencies, and phases, and report approaches deferred through simplification."
---

# PAW MVP Simplicity Guard

> **Execution Context**: Run this skill in a read-only subagent after
> `paw-mvp-scope-guard` returns `PASS`. Return a structured verdict to the
> PAW-MVP agent. Do not modify planning artifacts, source files, or git state.

Determine whether the accepted MVP scope uses the least architecture and
operational machinery needed to deliver it. This gate decides **how simply the
MVP can be built**. It must not remove accepted scope.

## Required Inputs

Read these artifacts from `.paw/work/<work-id>/`:

- `WorkflowContext.md`
- `ImplementationPlan.md`
- `CodeResearch.md`
- `Spec.md` when present
- The latest `paw-mvp-scope-guard` result supplied by the orchestrator

If the plan, code research, or passing scope result is unavailable, return
`BLOCKED`.

## Simplicity Principles

- Reuse existing components, conventions, dependencies, and deployment paths.
- Prefer a direct vertical slice over generalized infrastructure.
- Prefer one independently testable implementation phase.
- Add phases only for unavoidable dependency ordering, safe migration, or a
  distinct risk boundary.
- Retain complexity required by correctness, security, compatibility, migration
  safety, or explicit acceptance criteria.
- Compare alternatives by total implementation, verification, operational, and
  maintenance burden.

Read the [shared test-value contract](../paw-mvp-completeness-guard/references/test-value-contract.md)
when assessing verification design. Require a lightweight rationale per
proposed or materially changed test group in the plan's existing verification
sections. Identify unnecessary harnesses, speculative matrices, or duplicate
protection while preserving meaningful boundary and failure coverage.
Material avoidable testing complexity requires `REVISE`, not a blanket demand
for fewer tests. Report the evidence and smaller alternative in the existing
Findings and Required Simplifications sections; do not execute checks here.

## Review

Inspect the plan for:

- **Premature abstractions**: new frameworks, layers, interfaces, factories, or
  extension points without a current MVP need
- **Missed reuse**: new components duplicating verified repository capabilities
- **Dependency growth**: packages, services, processes, or infrastructure that
  existing tools can avoid
- **Architectural spread**: changes to unrelated subsystems or public contracts
- **Excessive phases**: separations that do not provide independent validation,
  migration safety, or risk isolation
- **Hidden coupling**: phases described as independent but requiring coordinated
  changes to function or test
- **Operational burden**: unnecessary configuration, deployment, observability,
  rollback, or maintenance work
- **Verification burden**: a design whose test matrix is larger than an
  equivalent simpler design

For each finding:

1. Cite the plan section.
2. Cite the code research or requirement establishing the simpler option.
3. Describe the smaller architectural outcome, not implementation steps.
4. Confirm that it preserves all `KEEP` scope and required quality.

Complexity indicators are prompts for evidence, not automatic failures.

## Deferred Candidate Reporting

When simplifying leaves a future-valued capability or approach out of the MVP,
report what was set aside, why the smaller solution suffices now, that solution,
a concrete revisit trigger, and evidence. Include candidates on non-`PASS`
verdicts as proposals; the orchestrator records accepted decisions in
the indexed deferred-work folder through `paw-mvp-defer`.

Do not invent future work, defer accepted `KEEP` behavior, or turn every rejected
implementation into a backlog item. Document only actual candidates. This
skill must not write or commit the record.

## Verdict

- `PASS`: The plan uses proportionate architecture, dependencies, and phases.
- `REVISE`: The same MVP can be delivered with materially less machinery.
- `ESCALATE`: Simplification requires a material architecture, compatibility, or
  operational trade-off that needs user authority.
- `BLOCKED`: Required evidence or the passing scope result is unavailable.

## Completion Response

Return exactly:

```markdown
## MVP Simplicity Result: [PASS|REVISE|ESCALATE|BLOCKED]

### Architecture Profile
- Reused capabilities: [existing components and paths]
- New capabilities: [new components and why each is necessary]
- Planned phases: [count and justification]

### Findings
- [BLOCKING|ADVISORY] [ImplementationPlan.md section] — [finding and evidence]

### Accepted Complexity
- [Complexity retained because it is required, or "None"]

### Required Simplifications
- [Smaller architectural outcome, or "None"]

### Deferred Candidates
| Item | Why deferred | MVP alternative | Revisit when | Evidence |
| ---- | ------------ | --------------- | ------------ | -------- |
| [capability or approach] | [reason] | [smaller solution] | [trigger] | [source and rationale] |

### Recommendation
- Next activity: [paw-mvp-completeness-guard|paw-planning|human decision|required artifact recovery]
- [One-sentence reason]
```

Use `BLOCKING` only for findings that determine a non-`PASS` verdict. Omit no
section. Write `- None` when a section has no items, including Deferred
Candidates; do not emit a placeholder table row.
