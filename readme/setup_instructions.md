# New Developer Setup

## Prerequisites

- [ ] VS Code installed with an AI assistant extension (e.g., GitHub Copilot, Antigravity, Cursor)
- [ ] Access to your team's project repositories
- [ ] Atlassian account (for Jira/Confluence integration)
- [ ] Figma account (for design token extraction)

---

## ⚠️ First-Time Setup (REQUIRED)

> [!IMPORTANT]
> **Before using the agents**, you MUST configure the placeholder values in the configuration files.
> The workflow files use `<PLACEHOLDER>` values that must be replaced with your organization's settings.

### Configuration Files to Update

| File | What to Configure | Priority |
|------|-------------------|----------|
| [`shared/configuration.md`](../shared/configuration.md) | System name, Atlassian settings, Workspace root, Projects registry | **Required** |
| [`engineering/configuration.md`](../engineering/configuration.md) | Confluence folder IDs, Jira custom field IDs | **Required for Jira integration** |
| [`manager/configuration.md`](../manager/configuration.md) | Sprint thresholds, Jira conventions | Optional (uses sensible defaults) |

### Quick Configuration Checklist

1. **Open `shared/configuration.md`** and replace:
   - `<YOUR_SYSTEM_NAME>` → Your system identifier (e.g., `MyProject`, `TeamAlpha`)
   - `<YOUR_ORG>.atlassian.net` → Your Atlassian Cloud ID
   - `<PROJECT_KEY>` → Your Jira project key (e.g., `PROJ`, `DEV`)
   - `<SPACE_KEY>` → Your Confluence space key
   - Add your actual projects to the **Registered Projects** table

2. **Open `engineering/configuration.md`** and replace:
   - `<PRODUCT_SPECS_FOLDER_ID>` → Your Confluence folder ID for Product Specs
   - `<TECH_SPECS_FOLDER_ID>` → Your Confluence folder ID for Tech Specs
   - Configure custom fields if your Jira uses them (or remove if not needed)

---

## Step 0: Configure MCP Servers

The agents require two MCP (Model Context Protocol) servers for full functionality:

### Atlassian MCP Server (Required)

Provides Jira and Confluence integration for epics, tasks, and specs.

- [ ] Install the Atlassian MCP server extension
- [ ] Configure authentication with your Atlassian account
- [ ] Verify access to the Jira project and Confluence space

**Test**: Run `${MCP_ATLASSIAN_GET_USER_INFO}` to verify your connection.

### Figma MCP Server (Optional - Required for Frontend Tasks)

Provides design token extraction for pixel-perfect UI implementation.

- [ ] Install the Figma MCP server extension
- [ ] Configure authentication with your Figma account
- [ ] Verify access to your team's design files

**Test**: Use `${MCP_FIGMA_GET_DESIGN}` on a Figma link to verify.

> [!TIP]
> MCP servers are configured in your VS Code settings or `.vscode/mcp.json`.
> Ask your team lead for the MCP server configuration files.

---

## Step 1: Clone Repositories

```bash
# 1. Create your workspace directory (choose your preferred location)
mkdir <YOUR_WORKSPACE_ROOT>   # e.g., ~/projects or C:\Projects

# 2. Clone all project repositories into this directory
cd <YOUR_WORKSPACE_ROOT>
# Clone all projects you'll be working with
git clone <project-frontend-repo>
git clone <project-backend-repo>
# ... etc.

# 3. Clone this workflows repository to your preferred location
# Common locations:
#   - Antigravity: ~/.gemini/antigravity/global_workflows
#   - General: ~/ai-workflows or C:\ai-workflows
git clone <this-repo> <YOUR_WORKFLOWS_PATH>
```

## Step 2: Configure Workspace Root

- [ ] Open [`shared/configuration.md`](../shared/configuration.md)
- [ ] Set your `${WORKSPACE_ROOT}` value to match your local projects directory

| Platform | Example `WORKSPACE_ROOT` |
|----------|-------------------------|
| Windows | `C:\Projects\MySystem` |
| macOS | `/Users/yourname/projects` |
| Linux | `/home/yourname/projects` |

> [!NOTE]
> All paths in the agents use variable substitution (e.g., `${WORKSPACE_ROOT}/my-project`).
> The agents resolve these at runtime based on your configured paths.

## Step 3: VS Code Workspace Setup

- [ ] Open VS Code
- [ ] **File → Add Folder to Workspace...** for each project listed in your configuration
- [ ] **File → Save Workspace As...** → Save as `MySystem.code-workspace` (use your system name)

## Step 4: Verify Setup

Run this quick verification:

- [ ] All your configured projects visible in VS Code Explorer sidebar
- [ ] Invoke `/engineering-agent` in your AI assistant to test (it should read configuration successfully)

> [!NOTE]
> These workflows work with any AI assistant that supports slash commands and MCP (Model Context Protocol).

---

## 📁 Directory Structure

```
global_workflows/
├── readme/
│   ├── README.md                     # Entry point
│   ├── setup_instructions.md         # This file
│   └── agents_diagram.md             # Agent hierarchy and usage
├── shared/
│   ├── configuration.md              # 🔑 Project registry (SINGLE SOURCE OF TRUTH)
│   └── mcp-config.md                 # MCP tool references
│
├── engineering-agent.md              # Feature lifecycle workflow
├── engineering/                      # Configuration & mode files
│   ├── configuration.md              # 🔧 Atlassian config (Jira/Confluence)
│   ├── core-rules.md                 # Agent behavior rules
│   ├── design/                       # Figma extraction protocol
│   ├── modes/                        # Mode-specific instructions
│   └── templates/                    # Epic, Tech Spec, Task templates
│
├── map-codebase-agent.md             # Project AI instructions generator
├── mapcodebase/                      # Phase files for extraction
│   └── configuration.md              # Output paths
│
├── system-architecture-agent.md      # Cross-project architecture generator
├── system-architecture/              # Phase files for system docs
│   └── configuration.md              # Output paths
│
├── manager-agent.md                  # Engineering Lead's Co-Pilot
└── manager/                          # Manager agent configuration
    ├── configuration.md              # Manager settings
    └── modes/                        # Manager modes (beat, risk, report)
```

---

## 🔧 Configuration Reference

### Files You May Need to Update

| File | What to Configure | When |
|------|-------------------|------|
| [`shared/configuration.md`](../shared/configuration.md) | System name, Atlassian settings, Projects | Initial setup + when projects are added |
| [`engineering/configuration.md`](../engineering/configuration.md) | Confluence folder IDs, Jira custom fields | Initial setup + if Atlassian instance changes |
| [`manager/configuration.md`](../manager/configuration.md) | Sprint thresholds, WIP limits | When tuning delivery metrics |

### Finding Atlassian IDs

| ID Type | How to Find |
|---------|-------------|
| Cloud ID | Run `${MCP_ATLASSIAN_GET_RESOURCES}` or check your Atlassian URL (e.g., `myorg.atlassian.net`) |
| Jira Project Key | Look at issue keys (e.g., if issues are `PROJ-123`, the key is `PROJ`) |
| Confluence Space Key | Check the URL when viewing a space (e.g., `.../spaces/DOCS/...`) |
| Folder IDs | Navigate to folder in Confluence, extract numeric ID from URL |
| Custom Field IDs | Use Jira REST API or Admin settings → Custom Fields |

---

## ➕ Adding New Projects

- [ ] Add project row to [`shared/configuration.md`](../shared/configuration.md)
- [ ] Run `/map-codebase-agent` on the new project
- [ ] Run `/system-architecture-agent` to update cross-project docs
- [ ] Add project folder to VS Code workspace

---

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| Agent can't find project | Verify project is in `shared/configuration.md` and added to VS Code workspace |
| "Placeholder value" errors | Complete the configuration setup above - replace all `<PLACEHOLDER>` values |
| `.ai-instructions/` not found | Run `/map-codebase-agent` on the project first |
| Cross-project context missing | Run `/system-architecture-agent` to generate system architecture docs |
| Jira/Confluence errors | Check `engineering/configuration.md` for correct Atlassian settings |
| MCP server errors | Verify MCP server is installed and authenticated |
