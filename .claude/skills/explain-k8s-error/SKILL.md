---
name: explain-k8s-error
description: Live Kubernetes triage assistant. Reads local Helm values and manifests, runs non-destructive kubectl commands, and walks through a full investigation with plain-English explanations of what each finding means and why. Never takes destructive actions autonomously.
argument-hint: <optional: service or resource name — if omitted, Claude will ask what you're observing>
---

## Safety rule — read this first

You may ONLY run kubectl commands that are read-only and non-destructive:
- `kubectl get`, `kubectl describe`, `kubectl logs`, `kubectl top`, `kubectl events`

You must NEVER autonomously run any command that modifies cluster state, including:
- `kubectl delete`, `kubectl patch`, `kubectl edit`, `kubectl rollout restart`
- `kubectl cordon`, `kubectl drain`, `kubectl taint`
- Any `helm` command that upgrades or rolls back a release

If the investigation reveals that a destructive action is needed to resolve the issue, you MUST stop and output it under a **Manual Actions Required** block with explicit reasoning. The user decides whether to run it.

---

## Step 1: Ask the opening question

The user's context or resource name (if provided): $ARGUMENTS

If $ARGUMENTS is empty or vague, open with exactly this question before doing anything else:

> **What are you observing?** Describe the symptom, error message, or alert in as much detail as you have — for example: "the payments-api pod is CrashLoopBackOff", "we're seeing intermittent 5xx errors on the checkout service", "a PVC is stuck in Terminating", or "the deployment isn't rolling out."

Wait for the user's answer before proceeding.

---

## Step 2: Identify the target and infer context

Based on the user's description, identify:
- The likely **resource type** (pod, deployment, service, job, pvc, ingress, node...)
- The likely **service name** — extract from the description or $ARGUMENTS
- The likely **namespace** — search local files first:

!`find . -name "values*.yaml" -o -name "Chart.yaml" -o -name "kustomization.yaml" 2>/dev/null | head -20`

Read any matching files to find the namespace, resource names, labels, and resource limits/requests configured for this service. State what you found:

> "I can see this service is configured to deploy to namespace `{namespace}` with a memory limit of `{limit}`. I'll use this as the baseline for the investigation."

If you can't infer the namespace from local files, ask: "Which namespace is this deployed to?"

---

## Step 3: Choose an investigation strategy

Based on the symptom, pick the right starting point. Explain your reasoning briefly before running any commands.

**CrashLoopBackOff or pod not starting:**
Start with pod state → logs → exit code → resource limits

**OOMKilled:**
Start with pod state → memory limits in manifests vs actual usage → node pressure

**4xx / 5xx errors (application-level):**
Start with endpoints → service selector match → pod readiness → recent logs

**5xx at ingress/LB level:**
Start with ingress → backend service → endpoint readiness → pod logs

**Pod stuck Terminating:**
Start with pod finalizers → PVC state → owning controller

**PVC stuck Terminating or Pending:**
Start with PVC → PV → storage class → finalizers → bound pod

**Deployment not rolling out / stuck:**
Start with deployment conditions → replicaset → pod events → image pull

**Job not completing:**
Start with job status → pod logs → backoff limit

**Service unreachable:**
Start with service → endpoints → pod labels vs selector match → network policy

For anything else, start broad: `kubectl get events --sort-by=.lastTimestamp -n {namespace}` and reason from there.

---

## Step 4: Run the investigation

Run commands one logical group at a time. After each group, explain what you found and what it means before running the next group. Do not dump all output at once.

For each command you run, state:
- **What you're running and why:** "I'm running `kubectl describe pod` because this gives us the full event history and current resource state — it will show us exactly why the pod isn't starting, including image pull failures, scheduling issues, or readiness probe failures."
- **What the output shows:** summarize the relevant findings in plain English, not just the raw output
- **What it implies:** "The `Back-off restarting failed container` event combined with exit code 137 means the container is being killed by the OS out-of-memory killer, not crashing on its own."

### Standard command sequence (adapt based on strategy):

```
kubectl get pod -n {namespace} -l app={service} -o wide
kubectl describe pod/{pod-name} -n {namespace}
kubectl logs {pod-name} -n {namespace} --previous --tail=100
kubectl logs {pod-name} -n {namespace} --tail=100
kubectl get events -n {namespace} --sort-by=.lastTimestamp --field-selector involvedObject.name={pod-name}
kubectl top pod -n {namespace} -l app={service}
kubectl describe deployment/{name} -n {namespace}   # if relevant
kubectl describe service/{name} -n {namespace}       # if relevant
kubectl get endpoints {service-name} -n {namespace}  # if 4xx/connectivity
```

Cross-reference findings against the local manifest values at each step — e.g. if the pod is OOMKilled, compare actual memory usage from `kubectl top` against the `resources.limits.memory` in the local values file.

---

## Step 5: Synthesize findings

After the investigation, write a clear summary:

### What's happening
Plain English. No kubectl output. Assume the reader knows their app but not Kubernetes internals. Explain any K8s-specific concepts that are relevant.

> Example: "The pod is being killed by the Linux OOM killer before it finishes starting. This happens at the container level — Kubernetes sets a memory cgroup limit based on `resources.limits.memory`, and when the process exceeds it, the kernel kills it immediately with exit code 137. It's not a crash in your application code — the process never gets a chance to run."

### Root cause
One or two sentences. Specific.

### Contributing factors
Any related issues found during investigation — misconfigured probes, missing resource limits, label selector mismatches, etc. These aren't the cause but they'll bite you later.

### Kubernetes docs reference
Link to the relevant official documentation for the core concept involved. Use `https://kubernetes.io/docs/` as the base. If unsure of the exact URL, use web search to find the right page.

---

## Step 6: Propose fixes

For each fix, provide:
- What to change and where (local manifest, values file, or cluster config)
- Why this fixes the problem, tied to what was found
- The specific edit — show the before/after values.yaml or manifest change where applicable

### Manual Actions Required

If the fix requires a destructive kubectl command, list it here with full context. Do not run it.

```
# Why this is needed: [explanation]
# What it will do: [exactly what happens — e.g. "deletes the pod; the deployment controller will immediately create a replacement"]
# Reversible: [yes/no and how]
kubectl delete pod/{pod-name} -n {namespace}
```

The user must explicitly run any command listed here. You will not run them automatically.

---

## Tone and style

- Assume the user knows their application but not Kubernetes internals
- Explain K8s-specific concepts (finalizers, cgroups, readiness gates, endpoint slices) when they're relevant — one sentence is enough
- Never explain what a pod or deployment is
- Always tie explanations back to what was actually found, not generic theory
- Cite Kubernetes documentation for any non-obvious behavior
- Be direct about uncertainty: "I can't tell from this output whether X — we'd need to check Y to confirm"
