# Configuration Schema — `.daiter/config.toml`

## Overview

The configuration file is generated during `/daiter init` and lives at `.daiter/config.toml` in the project root. It controls all aspects of the daiter workflow.

## Schema

### `[project]` — Project Detection Results

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `languages` | array of strings | yes | Detected programming languages. First entry is primary. |
| `frameworks` | array of strings | no | Detected frameworks. Empty if none detected. |
| `structure` | string | yes | One of: `"single"`, `"monorepo"`, `"workspace"` |

```toml
[project]
languages = ["rust", "python"]
frameworks = ["ractor", "tokio"]
structure = "workspace"
```

### `[workflow]` — Workflow Behavior

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `gate_mode` | string | `"guided"` | How strictly phases enforce order. `"enforced"` = must follow order, `"guided"` = warns but allows skipping, `"independent"` = no ordering |
| `auto_wiki_update` | bool | `true` | Whether the wiki actor automatically updates CONTEXT.md files |
| `actor_reassess_interval` | integer | `5` | Number of review cycles between actor self-assessments |

```toml
[workflow]
gate_mode = "guided"
auto_wiki_update = true
actor_reassess_interval = 5
```

### `[wiki]` — Wiki/Context File System

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `granularity` | string | `"hybrid"` | `"auto"` = daiter decides which dirs get files, `"manual"` = user specifies, `"hybrid"` = auto-suggest + user confirms |
| `file_name` | string | `"CONTEXT.md"` | Name for wiki files. Can be customized (e.g., `"CRATE.md"`, `"MODULE.md"`) |
| `enabled` | bool | `true` | Whether the wiki system is active |

```toml
[wiki]
granularity = "hybrid"
file_name = "CONTEXT.md"
enabled = true
```

### `[actors]` — Actor Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | array of strings | (all defaults) | List of active actor names |
| `custom` | array of strings | `[]` | Names of custom actors (files in `.daiter/actors/`) |
| `retired` | array of strings | `[]` | Previously active actors that self-retired |

```toml
[actors]
enabled = ["architect", "security-auditor", "performance-critic", "wiki-actor", "language-specialist", "framework-specialist", "dependency-reviewer"]
custom = ["api-design"]
retired = []
```

### `[actors.<name>]` — Per-Actor Configuration

Each actor can have its own configuration section. Common keys:

| Key | Type | Description |
|-----|------|-------------|
| `language` | string | For language-specialist: which language to focus on |
| `framework` | string | For framework-specialist: which framework to focus on |
| `focus` | array of strings | Specific areas this actor should prioritize |

```toml
[actors.language-specialist]
language = "rust"
focus = ["ownership patterns", "lifetime management", "error handling idioms"]

[actors.framework-specialist]
framework = "ractor"
focus = ["actor model patterns", "supervision trees", "message passing"]
```

### `[tools]` — Self-Generating Tools Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | bool | `true` | Whether the tooling system is active |
| `max_tools` | integer | `20` | Maximum number of tools to prevent unbounded growth |
| `auto_cleanup` | bool | `true` | Remove tools unused for 10+ cycles |

```toml
[tools]
enabled = true
max_tools = 20
auto_cleanup = true
```

### `[user]` — User Skill & Preferences

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `skill_level` | string | `"practitioner"` | User's self-assessed skill level. One of: `"novice"`, `"apprentice"`, `"practitioner"`, `"expert"`, `"master"` |
| `skill_score` | float | `5` | Internal adaptive score (1-10), auto-calibrated each interaction. See `reference/skill-levels.md` for calibration protocol. |
| `feedback_mode` | string | `"active"` | Teaching intensity. `"active"` = explain decisions and teach patterns, `"passive"` = explain on ask or for risky/non-obvious decisions only, `"off"` = no teaching, pure execution mode |

```toml
[user]
skill_level = "practitioner"
skill_score = 5
feedback_mode = "active"
```

## Validation Rules

- `languages` must contain at least one entry
- `structure` must be one of the three valid values
- `gate_mode` must be one of the three valid values
- `granularity` must be one of the three valid values
- Actor names in `enabled`, `custom`, and `retired` must not overlap
- `actor_reassess_interval` must be a positive integer
- `max_tools` must be a positive integer
- `skill_level` must be one of: `"novice"`, `"apprentice"`, `"practitioner"`, `"expert"`, `"master"`
- `skill_score` must be a multiple of 0.5 between 1 and 10
- `feedback_mode` must be one of: `"active"`, `"passive"`, `"off"`

## Example Full Config

```toml
[project]
languages = ["typescript"]
frameworks = ["next.js", "prisma"]
structure = "single"

[workflow]
gate_mode = "guided"
auto_wiki_update = true
actor_reassess_interval = 5

[wiki]
granularity = "hybrid"
file_name = "CONTEXT.md"
enabled = true

[actors]
enabled = ["architect", "security-auditor", "performance-critic", "wiki-actor", "language-specialist", "framework-specialist", "dependency-reviewer"]
custom = []
retired = []

[actors.language-specialist]
language = "typescript"
focus = ["type narrowing", "discriminated unions", "async patterns"]

[actors.framework-specialist]
framework = "next.js"
focus = ["app router patterns", "server components", "data fetching"]

[user]
skill_level = "practitioner"
skill_score = 5
feedback_mode = "active"

[tools]
enabled = true
max_tools = 20
auto_cleanup = true
```
