---
name: paw-mvp-ident
description: "Use to compose shared current-code scope, simplification, and completeness assessments into .paw/reviews/mvp-ident-<branch>/REVIEW.md without a PAW workflow."
---

# PAW MVP Current-Code Review

> **Execution Context**: Run directly in the PAW-MVP-Ident session. Analyze code
> read-only and write only the selected `REVIEW.md` artifact. Do not implement
> recommendations, stage or commit files, push, switch branches, create PRs,
> install dependencies, or run commands that mutate the repository.

## Resolve the Review

1. Resolve the current git repository root and checkout. Review the user's
   requested subsystem or, when no scope is given, the repository's own source.
   If a different PAW execution checkout is requested, prove its identity from
   the supplied workflow context and git state; do not use the caller checkout
   as a fallback. If the checkout cannot be proven, report `BLOCKED`.
2. Capture branch, HEAD, timestamp, and working-tree state. For a repository
   without a first commit, record HEAD as `UNBORN` and still assess its files.
   Assess actual files, including staged, unstaged, and relevant untracked
   source, not only a diff or committed HEAD. Exclude PAW artifacts,
   ignored/generated content, vendored code, and submodule internals unless
   explicitly included by the user.
3. Establish the primary actor, core MVP outcome, acceptance criteria, and
   preservation constraints from user input or existing requirements and docs.
   Read existing PAW artifacts when relevant, but do not require
   WorkflowContext.md, CodeResearch.md, Spec.md, or ImplementationPlan.md.
   Label inferred goals as assumptions. If ambiguous goals materially affect
   classification, ask for clarification; when unavailable, retain the question
   and continue with independently assessable areas. Mark the final report
   `PARTIAL` rather than inventing approved scope.
4. Choose a review identifier `mvp-ident-<branch-slug>`: lowercase the branch,
   replace non-alphanumeric runs with hyphens, and trim hyphens. If the slug
   is empty, use `branch-<short-sha>` or `branch-unborn` instead. For detached
   HEAD, use `mvp-ident-detached-<short-sha>`. If that directory already exists,
   select the first available suffix `-2`, `-3`, and so on. Never overwrite an
   earlier PAW review.
5. Write to `.paw/reviews/<identifier>/REVIEW.md` inside the proven checkout.
   This matches PAW's standalone review directory convention but is owned by
   this skill, not the upstream PR-review output pipeline. Do not create other
   workflow artifacts or alter ignore rules. Verify the resolved output path
   stays inside the proven checkout, including through existing symlinks, and
   recheck that the selected directory is unused before writing.

## Compose Shared Assessments

Read the [shared current-code contract](references/current-code-assessment.md).
Build its in-memory context from repository inspection and user requirements.
Map entry points, components, consumers, dependencies, and tests with file:line
evidence; do not create workflow artifacts or invent missing verification.
Inspect tests and existing recorded outcomes only; do not run tests or builds
during context collection or synthesis.

Execute these skills in order with explicit `assessment_mode: current-code`:

1. `paw-mvp-scope-guard`: MVP alignment and proposed scope decisions (`SCOPE`).
2. `paw-simplify`: Behavior-preserving reduction candidates (`SIMPLICITY`).
3. `paw-mvp-completeness-guard`: Current core-journey gaps (`COMPLETENESS`).

Run each by loading the named skill first in a read-only subagent. Supply the
same checkout, snapshot, scope, baseline, evidence map, and shared result
contract to every call. Pass all earlier results to later calls, including
partial results and blockers. Do not run `paw-mvp-simplicity-guard` here: it is
a plan-design gate; `paw-simplify` owns current-code simplification.

Accumulate results instead of short-circuiting on `PARTIAL`, `BLOCKED`, or
findings. Only an inaccessible checkout prevents all assessments. If a skill
cannot load, fails, or returns an invalid result, report the failed assessment
as unavailable and continue with the others; never silently substitute local
assessment logic or claim it completed. If read-only subagent execution is
unavailable, load the shared skills sequentially in this session and retain
their distinct results. This fallback changes execution only, not their
contracts, ownership, or the explicit mode.

This skill owns context collection and synthesis, not assessment criteria.
Do not repeat the shared skills' scope, simplification, or completeness review
locally. The default planning pipeline still requires its prerequisites and
short-circuit verdict handling; current-code results are not planning PASSes.

## Synthesize the Review

- Retain each assessment's category, status, coverage, evidence, and questions.
- Deduplicate equivalent findings while preserving source skill/ID provenance.
  Do not merge different requirements or drop contradictory recommendations.
- Rank grounded findings and assign report IDs without turning recommendations
  into approved changes.
- Combine alignment rows without treating `PROPOSE_DEFER` as removal authority.
  A capability can be `KEEP` and `SIMPLIFY`; show both. For conflicts, preserve
  both assessments' evidence and mark the disputed classification
  `NEEDS_CONTEXT` rather than choosing silently.
- Aggregate verification conservatively. A PASS applies only to the recorded
  revision and covered scope; retain failures, gaps, and unrun checks.
- Merge actual deferred proposals, not rejected ideas or accepted commitments.
- Preserve partial coverage and missing assessments in report limitations.

## Write the Review

Use stable finding IDs within this report, ranked by priority, benefit, and
risk. Only include actionable findings with evidence; do not manufacture
findings or turn observations into speculative future work.

Use this structure, replacing every placeholder. For empty sections write
`None`; omit empty template tables and candidate examples.

```markdown
# PAW MVP Identification Review

Status: COMPLETE | PARTIAL | BLOCKED
Review ID: <identifier>
Repository: <repository name>
Branch: <branch or detached HEAD>
HEAD: <commit SHA or UNBORN>
Reviewed at: <UTC timestamp>
Code state: <clean or dirty; staged, unstaged, and untracked scope>

## Summary
<main opportunities, alignment assessment, and limitations>

## MVP Baseline
- Basis: USER_CONFIRMED | DOCUMENTED | INFERRED | UNKNOWN
- Primary actor: <actor or unknown>
- Core outcome: <observable outcome or unknown>
- Preserved contracts: <supported behavior and non-negotiable constraints>

## Reviewed Scope
- Included: <inspected source areas>
- Excluded or unreviewed: <areas and reasons>

## Assessment Results
| Skill | Category | Status | Coverage and limitations |
| ----- | -------- | ------ | ------------------------ |
| <shared skill> | SCOPE / SIMPLICITY / COMPLETENESS | COMPLETE / PARTIAL / BLOCKED / UNAVAILABLE | <actual coverage> |

## MVP Alignment
| Component or capability | Classification | Rationale and evidence |
| ----------------------- | -------------- | ---------------------- |
| <component> | KEEP / SIMPLIFY / PROPOSE_DEFER / NEEDS_CONTEXT | <reason and citation> |

## Findings
### IDENT-001: <finding title>
- Category: SCOPE | SIMPLICITY | COMPLETENESS
- Assessment source: <shared skill and finding IDs>
- Priority: MUST | SHOULD | COULD
- Evidence and consumers: <file:line citations and consumer analysis>
- Current behavior: <observed behavior>
- Recommendation: <bounded change proposed, not approved>
- Preserved behavior: <contracts that must remain>
- Reduction: <measure, baseline, target, and method; or not applicable>
- Risk and blast radius: <affected components and uncertainty>
- Verification: <baseline/characterization and acceptance checks>
- Rollback: <safe reversal or migration approach>
- Decision needed: <selection or explicit feature-removal approval>

## Baseline Verification
- Status: NOT_RUN | PASS | KNOWN_FAILURES | INSUFFICIENT
- Evidence: <recorded command results and revision, or none>
- Required checks: <verification needed before changes>

## Deferred Candidates
| Item | Why deferred | MVP alternative | Revisit when | Evidence |
| ---- | ------------ | --------------- | ------------ | -------- |
| <proposed item> | <reason> | <smaller alternative or None> | <trigger> | <citation and rationale> |

## Open Questions
<unresolved MVP requirements, consumers, or compatibility questions>

## Suggested Next Steps
<candidate selection, research/verification prerequisites, then PAW planning>
```

All deferred candidates are proposals, not accepted scope changes. Do not run
`paw-mvp-defer` from this review; it writes repository docs and commits. The
user can select candidates in a subsequent PAW workflow for planning or
committed deferral documentation.

Use relative links resolved from the report directory for citations. Cite the
working-tree files actually inspected; do not use HEAD permalinks for modified
or untracked content.

## Verify and Return

- `COMPLETE`: All three assessments completed for the declared scope, or
  completed with no justified findings. Findings alone do not lower status.
- `PARTIAL`: At least one useful assessment exists, but a result is partial,
  blocked, unavailable, or the assessed code changed.
- `BLOCKED`: No assessment can be completed because essential source, checkout,
  or assessment execution is unavailable.

Treat a valid partial assessment as useful even if it only identifies evidence
gaps. If all results are blocked or unavailable, use `BLOCKED`. If all are
complete but conflicting findings or unresolved MVP goals remain, use `PARTIAL`.

Recheck HEAD and working-tree state before finishing, excluding only the new
review directory. If assessed source changed, revalidate affected findings or
mark the report `PARTIAL` with the stale-snapshot limitation.

Verify the report was written at the selected path, all sections are populated,
citations resolve, and no source or git changes were made by this skill.
If blocked after the checkout is proven, still write the report with specific
blockers. If the output cannot be written, report the failure without claiming
an artifact exists.

Return the report path, status, finding counts by category, and key limitations.
Do not start implementation. A review report does not replace PAW research,
user approval, MVP guards, or plan review.
