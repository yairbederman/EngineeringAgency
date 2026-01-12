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

### addLabel(taskId, label)

```
1. Call ${MCP_ATLASSIAN_EDIT_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: taskId
   - fields: {
       "labels": { "add": [label] }
     }
2. Return: success/fail

Fallback (if edit fails):
   - Add comment: "📌 Label added: {label}"
   - Return: success with warning
```

### removeLabel(taskId, label)

```
1. Call ${MCP_ATLASSIAN_EDIT_ISSUE}:
   - cloudId: ${ATLASSIAN_CLOUD_ID}
   - issueIdOrKey: taskId
   - fields: {
       "labels": { "remove": [label] }
     }
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

## Change Log Operations

> **Design Principle**: Leverage native Atlassian versioning where possible.
> Confluence pages have built-in versioning. Jira issues have change history.

### archiveArtifact(artifactType, artifactId)

Archives the current version before making changes.

**Confluence**: No action needed - page versions are automatic.
**Jira**: No action needed - issue history is automatic.

```
1. If Confluence page:
   a. Call ${MCP_CONFLUENCE_GET_PAGE} to get current version number
   b. Record version number for rollback reference
   c. Note: Confluence auto-versions on every save
2. If Jira issue:
   a. Add comment: "📌 Checkpoint before change: version {timestamp}"
   b. Jira tracks all field changes automatically
3. Return: { previousVersion: number } (for Confluence) or { success: true } (for Jira)
```

### rollbackArtifact(artifactType, artifactId, targetVersion)

Restores an artifact to a previous version.

**Confluence**: Restore from page version history.
**Jira**: Not directly supported (use change log for manual rollback).

```
1. If Confluence page:
   a. Call ${MCP_CONFLUENCE_GET_PAGE_HISTORY} or equivalent
   b. Find target version content
   c. Call ${MCP_CONFLUENCE_UPDATE_PAGE} with old content
   d. Add comment: "🔄 Rolled back to version {targetVersion}"
2. If Jira issue:
   a. Add comment: "⚠️ ROLLBACK REQUESTED - See version at {timestamp}"
   b. Display previous field values from history
   c. Manual intervention may be required for field restoration
3. Return: success/fail with details
```

### logChange(artifactType, artifactId, change)

Appends a change entry to the artifact.

| Artifact Type | Storage Location | MCP Tool |
|---------------|------------------|----------|
| `product-spec` | Confluence page "Change Log" section | `confluence_update_page` |
| `epic` | Jira issue comment + description update | `jira_add_comment` |
| `tech-spec` | Confluence page "Change Log" section | `confluence_update_page` |
| `task` | Jira issue comment | `jira_add_comment` |

**Parameters**:
```json
{
  "type": "increment" | "adjustment" | "refinement" | "rollback",
  "source": "PM" | "DEV" | "STAKE" | "QA" | "TECH" | "EXT",
  "priority": "P0" | "P1" | "P2" | "P3",
  "description": "What changed",
  "impactedAreas": ["Epic", "TechSpec", "Tasks"]
}
```

**Implementation**:
```
1. Format change as structured comment:
   
   🔄 **CHANGE LOG ENTRY**
   - **Date**: {timestamp}
   - **Source**: {source} (PM/DEV/STAKE/QA/TECH/EXT)
   - **Priority**: {priority}
   - **Type**: {type} (increment/adjustment/refinement)
   - **Description**: {description}
   - **Impact**: {impactedAreas}
   
2. For Confluence pages:
   a. Call ${MCP_CONFLUENCE_GET_PAGE} to read current content
   b. Locate or create "## Change Log" section at end
   c. Append formatted entry to table
   d. Call ${MCP_CONFLUENCE_UPDATE_PAGE} to save
   
3. For Jira issues:
   a. Call ${MCP_JIRA_ADD_COMMENT}:
      - issueIdOrKey: artifactId
      - body: formatted change entry (with 🔄 prefix for filtering)
   b. Optionally update description to include change count

4. Return: { success: true, entryId: commentId or version }
```

### getChangeHistory(artifactType, artifactId)

Returns the change log for an artifact.

```
1. If Confluence page:
   a. Call ${MCP_CONFLUENCE_GET_PAGE}
   b. Parse "## Change Log" section
   c. Extract table rows as array
   d. Return: Array of change entries

2. If Jira issue:
   a. Call ${MCP_JIRA_GET_COMMENTS}
   b. Filter for comments with "🔄" prefix
   c. Parse change log format from each comment
   d. Return: Array of change entries sorted by date
```

### cascadeChange(rootArtifact, change)

Propagates a change to all linked downstream artifacts.

```
1. Identify linked artifacts:
   - Epic → Get child tasks via JQL: "parent = {epicId}"
   - Epic → Get linked Confluence pages via issue links
   
2. For each linked artifact:
   a. Call logChange with cross-reference:
      - description: "📎 Cascaded from {rootArtifact.key}: {change.description}"
      - type: "adjustment"
   b. If task affected by scope change:
      - Add label "needs-review" via ${MCP_JIRA_EDIT_ISSUE}
      
3. Return: { 
     cascaded: ["PROJ-124", "PROJ-125"],
     requiresReApproval: ["PROJ-123"]
   }
```

### generateChangeReport(epicId, dateRange?)

Generates a stakeholder Change Summary Report.

**Parameters**:
- `epicId`: Jira Epic key (e.g., "PROJ-123")
- `dateRange`: Optional `{ from: "2026-01-01", to: "2026-01-11" }`

**Implementation**:
```
1. Call ${MCP_JIRA_GET_ISSUE} for epic
2. Call ${MCP_JIRA_SEARCH_JQL} for child tasks: "parent = {epicId}"
3. For each issue, call ${MCP_JIRA_GET_COMMENTS}
4. Filter for 🔄 prefix comments within dateRange
5. Aggregate by priority and type
6. Format as markdown report:

   # Change Summary Report - {epicSummary}
   
   **Epic**: [{epicKey}]({jiraUrl})
   **Period**: {from} to {to}
   **Total Changes**: {count}
   
   ## P0 Changes (Blockers)
   - {list or "None"}
   
   ## Scope Changes
   | # | Date | Issue | Description | Source |
   |---|------|-------|-------------|--------|
   | 1 | 2026-01-11 | PROJ-124 | Added blog feature | DEV |
   
   ## Tasks Requiring Review
   - [PROJ-125] - Hero Section (needs-review label)

7. Option A: Create Confluence page with report
   - Call ${MCP_CONFLUENCE_CREATE_PAGE} in project space
   
   Option B: Add as Epic description appendix
   - Call ${MCP_JIRA_EDIT_ISSUE} to update description
   
8. Return: { reportUrl: url, summary: { total, p0, scope, minor } }
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Cloud ID not found | `❌ ATLASSIAN_CLOUD_ID not configured. Run setup.` |
| Project not found | `❌ Project ${key} not found. Check JIRA_PROJECT_KEY.` |
| Permission denied | `❌ No permission to create issues in ${project}.` |
| MCP unavailable | `❌ Atlassian MCP server not connected.` |
| Version not found | `❌ Confluence version {version} not found for page.` |
| Cascade failed | `⚠️ Cascade partially failed. Updated: [list]. Failed: [list]` |
| Label creation failed | `⚠️ Could not add "needs-review" label. Please add manually.` |
