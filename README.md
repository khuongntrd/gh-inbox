# gh-inbox

Personal GitHub inbox via the `gh` CLI. Lists open PRs and issues across an organization, filtered for things that need your attention.

## Requirements

- [`gh`](https://cli.github.com/) authenticated via `gh auth login`
- `jq`

## Install

```sh
gh extension install khuongntrd/gh-inbox
```

## Setup

The org is resolved in this order:

1. `--org <org>` flag
2. `$GITHUB_ORG`
3. Owner of the current repo (via `gh repo view`) when run inside a clone

Set a default if you want it to work anywhere:

```sh
export GITHUB_ORG=your-org
```

## Commands

```sh
gh inbox pr review     # PRs where you are a requested reviewer (excludes drafts)
gh inbox pr mine       # PRs you authored
gh inbox pr changes    # PRs with review decision CHANGES_REQUESTED
gh inbox todo          # Issues assigned to you with a "Todo-like" project status
```

Each command prints one URL per line, suitable for piping:

```sh
gh inbox pr review | xargs -n1 open
```

## Options

| Flag | Description | Default |
| --- | --- | --- |
| `--org <org>` | Override the org | `$GITHUB_ORG` |
| `--status "A,B,C"` | Comma-separated status names for `todo` | `📝 Todo` |
| `--debug` | Verbose logs + dump raw GraphQL to `debug_*.json` | off |

## Environment

| Variable | Purpose |
| --- | --- |
| `GITHUB_ORG` | Default organization |
| `INBOX_TODO_STATUS` | Default status filter for `todo` (overridden by `--status`) |
| `DEBUG=1` | Same as `--debug` |

## Examples

```sh
# Reviews waiting on you in a specific org
gh inbox pr review --org acme

# Custom todo statuses
gh inbox todo --status "Todo,Ready,In Progress"
INBOX_TODO_STATUS="Todo,Ready" gh inbox todo
```

## Debug

```sh
DEBUG=1 gh inbox pr review
```

Writes the raw GraphQL response to `debug_prs.json` / `debug_issues.json` in the current directory.
