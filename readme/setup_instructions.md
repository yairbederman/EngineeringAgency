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
mkdir ~/projects/MySystem  # or C:\Projects\MySystem on Windows
cd ~/projects/MySystem

# 2. Clone all project repos here
git clone <project-1-repo>
git clone <project-2-repo>
# ...

# 3. Clone workflows repo to the EXACT path below
#    macOS/Linux:
git clone <this-repo> ~/.gemini/antigravity/global_workflows
#    Windows:
git clone <this-repo> %USERPROFILE%\.gemini\antigravity\global_workflows
```

> [!IMPORTANT]
> **The path `.gemini/antigravity/global_workflows` is required!**
> The AI assistant auto-discovers workflows from this specific directory.
> Cloning elsewhere will NOT register the `/engineering-agent` commands.
> After cloning, restart your IDE to trigger agent discovery.

---

## Part C: Configure

### Required: `shared/configuration.md`

Open [`shared/configuration.md`](../shared/configuration.md) and replace:

| Placeholder | Replace With | Example |
|-------------|--------------|---------|
| `<YOUR_SYSTEM_NAME>` | Your system name | `MySystem` |
| `<YOUR_ORG>.atlassian.net` | Your Atlassian Cloud ID | `mycompany.atlassian.net` |
| `<PROJECT_KEY>` | Your Jira project key | `PROJ` |
| `<SPACE_KEY>` | Your Confluence space key | `DOCS` |
| `${WORKSPACE_ROOT}` | Path to your projects | `C:/Projects/MySystem` |

Add your projects to the **Registered Projects** table.

### Required for Jira: `engineering/configuration.md`

Open [`engineering/configuration.md`](../engineering/configuration.md) and set:

| Placeholder | How to Find |
|-------------|-------------|
| `<PRODUCT_SPECS_FOLDER_ID>` | Navigate to folder in Confluence → extract ID from URL |
| `<TECH_SPECS_FOLDER_ID>` | Same as above |

### Optional: Jira Custom Fields

If your Jira instance requires mandatory fields when creating issues (e.g., "Cross-Project Impact", "Team", etc.):

1. Open [`engineering/configuration.md`](../engineering/configuration.md)
2. Find the **"Jira Required Custom Fields"** section
3. Add one row per mandatory field with Field Name, Field ID, and Default Value
4. See the file's instructions for how to find Field IDs in Jira Admin

> [!TIP]
> Skip this section if your Jira has no mandatory custom fields on issue creation.

---

## Part D: Verify Setup

1. Open VS Code/Cursor
2. **File → Add Folder to Workspace** for each project
3. **File → Save Workspace As** → `MySystem.code-workspace`
4. Run `/engineering-agent` — should load configuration without errors

---

## Part E: Migrating to a New Machine

Quick checklist for copying this system to another developer's machine:

```bash
# On new machine:

# 1. Clone workflows
git clone <workflows-repo> ~/.gemini/antigravity/global_workflows

# 2. Clone projects
mkdir ~/projects/MySystem && cd ~/projects/MySystem
git clone <project-1> && git clone <project-2> ...

# 3. Configure
# Edit: global_workflows/shared/configuration.md
#   → Set WORKSPACE_ROOT to your local path
#   → Set Atlassian Cloud ID (same as team)
#
# Edit: global_workflows/engineering/configuration.md
#   → Set Confluence folder IDs (same as team)

# 4. Install MCP servers (Part A above)

# 5. Setup IDE workspace
#   → Add all project folders
#   → Save workspace file

# 6. Verify
/engineering-agent
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
│   ├── configuration.md              # 🔑 Global config (EDIT THIS)
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
| Jira/Confluence errors | Check `engineering/configuration.md` for correct IDs |
| MCP server errors | Re-authenticate MCP server in IDE settings |

---

## ➕ Adding New Projects

1. Add row to [`shared/configuration.md`](../shared/configuration.md)
2. Run `/map-codebase-agent` on the new project
3. Run `/system-architecture-agent` to update cross-project docs
4. Add folder to VS Code workspace
