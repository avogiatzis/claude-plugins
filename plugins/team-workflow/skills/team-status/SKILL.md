---
name: team-status
description: Report the current status of all services and active work from GitHub Project #5 + git.
---

Execute immediately. Tasker is RETIRED — do not call any `mcp__tasker__*` tool. (`_architecture/NEXT-SESSION.md` is retired — do not look for it.)

1. Read `_architecture/PLATFORM-STATE.md`.
2. Query the board, grouped by Status:
   ```
   gh project item-list 5 --owner Innovation-Philosophy --format json --limit 400
   ```
   Summarise counts per Status (Todo / In Progress / Done) and list the **In Progress** issues with their number, title, and parent epic.
3. For each In-Progress issue, surface its working branch/PR (from its `<!-- handoff -->` comment, if any) and its worktree via `git worktree list`.
4. Check in-session `TaskList` for the current team's task assignments.
5. For each service directory with a `PROGRESS.md`, read it.

Report to the user:
- Overall platform status (from PLATFORM-STATE.md).
- Active teammates and what they're working on.
- Board summary: Todo / In Progress / Done counts, with the In-Progress issues enumerated (number, title, epic, branch/PR).
- Any unresolved escalations (`_architecture/escalations/`).
- Test counts across services (if available from PROGRESS.md).
