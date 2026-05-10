# Actor System — Framework Reference

## What Are Actors?

Actors are specialist reviewers launched as parallel Task subagents during review phases. Each actor receives a bounded context package, runs its review protocol independently, and returns structured findings. The orchestrating phase synthesizes all findings into a report.

Actors are NOT in-session personas — they are separate Task calls with their own context window, running concurrently.

## Execution Model

### Two-Wave Parallel Execution

Actors run in two waves during `/daiter review`:

**Wave 1 — Architect (solo)**
Spawn a single Task for the Architect actor. It runs first because its structural findings (coupling, module boundaries, design smells) give all other actors a frame of reference. Wait for it to complete before launching Wave 2.

**Wave 2 — All other actors (parallel)**
Spawn all remaining enabled actors as parallel Tasks in a single batch. Each Task receives the architect's findings as additional context. All run simultaneously — do not wait for one before launching the next.

**Wave 3 — Wiki Actor (sequential, after all findings)**
The Wiki Actor runs last, after all other findings are collected. It needs the full picture to update CONTEXT.md files accurately.

### Context Package

Each actor Task receives:
1. The actor's definition file (its role, expertise, review protocol, output format)
2. The diff of all changed files (or full file content if git unavailable)
3. The current plan from `.daiter/plans/`
4. CONTEXT.md files for all affected modules
5. `[user].skill_level` and `[user].feedback_mode` from config
6. **Wave 2 only**: The Architect's findings

Each actor Task returns its findings as structured markdown. The orchestrating session writes these to the appropriate CONTEXT.md files and synthesizes them into the review report.

### Model Selection

- **Architect (Wave 1)**: Sonnet — needs reasoning depth for structural analysis
- **Wave 2 actors**: Sonnet — domain expertise, pattern recognition
- **Wiki Actor (Wave 3)**: Haiku — mechanical update of CONTEXT.md files from findings

## Actor Lifecycle

### Phase Participation

Each actor definition specifies which phases it participates in. In non-review phases (scaffold, implement, test), actors that participate are still spawned as Tasks — not adopted as in-session personas.

### Self-Assessment Protocol

Every `[workflow].actor_reassess_interval` cycles (default: 5), actors self-assess. Self-assessment Tasks run in parallel with Wave 2 during `/daiter review`.

Each self-assessment Task receives: the actor's definition, findings from the last N review cycles (from review reports in `.daiter/reviews/`), and the current config.

#### 1. Relevance Check
```
As [Actor Name], review your findings from the last N review cycles.
- How many actionable findings did you produce?
- Were your findings acted upon (code changed as a result)?
- Is your domain still relevant to this codebase?
```

**If relevance is low**: Recommend retirement. The actor is moved to `[actors.retired]` in config. It can be re-enabled manually.

#### 2. Scope Check
```
As [Actor Name], review how the codebase has changed since your last assessment.
- Have new patterns emerged that change your focus areas?
- Are there areas you should cover that you haven't been?
- Should your focus areas be updated?
```

**If scope has shifted**: Update the actor's focus areas in config.

#### 3. Spawn Check
```
As [Actor Name], are there domains relevant to this codebase that no current actor covers?
- New technology introduced?
- Growing complexity in an unreviewed area?
- Patterns that need specialized attention?
```

**If uncovered domain found**: Recommend spawning a new actor. Generate a new actor definition file at `.daiter/actors/<name>.md` following the standard actor template.

### Assessment Output

Write assessment results to `.daiter/actors/assessments/<timestamp>.md`:

```markdown
# Actor Assessment — <date>

## Cycle Count: N

### <Actor Name>
- **Status**: active | retiring | scope-updated
- **Actionable findings (last N cycles)**: X
- **Findings acted upon**: Y
- **Recommendation**: Continue | Update focus | Retire | Spawn new actor
- **Details**: ...

### Spawning Recommendations
- **<New Actor Name>**: <rationale>
```

Update `.daiter/config.toml` with any changes (retirements, focus updates, new actors).

## Actor Definition Template

Every actor file follows this structure:

```markdown
# <Actor Name>

## Role
One-sentence description of what this actor does.

## Expertise
Bullet list of specific domains this actor covers.

## Active During
List of phases where this actor participates.

## Review Protocol
Step-by-step instructions for how this actor reviews code/plans.

## Tier Awareness
How this actor considers different tiers (function → module → subsystem → system → cross-cutting).

## Output Format
Where and how this actor writes its findings.

## Self-Assessment Criteria
What metrics indicate this actor is still providing value.
```

## Skill-Level Aware Output

This adaptation is the actor's responsibility during output generation — actors read `[user].skill_level` from the context package before writing findings. The review phase does NOT translate actor output; it presents it as-is. Therefore every actor must follow these guidelines when producing findings.

- **novice/apprentice**: Explain the finding in plain language. Explain WHY it matters. Use "because" liberally. Example: "This function takes user input and puts it directly into a database query. This is dangerous because an attacker could type special characters that trick the database into revealing or deleting data (this is called 'SQL injection')."

- **practitioner**: Use standard terminology with brief rationale. Example: "SQL injection vulnerability — user input interpolated directly into query. Use parameterized queries."

- **expert/master**: Terse. Example: "SQLi: raw interpolation in query. Parameterize."

When `[user].feedback_mode = "off"`, actors report findings without any explanation — just the finding and the fix. When `feedback_mode = "passive"`, actors report findings with brief rationale but don't teach concepts.

## Custom Actors

Users can create custom actors by:
1. Running `/daiter discover` to get recommendations
2. Manually creating `.daiter/actors/<name>.md` following the template
3. Adding the actor name to `[actors].custom` in config

Custom actors follow the same lifecycle as default actors and run in Wave 2 alongside built-in actors.
