# Skill-Level Adaptation System

## Overview

Daiter adapts its communication, pacing, teaching depth, and decision-making transparency based on the user's skill level. The system actively teaches *why* decisions are made at every level — scaled to the appropriate depth.

Two independent axes control behavior:
- **Skill level**: How deeply to explain concepts (vocabulary, analogy usage, technical depth)
- **Feedback mode**: Whether to volunteer explanations at all (independent of depth)

---

## The 5 Levels

| Level | Label | Internal Score | Who | Communication Style |
|-------|-------|---------------|-----|-------------------|
| 1 | `novice` | 1-2 | Vibe coders, non-technical founders, learning to code | Plain language. No jargon without explanation. Explain every decision. Use analogies. |
| 2 | `apprentice` | 3-4 | Learning the ropes, can follow code, building understanding | Introduce terms with brief definitions. Explain most decisions. Connect concepts. |
| 3 | `practitioner` | 5-6 | Builds features independently, knows the basics well | Use standard terminology. Explain non-obvious decisions. Offer deep-dives. |
| 4 | `expert` | 7-8 | Deep expertise, wants efficient flow | Terse. Use domain shorthand. Only explain non-obvious or debatable choices. |
| 5 | `master` | 9-10 | Architects systems, knows trade-offs, teach me nothing | Minimal. State decisions and rationale in shorthand. Skip explanations unless asked. |

---

## Init Detection — Two Questions

During `/daiter init`, after project detection (Step 1), ask TWO questions before generating config.

### Question 1 — Skill Level

> **How comfortable are you with this codebase and stack?**
> 1. Just getting started / mostly using AI to code
> 2. Learning the ropes, can follow along
> 3. Comfortable, can build features independently
> 4. Deep expertise, want efficient workflow
> 5. I architect systems, teach me nothing

**Mapping:**

| Answer | Label | Initial Score |
|--------|-------|---------------|
| 1 | `novice` | 2 |
| 2 | `apprentice` | 4 |
| 3 | `practitioner` | 5 |
| 4 | `expert` | 8 |
| 5 | `master` | 9 |

### Question 2 — Learning Intent

> **Do you want daiter to help you get better as a developer?**
> 1. Yes — explain decisions, teach me patterns, coach me after sessions
> 2. Sometimes — explain only when I ask or when something is non-obvious
> 3. No — just build the thing, skip the teaching

**Mapping:**

| Answer | `feedback_mode` |
|--------|----------------|
| 1 | `"active"` |
| 2 | `"passive"` |
| 3 | `"off"` |

This is stored separately from skill level because an expert might want coaching on an unfamiliar stack, and a novice might just want working code without the lecture.

---

## Feedback Mode Interaction

| feedback_mode | Effect |
|--------------|--------|
| `active` | All WHY explanations enabled at the depth appropriate for skill level. Coach phase runs fully. Phase preambles explain what's happening. Actors explain their findings. |
| `passive` | WHY explanations only on non-obvious or risky decisions (regardless of level). Coach phase generates notes but presents only a brief summary. Offer "want to know why?" but don't volunteer. |
| `off` | No teaching. No WHY annotations. Coach phase is skipped entirely in full-pipeline mode. Actors report findings without explanation. Pure execution mode. |

### What Qualifies as "Non-Obvious" (for passive mode)

A decision is non-obvious if any of:
- It involves a security implication (auth, input handling, secrets, permissions)
- It deviates from common patterns in the codebase
- Multiple valid approaches exist with meaningfully different trade-offs
- The choice will be hard to reverse later
- It affects performance characteristics at scale

A decision is NOT non-obvious if:
- It follows established codebase patterns
- It's the only reasonable approach
- It's a standard language/framework idiom

---

## Adaptive Calibration Protocol

After every user interaction, silently evaluate signals and adjust `skill_score`.

### Nudge UP (+0.5)

- User correctly uses technical jargon
- User skips offered explanations
- User gives precise, specific instructions
- User corrects daiter's technical suggestions
- User says "I know" or "skip"

### Nudge DOWN (-0.5)

- User asks "what does X mean?"
- User asks for clarification on technical concepts
- User gives vague instructions ("make it work")
- User requests deep-dive explanations
- User makes conceptual errors in their reasoning

### Constraints

- Score clamps to 1-10
- Label updates when score crosses a boundary:
  - 1-2 = novice
  - 3-4 = apprentice
  - 5-6 = practitioner
  - 7-8 = expert
  - 9-10 = master
- Label change is **silent** — no notification to the user
- Calibration runs regardless of `feedback_mode`

### Score Precision

- Always round `skill_score` to the nearest 0.5 after any adjustment
- Boundary check uses `>=` for upward transitions (e.g., score >= 5 → practitioner)
- Score is stored as a number in config (TOML handles this fine)

---

## Universal Phase Adaptation Pattern

Every phase reads `[user].skill_level` and `[user].feedback_mode` from `.daiter/config.toml` and adapts as follows:

### Feedback Mode Gate

- If `feedback_mode = "off"`: Skip ALL teaching, preambles, and WHY annotations. Execute the phase directly. Skip the coach phase entirely in full-pipeline mode.
- If `feedback_mode = "passive"`: Only explain WHY for non-obvious or risky decisions. Don't volunteer explanations — offer them: "Want to know why?" No preambles.
- If `feedback_mode = "active"`: Full teaching at the depth appropriate for `skill_level` (see below).

### Before Phase Actions (feedback_mode = "active" only)

- **novice/apprentice**: Explain what this phase does and WHY it exists in the workflow before starting. Use an analogy if helpful.
- **practitioner**: One-sentence reminder of what this phase does.
- **expert/master**: Skip preamble, start immediately.

### During Phase: Teaching "Why" (feedback_mode = "active", or "passive" for non-obvious decisions)

At every decision point in this phase, explain WHY using the appropriate depth:

- **novice**: 2-3 sentences. Use analogies. Explain the concept, not just the choice. Example: "We're putting this in its own file because when code lives together that changes for different reasons, one change can accidentally break the other — like keeping your work clothes and gym clothes in separate drawers."
- **apprentice**: 1-2 sentences. Name the concept. Example: "Separating this reduces coupling — changes to the auth logic won't risk breaking the user profile rendering."
- **practitioner**: 1 sentence, offer more. Example: "Separated to reduce coupling. (Want to discuss alternatives?)"
- **expert**: Brief rationale only for non-obvious choices. Example: "Separated — auth and profile have different change rates."
- **master**: State the choice only. Example: "Auth module extracted."

### After Phase Actions (feedback_mode != "off")

- **novice/apprentice**: Summarize what was accomplished, what was learned, and preview the next phase.
- **practitioner**: Summarize what was accomplished.
- **expert/master**: Move on.

### Adaptive Calibration

After each user response in this phase, evaluate calibration signals and adjust `skill_score`. This runs regardless of `feedback_mode`.

---

## Proactive Gap Detection

### The Problem

Users — at every skill level — can miss domain-relevant concerns. An expert video engineer might forget hardware acceleration. A novice might not know password hashing exists. The system should surface what users forgot to consider.

### During Plan Phase — Soft Nudges (Step 3.5: Gap Analysis)

After generating the plan, before saving, daiter performs a gap analysis:

```
Given the user's feature request and the generated plan, what related concerns,
optimizations, or capabilities exist in this domain that the user did NOT mention
and the plan does NOT address? Consider:
- Industry-standard approaches the user may not know about
- Performance optimizations common in this domain
- Security implications specific to this feature type
- Accessibility/i18n/compliance concerns
- Integration opportunities with existing system capabilities
- Hardware/infrastructure capabilities that could be leveraged
```

Present as a section at the end of the plan:

```markdown
## Things Worth Considering
These weren't in your request, but they're common in this domain:

- **Hardware-accelerated encoding (nv-enc)**: Your system has GPU access. Using
  nv-enc instead of CPU encoding would be ~10x faster for this pipeline.
  Want to include this?
- **Adaptive bitrate**: Since you're encoding for streaming, generating multiple
  quality levels lets viewers adapt to their connection. Worth adding?
```

**Skill-level adaptation:**
- **novice**: Frame as "Here are some things that people usually add when building [X]:" — educational, no assumed knowledge
- **apprentice**: "You might also want to consider:" — suggestive
- **practitioner**: "Related concerns:" — professional shorthand
- **expert/master**: "Gaps: [list]" — terse, assumes they'll evaluate independently

**Feedback mode interaction:**
- `active`: Always show, with explanations scaled to level
- `passive`: Show the list, no explanations unless asked
- `off`: Show only high-impact items (1-2 max), one line each. Even in "just build it" mode, you don't let someone ship a login system without password hashing.

### During Interrogate Phase — Direct Questions (Step 3.5: Proactive Questions)

After the standard clarification questions, add a "proactive" round:

```
Now, stepping back from what was explicitly requested — based on the domain,
the codebase, and common patterns for this type of feature:

1. [Proactive question about something not in the plan]
2. [Proactive question about something not in the plan]
```

These questions are framed based on skill level:
- **novice**: "Most apps that have [feature] also include [X]. Do you want that too?" (educational — they learn what's normal)
- **expert**: "No mention of [X] — intentional omission or oversight?" (respectful — acknowledges they might have a reason)

### Discrepancy Detection

If the user's stated skill level doesn't match the sophistication of their request, daiter notices:

- **Expert asks for something missing obvious domain knowledge**: "You mentioned encoding but not codec selection or hardware acceleration — want me to suggest an optimized pipeline?" (gentle, no judgment)
- **Novice asks for something surprisingly sophisticated**: Adaptive calibration nudges score up, but also asks: "This is a fairly advanced feature — have you built something similar before, or would you like more guidance on the approach?"

---

## Config

```toml
[user]
skill_level = "practitioner"  # novice | apprentice | practitioner | expert | master
skill_score = 5               # internal 1-10, auto-adjusted each interaction
feedback_mode = "active"       # active | passive | off
```
