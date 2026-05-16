---
name: team-checkpoint
description: Save all progress mid-session without stopping — commit, update HANDOFF.md + tasker session fields, continue working.
---

Execute all of the following steps immediately. Do not stop the team — this is a save point, not a shutdown.

STEP 1: Send a message to all active teammates:
"Checkpoint now. Commit all current work, update PROGRESS.md with exact current state, run tests, message me when done."

STEP 2: Wait for all teammates to confirm.

STEP 3: Update `_architecture/PLATFORM-STATE.md` with current state from teammate reports.

STEP 4: Write/update `_architecture/artifacts/<id>/HANDOFF.md` for the focused tasker, using the same 10 required sections defined in `/team-stop` STEP 4b.

STEP 5: Call `mcp__tasker__tasker_update` setting: `handoffPath: "_architecture/artifacts/<id>/HANDOFF.md"`, `lastSessionEndedAt: <ISO now>`, `lastSessionTerminal: <basename of $PWD>`.

STEP 6: Resume normal operation. Assign next tasks if teammates are idle. AI time tracking continues — do NOT call `tasker_ai_stop`.
