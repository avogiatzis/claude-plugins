---
name: team-start
description: Start a team session — auto-pick next tasker task, research if vague, create feature branch, spawn teammates, implement, then review cycle. Execute immediately without asking.
---

Execute all of the following steps immediately. Do not ask for confirmation. Do not describe what you will do. Just do it.

## STEP 1: Bootstrap project structure (if needed)

Check if `_architecture/` exists in the project root. If it does NOT exist, this is a brand new project — create the scaffolding:

1a. Create `_architecture/` directory with these files:

**`_architecture/PLATFORM-STATE.md`**:
```markdown
# Platform State

**Last updated:** [now]

## Services / Modules
_No services yet. Will be populated when work begins._

## Infrastructure
_TBD_
```

**`_architecture/CROSS-CUTTING-DECISIONS.md`**:
```markdown
# Cross-Cutting Decisions

Platform-wide rules that all services/modules must follow.

_No decisions yet. The architect will add decisions here as they're made._
```

**`_architecture/NEXT-SESSION.md`**:
```markdown
# Next Session Brief

_First session. No prior context._
```

1b. Create `_architecture/decisions/`, `_architecture/escalations/`, and `_architecture/propagations/` directories.

## STEP 1.5: Discovery gate

Check if `_architecture/agents/` exists and contains agent definition files:
- If missing or empty: invoke `/team-discover` to scan the project and generate domain expert agent definitions
- If exists: check `_architecture/agents/.fingerprint` for staleness via **two independent conditions** — trigger `/team-discover --force` if **either** fires:
  1. **Structural staleness:** recompute the directory structure hash (sorted service indicator file paths). If hash differs from the one stored inside `.fingerprint`, regenerate — the repo has grown/shrunk services since last discovery.
  2. **Temporal staleness (NEW):** check the `.fingerprint` file's modification time. If it is more than **3 days old** (wall clock, not commits), regenerate — framework best practices and service internals drift independently of directory structure, so cached agent definitions go stale even when the hash still matches.
  - If neither condition fires: proceed with cached agents.

This step ensures domain expert agents are available before team spawning and prevents stale agent definitions from persisting across weeks of project evolution. The discovery engine will research framework best practices and generate rich agent definitions automatically.

If `/team-discover` is not available (plugin not fully loaded), skip this step and proceed with manual agent selection.

## STEP 2: Pick up a task and start tracking time

Check if the user specified a tasker task (e.g., `/team-start on task #2` or context from the conversation).

**If a specific task was mentioned:** use that task.

**If no task specified (and tasker MCP is available):** auto-pick the next one:
1. Call `mcp__tasker__tasker_list` (status: "todo") to get all pending tasks
2. Pick the first task (lowest ID) — this is the next task in priority order
3. If no tasks exist, ask the user what to work on

**If no tasker MCP:** proceed without task tracking — user will direct the work.

Once a task is identified:
1. Call `mcp__tasker__tasker_get` to read the full task details
2. Update the tasker task status to `in-progress` via `mcp__tasker__tasker_update`
3. **Start AI time tracking immediately:** call `mcp__tasker__tasker_ai_start` with the task ID and the task title. Time starts NOW — all research, exploration, and planning counts as work.

## STEP 3: Read state files

Read these files now:
- `_architecture/NEXT-SESSION.md`
- `_architecture/PLATFORM-STATE.md`
- `_architecture/CROSS-CUTTING-DECISIONS.md`
- All files in `_architecture/escalations/`
- Any existing plan documents in the project (check `docs/` or project root)

## STEP 3.5: Classify task complexity (Scale-Adaptive Ceremony)

Auto-detect task complexity to right-size the workflow. Check for user overrides first:
- `/team-start --quick` → Force **Trivial**
- `/team-start --full` → Force **Complex**
- `/team-start --level {trivial|simple|standard|complex}` → Force specific level

If no override, auto-classify based on:

| Signal | Trivial | Simple | Standard | Complex |
|--------|---------|--------|----------|---------|
| Keywords in task | typo, bump, config, rename | bug fix, small feature, patch | feature, integration, refactor | migrate, new service, multi-service, architecture |
| Services affected | 1 file | 1 service | 1-2 services | 3+ services |
| Estimated scope | 1-3 files | 3-10 files | 10-20 files | 20+ files |
| Has subtasks | No | Maybe | Yes | Yes, with dependencies |

### Write complexity artifact

After classifying, persist the complexity level so the enforcement hook can read it:
1. Create `_architecture/artifacts/{task-id}/` if it doesn't exist
2. Write `_architecture/artifacts/{task-id}/complexity.txt` containing just the level: `trivial`, `simple`, `standard`, or `complex`

### Workflow per complexity level:

**Trivial** — Direct fix, no ceremony:
- Skip brainstorming, skip team spawn, skip review-cycle
- Direct fix → commit → push → done
- Skip to STEP POST-IMPLEMENTATION (just commit and update tasker)

**Simple** — Light ceremony:
- Light brainstorming (confirm approach, skip Socratic exploration)
- 1 teammate max
- Abbreviated review: code reviewer only (skip domain/build/adversarial/edge-case)

**Standard** — Full current flow:
- Full brainstorming → plan → team → review-cycle (all 6 agents)

**Complex** — Full flow + extras:
- PRFAQ exercise first (invoke `/prfaq` skill if available)
- Named research agents (Step 4)
- Full brainstorming → plan → readiness gate (`/ready-check`) → team → review-cycle (all 6 agents)
- Retro after completion (`/retro`)

Report the classification to the user: "Task classified as **{level}**. Adjusting workflow accordingly."

## STEP 4: Research phase (if task is vague)

Evaluate whether the task has enough detail to immediately start implementation. A task is **clear enough** if you know:
- Which service(s) are affected
- What the expected behavior should be
- Where in the code to look

A task is **too vague** if it's just a symptom or a one-liner without context (e.g., "X doesn't work", "user can't see Y").

**If the task is clear enough:** skip to Step 5.

**If the task is vague:**

1. **Investigate the problem** using role-specific research agents based on complexity:

   **Trivial/Simple tasks:** Research directly — use Explore agents to search codebase, read files, check git history. No role specialization needed.

   **Standard/Complex tasks:** Spawn named research agents in parallel for focused investigation:

   - **Analyst Agent** (general-purpose): "You are a requirements analyst. For this task: '{task description}'. Determine: What is the user trying to achieve? What are the acceptance criteria? What does 'done' look like? What are the explicit and implicit requirements? Search the codebase for related features, read relevant docs, and report your findings."

   - **Architect Agent** (general-purpose): "You are a technical architect. For this task: '{task description}'. Determine: What services/modules are affected? What's the best technical approach? What patterns does this codebase already use for similar problems? What are the technical risks? Search the codebase for existing patterns, read service configs, and report your findings."

   - **Domain Expert Agent** (general-purpose, Complex tasks only): "You are a domain expert. For this task: '{task description}'. Determine: What business rules apply? What edge cases matter based on the domain? What has gone wrong in this area before? Check git history for related bugs/fixes, read domain-specific code, and report your findings."

   Synthesize findings from all research agents into a unified picture.

2. **Report findings to the user:**
   - Root cause hypothesis (or multiple candidates)
   - Which services need changes
   - Proposed approach
   - Research agent findings summary
   - Ask: "Does this look right? Should I proceed?"

3. **Wait for user confirmation** before moving to Step 5. The user may have additional context.

Note: Time is already being tracked from Step 2. Research is billable work.

## STEP 4.5: PRFAQ exercise (Complex tasks or --prfaq flag only)

If the task is classified as **Complex** or the user passed `--prfaq`:
1. Invoke the `/prfaq` skill if it exists
2. The PRFAQ output provides customer-first context for the brainstorming phase
3. If `/prfaq` skill doesn't exist, ask the 3 lightweight questions inline:
   - Who benefits from this change?
   - What changes for them?
   - Why now?

Skip this step for Trivial, Simple, and Standard tasks (unless --prfaq flag).

## STEP 4.7: Create artifact directory

If this task has a task ID (from Tasker or TaskCreate):
1. Create `_architecture/artifacts/{task-id}/` directory (may already exist from Step 3.5)
2. If research was done (Step 4), save research findings as `_architecture/artifacts/{task-id}/research.md`
3. If PRFAQ was done (Step 4.5), it will have saved its own artifact

This directory will be used by subsequent phases (brainstorming, planning, review-cycle, retro) to build the artifact chain.

## STEP 5: Create branch and team

1. Derive a branch slug from the task title
2. Create or checkout the feature branch from `main`:
   - If branch exists: checkout and pull
   - If new: `git checkout -b feature/{slug} main` or `git checkout -b fix/{slug} main`
3. Call TeamCreate with a team name derived from the project directory name (or task slug). Use agent_type "architect".

## STEP 6: Spawn teammates and create tasks

1. **Load discovered agents:** Read all agent definitions from `_architecture/agents/*.md` (skip `.fingerprint`)
2. **Select relevant agents:** Based on the task's affected services/directories, select the agents whose `service:` field matches
3. **Spawn teammates:** For each selected agent:
   - Use `general-purpose` subagent_type
   - Set `name` to the agent's `name` field (e.g., "pylon-dev")
   - Include the full agent definition content in the prompt as context
   - Set `team_name` to the current team
4. **Create tasks:** Using TaskCreate, create tasks with clear descriptions, definitions of done, dependencies, and owners assigned to the spawned agents

If no discovered agents exist (discovery was skipped or failed), fall back to spawning generic `general-purpose` agents with service-specific context from CLAUDE.md.

For frontend tasks that affect visible UI, include in the description:
- "You MUST self-verify with Playwright MCP after implementation"
- "You MUST use the frontend-design skill for any new components or redesigns"

## STEP 7: Confirm to the user

Report: task being worked on, feature branch, team composition, task assignments.

## PRE-IMPLEMENTATION: Readiness Gate (Standard and Complex tasks only)

After brainstorming and planning are complete, but BEFORE spawning implementation teammates:

1. If task complexity is **Standard** or **Complex**: invoke `/ready-check` skill
2. If verdict is **PASS**: proceed to implementation
3. If verdict is **CONCERNS**: present concerns to user, ask whether to proceed or address first
4. If verdict is **FAIL**: stop. Fix the planning gaps before implementation begins.

Skip this gate for Trivial and Simple tasks.

## POST-IMPLEMENTATION — MANDATORY REVIEW GATE

When all implementation tasks are complete:
1. Commit and push all work to the feature branch

2. **Run review based on complexity (DO NOT SKIP):**
   - **Trivial**: no review required
   - **Simple**: spawn a code reviewer agent, wait for report, save as `_architecture/artifacts/{task-id}/review-report-iteration-1.md`
   - **Standard/Complex**: invoke `/team-review-cycle` — full 6-agent review with fix loops

   The enforcement hook WILL BLOCK `gh pr create` if no review report exists for Standard/Complex tasks.
   DO NOT rationalize skipping review ("it's just tests", "it's obvious", "it's a small change").

3. When review passes, create PR from feature branch -> main
4. Update tasker task with PR link and mark as done
5. **If task was Complex or 3+ tasks were completed:** prompt user for retrospective — "Significant work completed. Run `/retro` to capture lessons learned?"
6. Report to user for manual verification
