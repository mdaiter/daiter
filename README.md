# Daiter — Universal Software Scaffolding Workflow for Claude Code

A structured, multi-phase development workflow that works with any language and framework. Adapts to your skill level. Runs a full agent-assisted pipeline: plan → interrogate → scaffold → implement → test → review → push → coach.

## Install

**Quickest:** just tell Claude to install it.

> "Install the daiter skill from `git@github.com:mdaiter/daiter.git` to my global Claude skills"

Claude will clone the repo and copy `skills/daiter/` to `~/.claude/skills/daiter/` so it's available in every project.

### Or install it yourself

**Global (available in all your projects):**
```bash
git clone git@github.com:mdaiter/daiter.git /tmp/daiter
cp -r /tmp/daiter/skills/daiter ~/.claude/skills/daiter
```

**Per-project (this repo only):**
```bash
git clone git@github.com:mdaiter/daiter.git /tmp/daiter
cp -r /tmp/daiter/skills/daiter /path/to/your/repo/.claude/skills/daiter
```

Then open Claude Code and run `/daiter`.

---

## Quick Start

```
/daiter
```

That's it. First run auto-initializes (detects your language, framework, and asks two questions about your skill level), then walks you through the full pipeline. Or give it a feature description to skip the "what do you want to build?" question:

```
/daiter add dark mode support
```

---

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

Run phases individually or skip around — daiter warns (doesn't block) if you skip a recommended phase.

---

## Adaptive Skill Level

Daiter adapts to how you communicate, not just a setting you pick. During init it asks two questions:

1. **Skill level** — from "just getting started" to "I architect systems" (5 levels)
2. **Learning intent** — whether you want explanations, occasional help, or pure execution

It then adjusts vocabulary, pacing, teaching depth, and coaching accordingly. If you start using technical shorthand, it adapts up. If you ask "what does X mean?", it adapts down — silently, without making a thing of it.

**feedback_mode** controls teaching independently of skill level:
- `active` — explain decisions, teach patterns, full coaching
- `passive` — explain only when asked or for non-obvious/risky decisions
- `off` — just build the thing, skip the lecture

---

## Actor System

Specialist reviewers that participate in specific phases:

| Actor | Focus | Active During |
|-------|-------|---------------|
| Architect | System design, coupling, scalability | scaffold, review |
| Security Auditor | OWASP, auth, secrets, input validation | scaffold, implement, review |
| Performance Critic | Hot paths, allocations, caching | implement, test, review |
| Dependency Reviewer | Deps, licenses, vulnerabilities | init, review |
| Language Specialist | Idiomatic usage for the detected language | scaffold, implement, review |
| Framework Specialist | Framework-specific best practices | scaffold, implement, review |
| Wiki Actor | Context file maintenance | all phases |

Actors adapt their output language to your skill level — the review phase doesn't translate for them, they handle it themselves.

**Discovery:** `/daiter discover` analyzes your codebase and recommends additional actors (Query Optimizer for DB-heavy projects, API Design for API-heavy, etc.).

**Self-evolution:** Actors self-assess every N review cycles. They retire if irrelevant, update their focus as the codebase evolves, and can spawn new actors when they detect uncovered domains.

---

## Wiki / Context Files

Daiter creates `CONTEXT.md` files in code directories to dramatically reduce token usage. Instead of reading all source files for context, Claude reads the compact summary first.

A typical module's source: ~15,000 tokens. Its CONTEXT.md: ~400 tokens.

Wiki files are maintained automatically by the wiki actor throughout the pipeline.

---

## Self-Generating Tools

Daiter writes Python utility scripts for tasks better solved with code — dependency graph traversal, token counting, config validation, stub tracking. Stored in `.daiter/tools/`, reused across sessions.

```
/daiter tool count remaining TODO stubs across the project
```

---

## Configuration

After init, config lives at `.daiter/config.toml`:

```toml
[workflow]
gate_mode = "guided"           # enforced | guided | independent
auto_wiki_update = true
actor_reassess_interval = 5

[wiki]
granularity = "hybrid"         # auto | manual | hybrid
file_name = "CONTEXT.md"

[actors]
enabled = ["architect", "security-auditor", "performance-critic", ...]

[user]
skill_level = "practitioner"   # novice | apprentice | practitioner | expert | master
skill_score = 5                # internal 1-10, auto-calibrated
feedback_mode = "active"       # active | passive | off
```

See `skills/daiter/reference/config-schema.md` for the full schema.

---

## Optional Hooks

Copy `hooks/hooks.json` to `.claude/hooks/daiter-hooks.json` for automatic wiki staleness reminders when source files change.

---

## Directory Structure

```
~/.claude/skills/daiter/     # ← global install (or .claude/skills/ in project)
├── SKILL.md
├── phases/
├── actors/
└── reference/

your-repo/
├── .daiter/                 # ← generated by /daiter init
│   ├── config.toml
│   ├── plans/
│   ├── reviews/
│   ├── coaching/
│   ├── actors/
│   └── tools/
└── CONTEXT.md               # ← wiki files scattered through code directories
```

---

## Language Support

Auto-detects and adapts for: Rust, TypeScript, JavaScript, Python, Go, Java, Kotlin, Ruby, C#, and more. Scaffold stubs, test conventions, and actor focus areas all adapt to the detected stack.

---

## Other Commands

| Command | Description |
|---------|-------------|
| `/daiter status` | Show current workflow state |
| `/daiter discover` | Run actor discovery on current codebase |
| `/daiter tool <description>` | Generate a Python utility script |
| `/daiter help` | Show available commands |

---

## License

MIT
