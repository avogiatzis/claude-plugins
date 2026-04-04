---
name: distill
description: Compress large documents into LLM-optimized summaries preserving decisions, constraints, and rules. Use when briefing teammates, managing context windows, or preparing review packages. Targets 30-50% token reduction.
---

Execute all of the following steps immediately. Do not ask for confirmation.

## PURPOSE

Large project documents (PLATFORM-STATE.md, CROSS-CUTTING-DECISIONS.md, review reports) can consume excessive context window when briefing teammates or loading session state. This skill compresses them while preserving all actionable information.

Inspired by BMAD-METHOD's document distillation approach.

## STEP 1: Identify source document

The user will specify a document path, or this skill is invoked programmatically with a file path.

If no path specified, offer these common targets:
1. `_architecture/PLATFORM-STATE.md`
2. `_architecture/CROSS-CUTTING-DECISIONS.md`
3. `_architecture/NEXT-SESSION.md`
4. A git diff (for review package distillation)

Read the full source document.

## STEP 2: Classify content

Scan the document and classify every section/paragraph into:

| Priority | Keep | Examples |
|----------|------|----------|
| **P0 — Must keep** | Verbatim or near-verbatim | Active rules, constraints, decisions, current status, blockers, API contracts |
| **P1 — Summarize** | Compress to key points | Context, rationale, history that informs decisions |
| **P2 — Drop** | Remove entirely | Verbose explanations, examples of obvious things, repeated information, historical details no longer relevant, formatting fluff |

## STEP 3: Compress

Apply these transformations:

### For P0 content (preserve):
- Keep exact wording for rules, constraints, and decisions
- Keep file paths, line numbers, version numbers exactly
- Keep status indicators (PASS/FAIL, done/in-progress)
- Convert verbose lists to compact tables where possible

### For P1 content (summarize):
- Reduce paragraphs to 1-2 sentences capturing the key point
- Replace explanations with the conclusion only
- Merge related items into single statements
- Drop "for example" sections unless the example IS the point

### For P2 content (drop):
- Remove entirely
- Do not leave placeholder text like "[removed for brevity]"

### General rules:
- Use shorthand where clear in context
- Convert prose to bullet points
- Use tables for structured data
- Remove markdown formatting that adds tokens without information (excessive headers, horizontal rules, decorative elements)
- Preserve all code blocks, commands, and file paths exactly

## STEP 4: Validate

Before outputting, verify:
- [ ] No decisions or constraints were lost
- [ ] No file paths or version numbers were changed
- [ ] No active rules were summarized away
- [ ] Current status of all tracked items is preserved
- [ ] The distilled version is self-contained (doesn't reference removed sections)

Count tokens (approximate): original vs distilled. Target: 30-50% reduction.

If reduction < 20%: The document is already lean. Report "Document is already concise. No significant compression possible."

If reduction > 70%: You may have dropped too much. Re-check P1 items — some may need to be P0.

## STEP 5: Output

Return the distilled document with a header comment:

```markdown
<!-- Distilled from {source_path} on {date}. Original: ~{N} tokens, Distilled: ~{M} tokens ({X}% reduction) -->
```

If invoked programmatically (by another skill like /team-start), return the distilled content directly for use in agent briefings.

If invoked by user, present the distilled version and ask: "Want me to save this as `{source_path}.distilled.md` or use it as-is?"

## USE CASES

### Teammate Briefing (used by /team-start)
Distill PLATFORM-STATE + CROSS-CUTTING-DECISIONS into a compact briefing for spawning teammates. Include only: service states, active rules, current task context.

### Review Package (used by /team-review-cycle)
Distill a large git diff into focused review packages per service. Group changes by concern (data model, API, UI, tests) rather than by file.

### Session Handoff (used by /team-stop)
Distill NEXT-SESSION.md if it's grown too long. Keep: what to do next, blockers, which teammates to spawn. Drop: detailed history of what was done.

## NOTES
- This skill is most valuable for projects with 3+ services where state files grow large
- For small projects, distillation may not be needed — skip if source is < 500 tokens
- Never distill CROSS-CUTTING-DECISIONS.md rules — those are always P0
- When in doubt about P1 vs P2, keep it (P1). Better slightly longer than missing info.
