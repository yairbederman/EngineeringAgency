# Global Workflows

AI agent system for software development lifecycle — from specs to code to PR.

---

## 🚀 5-Minute Quick Start

> Get up and running in 3 steps!

**Step 1: Clone to the workflows directory (ONE TIME per machine)**

```bash
# macOS/Linux:
git clone <this-repo> ~/.gemini/antigravity/global_workflows

# Windows (PowerShell):
git clone <this-repo> $env:USERPROFILE\.gemini\antigravity\global_workflows
```

**Step 2: Create workspace configuration**

```bash
# macOS/Linux:
mkdir -p ~/projects/MySystem/Agent_Config
cp ~/.gemini/antigravity/global_workflows/readme/agent-config.template.md ~/projects/MySystem/Agent_Config/agent-config.md

# Windows (PowerShell):
New-Item -ItemType Directory -Force -Path C:\Projects\MySystem\Agent_Config
Copy-Item $env:USERPROFILE\.gemini\antigravity\global_workflows\readme\agent-config.template.md C:\Projects\MySystem\Agent_Config\agent-config.md
```

Edit `Agent_Config/agent-config.md` with your settings.

**Step 3: Run!**

```
/engineering-agent
```

> [!TIP]
> Restart your IDE after cloning for agent discovery. Run `/system-health-agent` to validate.

---

## 🔐 Portability Architecture

> [!IMPORTANT]
> **This system is 100% portable.** Clone once per machine, configure once per workspace.

### Key Principle: Separation of Code and Configuration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        YOUR MACHINE                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ~/.gemini/antigravity/global_workflows/  ← PORTABLE (git-managed)       │
│  ├── engineering-agent.md                                                │
│  ├── map-codebase-agent.md                                               │
│  ├── system-architecture-agent.md                                        │
│  └── readme/agent-config.template.md         ← Template only               │
│                                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  C:/Projects/WorkspaceA/                  ← WORKSPACE A                  │
│  ├── Agent_Config/agent-config.md           ← Config for Workspace A       │
│  ├── frontend-app/                                                       │
│  └── backend-api/                                                        │
│                                                                          │
│  C:/Projects/WorkspaceB/                  ← WORKSPACE B                  │
│  ├── Agent_Config/agent-config.md           ← Different config             │
│  └── ...                                                                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### What Makes It Portable

| Component | Location | Git Managed? | Contains |
|-----------|----------|--------------|----------|
| **Agent Code** | `~/.gemini/antigravity/global_workflows/` | ✅ Yes | Workflow logic, phases, personas |
| **Workspace Config** | `${WORKSPACE}/Agent_Config/agent-config.md` | ❌ No (gitignored) | YOUR settings, credentials, projects |

### Detection Order

Agents look for `agent-config.md` in these locations:

1. `${WORKSPACE_ROOT}/Agent_Config/agent-config.md` ← **Recommended**
2. `${WORKSPACE_ROOT}/agent-config.md` ← Fallback

---

## 📝 What Goes in `agent-config.md`

| Section | What to Configure | Example |
|---------|-------------------|---------|
| **Storage Mode** | `local` or `atlassian` | `local` |
| **Workspace Settings** | System name, paths | `SYSTEM_NAME: MySystem` |
| **Atlassian** | Cloud ID, Jira/Confluence keys | *(only if using atlassian mode)* |
| **Registered Projects** | Auto-populated by `/map-codebase-agent` | List of your projects |

> [!TIP]
> Projects are auto-added! Run `/map-codebase-agent` on any project — it registers automatically.

---

## 📋 Prerequisites

- Git
- VS Code, Cursor, or IDE with AI assistant + slash commands
- Atlassian account *(optional — can work in local-only mode)*
- Figma account *(optional — for frontend design tokens)*

## 📖 Documentation

| Doc | Purpose |
|-----|---------|
| [Setup Guide](readme/setup_instructions.md) | Installation & configuration |
| [Config Template](readme/agent-config.template.md) | Workspace configuration template |
| [Agents Overview](readme/agents_diagram.md) | Workflow hierarchy & when to run each |
| [Manager Guide](readme/manager-usage-guide.md) | Sprint health, risk, status reporting |

---

## 📋 For Team Leaders — Migration Guide

### Migrating to a New Machine

```bash
# 1. Clone workflows (ONE TIME per machine)
# macOS/Linux:
git clone <this-repo> ~/.gemini/antigravity/global_workflows
# Windows (PowerShell):
git clone <this-repo> $env:USERPROFILE\.gemini\antigravity\global_workflows

# 2. Clone project repos
# macOS/Linux:
mkdir -p ~/projects/MySystem && cd ~/projects/MySystem
# Windows (PowerShell):
mkdir -Force C:\Projects\MySystem; cd C:\Projects\MySystem
git clone <project-1> && git clone <project-2>

# 3. Create Agent_Config folder and config file
mkdir Agent_Config
cp ~/.gemini/antigravity/global_workflows/readme/agent-config.template.md ./Agent_Config/agent-config.md
# Edit Agent_Config/agent-config.md with your settings

# 4. Install MCP servers (optional — see readme/setup_instructions.md)

# 5. Verify
/system-health-agent
/engineering-agent
```

### Migration Checklist

- [ ] Clone `global_workflows` repo to `.gemini/antigravity/`
- [ ] Clone project repos to workspace directory
- [ ] Create `Agent_Config/agent-config.md` (copy from template)
- [ ] Configure SYSTEM_NAME
- [ ] *(If using Atlassian)* Configure Cloud ID, Jira Key, Confluence settings
- [ ] Install MCP servers *(optional)*
- [ ] Test `/engineering-agent`

---

## 🤖 Available Agents

| Agent | Purpose |
|-------|---------|
| `/engineering-agent` | Feature lifecycle: specs → epic → tasks → code → PR |
| `/map-codebase-agent` | Generate `.ai-instructions/` for a project |
| `/system-architecture-agent` | Cross-project architecture docs |
| `/manager-agent` | Sprint health, risk radar, stakeholder updates |

## 🔗 Agent Hierarchy

```
/map-codebase-agent      → Run per project (generates AI context)
        ↓
/system-architecture-agent → Run once (aggregates all projects)
        ↓
/engineering-agent        → Daily work (uses all above)
        ↓
/manager-agent            → Oversight (observes delivery metrics)
```
