---
name: team-recover
description: Recover from a session that ended without a clean /team-stop — audit state and reconstruct.
---

Execute all of the following steps immediately. The previous session ended without a clean /team-stop. Reconstruct the state.

STEP 1: Read `_architecture/NEXT-SESSION.md` (may be stale — treat as starting point, not truth).

STEP 2: Read `_architecture/PLATFORM-STATE.md`.

STEP 3: For each service directory that has a `PROGRESS.md`, read it. These are the most recent source of truth since agents update them on every task completion.

STEP 4: For each service directory, check git status and recent commits:
- `git log --oneline -5` — what was committed
- `git status` — any uncommitted work
- `git stash list` — any stashed work

STEP 5: Reconcile: Compare PROGRESS.md state with git state. If there's uncommitted work that PROGRESS.md says was completed, the session died between commit and PROGRESS.md update — the work is likely complete.

STEP 6: Update `_architecture/PLATFORM-STATE.md` with reconciled state.

STEP 7: Write a fresh `_architecture/NEXT-SESSION.md` based on the reconciled state.

STEP 8: Report to the user:
- What state was recovered
- Any work that may have been lost (uncommitted changes)
- Recommended next steps
- Which teammates to spawn to continue
