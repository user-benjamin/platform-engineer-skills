# project-proposal

Generates a structured project proposal from a plain-English description.

## Usage

```
/project-proposal migrate our on-prem Kafka cluster to Confluent Cloud — currently managing 3 brokers ourselves, ops toil is killing the team
```

Claude will produce a fully populated proposal with an executive summary, problem statement, goals, success metrics, timeline, resource requirements, risks, stakeholders, and out-of-scope section.

## Example Output

```markdown
# Confluent Cloud Migration

**Status:** Draft
**Owner:** Platform Engineering
**Date:** 2026-04-17
...
```

## Phase 1 (current)

Generates proposal content as markdown. Copy/paste into Confluence, Notion, or a GitHub document.

## Phase 2 (planned): Confluence API Integration

Automatically creates the page in a target Confluence space via the REST API.

### Setup

1. Generate a Confluence API token at `https://id.atlassian.com/manage-profile/security/api-tokens`
2. Add to your environment:
   ```bash
   export CONFLUENCE_BASE_URL=https://your-org.atlassian.net/wiki
   export CONFLUENCE_USER=your@email.com
   export CONFLUENCE_API_TOKEN=your-token
   export CONFLUENCE_SPACE_KEY=PLAT   # target space key
   ```
3. Install dependencies:
   ```bash
   pip install anthropic requests
   ```

Phase 2 will use `agents/proposal_publisher.py` to POST the generated proposal
directly to Confluence and return the page URL.
