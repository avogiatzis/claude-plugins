---
name: team-stop
description: Gracefully end a team session — checkpoint all work, update state, write handoff, shutdown team, close tasker tracking.
---

Execute all of the following steps immediately. Do not ask for confirmation.

STEP 1: Send a checkpoint message to all active teammates:
"Checkpoint now — session ending. Commit all work, update PROGRESS.md, run tests, message me when done."

STEP 2: Wait for all teammates to confirm checkpoint complete.

STEP 3: Update `_architecture/PLATFORM-STATE.md` with the final state of all services, incorporating teammate reports.

STEP 4: Write `_architecture/NEXT-SESSION.md` with:
- What was accomplished this session (list every completed task)
- What's in progress (check each service's PROGRESS.md)
- What to do next (ordered priority list)
- Any blockers or pending decisions
- Which teammates to spawn next session

STEP 5: Prompt for retrospective (if significant work was done)

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

STEP 8: Stop AI time tracking and update tasker tasks (if tasker MCP is available).

**Close ALL active AI sessions (not just the current one).** `tasker_ai_stop` closes one active session per call (FIFO — oldest first), so sessions left open by prior mid-session `team-start` or orchestration glitches accumulate bogus duration otherwise. Follow this loop:

1. Call `mcp__tasker__tasker_time_list` and count entries where the `end` field is absent — these are active sessions.
2. For each active session (cap the loop at 10 iterations as a safety bound), call `mcp__tasker__tasker_ai_stop` with a description that includes: the current task's outcome summary, plus — if the entry's `taskId` differs from the current task — a note that it is being force-closed because it was a leaked prior session.
3. After the loop, call `mcp__tasker__tasker_time_list` once more and verify no entries are missing `end`. If any remain, report the leak to the user explicitly.
4. Report the total count of sessions closed (usually 1; more indicates prior leakage that has now been cleaned up).

Then update task state:
- For any tasker tasks worked on this session:
  - If completed: mark as `done` via `mcp__tasker__tasker_update` with notes summarizing what was done (PR link, key changes, review status)
  - If still in progress: update notes with current status and what remains
- Call `mcp__tasker__tasker_hours` with period "today" to show time tracked
- Call `mcp__tasker__tasker_list` to show the current state of all tasks to the user

STEP 9: Confirm to the user that the session is cleanly closed and all state is saved. Include:
- Tasks completed this session
- Time tracked this session (from tasker_hours)
- Current tasker task board summary
- Branch/PR status
