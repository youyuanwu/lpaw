---
name: paw-mvp-defer
description: "Use during PAW MVP scope reduction, review-feedback triage, or repository simplification to record deferred work in an indexed per-work folder, preserve decisions, and commit the documentation."
---

# PAW MVP Deferred Work

> **Execution Context**: Run directly in the PAW or PAW-MVP session, not a read-only
> review subagent. This skill owns deferred-documentation writes and commits.
> It does not implement deferred features or alter approved product scope.

## Inputs and Checkout

- Read the active `WorkflowContext.md` for Work ID, target branch, execution
  binding, and artifact lifecycle.
- Receive the core outcome, guard or `paw-simplify` candidates (including
  `paw-mvp-review-guard` feedback IDs/URLs), accepted simplifications, or user
  decisions from the orchestrator.
- Load `paw-git-operations` before mutations. Prove the execution checkout and
  expected branch; use that checkout for all reads, writes, and git operations.
- Use the normalized Work ID as the record filename. If it is unsafe,
  ambiguous, or collides with the index, return `BLOCKED`; do not invent an ID.

## Repository Layout

Store normal project documentation outside `.paw/`:

```text
docs/mvp-deferred/
├── README.md
├── <work-id>.md
└── <another-work-id>.md
```

`README.md` is a concise index with one row per work item, sorted by Work ID:

```markdown
# Deferred MVP work

| Work ID | Core outcome | Record |
| ------- | ------------ | ------ |
| <work-id> | <observable outcome> | [Deferred decisions](./<work-id>.md) |
```

Each `<work-id>.md` contains only that work item's decisions:

```markdown
# <Work title>: deferred MVP work

- Work ID: <work-id>
- Core outcome: <observable outcome>

## Decisions

### <stable item ID>: <item>
- Status: PROPOSED | DEFERRED | PROMOTED | IMPLEMENTED | REJECTED
- Reason: <why deferred or simplified>
- MVP alternative: <chosen smaller solution, or None>
- Revisit when: <observable trigger or user decision>
- Evidence: <self-contained rationale and durable references>
```

Keep the index and work record in sync. Verify relative links resolve to actual
records. Do not create a folder, index row, or empty work record when there are
no candidates or existing decisions.

## Decision Handling

- Record actual deferred scope, future-valued approaches replaced during
  simplification, and candidate decisions. Do not invent future work or turn
  every rejected implementation into a backlog item.
- Guard recommendations start as `PROPOSED`. Work deferred under the MVP
  Contract becomes `DEFERRED`; explicit requirements and material trade-offs
  remain proposed until the user decides.
- On promotion or rejection, update the same item to `PROMOTED` or `REJECTED`
  with the reason. Mark `IMPLEMENTED` only after verification.
- Use stable item IDs and deduplicate repeated candidates. Preserve the prior
  rationale and record why a status changes rather than deleting history.
- For review-driven candidates, retain the source feedback IDs/URLs and
  distinguish proposed from agreed dispositions. Documentation does not resolve
  a review thread or authorize ignoring a required fix.
- Preserve all other Work IDs and entries. A later `PASS` or empty candidate
  list never resets existing documentation.
- Make entries understandable after `.paw/` cleanup; disposable artifact links
  alone are insufficient evidence.
- This record is required project documentation, not an extra implementation
  phase or permission to implement deferred work.

## Legacy Record Migration

If `docs/mvp-deferred.md` exists, move its decisions into the matching per-work
records and add index links. Preserve every entry, ID, status, and rationale.
Update tracked references to the old path.

Remove the legacy file only after verifying a lossless transfer and include its
removal in the documentation commit. If Work IDs or ownership cannot be
established, return `BLOCKED` rather than guessing or overwriting user content.
Do not repeat migration when the legacy file is already absent.

## Git Persistence

- Commit changes to the index and affected work records together before the
  orchestrator replans, pauses, starts implementation, or opens a PR.
- Use explicit file paths, not `git add .`, `git add -A`, or staging the whole
  folder. Use a path-limited commit so unrelated staged user changes remain
  untouched. Include only owned documentation changes; migration may include
  the legacy file and updated references.
- Follow repository commit conventions. Do not make an empty commit when the
  committed records already match the decisions.
- `commit-and-clean`, `commit-and-persist`, and `never-commit` control `.paw/`
  artifacts, not this folder. Commit and retain the documentation in all modes.
- If commits are prohibited, paths are ignored, changes conflict with user
  edits, or git operations fail, return `BLOCKED` with recovery guidance.

For the index and each affected record, verify:

- `git ls-files --error-unmatch -- <path>` succeeds.
- `git cat-file -e HEAD:<path>` succeeds.
- `git diff --exit-code HEAD -- <path>` shows no pending changes.
- The committed content contains the expected decisions and valid index links.

After branch merges or PAW cleanup, verify the latest index links and decisions
remain in the target branch's committed tree. Before final PR creation or
workflow completion, a local file, staged content, or an unmerged branch commit
is insufficient; return `BLOCKED` if target-branch persistence is missing.

## Completion Response

```markdown
## MVP Deferred Documentation Result: [COMMITTED|UNCHANGED|BLOCKED]

### Files
- [index and work record paths, or "None"]

### Decisions
- [stable item ID and status, or "None"]

### Git Persistence
- Commit: [verified commit SHA, or "None"]
- Target branch: [VERIFIED|PENDING_MERGE|NOT_APPLICABLE]

### Blockers
- [specific issue and recovery guidance, or "None"]
```

`COMMITTED` requires a verified documentation commit. `UNCHANGED` requires
verified existing content, or no decisions and no records to create.
`PENDING_MERGE` is permitted during planning and phase work, never at final PR
creation or workflow completion. Do not report success after a persistence
failure.
