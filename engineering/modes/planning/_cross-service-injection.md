# Cross-Service API Auto-Injection Protocol

> **Purpose**: Automatically inject cross-service API context from `/system-architecture-agent` outputs into Tasks.

---

## When to Use

This protocol applies when:
1. **TechSpec Mode** (Step 2): Identifying affected projects and cross-project impacts
2. **TaskPlanning Mode** (Step 3D): Populating Backend task API context

---

## Data Sources

| File | Content | Use Case |
|------|---------|----------|
| `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json` | Service dependencies | Identify which services call which |
| `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json` | API contracts | Get endpoint signatures for tasks |
| `${SYSTEM_ARCH_OUTPUT}/analysis/unified-domain-model.json` | Canonical entities | Resolve shared type definitions |

---

## Auto-Injection Algorithm

### Step 1: Detect Cross-Project Scope

From Epic description or Tech Spec, identify affected projects:
```
affectedProjects = [PROJECT_A, PROJECT_B, ...]
```

### Step 2: Load Cross-Service APIs

Read `cross-service-apis.json` and filter:
```
relevantAPIs = crossServiceCalls.filter(call =>
  affectedProjects.includes(call.caller) OR
  affectedProjects.includes(call.callee)
)
```

### Step 3: Build Injection Map

For each task, check if it involves a cross-service call:

```markdown
### Cross-Service API Context (Auto-Injected)

**Endpoint**: `[METHOD] [/path]`
**Caller**: [caller-project] → **Callee**: [callee-project]

**Request**:
- Type: `[RequestType]` (defined in: [project])

**Response**:
- Type: `[ResponseType]` (defined in: [project])

**Used By**: `[ClassName].[methodName]`

**Verification**:
- File: `[FileName]:[lineNumber]`
- Code: `[snippet]`
```

### Step 4: Inject Shared Types

If task uses a shared type, add canonical reference:
```markdown
**Shared Type Reference**:
- `[TypeName]` → Canonical: `[canonical-project]/[path]/[TypeName].[ext]`
- Used in: [project-1], [project-2], ...
```

---

## Integration Points

### In TechSpec Mode (Step 2)

After identifying affected projects, automatically:
1. Read `service-topology.json`
2. Expand scope to include services that:
   - Are called by affected services (`callsServices`)
   - Call affected services (`calledBy`)
3. Document in Tech Spec "Cross-Project Impact" section

### In TaskPlanning Mode (Step 3D)

For Backend Tasks that involve cross-service calls:
1. Read `cross-service-apis.json`
2. Match task's target file/service to `crossServiceCalls`
3. Inject full API contract into task description

---

## Failure Handling

If `${SYSTEM_ARCH_OUTPUT}` files don't exist:
1. **Emit Warning**: "System architecture files not found"
2. **Recommend**: "Run `/system-architecture-agent` for cross-project visibility"
3. **Proceed with Caution**: Document assumptions in Epic/Tasks
4. **Add Label**: `needs-system-architecture` to Epic
