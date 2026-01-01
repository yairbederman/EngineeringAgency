---
description: Phase 3.7 - Extract state management contracts with full action payloads and selector types
---

# Phase 3.7: Extract State Management Contracts

## Goal
Extract full state management contracts with action payloads, async thunks, and selector return types—enabling Tech Specs to specify exact state mutations.

## Trigger Condition
**Execute this phase IF** `source-structure.json.detectedStack.type` includes "Frontend" AND any of:
- Redux Toolkit (`createSlice`, `configureStore`)
- Legacy Redux (`createStore`, `combineReducers`)
- Zustand (`create()`)
- Pinia (`defineStore`)
- MobX (`makeObservable`, `@observable`)
- Recoil (`atom`, `selector`)
- Jotai (`atom`)

**Skip IF**: No state management detected OR backend-only project.

## Input
Use `source-structure.json.discoveredLocations.state` from Phase 1.

## Relationship to Phase 4
Phase 4's `function-registry.json.stateModules` captures basic state info. This phase **extends** that data with:
- Full action payloads (not just names)
- Async thunk lifecycle actions
- Selector implementation details
- State update patterns

## Steps

### 1. Detect State Management Pattern
Already detected in Phase 1, but verify:

| Pattern | Indicators |
|---------|------------|
| Redux Toolkit | `@reduxjs/toolkit`, `createSlice` |
| Zustand | `zustand`, `create()` |
| Pinia | `pinia`, `defineStore` |
| MobX | `mobx`, `makeObservable` |
| Recoil | `recoil`, `atom`, `selector` |

### 2. Extract Full Slice/Store Definitions

For EACH state module:

#### State Shape (with types)
```typescript
// Not just field names, but full type structure
stateShape: {
  items: {
    type: "[EntityType][]",
    initialValue: "[]"
  },
  isLoading: {
    type: "boolean", 
    initialValue: "false"
  }
}
```

#### Action Payloads
For EACH action, extract the FULL payload type:

```typescript
// Input: setItems: (state, action: PayloadAction<[EntityType][]>) => ...
// Output:
{
  "setItems": {
    "payloadType": "[EntityType][]",
    "payloadFields": {
      // If payload is an object, expand its fields
    },
    "reducerLogic": "state.items = action.payload"
  }
}
```

### 3. Extract Async Thunk Contracts

For EACH `createAsyncThunk` or equivalent:

| Field | Source |
|-------|--------|
| `thunkName` | First argument to `createAsyncThunk` |
| `argType` | Type of the thunk argument |
| `returnType` | Return type of the async function |
| `pendingMutation` | What happens in `.pending` |
| `fulfilledMutation` | What happens in `.fulfilled` |
| `rejectedMutation` | What happens in `.rejected` |
| `apiCall` | Which API client/method is called |

### 4. Extract Selector Contracts

For EACH selector (createSelector, simple selectors):

| Field | Source |
|-------|--------|
| `selectorName` | Function/variable name |
| `inputSelectors` | Dependencies on other selectors |
| `returnType` | Full return type with fields |
| `memoized` | Uses createSelector or useMemo |
| `derivedFrom` | Which state slices it reads |

### 5. Build State Flow Diagrams
Document key state update flows:

```
User Action → Thunk Dispatch → API Call → Fulfilled → State Update
```

## Output

### `analysis/state-contracts.json`
```json
{
  "detectedPattern": "redux-toolkit",
  "slices": {
    "[sliceName]": {
      "file": "[path]",
      "stateShape": {
        "[fieldName]": {
          "type": "[resolved type]",
          "initialValue": "[value]",
          "nullable": false
        }
      },
      "actions": {
        "[actionName]": {
          "type": "sync | prepare",
          "payloadType": "[type or object shape]",
          "payloadFields": {
            "[field]": "[type]"
          },
          "reducerLogic": "[brief description of mutation]"
        }
      },
      "asyncThunks": {
        "[thunkName]": {
          "file": "[path if separate]",
          "argType": "[type]",
          "argFields": {
            "[field]": "[type]"
          },
          "returnType": "[type]",
          "apiCall": {
            "client": "[ApiClient]",
            "method": "[methodName]",
            "verifiedIn": "api-contracts.json#[ref]"
          },
          "lifecycle": {
            "pending": "[state mutation]",
            "fulfilled": "[state mutation]",
            "rejected": "[state mutation]"
          }
        }
      },
      "selectors": {
        "[selectorName]": {
          "file": "[path]",
          "returnType": "[type]",
          "inputSelectors": ["[selectorName]"],
          "derivedFrom": ["[sliceName.field]"],
          "memoized": true
        }
      }
    }
  },
  "_stateFlows": [
    {
      "name": "[Flow Name]",
      "trigger": "user submits form",
      "sequence": [
        "dispatch([thunkName](params))",
        "[thunkName].pending → isLoading = true",
        "[ApiClient].[method](params)",
        "[thunkName].fulfilled → items = payload",
        "select[Items]() returns results"
      ]
    }
  ],
  "_coverage": {
    "slicesExtracted": 8,
    "actionsExtracted": 45,
    "thunksExtracted": 12,
    "selectorsExtracted": 23
  }
}
```

## Cross-Reference Requirements

### With api-contracts.json
Each `asyncThunk.apiCall` MUST reference a verified endpoint:
```json
"apiCall": {
  "client": "[ApiClient]",
  "method": "[methodName]",
  "verifiedIn": "api-contracts.json#[ApiClient][0]"
}
```

### With entity-contracts.json
State shape types MUST resolve to entities:
```json
"stateShape": {
  "items": {
    "type": "[EntityType][]",
    "entityRef": "entity-contracts.json#[EntityType]"
  }
}
```

## Critical Rules

1. **Full Payloads**: Extract complete payload types, not just action names
2. **Thunk Lifecycle**: Document all three lifecycle states (pending/fulfilled/rejected)
3. **Selector Types**: Resolve selector return types fully
4. **API Verification**: Every thunk API call must verify against `api-contracts.json`
5. **Entity References**: State types must reference `entity-contracts.json`
6. **State Flows**: Document at least 3 key flows for complex apps
