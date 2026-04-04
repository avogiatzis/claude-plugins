---
name: retro
description: Run a post-epic retrospective — analyzes git history, review reports, escalations, and time tracking to generate lessons learned with actionable improvements. Use after PR merge or significant feature completion.
---

Execute all of the following steps immediately. Do not ask for confirmation.

## PURPOSE

Formalized learning loop after completing significant work. Prevents the same mistakes from recurring across sessions. Inspired by BMAD-METHOD's retrospective workflow.

## STEP 1: Gather evidence

Collect data from multiple sources (run in parallel where possible):

### Git History
- `git log --oneline` for the feature branch (or last N commits if on main)
- `git diff --stat` for total scope of changes
- Count: files changed, insertions, deletions
- Identify: how many commits, any reverts, any fixup commits (indicate rework)

### Review Reports
- Check `_architecture/artifacts/{task-id}/review-report-*.md` for review cycle results
- Note: how many iterations needed, what issues were found, what was auto-fixed vs escalated
- If no artifacts exist, check conversation history for review results

### Escalations
- Read all files in `_architecture/escalations/` related to this work
- Note: what was escalated, was it resolved, how long did resolution take

### Time Tracking (if Tasker MCP available)
- Call `mcp__tasker__tasker_hours` with period "today" or relevant period
- Note: total time, breakdown by phase (research vs implementation vs review)

### Task Completion
- Check TaskList for completed/incomplete tasks
- Note: tasks completed vs planned, any tasks that were added mid-session, any dropped

## STEP 1.5: Agent Staleness Check

Check if the project's domain expert agents need regeneration:

1. **Check fingerprint age:** Read `_architecture/agents/.fingerprint` if it exists. Note the generation date from the `# Generated:` line.

2. **Detect new services:** Check git history for new service indicator files added since the fingerprint was generated:
   ```bash
   git log --diff-filter=A --name-only --since="{fingerprint-date}" -- '*.csproj' '**/package.json' '**/go.mod' '**/Cargo.toml' '**/pom.xml' '**/pyproject.toml'
   ```

3. **Detect convention changes:** Check if project-level conventions changed since fingerprint:
   ```bash
   git log --oneline --since="{fingerprint-date}" -- CLAUDE.md _architecture/CROSS-CUTTING-DECISIONS.md
   ```

4. **Report findings:**
   - If new services found: "New service detected: {path}. Agent definitions may be stale. Run `/team-discover` to regenerate."
   - If conventions changed: "Project conventions updated since last discovery. Consider running `/team-discover --force` to update agent best practices."
   - If no fingerprint exists: "No agent definitions found. Run `/team-discover` to generate domain expert agents."
   - If everything is current: skip (no report needed)

5. **Add to action items** (Step 3) if staleness detected:
   ```
   - **Action**: Run `/team-discover --force` to regenerate agent definitions
     **Applies to**: team-discover, team-start
     **Why**: {reason — new service, convention changes, etc.}
     **Priority**: Important
   ```

## STEP 2: Analyze patterns

Based on the evidence, analyze:

### What Went Well
- Implementations that passed review on first iteration
- Good architectural decisions that simplified work
- Effective patterns or approaches worth repeating
- Accurate estimates or scoping

### What Didn't Go Well
- Review iterations > 1 (what caused rework?)
- Escalated issues (what blocked progress?)
- Scope creep (tasks added that weren't planned)
- Failed approaches (what was tried and abandoned?)
- Time overruns (where did time go unexpectedly?)

### Root Causes
For each problem identified, ask "why" until you reach the root:
- Was the spec incomplete? → Planning gap
- Was the approach wrong? → Research/architecture gap
- Was a dependency missed? → Dependency analysis gap
- Was the code buggy? → Testing gap
- Was the scope too large? → Decomposition gap

### Metrics
| Metric | Value |
|--------|-------|
| Tasks planned | X |
| Tasks completed | Y |
| Tasks added mid-session | Z |
| Review iterations | N |
| Critical issues found in review | N |
| Escalations | N |
| Reverts / fixup commits | N |
| Time tracked | Xh Xm |

## STEP 3: Generate action items

For each root cause, create a specific, actionable improvement:

Format:
```
- **Action**: [specific change to make]
  **Applies to**: [which skill, rule, or process]
  **Why**: [root cause this addresses]
  **Priority**: Critical / Important / Nice-to-have
```

Examples:
- "Add database migration check to /ready-check checklist" → applies to ready-check skill
- "Always run integration tests before review-cycle, not just unit tests" → applies to review-cycle
- "Break epics with > 5 services into multiple sessions" → applies to /team-start scoping

Include any agent staleness action items from Step 1.5.

## STEP 4: Save retrospective

Save to: `_architecture/decisions/RETRO-{YYYY-MM-DD}-{task-slug}.md`

With structure:
```markdown
---
type: retrospective
task-id: {id}
date: {ISO date}
branch: {branch name}
---

# Retrospective: {task title}

## Summary
{1-2 sentence summary of what was done}

## Metrics
{metrics table from Step 2}

## What Went Well
{bulleted list}

## What Didn't Go Well
{bulleted list with root causes}

## Action Items
{formatted action items from Step 3}

## Process Changes
{any changes to CROSS-CUTTING-DECISIONS.md recommended}
```

Also save to artifact chain if it exists:
`_architecture/artifacts/{task-id}/retro.md`

## STEP 5: Apply learnings

If any action items have Priority: Critical and affect platform rules:
1. Read current `_architecture/CROSS-CUTTING-DECISIONS.md`
2. Propose the addition to the user
3. If approved, update the file

If action items affect a specific skill:
1. Note which skill file needs updating
2. Report to user: "Retro suggests updating {skill} — {what to change}"

## STEP 6: Report

Present the full retrospective to the user with emphasis on action items.

## NOTES
- This skill is triggered by /team-stop when significant work was done (3+ tasks completed or Complex task)
- Can also be invoked directly via `/retro`
- Focus on patterns, not individual mistakes — the goal is systemic improvement
- Keep action items specific and achievable, not vague aspirations
- If time tracking isn't available, skip that section rather than guessing
