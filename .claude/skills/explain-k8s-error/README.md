# explain-k8s-error

A live Kubernetes triage assistant. Rather than explaining errors in the abstract, it reads your local Helm values and manifests alongside live `kubectl` output to investigate what's actually happening in your cluster — then explains every finding in plain English, with reasoning and docs links.

## Usage

Run it from the repo where your Helm chart or Kustomize manifests live:

```
/explain-k8s-error
```

Claude will ask what you're observing, then take it from there.

Or give it a head start:

```
/explain-k8s-error payments-api is CrashLoopBackOff
/explain-k8s-error intermittent 5xx on checkout service
/explain-k8s-error PVC data-postgres-0 stuck Terminating
```

## What it does

1. **Reads your local manifests** — finds `values*.yaml`, `Chart.yaml`, or `kustomization.yaml` in the current directory to understand the intended configuration: namespace, resource limits, labels, replicas, probes.

2. **Picks an investigation strategy** — the starting point for a CrashLoopBackOff is different from a 5xx at the ingress level. Claude explains which strategy it's using and why.

3. **Runs non-destructive kubectl commands** — `get`, `describe`, `logs`, `top`, `events`. After each group of commands, it explains what was found and what it means before moving on.

4. **Cross-references live state against your manifests** — if the pod is OOMKilled, it compares actual memory usage against `resources.limits.memory` in your values file.

5. **Synthesizes a plain-English explanation** — what's happening, the root cause, contributing factors, and a link to the relevant Kubernetes documentation.

6. **Proposes fixes** — including the exact manifest change needed, with before/after diffs where applicable.

## Safety

Claude will never autonomously run any command that modifies cluster state. If a fix requires a destructive action (deleting a stuck pod, removing a finalizer, restarting a deployment), it is listed under a **Manual Actions Required** block with full context — what the command does, why it's needed, and whether it's reversible. You run it; Claude doesn't.

## Audience

Designed for developers who know their application but aren't Kubernetes experts. Kubernetes-specific concepts (finalizers, cgroups, endpoint slices, readiness gates) are explained briefly when they're relevant to the investigation — never as a lecture, always tied to what was actually found.
