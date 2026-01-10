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
├── _registry.json              # Index for ID generation and lookup
└── epics/
    └── EPIC-001-user-auth/     # Folder: {epicId}-{title-slug}
        ├── epic.md             # Epic content (always named epic.md)
        ├── tech-spec.md        # Tech spec (if created)
        └── tasks/
            ├── TASK-001.md     # Tasks for this epic
            └── TASK-002.md
```

> [!NOTE]
> **Folder naming**: `{epicId}-{title-slug}` (e.g., `EPIC-001-user-auth`)
> **File naming**: Always `epic.md` within the folder for consistency

---

## Registry Schema

**File**: `${LOCAL_SPECS_PATH}/_registry.json`

```json
{
  "nextId": {
    "EPIC": 2,
    "TASK": 5
  },
  "epics": {
    "EPIC-001": {
      "title": "User Authentication",
      "slug": "user-auth",
      "status": "open",
      "created": "2026-01-10",
      "hasSpec": true,
      "tasks": ["TASK-001", "TASK-002"]
    }
  }
}
```

---

## Operations

### createEpic(title, content)

```
1. Read _registry.json (create if missing with defaults)
2. Generate ID: EPIC-{nextId.EPIC:03d} (e.g., EPIC-001)
3. Generate slug: slugify(title) (e.g., "User Auth" → "user-auth")
4. Create folder: epics/EPIC-001-user-auth/
5. Write file: epics/EPIC-001-user-auth/epic.md
6. Update registry:
   - Increment nextId.EPIC
   - Add entry to epics object
7. Return: "EPIC-001"
```

### getEpic(epicId)

```
1. Read _registry.json
2. Look up epic by ID
3. Build path: epics/{epicId}-{slug}/epic.md
4. Read and return content
```

### createSpec(title, content, epicId)

```
1. Find epic folder from registry
2. Write file: epics/{epicId}-{slug}/tech-spec.md
3. Update registry: set hasSpec = true
4. Return: "{epicId}/tech-spec"
```

### createTask(title, content, epicId)

```
1. Find epic folder from registry
2. Generate ID: TASK-{nextId.TASK:03d}
3. Create tasks/ folder if missing
4. Write file: epics/{epicId}-{slug}/tasks/TASK-001.md
5. Update registry:
   - Increment nextId.TASK
   - Add task ID to epic's tasks array
6. Return: "TASK-001"
```

### getTasks(epicId)

```
1. Read _registry.json
2. Return epics[epicId].tasks array
```

---

## File Templates

### epic.md

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
| Tech Spec | [tech-spec.md](./tech-spec.md) |
| Tasks | See [tasks/](./tasks/) |
```

### task.md

```markdown
# {title}

**ID**: {taskId}  
**Epic**: [{epicId}](../epic.md)  
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
