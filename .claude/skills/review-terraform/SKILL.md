---
name: review-terraform
description: Review a terraform plan for security issues, cost surprises, missing tags, and — critically — changes that Terraform shows as updates but actually trigger resource replacement or destruction. Uses web search to verify known destructive-but-hidden patterns per provider docs.
argument-hint: <optional: path to plan file, or leave blank to run terraform plan live>
---

The user wants a Terraform plan review. Their context or plan file path (if provided): $ARGUMENTS

## Step 1: Get the plan

If the user provided a file path, read it:
!`cat $ARGUMENTS 2>/dev/null`

Otherwise, run the plan live:
!`terraform plan -no-color 2>&1`

If neither works, ask the user to paste the plan output or run `terraform plan -out=tfplan && terraform show -no-color tfplan`.

## Step 2: Parse and categorize

Read through every resource in the plan. For each change, assign it to one of three categories:

### Category A — Routine (no action needed)
Changes that are expected, safe, and low-risk. Report these in a collapsed summary — the reviewer just needs to know they're accounted for.
- Tag/label updates
- IAM policy document reformatting with no semantic change
- Annotation or metadata changes
- Version bumps on well-understood dependencies where the change is clearly intentional

### Category B — Worth Reviewing
Changes that are fine in most cases but warrant a second look. Flag these individually with a brief note on what to verify.
- Security group rule additions (is this port/CIDR intentional?)
- IAM permission additions (are these least-privilege?)
- Instance type or size changes (cost impact?)
- Storage size changes (is this increase intentional? decreases are often destructive)
- New resources being created (expected? correctly sized?)
- Resources being destroyed (is this intentional?)

### Category C — Destructive Surprises
**This is the most important category.** These are changes that Terraform shows as `~` (update) or even just a diff, but that actually trigger resource replacement or destruction at the provider level. Terraform often cannot detect these because the provider doesn't correctly annotate the `ForceNew` behavior.

For every resource being modified, check:

1. **Look for explicit signals in the plan itself:**
   - `# forces replacement` annotation on any attribute
   - `-/+` destroy-then-create markers
   - `replacement` in the plan output

2. **Apply known patterns from memory** — these are confirmed destructive-but-hidden:
   - `aws_eks_cluster`: any change to `name` destroys the cluster and all node groups
   - `aws_eks_node_group`: changes to `node_group_name`, `subnet_ids`, `instance_types` (in some provider versions), or `launch_template` `id` (vs `name`) trigger replacement
   - `aws_rds_instance`: `identifier` rename destroys the instance; `engine_version` minor upgrades may force reboot
   - `aws_elasticache_replication_group`: `replication_group_id` rename destroys the cluster
   - `aws_db_subnet_group`: `name` change destroys and recreates
   - `aws_lb` / `aws_alb`: changing `name`, `internal`, `load_balancer_type`, or `subnets` (removing) forces replacement
   - `aws_security_group`: changing `name` or `vpc_id` forces replacement
   - `aws_iam_role`: `name` or `name_prefix` change forces replacement
   - `aws_s3_bucket`: `bucket` name change creates a new bucket (old one not deleted — data stays, but references break)
   - `google_container_cluster`: `name`, `location`, `network`, `subnetwork` force replacement
   - `azurerm_kubernetes_cluster`: `name`, `resource_group_name`, `location`, `dns_prefix` force replacement

3. **For any resource type + attribute combination NOT covered above**, use web search to check:
   - Search: `terraform [provider_resource_type] [attribute_name] "forces new resource"` or `"forces replacement"`
   - Check the Terraform Registry documentation for that resource type
   - Look for `ForceNew: true` or `Requires replacement` notes in provider docs

## Step 3: Write the review

Structure your output as follows:

---

## Terraform Plan Review

**Plan summary:** {X to add, Y to change, Z to destroy — from the plan footer}

---

### Destructive Surprises
{If any — list each one with:}
- **Resource:** `resource_type.resource_name`
- **Attribute changed:** `attribute_name` from `old_value` → `new_value`
- **Why it's destructive:** [explain what actually happens — e.g. "EKS cluster replacement also destroys all managed node groups and requires re-joining worker nodes"]
- **Source:** [Terraform registry link or plan annotation]
- **Recommendation:** [rename vs use lifecycle ignore_changes vs split into a separate targeted apply]

If none: state explicitly "No hidden destructive changes detected."

---

### Worth Reviewing
{List each Category B item with a one-line flag and what to verify}

---

### Routine Changes
{One-line summary: "N routine changes (tag updates, metadata, X, Y) — no action needed"}

---

### Other Observations
{Security, cost, missing tags, naming conventions, anything else worth noting that doesn't fit above}

---

## Step 4: Conclude

End with a clear recommendation:
- **Proceed** — no destructive surprises, Category B items reviewed and understood
- **Proceed with caution** — Category B items need sign-off, no destructive surprises
- **Stop** — destructive surprises found; do not apply without explicit sign-off and a rollback plan
