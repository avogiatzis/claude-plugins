---
name: team-review
description: Run an architectural review across all active services using the reviewer agent.
---

Execute immediately:

1. Read `_architecture/PLATFORM-STATE.md` to identify active services
2. Read `_architecture/CROSS-CUTTING-DECISIONS.md` for the rules to review against

3. For each active service, spawn a reviewer agent (or a single reviewer for all):
   - Use the `reviewer` agent type
   - Set `team_name` to the project team name
   - Instruct them to review recent changes (git diff since last checkpoint or last 5 commits)
   - Review against: cross-cutting decisions, service boundaries, security, test coverage

4. Collect review results from the reviewer agent(s)

5. Report findings to the user:
   - CRITICAL issues (must fix)
   - WARNINGS (should fix)
   - Suggestions
   - Overall code health assessment
