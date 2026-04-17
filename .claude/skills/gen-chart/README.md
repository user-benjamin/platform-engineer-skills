# gen-chart

Generates a production-grade Helm chart or Kustomize manifest set for a service. Scans the repo for context (Dockerfile, existing manifests, package metadata), asks about anything it can't infer, walks you through component selection with plain-English explanations, then writes all files to disk.

## Usage

```
/gen-chart
```

Claude will open a short conversation to determine paradigm, environments, and components before generating anything.

```
/gen-chart helm
/gen-chart kustomize payments-api
```

Passing the paradigm (and optionally service name) skips the first question.

## Conversation flow

1. **Paradigm** — Helm, Kustomize, or raw? If you're unsure, Claude will explain the tradeoffs and suggest how to find out what your org uses.
2. **Repo scan** — Claude reads your Dockerfile, `package.json`, `go.mod`, and any existing manifests to infer service name, port, and runtime. Reports what it found.
3. **Gaps** — Anything it couldn't infer (image repository, port, tag strategy), it asks about explicitly. No assumptions.
4. **Environments** — You name them. `dev/qa/prod`, `merge/preprod/prod`, whatever your org uses — those exact names appear in every generated file.
5. **Components** — Every optional component is listed with a plain-English explanation and, where relevant, a caveat. You pick what to include. "All of them" is a valid answer.
6. **Confirmation** — Claude lists every file it's about to write and asks before touching the filesystem.
7. **Write** — Files are written to disk and confirmed against `git status`.

## Components

Always included:
- Resource limits and requests
- Liveness and readiness probes
- Security context (non-root, no privilege escalation)
- Rolling update strategy (`maxSurge: 1, maxUnavailable: 0`)

Optional (with explanations and caveats during selection):
- **HPA** — autoscaling based on CPU/memory
- **PDB** — pod disruption budget (read the caveat — this one bites people)
- **External Secrets** — pulls secrets from AWS Secrets Manager, Vault, etc.
- **Gateway API (HTTPRoute)** — external traffic routing via Kubernetes Gateway API (the modern replacement for deprecated Ingress)
- **ServiceMonitor** — Prometheus scraping
- **NetworkPolicy** — least-privilege pod networking
- **Pod Anti-Affinity** — spread replicas across nodes (soft rule, never blocks scheduling)
- **ServiceAccount + RBAC** — dedicated service account for IRSA or K8s API access

## After generation

Every run ends with a **Before You Deploy** section listing values that need updating before the chart is usable — health check paths, Gateway name, image repository, hostnames per environment, secret store paths.
