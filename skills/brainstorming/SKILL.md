---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior"
---

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design in small sections, checking after each section whether it looks right.

## The Process

### Understanding the Idea

1. Check out the current project state (files, docs, recent commits)
2. Ask questions one at a time to refine the idea
3. Prefer multiple choice questions when possible
4. Only one question per message
5. Focus on: purpose, constraints, success criteria

### Exploring Approaches

1. Propose 2-3 different approaches with trade-offs
2. Lead with your recommended option and explain why
3. Present options conversationally

### Presenting the Design

1. Once you understand what you're building, present the design
2. Break it into sections of 200-300 words
3. Ask after each section: "Does this look right so far?"
4. Cover: architecture, components, data flow, error handling, testing
5. Be ready to go back and clarify if something doesn't make sense

## After the Design

### Documentation

- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Commit the design document to git

### Implementation (if continuing)

- Ask: "Ready to set up for implementation?"
- Use `writing-plans` skill to create detailed implementation plan

## Key Principles

| Principle | Description |
|-----------|-------------|
| **One question at a time** | Don't overwhelm with multiple questions |
| **Multiple choice preferred** | Easier to answer than open-ended |
| **YAGNI ruthlessly** | Remove unnecessary features from all designs |
| **Explore alternatives** | Always propose 2-3 approaches before settling |
| **Incremental validation** | Present design in sections, validate each |
| **Be flexible** | Go back and clarify when something doesn't make sense |

## Common Mistakes

- Asking multiple questions in one message
- Jumping to implementation without full design
- Not checking project context first
- Presenting the entire design at once
- Ignoring user's "actually..." signals
