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

### Step 1.5: Enumerate Internal Submodules (MANDATORY)

> [!IMPORTANT]
> **Do NOT rely solely on AI instructions for submodule lists.**
> You MUST enumerate submodules by scanning the actual project directory structure.

**For each project**, detect submodules by:

1. **Run `list_dir` on project root**:
   - Identify directories that represent submodules (code modules, not build artifacts)
   - Exclude non-code directories: `.git`, `.gradle`, `.idea`, `node_modules`, `build`, `dist`, `bin`, `out`, `target`

2. **Classify submodule types by naming convention**:
   | Pattern | Type |
   |---------|------|
   | Contains `util`, `common`, `shared` | Core/Utility |
   | Contains `client`, `ws`, `api`, `sdk` | Service Client |
   | Contains `web`, `rest`, `controller` | Web/API Module |
   | Contains `external`, `integration`, `connector` | External Integration |
   | Other code directories | Application Module |

3. **Document in service entry**:
   ```json
   {
     "name": "<project-name>",
     "type": "<type>",
     "internalModules": [
       { "name": "<submodule-dir-name>", "type": "<inferred-type>" }
     ]
   }
   ```

**For Library/Monorepo Projects (CRITICAL)**:
- These often have MANY submodules (10+)
- You MUST enumerate ALL code directories at the project root
- Cross-reference with build config (`settings.gradle`, `package.json` workspaces, `pom.xml` modules) if available

**Minimum Module Coverage Requirements:**

| Service Type | Minimum Modules Expected | If Below Minimum |
|--------------|--------------------------|------------------|
| Frontend | ≥ 2 (at least UI/components and app/routing) | Re-scan project structure |
| Backend Service | ≥ 1 (at least main service package) | Check src/main/java subdirectories |
| Shared Library | ≥ 2 (must enumerate all published modules) | Check settings.gradle or pom.xml |

> [!WARNING]
> **If a service has 0 `internalModules` after enumeration:**
> 1. Re-run `list_dir` on the project's source root (`src/` or `src/main/java/`)
> 2. For Java projects: check `src/main/java/<package>/` subdirectories
> 3. For Node projects: check for `packages/`, `apps/`, or `src/` subdirectories
> 4. If still 0 modules → Add warning to `_warnings[]`:
>    ```json
>    {
>      "type": "no-internal-modules",
>      "service": "<service-name>",
>      "reason": "No submodules detected after enumeration - verify project structure is correct"
>    }
>    ```

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
> **Finding a configuration key is NOT enough.** You must verify it is **USED** in active code.

> [!CAUTION]
> **⛔ BLOCKING GATE**: You MUST execute `grep_search` for EACH claimed dependency before adding to `callsServices`.
> Skipping this step is a workflow failure. If no search was performed, the dependency CANNOT be added.
> Do NOT trust `backendOwner` or `description` fields in `api-contracts.json` without code verification.

**For each potential dependency**, perform a Two-Step Verification:

1. **Verify Configuration Existence**:
   - Search for the BaseUrl/ServiceUrl key in config files.
   - *Example*: `grep_search("api.*BaseUrl", "${PROJECT_PATH}")`

2. **Verify Active Code Usage (CRITICAL)**:
   - If a URL key exists (e.g., `apiTargetServiceBaseUrl`), you **MUST** search for usages of that variable in the codebase.
   - **Rule**: If a URL is defined but NOT used in any Client/Service class, it is a **Dormant Dependency**. Do NOT include it in the graph.
   - **Rule**: Distinguish **Transitive vs Direct**.
     - *Case*: Service A calls Service B. Service B calls Service C.
     - *Check*: Does Service A have a `ServiceCApiClient` that uses `apiTargetServiceBaseUrl`?
     - *Result*: If NO, then Service A -> Service C is **FALSE**. Only Service A -> Service B is TRUE.

3. **Only include verified dependencies**:
   - If config exists AND key is used in code → Include in `callsServices`
   - If config missing OR key unused → **Exclude** and add to `_warnings`

**Warning entry format**:
```json
{
  "type": "dormant-dependency",
  "project": "<project-name>",
  "claimed": "<claimed-target-service>",
  "reason": "Config key found but no active code usage detected"
}
```

### Step 2.6: Frontend BFF Route Verification (MANDATORY for Frontend projects)

> [!CAUTION]
> **⛔ BLOCKING**: For frontend projects with Next.js API routes or BFF proxies, do NOT trust `api-contracts.json` descriptions that claim backend targets.

For EACH claimed route → backend mapping in `api-contracts.json._nextApiRoutes`:

1. **Open the actual route file** (`route.ts` or `route.js`) using `view_file`
2. **Search for the backend service name or URL**:
   - `grep_search("<claimed-backend>", "${ROUTE_DIR}")`
   - Look for `process.env.*_URL`, `fetch()` calls, or SDK imports
3. **Decision**:
   - If backend reference found → Include dependency with `codeEvidence`
   - If NOT found → Mark as `_unverified` and **EXCLUDE** from topology

**Example of false positive to catch**:
```json
// api-contracts.json says:
"<route-category>": { "description": "<category> proxies to <claimed-backend>" }

// But grep_search("<backend-name>", "src/app/api/<route-category>") returns NO RESULTS
// → DO NOT add <frontend> → <claimed-backend> dependency
```

### Step 2.7: Evidence Quality Gate (BLOCKING)

> [!CAUTION]
> **⛔ BLOCKING**: Every dependency MUST have HIGH-QUALITY code evidence.
> Evidence that uses phrases like "inferred", "assumed", "standard pattern" is INVALID and MUST NOT appear in the final output.

**Evidence Quality Requirements:**

| Quality Level | Criteria | Allowed in `dependencies[]`? |
|--------------|----------|------------------------------|
| **HIGH** | Specific `file:line` with code snippet | ✅ Yes |
| **MEDIUM** | File reference without line number | ⚠️ With warning in `_warnings[]` |
| **LOW/INVALID** | "Inferred", "pattern-based", "assumed", "standard" | ❌ No - must verify or exclude |

**For each dependency, verify:**
1. `codeEvidence` contains a file path (relative or absolute)
2. `codeEvidence` contains a line number reference (`:L123` or `:123` format)
3. Evidence does NOT contain forbidden phrases: "inferred", "assumed", "pattern", "standard"

**If evidence fails quality check:**
1. Run `grep_search` to find actual code evidence for the dependency
2. For library dependencies: search for specific import statements (e.g., `import com.example.common`)
3. For HTTP dependencies: search for client class usage or URL construction
4. If no high-quality evidence found → **Exclude from `dependencies[]`** and add to `_warnings[]`:
   ```json
   {
     "type": "low-quality-evidence-removed",
     "from": "<source>",
     "claimed": "<target>",
     "originalEvidence": "<the vague evidence that was rejected>",
     "reason": "Evidence did not meet quality requirements"
   }
   ```

**Library Dependency Evidence Requirements:**
For dependencies of type `library`, evidence MUST include:
- Specific import statement with package path
- File and line number where import occurs
- Example: `OrderService.java:8 - import com.example.common.model.SessionData`

**Do NOT use generic evidence like:**
- ❌ "Standard Spring Boot dependency pattern"
- ❌ "Uses LTS common utilities"
- ❌ "Inferred from architecture"

### Step 3: Build Dependency Graph

Create edges representing runtime calls:

> [!IMPORTANT]
> **Every dependency MUST include `codeEvidence`**. Dependencies without verification CANNOT appear in the `dependencies[]` array.

```json
{
  "generatedAt": "<ISO-8601 timestamp>",
  "services": [
    {
      "name": "<service-name>",
      "type": "<Frontend | Backend | Data Service | Shared Library>",
      "role": "<from project-inventory.json>",
      "exposedEndpoints": "<count from api-contracts.json>",
      "callsServices": ["<list of VERIFIED service names>"],
      "calledBy": ["<computed in Step 3.5>"]
    }
    // ... one entry per ready project
  ],
  "dependencies": [
    {
      "from": "<calling-service>",
      "to": "<called-service>",
      "type": "<http | grpc | message-queue | library>",
      "description": "<why this dependency exists>",
      "codeEvidence": "<file>:<line> - <what was found>",
      "verified": true
    }
    // ... one entry per VERIFIED dependency edge ONLY
  ],
  "layers": {
    "presentation": ["<frontend services>"],
    "api": ["<backend API services>"],
    "integration": ["<external/CMS services>"],
    "data": ["<database-focused services>"],
    "shared": ["<library projects>"]
  },
  "_verificationSummary": {
    "totalClaimed": "<number of dependencies initially claimed>",
    "verified": "<number that passed verification>",
    "excluded": "<number excluded due to no code evidence>"
  },
  "_warnings": [
    {
      "type": "unverified-dependency-removed",
      "from": "<source>",
      "claimed": "<target>",
      "reason": "<why it was excluded>"
    }
  ]
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

### Step 6: Final Dependency Verification Gate (BLOCKING)

> [!CAUTION]
> **⛔ BLOCKING**: Before finalizing `service-topology.json`, EVERY dependency MUST be verified.

**For EACH project in `project-inventory.json`:**

1. **Read** the project's `function-registry.json.crossProjectDependencies` (if backend) OR `api-contracts.json._backendMapping` (if frontend)

2. **Check for `codeEvidence`** field:
   - If `codeEvidence` present → Dependency is pre-verified, include it
   - If `codeEvidence` missing → Run live verification:
     - For frontend: `grep_search("<target-service>", "${PROJECT_PATH}/src")`
     - For backend: `grep_search("<target-baseUrl>|<target-ServiceClient>", "${PROJECT_PATH}/src")`

3. **Decision**:
   - Evidence found → Add to `dependencies[]` with `codeEvidence` field
   - No evidence → Add to `_warnings[]` and EXCLUDE from topology

**Verification Outcome Requirements:**

| Outcome | Action |
|---------|--------|
| `codeEvidence` exists | Include in `dependencies[]` with `verified: true` |
| Live search finds evidence | Include with new `codeEvidence` from search |
| No evidence found | Exclude from topology, add to `_warnings[]` |

**Final Check**: Count dependencies:
- `_verificationSummary.totalClaimed` = initial count from all projects
- `_verificationSummary.verified` = count in `dependencies[]`
- `_verificationSummary.excluded` = count in `_warnings[]` with `type: "unverified-dependency-removed"`

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
  target-service:
    url: http://<service-name>:<port>
```

### Dormant Dependency Detection

**False Positive Example (Do NOT map):**
```typescript
// urls.ts
export const apiTargetServiceBaseUrl = process.env.TARGET_SERVICE_URL; // Defined here
// BUT... never imported or used in any ApiClient.ts file
```

**True Positive Example (Map this):**
```typescript
// ServiceApiClient.ts
import { apiTargetServiceBaseUrl } from './urls'; // Imported
// ...
return fetch(`${apiTargetServiceBaseUrl}/endpoint`); // Used!
```
