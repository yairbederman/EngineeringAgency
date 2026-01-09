# Project Registry

> **Auto-populated by `/map-codebase-agent`** — When you run the agent on a new project, it automatically adds an entry here.
>
> **Related Files**:
> - `shared/configuration.md` — Global settings (SYSTEM_NAME, WORKSPACE_ROOT)
> - `shared/atlassian-config.md` — Jira/Confluence settings

---

## Registered Projects

| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `${PROJECT_WG_CLIENT}` | wg-client | Frontend | Client | `${WORKSPACE_ROOT}/wg-client` |

---

## Adding New Projects

Simply run `/map-codebase-agent` on any project — it will:
1. Detect project type (Frontend/Backend/Library)
2. Ask for confirmation
3. Auto-add to this table
4. Generate `.ai-instructions/`

After adding projects, run `/system-architecture-agent` to update cross-project docs.

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

- `/engineering-agent` — Project selection during TaskPlanning
- `/system-architecture-agent` — Scanning all projects
- `/map-codebase-agent` — Auto-registration target
