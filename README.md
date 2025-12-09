# Global Workflows

AI agent workflows for code generation, architecture extraction, and feature lifecycle management.

## Available Workflows

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `/engineering-agent` | Feature lifecycle: ProductSpecReview → FeaturePlanning → TechSpec → Implementation | Start of any feature work |
| `/map-codebase-agent` | Extract AI instructions from a single project | Before `/engineering-agent` can work on a project |
| `/system-architecture-agent` | Generate cross-project architecture documentation | After all projects have AI instructions |

## Workflow Hierarchy

```
/map-codebase-agent           # Per-project (run on each)
         ↓
/system-architecture-agent    # Cross-project (run once)
         ↓
/engineering-agent            # Feature work (uses all above)
```

## Directory Structure

```
global_workflows/
├── engineering-agent.md          # Feature lifecycle workflow
├── map-codebase-agent.md         # Project AI instructions generator
├── system-architecture-agent.md  # Cross-project architecture generator
├── mapcodebase/                  # Phase files for map-codebase-agent
├── engineering/                  # Mode files for engineering-agent
└── system-architecture/          # Phase files for system-architecture-agent
```

## Quick Start

### 1. Generate Per-Project AI Instructions
```
/map-codebase-agent
```
Run on each project to generate `.ai-instructions/`.

### 2. Generate System Architecture
```
/system-architecture-agent
```
Run once after all projects have AI instructions. Outputs to `C:\My Projects\WG3\system-architecture\`.

### 3. Develop Features
```
/engineering-agent
```
Use for ProductSpecReview, FeaturePlanning, TechSpec, and Implementation.

## Adding New Projects

1. Add project to `system-architecture/configuration.md`
2. Run `/map-codebase-agent` on the new project
3. Re-run `/system-architecture-agent` to update cross-project docs