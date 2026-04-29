---
name: gh-inbox
description: Personal GitHub inbox via the `gh inbox` CLI extension. Lists open PRs and issues across an organization that need the user's attention — review requests, authored PRs, changes-requested PRs, and todo-status assigned issues. Load when the user asks "what's on my plate", "what do I need to review", "what PRs are waiting on me", "what changes were requested", "show my todos", "what GitHub work is open", "catch me up on GitHub", or otherwise wants a triaged view of their GitHub work.
---

# gh-inbox

Personal GitHub triage. Each command prints one URL per line by default, or a JSON array of `{title, description, url}` with `--format json`.

## When to use

Use `gh inbox` instead of `gh pr list` / `gh issue list` / raw GraphQL when the user wants a **triaged, attention-filtered view** scoped to themselves across an entire org. The tool already filters by reviewer, author, review decision, and project status — don't re-implement that logic.

## Commands

| Command | What it returns |
| --- | --- |
| `gh inbox pr review` | Open, non-draft PRs where the user is a requested reviewer |
| `gh inbox pr mine` | Open PRs the user authored |
| `gh inbox pr changes` | Open PRs with `reviewDecision == CHANGES_REQUESTED` |
| `gh inbox todo` | Open issues assigned to the user with a "Todo-like" project status |

## Org resolution

Resolved in this order: `--org <org>` → `$GITHUB_ORG` → owner of the current repo (`gh repo view`). If none is set and not in a repo, the command errors.

## Useful flags

- `--org <org>` — override the org
- `--status "📝 Todo,Todo,Ready,In Progress"` — comma-separated status names for `todo` (default: `📝 Todo`)
- `--format simple|json` — `simple` is one URL per line (default); `json` returns `[{title, description, url}]` for parsing
- `--debug` — verbose logs + dump raw GraphQL to `debug_prs.json` / `debug_issues.json`

## Patterns

Open everything needing review:

```sh
gh inbox pr review | xargs -n1 open
```

Read titles + bodies for triage (best when you want Claude to summarize):

```sh
gh inbox pr review --format json
```

Custom todo statuses:

```sh
gh inbox todo --status "Todo,Ready,In Progress"
```

## Requirements

- `gh` authenticated (`gh auth login`)
- `jq`

## Notes for Claude

- Prefer `--format json` when piping into further analysis or summarization — `simple` is for human eyeballing.
- All four commands are read-only; safe to run without confirmation.
- If the user is in a non-org repo (e.g. personal fork), `gh repo view` may resolve a username as the org and the GraphQL `organization(login:)` query will fail. In that case, ask which org to target.
- The tool fetches the first 50 repos × 50 PRs per call — fine for most orgs but may miss items in very large orgs.
