---
name: team-stop
description: Gracefully end a team session — write per-tasker HANDOFF.md, update tasker session fields, shutdown team, close tasker tracking.
---

Execute all of the following steps immediately. Do not ask for confirmation.

STEP 1: Send a checkpoint message to all active teammates:
"Checkpoint now — session ending. Commit all work, update PROGRESS.md, run tests, message me when done."

STEP 2: Wait for all teammates to confirm checkpoint complete.

STEP 3: Update `_architecture/PLATFORM-STATE.md` with the final state of all services, incorporating teammate reports.

STEP 4: Write per-tasker handoff.

For each tasker task that was worked on this session (the focused one + any others touched):

4a. If `tasker_get` shows `status === "done"`:
- Clear session fields via `mcp__tasker__tasker_update`: set `currentWorktree: null`, `currentBranch: null`, `handoffPath: null`, `lastSessionEndedAt: null`, `lastSessionTerminal: null`.
- Leave any existing `_architecture/artifacts/<id>/HANDOFF.md` in place as a historical record (it joins `retro.md` as a lifecycle artifact). Do NOT delete it.
- Skip to STEP 5.

4b. Otherwise (in-progress or paused):
- Write or update `_architecture/artifacts/<id>/HANDOFF.md` from session state. Required sections, in order:
  - Header: `# Handoff — Tasker #<id>: <title>`
  - `**Last session ended:** <ISO timestamp with timezone> (terminal: <basename of $PWD>)`
  - `**Worktree:** <absolute path>`
  - `**Branch:** <branch> (tracking <upstream> @ <upstream sha>)`
  - `**Tip:** <local HEAD sha> — clean | uncommitted: <NONE | <count>>`
  - `**PR:** <PR# (state)> — <URL>` (or `none`)
  - `## Next action (one imperative sentence)` — one literal imperative sentence
  - `## Blockers / decisions awaiting human` — bullet list, or `- none`
  - `## Files I'm touching (sibling-contention guard)` — bullet list of file paths relative to the sub-repo or workspace root
  - `## Pre-flight sanity (run before resuming)` — fenced bash block of commands
  - `## Anchors (don't re-narrate — just open)` — bullet list of file paths
  - `## NOT for this session` — bullet list, or `- (none)`
- Call `mcp__tasker__tasker_update` setting: `handoffPath: "_architecture/artifacts/<id>/HANDOFF.md"`, `lastSessionEndedAt: <ISO now>`, `lastSessionTerminal: <basename of $PWD>`. Do NOT touch `currentWorktree` or `currentBranch` here — those were set at `/team-start` and remain stable.

STEP 5: Prompt for retrospective (if significant work was done).

Check if significant work was completed this session:
- 3 or more tasks completed, OR
- Task was classified as Complex, OR
- Review cycle required 2+ iterations (indicating rework)

If significant work was done:
- Ask the user: "Significant work completed this session. Run `/retro` to capture lessons learned? (y/n)"
- If yes: invoke the `/retro` skill before proceeding to shutdown
- If no or user declines: proceed to shutdown

STEP 6: Send shutdown requests to all teammates. Wait for shutdown confirmations.

STEP 7: Call TeamDelete to clean up the team.

STEP 8: Close ALL active AI sessions (not just the current one).

`tasker_ai_stop` closes one active session per call (FIFO — oldest first), so sessions left open by prior mid-session `team-start` or orchestration glitches accumulate bogus duration otherwise:

1. Call `mcp__tasker__tasker_time_list` and count entries where the `end` field is absent — these are active sessions.
2. For each active session (cap loop at 10 iterations as a safety bound), call `mcp__tasker__tasker_ai_stop` with a description that includes the current task's outcome summary, plus — if the entry's `taskId` differs from the current task — a note that it is being force-closed because it was a leaked prior session.
3. After the loop, call `mcp__tasker__tasker_time_list` once more and verify no entries are missing `end`. If any remain, report the leak to the user explicitly.
4. Report the total count of sessions closed (usually 1; more indicates prior leakage that has now been cleaned up).

STEP 9: Confirm to the user that the session is cleanly closed. Include:
- Tasks completed this session
- Time tracked this session (from tasker_hours)
- Current tasker task board summary
- Branch/PR status
- HANDOFF.md path(s) written this session
