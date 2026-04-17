---
name: project-proposal
description: Generate a structured project proposal from a plain-English description. Produces an executive summary, goals, success metrics, timeline, resource requirements, risks, and stakeholder sections.
argument-hint: <describe the project or initiative in plain English>
---

The user wants to create a project proposal. Their input is: $ARGUMENTS

Generate a complete, production-quality project proposal using the template below.

## Instructions

**Status:** Default to `Draft`. Only use `Proposed` if the user indicates it has been reviewed.

**Executive Summary:** Write 2–3 sentences. What is the project, why does it matter, and what does success look like? This is what a VP skims — make it count.

**Problem Statement:** Be specific about the pain. Quantify where possible (e.g. "manual process takes 4 hours per week", "3 outages in the last quarter caused by X"). A reader should feel the problem without needing prior context.

**Goals:** Write 3–5 goals that are outcome-oriented, not activity-oriented. "Reduce deploy time by 50%" is a goal. "Implement a new CI/CD pipeline" is a task.

**Success Metrics:** For each goal, define at least one measurable metric. Prefer metrics that can be tracked in a dashboard — error rates, latency p99, time-to-deploy, ticket counts. Specify the target value and timeframe.

**Proposed Solution:** 2–4 sentences on the approach. Not an implementation plan — a high-level direction. Include the key technical or process decisions that define the approach.

**Timeline & Milestones:** Infer a realistic timeline from the scope of the project. Use phases (Discovery, Build, Rollout) rather than precise dates unless the user provides them. Each milestone should have a clear deliverable.

**Resource Requirements:** Infer from the project description:
- **Team:** engineering effort in weeks or headcount, any specialist roles needed
- **Infrastructure:** new services, cloud spend, tooling licenses
- **Dependencies:** other teams, external vendors, approvals needed

**Risks & Mitigations:** Identify 3–5 realistic risks — technical, organizational, or timeline-related. For each, provide a concrete mitigation. Don't list generic risks that apply to every project.

**Stakeholders:** Infer from context who needs to be aware, consulted, or who must approve. Use RACI framing: Responsible, Accountable, Consulted, Informed.

**Out of Scope:** Call out 2–3 things this project explicitly does NOT cover. This is critical for preventing scope creep.

## Template

Use this exact template structure:

!`cat ~/.claude/templates/project-proposal.md`

## Output

Write the completed proposal as a markdown code block so it can be copied directly into Confluence, Notion, or a GitHub document. After the proposal, add a brief **Notes** section with any assumptions you made or questions the sponsor should answer before socializing the proposal.
