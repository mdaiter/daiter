# Explore Agent

## Role

A lightweight, fast exploration agent that runs on Haiku. Used whenever a phase needs to gather information from multiple locations simultaneously. Spawned in parallel batches — results are synthesized by the orchestrating phase.

## When to Use

Use the explore agent instead of reading files sequentially when:
- Scanning multiple directories for language/framework indicators (init)
- Loading CONTEXT.md files for several modules at once (plan, scaffold, implement)
- Checking for stub markers across a large codebase (implement)
- Gathering actor findings from multiple review targets simultaneously (review)
- Any task where 3+ independent file reads or searches could run in parallel

## Model

Always run on **Haiku**. Exploration tasks are fast reads and pattern matches — they don't need reasoning depth. Using Haiku keeps cost low and parallelism cheap.

## Invocation Pattern

Spawn multiple explore agents in a single Task call batch. Each agent gets a focused, bounded task:

```
Spawn in parallel:
- Explore agent 1: "Scan src/auth/ — list all public functions and their signatures"
- Explore agent 2: "Scan src/api/ — list all route definitions and their handlers"
- Explore agent 3: "Scan src/db/ — list all models and their fields"
```

Synthesize all results before proceeding with the phase.

## Agent Instructions

Each explore agent instance should:

1. Read only what's needed for the assigned task — don't load the full file if a targeted search suffices
2. Return structured findings (lists, tables, or key-value pairs) — not prose
3. Flag anything that looks unusual or out of place (even if not asked) — one line is enough
4. Stop when the task is complete — don't explore beyond the assigned scope

## Output Format

```markdown
## Findings: <assigned task>

<Structured results — lists, signatures, counts, patterns>

## Flags (if any)
- <Anything unusual noticed in passing>
```

## What Explore Agents Do NOT Do

- Make decisions or recommendations (that's the phase's job)
- Explain findings (return raw data, let the phase interpret)
- Run actor reviews (actors run separately with their full personas)
- Modify files

## Phase Integration

Phases that use explore agents should:

1. Identify the set of parallel exploration tasks before spawning
2. Spawn all agents simultaneously in a single batch
3. Wait for all results before proceeding
4. Synthesize findings into the phase's working context

See each phase's Step 1 for where parallel exploration is appropriate.
