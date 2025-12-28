# New Developer Setup

## Prerequisites

- [ ] VS Code installed with an AI assistant extension (e.g., GitHub Copilot, Antigravity, Cursor)
- [ ] Access to WG3 project repositories
- [ ] Atlassian account (for Jira/Confluence integration)
- [ ] Figma account (for design token extraction)

## Step 0: Configure MCP Servers

The agents require two MCP (Model Context Protocol) servers for full functionality:

### Atlassian MCP Server (Required)

Provides Jira and Confluence integration for epics, tasks, and specs.

- [ ] Install the Atlassian MCP server extension
- [ ] Configure authentication with your Atlassian account
- [ ] Verify access to the WG3 Jira project and Confluence space (see [`engineering/configuration.md`](../engineering/configuration.md) for keys)

**Test**: Run `${MCP_ATLASSIAN_GET_USER_INFO}` to verify your connection.

### Figma MCP Server (Optional - Required for Frontend Tasks)

Provides design token extraction for pixel-perfect UI implementation.

- [ ] Install the Figma MCP server extension
- [ ] Configure authentication with your Figma account
- [ ] Verify access to WG3 design files

**Test**: Use `${MCP_FIGMA_GET_DESIGN}` on a Figma link to verify.

> [!TIP]
> MCP servers are configured in your VS Code settings or `.vscode/mcp.json`.
> Ask your team lead for the MCP server configuration files.

## Step 1: Clone Repositories

```bash
# 1. Create your workspace directory (choose your preferred location)
mkdir <YOUR_WORKSPACE_ROOT>   # e.g., ~/WG3 or C:\Projects\WG3

# 2. Clone all WG3 project repositories into this directory
cd <YOUR_WORKSPACE_ROOT>
# Clone all projects listed in shared/configuration.md
git clone <wg-client-repo>
git clone <wg-cms-api-repo>
# ... etc.

# 3. Clone this workflows repository to your preferred location
# Common locations:
#   - Antigravity: ~/.gemini/antigravity/global_workflows
#   - General: ~/ai-workflows or C:\ai-workflows
git clone <this-repo> <YOUR_WORKFLOWS_PATH>
```

## Step 2: Configure Workspace Root

- [ ] Open [`shared/configuration.md`](../shared/configuration.md)
- [ ] Note down your local projects directory (this is your `WORKSPACE_ROOT`)

| Platform | Example `WORKSPACE_ROOT` |
|----------|-------------------------|
| Windows | `C:\My Projects\WG3` |
| macOS | `/Users/yourname/WG3` |
| Linux | `/home/yourname/WG3` |

> [!NOTE]
> All paths in the agents use variable substitution (e.g., `${WORKSPACE_ROOT}/wg-client`).
> The agents resolve these at runtime based on your VS Code workspace.

## Step 3: VS Code Workspace Setup

- [ ] Open VS Code
- [ ] **File → Add Folder to Workspace...** for each project listed in [`shared/configuration.md`](../shared/configuration.md)
- [ ] **File → Save Workspace As...** → Save as `WG3.code-workspace`

## Step 4: Verify Setup

Run this quick verification:

- [ ] All projects from [`shared/configuration.md`](../shared/configuration.md) visible in VS Code Explorer sidebar
- [ ] Each project has `.ai-instructions/` folder (if previously mapped)
- [ ] Invoke `/engineering-agent` in your AI assistant to test

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
│   └── configuration.md              # 🔑 Project registry (SINGLE SOURCE OF TRUTH)
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

## 🔧 Configuration Values

### Files You May Need to Update

| File | What to Configure | When |
|------|-------------------|------|
| [`shared/configuration.md`](../shared/configuration.md) | Add/remove projects | When new projects are added to the team |
| [`engineering/configuration.md`](../engineering/configuration.md) | Jira/Confluence settings | If Atlassian instance changes |

### Atlassian Configuration

See [`engineering/configuration.md`](../engineering/configuration.md) for current Jira/Confluence settings (project keys, space keys, folder IDs).

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
| `.ai-instructions/` not found | Run `/map-codebase-agent` on the project first |
| Cross-project context missing | Run `/system-architecture-agent` to generate `system-architecture/` |
| Jira/Confluence errors | Check `engineering/configuration.md` for correct Atlassian settings |
