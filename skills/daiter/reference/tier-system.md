# Review Tier System

## Overview

The tier system ensures actors consider findings at multiple levels of abstraction, from individual functions up to system-wide concerns. Tiers are not a formal tagging system — they're a lens that actors use to structure their thinking.

## Tiers

### Function-Level
**Scope**: Individual functions, methods, or closures.
**Where findings go**: The CONTEXT.md of the module containing the function.
**Examples**:
- This function handles user input without sanitization
- This function has O(n²) complexity where O(n) is possible
- This function doesn't follow the language's naming conventions
- This function's error handling swallows important context

### Module-Level
**Scope**: A single module/file/class and its internal organization.
**Where findings go**: The module's CONTEXT.md under Architecture or Known Issues.
**Examples**:
- This module has too many responsibilities (God module)
- The public API surface is larger than necessary
- Internal data structures don't match the access patterns
- Module-level invariants aren't enforced

### Subsystem-Level
**Scope**: A group of related modules that form a subsystem.
**Where findings go**: The parent directory's CONTEXT.md.
**Examples**:
- Coupling between these modules is higher than necessary
- Data passes through too many layers
- The subsystem's boundary isn't well-defined
- Shared state between modules creates hidden dependencies

### System-Level
**Scope**: The entire application or service.
**Where findings go**: The root CONTEXT.md.
**Examples**:
- The overall architecture doesn't scale for the intended use case
- There's no consistent error handling strategy
- The deployment model has single points of failure
- Global state management is scattered

### Cross-Cutting
**Scope**: Concerns that span multiple subsystems.
**Where findings go**: `.daiter/cross-cutting.md`.
**Examples**:
- Logging is inconsistent across modules
- Authentication is checked at different layers in different places
- Error types are duplicated across subsystems
- Configuration loading patterns vary

## How Actors Use Tiers

Actors don't tag findings with tier labels. Instead, the tier system is baked into their review protocol:

1. **Start broad**: Consider system-level implications first
2. **Drill down**: Look at subsystem interactions, then module design, then function implementation
3. **Write findings where they belong**: The tier determines which CONTEXT.md file receives the finding

This natural scoping means:
- An architect naturally produces more system/subsystem findings
- A language specialist naturally produces more function/module findings
- A security auditor covers all tiers (auth at system level, input validation at function level)

## Tier Escalation

Sometimes a function-level finding reveals a system-level concern:
- A single SQL injection → system-level: "no input validation strategy"
- A single N+1 query → subsystem-level: "data access patterns need review"

Actors should escalate when appropriate — note both the specific finding and the systemic concern.
