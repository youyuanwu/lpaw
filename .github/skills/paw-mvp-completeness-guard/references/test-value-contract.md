# Shared Test-Value Contract

Use with planning, implementation, review, and current-code assessment. Evaluate
two independent properties: whether testing is proportionate to the risk, and
whether its assertions can detect the relevant broken behavior. Neither test
count nor coverage percentage alone establishes either property.

This reference adds criteria, not an activity, verdict, or permission to run
tools. Read-only guards and current-code assessments inspect tests and recorded
results only; execution and controlled experiments belong to authorized
implementation/verification activities.

## Lightweight Test Rationale

For each proposed or materially changed test group, briefly identify:

| Question | Required reasoning |
| -------- | ------------------ |
| What does it protect? | An acceptance criterion, supported contract, or concrete failure risk. |
| What defect should it detect? | A plausible incorrect implementation that should make this test fail. |
| What does it observe? | Expected output, state, error, or contractual interaction, with an expectation independent of the logic being tested. |
| Why this coverage? | Existing coverage reused and the specific gap requiring any addition or change. |
| Why this test level? | The cheapest reliable level that exercises the behavior, including real boundaries when integration is the risk. |

Use existing plan verification sections, review findings, or verification notes;
do not create another mandatory artifact or per-test inventory. Group tests by
behavior/risk. When assessing unchanged tests, document evidence for findings,
not retrospective paperwork for the entire suite.

## Proportionate Coverage

- Prefer existing tests, fixtures, runners, and helpers. Add a case or assertion
  to an appropriate test before introducing a new suite or harness.
- Select meaningful boundaries and distinct failure classes instead of
  arbitrary input permutations. Exhaustive, property-based, or multi-layer
  testing is justified when it catches distinct material risks; volume alone
  does not establish excess.
- Avoid repeating the same protection across unit, integration, and end-to-end
  tests without explaining each layer's additional failure detection. Do not
  replace boundary coverage with cheaper isolated mocks that miss the risk.
- Prefer externally observable contracts over private helper structure or
  incidental call order. Interaction/order assertions are appropriate when
  the interaction itself is a required contract.
- Existing tests can adequately cover a code change without test-file edits.
  Cite the relevant cases/assertions and their applicability to the change;
  implementation approval also needs current execution evidence.
- Do not add quotas for test count, coverage, runtime, or mutation score unless
  explicitly required by the project. Preserve existing required checks and
  agreed test deliverables; changing those requires the normal decision and
  replanning process, not silently weakening verification.
- Before removing or consolidating tests, show which distinct protections
  remain, where they are exercised, and how preservation will be verified.
  Similar names, inputs, or lines are not proof of redundancy.

## Effective Assertions

Trace the path from the input through the actual behavior under test to the
assertion. Flag:

- Mocking/replacing the behavior being tested and then asserting the mock's
  configured result. Mock external boundaries as needed, not the target logic.
- Computing the expected result with the same production logic, or copying its
  algorithm so the test repeats the same defect. Use contract-derived examples,
  independently justified invariants, or an independent oracle.
- Passing without exercising the target or reaching the required assertion,
  including empty parameter sets, conditional assertions, swallowed failures,
  skipped cases, and unawaited asynchronous work. An intentional skip is not
  passing evidence for the skipped behavior.
- Checking only existence, truthiness, non-null output, or lack of an exception
  when the contract specifies exact content, state, error semantics, or effects.
  Such checks are sufficient only when they express the actual requirement.
- Broad snapshots or permissive assertions that do not distinguish the
  identified plausible defect. Require a justified expected result, not merely
  a newly accepted snapshot of whatever the implementation produces.

For important regression fixes, prefer fail-before/pass-after evidence when
practical. Confirm the earlier failure is caused by the intended assertion,
not setup, compilation, or an unrelated failure. For suspicious assertions,
selectively use a controlled fault in an isolated, disposable fixture or
checkout, then verify cleanup; never disturb user changes. No mandatory full
mutation-testing campaign or new tooling is implied.

Record which defect-detection claims were executed versus inferred by
inspection, with commands, scope, revision/dirty state, outcomes, and limitations
for actual runs. If an experiment is impractical, say so and assess the remaining
evidence; its absence alone is not a blocker. Ineffective coverage of a required
behavior remains a gap even when every current test passes.

## Applying the Criteria

- Simplicity assessment identifies avoidable testing machinery or duplicate
  protection without dropping needed behavioral coverage.
- Completeness assessment identifies missing or vacuous protection of required
  behavior. Repair the smallest relevant test group, not an arbitrary quota.
- Review-feedback triage requires a concrete gap before authorizing "add more
  tests." Existing adequate coverage can justify no change; known missing
  required coverage cannot be deferred as MVP scope. Preserve unresolved
  reviewer blockers and request reassessment with evidence.
- Implementation review inspects actual assertions and current verification,
  not just test-file presence, new test counts, or a passing runner summary.
- For prompt/agent workflows, schema and phrase-presence checks establish
  structural properties only. Behavioral confidence requires representative
  controlled workflow evaluations with expected dispositions, actions, and
  artifacts; do not report structural checks as proof of workflow behavior.

Use the calling skill's existing findings, verification fields, and verdicts.
Planned checks are not executed results. Current-code reviews report unrun
checks as `NOT_RUN` and keep assessment completeness separate from readiness.
