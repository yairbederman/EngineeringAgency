# Global Workflows

AI agent workflows for code generation, architecture extraction, and feature lifecycle management.

## Available Workflows

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `/engineering-agent` | Feature lifecycle: ProductSpecReview → FeaturePlanning → TechSpec → Implementation | Start of any feature work |
| `/map-codebase-agent` | Extract AI instructions from a single project | When project structure changes significantly |
| `/system-architecture-agent` | Generate cross-project architecture documentation | When adding new projects or major API changes |

## Workflow Hierarchy

```
/map-codebase-agent           # Per-project (run on each)
         ↓
/system-architecture-agent    # Cross-project (run once)
         ↓
/engineering-agent            # Feature work (uses all above)
```

---

## New Developer Setup

### Prerequisites

- VS Code with GitHub Copilot or compatible AI assistant
- Atlassian MCP authentication configured

### Setup Checklist

- [ ] Clone this repository to `~/.gemini/antigravity/global_workflows`
- [ ] Clone all WG3 project repositories to the same parent directory (e.g., `C:\My Projects\WG3` or `/Users/dev/WG3`)
- [ ] Open a VS Code workspace containing all WG3 projects
- [ ] Verify all projects appear as workspace folders in VS Code's Explorer sidebar

> **Note**: The `${WORKSPACE_ROOT}` variable references the parent directory containing all projects. Ensure all WG3 projects are in the same parent directory and added to your VS Code workspace.

---

## Directory Structure

```
global_workflows/
├── engineering-agent.md          # Feature lifecycle workflow
├── map-codebase-agent.md         # Project AI instructions generator
├── system-architecture-agent.md  # Cross-project architecture generator
├── shared/
│   └── projects.md               # Project registry (single source of truth)
├── mapcodebase/                  # Phase files for map-codebase-agent
├── engineering/                  # Mode files for engineering-agent
└── system-architecture/          # Phase files for system-architecture-agent
```

---

## Quick Start

### 1. Develop Features
```
/engineering-agent
```
Use for ProductSpecReview, FeaturePlanning, TechSpec, and Implementation.

### 2. Update Project AI Instructions (When Needed)
```
/map-codebase-agent
```
Run when a project's structure changes significantly.

### 3. Update System Architecture (When Needed)
```
/system-architecture-agent
```
Run when adding new projects or after major API changes.

---

## Adding New Projects

1. Add project to [`shared/projects.md`](shared/projects.md)
2. Run `/map-codebase-agent` on the new project
3. Re-run `/system-architecture-agent` to update cross-project docs