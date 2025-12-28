---
description: Phase 4 - Map function/hook dependencies and state management with exhaustive coverage
---

# Phase 4: Map Dependencies

## Goal
Map ALL functions/hooks/services with their dependencies. Extract ALL state management. **Exhaustive—every file counted.**

## Input
Use `source-structure.json.discoveredLocations` for hooks, state, and services.

## Steps

### 1. Dynamic State Management Detection
Scan ALL files in `discoveredLocations.state` for patterns:

| Pattern | What It Indicates |
|---------|-------------------|
| `createStore`, `configureStore` | Store initialization |
| `createSlice`, `reducer` | State slice/reducer |
| `initialState`, `defaultState` | State shape definition |
| `set*`, `update*`, `add*`, `remove*` | Action/mutation functions |
| `get*`, `select*`, `use*Selector` | Selector functions |
| `useStore`, `useSelector` | Store hooks |

**Works for**: Redux, Zustand, Pinia, MobX, Recoil, Jotai, Vuex, or any custom solution.

### 2. Exhaustive Hook/Function Scanning
For EACH directory in `discoveredLocations.hooks` and `discoveredLocations.services`:
1. List ALL files
2. For EACH exported function, extract:
   - Name, file, signature
   - Parameters with **resolved types** (not `any`)
   - Return type with field structure
   - Dependencies (what it imports/calls)

**Minimum Required Fields** (for EVERY hook):
| Field | Description |
|-------|-------------|
| `file` | Absolute path |
| `parameters` | Object with `{ [param]: { type, required } }` |
| `returnType` | Resolved type with field structure |
| `dependencies.state` | Array of state keys read |
| `dependencies.hooks` | Array of hooks called internally |
| `calls.external` | Array of API clients/methods called |
| `calls.dispatches` | Array of state actions dispatched |

**Coverage Rule**: 
- Count files in each hook directory
- Documented count must equal file count OR log skip reason
- Valid skip reasons: `utility`, `index-only`, `deprecated`, `test-file`

**INVALID skip reasons** (extraction MUST continue):
- `"Pending deep scanning"` → Scan NOW in this execution
- `"Too many files"` → Scan ALL files, no excuses
- `"Complex"` → Scan anyway, mark individual `_unresolved` items
- Any reason with "later" or "pending" → Invalid, scan now

> **⛔ BLOCKING ENFORCEMENT**: Before proceeding to Phase 5, grep the generated `function-registry.json` for "Pending" or "pending". If found, **HALT** and re-extract the undocumented directories immediately. This is a hard gate.

### 3. Build Call Chains
Document at least ONE call chain for each detected flow:
- **Search/Query flow** (user input → API → results)
- **Create/Submit flow** (form → validation → API)
- **Auth flow** (if applicable)
- **Payment/Transaction flow** (if applicable)

### 3.5: Cross-Verify External Calls (BLOCKING)

> [!IMPORTANT]
> Every `calls.external` entry MUST be verified against `api-contracts.json`.
> This ensures consistency between dependency mapping and API contract documentation.

**For each hook/service with `calls.external`:**
1. Look up the called API client in `api-contracts.json`
2. Verify the specific method/endpoint exists in that client's `endpoints[]`
3. If not found:
   - Search the codebase for the actual call location
   - Update `api-contracts.json` if endpoint was missed in Phase 3
   - OR mark as `_unresolved.externalCalls` with actionable reason

**Output Requirement** (enhanced `calls.external` format):
```json
"calls": {
  "external": [
    {
      "client": "<ApiClientName>",
      "method": "<methodName>",
      "verifiedIn": "api-contracts.json#<ClientName>[<endpointIndex>]",
      "codeEvidence": "<file>:<line>"
    }
  ]
}
```

**If verification fails**:
```json
"_unresolved": {
  "externalCalls": [
    {
      "hook": "<hookName>",
      "claimed": "<ClientName>.<methodName>",
      "reason": "Method not found in api-contracts.json - may need Phase 3 re-extraction"
    }
  ]
}
```

### 4. Cross-Project Dependency Verification (Conditional)

> [!IMPORTANT]
> **Constraint**: This step applies based on the `detectedStack` from Phase 1.

**IF `detectedStack.type` includes "Backend" (JVM, Python, Go, Node-Backend):**
> **⛔ BLOCKING**: EVERY claimed cross-project dependency MUST be verified with code evidence.

**Step 1: Detect HTTP Client Patterns** (Framework-Agnostic)

Search the project for common HTTP client patterns:

| Ecosystem | Search Pattern |
|-----------|----------------|
| JVM/Spring | `RestTemplate`, `WebClient`, `FeignClient`, `@FeignClient` |
| Node.js | `fetch`, `axios`, `http`, `got`, `request` |
| Python | `requests`, `httpx`, `aiohttp`, `urllib` |
| Go | `http.Client`, `http.Get`, `http.Post` |
| .NET | `HttpClient`, `RestClient`, `WebRequest` |

**Step 2: Extract Target Service URLs**

For EACH HTTP client usage found:
1. Identify the target URL or config key (e.g., `${<SERVICE>_URL}`, `api.<service>.url`)
2. Cross-reference with `${GLOBAL_WORKFLOWS_ROOT}/shared/configuration.md` to identify target project
3. Verify the call is ACTIVE (not commented out, in test files, or dead code)

**Step 3: Document with Code Evidence**

```json
"crossProjectDependencies": [
  {
    "target": "<target-project-from-registry>",
    "callType": "http" | "library",
    "codeEvidence": "<file>:<line> - <what was found>",
    "verified": true
  }
]
```

**Step 4: Handle Unverifiable Claims**

If a config key exists but NO active code usage is found:

```json
"_excludedDependencies": [
  {
    "claimed": "<target-project>",
    "configFound": "<config-file>:<key>",
    "reason": "Config exists but no active client usage found in code",
    "searchPerformed": "Searched for <pattern> in ${PROJECT_PATH}/src"
  }
]
```

> [!IMPORTANT]
> **Do NOT include dependencies in `crossProjectDependencies` unless `verified: true`**. Unverified claims go to `_excludedDependencies`.


## Output

### `analysis/function-registry.json`
```json
{
  "stateModules": {
    "[moduleName]": {
      "file": "[path]",
      "stateShape": {
        "[field]": "[type]"
      },
      "actions": ["[action names]"],
      "selectors": ["[selector names]"]
    }
  },
  "hooks": {
    "[hookName]": {
      "file": "[path]",
      "parameters": {
        "[param]": { "type": "[resolved]", "required": true }
      },
      "returnType": "[type with fields]",
      "dependencies": {
        "state": ["[which state modules it reads]"],
        "hooks": ["[which hooks it calls]"],
        "navigation": ["[router/navigation hooks]"]
      },
      "calls": {
        "internal": ["[functions/services called]"],
        "external": ["[API clients called]"],
        "dispatches": ["[actions dispatched]"]
      }
    }
  },
  "services": {
    "[serviceName]": {
      "file": "[path]",
      "methods": ["[methodName]"],
      "dependencies": ["[what it calls]"]
    }
  },
  "_coverage": {
    "hookDirectories": [
      { "path": "[dir]", "files": 21, "documented": 21 }
    ],
    "stateDirectories": [
      { "path": "[dir]", "files": 14, "documented": 14 }
    ]
  },
  "_entityUsage": {
    "[EntityName]": {
      "usedInAPIs": ["[ApiClient].[method]"],
      "usedInHooks": ["[hookName]"],
      "usedInComponents": ["[ComponentName]"],
      "stateSlices": ["[sliceName]"]
    }
  }
}
```

### Entity Usage Map (Conditional Requirement)

> **Constraint**: Required if `detectedStack` involves frontend/client logic or complex data flow.

> **⛔ BLOCKING**: If required by stack, `_entityUsage` section MUST be present in `function-registry.json`.

Build `_entityUsage` by cross-referencing:
1. For EACH entity in `entity-contracts.json` with `kind: "class"` or frequently-used interfaces:
   - Search `api-contracts.json` for endpoints using this entity as request/response type
   - Search hooks for functions returning or consuming this entity type
   - Search state modules for slices storing this entity
2. Output reverse mapping:

```json
"_entityUsage": {
  "TouristTrip": {
    "usedInAPIs": ["SearchApiClient.searchResults", "PreOrderApiClient.prebook"],
    "usedInHooks": ["useSearchDealsFlight", "usePriceBar"],
    "usedInState": ["searchSlice.searchResults", "reservationSlice.trip"],
    "usedInComponents": ["TripCard", "SearchResults"]
  }
}
```

**Purpose**: Enables /engineering-agent to trace data flow from DB → API → State → UI for TechSpec § 5.4

### `deep-dive/dependency-chains.md`
```markdown
# Dependency Chains

## [Flow Name] Flow

```
[Entry Point]
  └── [Layer 1]
       ├── [Action/State Change]
       └── [Layer 2]
            └── [External/API Call]
```

## State Structure
```
RootState
├── [stateKey]: [sliceName]   [description]
└── ...
```

## Key Dependencies Table
| Function | State Reads | API Calls | Dispatches |
|----------|-------------|-----------|------------|
```

## Critical Rules
1. **Exhaustive hooks**: Every file in hook directories must be documented
2. **Exhaustive state**: Every state module must have full state shape
3. **No `any`**: Resolve types or document in `_unresolved`
4. **Minimum 3 call chains**: At least 3 distinct flows documented
5. **Deep-dive completeness**: All required files must be generated

---

## Additional Deep-Dive Outputs

### `deep-dive/data-flow.md`
Document end-to-end data flows with specific function/component names:
```markdown
# Data Flows

## [Flow Name] (e.g., Search Flow)
```
User Input
  └── [ComponentName] (on submit)
       └── [hookName]
            ├── dispatch([actionName])
            └── [ApiClient].[method]()
                 └── Response → dispatch([resultAction])
                      └── [ResultComponent] renders
```
```
Minimum 3 flows documented (e.g., Search, Submit, Auth, Payment).

### `deep-dive/component-registry.md` (Generated in Phase 5)
### `deep-dive/testing-strategy.md` (Generated in Phase 5)
### `deep-dive/debugging-guide.md` (Generated in Phase 5)
