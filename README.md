# Global Workflows

AI agent system for software development lifecycle — from specs to code to PR.

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

## ⚡ Quick Start

```bash
# 1. Clone this repo to your workflows directory
#    macOS/Linux: ~/.gemini/antigravity/global_workflows
#    Windows:     %USERPROFILE%\.gemini\antigravity\global_workflows
git clone <this-repo> <your-workflows-path>

# 2. Configure your environment
#    Edit: shared/configuration.md (required)
#       Set: SYSTEM_NAME, WORKSPACE_ROOT
#       Set: ATLASSIAN_CLOUD_ID, JIRA_PROJECT_KEY, CONFLUENCE_SPACE_KEY (if using Atlassian)
#       Update: Registered Projects table
#    Edit: engineering/configuration.md (if using Jira/Confluence)
#       Set: PRODUCT_SPECS_FOLDER_ID, TECH_SPECS_FOLDER_ID
#       Set: Jira Required Custom Fields (if your Jira mandates fields on issue creation)

# 3. Open your projects and workflows in VS Code/Cursor

# 4. Run an agent
/engineering-agent
```

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
#    macOS/Linux: ~/.gemini/antigravity/global_workflows
#    Windows:     %USERPROFILE%\.gemini\antigravity\global_workflows
git clone <this-repo> <your-workflows-path>

# 2. Clone all project repos to a common directory
mkdir ~/projects/MySystem  # or C:\Projects\MySystem on Windows
cd ~/projects/MySystem
git clone <project-1> && git clone <project-2> ...

# 3. Configure paths
#    Edit: shared/configuration.md
#       Set: SYSTEM_NAME, WORKSPACE_ROOT
#       Set: ATLASSIAN_CLOUD_ID, JIRA_PROJECT_KEY, CONFLUENCE_SPACE_KEY (if using Atlassian)
#       Update: Registered Projects table
#    Edit: engineering/configuration.md (if using Jira/Confluence)
#       Set: PRODUCT_SPECS_FOLDER_ID, TECH_SPECS_FOLDER_ID
#       Set: Jira Required Custom Fields (add your org's mandatory fields)

# 4. Install MCP servers (see readme/setup_instructions.md)

# 5. Verify
/engineering-agent  # Should load without errors
```

### Migration Checklist

- [ ] Clone `global_workflows` repo
- [ ] Clone all project repos to single parent directory
- [ ] Update `shared/configuration.md`: SYSTEM_NAME, WORKSPACE_ROOT, Projects table
- [ ] *(If using Atlassian)* Set Cloud ID, Jira Project Key, Confluence Space Key
- [ ] *(If using Atlassian)* Update `engineering/configuration.md`: Folder IDs
- [ ] *(If Jira requires custom fields)* Configure "Jira Required Custom Fields" section
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
