# Phase 2: Service Topology

> **Goal**: Map service responsibilities and dependencies to understand how projects interact at runtime.

---

## Input

- `${ANALYSIS_DIR}/project-inventory.json` (from Phase 1)
- Each project's `api-contracts.json`

## Output

- `${ANALYSIS_DIR}/service-topology.json`

---

## Instructions

### Step 1: Classify Service Types

For each `ready` project from Phase 1:

| Type | Characteristics |
|------|-----------------|
| **Frontend** | Has `type: "Frontend"`, calls backend APIs |
| **API Gateway** | Proxies requests to multiple backends |
| **Backend Service** | Exposes REST/GraphQL endpoints, may call other services |
| **Data Service** | Primarily database operations, rarely calls other services |
| **Shared Library** | No runtime, provides types/utilities |

### Step 2: Detect Dependencies

Analyze each project to determine what it CALLS:

**For Frontend Projects**:
- Read `api-contracts.json`
- Look for `backendOwner` field in endpoint definitions
- Identify unique backend services called

**For Backend Projects**:
- Search for HTTP client configurations (RestTemplate, WebClient, Feign, axios)
- Look for external service URLs in configuration files
- Check for `@FeignClient` or similar annotations

### Step 2.5: Verify Dependencies Against Code (MANDATORY)

> [!IMPORTANT]
> AI instructions may contain stale or inaccurate dependencies.
> You MUST verify each claimed dependency against actual source code.

**For each backend project**, verify external service calls exist in code:

1. **Search for BaseUrl patterns**:
   ```bash
   grep_search("api.*BaseUrl", "${PROJECT_PATH}/src/main/java")
   ```
   
2. **Match claimed vs actual**:
   | Claimed in AI Instructions | grep Result | Status |
   |---------------------------|-------------|--------|
   | `apiCmsBaseUrl` | Found / Not Found | ✅ / ❌ |
   | `apiDataBaseUrl` | Found / Not Found | ✅ / ❌ |

3. **Only include verified dependencies**:
   - If grep finds the BaseUrl → Include in `callsServices`
   - If grep does NOT find it → **Exclude** and add to `_warnings`

**Warning entry format**:
```json
{
  "type": "unverified-dependency",
  "project": "wg-booking-api",
  "claimed": "wg-cms-api",
  "source": "copilot-instructions.md",
  "reason": "No apiCmsBaseUrl found in project config files"
}
```

### Step 3: Build Dependency Graph

Create edges representing runtime calls:

```json
{
  "generatedAt": "ISO timestamp",
  "services": [
    {
      "name": "wg-client",
      "type": "Frontend",
      "role": "Next.js web application",
      "exposedEndpoints": 0,
      "callsServices": ["wg-data-api", "wg-search-api", "wg-ordermanager-api", "wg-payment-api"],
      "calledBy": []
    },
    {
      "name": "wg-data-api",
      "type": "Backend",
      "role": "Site data and configuration",
      "exposedEndpoints": 48,
      "callsServices": ["wg-cms-api"],
      "calledBy": ["wg-client"]
    }
  ],
  "dependencies": [
    {
      "from": "wg-client",
      "to": "wg-data-api",
      "type": "http",
      "description": "Frontend fetches site configuration"
    },
    {
      "from": "wg-client",
      "to": "wg-search-api",
      "type": "http",
      "description": "Frontend searches for packages"
    }
  ],
  "layers": {
    "presentation": ["wg-client"],
    "api": ["wg-data-api", "wg-search-api", "wg-ordermanager-api", "wg-payment-api", "wg-tripdetails-api", "wg-ancillary-api"],
    "integration": ["wg-cms-api"]
  }
}
```

### Step 3.5: Compute Reverse Dependencies (calledBy)

For each service, compute which services call it:

1. Initialize empty `calledBy` array for each service
2. For each dependency edge (from → to):
   - Add `from` to `to.calledBy`
3. This enables **impact analysis**: When modifying a service, check `calledBy` to find affected consumers

**Purpose**: Enables `/engineering-agent` to answer "If I change this API, what breaks?"

### Step 4: Detect Circular Dependencies

Check for cycles in the dependency graph:
- A → B → C → A would be a circular dependency
- Flag any cycles found in `_warnings`

### Step 5: Identify Orphan Services

Services that:
- Are NOT called by any other service
- Do NOT call any other service

These may be:
- Standalone utilities
- Misconfigured
- Legacy/unused

---

## Output Schema

```json
{
  "generatedAt": "ISO timestamp",
  "services": [/* array of service nodes */],
  "dependencies": [/* array of edges */],
  "layers": {
    "presentation": [],
    "api": [],
    "integration": [],
    "data": []
  },
  "_orphans": ["service names with no connections"],
  "_warnings": [
    {
      "type": "circular-dependency",
      "path": ["service-a", "service-b", "service-a"]
    }
  ],
  "_coverage": {
    "projectsAnalyzed": 8,
    "dependenciesFound": 12
  }
}
```

---

## Validation

| Check | Requirement |
|-------|-------------|
| All ready projects included | Count matches Phase 1 ready count |
| No unresolved references | All `callsServices` entries exist in `services` |
| Layers assigned | Every service has a layer |
| Dependencies verified | All dependencies confirmed via grep search |
| Unverified flagged | Any unverified deps documented in `_warnings` |

---

## Dependency Detection Patterns

### Java/Spring
```java
// Look for these in service files:
@FeignClient(name = "other-service")
restTemplate.getForObject("http://other-service/...")
webClient.get().uri("http://other-service/...")
```

### Node.js/TypeScript
```typescript
// Look for these patterns:
fetch('/api/data-service/...')
axios.get(process.env.DATA_API_URL + '/...')
const client = new SomeServiceClient()
```

### Configuration Files
```yaml
# application.yml, .env, etc.
services:
  data-api:
    url: http://wg-data-api:8080
```
