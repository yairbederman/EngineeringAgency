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

## Atlassian Configuration

> **Cloud ID**: Use `mcp0_getAccessibleAtlassianResources` to discover your Cloud ID.

| Variable | Value | Description |
|----------|-------|-------------|
| `${ATLASSIAN_CLOUD_ID}` | `<YOUR_ORG>.atlassian.net` | Base Cloud ID (e.g., `mycompany.atlassian.net`) |
| `${JIRA_PROJECT_KEY}` | `<PROJECT_KEY>` | Default Project Key (e.g., `PROJ`) |
| `${CONFLUENCE_SPACE_KEY}` | `<SPACE_KEY>` | Default Space Key (e.g., `DOCS`) |

---

## Workspace Configuration

> [!IMPORTANT]
> **Team Setup Required**: Each team member must set `WORKSPACE_ROOT` to their local projects directory.

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `${WORKSPACE_ROOT}` | Root directory containing all projects | `C:/Projects/MySystem` or `/home/user/projects` |

---

## Registered Projects

> **Instructions**: Add your project entries below. Each project should have:
> - A unique variable name for reference in workflows
> - The actual folder name
> - Type: `Frontend`, `Backend`, `Library`, `Infrastructure`, etc.
> - A brief role description
> - Path relative to `${WORKSPACE_ROOT}`

| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `${PROJECT_EXAMPLE_1}` | `<project-name-1>` | Frontend | Web application | `${WORKSPACE_ROOT}/<project-name-1>` |
| `${PROJECT_EXAMPLE_2}` | `<project-name-2>` | Backend | API service | `${WORKSPACE_ROOT}/<project-name-2>` |
| `${PROJECT_EXAMPLE_3}` | `<project-name-3>` | Library | Shared utilities | `${WORKSPACE_ROOT}/<project-name-3>` |

> **Example entries** (replace with your actual projects):
> ```
> | `${PROJECT_FRONTEND}` | my-client | Frontend | React/Next.js web app | `${WORKSPACE_ROOT}/my-client` |
> | `${PROJECT_API}` | my-api | Backend | Spring Boot API | `${WORKSPACE_ROOT}/my-api` |
> | `${PROJECT_CORE}` | my-core | Library | Shared libraries | `${WORKSPACE_ROOT}/my-core` |
> ```

---

## Adding New Projects

1. Add a row to the table above
2. Run `/map-codebase-agent` on the new project
3. Run `/system-architecture-agent` to update cross-project docs

---

## Project Selection Criteria

> **Customize this table** based on your project structure.

| Work Type | Use Project Variable |
|-----------|---------------------|
| Frontend work (UI, forms, displays) | `${PROJECT_FRONTEND}` |
| API/Backend services | `${PROJECT_API}` |
| Shared libraries/utilities | `${PROJECT_CORE}` |
| Full-stack features | Multiple projects |

---

## Used By

- `/engineering-agent` – for project selection during TechSpec/TaskPlanning
- `/system-architecture-agent` – for scanning all projects
- `/map-codebase-agent` – target project validation
