# Lognet Agent Configuration

> **Purpose**: Single source of truth for all environment-specific configuration.
> When migrating to a new environment, update the paths in this file only.

---

## Installation

After cloning, `AGENT_ROOT` is configured to use relative paths (`./feature-lifecycle`) by default. No update is required if the standard directory structure is maintained.

---

## Agent Paths

| Variable | Value | Description |
|----------|-------|-------------|
| `AGENT_ROOT` | `./feature-lifecycle` | Base path for all agent files (Relative to global_workflows root) |

### Agent Files (relative to AGENT_ROOT)

| File | Relative Path |
|------|---------------|
| Core Rules | `core-rules.md` |
| Workflow Validation | `workflow-validation.md` |
| **Modes** | |
| Product Spec Review | `modes/planning/product-spec-review.md` |
| Feature Planning | `modes/planning/feature-planning.md` |
| Tech Spec | `modes/planning/tech-spec.md` |
| Task Planning | `modes/planning/task-planning.md` |
| Implementation | `modes/implementation.md` |
| BugFix | `modes/bugfix.md` |
| **Templates** | |
| Epic Template | `templates/epic.md` |
| Tech Spec Template | `templates/tech-spec.md` |
| Task Template | `templates/task.md` |

---

## Atlassian Configuration

### Jira

| Variable | Setting | Value |
|----------|---------|-------|
| `${ATLASSIAN_CLOUD_ID}` | Cloud ID | `lognetsystems.atlassian.net` |
| `${JIRA_PROJECT_KEY}` | Project Key | `W0` |
| | Project Board | [W0 Board](https://lognetsystems.atlassian.net/jira/software/projects/W0) |

### Confluence

| Variable | Setting | Value |
|----------|---------|-------|
| `${CONFLUENCE_SPACE_KEY}` | Space Key | `WGPro30` |
| `${PRODUCT_SPECS_FOLDER_ID}` | Product Specs Folder ID | `260177923` |
| `${TECH_SPECS_FOLDER_ID}` | Tech Specs Folder ID | `259883024` |
| | Product Specs URL | [Folder](https://lognetsystems.atlassian.net/wiki/spaces/WGPro30/folder/260177923) |
| | Tech Specs URL | [Folder](https://lognetsystems.atlassian.net/wiki/spaces/WGPro30/folder/259883024) |

### Custom Fields

| Variable | Field | ID | Value | Description |
|----------|-------|-----|-------|-------------|
| `${CROSS_PROJECT_IMPACT_FIELD}` | Cross-Project Impact | `customfield_10225` | `10635` (= "None") | Required for all tasks |

---

## Workspace Projects

| Variable | Name | Role | Relative Path |
|----------|------|------|---------------|
| `${PROJECT_FRONTEND}` | wg-client | Frontend (React) | `./wg-client` |
| `${PROJECT_CMS_API}` | wg-cms-api | CMS Backend | `./wg-cms-api` |
| `${PROJECT_DATA_API}` | wg-data-api | Data Backend | `./wg-data-api` |

> **Adding new projects**: Add a row to this table and update agent instructions that reference project roles.

### Project Selection Criteria

| Work Type | Use Project Variable |
|-----------|---------------------|
| Frontend work (UI, forms, displays) | `${PROJECT_FRONTEND}` |
| CMS/Admin operations | `${PROJECT_CMS_API}` |
| Data processing, external APIs | `${PROJECT_DATA_API}` |
| Full-stack features | Multiple projects |

### Per-Project Config Files

| Variable | Path | Description |
|----------|------|-------------|
| `${COPILOT_INSTRUCTIONS_PATH}` | `.ai-instructions/copilot-instructions.md` | Architecture & patterns |
| `${FILE_CATEGORIZATION_PATH}` | `.ai-instructions/analysis/file-categorization.json` | Component organization |
| `${DESIGN_TOKENS_PATH}` | `tailwind.config.js` or `theme.ts` | Design system tokens (frontend) |

---

## MCP Tool References

### Atlassian (mcp0_*)

| Tool | Purpose |
|------|---------|
| `mcp0_getConfluencePage` | Read Confluence pages |
| `mcp0_createConfluencePage` | Create new Confluence pages |
| `mcp0_updateConfluencePage` | Update Confluence pages |
| `mcp0_createConfluenceFooterComment` | Add comments to pages |
| `mcp0_getJiraIssue` | Read Jira issues |
| `mcp0_createJiraIssue` | Create Jira issues (Epic/Task) |
| `mcp0_editJiraIssue` | Update Jira issues |
| `mcp0_addCommentToJiraIssue` | Add comments to issues |
| `mcp0_transitionJiraIssue` | Change issue status |
| `mcp0_searchJiraIssuesUsingJql` | Search issues with JQL |

### Figma (mcp1_*)

| Tool | Purpose |
|------|---------|
| `mcp1_get_design_context` | Extract design tokens from Figma frame |
| `mcp1_get_screenshot` | Capture screenshot of Figma node |
| `mcp1_get_metadata` | Get node metadata (structure overview) |
| `mcp1_get_variable_defs` | Get variable definitions (colors, spacing) |

