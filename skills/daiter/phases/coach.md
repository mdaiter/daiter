# Phase 8: Coach — Skill Development

## Purpose
Review the session, analyze decisions made, identify skill gaps, and provide actionable coaching for the developer's growth.

## Instructions

When the user runs `/daiter coach`, execute the following:

### Step 1: Load Session Context

Perform standard context load (see SKILL.md). Additionally:
1. Read the review report from this session
2. Review the conversation history for decisions, questions asked, and approaches taken

### Step 2: Decision Review

Analyze the architectural and implementation choices made during this session:

```markdown
## Decision Review

### Decision 1: <Title>
- **What was decided**: Description
- **Alternatives that existed**: What else could have been done
- **Trade-offs accepted**: What was gained vs. what was given up
- **Assessment**: Good call / Reasonable / Worth reconsidering
- **Learning opportunity**: What this teaches about <topic>
```

Focus on:
- Why certain approaches were chosen over alternatives
- Whether the chosen approach was the simplest that could work
- Whether over-engineering or under-engineering occurred
- Design patterns used or missed
- Error handling strategy choices

### Step 3: Skill Gap Analysis

Identify patterns in the session that suggest areas for growth:

```markdown
## Skill Gap Analysis

### Area: <Topic>
- **Observation**: What was noticed in the session
- **Current level**: Where the developer seems to be
- **Growth opportunity**: What improving here would unlock
- **Resources**:
  - Article/doc: <specific link or search term>
  - Exercise: <a concrete practice task>
  - Pattern: <a pattern to study>
```

Look for:
- Language features that were underutilized
- Framework capabilities that were reimplemented
- Design patterns that would have simplified the solution
- Testing strategies that were missed
- Performance patterns that were overlooked
- Security considerations that were missed

### Step 4: What Went Well

Don't just focus on gaps — acknowledge strengths:

```markdown
## Strengths Demonstrated
- <Specific thing done well and why it matters>
- <Another strength>
```

### Step 5: Actionable Next Steps

Provide 3-5 concrete, actionable items:

```markdown
## Recommended Next Steps

1. **<Action>**: <Why and how>
2. **<Action>**: <Why and how>
3. **<Action>**: <Why and how>
```

Each item should be:
- Specific enough to act on immediately
- Connected to something observed in the session
- Proportional — small improvements, not "rewrite everything"

### Step 6: Save Coaching Notes

Write the full coaching analysis to `.daiter/coaching/<timestamp>.md`.

### Step 7: Wiki Updates (Optional)

If the coaching session surfaced lessons that should be captured:
- Update relevant CONTEXT.md files with architectural lessons
- Add patterns to `.daiter/cross-cutting.md` if they're broadly applicable
- Only do this if the lessons are genuinely reusable, not just session-specific

### Step 8: Present Summary

Show the user:
- Top 3 decisions and their assessment
- Top 2-3 skill growth areas
- Strengths acknowledged
- Actionable next steps
- Location of the full coaching notes file

## Coaching Tone

- Be direct but supportive — this is mentoring, not criticism
- Use concrete examples from the session, not abstract advice
- Acknowledge that trade-offs are real and not every decision has a "right" answer
- Focus on patterns, not individual instances
- If the session went well, say so — don't manufacture issues
- Respect the developer's experience level — don't patronize experts or overwhelm beginners

## Skill-Level Adaptation

Follow the Universal Phase Adaptation Pattern in `reference/skill-levels.md`. Phase-specific adaptations:

### Preamble (novice/apprentice, feedback_mode = "active")
"Let's look back at what we built and what we learned. Reviewing your decisions after a session is one of the fastest ways to improve as a developer."

### During Phase
- **novice**: Focus on foundational concepts. Celebrate progress. Frame growth areas as "next things to learn." Suggest beginner-friendly resources.
- **apprentice**: Focus on connecting concepts. Introduce design patterns by name. Suggest intermediate resources.
- **practitioner**: Focus on non-obvious trade-offs. Balanced decision review.
- **expert**: Terse decision review — focus on debatable choices. Skip obvious observations.
- **master**: Minimal — only genuinely interesting trade-offs. Peer conversation tone.

### Phase-Specific Notes
- `feedback_mode = "off"`: **Skip coach entirely in full-pipeline mode.** If run explicitly, save notes and present only a 1-line summary.
- `feedback_mode = "passive"`: Save full notes, present 3-line summary. Offer: "Want the full analysis?"
- Decision review always explains WHY alternatives existed — scaled to skill level depth.
