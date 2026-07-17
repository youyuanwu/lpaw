# review-loop

A GitHub Copilot CLI plugin that runs an iterative **coder → reviewer → fix** loop
until a reviewer approves, keeping numbered review history under `.reviews/`.

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
copilot plugin marketplace add OWNER/lpaw
copilot plugin install review-loop@review-loop-marketplace
```

Refresh, update, or remove later:

```bash
copilot plugin marketplace update review-loop-marketplace
copilot plugin update review-loop
copilot plugin uninstall review-loop
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
copilot plugin uninstall review-loop
```

Optionally, also unregister the marketplace. This is refused if any of its
plugins are still installed; pass `--force` to remove those too:

```bash
copilot plugin marketplace remove review-loop-marketplace
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

Each task gets two paired logs: `.reviews/review-NNN.md` (reviewer comments) and
`.reviews/response-NNN.md` (the coder's reply). Every review file has a top
`Status:` / `Round:` summary and a `## Round N` section per review pass; the
matching response file records, per round, what the coder **Addressed** and what
it **Rejected** with rationale. The reviewer reads the response before each new
round, so the full iteration history is preserved and auditable.

## License

MIT
