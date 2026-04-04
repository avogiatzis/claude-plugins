---
name: team-discover
description: Scan project structure, research framework best practices, and generate domain expert agent definitions. Run automatically by /team-start when agents are missing or stale, or manually to regenerate.
argument-hint: [--force]
---

Execute all of the following steps immediately. Do not ask for confirmation.

## PURPOSE

Analyzes any project's structure, tech stack, and conventions, then researches current framework best practices and generates rich domain expert agent definitions. These agents are used by `/team-start` when spawning teammates.

## STEP 1: Check Staleness

If `--force` was passed, skip to Step 2.

If `_architecture/agents/.fingerprint` exists:
1. Compute a fresh fingerprint:
   - Use Glob to find all service indicator files: `**/*.csproj`, `**/package.json`, `**/go.mod`, `**/Cargo.toml`, `**/pom.xml`, `**/pyproject.toml`
   - Exclude: `node_modules`, `bin`, `obj`, `.git`, `dist`, `build`, `vendor`, `_architecture`, `*.Test*`, `*.test*`
   - Sort the file paths alphabetically
   - Create a string of all sorted paths joined by newlines
2. Read the stored fingerprint from `_architecture/agents/.fingerprint`
3. Compare the path list (ignore the metadata lines starting with `#`)
4. If they match: report "Agent definitions are up to date. Use `/team-discover --force` to regenerate." and stop.
5. If they differ: proceed to Step 2.

If no `.fingerprint` exists: proceed to Step 2.

## STEP 2: Structure Scan

Discover all services/modules in the project:

1. Use Glob to find service indicator files:
   - `**/*.csproj` (exclude `*Test*`, `*test*`)
   - `**/package.json` (exclude `node_modules`)
   - `**/go.mod`
   - `**/Cargo.toml`
   - `**/pom.xml`
   - `**/pyproject.toml`
   - `**/requirements.txt`

2. Group by service boundary:
   - Each directory containing a service indicator is a potential "service"
   - For monorepos: look for common parent patterns (`src/`, `services/`, `packages/`, `apps/`)
   - Collapse test projects — they belong to their parent service, not standalone
   - Collapse shared/common libraries — note them as shared dependencies

3. For each discovered service, record:
   - **Name**: derived from directory name or project file
   - **Path**: relative path from project root
   - **Type**: backend, frontend, library, orchestration, utility
   - **Indicator file**: which file identified it

4. Report discovered services to the user in a table:
   ```
   | # | Service | Path | Type | Stack Indicator |
   ```

## STEP 3: Code Analysis (per service)

For each discovered service, gather deep context:

### 3a. Project-Level Context
- Read root `CLAUDE.md` if it exists — extract cross-cutting rules, patterns, conventions
- Read `_architecture/CROSS-CUTTING-DECISIONS.md` if it exists — extract platform-wide rules
- Read `_architecture/PLATFORM-STATE.md` if it exists — extract current service states

### 3b. Service-Level Analysis
For each service:

**Read the service's CLAUDE.md** (if exists at `{service-path}/CLAUDE.md` or `{service-path}/.claude/CLAUDE.md`)

**Extract tech stack from project file:**

For .NET (`.csproj`):
- Target framework (net9.0, net10.0, etc.)
- Key NuGet packages — look for: FastEndpoints, MediatR, Mediator.SourceGenerator, EntityFrameworkCore, FluentResults, Semantic Kernel, Blazor, Syncfusion, MongoDB.Driver, Aspire
- SDK type (web, worker, classlib, blazorserver)

For Node (`package.json`):
- Node/npm version constraints
- Key dependencies — look for: react, next, vue, angular, express, fastify, prisma, drizzle, tailwind, tanstack-query, zustand, redux
- Dev dependencies — look for: jest, vitest, playwright, cypress, eslint, prettier, typescript

For Go (`go.mod`):
- Go version
- Key dependencies — look for: gin, echo, fiber, gorm, sqlx, cobra, viper

For Python (`pyproject.toml` / `requirements.txt`):
- Python version
- Key dependencies — look for: django, flask, fastapi, sqlalchemy, pydantic, pytest, celery

For Rust (`Cargo.toml`):
- Rust edition
- Key dependencies — look for: actix-web, axum, tokio, serde, diesel, sqlx

**Scan for architectural patterns:**
- Check for test directories and frameworks
- Look for migration directories (database)
- Check for Docker/container files
- Look for CI/CD configuration

**Read recent git history:**
```bash
git log --oneline -20 -- {service-path}
```

## STEP 4: Best Practices Research (parallel)

For each discovered service, spawn a research agent in the background using the Agent tool:

**Agent prompt template:**
```
You are a framework research specialist. Research current best practices for this technology stack:

Stack: {list of frameworks with versions}

For EACH major framework/library in the stack, use the context7 MCP tools:
1. Call resolve-library-id with the library name to find the correct documentation ID
2. Call query-docs with that ID to fetch current best practices, patterns, and pitfalls

If context7 doesn't have a library, use WebSearch as fallback.

For each framework, report:
### {Framework} {version}
- **Recommended patterns**: current idiomatic usage patterns
- **Common pitfalls**: mistakes to avoid, deprecated patterns
- **Performance guidance**: key performance considerations
- **Testing best practices**: how to test effectively with this framework
- **Security considerations**: common security issues and mitigations

Keep each framework section focused and actionable — 10-20 bullet points max.
Report findings as structured markdown.
```

Spawn all research agents in parallel (one per service). Wait for all to complete.

## STEP 5: Generate Agent Definitions

Create `_architecture/agents/` directory if it doesn't exist.

For each discovered service, write `_architecture/agents/{service-slug}.md`:

```markdown
---
name: {service-slug}-dev
service: {relative-service-path}
stack: {one-line stack summary, e.g. ".NET 10, FastEndpoints, EF Core 10, SQL Server"}
generated: {ISO date}
---

You are a developer for the {service-name} service located at `{service-path}`.

## Tech Stack
{bullet list of detected frameworks with versions}

## Project Conventions (AUTHORITATIVE — always follow these)
{conventions extracted from CLAUDE.md and CROSS-CUTTING-DECISIONS.md}
{include: response patterns, DB access rules, import rules, naming conventions}
{these OVERRIDE any general best practices below — if there's a conflict, follow these}

## Framework Best Practices (apply unless project conventions say otherwise)
{content from research agents, organized by framework}

### {Framework 1} {version}
{researched best practices}

### {Framework 2} {version}
{researched best practices}

## Known Gotchas
{project-specific gotchas from CLAUDE.md}
{anti-patterns identified from CROSS-CUTTING-DECISIONS.md}
{any patterns from git history that suggest recurring issues}

## Test Infrastructure
- **Framework**: {detected test framework}
- **Test directory**: {path to tests}
- **Run command**: {e.g., "dotnet test", "npm test", "go test ./..."}
- **Conventions**: {any test patterns from CLAUDE.md or detected from code}
```

## STEP 6: Write Fingerprint and Report

Write `_architecture/agents/.fingerprint`:
```
# Team Workflow Agent Fingerprint
# Generated: {ISO date}
# Services: {count}
{sorted list of service indicator file paths, one per line}
```

Report to the user:
- Number of services discovered
- Table of generated agents with their stack summaries
- Frameworks researched
- Remind: "These agents will be used automatically by `/team-start` when spawning teammates. Run `/team-discover --force` to regenerate."
