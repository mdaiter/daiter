# Phase 2: Interrogate — Requirement Refinement

## Purpose
Refine the plan through targeted questions until Claude has full clarity on requirements, edge cases, and constraints.

## Instructions

When the user runs `/daiter interrogate`, execute the following:

### Step 1: Load Plan

Perform standard context load (see SKILL.md). Additionally:
1. Find the most recent plan in `.daiter/plans/` (or the plan specified by the user)
2. Read the plan file completely

### Step 2: Analyze Gaps

Review the plan for:

1. **Ambiguous requirements**: Anything that could be interpreted multiple ways
2. **Missing edge cases**: Error conditions, boundary values, concurrent access, empty states
3. **Unstated assumptions**: Things the plan assumes but doesn't explicitly state
4. **Integration unknowns**: How this change interacts with existing code
5. **Open questions**: Items explicitly marked as needing clarification in the plan

### Step 3: Structured Questioning

Ask questions in tier order, starting broad and drilling down:

**System-level** (ask first):
- Does this change affect the overall system architecture?
- Are there deployment or migration considerations?
- Does this interact with external systems?

**Subsystem-level**:
- How does this change affect neighboring modules?
- Are there data flow changes between subsystems?
- Are there new dependencies between subsystems?

**Module-level**:
- What is the expected public API surface?
- How should errors be handled and propagated?
- What are the invariants this module must maintain?

**Function-level**:
- What are the edge cases for key functions?
- What are the performance requirements?
- What inputs should be validated?

Ask 3-5 questions at a time, wait for answers, then drill deeper if needed. Don't overwhelm with too many questions at once.

### Step 3.5: Proactive Questions — "Questions You Should Have Asked"

After the standard clarification questions, step back from what was explicitly requested. Based on the domain, the codebase, and common patterns for this type of feature, surface things the user didn't think to ask.

Generate 2-4 proactive questions about concerns NOT already in the plan:

```
Now, stepping back from what was explicitly requested — based on the domain,
the codebase, and common patterns for this type of feature:

1. [Proactive question about something not in the plan]
2. [Proactive question about something not in the plan]
```

**Skill-level adaptation for question framing:**
- **novice**: "Most apps that have [feature] also include [X]. Do you want that too?" (educational — they learn what's normal)
- **apprentice**: "Have you considered [X]? It's common with this kind of feature." (suggestive)
- **practitioner**: "[X] — need it?" (concise)
- **expert/master**: "No mention of [X] — intentional omission or oversight?" (respectful — acknowledges they might have a reason)

**Discrepancy detection**: If the user's stated skill level doesn't match the sophistication of their request:
- **Expert missing obvious domain knowledge**: "You mentioned [X] but not [Y] — want me to suggest an optimized approach?" (gentle, no judgment)
- **Novice with surprisingly sophisticated request**: Nudge calibration score up, and ask: "This is a fairly advanced feature — have you built something similar before, or would you like more guidance on the approach?"

### Step 4: Capture Decisions

After each round of answers:

1. Update the plan file with decisions made:
   - Move items from "Open Questions" to "Design Decisions"
   - For each captured decision, note WHY that decision was made, not just what was decided
   - Add edge cases to the risk assessment
   - Update affected modules if scope changed
2. Show what was updated in the plan

### Step 5: Completion Check

After each round, assess whether the plan is sufficiently clear to proceed:

- **Ready**: All open questions resolved, edge cases identified, approach is unambiguous → suggest `/daiter scaffold`
- **Needs more**: Still have open questions or ambiguities → continue questioning
- **Scope change**: Answers revealed the scope is different than planned → suggest re-planning with `/daiter plan`

### Step 6: Skip Handling

If the user runs `/daiter scaffold` directly without having run `/daiter interrogate`, **stop and run interrogate first** before proceeding to scaffold. Interrogate is a mandatory gate in the pipeline — the only exception is if the user explicitly overrides ("I want to skip interrogate"). In that case, acknowledge the trade-off and proceed.

## Question Quality Guidelines

- Ask questions that will materially change the implementation, not trivial ones
- Prefer concrete examples: "What should happen when X?" over "Have you considered X?"
- Group related questions together
- If the user says "use your best judgment" for a question, make a decision and document it in the plan
- Don't re-ask questions that the plan already answers clearly

### Question Vocabulary Adaptation

Adapt question complexity and vocabulary to skill level:

- **novice**: Use plain language. Example: "What should happen when someone types nothing and hits submit?"
- **apprentice**: Introduce terms. Example: "How should we handle empty input validation? (That's when someone submits without filling in the field.)"
- **practitioner**: Standard terminology. Example: "Empty input validation strategy?"
- **expert/master**: Terse, domain shorthand. Example: "Edge case: empty input validation strategy?"

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Asking questions now prevents having to throw away code later — it's like measuring twice before cutting."

### During Phase
- Question vocabulary adapts to skill level (see Question Vocabulary Adaptation above)
- Proactive question framing adapts per skill level (see Step 3.5)
- Discrepancy detection runs at all levels

### Phase-Specific Notes
- **novice/apprentice** after phase: "Great — we've filled in the gaps. Now I'll create the code structure (scaffolding) based on everything we discussed."
