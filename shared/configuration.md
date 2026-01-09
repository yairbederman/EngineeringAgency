# Shared Configuration

> **Single Source of Truth**: All agents reference this file for global settings and the project list.
> When adding/removing projects or changing global constants, update **only this file**.
>
> **Global Tool Config**: See `shared/mcp-config.md` for MCP tool definitions.
> **Error Handling**: See `shared/error-codes.md` for structured error codes.

---

## ⚠️ Setup Required

> [!IMPORTANT]
> **Before first use**, you MUST configure the values in this file for your environment.
> All placeholder values (`<PLACEHOLDER>`) must be replaced with your organization's settings.
> See [readme/setup_instructions.md](../readme/setup_instructions.md) for detailed setup guide.

---

## Global Constants

| Variable | Value | Description |
|----------|-------|-------------|
| `SYSTEM_NAME` | `<YOUR_SYSTEM_NAME>` | Global system identifier (e.g., `MySystem`) |
| `SYSTEM_ARCH_OUTPUT_ROOT` | `${WORKSPACE_ROOT}/${SYSTEM_NAME}-system-architecture` | Contract path for system architecture artifacts |
| `GLOBAL_WORKFLOWS_ROOT` | `.` | Root path for global_workflows directory |

---

## Atlassian Configuration (Jira/Confluence)

> **📁 Separate File**: [atlassian-config.md](atlassian-config.md)
>
> All Jira and Confluence settings are in a dedicated file:
> - Cloud ID, Jira Project Key, Confluence Space Key
> - Confluence Folder IDs (Product Specs, Tech Specs)
> - Jira Custom Fields (optional)
>
> Skip if working in local-only mode (no Atlassian integration).

---

## Workspace Configuration

> [!IMPORTANT]
> **Team Setup Required**: Each team member must set `WORKSPACE_ROOT` to their local projects directory.

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `${WORKSPACE_ROOT}` | Root directory containing all projects | `C:/Projects/MySystem` or `/home/user/projects` |

---

## Registered Projects

> **📁 Separate File**: [projects.md](projects.md)
>
> Project registry (auto-populated by `/map-codebase-agent`).
> When you run the agent on a project, it automatically registers there.

---

## Used By

- `/engineering-agent` – for project selection during TechSpec/TaskPlanning
- `/system-architecture-agent` – for scanning all projects
- `/map-codebase-agent` – target project auto-registration
