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
copilot plugin install lpaw@lpaw-marketplace
```

Refresh, update, or remove later:

```bash
copilot plugin marketplace update lpaw-marketplace
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
copilot plugin marketplace remove lpaw-marketplace
```

Confirm it's gone:

```bash
copilot plugin list
```

## Usage

Drive the loop with the `orchestrator` agent, which coordinates the `coder` and
`reviewer` subagents until the reviewer approves:

```bash
copilot --agent orchestrator -p "Add input validation to the login handler"
```

Or switch agents inside a session with `/agent`.

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
