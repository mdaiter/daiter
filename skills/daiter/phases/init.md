# Phase 0: Init — Bootstrap Repository

## Purpose
Detect the project's language, framework, and structure. Generate configuration, wiki context files, and optionally discover relevant actors.

## Instructions

When the user runs `/daiter init`, execute the following steps in order:

### Step 1: Project Detection

Scan the repository to detect:

1. **Languages**: Look for language-specific files:
   - `Cargo.toml` → Rust
   - `package.json` → JavaScript/TypeScript (check for `tsconfig.json` to confirm TS)
   - `requirements.txt`, `pyproject.toml`, `setup.py` → Python
   - `go.mod` → Go
   - `pom.xml`, `build.gradle` → Java/Kotlin
   - `Gemfile` → Ruby
   - `*.csproj`, `*.sln` → C#
   - Mixed → note all detected languages

2. **Frameworks**: Look for framework indicators:
   - `next.config.*` → Next.js
   - `angular.json` → Angular
   - `vite.config.*` → Vite
   - Django/Flask/FastAPI imports in Python files
   - `ractor`, `actix`, `tokio` in Cargo.toml dependencies
   - Spring Boot indicators in Java
   - `Gemfile` with `rails` → Rails

3. **Structure**:
   - Multiple `Cargo.toml` / `package.json` at different levels → monorepo/workspace
   - Single project file at root → single project
   - Lerna, Nx, Turborepo config → monorepo

Present findings to the user for confirmation before proceeding.

### Step 1.5: Skill-Level Assessment

Ask the user TWO questions to calibrate communication and teaching behavior. See `reference/skill-levels.md` for full details.

**Question 1 — Skill Level:**

> **How comfortable are you with this codebase and stack?**
> 1. Just getting started / mostly using AI to code
> 2. Learning the ropes, can follow along
> 3. Comfortable, can build features independently
> 4. Deep expertise, want efficient workflow
> 5. I architect systems, teach me nothing

Map to `skill_level` and `skill_score`:
- 1 → `novice`, score 2
- 2 → `apprentice`, score 4
- 3 → `practitioner`, score 5
- 4 → `expert`, score 8
- 5 → `master`, score 9

**Question 2 — Learning Intent:**

> **Do you want daiter to help you get better as a developer?**
> 1. Yes — explain decisions, teach me patterns, coach me after sessions
> 2. Sometimes — explain only when I ask or when something is non-obvious
> 3. No — just build the thing, skip the teaching

Map to `feedback_mode`:
- 1 → `"active"`
- 2 → `"passive"`
- 3 → `"off"`

Store these values in the `[user]` section of the config (see Step 2).

**Adapt remaining init steps based on answers:**
- **novice/apprentice**: Give a guided tour of what each config option means and what each init step does.
- **practitioner**: Brief explanation of init steps.
- **expert/master**: Generate config silently, show only the summary.

### Step 2: Generate Configuration

Create `.daiter/config.toml` with detected values. Use the schema from `reference/config-schema.md`.

Default configuration:
```toml
[project]
languages = ["<detected>"]
frameworks = ["<detected>"]
structure = "<detected>"

[workflow]
gate_mode = "guided"
auto_wiki_update = true
actor_reassess_interval = 5

[wiki]
granularity = "hybrid"
file_name = "CONTEXT.md"

[actors]
enabled = ["architect", "security-auditor", "performance-critic", "wiki-actor", "language-specialist", "framework-specialist", "dependency-reviewer"]
custom = []
retired = []

[actors.language-specialist]
language = "<detected>"
focus = [<language-appropriate defaults>]

[actors.framework-specialist]
framework = "<detected>"
focus = [<framework-appropriate defaults>]

[user]
skill_level = "<from Step 1.5>"
skill_score = <from Step 1.5>
feedback_mode = "<from Step 1.5>"

[tools]
enabled = true
max_tools = 20
auto_cleanup = true
```

### Step 3: Generate Wiki Context Files

1. Analyze directory structure to identify significant code directories
2. Suggest a list of directories that should have CONTEXT.md files
3. Present the list to the user with estimated token savings:
   - Calculate approximate token count of all source files in each directory
   - Compare against estimated CONTEXT.md size (~200-500 tokens each)
   - Show potential savings: "Instead of reading ~15,000 tokens of source, Claude reads ~400 tokens of context"
4. After user confirmation, generate CONTEXT.md files following the template in `reference/wiki-system.md`
5. Read source files to populate each CONTEXT.md with accurate content

### Step 4: Actor Discovery

Run actor discovery (see `/daiter discover` logic):

1. Analyze the codebase for patterns that suggest additional actors:
   - Database files/ORMs → "Query Optimizer" actor
   - API route definitions → "API Design" actor
   - Multiple services/containers → "Integration" actor
   - ML/data processing code → "Data Quality" actor
   - Heavy CSS/UI code → "Accessibility" actor
   - Infrastructure as code → "Infrastructure" actor
2. Present recommendations to the user
3. For accepted actors, generate definition files at `.daiter/actors/<name>.md`
4. Update config with new actors

### Step 5: Initialize Support Directories

Create the following directories:
- `.daiter/plans/`
- `.daiter/reviews/`
- `.daiter/coaching/`
- `.daiter/actors/assessments/`
- `.daiter/tools/`

Create `.daiter/tools/registry.json`:
```json
{
  "tools": [],
  "last_cleanup": null
}
```

### Step 6: Hooks (Optional)

Ask the user if they want to install optional hooks:
- **Auto-wiki update**: Triggers wiki staleness check after file changes
- **Actor reassessment**: Runs actor self-assessment at configured intervals

If yes, guide them to copy `hooks/hooks.json` to `.claude/hooks/daiter-hooks.json` or merge into existing hooks config.

### Step 7: Summary

Present a summary of everything that was set up:
- Languages and frameworks detected
- Number of wiki context files created
- Actors enabled
- Estimated token savings per session
- Next steps: suggest running `/daiter plan <feature>` to start their first workflow

## Handling `/daiter discover`

When invoked as `/daiter discover` (standalone), run only Step 4 (Actor Discovery) from the init process. This allows users to re-run discovery after the codebase has evolved.

## Error Handling

- If `.daiter/` already exists, ask user if they want to re-initialize (will overwrite config)
- If no language is detected, ask the user to specify manually
- If the repo is empty, create minimal config and suggest coming back after initial code is written

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Since init creates the config, adaptation applies to Steps 2-7 after the skill assessment in Step 1.5.

### Preamble (novice/apprentice, feedback_mode = "active")
Not applicable — init is the first phase, so context is established by Step 1.5 questions.

### During Phase
- **novice**: Explain what each config section means. Explain CONTEXT.md files and token savings. Explain actors with an analogy ("specialist consultants — each looks through a different lens"). Walk through each step.
- **apprentice**: Brief explanation of key config options. Explain the wiki system concept. List actors with one-line descriptions.
- **practitioner**: Show config, note anything non-obvious.
- **expert/master**: Generate config, show summary only.

### Phase-Specific Notes
- Adaptation applies only after Step 1.5 (the config doesn't exist yet before that)
- **novice/apprentice**: Celebrate setup completion, preview the full workflow with examples
