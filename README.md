# Global Workflows

AI agent system for software development lifecycle — from specs to code to PR.

## 📖 Documentation

| Doc | Purpose |
|-----|---------|
| [Setup Guide](readme/setup_instructions.md) | Installation & configuration |
| [Agents Overview](readme/agents_diagram.md) | Workflow hierarchy & when to run each |
| [Manager Guide](readme/manager-usage-guide.md) | Sprint health, risk, status reporting |

## ⚡ Quick Start

```bash
# 1. Clone this repo to your workflows directory
git clone <this-repo> ~/.gemini/antigravity/global_workflows

# 2. Configure your environment
#    Edit: shared/configuration.md (required)
#    Edit: engineering/configuration.md (for Jira/Confluence)

# 3. Open your projects workspace in VS Code/Cursor

# 4. Run an agent
/engineering-agent
```

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
