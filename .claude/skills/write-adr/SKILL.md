---
name: write-adr
description: Write a fully populated Architecture Decision Record from a plain-English description of a technical decision. Interviews the user if context is thin, then writes the ADR to docs/adr/.
argument-hint: <describe the decision or the problem you're solving>
---

The user wants to write an Architecture Decision Record. Their input is: $ARGUMENTS

Generate a complete, production-quality ADR using the template below.

## Instructions

**ADR Number:** Check for existing ADRs in `docs/adr/` using `!`ls docs/adr/ 2>/dev/null | grep -E '^[0-9]+' | sort | tail -1``. Increment by 1. If the directory is empty or doesn't exist, start at ADR-0001.

**Status:** Default to `Proposed`. Only use `Accepted` if the user explicitly says the decision has already been made.

**Deciders:** Infer from context if mentioned. Otherwise use a placeholder like `{platform-team}`.

**Context:** This is the most important section. Explain the situation clearly enough that a new engineer joining the team in a year can understand why this decision was necessary. Include:
- What problem or constraint triggered this decision
- Any prior decisions or existing architecture that frames the choice
- Non-obvious technical or business pressures

**Decision Drivers:** Extract the real constraints from the user's description — performance, cost, operational complexity, team expertise, compliance, time-to-market. Don't list generic drivers that apply to everything.

**Options Considered:** Generate 2–3 realistic options. Always include the status quo or "do nothing" as an option if relevant. For each option, be honest about both pros and cons — an option with no cons is a bad option analysis.

**Decision:** State the chosen option and justify it specifically against the decision drivers. "We chose X because it best satisfies driver Y and Z, despite the trade-off of W" is good. "We chose X because it is the best option" is not.

**Consequences:** Be candid. Positive outcomes should be specific and tied to the drivers. Negative trade-offs and risks should be honest — future readers need to understand what was accepted, not just what was gained.

**Output path:** Write the file to `docs/adr/ADR-{number}-{kebab-case-title}.md` if the user is working in a repo where that makes sense. Otherwise output as a markdown code block.

## Template

Use this exact template structure:

!`cat ~/.claude/templates/adr.md`

## Output

Write the completed ADR as a markdown code block. After the ADR, add a brief **Notes** section with:
- Any assumptions made about options that weren't mentioned by the user
- Questions the deciders should answer before changing status to Accepted
- Suggestions for related ADRs that may need to be written or updated
