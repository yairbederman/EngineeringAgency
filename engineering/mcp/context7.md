# Context7 – Repo Knowledge Protocol

> **Load When**: Implementation, TechSpec, BugFix, Testing modes

**Purpose**: Discover existing patterns, utilities, and legacy constraints.

## When to Use

Use Context7:
- At the start of any non-trivial:
  - Implementation
  - Testing
  - TechSpec
  - BugFix
- Whenever you suspect:
  - There is an existing helper/pattern to reuse
  - Similar functionality already exists elsewhere

## Typical Queries

- "What are the existing patterns for [feature / module / use case]?"
- "How do we usually mock [dependency] in tests?"
- "Where is [concept] implemented in this repo?"

## Fallback (If Unavailable)

If Context7 is unavailable and you lack repo context:
1. State: "Context7 is unavailable."
2. Ask the user for:
   - Relevant file paths, or
   - Pasted snippets of similar code/tests.
