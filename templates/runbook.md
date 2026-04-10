# Runbook: {incident or alert name}

**Service:** {service name}
**Severity:** {P1 | P2 | P3}
**Last Updated:** {date}
**Owner:** {team or individual}

## Symptoms

{What does this look like when it's happening? What alerts fire? What do users
experience? Be specific — include metric names, error messages, or log patterns
where known.}

- {symptom 1}
- {symptom 2}

## Likely Causes

{Ranked by frequency/likelihood}

1. **{cause}** — {brief explanation}
2. **{cause}** — {brief explanation}
3. **{cause}** — {brief explanation}

## Investigation Steps

{Walk through how to confirm the cause. Include exact commands.}

### 1. Confirm the scope
```bash
{command}
```

### 2. Check recent changes
```bash
{command}
```

### 3. Inspect logs
```bash
{command}
```

### 4. Check dependencies
```bash
{command}
```

## Remediation

### For cause 1: {cause name}
```bash
{remediation command or steps}
```

### For cause 2: {cause name}
```bash
{remediation command or steps}
```

## Escalation

| Condition | Escalate To | How |
|---|---|---|
| {condition} | {person/team} | {Slack channel / PagerDuty policy} |

## Post-Incident

- [ ] Incident timeline documented
- [ ] Post-mortem scheduled (P1/P2 only)
- [ ] Follow-up tickets created
- [ ] Runbook updated with new findings
