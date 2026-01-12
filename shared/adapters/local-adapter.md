# Local Markdown Adapter

> **Backend**: `local`  
> **Purpose**: Store epics, specs, and tasks as markdown files in the local filesystem.

---

## Configuration

**Required** (from `agent-config.md`):

| Variable | Default | Description |
|----------|---------|-------------|
| `LOCAL_SPECS_PATH` | `./.specs` | Root directory for local storage |

---

## Directory Structure

```
${LOCAL_SPECS_PATH}/
├── _registry.json                       # Index for ID generation and lookup
├── product-specs/                        # Product specifications (source of truth)
│   └── SPEC-001-feature-name/
│       └── feature-name-product-spec.md  # Named: {slug}-product-spec.md
└── epics/
    └── EPIC-001-feature-name/            # Folder: {epicId}-{title-slug}
        ├── feature-name-epic.md          # Named: {slug}-epic.md
        ├── feature-name-tech-spec.md     # Named: {slug}-tech-spec.md
        └── tasks/
            ├── TASK-001-task-title.md    # Named: {taskId}-{slug}.md
            └── TASK-002-another-task.md
```

> [!NOTE]
> **Folder naming**: `{id}-{title-slug}` (e.g., `SPEC-001-feature-name`, `EPIC-001-feature-name`)
> **File naming**: `{slug}-{type}.md` (e.g., `feature-name-product-spec.md`, `feature-name-epic.md`)
> This ensures files are easily identifiable when open in editors.

---

## Registry Schema

**File**: `${LOCAL_SPECS_PATH}/_registry.json`

```json
{
  "nextId": {
    "SPEC": 2,
    "EPIC": 2,
    "TASK": 5
  },
  "productSpecs": {
    "SPEC-001": {
      "title": "Feature Name",
      "slug": "feature-name",
      "status": "approved",
      "created": "2026-01-10",
      "gapsResolved": true,
      "linkedEpic": "EPIC-001"
    }
  },
  "epics": {
    "EPIC-001": {
      "title": "Feature Name",
      "slug": "feature-name",
      "status": "open",
      "created": "2026-01-10",
      "productSpec": "SPEC-001",
      "hasSpec": true,
      "tasks": ["TASK-001", "TASK-002"]
    }
  },
  "tasks": {
    "TASK-001": {
      "title": "Setup Project",
      "slug": "setup-project",
      "epicId": "EPIC-001",
      "status": "open",
      "type": "task",
      "created": "2026-01-10",
      "commentCount": 0
    }
  }
}
```

> [!NOTE]
> **Task Status Values**: `open`, `in-progress`, `in-review`, `done`, `blocked`
> **Task Type Values**: `task` (default), `bug`

---

## Operations

### Product Spec Operations

> **Purpose**: Product Spec is the source of truth created before Epic.

### createProductSpec(title, content)

```
1. Read _registry.json (create if missing with defaults)
2. Generate ID: SPEC-{nextId.SPEC:03d} (e.g., SPEC-001)
3. Generate slug: slugify(title) (e.g., "Feature Name" → "feature-name")
4. Create folder: product-specs/SPEC-001-feature-name/
5. Write file: product-specs/SPEC-001-feature-name/feature-name-product-spec.md
6. Update registry:
   - Increment nextId.SPEC
   - Add entry to productSpecs object with status: "draft"
7. Return: "SPEC-001"
```

### getProductSpec(specId)

```
1. Read _registry.json
2. Look up spec by ID in productSpecs
3. Build path: product-specs/{specId}-{slug}/{slug}-product-spec.md
4. Read and return content with metadata
```

### updateProductSpec(specId, content)

```
1. Find spec folder from registry
2. Overwrite file: product-specs/{specId}-{slug}/{slug}-product-spec.md
3. Update registry if needed (e.g., gapsResolved, status)
4. Return: success
```

### linkProductSpecToEpic(specId, epicId)

```
1. Update productSpecs[specId].linkedEpic = epicId
2. Update epics[epicId].productSpec = specId
3. Write updated _registry.json
4. Return: success
```

---

### Epic Operations

### createEpic(title, content)

```
1. Read _registry.json (create if missing with defaults)
2. Generate ID: EPIC-{nextId.EPIC:03d} (e.g., EPIC-001)
3. Generate slug: slugify(title) (e.g., "User Auth" → "user-auth")
4. Create folder: epics/EPIC-001-user-auth/
5. Write file: epics/EPIC-001-user-auth/user-auth-epic.md
6. Update registry:
   - Increment nextId.EPIC
   - Add entry to epics object
7. Return: "EPIC-001"
```

### getEpic(epicId)

```
1. Read _registry.json
2. Look up epic by ID
3. Build path: epics/{epicId}-{slug}/{slug}-epic.md
4. Read and return content
```

### createSpec(title, content, epicId)

```
1. Find epic folder from registry
2. Write file: epics/{epicId}-{slug}/{slug}-tech-spec.md
3. Update registry: set hasSpec = true
4. Return: "{epicId}/tech-spec"
```

### createTask(title, content, epicId)

```
1. Find epic folder from registry
2. Generate ID: TASK-{nextId.TASK:03d}
3. Generate task slug: slugify(title) (e.g., "Setup Project" → "setup-project")
4. Create tasks/ folder if missing
5. Write file: epics/{epicId}-{slug}/tasks/TASK-001-setup-project.md
6. Update registry:
   - Increment nextId.TASK
   - Add task ID to epic's tasks array
7. Return: "TASK-001"
```

### getTasks(epicId)

```
1. Read _registry.json
2. Return epics[epicId].tasks array
```

### getTask(taskId)

```
1. Read _registry.json
2. Look up task by ID in tasks object
3. Find epicId from task entry
4. Build path: epics/{epicId}-{epicSlug}/tasks/{taskId}-{taskSlug}.md
5. Read file content
6. Return: { id, title, content, epicId, status, type }
```

### updateTaskStatus(taskId, status)

```
1. Read _registry.json
2. Update tasks[taskId].status = status
3. Write updated _registry.json
4. Append status change to task markdown file:
   
   ---
   ### {timestamp} | Status: {status}
   
5. Return: success/fail
```

### addTaskComment(taskId, comment)

```
1. Read _registry.json
2. Increment tasks[taskId].commentCount
3. Write updated _registry.json
4. Append comment to task markdown file:
   
   ---
   ### {timestamp} | Comment
   {comment}
   
5. Return: success/fail
```

### addLabel(taskId, label)

```
1. Read _registry.json
2. Find task entry
3. Initialize labels array if missing: tasks[taskId].labels = []
4. If label already exists: Return success (no-op)
5. Append label to array: tasks[taskId].labels.push(label)
6. Write updated _registry.json
7. Update task markdown frontmatter/header:
   
   **Labels**: [unverified], [needs-review]
   
8. Return: success/fail
```

### removeLabel(taskId, label)

```
1. Read _registry.json
2. Find task entry
3. If no labels array or label not found: Return success (no-op)
4. Remove label from array
5. Write updated _registry.json
6. Update task markdown frontmatter/header
7. Return: success/fail
```

---

## File Templates

### {slug}-epic.md

```markdown
# {title}

**ID**: {epicId}  
**Status**: Open  
**Created**: {date}

---

## Goal

{content.goal}

## Acceptance Criteria

{content.criteria}

## Scope

**In**: {content.inScope}  
**Out**: {content.outScope}

---

## Links

| Type | Link |
|------|------|
| Tech Spec | [{slug}-tech-spec.md](./{slug}-tech-spec.md) |
| Tasks | See [tasks/](./tasks/) |
```

### TASK-{id}-{task-slug}.md

```markdown
# {title}

**ID**: {taskId}  
**Epic**: [{epicId}](../{slug}-epic.md)  
**Status**: Open

---

{content}
```

---

## ID Format

| Type | Pattern | Example |
|------|---------|---------|
| Epic | `EPIC-{NNN}` | `EPIC-001`, `EPIC-042` |
| Task | `TASK-{NNN}` | `TASK-001`, `TASK-123` |
| Spec | `{epicId}/tech-spec` | `EPIC-001/tech-spec` |

---

## Change Log Operations

> **Purpose**: Support mid-flow change requests with versioning, rollback, and traceability.

### Directory Structure (Extended)

```
${LOCAL_SPECS_PATH}/
├── _registry.json
├── product-specs/
│   └── SPEC-001-feature-name/
│       ├── feature-name-product-spec.md    # Current active
│       └── .versions/                       # Version archive
│           ├── product-spec_v1.0.md
│           └── product-spec_v1.1.md
└── epics/
    └── EPIC-001-feature-name/
        ├── feature-name-epic.md
        ├── feature-name-tech-spec.md
        ├── .versions/                       # Version archive
        │   ├── epic_v1.0.md
        │   └── tech-spec_v1.0.md
        └── tasks/
```

### Registry Schema (Extended)

```json
{
  "nextId": { "SPEC": 2, "EPIC": 2, "TASK": 5 },
  "productSpecs": {
    "SPEC-001": {
      "title": "Feature Name",
      "version": "1.1",
      "changeLog": [
        {
          "version": "1.1",
          "date": "2026-01-11",
          "type": "increment",
          "source": "DEV",
          "priority": "P2",
          "description": "Added blog feature requirement",
          "impactedAreas": ["Epic", "TechSpec"]
        }
      ]
    }
  },
  "epics": { },
  "tasks": { }
}
```

---

### archiveArtifact(artifactType, artifactId)

Archives the current version before making changes.

```
1. Read _registry.json
2. Determine artifact path based on type:
   - "product-spec" → product-specs/{specId}-{slug}/
   - "epic" → epics/{epicId}-{slug}/
   - "tech-spec" → epics/{epicId}-{slug}/
   - "task" → epics/{epicId}-{slug}/tasks/
3. Create .versions/ folder if missing
4. Get current version from registry (default: "1.0")
5. Copy current file to .versions/{type}_v{version}.md
6. Increment version number:
   - Minor change: 1.0 → 1.1
   - Major change: 1.0 → 2.0
7. Update registry with new version number
8. Return: { previousVersion: "1.0", newVersion: "1.1" }
```

### rollbackArtifact(artifactType, artifactId, targetVersion)

Restores an artifact to a previous version.

```
1. Read _registry.json
2. Build path to .versions/{type}_v{targetVersion}.md
3. If file doesn't exist:
   - Return error: "Version {targetVersion} not found"
4. Read target version content
5. Archive current version (call archiveArtifact)
6. Overwrite current file with target version content
7. Add change log entry:
   - type: "rollback"
   - description: "Rolled back to version {targetVersion}"
8. Update registry version to reflect rollback
9. Return: { restoredVersion: targetVersion, archivedVersion: previousVersion }
```

### logChange(artifactType, artifactId, change)

Appends a change entry to an artifact's change log.

**Parameters**:
```json
{
  "type": "increment" | "adjustment" | "refinement" | "rollback",
  "source": "PM" | "DEV" | "STAKE" | "QA" | "TECH" | "EXT",
  "priority": "P0" | "P1" | "P2" | "P3",
  "description": "What changed",
  "impactedAreas": ["Epic", "TechSpec", "Tasks"],
  "codeImpact": { "filesAffected": [], "tasksAffected": [], "reworkEstimate": "2h" } | null,
  "testImpact": { "testsModified": [], "testsAdded": [] } | null
}
```

**Implementation**:
```
1. Read _registry.json
2. Find artifact entry
3. Initialize changeLog array if missing
4. Create change entry:
   {
     "version": newVersion,
     "date": timestamp,
     "type": change.type,
     "source": change.source,
     "priority": change.priority,
     "description": change.description,
     "impactedAreas": change.impactedAreas,
     "codeImpact": change.codeImpact,
     "testImpact": change.testImpact
   }
5. Append to changeLog array
6. Write updated _registry.json
7. Append to artifact markdown file:

   ---
   ## Change Log
   
   | Version | Date | Source | Type | Priority | Description |
   |---------|------|--------|------|----------|-------------|
   | v1.1 | 2026-01-11 | DEV | increment | P2 | Added blog feature |
   
8. Return: { entryId: changeLog.length }
```

### getChangeHistory(artifactType, artifactId)

Returns the change log for an artifact.

```
1. Read _registry.json
2. Find artifact entry
3. Return: artifact.changeLog || []
```

### cascadeChange(rootArtifact, change)

Propagates a change to all linked downstream artifacts.

```
1. Read _registry.json
2. Identify linked artifacts:
   - product-spec → linked epic
   - epic → tech-spec, tasks
   - tech-spec → tasks
3. For each linked artifact:
   a. Call logChange with cross-reference:
      - description: "Cascaded from {rootArtifact.id}: {change.description}"
      - type: "adjustment"
   b. If task: Update status to include "needs-review" marker
4. Return: { 
     cascaded: ["EPIC-001", "TASK-001", "TASK-002"],
     requiresReApproval: ["EPIC-001"] // if change.type is "adjustment" or "increment"
   }
```

### generateChangeReport(epicId, dateRange?)

Generates a stakeholder Change Summary Report.

**Parameters**:
- `epicId`: Target epic
- `dateRange`: Optional `{ from: "2026-01-01", to: "2026-01-11" }`

**Implementation**:
```
1. Read _registry.json
2. Get epic and all linked artifacts (product-spec, tech-spec, tasks)
3. Collect all changeLog entries within dateRange
4. Group by priority and type
5. Generate markdown report:

   # Change Summary Report - {epicTitle}
   
   **Period**: {dateRange.from} to {dateRange.to}
   **Total Changes**: {count}
   
   ## P0 Changes (Blockers)
   - {list or "None"}
   
   ## Scope Changes
   | # | Date | Description | Source | Status |
   |---|------|-------------|--------|--------|
   | 1 | 2026-01-11 | Added blog feature | DEV | ✅ |
   
   ## Minor Adjustments
   - {list}
   
   ## Documentation Status
   | Artifact | Version | Last Updated | Aligned |
   |----------|---------|--------------|---------|
   | ProductSpec | v1.2 | 2026-01-11 | ✅ |
   | Epic | v1.1 | 2026-01-11 | ✅ |
   | TechSpec | v1.0 | 2026-01-10 | ⚠️ |

6. Write report to: epics/{epicId}-{slug}/change-report-{date}.md
7. Return: { reportPath: path, summary: { total, p0, scope, minor } }
```

---

### listVersions(artifactType, artifactId)

Lists all available versions for an artifact.

```
1. Build path to .versions/ folder
2. List all files matching pattern {type}_v*.md
3. Parse version numbers
4. Return: ["1.0", "1.1", "1.2"] sorted descending
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Registry not found | Create with default: `{ "nextId": { "EPIC": 1, "TASK": 1 }, "epics": {} }` |
| Epic not found | `❌ Epic not found: [epicId]` |
| Write failed | `❌ Failed to write: [path]. Check permissions.` |
| Version not found | `❌ Version [version] not found for [artifactId]` |
| Cascade failed | `⚠️ Cascade partially failed. Updated: [list]. Failed: [list]` |
