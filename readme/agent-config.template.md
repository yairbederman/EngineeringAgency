# Workspace AI Configuration Template

> **Copy this file** to your workspace's `Agent_Config/` folder as `agent-config.md` and fill in the values.
>
> **Location**: `${YOUR_WORKSPACE_ROOT}/Agent_Config/agent-config.md`

---

## Storage Mode

> **Choose your storage backend**: `atlassian` for Jira/Confluence integration, `local` for file-based specs.

| Variable | Value | Options |
|----------|-------|---------|
| `STORAGE_BACKEND` | `local` | `atlassian` or `local` |
| `LOCAL_SPECS_PATH` | `./.specs` | Path for local spec storage (only if `local` mode) |

---

## Workspace Settings

> **Required**: These values identify your workspace and system.

| Variable | Value | Description |
|----------|-------|-------------|
| `SYSTEM_NAME` | `<YOUR_SYSTEM_NAME>` | System identifier (e.g., `MySystem`, `ClientProject`) |
| `WORKSPACE_ROOT` | `.` | Root directory (usually `.` for current workspace) |
| `SYSTEM_ARCH_OUTPUT_ROOT` | `${WORKSPACE_ROOT}/${SYSTEM_NAME}-system-architecture` | Output path for system architecture docs |

---

## Atlassian Configuration (Optional)

> **Skip if using `local` storage mode.**
>
> **How to find Cloud ID**: Use `mcp0_getAccessibleAtlassianResources` tool.

### Cloud Connection

| Variable | Value | Example |
|----------|-------|---------|
| `ATLASSIAN_CLOUD_ID` | `<YOUR_ORG>.atlassian.net` | `mycompany.atlassian.net` |
| `JIRA_PROJECT_KEY` | `<PROJECT_KEY>` | `PROJ` |
| `CONFLUENCE_SPACE_KEY` | `<SPACE_KEY>` | `DOCS` |

### Confluence Folders

> **How to find Folder IDs**: Navigate to folder in Confluence → ID is in URL.

| Variable | Value | Description |
|----------|-------|-------------|
| `PRODUCT_SPECS_FOLDER_ID` | `<FOLDER_ID>` | Folder for Product Specs |
| `TECH_SPECS_FOLDER_ID` | `<FOLDER_ID>` | Folder for Tech Specs |

### Jira Custom Fields (Optional)

> **Only add fields YOUR Jira mandates** on issue creation. Delete example rows.

| Variable | Field Name | Field ID | Default Value | Description |
|----------|------------|----------|---------------|-------------|
| _Example:_ `TEAM_FIELD` | Team | `customfield_12345` | `Backend` | _Delete this example row_ |

---

## Registered Projects

> **Auto-populated by `/map-codebase-agent`** — Run the agent on each project to register here.
>
> **Manual registration**: Add rows below using the format shown.

| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `PROJECT_1` | my-frontend | frontend | client | `./my-frontend` |
| `PROJECT_2` | my-api | backend | api | `./my-api` |

### Project Selection Criteria

| Work Type | Example Variable |
|-----------|------------------|
| Frontend work (UI, forms, displays) | `PROJECT_1` |
| API/Backend services | `PROJECT_2` |
| Shared libraries/utilities | As applicable |
| Full-stack features | Multiple projects |

---

## Example: Complete Configuration

```markdown
# agent-config.md for MyCompany Project

## Storage Mode
| Variable | Value |
|----------|-------|
| `STORAGE_BACKEND` | `atlassian` |

## Workspace Settings
| Variable | Value |
|----------|-------|
| `SYSTEM_NAME` | `MyCompany` |
| `WORKSPACE_ROOT` | `.` |

## Atlassian Configuration
| Variable | Value |
|----------|-------|
| `ATLASSIAN_CLOUD_ID` | `mycompany.atlassian.net` |
| `JIRA_PROJECT_KEY` | `MC` |
| `CONFLUENCE_SPACE_KEY` | `DOCS` |
| `PRODUCT_SPECS_FOLDER_ID` | `12345678` |
| `TECH_SPECS_FOLDER_ID` | `87654321` |

## Registered Projects
| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `PROJECT_WEB` | mc-web | frontend | client | `./mc-web` |
| `PROJECT_API` | mc-api | backend | api | `./mc-api` |
| `PROJECT_CORE` | mc-core | library | shared | `./mc-core` |
```

---

## Used By

- `/engineering-agent` — Feature lifecycle, issue creation
- `/map-codebase-agent` — Project registration
- `/system-architecture-agent` — Cross-project scanning
- `/manager-agent` — Sprint metrics and reporting
- `/system-health-agent` — Health diagnostics and project status
