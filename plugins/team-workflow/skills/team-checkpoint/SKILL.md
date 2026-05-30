---
name: team-checkpoint
description: Save all progress mid-session without stopping — commit, upsert the GitHub handoff comment + refresh the epic active-sessions block, continue working. No tasker, no time tracking.
---

Execute all of the following steps immediately. Do NOT stop the team — this is a save point, not a shutdown. Tasker is RETIRED — do not call any `mcp__tasker__*` tool. There is no time tracking.

**Canonical IDs:** repo `Innovation-Philosophy/Lisi-Core`, project `5` / `PVT_kwDOCa5KQM4BZN-K`, Status In Progress option `47fc9ee4` on field `PVTSSF_lADOCa5KQM4BZN-KzhUOPWE`.

STEP 1: Send to all active teammates: "Checkpoint now. Commit all current work, update PROGRESS.md with exact current state, run tests, message me when done."

STEP 2: Wait for all teammates to confirm.

STEP 3: Update `_architecture/PLATFORM-STATE.md` with current state from teammate reports.

STEP 4: **Upsert the handoff comment** on the focused issue (and any other touched issue) — identical mechanism and section list to `/team-stop` STEP 4a/4b (sentinel `<!-- handoff -->`; PATCH the existing handoff comment if one exists, else `gh issue comment`). Use the live worktree/branch/tip/PR state.

STEP 5: **Keep the issue In Progress** — do NOT close it or set Done. Ensure its board Status is In Progress (option `47fc9ee4`) and refresh the parent epic's `active-sessions` block (regenerate from the board between the `<!-- active-sessions -->` sentinels via `gh issue edit <epic#> --body-file`) so it still lists this issue with its branch + last-touched.

STEP 6: Resume normal operation. Assign next tasks if teammates are idle.
