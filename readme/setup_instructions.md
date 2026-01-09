# Setup Instructions

## Prerequisites

- [ ] AI assistant with slash commands (VS Code + Copilot, Cursor, Antigravity)
- [ ] Git access to your project repositories
- [ ] Atlassian account (Jira/Confluence) — optional for local-only mode
- [ ] Figma account — optional, for frontend design tokens

---

## Part A: Install MCP Servers

MCP (Model Context Protocol) servers enable Jira, Confluence, and Figma integration.

### Atlassian MCP Server (Recommended)

1. Install the Atlassian MCP extension for your IDE
2. Authenticate with your Atlassian account
3. Verify: `mcp0_getAccessibleAtlassianResources` should return your Cloud ID

### Figma MCP Server (Optional)

1. Install the Figma MCP extension
2. Authenticate with your Figma account
3. Verify: Can fetch design from a Figma link

> [!TIP]
> MCP config lives in `.vscode/mcp.json` or IDE settings. Ask your team lead for config files.

---

## Part B: Clone Repositories

```bash
# 1. Create workspace directory
# macOS/Linux:
mkdir -p ~/projects/MySystem && cd ~/projects/MySystem
# Windows (PowerShell):
mkdir -Force C:\Projects\MySystem; cd C:\Projects\MySystem

# 2. Clone all project repos here
git clone <project-1-repo>
git clone <project-2-repo>
# ...

# 3. Clone workflows repo to the EXACT path below
# macOS/Linux:
git clone <this-repo> ~/.gemini/antigravity/global_workflows
# Windows (PowerShell):
git clone <this-repo> $env:USERPROFILE\.gemini\antigravity\global_workflows
```

> [!IMPORTANT]
> **The path `.gemini/antigravity/global_workflows` is required!**
> The AI assistant auto-discovers workflows from this specific directory.
> Cloning elsewhere will NOT register the `/engineering-agent` commands.
> After cloning, restart your IDE to trigger agent discovery.

---

## Part C: Configure

### Step 1: Project Settings — `shared/configuration.md`

Open [`shared/configuration.md`](../shared/configuration.md) and configure:

| Section | Placeholder | Replace With | Example |
|---------|-------------|--------------|---------|
| **Global Constants** | `<YOUR_SYSTEM_NAME>` | Your system name | `MySystem` |
| **Workspace Config** | `${WORKSPACE_ROOT}` | Path to projects | `C:/Projects/MySystem` |

> **Projects are auto-added!** Run `/map-codebase-agent` on any project — it registers to `shared/projects.md` automatically.

### Step 2: Jira/Confluence — `shared/atlassian-config.md` (Optional)

> Skip this step if working in local-only mode (no Atlassian integration).

Open [`shared/atlassian-config.md`](../shared/atlassian-config.md) and configure:

| Section | Placeholder | Replace With | Example |
|---------|-------------|--------------|---------|
| **Cloud Connection** | `<YOUR_ORG>.atlassian.net` | Your Cloud ID | `mycompany.atlassian.net` |
| **Cloud Connection** | `<PROJECT_KEY>` | Jira project key | `PROJ` |
| **Cloud Connection** | `<SPACE_KEY>` | Confluence space key | `DOCS` |
| **Confluence Folders** | `<PRODUCT_SPECS_FOLDER_ID>` | Folder ID from URL | `12345678` |
| **Confluence Folders** | `<TECH_SPECS_FOLDER_ID>` | Folder ID from URL | `87654321` |

> [!TIP]
> **Jira Custom Fields** section is optional. Only configure if your Jira mandates fields on issue creation.

---

## Part D: Verify Setup

1. Open VS Code/Cursor
2. **File → Add Folder to Workspace** for each project
3. **File → Save Workspace As** → `MySystem.code-workspace`
4. Run `/verify-setup-agent` — validates all configuration
5. Run `/engineering-agent` — should load configuration without errors

> [!TIP]
> Use `/verify-setup-agent` anytime to diagnose configuration issues.

---

## Part E: Migrating to a New Machine

Quick checklist for copying this system to another developer's machine:

```bash
# On new machine:

# 1. Clone workflows
# macOS/Linux:
git clone <workflows-repo> ~/.gemini/antigravity/global_workflows
# Windows (PowerShell):
git clone <workflows-repo> $env:USERPROFILE\.gemini\antigravity\global_workflows

# 2. Clone projects
# macOS/Linux:
mkdir -p ~/projects/MySystem && cd ~/projects/MySystem
# Windows (PowerShell):
mkdir -Force C:\Projects\MySystem; cd C:\Projects\MySystem

git clone <project-1> && git clone <project-2>

# 3. Configure (just 2 values!)
# Edit: global_workflows/shared/configuration.md
#   → Set SYSTEM_NAME, WORKSPACE_ROOT

# 4. Configure Jira/Confluence (optional)
# Edit: global_workflows/shared/atlassian-config.md
#   → Set Cloud ID, Jira Project Key, Confluence Space Key
#   → Set Confluence Folder IDs

# 5. Install MCP servers (Part A above)

# 6. Setup IDE workspace
#   → Add all project folders
#   → Save workspace file

# 7. Verify
/verify-setup-agent       # Validates configuration
/engineering-agent  # Should load without errors
```

### Files to Copy vs Configure

| File | Copy As-Is? | Notes |
|------|-------------|-------|
| All workflow files | ✅ Yes | Generic, portable |
| `shared/configuration.md` | ❌ Update | Set local `WORKSPACE_ROOT` |
| Atlassian IDs | ✅ Same | Team shares same Cloud ID |
| MCP config | ❌ Re-auth | Each user authenticates separately |

---

## 📁 Directory Structure

```
global_workflows/
├── README.md                         # Quick start
├── readme/
│   ├── README.md                     # Full documentation index
│   ├── setup_instructions.md         # This file
│   ├── agents_diagram.md             # Workflow hierarchy
│   └── manager-usage-guide.md        # Manager agent guide
├── shared/
│   ├── configuration.md              # 🔑 Core config: SYSTEM_NAME, WORKSPACE_ROOT
│   ├── atlassian-config.md           # 🔑 Jira/Confluence (optional)
│   ├── projects.md                   # 📁 Project registry (auto-populated)
│   ├── mcp-config.md                 # MCP tool references
│   └── error-codes.md                # Error code reference
├── engineering-agent.md              # Feature lifecycle agent
├── engineering/                      # Engineering agent files
├── map-codebase-agent.md             # Project scanner agent
├── mapcodebase/                      # Map codebase agent files
├── system-architecture-agent.md      # Cross-project agent
├── system-architecture/              # System arch agent files
├── manager-agent.md                  # Delivery oversight agent
└── manager/                          # Manager agent files
```

---

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Placeholder value" errors | Complete Part C — replace all `<PLACEHOLDER>` values |
| Agent can't find project | Add project to `shared/configuration.md` + VS Code workspace |
| `.ai-instructions/` not found | Run `/map-codebase-agent` on the project |
| Jira/Confluence errors | Check `shared/configuration.md` for correct IDs and folder IDs |
| MCP server errors | Re-authenticate MCP server in IDE settings |

---

## ➕ Adding New Projects

1. Add row to [`shared/configuration.md`](../shared/configuration.md)
2. Run `/map-codebase-agent` on the new project
3. Run `/system-architecture-agent` to update cross-project docs
4. Add folder to VS Code workspace
