# Team Workflow Plugin

Team-based development workflow plugin for Claude Code with intelligent project discovery, session management, agent teams, tasker integration, and review enforcement.

## Installation

```bash
claude plugin install /path/to/team-workflow
```

## Skills

| Skill | Description |
|-------|-------------|
| **team-start** | Start a team session -- auto-pick next task, create feature branch, spawn teammates, implement, then review cycle. |
| **team-stop** | Gracefully end a team session -- checkpoint all work, update state, write handoff, shutdown team. |
| **team-checkpoint** | Save all progress mid-session without stopping -- commit, update state, continue working. |
| **team-recover** | Recover from a session that ended without a clean stop -- audit state and reconstruct. |
| **team-review** | Run an architectural review across all active services using the reviewer agent. |
| **team-review-cycle** | Run a multi-agent review cycle on the current feature branch -- code review, tests, domain validation, then fix loop. |
| **team-status** | Report the current status of all services and active work. |
| **team-discover** | Scan any project, research framework best practices, and generate domain expert agent definitions automatically. |
| **prfaq** | Run a Working Backwards exercise (Press Release + FAQ) to clarify user value before technical design. |
| **distill** | Compress large documents into LLM-optimized summaries preserving decisions, constraints, and rules. |
| **ready-check** | Validate implementation readiness before execution -- checks spec completeness, architecture decisions, dependencies, test strategy. |
| **retro** | Run a post-epic retrospective -- analyzes git history, review reports, and time tracking to generate lessons learned. |

## Hooks

A **PreToolUse** hook gates PR creation based on task complexity and whether a review was run. If a task is non-trivial and `team-review` or `team-review-cycle` has not been executed in the current session, the hook blocks `gh pr create` and prompts the user to run a review first. This enforces review discipline without requiring manual checklists.

## Discovery

The `/team-discover` skill scans any project to understand its structure, tech stack, and conventions. It researches framework best practices using available documentation tools, then generates domain expert agent definitions tailored to the project. This means you can point the plugin at a new codebase and get purpose-built agents without manual configuration.

## Credits

Inspired by [BMAD-METHOD](https://github.com/bmadcode/BMAD-METHOD).
