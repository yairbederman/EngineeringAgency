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

### Entity Usage Map (MANDATORY for /lognet TechSpec)

> **⛔ BLOCKING**: `_entityUsage` section MUST be present in `function-registry.json`. Omission is a Phase 4.5 gate failure.

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

**Purpose**: Enables /lognet to trace data flow from DB → API → State → UI for TechSpec § 5.4

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
