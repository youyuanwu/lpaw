# lpaw

**Looped Agent Workflow** — a GitHub Copilot CLI plugin that runs an iterative
**coder → reviewer → fix** loop until a reviewer approves, keeping numbered review
history under `.reviews/`. It also provides a PAW workflow for minimal viable
product delivery without scope creep and a skill for assessing existing
repository simplification.

## Components

| Type  | Name                   | Purpose                                                        |
| ----- | ---------------------- | -------------------------------------------------------------- |
| Agent | `coder`                | Implements the task and resolves review comments.              |
| Agent | `reviewer`             | Records actionable comments in numbered `.reviews/` files.     |
| Agent | `orchestrator`         | Drives the loop by delegating to `coder` and `reviewer`.       |
| Agent | `PAW-MVP`                    | Runs PAW for the smallest complete MVP without scope creep. |
| Skill | `paw-mvp-scope-guard`        | Keeps only capabilities required for the core user outcome. |
| Skill | `paw-mvp-simplicity-guard`   | Minimizes architecture, dependencies, and implementation phases. |
| Skill | `paw-mvp-completeness-guard` | Ensures the narrow MVP remains usable, safe, and testable. |
| Skill | `paw-mvp-defer`              | Organizes deferred decisions and commits their documentation. |
| Skill | `paw-simplify`               | Assesses existing code for evidence-backed, behavior-preserving simplification. |

## Requirements

- [GitHub Copilot CLI](https://github.com/github/copilot-cli) installed and authenticated.

## Install

### From a marketplace (recommended)

```bash
copilot plugin marketplace add youyuanwu/lpaw
copilot plugin marketplace browse lpaw
copilot plugin install lpaw@lpaw
```

Refresh, update, or remove later:

```bash
copilot plugin marketplace update lpaw
copilot plugin update lpaw
copilot plugin uninstall lpaw
```

### Local (development)

```bash
copilot plugin install /absolute/path/to/lpaw
copilot plugin list
```

> Re-run `copilot plugin install <path>` after editing plugin files to refresh the cache.

## Uninstall

Remove the plugin:

```bash
copilot plugin uninstall lpaw
```

Optionally, also unregister the marketplace. This is refused if any of its
plugins are still installed; pass `--force` to remove those too:

```bash
copilot plugin marketplace remove lpaw
```

Confirm it's gone:

```bash
copilot plugin list
```

## Usage

The plugin ships four agents. For the lpaw review loop, invoke `orchestrator`,
which coordinates the `coder` and `reviewer` subagents and loops until the
reviewer approves (default: up to 10 rounds).

For a minimal product workflow, invoke `PAW-MVP`. It runs
[Phased Agent Workflow](https://github.com/lossyrob/phased-agent-workflow)
in full mode with three focused MVP gates between planning and PAW's standard
plan review.

### One-shot (non-interactive)

```bash
copilot --agent orchestrator -p "Add input validation to the login handler"
```

### Interactive session

Start Copilot from your project root, then select or address the agent:

```bash
copilot
```

Inside the session:

- Pick the agent from a menu:

  ```text
  /agent
  ```

  Choose `orchestrator`, then type your task as a normal prompt.

- Or name the agent directly in your prompt — Copilot infers which to use:

  ```text
  Use the orchestrator agent to add retry logic to the HTTP client
  ```

### Writing a good prompt

The orchestrator works best when the task is concrete and verifiable. Include:

- **What to build/change** and **where** (files, module, or feature).
- **Acceptance criteria** the reviewer can check (behavior, edge cases, tests).
- **Constraints** (style, dependencies to avoid, performance limits).
- Optionally, a **round limit** — e.g. "with at most 3 review rounds".

Examples:

```text
Use the orchestrator agent to add pagination to the /users API.
Requirements: page & page_size query params, default 20 / max 100, return
total count, 400 on invalid params, and cover the edge cases with tests.
```

```text
Use the orchestrator to refactor config loading into a single module,
keeping behavior identical. Max 3 review rounds.
```

### Running individual agents

You can also invoke a single role directly:

- Just a review of your current changes (no code edits):

  ```bash
  copilot --agent reviewer -p "Review my staged changes"
  ```

- Address an existing review file with the coder:

  ```bash
  copilot --agent coder -p "Address the comments in .reviews/review-002.md"
  ```

### What to expect

- The orchestrator gives a one-line status after each round (e.g.
  `Round 2 (review-002.md): reviewer requested more work`).
- Progress is recorded under `.reviews/` (see [Review history](#review-history)).
- The loop stops when a `review-NNN.md` reaches `Status: APPROVED`, or when the
  round limit is hit — after which any remaining comments are reported.

> Tip: subagent delegation requires a model that supports the `task` tool. If the
> orchestrator says it can't spawn subagents, switch models with `/model` and pick
> a capable first-party model.

## PAW MVP workflow

Install PAW separately alongside lpaw:

```bash
copilot plugin marketplace add lossyrob/phased-agent-workflow
copilot plugin install paw-workflow@paw-workflow
```

Then invoke the wrapper directly:

```bash
copilot --agent PAW-MVP -p "Build the smallest usable API rate-limiting MVP"
```

`PAW-MVP` is a top-level PAW-compatible orchestrator, not a parent that runs PAW
as a nested subagent. It defaults new work to PAW's full workflow with local
review strategy and planning-document review enabled, while targeting one
testable end-to-end slice. It prefers existing components, defers secondary
workflows and speculative extensibility, and preserves correctness, security,
compatibility, and validation work. Minimal product scope does not require an
abbreviated development process.

The agent runs three gates after every `paw-planning` activity, including
plan-review revisions and promoted phase candidates:

1. **Scope** classifies planned capabilities as keep, defer, or remove.
2. **Simplicity** rejects unnecessary abstractions, dependencies, architectural
   spread, and implementation phases.
3. **Completeness** confirms the reduced plan still provides a usable, safe,
   testable end-to-end product slice.

Optional phase candidates are deferred unless they are necessary for the core
MVP outcome. Each gate reads PAW's planning artifacts without editing them and
returns one of:

- `PASS` — proceed to `paw-plan-review`.
- `REVISE` — remove or defer nonessential work, then run the guard again.
- `ESCALATE` — pause for an MVP scope or architecture decision.
- `BLOCKED` — recover missing or inconsistent planning evidence.

### Configuration defaults and overrides

New workflows use these fallback defaults:

```yaml
workflow_mode: full
review_strategy: local
planning_docs_review: enabled
```

Override them in your request, through a PAW preset, or through user-level PAW
defaults. Precedence, from lowest to highest, is PAW defaults, wrapper defaults,
user-level defaults, the selected preset, and explicit request settings.

For example, request a shorter process explicitly:

```bash
copilot --agent PAW-MVP -p "Build the API rate-limiting MVP. Use workflow_mode: minimal and planning_docs_review: disabled."
```

PAW's configuration constraints still apply: minimal mode requires local
review strategy. The agent validates settings and confirms the resolved
configuration before initialization. Resuming work preserves its existing
configuration unless you request a change.

Changing workflow mode or human-review settings does not bypass the MVP scope,
simplicity, and completeness gates or committed deferred-work documentation.

### Committed deferred-work documentation

`PAW-MVP` runs the `paw-mvp-defer` utility skill directly whenever scope is
deferred or a simpler approach leaves a useful capability for later. The skill
organizes committed documentation in the target repository:

```text
docs/mvp-deferred/
├── README.md          # Index linking each workflow's record
├── <work-id>.md       # Decisions for one workflow
└── <another-work-id>.md
```

The index has one entry per Work ID; each work record includes:

- Work ID and core MVP outcome
- Stable item ID and decision status
- Reason for deferral and the chosen MVP alternative
- A concrete trigger for revisiting the item
- Enough rationale and durable references to survive PAW artifact cleanup

Guard suggestions are recorded as `PROPOSED`, not silently treated as approved.
Accepted deferrals become `DEFERRED`; subsequent promotion, implementation, or
rejection updates the same entry. Replanning and later `PASS` results never
erase earlier decisions.

The guards remain read-only. `paw-mvp-defer` writes and commits the index and
affected work records together before replanning, pausing, implementation, or PR
creation. It checks that the records reach the target branch's committed tree
and survive cleanup. The folder is normal project documentation outside
`.paw/`, so it remains committed in all PAW artifact lifecycle modes, including
`never-commit` and `commit-and-clean`. An explicit prohibition on repository
commits is reported as a blocker rather than silently bypassed.

Existing `docs/mvp-deferred.md` records are migrated into the indexed folder
without losing entries or history. No empty folder or work record is created
when there are no candidates or deferrals, and no extra implementation phase is
added solely for this documentation.

The wrapper currently tracks PAW agent revision
`465a5c9743f66722634cb0fb057b1580169a0661`. When updating the PAW submodule,
compare its `agents/PAW.agent.md` orchestration contract with
`.github/agents/PAW-MVP.agent.md` and carry forward upstream workflow changes
without removing the MVP guard pipeline.

## Simplifying an existing repository

`paw-simplify` is a read-only assessment skill, not an automatic cleanup agent.
It runs after PAW code research and before planning. It identifies bounded
opportunities to remove unused machinery, consolidate duplication, flatten
unnecessary layers, and replace dependencies with existing capabilities.

Use the opt-in path in `PAW-MVP`:

```bash
copilot --agent PAW-MVP -p "Simplify the configuration subsystem. Preserve existing behavior and public contracts. Assess with paw-simplify and present candidates for approval before planning changes."
```

Or instruct the standard PAW agent explicitly, with both plugins installed:

```text
Use PAW to simplify the configuration subsystem without removing features.
Establish baseline verification, then delegate to paw-simplify after
paw-code-research and before paw-planning. Present candidates for approval.
Plan only selected candidates, retain preserved contracts and measurement
targets through implementation and review, and verify the resulting reduction.
Use paw-mvp-defer to document and commit actual postponed candidates.
```

The assessment returns `READY`, `NEEDS_DECISION`, or `BLOCKED`. `READY` means
ready for candidate selection, not permission to change code. Each candidate
includes evidence, consumer analysis, preserved behavior, a measurable
reduction, risks, verification requirements, and rollback.

Feature removal requires explicit approval; no references found is not proof
of unused code. Missing baseline tests or uncertain consumers must be resolved
before implementation. Changes proceed through normal PAW planning and reviewed
phases, with postponed candidates recorded in the deferred-work folder. The
standard PAW agent does not gain an automatic transition from installing lpaw;
use the explicit instruction above. Ordinary MVP feature work is unchanged.

## Review history

Review artifacts use **one file per round** under `.reviews/`, numbered three
digits:

- `review-NNN.md` — the reviewer's comments for round N (`Status:` / `Round:`
  header plus a `- [ ]` checklist).
- `response-NNN.md` — the coder's reply for round N (same number), listing what
  was **Addressed** and **Rejected** with rationale.

The round number equals the file number (`review-001.md` = Round 1,
`review-002.md` = Round 2, …). Each new review pass is a new file; the reviewer
reads the previous round's `review-(N-1).md` and `response-(N-1).md` before
writing, so the full iteration history is preserved and auditable. An
`APPROVED` review file marks the end of the task.

## License

MIT
