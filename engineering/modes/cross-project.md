# Cross-Project Feature Flow

> **Purpose**: Coordinate changes across multiple projects in a single feature.
> 
> **When to Use**: Features spanning Frontend + Backend, or multiple backend services.

---

## Prerequisites

1. **System Architecture Must Exist**: Verify `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json` exists
2. **If Missing**: Recommend running `/system-architecture-agent` first

---

## Task Ordering (Dependency Layers)

Tasks across projects must be ordered by dependency layer:

| Layer | Description | Example Projects |
|-------|-------------|------------------|
| **Layer 0** | Database/Migrations | Schema changes in any backend |
| **Layer 1** | Backend Services | Services that depend on Layer 0 |
| **Layer 2** | API Integration | Endpoints exposed to frontend |
| **Layer 3** | Frontend | UI consuming Layer 2 APIs |

---

## Branch Strategy

- Each project gets its own feature branch
- Format: `feature/[EpicKey]-[project-name]`
- Example: `feature/${JIRA_PROJECT_KEY}-XXX-[project-name]`, `feature/${JIRA_PROJECT_KEY}-XXX-[project-name]`

---

## Implementation Sequence (STRICT)

1. **Complete ALL Layer 0 tasks first** across all affected projects
2. Mark Layer 0 tasks as "In Review" 
3. **Wait for Layer 0 PRs to merge** before starting Layer 1
4. Continue upward through layers
5. Frontend (Layer 3) tasks are always LAST

---

## Cross-Project Task Template Addition

For tasks in multi-project features, add to task description:

```markdown
### Cross-Project Context
- **Epic**: [Epic Key]
- **Layer**: [0-3]
- **Depends On Projects**: [List other project branches this depends on]
- **Blocks Projects**: [List other project branches waiting on this]
```

---

## Identifying Cross-Project Scope

During TechSpec, use `service-topology.json` to identify:
1. Which services the feature touches (`callsServices`)
2. Which services might break if APIs change (`calledBy`)
3. Add ALL impacted projects to the Tech Spec's "Implementation Inventory"

---

## Auto-Detection Rule

Cross-project mode is automatically activated when **any** of these conditions are met:

| Condition | Detection |
|-----------|-----------|
| Epic mentions multiple projects | e.g., "project-frontend" AND "project-backend" in description |
| TechSpec identifies 2+ project roots | Step 2 returns multiple `${PROJECT_*}` variables |
| Feature involves both Frontend + Backend | Any UI change + API change |
| Service topology shows dependencies | `callsServices` or `calledBy` is non-empty for affected service |

---

## TaskPlanning Integration

When creating tasks for cross-project features:

### Step 1: Read Cross-Service APIs
```
Read: ${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json
```

Extract the API contract for service-to-service calls involved in this feature.

### Step 2: Inject API Context into Tasks

For backend tasks that expose APIs consumed by other projects:
```markdown
### Cross-Service API Contract
- **Endpoint**: `<METHOD> <path>`
- **Consumed By**: `<list of caller services>`
- **Contract Source**: `cross-service-apis.json` → `crossServiceCalls` where `callee` = `<this-service>`
```

For frontend tasks that call backend APIs:
```markdown
### Backend API to Call
- **Endpoint**: `<METHOD> <path>`
- **Owner**: `<callee-service>`
- **Contract Source**: `cross-service-apis.json` → `crossServiceCalls` where `caller` = `<this-service>`
```

### Step 3: Set Task Dependencies

Use the Layer system to set Jira task links:
- Layer 0 tasks: No dependencies (first)
- Layer 1+ tasks: Link "Depends On" to all lower-layer tasks

---

## Example: Reading cross-service-apis.json

```json
// From cross-service-apis.json (actual schema from Phase 3)
{
  "crossServiceCalls": [
    {
      "id": "<unique-call-id>",
      "caller": "<caller-service>",
      "callee": "<callee-service>",
      "endpoint": {
        "method": "<HTTP method>",
        "path": "<endpoint path>"
      },
      "request": {
        "type": "<RequestDTO>",
        "definedIn": "<owning project>",
        "fields": {}
      },
      "response": {
        "type": "<ResponseDTO>",
        "definedIn": "<owning project>"
      },
      "usedBy": ["<list of components/modules using this call>"]
    }
  ],
  "sharedTypes": [...]
}
```

**How to Find APIs for a Service**:
1. Filter `crossServiceCalls` where `callee` equals the target service → APIs it **exposes**
2. Filter `crossServiceCalls` where `caller` equals the target service → APIs it **consumes**

**Inject into Task**:
- For backend task (service is `callee`): List all `caller` services that consume the endpoint
- For frontend task (service is `caller`): Include the `request`/`response` types from the contract

