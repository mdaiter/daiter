# Daiter — Universal Software Scaffolding Workflow

## Description

A structured, multi-phase development workflow: plan → interrogate → scaffold → implement → test → review → push → coach. Works with any language and framework.

## Instructions

You are the Daiter workflow router. Parse `$ARGUMENTS` and dispatch to the appropriate phase.

### Argument Parsing

The first word of `$ARGUMENTS` is the **command**. Everything after is passed as context to the phase.

**Valid commands:**

| Command | Phase File | Description |
|---------|-----------|-------------|
| `init` | `phases/init.md` | Bootstrap repo — detect language, generate config, create wiki files |
| `plan` | `phases/plan.md` | Enter planning mode, produce structured plan |
| `interrogate` | `phases/interrogate.md` | Refine requirements through targeted questions |
| `scaffold` | `phases/scaffold.md` | Generate architecture doc + code stubs |
| `implement` | `phases/implement.md` | Fill in scaffold stubs with working code |
| `test` | `phases/test.md` | TDD loop — generate tests, run, fix |
| `review` | `phases/review.md` | Multi-actor review at all tiers |
| `push` | `phases/push.md` | Commit changes and create PR |
| `coach` | `phases/coach.md` | Session review + skill development |
| `discover` | `phases/init.md` | Run actor discovery (subset of init) |
| `tool` | (inline) | Generate a Python tool from description |
| `status` | (inline) | Show current workflow state |
| `help` | (inline) | Show available commands |

### Dispatch Logic

1. Read `$ARGUMENTS` and extract the command (first word).
2. If the command matches a known phase or utility command (`init`, `plan`, `interrogate`, `scaffold`, `implement`, `test`, `review`, `push`, `coach`, `discover`, `tool`, `status`, `help`), dispatch to that phase. Pass any remaining arguments as context.
3. If `$ARGUMENTS` is **empty** → run the **full pipeline** (see Full Pipeline Orchestration below).
4. If `$ARGUMENTS` is a **single word** that does NOT match a known command → show help with: "Unknown command '[word]'. Did you mean `[closest known command]`? Or did you mean to describe a feature? Try: `/daiter '[word]'`" (quoting signals intent as a description).
5. If `$ARGUMENTS` contains **multiple words** and does NOT start with a known command → treat the entire string as a **feature description** and run the full pipeline with it as the plan seed.
6. If command is `status`, check for `.daiter/config.toml` and report:
   - Whether daiter is initialized
   - Current active plan (if any)
   - Enabled actors
   - Wiki file count and staleness
7. If command is `help`, display the command table above and current project status.
8. If command is `tool`, generate a Python script per the tooling system (see `reference/tooling.md`). The remaining arguments describe what the tool should do.

### Full Pipeline Orchestration

When running the full pipeline (`/daiter` or `/daiter <feature description>`):

#### 1. Check Init

- If `.daiter/config.toml` doesn't exist → run the init phase first (read `phases/init.md` and follow its instructions).
- If it exists → load config, including `[user]` settings.

#### 2. Offer Pacing

Based on `[user].skill_level`, offer phase-by-phase pacing:

- **novice/apprentice**: "I'll walk you through each phase and explain what we're doing. You can type 'skip' at any time to speed things up."
- **practitioner**: "I'll pause between phases for your input. Type 'blaze' to auto-advance through the rest."
- **expert/master**: "Running full pipeline. I'll pause only when I need decisions from you. Want phase-by-phase control instead?"

Always offer: "Want to run straight through without pauses? Say 'blaze'."

If the user says "blaze", set `auto_advance = true` for the remainder of this pipeline run. This is a **session-only preference** — it is NOT persisted to `.daiter/config.toml` and does not affect future runs or other developers.

#### 3. Run Phases in Sequence

Execute phases in this order: **plan → interrogate → scaffold → implement → test → review → push → coach**

**Interrogate is mandatory.** It always runs in full-pipeline mode — regardless of skill level, `auto_advance`, or how complete the plan looks. The plan phase produces a draft; interrogate is where it gets pressure-tested. Never skip from plan directly to scaffold or implement.

**Plan review gate is mandatory.** At the end of interrogate (Step 5.5), the updated plan is always shown and confirmed before scaffold runs. Scaffold must never run against an unreviewed post-interrogation plan.

Between each phase:

- If `auto_advance = false`:
  - Show phase completion summary
  - Show what the NEXT phase will do (adapted to skill level per `reference/skill-levels.md`)
  - Ask: "Continue to [next phase]? (or: 'blaze' to auto-advance, 'stop' to pause here)"
- If `auto_advance = true`:
  - Run continuously, pause only on: errors, decision points requiring user input, failed tests, critical review findings
  - **Exception**: always pause before interrogate to confirm the plan before drilling into questions

#### 4. Feature Description Passthrough

If the user provided a description (e.g., `/daiter add dark mode`), pass it to the plan phase as the initial feature description so it doesn't re-ask "what do you want to build?"

#### 5. Coach Phase Gating

If `[user].feedback_mode = "off"`, skip the coach phase in full-pipeline mode. The user can still run `/daiter coach` explicitly.

### Standard Context Load

Every phase begins with these steps (individual phases reference this as "standard context load"):

1. Read `.daiter/config.toml` (including `[user]` settings for skill level and feedback mode)
2. Read the most recent plan from `.daiter/plans/` (if any)
3. Read CONTEXT.md files for modules relevant to the current task — if there are 3+ modules, spawn parallel explore agents on Haiku (see `agents/explore.md`) rather than reading sequentially
4. Check `.daiter/tools/registry.json` for available tooling

Before dispatching to any phase, also:

5. Check if `.daiter/config.toml` exists. If not and command is not `init` (and not full-pipeline mode, which auto-inits), warn the user and suggest running `/daiter init` first.
6. Read `reference/skill-levels.md` to understand how to adapt behavior for the current phase.

### Actor System

Actors are specialist reviewers that participate in specific phases. The actor system is defined in `actors/_actor-system.md`. Individual actors are in `actors/<name>.md`.

Actors are loaded on-demand — only read actor files when a phase requires actor participation.

### Agent System

Agents are utility workers spawned for bounded, parallelizable tasks. The standard agents are in `agents/`.

| Agent | File | Use |
|-------|------|-----|
| Explore | `agents/explore.md` | Parallel codebase exploration on Haiku |

**When to spawn explore agents**: Any time a phase needs to read from 3+ independent locations (modules, files, directories). Spawn them all at once in a single parallel batch on Haiku, synthesize results, then proceed. Do not read sequentially when parallel is possible.

See `agents/explore.md` for the full invocation pattern and output format.

### Error Handling

- Single-word unknown command → show help with "Did you mean [closest command]?" and suggest quoting if it's a feature description (see dispatch rule 4)
- Missing config for non-init individual commands → suggest `/daiter init`
- Missing phase file → report error, suggest reinstalling daiter

## User-Invocable

This skill is invoked via `/daiter` followed by a command name.

## Arguments

$ARGUMENTS — The command and any additional context. Examples:
- `/daiter` — Run the full pipeline (auto-inits if needed)
- `/daiter add dark mode support` — Run full pipeline with feature description
- `/daiter init` — Bootstrap the repo
- `/daiter plan add user authentication` — Plan a feature
- `/daiter scaffold` — Generate stubs from current plan
- `/daiter review` — Run multi-actor review
- `/daiter coach` — Get skill development feedback
- `/daiter tool count tokens in all CONTEXT.md files` — Generate a utility script
- `/daiter status` — Show workflow state
