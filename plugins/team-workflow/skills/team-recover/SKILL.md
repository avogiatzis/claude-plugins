---
name: team-recover
description: Recover from a session that ended without a clean /team-stop — reconstruct from GitHub In-Progress issues, handoff comments, and git state.
---

Execute all of the following steps immediately. The previous session ended without a clean `/team-stop`. Reconstruct the state from GitHub + git. Tasker is RETIRED — do not call any `mcp__tasker__*` tool. (`_architecture/NEXT-SESSION.md` is also retired — do not look for it.)

STEP 1: List in-flight work from the board:
```
gh project item-list 5 --owner Innovation-Philosophy --format json --limit 400
```
Collect every issue with Status == "In Progress" — these are the sessions that were active when things died.

STEP 2: For each In-Progress issue `<n>`:
- Read its latest handoff comment: `gh issue view <n> --repo Innovation-Philosophy/Lisi-Core --json comments --jq '[.comments[]|select(.body|test("<!-- handoff -->"))]|last'`. This carries branch, tip SHA, next action, blockers, and files-touching.
- Locate its worktree: `git worktree list` → match the worktree whose branch contains `<n>-`.

STEP 3: Read `_architecture/PLATFORM-STATE.md`.

STEP 4: For each located worktree, check git reality:
- `git -C <worktree> log --oneline -5` — what was committed
- `git -C <worktree> status` — uncommitted work
- `git -C <worktree> stash list` — stashed work
Also read any `PROGRESS.md` in the affected service directories.

STEP 5: Reconcile. Compare the handoff comment's "Next action" + "Tip SHA" against the worktree's actual git state. If there's uncommitted work the handoff implies was done, the session died between commit and handoff — the work is likely present in the worktree.

STEP 6: Update `_architecture/PLATFORM-STATE.md` with the reconciled state.

STEP 7: Report to the user:
- Which In-Progress issues were recovered (number + title + epic) and their worktrees.
- Any work that may be lost (uncommitted/stashed changes).
- Recommended next action per issue (from its handoff comment).
- Which teammates to spawn to continue.
