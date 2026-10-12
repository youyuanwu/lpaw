---
name: paw-simplify
description: "Use after PAW code research and before planning to assess an existing repository for behavior-preserving simplification, unnecessary layers, duplication, and removable dependencies."
---

# PAW Repository Simplification Assessment

> **Execution Context**: Run in a read-only subagent after `paw-code-research`
> and before `paw-planning`. Return findings to the PAW or PAW-MVP orchestrator.
> Do not edit files, run mutating commands, install dependencies, or change git
> state. The orchestrator owns baseline checks, approval, and implementation.

Identify bounded, evidence-backed opportunities to reduce existing complexity.
Default to preserving behavior, public contracts, and supported user journeys.
Assess one requested subsystem at a time; do not turn a localized task into a
repository-wide rewrite.

## Required Inputs

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

1. Use code research to map entry points, consumers, dependencies, and tests.
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
7. Specify tests or characterization coverage proving preserved behavior,
   material failure cases, compatibility, and rollback. Missing coverage is
   prerequisite work, not permission to remove untested behavior.

Report existing baseline failures separately from potential regressions.
If baseline results are missing, failed, or do not cover the affected contracts,
return `NEEDS_DECISION` with the research or verification needed before changes.
The orchestrator establishes baseline coverage before implementation.

## Boundaries and Handoff

- Return recommendations, not approval to change code.
- Do not implement, rewrite the plan, create assessment artifacts, commit, or
  orchestrate the remaining PAW stages.
- Include uncertainty and file:line evidence for every candidate.
- Return no candidates when evidence does not justify a reduction; do not
  manufacture cleanup to satisfy the request.
- Separate useful postponed candidates from rejected or unsupported ideas.
  Return deferred candidates in the format consumed by `paw-mvp-defer`.
- After user selection, the orchestrator sends only approved candidates,
  preserved contracts, measurement targets, and verification prerequisites to
  `paw-planning`. Implement and review in bounded phases, then compare actual
  reductions against the baseline while checking preserved behavior.

## Result

- `READY`: Assessment is grounded and ready for candidate selection, or no
  justified candidates exist. This is not permission to implement.
- `NEEDS_DECISION`: Goals, consumer uncertainty, feature removal, compatibility,
  or baseline coverage require user input or additional evidence.
- `BLOCKED`: Required context, artifacts, or the execution checkout are missing.

## Completion Response

Return these sections, using `None` for empty lists and no placeholder rows:

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
