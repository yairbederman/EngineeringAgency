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

## Tech Stack Configuration (Optional)

> **Purpose**: Define the technology stack for this project. Used by `/engineering-agent` when generating Tech Specs and implementation tasks.
>
> **When to fill**: For new projects or when establishing tech standards. Leave blank to prompt for selection during planning.

### Project Type

| Variable | Value | Options |
|----------|-------|---------|
| `PROJECT_TYPE` | `<TYPE>` | `web-app`, `api`, `fullstack`, `mobile`, `cli`, `library` |

### Frontend Stack (if applicable)

> Skip this section for backend-only projects.

| Variable | Value | Options |
|----------|-------|---------|
| `FRONTEND_FRAMEWORK` | `<FRAMEWORK>` | `react`, `vue`, `angular`, `svelte`, `none` |
| `FRONTEND_META_FRAMEWORK` | `<META>` | `nextjs`, `nuxt`, `remix`, `astro`, `vite`, `none` |
| `FRONTEND_STYLING` | `<STYLING>` | `tailwind`, `css-modules`, `styled-components`, `vanilla-css`, `sass` |
| `FRONTEND_STATE` | `<STATE>` | `zustand`, `redux`, `jotai`, `pinia`, `none` |

### Backend Stack (if applicable)

> Skip this section for frontend-only projects.

| Variable | Value | Options |
|----------|-------|---------|
| `BACKEND_RUNTIME` | `<RUNTIME>` | `node`, `python`, `go`, `dotnet`, `java`, `rust` |
| `BACKEND_FRAMEWORK` | `<FRAMEWORK>` | `express`, `fastify`, `nestjs`, `hono`, `fastapi`, `gin`, `none` |
| `API_STYLE` | `<API>` | `rest`, `graphql`, `trpc`, `grpc` |

### Database Stack (if applicable)

| Variable | Value | Options |
|----------|-------|---------|
| `DATABASE_PRIMARY` | `<DB>` | `postgres`, `mysql`, `mongodb`, `supabase`, `firebase`, `sqlite`, `none` |
| `DATABASE_ORM` | `<ORM>` | `prisma`, `drizzle`, `typeorm`, `sqlalchemy`, `gorm`, `none` |

### Infrastructure

| Variable | Value | Options |
|----------|-------|---------|
| `DEPLOYMENT_TARGET` | `<TARGET>` | `vercel`, `aws`, `gcp`, `azure`, `docker`, `self-hosted` |
| `CI_CD` | `<CI>` | `github-actions`, `gitlab-ci`, `bitbucket-pipelines`, `none` |

### Testing Stack

| Variable | Value | Options |
|----------|-------|---------|
| `TEST_FRAMEWORK` | `<TEST>` | `jest`, `vitest`, `pytest`, `go-test`, `xunit` |
| `E2E_FRAMEWORK` | `<E2E>` | `playwright`, `cypress`, `none` |

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

## Tech Stack Configuration
| Variable | Value |
|----------|-------|
| `PROJECT_TYPE` | `fullstack` |
| `FRONTEND_FRAMEWORK` | `react` |
| `FRONTEND_META_FRAMEWORK` | `nextjs` |
| `FRONTEND_STYLING` | `tailwind` |
| `BACKEND_RUNTIME` | `node` |
| `BACKEND_FRAMEWORK` | `nestjs` |
| `API_STYLE` | `rest` |
| `DATABASE_PRIMARY` | `postgres` |
| `DATABASE_ORM` | `prisma` |
| `DEPLOYMENT_TARGET` | `vercel` |
| `TEST_FRAMEWORK` | `vitest` |
| `E2E_FRAMEWORK` | `playwright` |
```

---

## Used By

- `/engineering-agent` — Feature lifecycle, issue creation
- `/map-codebase-agent` — Project registration
- `/system-architecture-agent` — Cross-project scanning
- `/manager-agent` — Sprint metrics and reporting
