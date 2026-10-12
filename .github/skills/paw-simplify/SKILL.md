---
name: paw-simplify
description: "Use before PAW planning or in standalone current-code assessment mode to identify behavior-preserving simplification, unnecessary layers, duplication, and removable dependencies."
---

# PAW Repository Simplification Assessment

> **Execution Context**: Run in a read-only assessment session. In planning
> mode, run after `paw-code-research` and before `paw-planning`. Return findings
> to the caller. Do not edit files, run mutating commands, install dependencies,
> or change git state. The orchestrator owns checks, approval, and implementation.

Identify bounded, evidence-backed opportunities to reduce existing complexity.
Default to preserving behavior, public contracts, and supported user journeys.
Honor the caller's included and excluded source areas. Assess one requested
subsystem at a time; do not turn a localized task into a repository-wide rewrite.

## Assessment Mode

- `assessment_mode: plan` (default): Keep the existing PAW workflow inputs,
  readiness checks, Result, and Completion Response.
- `assessment_mode: current-code` (explicit only): Read the
  [shared current-code contract](../paw-mvp-ident/references/current-code-assessment.md).
  Accept its in-memory context instead of required workflow artifacts and use
  its result shape with category `SIMPLICITY`.

Unknown modes return `BLOCKED` with the invalid value. Missing artifacts do not
implicitly select current-code mode. Sections labeled Planning apply only to
`assessment_mode: plan`.

## Planning Inputs

- Active `WorkflowContext.md` and `CodeResearch.md`
- User's simplification goal and assessment boundaries
- `Spec.md` or equivalent user-approved requirements when available
- Existing `ImplementationPlan.md` when revising a plan
- Baseline verification results from the orchestrator: revision, commands,
  outcomes, and known failures

Read artifacts and code in the proven execution checkout specified by the
workflow context, not a different caller checkout. Return `BLOCKED` if required
artifacts or the checkout cannot be established. Label missing baseline results
as `MISSING`; never report unrun checks as passing.

## Behavior-Preservation Contract

Identify what must remain unchanged:

- Supported user journeys and externally observable success and failure behavior
- Public APIs, CLI options, configuration, formats, and integration contracts
- Persisted data, migrations, and backward compatibility
- Authorization, privacy, security controls, and required operational guarantees

Distinguish internal simplification from feature removal. Removing a supported
feature, integration, or contract requires explicit user approval and a
deprecation or migration decision; an MVP label is not authorization to delete
existing capabilities.

## Assessment

1. Use code research in planning mode, or the supplied evidence map in
   current-code mode, to trace entry points, consumers, dependencies, and tests.
   Inspect relevant code to substantiate findings; do not ask
   `paw-code-research` to critique or propose refactors.
2. Identify concrete opportunities to:
   - Remove code or dependencies whose lack of supported consumers is evidenced
   - Consolidate duplicate paths or competing mechanisms
   - Flatten layers that add no required behavior or boundary
   - Replace unnecessary machinery with an existing repository capability
3. Check dynamic loading, reflection, registration, configuration, generated
   code, scripts, and external consumers before calling a component unused.
   No references found is not proof of dead code. Keep uncertain candidates
   out of the implementation-ready list and report the evidence still needed.
4. Compare total implementation, verification, operational, and maintenance
   burden. Reject a simplification that merely moves complexity or introduces
   a replacement framework.
5. Rank candidates by evidenced benefit, compatibility risk, and blast radius.
   Prefer a small independently verifiable change over coordinated rewrites.
6. For each candidate, define the relevant complexity measure, current baseline,
   target reduction, and how both will be measured. Examples include dependency
   count, indirection depth, duplicate paths, or configuration variants. Fewer
   lines alone does not prove simplification; do not fabricate measurements.
   Label unmeasured baselines and proposed targets explicitly.
7. Specify tests or characterization coverage proving preserved behavior,
   material failure cases, compatibility, and rollback. Missing coverage is
   prerequisite work, not permission to remove untested behavior.

Report existing baseline failures separately from potential regressions. In
planning mode, missing, failed, or insufficient baseline results require
`NEEDS_DECISION` with the verification needed before changes. In current-code
mode, inspect tests and recorded results without running them, report unrun
checks as `NOT_RUN`, and return assessment completeness separately from
implementation readiness. Do not claim verification for a different revision
or a working tree with subsequent source edits.

## Boundaries and Handoff

- Return recommendations, not approval to change code.
- Do not implement, rewrite the plan, create assessment artifacts, commit, or
  orchestrate the remaining PAW stages.
- Include uncertainty and file:line evidence for every candidate.
- Return no candidates when evidence does not justify a reduction; do not
  manufacture cleanup to satisfy the request.
- Separate useful postponed candidates from rejected or unsupported ideas.
  Return deferred candidates in the format consumed by `paw-mvp-defer`.
- In current-code mode, add `SIMPLIFY` alignment rows only for evidenced
  candidates and `NEEDS_CONTEXT` for unresolved consumer questions. Treat a
  prior scope assessment as context, not authority to delete supported behavior.
- After user selection, the orchestrator sends only approved candidates,
  preserved contracts, measurement targets, and verification prerequisites to
  `paw-planning`. Implement and review in bounded phases, then compare actual
  reductions against the baseline while checking preserved behavior.

## Planning Result

- `READY`: Assessment is grounded and ready for candidate selection, or no
  justified candidates exist. This is not permission to implement.
- `NEEDS_DECISION`: Goals, consumer uncertainty, feature removal, compatibility,
  or baseline coverage require user input or additional evidence.
- `BLOCKED`: Required context, artifacts, or the execution checkout are missing.

## Planning Completion Response

In planning mode, return these sections, using `None` for empty lists and no
placeholder rows:

```markdown
## Repository Simplification Result: [READY|NEEDS_DECISION|BLOCKED]

### Assessment Scope
- Subsystem: [bounded target]
- Revision: [assessed revision]
- Goal: [observable complexity reduction]

### Preserved Contracts
- [behavior or contract and supporting evidence]

### Baseline Verification
- Status: [PASS|KNOWN_FAILURES|MISSING|INSUFFICIENT]
- Commands and revision: [recorded checks, or "Not available"]
- Coverage and failures: [known coverage, gaps, or failures]

### Ranked Candidates
#### [stable ID]: [REMOVE|CONSOLIDATE|FLATTEN|REPLACE] [candidate]
- Evidence and consumers: [file:line references and consumer findings]
- Preserved behavior: [contracts that remain unchanged]
- Reduction: [measure, baseline, target, and measurement method]
- Risk and blast radius: [affected components and uncertainty]
- Verification: [existing tests and prerequisite characterization coverage]
- Rollback: [reversible phase or migration approach]
- Decision needed: [selection or explicit authorization required]

### Deferred Candidates
| Item | Why deferred | MVP alternative | Revisit when | Evidence |
| ---- | ------------ | --------------- | ------------ | -------- |
| [candidate] | [reason] | [smaller solution or "None"] | [trigger] | [source and rationale] |

### Open Questions
- [unresolved consumer, scope, or verification question, or "None"]

### Recommendation
- Next activity: [candidate selection|additional research|baseline verification|required artifact recovery|no changes]
- [bounded next action; implementation requires approval and PAW planning]
```
