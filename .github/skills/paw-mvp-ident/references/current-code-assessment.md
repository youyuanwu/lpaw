# Shared Current-Code Assessment Contract

Use only when the caller explicitly selects `assessment_mode: current-code`.
This contract is shared by the scope, simplification, and completeness skills;
it does not replace their default planning contracts.

## In-Memory Context

The caller supplies:

- Proven repository checkout, branch, HEAD or `UNBORN`, and working-tree snapshot
- Included source areas and explicit exclusions
- Primary actor, core outcome, acceptance criteria, and preservation constraints
- Requirement basis: `USER_CONFIRMED`, `DOCUMENTED`, `INFERRED`, or `UNKNOWN`
- Evidence map of entry points, consumers, components, dependencies, and tests
- Recorded verification results with revision and scope, or `NOT_RUN`
- Earlier assessment results when available, including partial or blocked results

No PAW workflow or planning artifact is required. Do not create fake
WorkflowContext.md, CodeResearch.md, Spec.md, or ImplementationPlan.md to satisfy
planning-mode prerequisites. Inspect source to validate the supplied evidence.

If the checkout or source cannot be accessed, return `BLOCKED`. If the core
outcome or consumers are uncertain, assess the evidenced portions, label
assumptions, and return `PARTIAL` when the uncertainty materially limits the
assessment. Missing a previous result or a previous non-passing status does not
prevent independent assessment.

## Boundaries

- Analyze actual current files, not only HEAD or a diff.
- Preserve source, planning artifacts, deferred records, and git state. Do not
  write reports, install dependencies, or run mutating commands.
- Inspect tests and recorded outcomes; do not execute tests or builds. Unrun
  checks are `NOT_RUN`; results predating current edits are historical evidence,
  not a current `PASS`.
- Preserve supported behavior and public contracts. An MVP label never
  authorizes feature removal. All scope reductions and deferrals are proposals.
- Include file:line evidence, uncertainty, and verification prerequisites.
- Do not invoke another assessment skill or start planning/implementation.
  Return results to the caller, which owns composition and report writing.

## Status

- `COMPLETE`: The declared scope and relevant baseline were assessed with
  grounded findings, or no justified findings exist.
- `PARTIAL`: Material requirements, consumers, evidence, or coverage remain
  unresolved; include useful grounded findings and identify the limitation.
- `BLOCKED`: Essential checkout or source cannot be accessed.

Findings do not make an assessment fail. `COMPLETE` is not implementation
approval or a claim that tests passed. Unrun tests alone do not block an
assessment; report verification prerequisites separately.

## Result Shape

Each skill returns the same shape with its own category: scope uses `SCOPE`,
`paw-simplify` uses `SIMPLICITY`, and completeness uses `COMPLETENESS`.

Use this structure with no placeholders. Use `None` for empty sections; do not
emit sample rows. IDs are stable within each skill's result.

```markdown
## Current-Code Assessment: SCOPE | SIMPLICITY | COMPLETENESS

Status: COMPLETE | PARTIAL | BLOCKED

### Coverage
- Reviewed: <areas actually inspected>
- Unreviewed: <areas and reasons, or None>
- Baseline interpretation: <confirmed outcome or labeled assumptions>

### Alignment
| Component or capability | Classification | Rationale and evidence |
| ----------------------- | -------------- | ---------------------- |
| <component> | KEEP / SIMPLIFY / PROPOSE_DEFER / NEEDS_CONTEXT | <reason and citation> |

### Findings
#### <category>-001: <finding title>
- Priority: MUST | SHOULD | COULD
- Evidence and consumers: <file:line references and consumer findings>
- Current behavior: <observed behavior>
- Recommendation: <bounded proposal, not approval>
- Preserved behavior: <contracts that must remain>
- Reduction: <measure, baseline, target, and method; or Not applicable>
- Risk and blast radius: <affected components and uncertainty>
- Verification: <baseline/characterization and acceptance checks>
- Rollback: <safe reversal or migration approach>
- Decision needed: <selection or explicit feature-removal authorization>

### Baseline Verification
- Status: NOT_RUN | PASS | KNOWN_FAILURES | INSUFFICIENT
- Evidence: <recorded results, revision and scope, or None>
- Required checks: <verification prerequisites, or None>

### Deferred Candidates
| Item | Why deferred | MVP alternative | Revisit when | Evidence |
| ---- | ------------ | --------------- | ------------ | -------- |
| <proposed item> | <reason> | <alternative or None> | <trigger> | <source and rationale> |

### Open Questions
<requirements, consumer, compatibility, or coverage questions, or None>
```
