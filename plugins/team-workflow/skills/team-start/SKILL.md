---
name: team-start
description: Start a team session — resolve work from GitHub epics/issues (team-start <epic> | <issue#>), research if vague, create feature branch, spawn teammates, implement, then review cycle. Execute immediately without asking.
---

Execute all of the following steps immediately. Do not ask for confirmation. Do not describe what you will do. Just do it.

**Tracking is on GitHub.** Work items are GitHub Issues in `Innovation-Philosophy/Lisi-Core`, organised on Project #5 (https://github.com/orgs/Innovation-Philosophy/projects/5). Epics are tracking issues (label `epic`) with their tasks attached as native sub-issues. Tasker is RETIRED — do not call any `mcp__tasker__*` tool.

**Canonical IDs (Project #5):**
- Project number `5`, owner `Innovation-Philosophy`, repo `Innovation-Philosophy/Lisi-Core`, project-id `PVT_kwDOCa5KQM4BZN-K`
- Status field `PVTSSF_lADOCa5KQM4BZN-KzhUOPWE` → Todo `f75ad846`, In Progress `47fc9ee4`, Done `98236657`
- Priority field `PVTSSF_lADOCa5KQM4BZN-KzhUOQJ8`

## STEP 1: Bootstrap project structure (if needed)

Check if `_architecture/` exists in the project root. If it does NOT exist, this is a brand new project — create the scaffolding:

1a. Create `_architecture/` with `PLATFORM-STATE.md` (header + "No services yet") and `CROSS-CUTTING-DECISIONS.md` (header + "No decisions yet").

1b. Create `_architecture/decisions/`, `_architecture/escalations/`, and `_architecture/propagations/` directories.

## STEP 1.5: Discovery gate

Check if `_architecture/agents/` exists and contains agent definition files:
- If missing or empty: invoke `/team-discover` to scan the project and generate domain expert agent definitions.
- If exists: check `_architecture/agents/.fingerprint` for staleness via two independent conditions — trigger `/team-discover --force` if either fires:
  1. **Structural staleness:** recompute the directory-structure hash; if it differs from the stored one, regenerate.
  2. **Temporal staleness:** if `.fingerprint` mtime is more than 3 days old, regenerate.
  - If neither fires: proceed with cached agents.

If `/team-discover` is not available, skip and proceed with manual agent selection.

## STEP 2: Resolve the work item (GitHub)

Argument forms: `team-start <epic>` (epic name fuzzy / `#num`), `team-start <issue#>`, or none.

- **`<issue#>` or bare number:** target that issue directly. Go to STEP 2.5.
- **`<epic>`:** resolve to the epic tracking issue:
  ```
  gh issue list --repo Innovation-Philosophy/Lisi-Core --label epic --state open --json number,title
  ```
  Fuzzy-match the argument against titles; on ambiguity, list matches and ask which.
  Then read the epic's in-flight work — live-query its open sub-issues and cross-reference board Status:
  ```
  gh api repos/Innovation-Philosophy/Lisi-Core/issues/<epic#>/sub_issues --jq '.[] | {number,title,state}'
  ```
  - **If any sub-issue is In Progress:** LEAD WITH it — "In-progress: #N (last touched <age from its handoff comment>) — resume?". On yes, treat #N as the target and go to STEP 2.5 (resume path).
  - **Else build the shortlist:** open sub-issues, ordered by Priority (high>medium>low) then issue number; take the top 5; present them; the user picks. Auto-pick only if exactly one candidate.
- **none:** list epics with their open sub-issue counts and ask which to work on.

Once a target issue `<n>` is chosen, set NOTHING yet (claim happens in STEP 2.8 after the worktree exists).

## STEP 2.5: Resume-vs-new worktree (derived state)

Branch convention: `feature/<n>-<slug>` (or `fix/<n>-<slug>`), slug derived from the issue title.

- Look for an existing worktree on that branch: `git worktree list` → match a worktree whose checked-out branch contains `<n>-`.
- If found AND the issue Status == In Progress:
  - AskUserQuestion: "Issue #<n> has an active worktree at `<path>` on `<branch>`. Resume / new-worktree / abort?"
  - **Resume:** `cd <path>`; go to STEP 2.6.
  - **New-worktree:** create a fresh worktree (`superpowers:using-git-worktrees`) on a new branch off `main`; go to STEP 2.8.
  - **Abort:** stop the skill cleanly.
- Else (no worktree): create a worktree via `superpowers:using-git-worktrees` on `feature/<n>-<slug>` (or `fix/`) off `main`.

## STEP 2.6: Stale-handoff guard

Read the latest handoff comment on the issue (sentinel `<!-- handoff -->`):
```
gh issue view <n> --repo Innovation-Philosophy/Lisi-Core --json comments \
  --jq '[.comments[] | select(.body|test("<!-- handoff -->"))] | last'
```
If its `updatedAt` is more than 7 days ago, print the whole comment and AskUserQuestion "Handoff is <N> days old. Proceed?". On "no", abort cleanly. If within 7 days (or none exists), read it silently into context.

## STEP 2.7: Sibling-worktree contention scan

```
gh project item-list 5 --owner Innovation-Philosophy --format json --limit 400
```
For each item with Status == "In Progress" and number != `<n>`: read its handoff comment, extract the bullets under `## Files I'm touching`. Compare the union to this session's planned scope (from the issue body, a linked plan, or — if neither — ask the user "What files do you expect to touch?"). On overlap, print one warning block per colliding sibling, then AskUserQuestion "Sibling contention detected. Proceed / coordinate-first / abort?". On "abort", stop cleanly.

## STEP 2.8: Claim the issue

1. Set the issue's board Status → **In Progress**: find its project item id, then
   ```
   gh project item-edit --project-id PVT_kwDOCa5KQM4BZN-K --id <itemId> \
     --field-id PVTSSF_lADOCa5KQM4BZN-KzhUOPWE --single-select-option-id 47fc9ee4
   ```
2. Refresh the parent epic's `active-sessions` block: regenerate it from the board (list the epic's In-Progress sub-issues with their branch + last-touched), rewriting the text between `<!-- active-sessions -->` and `<!-- /active-sessions -->` via `gh issue edit <epic#> --body-file <file>`.

There is NO time tracking — do not start any timer.

## STEP 3: Read state files

Read now: `_architecture/PLATFORM-STATE.md`, `_architecture/CROSS-CUTTING-DECISIONS.md`, all files in `_architecture/escalations/`, and any existing plan documents (check `docs/` or project root, and `_architecture/artifacts/<n>/`).

## STEP 3.5: Classify task complexity (Scale-Adaptive Ceremony)

Check for overrides: `--quick` → Trivial; `--full` → Complex; `--level {trivial|simple|standard|complex}` → that level.

If no override, auto-classify (Trivial: typo/bump/config, 1 file; Simple: small bug/feature, 1 service; Standard: feature/integration/refactor, 1-2 services; Complex: migrate/new-service/multi-service/architecture, 3+ services).

Persist the level: create `_architecture/artifacts/<n>/` and write `complexity.txt` containing just `trivial|simple|standard|complex`.

Workflow per level:
- **Trivial** — direct fix → commit → push → done. Skip to POST-IMPLEMENTATION.
- **Simple** — light brainstorming; 1 teammate max; review = code reviewer only.
- **Standard** — full brainstorming → plan → team → review-cycle (all 6 agents).
- **Complex** — `/prfaq` first → full brainstorming → plan → `/ready-check` → team → review-cycle → `/retro` after.

Report: "Task classified as **{level}**. Adjusting workflow accordingly."

## STEP 4: Research phase (if the issue is vague)

A task is clear enough if you know the affected service(s), expected behavior, and where to look. If it's just a symptom/one-liner, it's vague.

If vague: investigate with Explore agents (Trivial/Simple) or named research agents in parallel (Standard/Complex) — Analyst, Architect, and (Complex only) Domain Expert. Synthesize findings, report to the user (root-cause hypothesis, affected services, proposed approach), and ask "Does this look right? Should I proceed?" before STEP 5. Save findings to `_architecture/artifacts/<n>/research.md`.

## STEP 4.5: PRFAQ exercise (Complex or `--prfaq` only)

Invoke `/prfaq` if it exists; otherwise ask inline: who benefits, what changes, why now. Skip for Trivial/Simple/Standard unless `--prfaq`.

## STEP 5: Create branch and team

1. The feature branch was created in STEP 2.5 (`feature/<n>-<slug>` off `main`). If resuming, it already exists — checkout and pull.
2. Call `TeamCreate` with a team name derived from the issue slug. Use agent_type "architect".

## STEP 6: Spawn teammates and create tasks

1. Read all agent definitions from `_architecture/agents/*.md` (skip `.fingerprint`).
2. Select agents whose `service:` field matches the issue's affected services/directories.
3. Spawn each as a `general-purpose` teammate (`name` = the agent's name, full definition in the prompt, `team_name` = current team).
4. Create in-session tasks via TaskCreate with clear definitions of done, dependencies, and owners.

For frontend tasks affecting visible UI, include: "You MUST self-verify with Playwright MCP" and "You MUST use the frontend-design skill for new components/redesigns."

If no discovered agents exist, fall back to `general-purpose` agents with service context from CLAUDE.md.

## STEP 7: Confirm to the user

Report: the issue being worked (number + title + epic), feature branch, team composition, task assignments.

## PRE-IMPLEMENTATION: Readiness Gate (Standard and Complex only)

After brainstorming + planning, before spawning implementation teammates: invoke `/ready-check`. PASS → proceed. CONCERNS → present, ask whether to proceed or fix first. FAIL → stop and fix planning gaps. Skip for Trivial/Simple.

## POST-IMPLEMENTATION — MANDATORY REVIEW GATE

When all implementation tasks are complete:
1. Commit and push all work to the feature branch.
2. **Run review based on complexity (DO NOT SKIP):**
   - **Trivial**: no review.
   - **Simple**: spawn a code reviewer agent, save report to `_architecture/artifacts/<n>/review-report-iteration-1.md`.
   - **Standard/Complex**: invoke `/team-review-cycle` (full 6-agent review with fix loops).
   The enforcement hook WILL BLOCK `gh pr create` if no review report exists for Standard/Complex. Do not rationalize skipping.
3. When review passes, create the PR from the feature branch → main. Put `Closes #<n>` in the PR body so merge auto-closes the issue. Follow the Stakeholder Summary template in CLAUDE.md.
4. The issue's Status moves to Done at `/team-stop` (or when the PR merges and closes it).
5. If Complex or 3+ tasks completed: prompt "Significant work completed. Run `/retro`?".
6. Report to the user for manual verification.
