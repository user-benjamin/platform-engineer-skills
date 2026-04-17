---
name: gen-chart
description: Generates a production-grade Helm chart or Kustomize manifests for a service. Scans the repo for context, asks about any gaps before assuming, confirms components and environments with the user, then writes files to disk.
argument-hint: <optional: paradigm (helm|kustomize) and/or service name>
---

The user wants to generate a Kubernetes chart or manifest set. Their context (if provided): $ARGUMENTS

---

## Phase 1: Determine the paradigm

Check $ARGUMENTS for a paradigm hint (`helm`, `kustomize`, `raw`).

If not specified, ask:

> **What packaging paradigm should I use?**
>
> - **Helm** — templated YAML with a `values.yaml` for configuration. Good if your org already uses Helm, if you need to publish the chart as a package, or if you want strong env-override patterns via `values.{env}.yaml`.
> - **Kustomize** — plain YAML `base/` with `overlays/` per environment. No templating language — patches are applied as strategic merges or JSON patches. Good if you prefer "it's just YAML" and your org uses Kustomize or ArgoCD with Kustomize.
> - **Raw manifests** — plain YAML, no tooling. Good for simple one-off resources or if you'll layer your own tooling on top.
>
> **Not sure?** Check if other services in your org use Helm (`Chart.yaml` in their repos) or Kustomize (`kustomization.yaml`). Your platform team will likely have a preference — it's worth asking before generating something that doesn't fit.

Wait for the user's answer before proceeding.

---

## Phase 2: Scan the repo for context

Before asking anything else, read what's available in the working directory. Run these — they are read-only:

!`find . -name "Dockerfile" -o -name "docker-compose.yml" -o -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "pom.xml" -o -name "build.gradle" 2>/dev/null | head -20`

!`find . -name "Chart.yaml" -o -name "kustomization.yaml" -o -name "values*.yaml" 2>/dev/null | head -20`

!`cat Dockerfile 2>/dev/null || true`

Read any files found to infer:
- **Service name** — from directory name, `package.json` name field, `go.mod` module name, or existing chart metadata
- **Container port** — from `EXPOSE` in Dockerfile or existing manifests
- **Language/runtime** — from Dockerfile base image or dependency files
- **Existing resource configuration** — from any existing values files

After scanning, report what you found and what you're inferring:

> "Here's what I found in this repo:
> - Service name: `{name}` (inferred from `{source}`)
> - Port: `{port}` (from `{source}`)
> - Runtime: `{runtime}` (from `{source}`)
> - [anything else relevant]"

---

## Phase 3: Ask about gaps

**Do not assume or invent values for anything missing.** For each of the following that you could not infer from the repo, ask before proceeding:

- **Service name** — if not found: "What should the service be named? This will be used as the Helm release name / Kustomize base directory name."
- **Container port** — if not found: "What port does the container listen on?"
- **Image repository** — if not found: "What's the container image repository? (e.g. `123456789.dkr.ecr.us-east-1.amazonaws.com/payments-api`)"
- **Image tag strategy** — "Should the image tag be pinned in values (e.g. `v1.2.3`) or templated as a CI variable (e.g. `{{ .Values.image.tag }}`)? Most teams use a CI variable."

Ask all gap questions in a single message, not one at a time.

---

## Phase 4: Environments

Ask:

> **What environments do you deploy to?** (e.g. `dev`, `staging`, `prod` — or your org's equivalent)
>
> I'll generate a separate values file (Helm) or overlay (Kustomize) for each one with appropriate defaults — lower resource limits and replica counts for non-production, production-grade settings for prod.

Wait for the answer. Use the exact names the user provides throughout all generated files.

---

## Phase 5: Component selection

Present every optional component with a plain-English explanation. For components with known gotchas, include a caveat.

Ask:

> **Which of the following should I include?** Say "all of them" or list the ones you want. For anything you're unsure about, the safer choice is to leave it out — it's easier to add later than to debug unexpected behavior from something you didn't fully intend to include.
>
> ---
>
> **HPA — Horizontal Pod Autoscaler**
> Automatically scales the number of pod replicas up or down based on CPU or memory usage. Good for services with variable traffic.
> ⚠️ *Requires the Kubernetes Metrics Server to be installed in your cluster. If it isn't, the HPA will be created but won't do anything. Check with your platform team.*
>
> **PDB — Pod Disruption Budget**
> Guarantees that a minimum number of your pods stay running during voluntary disruptions — like a node drain for maintenance or a cluster upgrade.
> ⚠️ *This can block rollouts and node drains if misconfigured. A common failure mode: `minAvailable: 1` on a single-replica deployment means the old pod can't be evicted during a rollout because it would violate the budget. If you run database migrations as part of your deploy, a PDB can cause those to stall. Leave this out if you're not sure — it's one to add deliberately once you understand your rollout behaviour.*
>
> **External Secrets**
> Pulls secrets from an external store (AWS Secrets Manager, Vault, GCP Secret Manager) into Kubernetes Secrets automatically. Better than storing secret values in your values files.
> ⚠️ *Requires the External Secrets Operator to be installed in your cluster. Ask your platform team if it's available.*
>
> **Gateway API (HTTPRoute)**
> Exposes your service outside the cluster via an HTTP/HTTPS endpoint using the Kubernetes Gateway API — the modern, standards-based replacement for the deprecated Ingress resource. Generates an `HTTPRoute` resource that attaches to a `Gateway` in your cluster.
> ⚠️ *Requires a Gateway API-compatible controller installed in your cluster (e.g. Envoy Gateway, Istio, Cilium, GKE Gateway Controller). You'll also need to know the name of the `Gateway` resource your platform team has provisioned — I'll ask for this if you select it.*
>
> **ServiceMonitor (Prometheus)**
> Tells Prometheus to scrape metrics from your service. Only useful if your service exposes a `/metrics` endpoint.
> ⚠️ *Requires the Prometheus Operator (commonly installed via `kube-prometheus-stack`). Ask your platform team if it's available.*
>
> **NetworkPolicy**
> Restricts which other pods and namespaces can send traffic to your service. Enforces least-privilege networking.
> ⚠️ *Only enforced if your cluster uses a CNI plugin that supports NetworkPolicy (Calico, Cilium, Weave). On clusters using the default kubenet or flannel without NetworkPolicy support, this resource will be created but silently ignored. Ask your platform team.*
>
> **Pod Anti-Affinity**
> Spreads your pod replicas across different nodes so a single node failure doesn't take down all replicas at once.
> ⚠️ *If your cluster doesn't have enough nodes to place each replica on a different node, pods may fail to schedule. I'll use a soft preference rule (`preferredDuringSchedulingIgnoredDuringExecution`) so this never blocks scheduling — it'll spread where it can.*
>
> **ServiceAccount + RBAC**
> Creates a dedicated Kubernetes ServiceAccount for your pods and optionally binds roles to it.
> ⚠️ *Only needed if your application talks to the Kubernetes API directly, or if you're using IRSA (IAM Roles for Service Accounts on EKS) to give your pods AWS permissions. If you're not doing either of those things, leave this out.*
>
> ---
>
> The following are **always included** — they're non-negotiable defaults for production-grade workloads:
> - **Resource limits and requests** — sets CPU and memory boundaries so your pod can't starve the node
> - **Liveness and readiness probes** — tells Kubernetes when your app is healthy and ready to receive traffic
> - **Security context** — runs the container as a non-root user with a read-only root filesystem where possible

If the user selects **Gateway API**, ask as a follow-up:
> "What is the name of the `Gateway` resource in your cluster that this `HTTPRoute` should attach to? And what hostname should it respond on per environment?"

Wait for the user's selections before generating anything.

---

## Phase 6: Confirm before writing

Summarise what you're about to generate:

> **Ready to generate. Here's what I'll create:**
>
> **Paradigm:** {helm | kustomize | raw}
> **Service:** `{name}`
> **Environments:** {list}
> **Components included:** {list}
> **Components excluded:** {list}
>
> **Files that will be written:**
> {list every file path}
>
> Shall I go ahead?

Wait for explicit confirmation before writing any files.

---

## Phase 7: Generate and write files

Write all files to disk. Do not output them as code blocks — write the actual files.

After writing, run:
!`git status 2>/dev/null || true`

Report which files were written and confirm they appear in git status as untracked.

---

## File structure

### Helm

```
{chart-root}/
  Chart.yaml
  values.yaml                  ← shared defaults
  values.{env}.yaml            ← one per environment (lower replicas/limits for non-prod)
  templates/
    _helpers.tpl
    deployment.yaml
    service.yaml
    serviceaccount.yaml        ← if selected
    httproute.yaml             ← if Gateway API selected
    hpa.yaml                   ← if selected
    pdb.yaml                   ← if selected
    externalsecret.yaml        ← if selected
    servicemonitor.yaml        ← if selected
    networkpolicy.yaml         ← if selected
    NOTES.txt
```

### Kustomize

```
k8s/
  base/
    kustomization.yaml
    deployment.yaml
    service.yaml
    serviceaccount.yaml        ← if selected
    [component files]
  overlays/
    {env}/
      kustomization.yaml
      patch-deployment.yaml    ← replica count, resource limits per env
      patch-httproute.yaml     ← if Gateway API selected, hostname per env
```

### Raw

```
k8s/
  deployment.yaml
  service.yaml
  [component files]
```

---

## Generation standards

Every generated file must:

- Include `app.kubernetes.io/name`, `app.kubernetes.io/instance`, and `app.kubernetes.io/version` labels on all resources
- Set `resources.requests` and `resources.limits` for CPU and memory on every container
- Include liveness and readiness probes (use HTTP GET on the app port at `/healthz` by default — note this in the output and tell the user to update the path if their app uses a different health check endpoint)
- Set `securityContext.runAsNonRoot: true` and `securityContext.allowPrivilegeEscalation: false`
- Set `imagePullPolicy: IfNotPresent` for non-prod, `Always` for prod
- Use `minReadySeconds: 10` and a `RollingUpdate` strategy with `maxSurge: 1, maxUnavailable: 0` on all deployments

For env-specific values files / overlays:
- Non-prod environments: `replicas: 1`, modest resource limits
- Prod: `replicas: 2` minimum, production-grade resource limits

After writing files, list any values the user must update before deploying (image repository, health check path, Gateway name, hostnames, secret store paths, etc.) under a **Before You Deploy** section.
