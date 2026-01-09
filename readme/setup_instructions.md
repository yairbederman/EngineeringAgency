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

## Part C: Configure Workspace

> [!IMPORTANT]
> **Key Change**: Configuration now lives in your **workspace**, not in `global_workflows`.
> This allows the same agents to work across multiple projects on the same machine.

### Step 1: Create Workspace Configuration

Copy the template to your workspace:

```bash
# macOS/Linux:
mkdir -p ~/projects/MySystem/Agent_Config
cp ~/.gemini/antigravity/global_workflows/readme/agent-config.template.md ~/projects/MySystem/Agent_Config/agent-config.md

# Windows (PowerShell):
New-Item -ItemType Directory -Force -Path C:\Projects\MySystem\Agent_Config
Copy-Item $env:USERPROFILE\.gemini\antigravity\global_workflows\readme\agent-config.template.md C:\Projects\MySystem\Agent_Config\agent-config.md
```

### Step 2: Configure `agent-config.md`

Open your workspace's `agent-config.md` and configure:

| Section | Placeholder | Replace With | Example |
|---------|-------------|--------------|---------|
| **Workspace Settings** | `<YOUR_SYSTEM_NAME>` | Your system name | `MySystem` |
| **Storage Mode** | `local` or `atlassian` | Your preference | `local` |

### Step 3: Atlassian Configuration (Optional)

> Skip if using `STORAGE_BACKEND = local`.

In the same `agent-config.md`, configure the Atlassian section:

| Variable | Replace With | Example |
|----------|--------------|---------|
| `ATLASSIAN_CLOUD_ID` | Your Cloud ID | `mycompany.atlassian.net` |
| `JIRA_PROJECT_KEY` | Jira project key | `PROJ` |
| `CONFLUENCE_SPACE_KEY` | Confluence space key | `DOCS` |
| `PRODUCT_SPECS_FOLDER_ID` | Folder ID from URL | `12345678` |
| `TECH_SPECS_FOLDER_ID` | Folder ID from URL | `87654321` |

> [!TIP]
> **Projects are auto-added!** Run `/map-codebase-agent` on any project — it registers to your `agent-config.md` automatically.

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

## Part E: Multiple Workspaces (Same Machine)

The architecture supports **multiple independent workspaces** on the same machine:

```
C:/Projects/
├── WorkspaceAlpha/
│   ├── agent-config.md       ← Config for Alpha projects
│   ├── alpha-web/
│   └── alpha-api/
│
├── WorkspaceBeta/
│   ├── agent-config.md       ← Config for Beta projects (different Jira, etc.)
│   ├── beta-frontend/
│   └── beta-services/
│
└── ClientProject/
    ├── agent-config.md       ← Config for client (their Jira/Confluence)
    └── client-app/
```

**Each workspace has its own**:
- `SYSTEM_NAME` and `WORKSPACE_ROOT`
- Atlassian settings (or local mode)
- Registered projects list

**All workspaces share**:
- The same `global_workflows` agents
- Agent logic, templates, and personas

---

## Part F: Migrating to a New Machine

Quick checklist for copying this system to another developer's machine:

```bash
# On new machine:

# 1. Clone workflows (ONE TIME per machine)
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

# 3. Create workspace config
cp ~/.gemini/antigravity/global_workflows/readme/agent-config.template.md ./agent-config.md
# Edit agent-config.md with your settings

# 4. Install MCP servers (Part A above)

# 5. Setup IDE workspace
#   → Add all project folders
#   → Save workspace file

# 6. Verify
/verify-setup-agent       # Validates configuration
/engineering-agent        # Should load without errors
```

### Files to Copy vs Configure

| Location | Copy As-Is? | Notes |
|----------|-------------|-------|
| `global_workflows/` | ✅ Yes | 100% portable, no edits needed |
| `Agent_Config/agent-config.md` | ❌ Create new | Each machine/workspace gets its own |
| MCP credentials | ❌ Re-auth | Each user authenticates separately |

---

## 📁 Directory Structure

```
~/.gemini/antigravity/
└── global_workflows/               # Portable agents (shared)
    ├── README.md
    ├── readme/
    │   ├── setup_instructions.md   # This file
    │   └── agent-config.template.md   # 🔑 Template for workspace config
    ├── shared/                     # Reference documentation
    │   ├── mcp-config.md           # MCP tool references
    │   └── error-codes.md          # Error code reference
    ├── engineering-agent.md
    ├── map-codebase-agent.md
    ├── system-architecture-agent.md
    └── ...

~/projects/MySystem/                # Your workspace
├── Agent_Config/
│   └── agent-config.md               # 🔑 Workspace configuration
├── project-1/
├── project-2/
└── MySystem.code-workspace
```

---

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Workspace Configuration Required" | Create `Agent_Config/agent-config.md` (see Part C) |
| "Placeholder value" errors | Complete `agent-config.md` — replace all `<PLACEHOLDER>` values |
| Agent can't find project | Run `/map-codebase-agent` on the project (auto-registers) |
| `.ai-instructions/` not found | Run `/map-codebase-agent` on the project |
| Jira/Confluence errors | Check `Agent_Config/agent-config.md` for correct Atlassian settings |
| MCP server errors | Re-authenticate MCP server in IDE settings |

---

## ➕ Adding New Projects

1. Run `/map-codebase-agent` on the new project (auto-registers to `agent-config.md`)
2. Run `/system-architecture-agent` to update cross-project docs
3. Add folder to VS Code workspace
