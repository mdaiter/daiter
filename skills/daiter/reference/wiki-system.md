# Wiki / Context File System

## Purpose

CONTEXT.md files are per-module knowledge bases that dramatically reduce token usage. Instead of Claude reading entire source files to understand a module, it reads the much smaller CONTEXT.md first and only dives into source when needed.

## How It Works

### Creation

During `/daiter init`:
1. The repo's directory structure is analyzed
2. Directories with significant code are identified (not config dirs, assets, etc.)
3. A CONTEXT.md file is created in each selected directory
4. Content is auto-populated by reading source files

### Maintenance

The wiki actor updates CONTEXT.md files during every phase:
- **plan**: Verifies context is current for planning accuracy
- **scaffold**: Creates CONTEXT.md for new modules
- **implement**: Updates API sections as code is written
- **test**: Notes coverage information
- **review**: Incorporates actor findings
- **push**: Final staleness check before commit

### Token Savings

A typical module with 5 source files (~500 lines total) requires ~15,000 tokens to read fully. Its CONTEXT.md captures the essential information in ~400 tokens — a 97% reduction for context-loading purposes.

Claude still reads source files when it needs implementation details, but the CONTEXT.md tells it *which* files to read and *what* to look for.

## CONTEXT.md Template

```markdown
# <Module Name>

## Purpose
One-paragraph description of what this module does and why it exists.

## Architecture
Key design decisions, patterns used, data flow.
- Pattern: <what pattern is used and why>
- Data flow: <how data enters, transforms, and exits>
- Key invariants: <what must always be true>

## Public API

### Functions
- `function_name(param: Type) -> ReturnType` — Brief description
- `another_function(a: A, b: B) -> C` — Brief description

### Types
- `TypeName` — Brief description
- `EnumName` — Brief description with variants

### Constants / Config
- `CONSTANT_NAME` — What it controls

## Dependencies

### This module depends on:
- `module_a` — What it uses from module_a
- `external_crate` — What it uses

### Depends on this module:
- `module_b` — What it uses from this module

## Known Issues / Tech Debt
- Issue description (flagged by <Actor> on <date>)

## Actor Notes

### Architect
Most recent architectural findings.

### Security Auditor
Most recent security findings.

### Performance Critic
Most recent performance findings.

### Language Specialist
Most recent language-specific findings.

### Framework Specialist
Most recent framework-specific findings.

## History
- <date>: <summary of change> (from plan: <plan reference>)
```

## File Naming

The default name is `CONTEXT.md` but can be configured in `.daiter/config.toml`:

```toml
[wiki]
file_name = "CONTEXT.md"  # or "CRATE.md", "MODULE.md", etc.
```

## Granularity Modes

### `auto`
Daiter decides which directories get CONTEXT.md files based on:
- Number of source files (>= 2)
- Total lines of code (>= 50)
- Presence of a clear module boundary (mod.rs, index.ts, __init__.py)

### `manual`
User specifies exactly which directories get CONTEXT.md files.

### `hybrid` (default)
Daiter suggests directories, user confirms and adjusts the list.

## Staleness Detection

A CONTEXT.md is stale when:
1. Source files in its directory were modified after the CONTEXT.md
2. The Public API section doesn't match actual exports (checked by parsing)
3. Dependencies listed don't match actual imports (checked by parsing)

The wiki actor checks staleness during every phase. For large projects, a `.daiter/tools/wiki_staleness.py` script can automate this check.

## Cross-Cutting Concerns

Findings that span multiple modules are written to `.daiter/cross-cutting.md` rather than any single module's CONTEXT.md:

```markdown
# Cross-Cutting Concerns

## Error Handling
<How errors are handled across the system>

## Logging
<Logging patterns and conventions>

## Authentication
<Auth patterns used across modules>

## <Other cross-cutting topic>
<Description>

## Actor Notes
<Cross-cutting findings from actors>
```

## Root CONTEXT.md

The root CONTEXT.md (in the project root) serves as the system-level overview:
- Overall architecture
- Module dependency graph (high-level)
- System-level design decisions
- System-level actor findings
