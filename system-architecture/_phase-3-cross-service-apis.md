# Phase 3: Cross-Service APIs

> **Goal**: Document API contracts between services—what one service calls from another.

---

## Input

- `${ANALYSIS_DIR}/service-topology.json` (from Phase 2)
- Each project's `api-contracts.json`

## Output

- `${ANALYSIS_DIR}/cross-service-apis.json`

---

## Instructions

### Step 1: Identify Cross-Service Calls

For each dependency edge from Phase 2:

```
<caller-service> → <callee-service>
```

Find the specific endpoints called.

### Step 2: Extract API Contracts

**From Client Side (caller)**:
- Search for API client code (fetch, axios, SDK)
- Match URL patterns to backend endpoints
- Extract request parameters and expected response types

**From Server Side (callee)**:
- Read `api-contracts.json`
- Match endpoints to client calls
- Include validation rules

### Step 3: Map DTOs Across Boundaries

For each cross-service call:

1. **Request DTO**: What the client sends
2. **Response DTO**: What the server returns
3. **Source Definition**: Where the type is defined (which project, which file)

Example:
```json
{
  "caller": "<caller-service>",
  "callee": "<callee-service>",
  "endpoint": {
    "method": "<HTTP method>",
    "path": "<endpoint path>",
    "fullPath": "<full proxied path if frontend>"
  },
  "request": {
    "type": "<RequestDTO name>",
    "definedIn": "<project that defines this type>",
    "file": "<relative path to source file>",
    "fields": {
      "<fieldName>": { "type": "<type>", "required": "<boolean>" }
    }
  },
  "response": {
    "type": "<ResponseDTO name>",
    "definedIn": "<project that defines this type>",
    "file": "<relative path to source file>"
  }
}
```

### Step 4: Handle Proxied Calls

Some services proxy to others. Document the full chain:

```json
{
  "chain": ["<origin-service>", "<intermediate-service>", "<final-service>"],
  "originEndpoint": "<first endpoint in chain>",
  "intermediates": [
    {
      "service": "<intermediate-service>",
      "endpoint": "<intermediate endpoint>",
      "transforms": "<what this service does to the data>"
    }
  ],
  "finalEndpoint": {
    "service": "<final-service>",
    "endpoint": "<final endpoint>"
  }
}
```

### Step 4.5: Verify No False Direct Dependencies (BLOCKING)

> [!CAUTION]
> **Transitive vs Direct Call Verification**
>
> Before documenting any `A → C` dependency, you MUST verify that A calls C **directly** by checking:
> 1. Does Service A have an API client for Service C in its codebase?
> 2. Is that client **actively USED** (not just configured/imported)?
> 3. Or does A actually call B, and B calls C?
>
> **If A → B → C is the actual path, do NOT document A → C as a direct call.**

**Verification Steps**:
1. For each potential `caller → callee` edge:
   - Search caller's codebase for imports/usage of callee's API client class
   - Verify at least one method from that client is invoked
   - If not found: The dependency does NOT exist
2. If caller uses IntermediaryClient which calls callee:
   - Document as: `caller → intermediary` and `intermediary → callee`
   - Document the intermediary in `chain[]` format above
3. Flag any corrected assumptions in `_warnings`:
   ```json
   {
     "type": "false-direct-dependency",
     "claimed": "<service-a> → <service-c>",
     "actual": "<service-a> → <service-b> → <service-c>",
     "evidence": "No <ServiceCClient> usage found in <service-a> codebase"
   }
   ```

---

## Output Schema

```json
{
  "generatedAt": "<ISO-8601 timestamp>",
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
    // ... one entry per cross-service call
  ],
  "sharedTypes": [
    {
      "name": "<TypeName>",
      "canonicalDefinition": {
        "project": "<owning project>",
        "file": "<relative path to source>"
      },
      "usedIn": ["<list of projects using this type>"]
    }
    // ... one entry per shared type
  ],
  "_coverage": {
    "dependenciesAnalyzed": "<count>",
    "endpointsDocumented": "<count>",
    "sharedTypesFound": "<count>"
  }
}
```

---

## Validation

| Check | Requirement |
|-------|-------------|
| All dependencies covered | Each edge from Phase 2 has ≥1 documented call |
| Types resolved | No `unknown` request/response types |
| Sources linked | Every type has `definedIn` and `file` |

---

## Common Patterns

### Frontend API Client Pattern
```typescript
// <frontend-project>/src/sdk/api/<serviceApi>.ts
export const <methodName> = (request: <RequestType>) =>
  api.post<<ResponseType>>('<proxied-endpoint-path>', request);
```

### Backend-to-Backend Pattern
```java
// In <caller-service> calling <callee-service>
@FeignClient(name = "<service-name>")
public interface <ServiceClient> {
    @GetMapping("/<resource>/{id}")
    <ResponseType> get<Resource>(@PathVariable String id);
}
```
