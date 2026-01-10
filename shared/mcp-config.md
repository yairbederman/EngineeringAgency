# MCP Tool Configuration

> **Source of Truth**: Maps logical tool variables to actual MCP tool calls.
> **Usage**: Use `${VARIABLE_NAME}` in workflow files. The Agent will resolve this to the tool name.

---

## Atlassian MCP (`atlassian-mcp-server`)

| Variable | Tool Name | Description |
|----------|-----------|-------------|
| `${MCP_ATLASSIAN_GET_ISSUE}` | `mcp_atlassian-mcp-server_getJiraIssue` | Get Jira Issue details |
| `${MCP_ATLASSIAN_CREATE_ISSUE}` | `mcp_atlassian-mcp-server_createJiraIssue` | Create new Jira Issue |
| `${MCP_ATLASSIAN_EDIT_ISSUE}` | `mcp_atlassian-mcp-server_editJiraIssue` | Update Jira Issue fields |
| `${MCP_ATLASSIAN_TRANSITION_ISSUE}` | `mcp_atlassian-mcp-server_transitionJiraIssue` | Change Jira Issue status |
| `${MCP_ATLASSIAN_ADD_COMMENT}` | `mcp_atlassian-mcp-server_addCommentToJiraIssue` | Add comment to Jira Issue |
| `${MCP_ATLASSIAN_SEARCH_JQL}` | `mcp_atlassian-mcp-server_searchJiraIssuesUsingJql` | Search Jira issues with JQL |
| `${MCP_ATLASSIAN_ADD_FOOTER_COMMENT}` | `mcp_atlassian-mcp-server_createConfluenceFooterComment` | Add footer comment to Confluence page |
| `${MCP_ATLASSIAN_GET_ISSUE_LINKS}` | `mcp_atlassian-mcp-server_getJiraIssueRemoteIssueLinks` | Get remote links for Issue |
| `${MCP_ATLASSIAN_GET_PAGE}` | `mcp_atlassian-mcp-server_getConfluencePage` | Get Confluence page content |
| `${MCP_ATLASSIAN_UPDATE_PAGE}` | `mcp_atlassian-mcp-server_updateConfluencePage` | Update Confluence page |
| `${MCP_ATLASSIAN_GET_USER_INFO}` | `mcp_atlassian-mcp-server_atlassianUserInfo` | Get current user info |
| `${MCP_ATLASSIAN_GET_RESOURCES}` | `mcp_atlassian-mcp-server_getAccessibleAtlassianResources` | Get accessible resources (Cloud ID) |

## Figma MCP (`figma-dev-mode-mcp-server`)

| Variable | Tool Name | Description |
|----------|-----------|-------------|
| `${MCP_FIGMA_GET_DESIGN}` | `mcp_figma-dev-mode-mcp-server_get_design_context` | Extract design tokens & node data |
| `${MCP_FIGMA_GET_VARS}` | `mcp_figma-dev-mode-mcp-server_get_variable_defs` | Get design system variables |
| `${MCP_FIGMA_GET_METADATA}` | `mcp_figma-dev-mode-mcp-server_get_metadata` | Get node metadata (structure overview) |
| `${MCP_FIGMA_GET_SCREENSHOT}` | `mcp_figma-dev-mode-mcp-server_get_screenshot` | Capture node screenshot |

## Generic / Other

| Variable | Tool Name | Description |
|----------|-----------|-------------|
| `${MCP_BROWSER_ACTION}` | `browser_subagent` | Browser automation (fallback) |
