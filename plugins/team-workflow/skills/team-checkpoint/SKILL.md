---
name: team-checkpoint
description: Save all progress mid-session without stopping — commit, update state, continue working.
---

Execute all of the following steps immediately. Do not stop the team — this is a save point, not a shutdown.

STEP 1: Send a message to all active teammates:
"Checkpoint now. Commit all current work, update PROGRESS.md with exact current state, run tests, message me when done."

STEP 2: Wait for all teammates to confirm.

STEP 3: Update `_architecture/PLATFORM-STATE.md` with current state from teammate reports.

STEP 4: Append a checkpoint entry to `_architecture/NEXT-SESSION.md`:
```
### Checkpoint — [timestamp]
- [service]: [current state]
```

STEP 5: Resume normal operation. Assign next tasks if teammates are idle.
