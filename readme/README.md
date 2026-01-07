# Global Workflows — Documentation

AI agent workflows for code generation, architecture extraction, and feature lifecycle.

---

## 🚀 For New Developers

### 1. First-Time Setup
See [setup_instructions.md](setup_instructions.md) for full guide.

**TL;DR:**
1. Clone this repo to `~/.gemini/antigravity/global_workflows` (or your preferred path)
2. Configure `shared/configuration.md` with your Atlassian/project settings
3. Add your project folders to VS Code workspace
4. Run `/engineering-agent` to verify

### 2. What's Inside

| Agent | When to Use | Output |
|-------|-------------|--------|
| `/engineering-agent` | Feature work, bugs | Jira tasks, code, tests, PRs |
| `/map-codebase-agent` | Project structure changed | `.ai-instructions/` in project |
| `/system-architecture-agent` | New project added | `system-architecture/` docs |
| `/manager-agent` | Daily/weekly check-ins | Risk reports, status briefs |

---

## 📋 For Team Leaders — Migration Guide

### Migrating to a New Machine

```bash
# 1. Clone the workflows repo
git clone <this-repo> ~/.gemini/antigravity/global_workflows

# 2. Clone all project repos to a common directory
mkdir ~/projects/MySystem
cd ~/projects/MySystem
git clone <project-1> && git clone <project-2> ...

# 3. Configure paths
#    Edit: shared/configuration.md
#    Set: WORKSPACE_ROOT = ~/projects/MySystem
#    Set: Atlassian Cloud ID, Project Key, Space Key

# 4. Install MCP servers (see setup_instructions.md)

# 5. Verify
/engineering-agent  # Should load without errors
```

### Migration Checklist

- [ ] Clone `global_workflows` repo
- [ ] Clone all project repos to single parent directory
- [ ] Update `shared/configuration.md` with local paths
- [ ] Update `engineering/configuration.md` with Atlassian IDs
- [ ] Install Atlassian MCP server
- [ ] Install Figma MCP server (optional, for frontend)
- [ ] Add project folders to VS Code/Cursor workspace
- [ ] Test `/engineering-agent`

---

## 📚 Full Documentation

- [**Setup Instructions**](setup_instructions.md) — Step-by-step installation
- [**Agents & Diagrams**](agents_diagram.md) — Workflow hierarchy, gates, usage
- [**Manager Usage Guide**](manager-usage-guide.md) — `/beat`, `/risk`, `/status`, `/retro`

---

## ⚡ Quick Links to Agents

- [`/engineering-agent`](../engineering-agent.md) — Feature lifecycle
- [`/map-codebase-agent`](../map-codebase-agent.md) — Project AI instructions
- [`/system-architecture-agent`](../system-architecture-agent.md) — Cross-project docs
- [`/manager-agent`](../manager-agent.md) — Delivery oversight