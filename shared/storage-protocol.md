# Storage Protocol

> **Purpose**: Routing layer for storage operations. Mode files call these operations; the protocol delegates to the active adapter.

---

## Adapter Registry

> [!NOTE]
> **Symmetric Architecture**: Every backend has an explicit adapter file.
> Adapters are loaded based on `STORAGE_BACKEND` from `agent-config.md`.

| Backend Value | Adapter File | Token Cost |
|---------------|--------------|------------|
| `atlassian` (default) | `adapters/atlassian-adapter.md` | ~400 |
| `local` | `adapters/local-adapter.md` | ~500 |
| `github` | `adapters/github-adapter.md` | _(future)_ |
| `linear` | `adapters/linear-adapter.md` | _(future)_ |

---

## Resolution Algorithm

```
1. Read STORAGE_BACKEND from Agent_Config/agent-config.md
2. If value is missing:
   → Default to "atlassian"
3. Look up value in Adapter Registry table above
4. If found: Load corresponding adapter file
5. If NOT found: ERROR "Unknown storage backend: [value]"
```

---

## Abstract Operations

> **Contract**: All adapters MUST implement these operations.

### Adapter Compliance Checklist

Before adding a new adapter, verify it implements:
- [ ] `createProductSpec(title, content)` → returns ID (local mode only)
- [ ] `updateProductSpec(specId, content)` → updates content (local mode only)
- [ ] `getProductSpec(specId)` → returns content (local mode only)
- [ ] `createEpic(title, content)` → returns ID
- [ ] `getEpic(epicId)` → returns content
- [ ] `createSpec(title, content, epicId)` → returns ID
- [ ] `createTask(title, content, epicId)` → returns ID
- [ ] `getTasks(epicId)` → returns ID[]
- [ ] Error handling with clear messages

> [!NOTE]
> **Product Spec Operations**: Required for `local` mode. For `atlassian` mode, product specs are managed in Confluence directly.

### Product Spec Operations (Local Mode)

> **Purpose**: Persists the product spec as the source of truth before Epic creation.

| Operation | Signature | Returns |
|-----------|-----------|---------|
| `createProductSpec` | `(title: string, content: markdown)` | `SpecId` (e.g., `SPEC-001`) |
| `getProductSpec` | `(specId: string)` | `{ id, title, content, status, gapsResolved }` |
| `updateProductSpec` | `(specId: string, content: markdown)` | `success/fail` |
| `linkProductSpecToEpic` | `(specId: string, epicId: string)` | `success/fail` |

### Epic Operations

| Operation | Signature | Returns |
|-----------|-----------|---------|
| `createEpic` | `(title: string, content: markdown)` | `EpicId` (e.g., `PROJ-123` or `EPIC-001`) |
| `getEpic` | `(epicId: string)` | `{ id, title, content, status }` |
| `updateEpic` | `(epicId: string, content: markdown)` | `success/fail` |

### Spec Operations

| Operation | Signature | Returns |
|-----------|-----------|---------|
| `createSpec` | `(title: string, content: markdown, epicId: string)` | `SpecId` |
| `getSpec` | `(specId: string)` | `{ id, title, content, epicId }` |

### Task Operations

| Operation | Signature | Returns |
|-----------|-----------|---------|
| `createTask` | `(title: string, content: markdown, epicId: string)` | `TaskId` |
| `getTasks` | `(epicId: string)` | `TaskId[]` |
| `getTask` | `(taskId: string)` | `{ id, title, content, epicId, status, type }` |
| `updateTaskStatus` | `(taskId: string, status: string)` | `success/fail` |
| `addTaskComment` | `(taskId: string, comment: markdown)` | `success/fail` |

**Valid Status Values**: `open`, `in-progress`, `in-review`, `done`, `blocked`

**Valid Type Values**: `task` (default), `bug`

> [!NOTE]
> Mode files call these operations abstractly. The active adapter handles backend-specific implementation (Jira transitions vs. local file updates).

### Link Operations

| Operation | Signature | Returns |
|-----------|-----------|---------|
| `linkItems` | `(sourceId: string, targetId: string)` | `success/fail` |

---

## Atlassian Implementation

> **Adapter File**: `adapters/atlassian-adapter.md`

See adapter file for complete operation-to-MCP mapping and Confluence integration details.

---

## Artifact Display Protocol

> **Purpose**: Gates display artifact links consistently regardless of backend.

| Backend | Epic Display | Task Display |
|---------|--------------|--------------|
| `atlassian` | `[PROJ-123](https://jira.url/...)` | `[PROJ-124](https://jira.url/...)` |
| `local` | `[EPIC-001](file:///.specs/epics/.../epic.md)` | `[TASK-001](file:///.specs/.../TASK-001.md)` |

---

## Error Handling

| Error | Response |
|-------|----------|
| Unknown backend value | `❌ Unknown STORAGE_BACKEND: "[value]". Valid options: atlassian, local` |
| Adapter file not found | `❌ Adapter file not found: [path]. Run setup or check configuration.` |
| Operation failed | Return operation-specific error from adapter |
