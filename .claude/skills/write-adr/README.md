# write-adr

Generates a fully populated Architecture Decision Record from a plain-English description of a technical decision.

## Usage

```
/write-adr we're choosing Karpenter over Cluster Autoscaler for node provisioning — Karpenter is faster and lets us use spot more aggressively
```

```
/write-adr should we use Istio or Linkerd for our service mesh? we need mTLS, traffic shaping, and the team has zero service mesh experience
```

Claude will produce a complete ADR with context, decision drivers, options considered (with honest pros/cons), the decision rationale, and consequences. It auto-detects existing ADR numbering from `docs/adr/` so you don't have to track it manually.

## Output

The ADR is output as a markdown code block, ready to paste into `docs/adr/ADR-{number}-{title}.md`. If the repo has a `docs/adr/` directory, Claude will suggest the correct filename.

## Phase 1 (current)

Generates ADR content as markdown. Copy into your repo's `docs/adr/` directory.

## Phase 2 (planned): Auto-write to repo

Automatically writes the ADR file to `docs/adr/`, stages it with `git add`, and opens a draft PR.

No additional setup needed — uses the local git environment.
