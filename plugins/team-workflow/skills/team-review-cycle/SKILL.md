---
name: team-review-cycle
description: Run a multi-agent review cycle on the current feature branch — code review, tests, domain validation, then fix loop (max 3 iterations). Invoke after implementation is complete.
---

Execute all of the following steps immediately. Do not ask for confirmation. Do not describe what you will do. Just do it.

## PRECONDITIONS

Before starting, ensure:
1. All implementation work is committed and pushed to the current feature branch
2. You know what was implemented (from task descriptions, teammate messages, or git log)

If uncommitted changes exist, commit and push them first.

## STEP 1: Gather context for reviewers

Run these in parallel:
1. `git log main..HEAD --oneline` — all commits on the feature branch
2. `git diff main...HEAD --stat` — summary of all changed files
3. `git diff main...HEAD` — full diff for review
4. Read `_architecture/CROSS-CUTTING-DECISIONS.md` if it exists
5. Check TaskList — gather what was implemented and definitions of done

Store the context:
- **WHAT_WAS_IMPLEMENTED**: Summary of all completed tasks
- **BASE_SHA**: `git merge-base main HEAD`
- **HEAD_SHA**: `git rev-parse HEAD`
- **AFFECTED_SERVICES**: Which services/directories were modified

## STEP 2: Spawn review agents (parallel)

Spawn ALL review agents in a single message for parallel execution.

### Agent 1: Code Reviewer
Use `superpowers:code-reviewer` subagent_type. Provide:
- What was implemented
- Git range (BASE_SHA..HEAD_SHA)
- Project rules from CROSS-CUTTING-DECISIONS.md or CLAUDE.md
- Ask for: Strengths, Critical/Important/Minor issues, Assessment (ready to merge?)

### Agent 2: Test Runner
Use project's test-runner agent if available (`.claude/agents/test-runner.md`), else `general-purpose`.
- Run all tests for affected services
- Report: total/passed/failed/skipped per service
- Identify new failures vs pre-existing
- Note test coverage gaps

### Agent 3: Domain Expert Reviewer
Use the domain-specific agent for the primary service affected (e.g., `lisi-data-dev` for Lisi-Data).
- Review business logic validity
- Check data integrity, API contracts, edge cases
- Validate against domain-specific patterns
- DO NOT write code — only review and report

### Agent 4: Build Verification
Use `general-purpose` agent.
- Run build for all affected services
- Report errors and warnings

### Agent 5: Adversarial Reviewer
Use `general-purpose` agent. Provide the full diff and what was implemented. Prompt:

"You are a cynical, thorough adversarial reviewer. Your job is NOT to review what was implemented — the code reviewer handles that. Your job is to find what's MISSING and what could go WRONG.

Analyze the diff and implementation summary. Find:
1. **Missing implementations** — features in the spec/tasks that aren't in the code
2. **Breakable paths** — race conditions, resource leaks, unhandled error paths
3. **Wrong assumptions** — hardcoded values, platform-specific assumptions, magic numbers
4. **Security gaps** — injection vectors, auth bypass, data exposure, input validation gaps
5. **Integration risks** — how this change affects other services, API contract breaks

Produce 5-10 ranked findings. For each:
- **Severity**: Critical / Important / Minor
- **Category**: Missing / Breakable / Assumption / Security / Integration
- **Location**: file:line (if applicable)
- **What could go wrong**: concrete scenario
- **Suggested mitigation**: specific fix or check"

### Agent 6: Edge Case Hunter
Use `general-purpose` agent. Provide the full diff. Prompt:

"You are an exhaustive edge-case analyst. For every function or method modified in this diff, analyze ALL branching paths.

For each branch point, systematically check:
- Null/undefined/empty inputs
- Boundary values (0, -1, MAX_INT, empty string, empty collection)
- Type coercion / casting edge cases
- Concurrent access / race conditions
- Error propagation (upstream failures, partial failures)
- Resource cleanup (exceptions mid-operation, disposal)
- Unicode / encoding edge cases (if string handling)
- Timezone / locale edge cases (if date/time handling)

Output findings as a structured list. For each:
- **File**: path
- **Line**: number
- **Function**: name
- **Edge case**: description
- **Current behavior**: what happens now
- **Expected behavior**: what should happen
- **Severity**: Critical / Important / Minor"

## STEP 3: Collect and synthesize results

Wait for all agents. Compile unified report:

```markdown
# Review Cycle — Iteration {N}/3

## Build: PASS/FAIL
## Tests: X passed, Y failed, Z new failures
## Code Review: {verdict}
## Domain Review: {verdict}

## Critical Issues (Must Fix)
1. [source: reviewer] issue description — file:line

## Important Issues (Should Fix)
1. [source: reviewer] issue description — file:line

## Minor / Suggestions
1. ...

## Adversarial Review: {count} findings
1. [{severity}] {category} — {description} — {file:line}

## Edge Case Analysis: {count} findings
1. [{severity}] {edge case} — {file:line} — Current: {behavior}, Expected: {expected}
```

**Present this report to the user.**

## STEP 3.5: Save review artifact

If `_architecture/artifacts/` exists for the current task, save the review report:

File: `_architecture/artifacts/{task-id}/review-report-iteration-{N}.md`

With frontmatter:
```yaml
---
task-id: {id}
phase: review
iteration: {N}
created: {ISO date}
verdict: PASS|ISSUES_FOUND|ESCALATE
predecessors: [plan.md, readiness.md]
---
```

## STEP 4: Decision point

### All clean → Done
Report "Review cycle passed. Ready for PR." and stop.

### Issues found AND iteration < 3 → Fix cycle (Step 5)

### Iteration = 3 AND still issues → Escalate
Report remaining issues. Say: "Max iterations reached. Remaining issues need manual attention." Stop.

## STEP 5: Fix cycle

1. Group issues by service
2. Spawn fix agents (one per affected service, using domain-specific agents, `mode: bypassPermissions`)
3. Each fix agent gets: specific issues, file paths, reviewer guidance, instruction to make minimal targeted fixes
4. Wait for fixes, commit and push
5. Return to STEP 1 (increment iteration counter)

## NOTES
- Max 3 iterations. Do not loop forever.
- Only fix Critical and Important issues. Minor/suggestions are noted but not auto-fixed.
- Adversarial and edge-case findings at Critical/Important severity are included in the fix cycle alongside code reviewer issues.
- Fix agents must make minimal, targeted changes — no refactoring.
- If a fix agent can't resolve an issue, escalate to the user.
- For Simple tasks (scale-adaptive ceremony), only spawn Agents 1-4. Skip Adversarial and Edge Case agents.
- For Standard and Complex tasks, spawn all 6 agents.
