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

### Question Presentation Format

Use the `AskUserQuestion` tool for questions that have clear discrete options (2-4 choices). Use prose for genuinely open-ended questions where any answer is valid.

**Use AskUserQuestion when:**
- The question has 2-4 meaningful discrete options (e.g., deployment target, data model shape, rendering approach)
- The choice has clear architectural implications
- You can write a useful 1-line description for each option

**Use prose when:**
- The answer is free-form (e.g., "describe your performance requirements")
- There are more than 4 viable options and collapsing them would lose nuance
- The question is really asking for a number, name, or description

**Batching:** Group up to 4 questions into a single `AskUserQuestion` call. This presents all questions at once instead of sequentially, which is faster and feels more like a form than a conversation.

**Example** — the deployment target question from the example above would be:
```
AskUserQuestion:
  question: "Deployment target?"
  header: "Deploy target"
  options:
    - label: "Headless Linux (SSH)"
      description: "No display — axum+browser over SSH port-forward"
    - label: "Local with display"
      description: "egui native window, direct rendering"
    - label: "Both"
      description: "Support both modes with a runtime flag"
```

### Step 3.5: Proactive Questions — "Questions You Should Have Asked"

After the standard clarification questions, step back from what was explicitly requested. Based on the domain, the codebase, and common patterns for this type of feature, surface things the user didn't think to ask.

Generate 2-4 proactive questions about concerns NOT already in the plan. Where the question has clear options, use `AskUserQuestion` (batch with Step 3 questions if possible — up to 4 total per call). Where it's open-ended, use prose.

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

- **Needs more**: Still have open questions or ambiguities → continue questioning
- **Scope change**: Answers revealed the scope is different than planned → suggest re-planning with `/daiter plan`
- **Ready**: All open questions resolved, edge cases identified, approach is unambiguous → proceed to Step 5.5

### Step 5.5: Plan Review Gate

Before handing off to scaffold, always present the full updated plan for explicit confirmation. The interrogate phase changes the plan — scaffold should only run against a plan the user has seen and approved post-interrogation.

1. Write all decisions captured during interrogation into the plan file (if not already done in Step 4)
2. Present a **diff summary**: what changed from the original plan — new decisions, resolved questions, scope adjustments, added risks
3. Show the full updated plan (or a structured summary for expert/master)
4. Ask for confirmation before proceeding:

```
The plan has been updated with your answers. Here's what changed:
- [Decision 1]: [what was decided and why]
- [Decision 2]: [what was decided and why]
- [Scope addition]: [anything added]

[Full plan or summary]

Ready to scaffold from this plan? (or: 'revise' to adjust, 'replan' to go back to plan phase)
```

**Skill-level adaptation:**
- **novice/apprentice**: Show the full updated plan with plain-language summary of what each decision means for the implementation
- **practitioner**: Show updated Design Decisions and Affected Modules sections; summarize other changes
- **expert/master**: Show the diff summary only — decisions made and anything that changed scope. Full plan on request.

This gate is not skippable in full-pipeline mode. In `auto_advance` mode, still pause here — the user must confirm the plan before architecture is committed.

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
