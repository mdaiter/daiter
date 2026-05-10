# Phase 3: Scaffold — Stub Generation

## Purpose
Generate three layers of scaffolding: a foundation architecture doc, idiomatic code stubs, and per-function implementation notes.

## Instructions

When the user runs `/daiter scaffold`, execute the following:

### Step 1: Load Context

Perform standard context load (see SKILL.md). Additionally:
1. If no plan exists, warn the user and suggest `/daiter plan` first

### Step 2: Foundation — Architecture Document

Create a prose architecture document in the plan file (append as a new section) or as a companion file. This document covers:

```markdown
## Foundation Architecture

### Module Boundaries
- What each module is responsible for
- What each module explicitly does NOT handle
- Clear ownership of data and behavior

### Data Flow
- How data enters the system
- How data moves between modules
- How data exits the system (responses, side effects, storage)

### Key Decisions
- Why this structure was chosen
- What alternatives were rejected and why
- Constraints that shaped the design

### Error Strategy
- How errors propagate across boundaries
- Where errors are handled vs. where they're returned
- User-facing vs. internal error representations
```

### Step 3: Scaffold — Code Stubs

Generate function/method declarations with structured TODO comments. Auto-detect the language from config and emit idiomatic stubs.

#### Language-Specific Stub Patterns

**Rust:**
```rust
/// Brief description of what this function does.
///
/// # Arguments
/// * `param` - Description
///
/// # Returns
/// Description of return value
///
/// # Errors
/// When and why this function returns errors
pub fn function_name(param: Type) -> Result<ReturnType, Error> {
    todo!("Implement: detailed description of what needs to happen")
}
```

**TypeScript:**
```typescript
/**
 * Brief description of what this function does.
 * @param param - Description
 * @returns Description of return value
 * @throws When and why this function throws
 */
export function functionName(param: Type): ReturnType {
  // TODO: Implement — detailed description of what needs to happen
  throw new Error("Not implemented");
}
```

**Python:**
```python
def function_name(param: Type) -> ReturnType:
    """Brief description of what this function does.

    Args:
        param: Description

    Returns:
        Description of return value

    Raises:
        ErrorType: When and why
    """
    raise NotImplementedError("Implement: detailed description of what needs to happen")
```

**Go:**
```go
// FunctionName does brief description.
// Parameters:
//   - param: Description
// Returns description of return value and error conditions.
func FunctionName(param Type) (ReturnType, error) {
	panic("not implemented: detailed description of what needs to happen")
}
```

**Other languages:** Adapt the pattern to the language's conventions. Use the language's standard "not implemented" marker.

#### Stub Content Rules

- Each stub should have a clear doc comment explaining purpose
- TODO/todo! messages should describe WHAT needs to happen, not HOW
- Include type signatures that match the architecture doc
- Create files in the correct directory structure per the plan
- Include necessary imports/use statements
- Add module-level documentation

### Step 4: Implementation Notes

For each stubbed function, generate implementation notes. These can be:
- Inline as expanded TODO comments (for simpler functions)
- In a companion `.daiter/impl-notes/<module>.md` file (for complex functions)

Implementation notes should include:
- **Algorithm or approach**: What strategy should be used
- **Key dependencies**: What other functions/modules this calls
- **Edge cases**: What the implementation needs to handle
- **Testing hints**: What should be tested for this function
- **Performance considerations**: Any hot-path awareness

### Step 5: Actor Reviews

Activate the following actors to review the scaffold:

1. **Architect**: Reviews module boundaries, coupling, data flow
2. **Language Specialist**: Reviews stub signatures for idiomatic patterns
3. **Framework Specialist**: Reviews for framework alignment

Read each actor's definition from `actors/` and execute their scaffold-phase review protocol. Incorporate feedback before presenting to the user.

### Step 6: Wiki Updates

Have the wiki actor update CONTEXT.md files:
- Create CONTEXT.md for any new directories
- Update Public API sections with new function signatures
- Update Architecture sections with design decisions from the foundation doc

### Step 7: Present Results

Show the user:
- Summary of files created/modified
- Count of stubs generated
- Any actor feedback that needs attention
- Suggest `/daiter implement` as next step

## Quality Criteria

- Every public function has a doc comment
- Every stub has a TODO/todo! with a meaningful description
- File structure matches the architecture doc
- No circular dependencies in the planned module graph
- Types and interfaces are defined before implementations that use them

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"It's like drawing the floor plan before building the house — we create empty outlines of every function so we can see the whole structure before filling in any details."

### During Phase
- **novice**: Show each stub and explain what each part means (signature, doc comment, todo marker). Explain why we use `todo!()` / `throw` / `raise` instead of leaving it blank.
- **apprentice**: Explain the stub pattern briefly. Name the concept (interface-first design).
- **practitioner+**: Generate stubs with brief notes only on non-obvious patterns.

### Phase-Specific Notes
- **novice/apprentice** after phase: "Now we have the skeleton — next we'll fill in the actual code."
