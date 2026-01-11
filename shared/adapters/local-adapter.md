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

## Error Handling

| Error | Response |
|-------|----------|
| Registry not found | Create with default: `{ "nextId": { "EPIC": 1, "TASK": 1 }, "epics": {} }` |
| Epic not found | `❌ Epic not found: [epicId]` |
| Write failed | `❌ Failed to write: [path]. Check permissions.` |
