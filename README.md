# lpaw

**Looped Agent Workflow** — a GitHub Copilot CLI plugin that runs an iterative
**coder → reviewer → fix** loop until a reviewer approves, keeping numbered review
history under `.reviews/`.

## Components

| Type  | Name           | Purpose                                                              |
| ----- | -------------- | ------------------------------------------------------------------- |
| Agent | `coder`        | Implements the task and resolves review comments.                    |
| Agent | `reviewer`     | Reviews changes and records numbered comments in `.reviews/`.        |
| Agent | `orchestrator` | Drives the loop by delegating to `coder` and `reviewer`.             |

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

The plugin ships three agents. In most cases you only invoke the `orchestrator`,
which coordinates the `coder` and `reviewer` subagents and loops until the
reviewer approves (default: up to 10 rounds).

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
