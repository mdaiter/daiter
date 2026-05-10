# Daiter — Universal Software Scaffolding Workflow for Claude Code

A structured, multi-phase development workflow that works with any language and framework. Drop it into your repo and immediately get agent-assisted planning, scaffolding, implementation, testing, review, and coaching.

## Quick Start

```bash
# 1. Copy the skills into your repo
cp -r skills/daiter /path/to/your/repo/.claude/skills/daiter

# 2. Open Claude Code in your repo
cd /path/to/your/repo
claude

# 3. Just run daiter — it handles everything
/daiter
```

That's it. `/daiter` auto-initializes on first run (detecting your language, framework, structure, and skill level), then walks you through the full pipeline: plan → interrogate → scaffold → implement → test → review → push → coach.

You can also give it a feature description directly:

```
/daiter add dark mode support
```

This skips the "what do you want to build?" question and runs the full pipeline with your description.

## Workflow Phases

| Phase | Command | Description |
|-------|---------|-------------|
| **Init** | `/daiter init` | Bootstrap — detect language, generate config, create wiki files |
| **Plan** | `/daiter plan <feature>` | Structured planning with risk assessment |
| **Interrogate** | `/daiter interrogate` | Refine requirements through targeted questions |
| **Scaffold** | `/daiter scaffold` | Generate architecture doc + idiomatic code stubs |
| **Implement** | `/daiter implement` | Fill in scaffold stubs with working code |
| **Test** | `/daiter test` | TDD loop — generate tests, run, fix, target 80%+ coverage |
| **Review** | `/daiter review` | Multi-actor review at all tiers |
| **Push** | `/daiter push` | Stage, commit, and create PR |
| **Coach** | `/daiter coach` | Session review + skill development feedback |

You can run phases individually or skip around — daiter will warn (not block) if you skip a recommended phase. But the simplest way is just `/daiter` to run the full pipeline.

## Adaptive Skill Level

Daiter adapts its communication to your skill level. During init, it asks two questions:

1. **Skill level** — from "just getting started" to "I architect systems" (5 levels)
2. **Learning intent** — whether you want explanations, occasional help, or pure execution

Based on your answers, daiter adjusts:
- **Vocabulary**: Plain language for beginners, domain shorthand for experts
- **Pacing**: More checkpoints for learners, fewer pauses for experts
- **Teaching**: Explains WHY decisions are made (scaled to depth), or skips teaching entirely
- **Coaching**: Full session reviews for learners, minimal for experts

The system silently recalibrates based on how you interact — if you start skipping explanations or using technical terms, it adapts up. If you ask "what does X mean?", it adapts down.

You can control teaching independently of skill level via `feedback_mode`:
- `active` — explain decisions, teach patterns
- `passive` — explain only when asked or for risky decisions
- `off` — just build the thing

See `skills/daiter/reference/skill-levels.md` for the full protocol.

## Actor System

Actors are specialist reviewers that participate in specific phases:

| Actor | Focus | Active During |
|-------|-------|---------------|
| Architect | System design, coupling, scalability | scaffold, review |
| Security Auditor | OWASP, auth, secrets, input validation | scaffold, implement, review |
| Performance Critic | Hot paths, allocations, caching | implement, test, review |
| Dependency Reviewer | Deps, licenses, vulnerabilities | init, review |
| Language Specialist | Idiomatic usage of detected language | scaffold, implement, review |
| Framework Specialist | Framework-specific best practices | scaffold, implement, review |
| Wiki Actor | Context file maintenance | all phases |

### Actor Discovery

```
/daiter discover
```

Analyzes your codebase and recommends additional actors (e.g., Query Optimizer for database-heavy projects, API Design for API-heavy projects).

### Self-Evolution

Actors self-assess every N review cycles (configurable). They can:
- **Retire** if their domain is no longer relevant
- **Mutate** their focus areas as the codebase evolves
- **Spawn** new actors when uncovered domains are detected

## Wiki / Context Files

Daiter creates `CONTEXT.md` files in code directories to dramatically reduce token usage. Instead of reading all source files for context, Claude reads the compact CONTEXT.md first.

A typical module's source files require ~15,000 tokens. Its CONTEXT.md captures the essentials in ~400 tokens.

Wiki files are automatically maintained by the wiki actor during every phase.

## Self-Generating Tools

Daiter can write Python scripts for tasks better solved with code: dependency graph traversal, token counting, config validation, staleness detection. Tools are stored in `.daiter/tools/` and reused across sessions.

```
/daiter tool count remaining TODO stubs across the project
```

## Configuration

After init, configuration lives in `.daiter/config.toml`. Key settings:

```toml
[workflow]
gate_mode = "guided"           # "enforced" | "guided" | "independent"
auto_wiki_update = true
actor_reassess_interval = 5

[wiki]
granularity = "hybrid"         # "auto" | "manual" | "hybrid"
file_name = "CONTEXT.md"

[actors]
enabled = ["architect", "security-auditor", ...]

[user]
skill_level = "practitioner"   # novice | apprentice | practitioner | expert | master
skill_score = 5                # internal 1-10, auto-adjusted
feedback_mode = "active"       # active | passive | off
auto_advance = false           # skip phase pauses in full pipeline
```

See `skills/daiter/reference/config-schema.md` for the full schema.

## Optional Hooks

Copy `hooks/hooks.json` to `.claude/hooks/daiter-hooks.json` for automatic wiki staleness reminders when source files are modified.

## Directory Structure

```
your-repo/
├── .claude/
│   └── skills/
│       └── daiter/          # ← copy this directory
│           ├── SKILL.md
│           ├── phases/
│           ├── actors/
│           └── reference/
├── .daiter/                 # ← generated by /daiter init
│   ├── config.toml
│   ├── plans/
│   ├── reviews/
│   ├── coaching/
│   ├── actors/
│   ├── tools/
│   └── cross-cutting.md
└── CONTEXT.md               # ← wiki files in code directories
```

## Language Support

Daiter works with any language. It auto-detects and adapts scaffold stubs, test conventions, and actor focus areas for:

- Rust, TypeScript/JavaScript, Python, Go, Java/Kotlin, Ruby, C#, and more

## Other Commands

| Command | Description |
|---------|-------------|
| `/daiter` | Run the full pipeline (auto-inits if needed) |
| `/daiter <description>` | Run full pipeline with feature description |
| `/daiter status` | Show current workflow state |
| `/daiter discover` | Run actor discovery |
| `/daiter tool <description>` | Generate a Python utility script |
| `/daiter help` | Show available commands |
