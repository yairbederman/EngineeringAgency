# Personas Index

## Overview

This folder contains domain-specific expert personas for the Engineering Agent. Each persona provides focused context, expertise, and quality standards for their assigned modes.

## Persona Registry

> **Source of Truth**: See [`../configuration.md`](../configuration.md) → Mode Registry → Mode Mapping
>
> The authoritative mode-to-persona mapping is defined in configuration.md.

### Available Personas

| Persona | File | Description |
|---------|------|-------------|
| **Product Manager** | `product-manager.md` | Product spec analysis, gap identification |
| **Designer** | `designer.md` | Figma extraction, design token mapping |
| **System Architect** | `system-architect.md` | API design, task decomposition, PR review |
| **Backend Developer** | `backend-developer.md` | API implementation, service layer, testing |
| **Frontend Developer** | `frontend-developer.md` | UI components, state management, visual verification |

## Loading Rules

When entering a mode, load the assigned persona file BEFORE processing mode-specific rules:

```
1. Load core-rules.md (always)
2. Load personas/{persona}.md (based on mode)
3. Load modes/{mode}.md (mode-specific rules)
```

## Persona File Structure

Each persona file follows this structure:

1. **Identity** - Role and expertise areas
2. **Domain Ownership** - Modes this persona controls
3. **Thinking Approach** - How this persona analyzes problems
4. **Quality Standards** - Non-negotiable quality bars
5. **Tools Proficiency** - How this persona uses available tools
