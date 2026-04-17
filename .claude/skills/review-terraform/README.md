# review-terraform

Reviews a Terraform plan with a focus on **changes that look safe but aren't** — resource modifications that Terraform shows as `~` (update) but actually trigger replacement or destruction at the provider level.

## The problem this solves

Terraform's plan output is not always honest about what will happen. Provider bugs and missing `ForceNew` annotations mean that some attribute changes show up as in-place updates but actually destroy and recreate the resource. Classic examples:

- Changing an EKS cluster `name` → Terraform shows `~`, but the entire cluster is destroyed along with all node groups
- Changing an RDS `identifier` → same plan marker, full instance replacement
- Changing an ALB `name` → Terraform shows `~`, load balancer is replaced (new DNS name, all connections dropped)

This skill catches these, explains what will actually happen, and tells you what to do instead.

## Usage

```
/review-terraform
```

Runs `terraform plan` live and reviews the output.

```
/review-terraform tfplan.txt
```

Reviews a saved plan file.

## Output structure

Every review produces four sections:

1. **Destructive Surprises** — changes showing as updates that will actually destroy resources. Includes the resource, attribute, what will actually happen, and a recommendation.
2. **Worth Reviewing** — security group additions, IAM changes, sizing changes, anything that warrants a second look.
3. **Routine Changes** — tag updates, metadata, expected diffs. Summarized, not enumerated.
4. **Other Observations** — cost flags, missing tags, naming issues.

Ends with a clear **Proceed / Proceed with caution / Stop** recommendation.

## Web search

For resource type + attribute combinations not covered by built-in patterns, the skill searches the Terraform Registry documentation to check for `ForceNew` behavior. This adds a few seconds but catches provider-specific gotchas that aren't in the built-in pattern list.

## Phase 2 (planned): Policy-as-code integration

Run `checkov` and `tfsec` automatically and fold findings into the review output.

```bash
pip install checkov
brew install tfsec
```
