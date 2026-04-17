# pr-description

Writes a pull request description by combining live `git diff` output with the current conversation context. If you've been working through a problem with Claude before opening the PR, it already knows the "why" — this skill turns that into a description a reviewer can actually use.

## Usage

Run it when you're ready to open a PR:

```
/pr-description
```

Or add context that isn't visible in the diff:

```
/pr-description this is a hotfix for the 3am incident, the root cause was X
```

## What it uses

- **`git diff main...HEAD`** — the full diff for what's in this PR
- **`git log main...HEAD`** — commit history for title and summary context
- **Conversation context** — problem descriptions, decisions made, things called out as out of scope, testing steps already discussed
- **Repo PR template** — checks `.github/pull_request_template.md` first; if found, fills it in rather than using the default structure

## Default template sections

If no repo template exists, the output follows:

- **Summary** — what changed and why, written for a reviewer who wasn't in the conversation
- **Problem** — what was wrong or missing before this PR
- **Changes** — logical changes, not a file inventory
- **Acceptance Criteria** — binary, reviewer-checkable conditions
- **Testing** — what was run and what the result was
- **Notes** — decisions, out-of-scope items, follow-up work

## Pairing with write-runbook

If you just resolved an incident and the fix is in this PR, run `/write-runbook` first to capture the session, then `/pr-description` — the PR description will pull context from the runbook discussion automatically.
