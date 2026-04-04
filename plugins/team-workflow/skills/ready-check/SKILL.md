---
name: ready-check
description: Validate implementation readiness before execution — checks spec completeness, architecture decisions, dependencies, test strategy, and acceptance criteria. Returns PASS/CONCERNS/FAIL verdict. Use after planning, before coding.
---

Execute all of the following steps immediately. Do not ask for confirmation.

## PURPOSE

This is a formal quality gate between planning and execution. It prevents wasted implementation time on incomplete or ambiguous specs. Inspired by BMAD-METHOD's implementation readiness check.

## STEP 1: Locate planning artifacts

Find the relevant planning documents:
1. Check for a design spec in `docs/superpowers/specs/` (most recent matching current task)
2. Check for a plan document (most recent in conversation or `docs/superpowers/plans/`)
3. Check `_architecture/artifacts/{task-id}/` if artifact chain exists
4. Read `_architecture/CROSS-CUTTING-DECISIONS.md` for platform constraints
5. Read TaskList for task descriptions and definitions of done

If NO spec or plan exists, immediately return **FAIL** with: "No planning artifacts found. Run brainstorming and writing-plans first."

## STEP 2: Run readiness checklist

Evaluate each item. Mark as PASS, CONCERN, or FAIL:

### Requirements Clarity
- [ ] Requirements are specific and measurable (not vague like "improve performance")
- [ ] Acceptance criteria exist for every requirement
- [ ] Edge cases and error scenarios are addressed
- [ ] No unresolved TBDs, TODOs, or "figure out later" in the spec

### Architecture Decisions
- [ ] Affected services/modules are identified with file paths
- [ ] Technical approach is decided (not "we could do X or Y")
- [ ] Data model changes are specified (if applicable)
- [ ] API contracts are defined (if applicable)
- [ ] No conflicts with CROSS-CUTTING-DECISIONS.md

### Dependencies
- [ ] External dependencies identified (APIs, libraries, services)
- [ ] Internal dependencies identified (other tasks, shared code)
- [ ] Database migrations planned (if applicable)
- [ ] No circular or unresolvable dependencies

### Test Strategy
- [ ] Test approach defined (unit, integration, e2e)
- [ ] Key test cases listed
- [ ] Expected behavior for each scenario documented
- [ ] Test data requirements identified

### Implementation Plan
- [ ] Tasks are decomposed into bite-sized steps
- [ ] Each task has clear start/end conditions
- [ ] Task dependencies are ordered correctly
- [ ] Estimated scope is reasonable (not trying to do too much)

## STEP 3: Render verdict

Count the results:
- **PASS**: All items pass or at most 2 concerns (none failed). Proceed to implementation.
- **CONCERNS**: 3+ concern items but no failures. List all concerns. Ask user: "These concerns exist but aren't blockers. Proceed anyway, or address first?"
- **FAIL**: Any failed items. List all failures. Say: "Implementation blocked. These items must be resolved before coding begins." Then list specific actions needed.

## STEP 4: Save artifact (if artifact chain exists)

If `_architecture/artifacts/` directory exists for this task, save the readiness report:

File: `_architecture/artifacts/{task-id}/readiness.md`

With frontmatter:
```yaml
---
task-id: {id}
phase: readiness
created: {ISO date}
verdict: PASS|CONCERNS|FAIL
predecessors: [design.md, plan.md]
---
```

## STEP 5: Report

Present the full checklist with verdicts, the overall verdict, and if FAIL/CONCERNS, the specific actions needed to resolve.

## NOTES
- This gate is mandatory for Standard and Complex tasks in the /team-start flow
- For Simple tasks, this can be skipped
- For Trivial tasks, this is always skipped
- If the user says "skip" or "override", respect that — they know their task best
- The goal is to CATCH missing pieces, not to add bureaucracy
