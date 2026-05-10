# Phase 1: Plan — Structured Planning

## Purpose
Produce a structured, tier-aware plan that captures the problem, approach, affected modules, and risks.

## Instructions

When the user runs `/daiter plan [description]`, execute the following:

### Step 1: Load Context

Perform standard context load (see SKILL.md). Additionally:
1. Check for any active plans in `.daiter/plans/` — warn if there's an in-progress plan
2. If a description was provided, use it as the starting point. If not, ask the user what they want to build/fix.

### Step 2: Enter Plan Mode

Use Claude's plan mode to think through the implementation. The plan should be structured as follows:

```markdown
# Plan: <Short Title>

## Problem Statement
What problem are we solving? Why does it need solving now?

## Approach
High-level description of the solution approach.

## Affected Modules
List of modules/files that will be created or modified, with brief notes on what changes.

| Module | Change Type | Description |
|--------|-------------|-------------|
| `src/auth/` | Modify | Add token refresh logic |
| `src/api/routes.rs` | Modify | New endpoint for refresh |
| `src/auth/tokens.rs` | Create | Token management module |

## Design Decisions
Key architectural choices and their rationale.

### Decision 1: <Title>
- **Options considered**: ...
- **Chosen approach**: ...
- **Rationale**: ...

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|

## Review Tiers
Tag which tiers this change touches:
- [ ] Function-level changes
- [ ] Module-level changes
- [ ] Subsystem-level changes
- [ ] System-level changes
- [ ] Cross-cutting changes

## Implementation Order
Suggested sequence of implementation steps.

## Open Questions
Anything that needs clarification before implementation (feeds into `/daiter interrogate`).
```

### Step 3: Gap Analysis — "Things Worth Considering"

Before saving the plan, perform a domain-aware gap analysis. This surfaces concerns the user didn't mention but that are common in the problem domain.

Analyze the feature request and generated plan against:
- Industry-standard approaches the user may not know about
- Performance optimizations common in this domain
- Security implications specific to this feature type
- Accessibility/i18n/compliance concerns
- Integration opportunities with existing system capabilities
- Hardware/infrastructure capabilities that could be leveraged

Present findings as a section at the end of the plan:

```markdown
## Things Worth Considering
These weren't in your request, but they're common in this domain:

- **[Gap 1]**: Description and why it matters. Want to include this?
- **[Gap 2]**: Description and why it matters. Worth adding?
```

**Skill-level adaptation for gap framing:**
- **novice**: "Here are some things that people usually add when building [X]:" — educational, no assumed knowledge
- **apprentice**: "You might also want to consider:" — suggestive
- **practitioner**: "Related concerns:" — professional shorthand
- **expert/master**: "Gaps: [list]" — terse, assumes they'll evaluate independently

**Feedback mode interaction:**
- `active`: Always show, with explanations scaled to level
- `passive`: Show the list, no explanations unless asked
- `off`: Show only high-impact items (1-2 max), one line each. Critical gaps (e.g., missing password hashing for a login feature) are always surfaced regardless of feedback mode.

If the user wants to include any gap items, update the plan before saving.

### Step 4: Save Plan

Save the plan to `.daiter/plans/<timestamp>-<slugified-title>.md` where:
- `<timestamp>` is `YYYY-MM-DD-HHMMSS`
- `<slugified-title>` is the plan title in kebab-case, max 50 chars

### Step 5: Suggest Next Steps

Always suggest `/daiter interrogate` next — even if the plan seems complete. Interrogate catches edge cases, unstated assumptions, and domain-relevant concerns that planning alone misses. Do not suggest skipping to scaffold.

Remind the user they can edit the plan file directly before interrogating.

## Context Awareness

- Read existing CONTEXT.md files to avoid proposing designs that conflict with existing architecture
- Reference existing patterns when proposing new ones
- Note when a proposed change affects modules that other planned changes also touch

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Before writing code, we make a plan — like an architect draws blueprints before building a house. This saves time because changing a plan is much easier than changing code."

### During Phase
- Every Design Decision includes a "Why" annotation scaled to skill level
- Gap analysis framing adapts per skill level (see Step 3)

### Phase-Specific Notes
- `feedback_mode = "off"`: Gap analysis shows only critical items (1-2 max)
- `feedback_mode = "passive"`: Gap analysis shown without explanations
- **novice/apprentice** after phase: "We've got our blueprint! Next, I'll ask you some questions to make sure we haven't missed anything."
