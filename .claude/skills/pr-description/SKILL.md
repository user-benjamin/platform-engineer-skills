---
name: pr-description
description: Writes a pull request description by combining live git diff context with the current conversation. Checks for an existing repo PR template first; falls back to a standard template with summary, problem, changes, acceptance criteria, testing, and notes.
argument-hint: <optional: any context about the PR not visible in the diff>
---

The user wants to write a pull request description. Additional context from the user (if provided): $ARGUMENTS

## Step 1: Gather context

Run these read-only commands to understand what's in this PR:

!`git diff main...HEAD --stat 2>/dev/null || git diff master...HEAD --stat 2>/dev/null || git diff --stat HEAD~1 2>/dev/null`

!`git log main...HEAD --oneline 2>/dev/null || git log master...HEAD --oneline 2>/dev/null || git log --oneline HEAD~5 2>/dev/null`

!`git diff main...HEAD 2>/dev/null || git diff master...HEAD 2>/dev/null || git diff HEAD~1 2>/dev/null`

Also look back through the current conversation for any relevant context — problem descriptions, decisions made, things explicitly called out as out of scope, testing steps already discussed.

## Step 2: Check for an existing PR template

!`cat .github/pull_request_template.md 2>/dev/null || cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null || cat docs/pull_request_template.md 2>/dev/null || true`

If a repo template exists, use it as the structure. Fill every section using context from the diff and conversation — do not leave placeholder text in any section.

If no repo template exists, use the shared template:

!`cat ~/.claude/templates/pull-request.md`

## Step 3: Write the description

**Title:** Write a concise PR title. Imperative mood, under 72 characters. "Add Gateway API HTTPRoute support" not "Added support for Gateway API HTTPRoutes" or "Gateway API changes".

**Summary:** Synthesise what the PR does from the diff and conversation context. Do not describe what files changed — describe what behaviour changed and why. A reviewer who hasn't been in this conversation should understand the PR from this section alone.

**Problem:** Use the conversation context first — this is where the "why" usually lives. Fall back to inferring from commit messages and diff if the conversation doesn't make it explicit.

**Changes:** Extract the meaningful logical changes from the diff. Group related file changes into a single bullet rather than listing every file. Skip formatting-only or generated file changes unless they're the point of the PR.

**Acceptance Criteria:** Write 3–5 specific, binary conditions. Derive these from the conversation (were there any explicit requirements or goals discussed?) and from the nature of the changes. These should be checkable by a reviewer without running the code.

**Testing:** Look for testing steps discussed in the conversation. If none were mentioned, describe what a reviewer could do to verify the change based on the diff — include specific commands, endpoints, or config values where you can infer them.

**Notes:** Include any decisions made during the conversation that a reviewer should understand, things intentionally left out of scope, or follow-up work that was identified but not included.

## Output

Write the completed PR description as a markdown code block so it can be pasted directly into GitHub, GitLab, or Bitbucket.

After the description, add a brief **Heads up** line if there's anything missing that the user should fill in before opening the PR — e.g. "Testing section assumes you have a working cluster — add specific output if you ran this against a real environment."
