# Cross-Project Feature Flow

> **Purpose**: Coordinate changes across multiple projects in a single feature.
> 
> **When to Use**: Features spanning Frontend + Backend, or multiple backend services.

---

## Prerequisites

1. **System Architecture Must Exist**: Verify `${SYSTEM_ARCH_ROOT}/analysis/service-topology.json` exists
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
- Example: `feature/W0-123-wg-data-api`, `feature/W0-123-wg-client`

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
