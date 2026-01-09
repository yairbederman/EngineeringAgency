# Global Workflows

AI agent system for software development lifecycle — from specs to code to PR.

---

## 🚀 5-Minute Quick Start

> Get up and running in 3 commands!

**Step 1: Clone to the workflows directory**

```bash
# macOS/Linux:
git clone <this-repo> ~/.gemini/antigravity/global_workflows

# Windows (PowerShell):
git clone <this-repo> $env:USERPROFILE\.gemini\antigravity\global_workflows
```

**Step 2: Configure (30 seconds)**

Edit `shared/configuration.md`:
```yaml
SYSTEM_NAME: MySystem
WORKSPACE_ROOT:
  # macOS/Linux: ~/projects/MySystem
  # Windows:     C:/Projects/MySystem
```

**Step 3: Run!**

```
/engineering-agent
```

> [!TIP]
> Restart your IDE after cloning for agent discovery. Run `/verify-setup-agent` to validate your configuration.

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
| [Agents Overview](readme/agents_diagram.md) | Workflow hierarchy & when to run each |
| [Manager Guide](readme/manager-usage-guide.md) | Sprint health, risk, status reporting |

## 📝 What You'll Configure

### `shared/configuration.md` (Required)

| Find This | Replace With | Example |
|-----------|--------------|---------|
| `<YOUR_SYSTEM_NAME>` | Your system name | `MySystem` |
| `${WORKSPACE_ROOT}` | Path to your projects | `C:/Projects/MySystem` |

> **Projects are auto-added!** Run `/map-codebase-agent` on any project — it registers automatically.

### `shared/atlassian-config.md` (Optional — skip for local-only mode)

| Find This | Replace With | Example |
|-----------|--------------|---------|
| `<YOUR_ORG>.atlassian.net` | Your Atlassian Cloud ID | `mycompany.atlassian.net` |
| `<PROJECT_KEY>` | Jira project key | `PROJ` |
| `<SPACE_KEY>` | Confluence space key | `DOCS` |
| `<PRODUCT_SPECS_FOLDER_ID>` | Folder ID from URL | `12345678` |
| `<TECH_SPECS_FOLDER_ID>` | Folder ID from URL | `87654321` |

## 🔌 How It Works (Plug & Play)

> **No installation required** — agents are auto-discovered!

When you clone this repo to the workflows directory (`~/.gemini/antigravity/global_workflows`), the AI assistant automatically detects all `.md` files with YAML frontmatter and registers them as slash commands:

| File | Becomes Command |
|------|-----------------|
| `engineering-agent.md` | `/engineering-agent` |
| `map-codebase-agent.md` | `/map-codebase-agent` |
| `system-architecture-agent.md` | `/system-architecture-agent` |
| `manager-agent.md` | `/manager-agent` |

**The path matters**: The AI assistant looks for workflows in the `.gemini/antigravity/` directory. Cloning to another location won't auto-register the commands.

> [!TIP]
> After cloning, restart your IDE to ensure agent discovery runs.

## 📋 For Team Leaders — Migration Guide

### Migrating to a New Machine

```bash
# 1. Clone the workflows repo
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

# 3. Configure shared/configuration.md (SYSTEM_NAME, WORKSPACE_ROOT)

# 4. (Optional) Configure shared/atlassian-config.md (Cloud ID, Jira Key, etc.)

# 5. Install MCP servers (see readme/setup_instructions.md)

# 6. Verify
/verify-setup-agent       # Validates configuration
/engineering-agent  # Should load without errors
```

### Migration Checklist

- [ ] Clone `global_workflows` repo
- [ ] Clone all project repos to single parent directory
- [ ] Update `shared/configuration.md`:
  - [ ] SYSTEM_NAME, WORKSPACE_ROOT
- [ ] *(If using Atlassian)* Update `shared/atlassian-config.md`:
  - [ ] Cloud ID, Jira Project Key, Confluence Space Key
  - [ ] Confluence Folder IDs
  - [ ] *(Optional)* Jira Custom Fields
- [ ] Install Atlassian MCP server *(optional — skip for local-only mode)*
- [ ] Install Figma MCP server *(optional — for frontend design tokens)*
- [ ] Add project folders to VS Code/Cursor workspace
- [ ] Test `/engineering-agent`

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
