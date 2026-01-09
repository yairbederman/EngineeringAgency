# Atlassian Configuration

> **Single Source of Truth**: All Jira and Confluence settings for all agents.
> Configure this file once per organization/team.
>
> **Related Files**:
> - `shared/configuration.md` — Global project settings
> - `shared/mcp-config.md` — MCP tool definitions

---

## ⚠️ Setup Required

> [!IMPORTANT]
> **Before first use**, replace all placeholder values (`<PLACEHOLDER>`) with your Atlassian instance settings.
> Skip this file entirely if working in local-only mode (no Jira/Confluence).

---

## Cloud Connection

> **How to find Cloud ID**: Use `mcp0_getAccessibleAtlassianResources` tool to discover your Cloud ID.

| Variable | Value | Description |
|----------|-------|-------------|
| `${ATLASSIAN_CLOUD_ID}` | `<YOUR_ORG>.atlassian.net` | Base Cloud ID (e.g., `mycompany.atlassian.net`) |
| `${JIRA_PROJECT_KEY}` | `<PROJECT_KEY>` | Default Jira Project Key (e.g., `PROJ`) |
| `${CONFLUENCE_SPACE_KEY}` | `<SPACE_KEY>` | Default Confluence Space Key (e.g., `DOCS`) |

---

## Confluence Folders

> **Instructions**: Replace placeholder values with your Confluence folder IDs.
> To find folder IDs, navigate to the folder in Confluence and extract the ID from the URL.

| Variable | Setting | Value |
|----------|---------|-------|
| `${PRODUCT_SPECS_FOLDER_ID}` | Product Specs Folder | `<PRODUCT_SPECS_FOLDER_ID>` |
| `${TECH_SPECS_FOLDER_ID}` | Tech Specs Folder | `<TECH_SPECS_FOLDER_ID>` |

---

## Jira Custom Fields (Optional)

> **Purpose**: Define custom fields YOUR Jira mandates when creating issues.
> Each organization has different required fields — add yours below.
>
> **Instructions**:
> 1. Identify which custom fields your Jira requires (check issue creation screens)
> 2. Add one row per field using the format below
> 3. Delete the example row when done
> 4. Leave table empty if no custom fields are required

| Variable | Field Name | Field ID | Default Value | Description |
|----------|------------|----------|---------------|-------------|
| `${CROSS_PROJECT_IMPACT_FIELD}` | Cross-Project Impact | `customfield_XXXXX` | `<VALUE_ID>` | Dependencies on other teams |
| `${CUSTOM_FIELD_2}` | _Your Field Name_ | `customfield_XXXXX` | `<value or N/A>` | _Delete or replace this row_ |

> [!TIP]
> **How to find Field IDs**: Navigate to Jira Admin → Issues → Custom Fields → Click on field → ID is in the URL.

---

## Used By

- `/engineering-agent` — Issue creation, Confluence page publishing
- `/manager-agent` — Sprint metrics, issue queries
