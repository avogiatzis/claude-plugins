---
name: team-stop
description: Gracefully end a team session — upsert per-issue handoff comments on GitHub, set Done/close on finished issues, shutdown team. No tasker, no time tracking.
---

Execute all of the following steps immediately. Do not ask for confirmation.

**Tracking is on GitHub** (Issues + Project #5). Tasker is RETIRED — do not call any `mcp__tasker__*` tool. There is no time tracking.

**Canonical IDs:** repo `Innovation-Philosophy/Lisi-Core`, project `5` / `PVT_kwDOCa5KQM4BZN-K`, Status field `PVTSSF_lADOCa5KQM4BZN-KzhUOPWE` → Todo `f75ad846`, In Progress `47fc9ee4`, Done `98236657`.

## STEP 1: Checkpoint all teammates

Send to all active teammates: "Checkpoint now — session ending. Commit all work, update PROGRESS.md, run tests, message me when done."

## STEP 2: Wait for all teammates to confirm checkpoint complete.

## STEP 3: Update `_architecture/PLATFORM-STATE.md` with the final state of all services, incorporating teammate reports.

## STEP 4: Upsert a handoff comment per touched issue

For each GitHub issue worked on this session (the focused one + any others touched):

4a. Build the handoff body. First line MUST be the sentinel `<!-- handoff -->`. Then these sections, in order:
- `**Last session ended:** <ISO timestamp with timezone> (terminal: <basename of $PWD>)`
- `**Worktree:** <absolute path>` _(informational only — resume re-derives via `git worktree list`)_
- `**Branch:** <branch> (tracking <upstream> @ <upstream sha>)`
- `**Tip:** <local HEAD sha> — clean | uncommitted: <NONE | count>`
- `**PR:** <PR# (state)> — <URL>` (or `none`)
- `## Next action (one imperative sentence)`
- `## Blockers / decisions awaiting human` — bullets, or `- none`
- `## Files I'm touching (sibling-contention guard)` — file paths relative to the sub-repo / workspace root
- `## Pre-flight sanity (run before resuming)` — fenced bash block
- `## Anchors (don't re-narrate — just open)` — file paths
- `## NOT for this session` — bullets, or `- (none)`

4b. Upsert (never blindly append a duplicate). Find an existing handoff comment's URL:
```
gh issue view <n> --repo Innovation-Philosophy/Lisi-Core --json comments \
  --jq '[.comments[] | select(.body|test("<!-- handoff -->"))] | last.url'
```
- If a URL is returned: extract the numeric comment id from its `#issuecomment-<id>` suffix, then
  `gh api --method PATCH repos/Innovation-Philosophy/Lisi-Core/issues/comments/<id> --field body=@<bodyfile>`.
  **The REST comments endpoint needs the numeric database id** — the `.id` field from `--json comments` is a GraphQL node id (`IC_…`) and will 404 against REST.
- Else (no URL): `gh issue comment <n> --repo Innovation-Philosophy/Lisi-Core --body-file <bodyfile>`

4c. **If the task is DONE** (PR merged or work complete):
- Set board Status → Done: `gh project item-edit --project-id PVT_kwDOCa5KQM4BZN-K --id <itemId> --field-id PVTSSF_lADOCa5KQM4BZN-KzhUOPWE --single-select-option-id 98236657`
- Close the issue: `gh issue close <n> --repo Innovation-Philosophy/Lisi-Core` (a merged PR with `Closes #<n>` may already have).
- Remove it from the parent epic's `active-sessions` block (regenerate the block from the board between the `<!-- active-sessions -->` sentinels via `gh issue edit <epic#> --body-file`).

4d. **Otherwise (in-progress/paused):** leave Status = In Progress and refresh the parent epic's `active-sessions` block so it still lists this issue with its branch + last-touched.

## STEP 5: Prompt for retrospective (if significant work was done)

Significant = 3+ issues closed this session, OR the task was Complex, OR the review cycle took 2+ iterations. If significant: ask "Significant work completed. Run `/retro`? (y/n)". On yes, invoke `/retro` before shutdown.

## STEP 6: Send shutdown requests to all teammates. Wait for shutdown confirmations.

## STEP 7: Call `TeamDelete` to clean up the team.

## STEP 8: Confirm clean closure to the user

Report (all from GitHub — no tasker, no time tracked):
- Issues closed this session: `gh issue list --repo Innovation-Philosophy/Lisi-Core --state closed --search "closed:>=<session-start-date>" --json number,title`
- Current board summary: `gh project item-list 5 --owner Innovation-Philosophy --format json` grouped by Status.
- Branch / PR status for the work.
- Links to the handoff comment(s) written this session.
