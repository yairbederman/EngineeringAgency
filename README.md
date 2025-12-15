# Global Workflows

AI agent workflows for code generation, architecture extraction, and feature lifecycle management.

---

## 🚀 New Developer Setup

### Prerequisites

- [ ] VS Code installed with GitHub Copilot extension
- [ ] Access to WG3 project repositories
- [ ] Atlassian account (for Jira/Confluence integration)
- [ ] Figma account (for design token extraction)

### Step 0: Configure MCP Servers

The agents require two MCP (Model Context Protocol) servers for full functionality:

#### Atlassian MCP Server (Required)

Provides Jira and Confluence integration for epics, tasks, and specs.

- [ ] Install the Atlassian MCP server extension
- [ ] Configure authentication with your Atlassian account
- [ ] Verify access to the WG3 Jira project and Confluence space (see [`engineering/configuration.md`](engineering/configuration.md) for keys)

**Test**: Run `mcp_atlassian-mcp-server_atlassianUserInfo` to verify your connection.

#### Figma MCP Server (Optional - Required for Frontend Tasks)

Provides design token extraction for pixel-perfect UI implementation.

- [ ] Install the Figma MCP server extension
- [ ] Configure authentication with your Figma account
- [ ] Verify access to WG3 design files

**Test**: Use `mcp_figma-dev-mode-mcp-server_get_design_context` on a Figma link to verify.

> [!TIP]
> MCP servers are configured in your VS Code settings or `.vscode/mcp.json`.
> Ask your team lead for the MCP server configuration files.

### Step 1: Clone Repositories

```bash
# 1. Create your workspace directory (choose your preferred location)
mkdir <YOUR_WORKSPACE_ROOT>   # e.g., ~/WG3 or C:\Projects\WG3

# 2. Clone all WG3 project repositories into this directory
cd <YOUR_WORKSPACE_ROOT>
# Clone all projects listed in shared/projects.md
git clone <wg-client-repo>
git clone <wg-cms-api-repo>
# ... etc.

# 3. Clone this workflows repository
git clone <this-repo> ~/.gemini/antigravity/global_workflows
```

### Step 2: Configure Workspace Root

- [ ] Open [`shared/projects.md`](shared/projects.md)
- [ ] Note down your local projects directory (this is your `WORKSPACE_ROOT`)

| Platform | Example `WORKSPACE_ROOT` |
|----------|-------------------------|
| Windows | `C:\My Projects\WG3` |
| macOS | `/Users/yourname/WG3` |
| Linux | `/home/yourname/WG3` |

> [!NOTE]
> All paths in the agents use variable substitution (e.g., `${WORKSPACE_ROOT}/wg-client`).
> The agents resolve these at runtime based on your VS Code workspace.

### Step 3: VS Code Workspace Setup

- [ ] Open VS Code
- [ ] **File → Add Folder to Workspace...** for each project listed in [`shared/projects.md`](shared/projects.md)
- [ ] **File → Save Workspace As...** → Save as `WG3.code-workspace`

### Step 4: Verify Setup

Run this quick verification:

- [ ] All projects from [`shared/projects.md`](shared/projects.md) visible in VS Code Explorer sidebar
- [ ] Each project has `.ai-instructions/` folder (if previously mapped)
- [ ] Type `/engineering-agent` in Copilot chat to test

---

## 📁 Directory Structure

```
global_workflows/
├── README.md                         # This file
├── shared/
│   └── projects.md                   # 🔑 Project registry (SINGLE SOURCE OF TRUTH)
│
├── engineering-agent.md              # Feature lifecycle workflow
├── engineering/                      # Configuration & mode files
│   ├── configuration.md              # 🔧 Atlassian config (Jira/Confluence)
│   ├── core-rules.md                 # Agent behavior rules
│   ├── modes/                        # Mode-specific instructions
│   └── templates/                    # Epic, Tech Spec, Task templates
│
├── map-codebase-agent.md             # Project AI instructions generator
├── mapcodebase/                      # Phase files for extraction
│   └── configuration.md              # Output paths
│
├── system-architecture-agent.md      # Cross-project architecture generator
└── system-architecture/              # Phase files for system docs
    └── configuration.md              # Output paths
```

---

## 🔧 Configuration Values

### Files You May Need to Update

| File | What to Configure | When |
|------|-------------------|------|
| [`shared/projects.md`](shared/projects.md) | Add/remove projects | When new projects are added to the team |
| [`engineering/configuration.md`](engineering/configuration.md) | Jira/Confluence settings | If Atlassian instance changes |

### Atlassian Configuration

See [`engineering/configuration.md`](engineering/configuration.md) for current Jira/Confluence settings (project keys, space keys, folder IDs).

---

## 🔄 Workflow Hierarchy

```
┌─────────────────────────┐
│  /map-codebase-agent    │  ← Run per project (generates .ai-instructions/)
└───────────┬─────────────┘
            ↓
┌─────────────────────────┐
│/system-architecture-agent│ ← Run once (aggregates all projects)
└───────────┬─────────────┘
            ↓
┌─────────────────────────┐
│   /engineering-agent    │  ← Daily feature work (uses all above)
└─────────────────────────┘
```

### When to Run Each Workflow

| Workflow | Trigger | Output |
|----------|---------|--------|
| `/map-codebase-agent` | Project structure changes significantly | `.ai-instructions/` in project |
| `/system-architecture-agent` | New project added or major API changes | `system-architecture/` docs |
| `/engineering-agent` | Any feature work, bug fixes | Jira tasks, code, tests |

---

## ✅ Quick Start (Daily Use)

### Start Feature Work
```
/engineering-agent
```
Guides you through: ProductSpecReview → FeaturePlanning → TechSpec → Implementation

### Refresh Project Knowledge (When Needed)
```
/map-codebase-agent
```
Run on a specific project when its structure changes.

### Refresh System Architecture (When Needed)
```
/system-architecture-agent
```
Run after adding new projects or major cross-service changes.

---

## ➕ Adding New Projects

- [ ] Add project row to [`shared/projects.md`](shared/projects.md)
- [ ] Run `/map-codebase-agent` on the new project
- [ ] Run `/system-architecture-agent` to update cross-project docs
- [ ] Add project folder to VS Code workspace

---

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| Agent can't find project | Verify project is in `shared/projects.md` and added to VS Code workspace |
| `.ai-instructions/` not found | Run `/map-codebase-agent` on the project first |
| Cross-project context missing | Run `/system-architecture-agent` to generate `system-architecture/` |
| Jira/Confluence errors | Check `engineering/configuration.md` for correct Atlassian settings |