# Self-Generating Python Tools System

## Purpose

Some tasks are more reliable or efficient as code than as prompts: dependency graph traversal, token counting, file tree analysis, config validation. Daiter can write Python scripts at runtime and store them in `.daiter/tools/` for reuse.

## How It Works

### Generation

When an actor or phase encounters a task that would benefit from a script:

1. Check `.daiter/tools/registry.json` for an existing tool that fits
2. If no suitable tool exists, generate a new Python script
3. Save it to `.daiter/tools/<name>.py`
4. Register it in `.daiter/tools/registry.json`
5. Run it

### Constraints

All generated tools must follow these rules:

- **Pure Python**: Standard library only — no pip installs required
- **Under 200 lines**: Keep tools focused and small
- **Self-documenting**: Include a module-level docstring explaining purpose and usage
- **CLI interface**: Accept arguments via `sys.argv` or `argparse`
- **Exit codes**: Return 0 on success, non-zero on failure
- **Stdout output**: Print results to stdout in a parseable format (JSON preferred for structured data)

### Tool Template

```python
#!/usr/bin/env python3
"""<Tool Name> — <One-line description>

Usage:
    python <tool_name>.py [arguments]

Description:
    <Multi-line description of what this tool does,
    when to use it, and what it outputs.>
"""

import sys
import json
# ... other stdlib imports

def main():
    # Implementation
    pass

if __name__ == "__main__":
    main()
```

## Registry

`.daiter/tools/registry.json` tracks available tools:

```json
{
  "tools": [
    {
      "name": "token_counter",
      "file": "token_counter.py",
      "description": "Estimates token counts for files and directories",
      "created": "2026-01-15",
      "last_used": "2026-01-20",
      "usage_count": 5
    }
  ],
  "last_cleanup": "2026-01-15"
}
```

## Common Tools

These are frequently useful and may be generated during init or early workflow cycles:

### `token_counter.py`
Estimates token counts for files/directories using a character-based heuristic (~4 chars per token). Helps inform wiki granularity decisions.

### `dep_graph.py`
Parses import statements and builds a module dependency graph. Outputs a JSON adjacency list. Supports multiple languages by detecting file extensions.

### `config_validator.py`
Validates `.daiter/config.toml` against the expected schema. Reports missing required fields, invalid values, and unknown keys.

### `wiki_staleness.py`
Compares CONTEXT.md timestamps against source file modification times. Reports which context files are potentially stale.

### `actor_metrics.py`
Aggregates actor findings across review cycles from `.daiter/reviews/`. Used during actor self-assessment to quantify contribution.

### `scaffold_completeness.py`
Scans for remaining `todo!()`, `// TODO: Implement`, `raise NotImplementedError`, and `panic("not implemented")` stubs. Reports completion percentage.

## Tool Lifecycle

### Auto-Cleanup

When `[tools].auto_cleanup = true`, tools unused for 10+ workflow cycles are candidates for removal. During cleanup:

1. Check each tool's `last_used` date
2. If unused for 10+ cycles, ask the user before removing
3. Remove the file and registry entry

### Manual Generation

Users can generate tools on demand:

```
/daiter tool count tokens in all CONTEXT.md files
```

This triggers tool generation with the description as input. The workflow:
1. Parse the natural language description
2. Check registry for existing tools that might fit
3. If none found, generate a new Python script
4. Register it
5. Run it and return results

### Tool Limit

`[tools].max_tools` (default 20) prevents unbounded growth. If the limit is reached:
1. List tools by usage count
2. Suggest removing least-used tools
3. User confirms before any removal

## Configuration

```toml
[tools]
enabled = true      # Set to false to disable tool generation entirely
max_tools = 20      # Maximum number of tools
auto_cleanup = true # Automatically suggest removing unused tools
```
