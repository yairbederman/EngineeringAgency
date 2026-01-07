# Engineering Agent Configuration

> **Purpose**: Single source of truth for all environment-specific configuration.
> When migrating to a new environment, update the paths in this file only.

---

## ⚠️ Setup Required

> [!IMPORTANT]
> **Before first use**, configure the Atlassian folder IDs and custom fields for your Jira/Confluence instance.
> All placeholder values (`<PLACEHOLDER>`) must be replaced with your organization's settings.

---

## Installation

After cloning, `AGENT_ROOT` is configured to use relative paths (`./engineering`) by default. No update is required if the standard directory structure is maintained.

---

## Agent Paths

| Variable | Value | Description |
|----------|-------|-------------|
| `AGENT_ROOT` | `./engineering` | Base path for all agent files (Relative to global_workflows root) |

### Core Files (relative to AGENT_ROOT)

| File | Relative Path |
|------|---------------|
| Core Rules | `core-rules.md` |
| Workflow Validation | `workflow-validation.md` |
| Gates & Approvals | `modes/_gates.md` |

---

## Mode Registry (Source of Truth)

> [!IMPORTANT]
> **Single Source of Truth**: All mode-to-file and mode-to-persona mappings are defined here.
> Other files should reference this section, not duplicate it.

### Mode Mapping

| Mode | Category | Rules File | Persona |
|------|----------|------------|---------|
| **ProductSpecReview** | Planning | `modes/planning/product-spec-review.md` | `personas/product-manager.md` |
| **DesignAnalysis** | Planning | `modes/planning/design-analysis.md` | `personas/designer.md` |
| **FeaturePlanning** | Planning | `modes/planning/feature-planning.md` | `personas/system-architect.md` |
| **TechSpec** | Planning | `modes/planning/tech-spec-review.md` | `personas/system-architect.md` |
| **TaskPlanning** | Planning | `modes/planning/task-planning.md` | `personas/system-architect.md` |
| **FastTrack** | Execution | `modes/execution/fast-track.md` | Track-based ↓ |
| **Implementation** | Execution | `modes/execution/orchestrator.md` | Track-based ↓ |
| **BugFix** | BugFix | `modes/bugfix/orchestrator.md` | Track-based ↓ |
| **Hotfix** | BugFix | `modes/bugfix/hotfix.md` | Track-based ↓ |
| **PullRequest** | Completion | `modes/completion/pull-request.md` | `personas/system-architect.md` |
| **CodeReview** | Completion | `modes/completion/code-review.md` | `personas/system-architect.md` |

### Track-Based Persona Selection

For Execution/BugFix modes, persona is selected based on task type:

| Track | Indicators | Persona |
|-------|------------|---------|
| **Backend** | `api`, `service`, `controller`, `.service.`, `.controller.` | `personas/backend-developer.md` |
| **Frontend** | `ui`, `component`, `form`, `.tsx`, `.vue`, `.css` | `personas/frontend-developer.md` |
| **Full-Stack** | Both indicators present | Backend first, then Frontend |

### Supporting Files

| Purpose | Relative Path |
|---------|---------------|
| **Cross-Project** | `modes/cross-project.md` |
| **Testing Policy** | `modes/execution/_testing-policy.md` |
| **Figma Extraction** | `design/figma-extraction-protocol.md` |
| **Validation Checklist** | `modes/planning/_validation-checklist.md` |

### Templates

| Template | Relative Path |
|----------|---------------|
| Epic | `templates/epic.md` |
| Tech Spec | `templates/tech-spec.md` |
| Task (Backend) | `templates/task-backend.md` |
| Task (Frontend) | `templates/task-frontend.md` |
| Template Contracts | `templates/_template-contracts.md` |

---

## Atlassian Configuration

> **Source**: [shared/configuration.md](../shared/configuration.md)
>
> All Atlassian settings (Cloud ID, Jira Project Key, Confluence Space Key) are now centralized in the shared configuration file.

### Confluence Folders

> **Instructions**: Replace placeholder values with your Confluence folder IDs.
> To find folder IDs, navigate to the folder in Confluence and extract the ID from the URL.

| Variable | Setting | Value |
|----------|---------|-------|
| `${PRODUCT_SPECS_FOLDER_ID}` | Product Specs Folder ID | `<PRODUCT_SPECS_FOLDER_ID>` |
| `${TECH_SPECS_FOLDER_ID}` | Tech Specs Folder ID | `<TECH_SPECS_FOLDER_ID>` |
| | Product Specs URL | [Folder](https://${ATLASSIAN_CLOUD_ID}/wiki/spaces/${CONFLUENCE_SPACE_KEY}/folder/${PRODUCT_SPECS_FOLDER_ID}) |
| | Tech Specs URL | [Folder](https://${ATLASSIAN_CLOUD_ID}/wiki/spaces/${CONFLUENCE_SPACE_KEY}/folder/${TECH_SPECS_FOLDER_ID}) |

### Custom Fields

> **Instructions**: Configure your Jira custom field IDs below.
> To find custom field IDs, use Jira's REST API or check your Jira admin settings.
> Remove or add rows based on your Jira configuration.

| Variable | Field | ID | Value | Description |
|----------|-------|-----|-------|-------------|
| `${CROSS_PROJECT_IMPACT_FIELD}` | Cross-Project Impact | `<CUSTOM_FIELD_ID>` | `<DEFAULT_VALUE_ID>` | Required for all tasks (optional - remove if not used) |
| `${JIRA_RANK_FIELD}` | Rank | `<RANK_FIELD_ID>` | N/A | Used for task ordering (e.g., `customfield_10019`) |

> **Note**: Custom fields are organization-specific. If your Jira doesn't use these fields, you can:
> 1. Remove references to them in the workflow files
> 2. Leave placeholders and they will be ignored

---

## Workspace Projects

> **📁 Source**: [shared/configuration.md](../shared/configuration.md) – Single source of truth for all agents.
>
> Read the shared configuration file for project variables, names, types, roles, paths, and selection criteria.

### Per-Project Config Files

| Variable | Path | Description |
|----------|------|-------------|
| `${COPILOT_INSTRUCTIONS_PATH}` | `.ai-instructions/copilot-instructions.md` | Architecture & patterns |
| `${FILE_CATEGORIZATION_PATH}` | `.ai-instructions/analysis/file-categorization.json` | Component organization |
| `${DESIGN_TOKENS_PATH}` | `tailwind.config.js` or `theme.ts` | Design system tokens (frontend) |

### System Architecture Paths (Cross-Project Context)

> **Source**: Generated by `/system-architecture-agent`

| Variable | Path | Description |
|----------|------|-------------|
| `${SYSTEM_ARCH_OUTPUT}` | `${SYSTEM_ARCH_OUTPUT_ROOT}` | Root for generated cross-project docs (from shared config) |
| `${SERVICE_TOPOLOGY_PATH}` | `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json` | Service dependencies |
| `${CROSS_SERVICE_APIS_PATH}` | `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json` | Inter-service API contracts |
| `${UNIFIED_DOMAIN_MODEL_PATH}` | `${SYSTEM_ARCH_OUTPUT}/analysis/unified-domain-model.json` | Canonical entities |
| `${SYSTEM_ARCH_DOC_PATH}` | `${SYSTEM_ARCH_OUTPUT}/system-architecture.md` | Master system doc |

> **Note**: `${WORKSPACE_ROOT}` and `${SYSTEM_ARCH_OUTPUT_ROOT}` are defined in [shared/configuration.md](../shared/configuration.md).

---

## MCP Tool References

> **Source**: [shared/mcp-config.md](../shared/mcp-config.md)
>
> All MCP tool references are now centralized in the shared MCP configuration file.
