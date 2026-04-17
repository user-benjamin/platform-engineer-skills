# write-runbook

Distills a live debugging or incident session into a reusable runbook. Rather than generating from a description, this skill reads back through the current conversation — the commands you ran, the findings, the dead ends, the fix — and produces a clean document a future on-caller can follow.

## Usage

Run it at the end of a debugging session, after the problem is resolved (or well-understood):

```
/write-runbook
```

Or with a title hint if the inferred name might be ambiguous:

```
/write-runbook payments pod OOMKill under batch load
```

## What it captures

- **Symptoms** — what actually fired or was observed, not generic descriptions
- **Likely causes** — ranked by what the session revealed, with confirmed/ruled-out notes
- **Investigation steps** — the effective path, with exact commands and generalized variables
- **Dead ends** — false trails that cost time, so future responders skip them
- **Remediation** — the fix, step by step, per cause
- **Follow-up** — tickets, ADRs, or monitoring gaps identified during the session

## When to use it

- After resolving a production incident
- After a long debugging session where the root cause wasn't obvious
- After on-call triage that involved multiple commands or teams
- Any time you think "I'll have to do this again someday"

## Phase 1 (current)

Outputs the runbook as a markdown code block. Save to `docs/runbooks/` or your team's runbook store.

## Phase 2 (planned): PagerDuty / Confluence Integration

Automatically attaches the runbook to the originating PagerDuty incident and publishes it to Confluence.

```bash
export PAGERDUTY_API_TOKEN=your-token
export CONFLUENCE_BASE_URL=https://your-org.atlassian.net/wiki
export CONFLUENCE_SPACE_KEY=OPS
```
