# Phase 4: Implement — Fill Scaffolds

## Purpose
Systematically implement all scaffold stubs, following implementation notes and architectural guidance.

## Instructions

When the user runs `/daiter implement`, execute the following:

### Step 1: Load Context

Perform standard context load (see SKILL.md). Additionally:
1. Read implementation notes from `.daiter/impl-notes/` if they exist
2. Scan for remaining stubs using parallel explore agents on Haiku (one per module or directory) if the project has 3+ affected modules:
   - Rust: `todo!()` macros
   - TypeScript: `throw new Error("Not implemented")`
   - Python: `raise NotImplementedError`
   - Go: `panic("not implemented")`
   - General: `// TODO: Implement` comments

### Step 2: Prioritize Implementation Order

Order stubs for implementation based on:

1. **Dependencies first**: Implement functions that other functions depend on before their callers
2. **Foundation types**: Data types, interfaces, and traits before implementations
3. **Core logic**: Business logic before glue code
4. **Integration points**: External integrations last (they're hardest to test in isolation)

Present the implementation order to the user and adjust if they have preferences.

### Step 3: Implement Iteratively

For each stub:

1. Read the stub and its TODO/implementation notes
2. Read CONTEXT.md for the containing module
3. Implement the function following:
   - The approach described in implementation notes
   - Idioms appropriate for the detected language
   - Patterns consistent with the existing codebase
   - The error handling strategy from the architecture doc
4. Replace the `todo!()`/`throw`/`raise` with working code
5. Remove the TODO comment but keep the doc comments

### Step 4: Checkpoint Reviews

At natural checkpoints (after completing a logical group of related functions), optionally consult actors:

- **Language Specialist**: Is the implementation idiomatic?
- **Framework Specialist**: Are framework patterns being used correctly?
- **Security Auditor**: Are there security concerns in the implementation?
- **Performance Critic**: Are there obvious performance issues?

Checkpoints are guided, not enforced — the user can ask for actor input at any time with `/daiter review` or continue implementing.

### Step 5: Progress Tracking

After each implementation group, report:
- Stubs completed vs. remaining
- Any stubs that couldn't be implemented (missing information, blocked by other work)
- Any deviations from the plan (and why)

Consider generating a `.daiter/tools/scaffold_completeness.py` tool to track remaining stubs if the project is large.

### Step 6: Wiki Updates

After implementation is complete, have the wiki actor update CONTEXT.md files:
- Update Public API with final signatures (if they changed from scaffold)
- Update Architecture with any implementation-driven design changes
- Note any new dependencies introduced during implementation

### Step 7: Transition

When all stubs are implemented:
- Summarize what was implemented
- Flag any areas of concern
- Suggest `/daiter test` as next step

If some stubs remain:
- List remaining stubs with reasons they weren't implemented
- Suggest whether to continue implementing or proceed to testing what's done

## Implementation Guidelines

- Don't over-engineer — implement what the stub describes, nothing more
- Follow existing patterns in the codebase
- If an implementation reveals a flaw in the scaffold, note it but implement as designed — architectural changes go through a new plan cycle
- If the implementation is straightforward and the stub's TODO is clear, implement without asking for confirmation
- If the implementation involves a non-obvious choice, explain the choice briefly

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Now we fill in the skeleton with working code. I'll go through each stub and write the actual implementation, checking in with you along the way."

### During Phase
- **Checkpoint frequency** scales with skill level:
  - **novice**: Pause after every 1-2 functions
  - **apprentice**: Checkpoint per logical group (3-5 functions)
  - **practitioner**: Checkpoint at module boundaries
  - **expert/master**: Pause only on decisions requiring user input

### Phase-Specific Notes
- **novice/apprentice** after phase: "The code is written — now we need to make sure it actually works correctly."
