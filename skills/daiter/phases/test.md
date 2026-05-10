# Phase 5: Test — TDD Loop

## Purpose
Generate tests for implemented code, run them, identify failures, and loop back to fix issues. Target 80%+ coverage.

## Instructions

When the user runs `/daiter test`, execute the following:

### Step 1: Load Context

Perform standard context load (see SKILL.md). Additionally, detect the testing framework:
   - Rust: built-in `#[test]`, check for `#[cfg(test)]` modules
   - TypeScript/JavaScript: look for jest, vitest, mocha config
   - Python: look for pytest, unittest patterns
   - Go: built-in `_test.go` files
   - Determine test file naming conventions from existing tests

### Step 2: Analyze Test Coverage Gaps

1. Identify all public functions/methods implemented in this cycle
2. Check which ones already have tests
3. Prioritize test generation:
   - **Must test**: Public API functions, error paths, boundary conditions
   - **Should test**: Internal functions with complex logic
   - **Nice to have**: Simple getters/setters, delegation functions

### Step 3: Generate Tests

For each function needing tests, generate:

1. **Happy path tests**: Expected inputs produce expected outputs
2. **Edge case tests**: Boundary values, empty inputs, maximum values
3. **Error path tests**: Invalid inputs, missing dependencies, network failures
4. **Integration tests**: If the function interacts with other modules, test the integration

Follow the project's existing test patterns and conventions. Place tests where the project convention expects them:
- Rust: `#[cfg(test)] mod tests` in the same file, or `tests/` directory
- TypeScript: `*.test.ts` or `*.spec.ts` alongside source or in `__tests__/`
- Python: `test_*.py` in `tests/` directory
- Go: `*_test.go` alongside source

### Step 4: Run Tests

Execute the test suite:
- Use the project's standard test command
- Capture output including failures and coverage if available
- If no test command is configured, detect and suggest one

### Step 5: Failure Loop

If tests fail:

1. **Analyze failure**: Read the error message and stack trace
2. **Categorize**:
   - **Implementation bug**: The code is wrong → fix the implementation
   - **Test bug**: The test expectation is wrong → fix the test
   - **Design flaw**: The approach doesn't work → flag for re-scaffolding
3. **Fix and re-run**: Apply the fix and run tests again
4. **Loop limit**: After 3 fix attempts for the same failure, flag it for human review

If a design flaw is found:
- Document the flaw
- Suggest running `/daiter scaffold` to re-design the affected part
- Don't force a fix that violates the architecture

### Step 6: Coverage Assessment

After all tests pass:

1. Run coverage if available (e.g., `cargo tarpaulin`, `jest --coverage`, `pytest --cov`)
2. Report coverage metrics:
   - Overall coverage percentage
   - Per-module coverage
   - Uncovered lines/branches
3. If coverage is below 80%:
   - Identify the biggest coverage gaps
   - Generate additional tests for uncovered paths
   - Re-run until target is met or remaining uncovered code is justified (e.g., unreachable error handlers)

### Step 7: Actor Reviews

Activate test-phase actors:

- **Security Auditor**: Review tests for missing security edge cases:
  - Are injection attacks tested?
  - Are auth boundaries tested?
  - Are error messages checked for information leakage?
- **Performance Critic**: Identify benchmark opportunities:
  - Are hot-path functions benchmarked?
  - Are there tests for performance-sensitive operations?
  - Suggest benchmark tests where appropriate

### Step 8: Wiki Updates

Have the wiki actor update CONTEXT.md files:
- Note test coverage in relevant modules
- Document any test-driven design changes

### Step 9: Summary

Present:
- Tests generated: count by type (unit, integration, edge case)
- Test results: pass/fail
- Coverage: overall and per-module
- Issues found and fixed
- Remaining concerns
- Suggest `/daiter review` as next step

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Tests are like a safety net — they check that your code does what you expect. We aim for 80% coverage, meaning 80% of your code is checked by at least one test."

### During Phase
- **novice**: Explain each test type (unit, integration, edge case) as generated. Explain coverage per-module.
- **apprentice**: Name test types and briefly explain coverage gaps.
- **practitioner+**: Note only non-obvious test strategies.

### Phase-Specific Notes
- **novice/apprentice** after phase: "Tests are passing! Next, specialist reviewers will check the code for security, performance, and design issues."
