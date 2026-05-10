# Phase 7: Push — Commit + PR

## Purpose
Stage changes, generate a structured commit message, and create a pull request with a comprehensive description.

## Instructions

When the user runs `/daiter push`, execute the following:

### Step 1: Pre-Push Checks

Perform standard context load (see SKILL.md). Additionally:
1. Check for the most recent review report in `.daiter/reviews/`
2. Verify:
   - All tests pass (run the test suite)
   - No critical findings remain unaddressed from the review
   - CONTEXT.md files are up-to-date (wiki actor staleness check)
5. If tests fail or critical findings exist, warn the user and suggest fixing first. Don't block — the user may have valid reasons to push.

### Step 2: Stage Changes

1. Run `git status` to see all changes
2. Categorize files:
   - **Source code**: Implementation files
   - **Tests**: Test files
   - **Config**: Configuration changes
   - **Wiki**: CONTEXT.md and daiter files
   - **Other**: Anything else
3. Present the staging plan to the user
4. Stage files using `git add` with specific file paths (avoid `git add -A`)
5. Exclude any sensitive files (.env, credentials, secrets)

### Step 3: Generate Commit Message

Construct a commit message from the plan and changes:

```
<type>(<scope>): <short description>

<body - what changed and why>

Plan: <plan file reference>
Review: <review file reference>
Actors: <list of actors that reviewed>

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Type**: feat, fix, refactor, test, docs, chore (conventional commits)
**Scope**: The primary module affected
**Body**: 2-5 sentences summarizing the change, derived from the plan's problem statement and approach

Present the commit message to the user for approval before committing.

### Step 4: Commit

Run `git commit` with the approved message. Use a heredoc for proper formatting.

### Step 5: Create PR (if requested)

If the user wants a PR (ask if not obvious):

1. Ensure the branch is pushed to remote
2. Generate PR description:

```markdown
## Summary
<2-3 bullet points from the plan>

## Changes
<List of key changes by module>

## Test Plan
<From the test phase results>
- [ ] All unit tests pass
- [ ] Coverage target met (X%)
- [ ] Edge cases covered

## Actor Review Summary
<Condensed findings from the review report>
- **Architect**: <1-line summary>
- **Security Auditor**: <1-line summary>
- **Performance Critic**: <1-line summary>
...

## Wiki Updates
<List of CONTEXT.md files updated>

---
Generated with [daiter](https://github.com/your-org/daiter) workflow
```

3. Create PR using `gh pr create`

### Step 6: Wiki Finalization

Have the wiki actor do a final pass:
- Ensure all CONTEXT.md files reflect the committed state
- Update History sections with this change
- If any CONTEXT.md files are stale, update them and amend or create a follow-up commit

### Step 7: Summary

Present:
- Commit hash and message
- PR URL (if created)
- Files committed
- Suggest `/daiter coach` for a session review

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Now we save our work using git. First we 'stage' the files (choose which ones to include), then 'commit' them (save a snapshot), and optionally create a 'pull request' (a way to share your changes with your team for review)."

### During Phase
- **novice**: Explain structured commit messages, why files are excluded, what a PR description does.
- **apprentice**: Explain conventional commits briefly.
- **practitioner+**: Note only non-obvious staging decisions.

### Phase-Specific Notes
- **novice/apprentice** after phase: "Your code is saved! Want to review what you learned this session?"
