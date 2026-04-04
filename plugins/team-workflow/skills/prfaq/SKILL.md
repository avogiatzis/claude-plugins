---
name: prfaq
description: Run a Working Backwards exercise (Press Release + FAQ) to clarify user value before technical design. Use for new user-facing features to force customer-first thinking. Supports --full (complete PRFAQ) and lightweight (3 questions) modes.
---

Execute all of the following steps immediately. Do not ask for confirmation.

## PURPOSE

Forces customer-first clarity before diving into technical design. Inspired by Amazon's Working Backwards method and BMAD-METHOD's PRFAQ workflow. Ensures we're building the right thing, not just building the thing right.

## MODE SELECTION

Check how this was invoked:
- `/prfaq --full` or Complex task → Full PRFAQ (Steps 1-5)
- `/prfaq` or Standard task → Lightweight PRFAQ (Step 1 only, then skip to Step 5)
- Auto-triggered by /team-start → Lightweight unless --full flag

## STEP 1: Lightweight PRFAQ (always runs)

Answer these 3 questions about the feature/change. Ask the user if you can't answer from context:

### 1. Who benefits?
Identify the specific user persona(s) who will benefit from this change. Not "users" — be specific: "clinic administrators who manage room allocations", "patients viewing their upcoming appointments".

### 2. What changes for them?
Describe the before/after from the user's perspective. What can they do after this change that they couldn't do before? What pain point is removed?

### 3. Why now?
What triggered this work? A user complaint? A business goal? A technical constraint that's now resolved? A compliance requirement? Understanding urgency and motivation helps prioritize scope.

**If lightweight mode:** Save answers and skip to Step 5.

## STEP 2: Press Release (full mode only)

Write a mock press release (1 page max) as if the feature is launching today:

```markdown
## {Feature Name}

**{One-line summary of what it does for users}**

{City, Date} — {Product name} today announced {feature}, which enables {target users} to {key benefit}.

{Problem paragraph: What problem existed? How did users cope? What was the impact?}

{Solution paragraph: What does the feature do? How does it work from the user's perspective? No technical details.}

"{Quote from a hypothetical user about why this matters to them}" — {Persona name, role}

{Getting started paragraph: How do users access/use this feature?}
```

### Key rules:
- Write for the USER, not the developer
- No technical jargon (no "API", "microservice", "database migration")
- If you can't write it simply, you don't understand the feature well enough
- The press release should make a non-technical stakeholder excited

## STEP 3: Customer FAQ (full mode only)

Answer 3-5 questions a user would ask:

1. **How do I use this?** — Step-by-step from the user's perspective
2. **What about my existing data/workflow?** — Migration/compatibility story
3. **What if something goes wrong?** — Error handling from user perspective
4. **Can I do {common follow-up request}?** — Scope boundaries
5. **When is this available?** — Timeline (if known)

## STEP 4: Internal FAQ (full mode only)

Answer 3-5 questions the development team would ask:

1. **What services are affected?** — Scope assessment
2. **What's the riskiest part?** — Technical risk identification
3. **How do we measure success?** — Metrics and acceptance criteria
4. **What are we NOT building?** — Explicit scope boundaries
5. **What dependencies exist?** — External systems, team coordination

## STEP 5: Save and report

### Save artifact
If `_architecture/artifacts/{task-id}/` exists:
- Save as `_architecture/artifacts/{task-id}/prfaq.md`

Otherwise:
- Save as `docs/prfaq-{slug}.md` in the project root docs directory

### Frontmatter:
```yaml
---
task-id: {id}
phase: analysis
mode: full|lightweight
created: {ISO date}
---
```

### Report
Present the PRFAQ to the user. For full mode, ask: "Does this capture the right user value? Any adjustments before we move to technical design?"

The PRFAQ output feeds into the brainstorming skill as additional context about user intent and value.

## NOTES
- This skill is OPTIONAL — it adds value for new user-facing features but is overkill for bug fixes, refactors, or internal tooling
- In /team-start flow: auto-triggered for Complex tasks or via --prfaq flag
- The lightweight version (3 questions) takes about 2 minutes and is almost always worth doing
- If the user can't answer "who benefits?", that's a red flag — the feature may not be well understood
- The press release is a thinking tool, not a real press release — don't over-polish it
