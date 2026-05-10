# Actor System — Framework Reference

## What Are Actors?

Actors are specialist personas that Claude adopts during specific workflow phases. Each actor brings domain expertise, a specific review lens, and writes findings into the wiki/context file system.

Actors are NOT separate processes or API calls — they are prompt-driven personas that Claude switches between within a single session.

## Actor Lifecycle

### Activation

Actors are activated when a phase requires their expertise. Each phase file specifies which actors participate. The workflow engine:

1. Reads `.daiter/config.toml` to get the list of `[actors].enabled`
2. For each actor required by the current phase, reads the actor's definition file
3. Claude adopts the actor's persona, reviews the relevant code/plan, and produces findings
4. Findings are written to the appropriate CONTEXT.md files and/or review reports

### Phase Participation

Each actor definition specifies which phases it participates in. Actors are only loaded when their phase is active — this saves tokens.

### Self-Assessment Protocol

Every `[workflow].actor_reassess_interval` cycles (default: 5), actors self-assess during the `/daiter review` phase. The self-assessment follows this protocol:

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

**If scope has shifted**: Update the actor's focus areas in config. For example:
```toml
[actors.language-specialist]
language = "rust"
focus = ["async patterns", "error handling"]  # was: ["ownership", "lifetimes"]
```

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

This adaptation is the actor's responsibility during output generation — actors read `[user].skill_level` from config before writing findings. The review phase does NOT translate actor output; it presents it as-is. Therefore every actor must follow these guidelines when producing findings.

When writing findings, actors must read `[user].skill_level` from `.daiter/config.toml` and adapt their language:

- **novice/apprentice**: Explain the finding in plain language. Explain WHY it matters. Use "because" liberally. Example: "This function takes user input and puts it directly into a database query. This is dangerous because an attacker could type special characters that trick the database into revealing or deleting data (this is called 'SQL injection')."

- **practitioner**: Use standard terminology with brief rationale. Example: "SQL injection vulnerability — user input interpolated directly into query. Use parameterized queries."

- **expert/master**: Terse. Example: "SQLi: raw interpolation in query. Parameterize."

When `[user].feedback_mode = "off"`, actors report findings without any explanation — just the finding and the fix. When `feedback_mode = "passive"`, actors report findings with brief rationale but don't teach concepts.

## Loading Actors

When a phase requires actors:

1. Read `.daiter/config.toml` to get enabled actors
2. For each required actor, check if a custom version exists at `.daiter/actors/<name>.md`
3. If no custom version, use the default from `actors/<name>.md` in the skill bundle
4. Apply any config overrides (focus areas, language, framework)
5. Execute the actor's review protocol
6. Write findings to appropriate locations

## Custom Actors

Users can create custom actors by:
1. Running `/daiter discover` to get recommendations
2. Manually creating `.daiter/actors/<name>.md` following the template
3. Adding the actor name to `[actors].custom` in config

Custom actors follow the same lifecycle as default actors.
