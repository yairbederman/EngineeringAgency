# Personas Index

## Overview

This folder contains domain-specific expert personas for the Engineering Agent. Each persona provides focused context, expertise, and quality standards for their assigned modes.

## Persona Registry

| Persona | File | Assigned Modes |
|---------|------|----------------|
| **Product Manager** | `product-manager.md` | ProductSpecReview |
| **Designer** | `designer.md` | DesignAnalysis |
| **System Architect** | `system-architect.md` | FeaturePlanning, TechSpec, TaskPlanning, PullRequest, CodeReview |
| **Backend Developer** | `backend-developer.md` | Implementation (Backend), BugFix (Backend), FastTrack (Backend) |
| **Frontend Developer** | `frontend-developer.md` | Implementation (Frontend), BugFix (Frontend), FastTrack (Frontend) |

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
