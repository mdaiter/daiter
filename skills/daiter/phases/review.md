# Phase 6: Review — Multi-Actor Review

## Purpose
Run all enabled actors against the current changes, producing tier-tagged findings written into the wiki system, with a summary report.

## Instructions

When the user runs `/daiter review`, execute the following:

### Step 1: Load Context

Perform standard context load (see SKILL.md). Additionally:
1. Identify all files changed in this workflow cycle:
   - If git is available, use `git diff` to find changed files
   - Otherwise, compare against the plan's affected modules list

### Step 2: Run Actor Reviews

For each enabled actor that participates in the review phase, load their definition from `actors/<name>.md` and execute their review protocol.

**Review order** (to allow later actors to build on earlier findings):

1. **Architect** — Structural review first, sets the frame
2. **Security Auditor** — Security review while architecture is fresh
3. **Language Specialist** — Idiomatic code review
4. **Framework Specialist** — Framework alignment review
5. **Performance Critic** — Performance review
6. **Dependency Reviewer** — Dependency changes review (only if dependency files changed)
7. **Wiki Actor** — Updates context files with all findings
8. **Custom actors** — Any user-defined or discovered actors

For each actor:
1. Read the actor's definition file
2. Adopt the actor's persona
3. Review all changed files through the actor's lens
4. Write findings to the appropriate CONTEXT.md files under `## Actor Notes > ### <Actor Name>`
5. For system-level findings, write to root CONTEXT.md
6. For cross-cutting findings, write to `.daiter/cross-cutting.md`

### Step 3: Generate Review Report

Create a summary report at `.daiter/reviews/<timestamp>.md`:

```markdown
# Review Report — <date>

## Summary
- **Files reviewed**: N
- **Actors participating**: list
- **Total findings**: N (X critical, Y high, Z medium, W low)

## Findings by Actor

### Architect
- Finding 1: [tier] description
- Finding 2: [tier] description

### Security Auditor
- Finding 1: [severity] description
...

## Critical Items (must address before merge)
- Item 1
- Item 2

## Recommendations (should address)
- Item 1
- Item 2

## Notes (informational)
- Item 1
- Item 2

## Action Items
- [ ] Fix: description (from Actor)
- [ ] Consider: description (from Actor)
```

### Step 4: Actor Self-Assessment (Periodic)

Check if the actor reassessment interval has been reached:

1. Count review cycles since last assessment (check `.daiter/actors/assessments/`)
2. If count >= `[workflow].actor_reassess_interval`:
   - Run the self-assessment protocol from `actors/_actor-system.md`
   - Each actor evaluates their own relevance, scope, and spawning recommendations
   - Write assessment to `.daiter/actors/assessments/<timestamp>.md`
   - Update `.daiter/config.toml` with any changes
   - Inform the user of any actor retirements, scope changes, or spawn recommendations

### Step 5: Present Results

Show the user:
- Count of findings by severity
- Critical items that need attention
- Any actor lifecycle changes (retirements, scope updates, new actor recommendations)
- Suggest next steps:
  - If critical findings: fix and re-review
  - If no critical findings: `/daiter push` to commit
  - If actor changes were recommended: confirm or adjust

## Review Quality Guidelines

- Actors should produce actionable findings, not vague suggestions
- Each finding should identify the specific code location
- Findings should be proportional — don't flag trivial issues as high severity
- Actors should acknowledge when code is well-written, not just find problems
- Cross-references between actor findings are valuable (e.g., "the architecture choice flagged by Architect also creates the performance concern noted here")

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Now your code gets reviewed by specialist 'actors' — each one looks at a different aspect. It's like having a team of experts review your work."

### During Phase
- Actor findings are adapted per skill level by the actors themselves (see `actors/_actor-system.md`)
- The review phase presents actor output as-is — it does NOT translate findings

### Phase-Specific Notes
- **novice/apprentice** after phase: "Once we fix the critical issues, we'll save and share the code."
