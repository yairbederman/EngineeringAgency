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

## Jira Advanced Configuration

> **Purpose**: Customize interaction with your Jira instance. All settings in this section are **optional-advanced**.
> Copy this section to your local configuration only if you need to override defaults.

### Limitations

| Variable | Default | Description |
|----------|---------|-------------|
| `JIRA_MAX_RESULTS` | `50` | Maximum results to fetch in JQL queries (System Safe Limit) |
| `JIRA_TIMEOUT_SECONDS` | `30` | API timeout duration for slow instances |

### Overrides

> **Use Case**: When your Jira workflow requires specific transition IDs that logic cannot auto-detect.

| Variable | Description | Example |
|----------|-------------|---------|
| `FORCE_TRANSITION_IDS` | JSON map of status names to transition IDs | `{"In Progress": "31", "Done": "41"}` |
| `STATUS_MAPPING` | JSON map of agent status to Jira status | `{"Review": "In Code Review"}` |

### Jira Required Custom Fields (User-Defined)

> **Purpose**: Define custom fields that YOUR Jira instance mandates when creating issues.
> Each organization has different required fields — add yours below.
>
> **Instructions**:
> 1. Identify which custom fields your Jira requires (check issue creation screens)
> 2. Add one row per field using the format below
> 3. Delete the example row when done
> 4. Leave table empty if no custom fields are required

| Variable | Field Name | Field ID | Default Value | Description |
|----------|------------|----------|---------------|-------------|
| `${CROSS_PROJECT_IMPACT_FIELD}` | Cross-Project Impact | `customfield_XXXXX` | `<VALUE_ID>` | Dependencies on other teams (common mandatory field) |
| `${CUSTOM_FIELD_2}` | _Your Field Name_ | `customfield_XXXXX` | `<value or N/A>` | _Add more fields as needed_ |

> [!TIP]
> **How to find Field IDs**: Navigate to Jira Admin → Issues → Custom Fields → Click on field → ID is in the URL.
>
> **Example configurations**:
> - Cross-Project Impact: `customfield_12345` → Default: `None`
> - Team: `customfield_10001` → Default: `Backend`
> - Sprint: `customfield_10020` → Default: N/A (set at planning time)

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
