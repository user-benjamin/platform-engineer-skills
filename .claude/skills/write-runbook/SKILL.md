---
name: write-runbook
description: Distill a debugging or incident session into a reusable runbook. Reviews the current conversation to extract symptoms, investigation steps, commands run, and resolution, then writes a clean runbook a future on-caller can follow.
argument-hint: <optional: name or description of the incident/alert — defaults to inferring from the conversation>
---

The user has just worked through a problem (debugging session, incident, investigation) and wants to capture what was learned as a reusable runbook. The optional title or context is: $ARGUMENTS

## Your job

Look back at the full conversation above. Extract what actually happened:

- What symptoms or alerts triggered the investigation
- What commands were run and what their output revealed
- What dead ends or red herrings were hit (these matter — they save future responders time)
- What the root cause turned out to be
- What the fix or mitigation was, step by step
- What follow-up work was identified

Then write a clean, generalized runbook that a future on-caller can follow **without needing to have been in this conversation**. Do not just transcribe the conversation — distill it into the right structure for a runbook.

## Instructions

**Symptoms:** Write what a responder would actually observe — alert names, error messages, log lines, user impact. Use what surfaced in the session, not generic placeholders.

**Likely Causes:** Rank by what the session revealed. If one cause was confirmed, mark it as such. If others were ruled out, note that too — ruling out fast is valuable.

**Investigation Steps:** Reconstruct the effective investigation path, not the full exploratory one. Cut the dead ends from the main flow but call them out in a "Things that looked suspicious but weren't" note if they cost significant time — future responders will hit the same false trails.

**Commands:** Extract the exact commands from the session. Generalize any environment-specific values (pod names, IPs, timestamps) into clearly labeled variables like `$POD_NAME` or `$NAMESPACE`.

**Remediation:** Write the fix as a step-by-step procedure. If there were multiple possible causes with different fixes, give a section per cause.

**Severity:** Infer from impact — user-facing outage = P1, degraded performance = P2, internal tool issue = P3.

**Escalation:** Infer from context. If the session involved paging someone or reaching out to another team, capture that trigger.

## Template

Use this exact template structure:

!`cat ~/.claude/templates/runbook.md`

## Output

Write the completed runbook as a markdown code block. After the runbook, add a brief **Session Notes** section capturing:
- Anything that would be lost without the conversation context (e.g. "this only happens on Tuesdays after the batch job runs")
- Suggested filename: `runbook-{kebab-case-title}.md`
- Any follow-up tickets or ADRs that should be written based on this incident
