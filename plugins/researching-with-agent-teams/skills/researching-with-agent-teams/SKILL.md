---
name: researching-with-agent-teams
description: Use when researching topics that inform strategic decisions, exploring multi-angle complex questions, or evaluating product or market opportunities where simplistic findings or demo-theatre examples would mislead the decision.
---

# Researching with Agent Teams

## Overview

Strategic research that informs real decisions is not a solo task. A single agent reading docs produces findings that sound complete because nobody attacked them, and that often miss the gap between "technically correct" and "business-realistic."

**Core principle:** If no agent attacked the finding, and no agent validated whether a real operator would actually ask that question, the finding is not ready for a decision.

This skill mandates a four-role workflow: parallel research → adversarial review → validation → synthesis. The synthesis stays with you, not an agent.

## When to Use

Use when:
- Research output will inform a strategic decision (architecture, product roadmap, market entry, vendor choice, ADR)
- Multiple independent angles exist (technical, business, customer-pain, competitive, regulatory)
- The audience is stakeholders who will act on the findings
- Cost of being wrong is meaningful (engineering capacity, customer trust, sales positioning)

Do NOT use when:
- Direct factual lookup (use Read, Grep, WebFetch)
- Single-domain code investigation (use the Explore subagent)
- The decision is reversible at near-zero cost
- The user explicitly asks for one quick answer

## The Four Roles

### Phase 1: Parallel research (2-4 agents, dispatched in one message)

Each agent owns ONE independent angle. Briefs must be self-contained (no reference to prior conversation), name file paths or URLs to read, specify output structure with a word cap, and require "cite file:line or URL for every claim". Cap each at 20-30 minutes.

Do not spawn one omnibus agent. Do not allow agents to roam across angles.

### Phase 2: Adversarial review (≥2 reviewers, parallel, after research lands)

Each reviewer attacks one dimension. Briefs include the instruction: "Be brutal. Your value is killing claims that don't deserve to survive."

**Reviewer A: product/user-value critique.** For each claimed feature, surface, or insight:
- Who specifically asks this? (role, frequency, workflow)
- What do they do today without it? (the "30-second browser tab" test)
- What's the failure mode under realistic conditions?
- Strongest counter-argument they could raise
- Verdict per item: Keep / Reframe / Kill

**Reviewer B: technical and business-model skepticism.** Attack:
- Effort estimates (what's not counted?)
- Privacy/security claims ("no PHI" is rarely as clean as claimed)
- Business-model silence (who pays, how much, for what?)
- Competitive-moat claims
- The SINGLE most damaging assumption in the proposal

### Phase 3: Validators (1-2, parallel)

**Data validator.** Every quantitative or factual claim cited by research agents must trace to a real artefact. Download the file, read the schema, run the grep, confirm the URL. If a research agent inferred something as "presumed available", validator confirms by direct inspection.

**Business-realism validator.** This role catches the "are we under audit?" anti-pattern. For every example user question, customer scenario, or workflow trigger in the findings:

- Would a real operator in that role actually ask this, or do they already know?
- Is the example demo theatre (sounds plausible, fails on first contact with a real user)?
- Does the proposed surface solve the *easy* part of a hard workflow?
- Would a real buyer name this when explaining a renewal decision?

Concrete failures this role must catch:
- "When does our accreditation expire?" — Quality Managers know; it's on the wall calendar
- "Are we under compliance review?" — Operators absolutely know; assessors are physically on site
- "Show me our published Star Rating" — Already public; one click on the public registry
- "Sales prospect lookup tool" — Sales already has a browser bookmark

The validator names every failed example and proposes reframe or kill.

### Phase 4: Synthesis (you, not an agent)

Hold all five agents' outputs in context simultaneously and produce the synthesis yourself. An agent dispatched to "synthesise" defers to whichever input it read most recently, and the judgment calls (which kill to accept, which reframe to embrace, which assumption is load-bearing) are the entire point of the exercise.

Synthesis must explicitly:
- Present the kills with the strongest counter-argument that survived
- Present the survivors with the reframe applied
- Name the single most-damaging assumption
- State what would need to be true (validation gates) before commitment

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "It's a small question, parallel is overkill" | If it informs a decision, run the full team. If it doesn't, this skill doesn't apply. |
| "Findings look good, skip the adversarial review" | They look good because nobody attacked them. That's the bug, not the signal. |
| "I'll have an agent synthesise to save my context" | Synthesis IS the judgment. Don't delegate the judgment. |
| "The example questions sound great" | Run the business-realism validator anyway. Real operators kill these examples reliably. |
| "I'll add adversarial review later if the findings hold up" | Adversarial review BEFORE synthesis is the point. After is theatre. |

## Red Flags — STOP and Run the Cycle

- About to write an exec summary or ADR from a single agent's output
- "The research agent's findings look complete, I'll synthesise from this"
- Example user questions in the draft that the audience already knows the answer to
- Proposed surface duplicates a 30-second browser-tab workflow the user already has
- No competitive landscape mentioned in the findings
- No "what would have to be false for this to be wrong" section in the draft

All of these mean: dispatch the team. The findings are not decision-ready.

## Pattern from Practice

A regulatory-data ingestion proposal went through this cycle: two research agents (operator pain + cross-plane analytics), two adversarial reviewers (product-surface + technical/business), one data validator, one business-realism validator, then synthesis. The original proposal claimed five customer features. After the cycle: three were killed (one because the example chat questions were demo theatre an operator would never ask), one was reframed, one survived. The killed features would have absorbed 3-4 epics building things operators don't pay for. Catching that before engineering committed was the value of the cycle.
