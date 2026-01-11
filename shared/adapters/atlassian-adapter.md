# Atlassian Adapter

> **Backend**: `atlassian` (default)  
> **Purpose**: Store epics, specs, and tasks in Jira/Confluence via MCP.

---

## Configuration

**Required** (from `agent-config.md`):

| Variable | Description |
|----------|-------------|
| `ATLASSIAN_CLOUD_ID` | Cloud instance (e.g., `mycompany.atlassian.net`) |
| `JIRA_PROJECT_KEY` | Project key for issue creation |
| `CONFLUENCE_SPACE_KEY` | Space for documentation (optional) |

**Optional**:

| Variable | Description |
|----------|-------------|
| `PRODUCT_SPECS_FOLDER_ID` | Confluence folder for Product Specs |
| `TECH_SPECS_FOLDER_ID` | Confluence folder for Tech Specs |
| Jira Custom Fields | Any mandatory fields for issue creation |

---

## MCP Tool Mapping

> **Reference**: See `shared/mcp-config.md` for full variable-to-tool mapping.

| Operation | MCP Variable | Parameters |
|-----------|--------------|------------|
| `createEpic` | `${MCP_ATLASSIAN_CREATE_ISSUE}` | `issueTypeName: "Epic"`, `projectKey`, `summary`, `description` |
| `getEpic` | `${MCP_ATLASSIAN_GET_ISSUE}` | `issueIdOrKey` |
| `updateEpic` | `${MCP_ATLASSIAN_EDIT_ISSUE}` | `issueIdOrKey`, `fields` |
| `createSpec` | `${MCP_ATLASSIAN_CREATE_ISSUE}` | `issueTypeName: "Task"`, `parent: epicId` |
| `createTask` | `${MCP_ATLASSIAN_CREATE_ISSUE}` | `issueTypeName: "Task"`, `parent: epicId` |
| `getTask` | `${MCP_ATLASSIAN_GET_ISSUE}` | `issueIdOrKey` |
| `getTasks` | `${MCP_ATLASSIAN_SEARCH_JQL}` | `jql: "parent = epicId"` |
| `updateTaskStatus` | `${MCP_ATLASSIAN_TRANSITION_ISSUE}` | `issueIdOrKey`, `transitionId` |
| `addTaskComment` | `${MCP_ATLASSIAN_ADD_COMMENT}` | `issueIdOrKey`, `body` |
| `linkItems` | `${MCP_ATLASSIAN_ADD_COMMENT}` | Or use remote issue links |

---

## Operations

### createEpic(title, content)

```
1. Read JIRA_PROJECT_KEY from agent-config.md
2. Read custom fields from agent-config.md (if any)
3. Call ${MCP_ATLASSIAN_CREATE_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - projectKey: ${JIRA_PROJECT_KEY}
   - issueTypeName: "Epic"
   - summary: title
   - description: content (markdown)
   - additional_fields: custom fields
4. Return: Issue key (e.g., "PROJ-123")
```

### getEpic(epicId)

```
1. Call ${MCP_ATLASSIAN_GET_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: epicId
2. Return: { id, title: summary, content: description, status }
```

### createSpec(title, content, epicId)

```
1. Call ${MCP_ATLASSIAN_CREATE_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - projectKey: ${JIRA_PROJECT_KEY}
   - issueTypeName: "Task"
   - parent: epicId
   - summary: "Tech Spec: " + title
   - description: content
2. Return: Issue key (e.g., "PROJ-124")
```

### createTask(title, content, epicId)

```
1. Call ${MCP_ATLASSIAN_CREATE_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - projectKey: ${JIRA_PROJECT_KEY}
   - issueTypeName: "Task"
   - parent: epicId
   - summary: title
   - description: content
2. Return: Issue key (e.g., "PROJ-125")
```

### getTasks(epicId)

```
1. Call ${MCP_ATLASSIAN_SEARCH_JQL}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - jql: "parent = epicId ORDER BY created ASC"
2. Return: Array of issue keys
```

### getTask(taskId)

```
1. Call ${MCP_ATLASSIAN_GET_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: taskId
2. Return: { id, title: summary, content: description, epicId: parent, status, type: issueType }
```

### updateTaskStatus(taskId, status)

```
1. Map status to Jira transition:
   - "in-progress" → "Start Progress" or equivalent
   - "in-review" → "Submit for Review" or equivalent
   - "done" → "Done" or equivalent
   - "blocked" → Add "Blocked" label or use custom field
2. Call ${MCP_ATLASSIAN_TRANSITION_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: taskId
   - transitionId: [mapped transition ID]
3. Return: success/fail
```

### addTaskComment(taskId, comment)

```
1. Call ${MCP_ATLASSIAN_ADD_COMMENT}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: taskId
   - body: comment (markdown)
2. Return: success/fail
```

---

## Confluence Integration

### Update Product Spec Page

```
1. Call ${MCP_ATLASSIAN_GET_PAGE} to read current page
2. Locate "Links" table in page body
3. Add Epic/Spec link row to table
4. Call ${MCP_ATLASSIAN_UPDATE_PAGE} to save
```

---

## ID Format

| Type | Pattern | Example |
|------|---------|---------|
| Epic | `{PROJECT_KEY}-{number}` | `PROJ-123` |
| Task | `{PROJECT_KEY}-{number}` | `PROJ-456` |
| Spec | `{PROJECT_KEY}-{number}` | `PROJ-789` |

---

## Error Handling

| Error | Response |
|-------|----------|
| Cloud ID not found | `❌ ATLASSIAN_CLOUD_ID not configured. Run setup.` |
| Project not found | `❌ Project ${key} not found. Check JIRA_PROJECT_KEY.` |
| Permission denied | `❌ No permission to create issues in ${project}.` |
| MCP unavailable | `❌ Atlassian MCP server not connected.` |
