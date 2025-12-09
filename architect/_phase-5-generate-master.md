---
description: Phase 5 - Generate master copilot-instructions.md with coverage summary
---

# Phase 5: Generate Master

## Goal
Synthesize all extracted data into `copilot-instructions.md` with pattern enforcement rules and **coverage summary**.

## Steps

### 1. Analyze Patterns
From previous phase outputs, identify:
- Common wrappers/utilities used across files
- Import conventions
- Naming patterns (kebab-case, PascalCase, etc.)
- Error handling patterns

### 2. Generate Pattern Rules
Create MUST/NEVER rules based on discovered patterns.

### 3. Aggregate Coverage
Combine `_coverage` sections from all previous phases.

### 4. Categorize Files
Group files by domain/layer for `file-categorization.json`.

**Granularity Rules**:
| Directory File Count | Action |
|---------------------|--------|
| ≤50 files | List ALL files explicitly |
| 51-200 files | List key files + subdirectory breakdown |
| >200 files | List top 50 most-imported + subdirectory summary |

### 5. Generate Deep-Dive Files
Create additional documentation files (see Output section).

## Output

### `analysis/file-categorization.json`

> **⛔ MANDATORY**: The `_layerDefinitions` section MUST be present. Omission blocks validation.

```json
{
  "_layerDefinitions": {
    "1-database": { "categories": ["migrations", "entities", "schemas"], "dependsOn": [] },
    "2-repository": { "categories": ["repositories"], "dependsOn": ["1-database"] },
    "3-service": { "categories": ["services"], "dependsOn": ["2-repository"] },
    "4-api": { "categories": ["controllers", "routes", "api-clients"], "dependsOn": ["3-service"] },
    "5-state": { "categories": ["redux-slices", "stores"], "dependsOn": ["4-api"] },
    "6-ui": { "categories": ["react-components", "pages", "layouts"], "dependsOn": ["5-state"] }
  },
  "[category]": {
    "layer": "[1-6]",
    "files": ["[file paths]"],
    "dependsOn": ["[category names]"]
  }
}
```

**Layer Assignment Rules**:
| Layer | Execution Order | Categories | When to Create Tasks |
|-------|-----------------|------------|----------------------|
| 1 | First (no deps) | migrations, entities | Before any other layer |
| 2 | After Layer 1 | repositories | After DB schema exists |
| 3 | After Layer 2 | services | After repositories exist |
| 4 | After Layer 3 | controllers, routes | After services exist |
| 5 | After Layer 4 | redux-slices | After API client exists |
| 6 | After Layer 5 | react-components | After state slice exists |

**Purpose**: Enables /engineering-agent TechSpec § 5.4 "Implementation Inventory" dependency ordering.

### `copilot-instructions.md`
```markdown
# [Project] AI Instructions

> [One-line description from detected stack]

## Project Overview
- **Ecosystem**: [from source-structure.json]
- **Stack**: [framework + language]
- **Layers**: [detected layers]

## Path Aliases
| Alias | Path | Purpose |
|-------|------|---------|

## File Organization
| Category | Path | Purpose |
|----------|------|---------|

## Architecture Diagram
```
[Generated from directory structure]
```

## Critical Patterns
### [Pattern Name]
```[lang]
// Correct usage
```

## Pattern Enforcement
| Category | Rule | Reference |
|----------|------|-----------|
| [cat] | MUST [rule] | [file] |

## Extraction Coverage

| Category | Found | Documented | Coverage |
|----------|-------|------------|----------|
| Types/Interfaces | X | X | 100% |
| API Clients | X | X | 100% |
| Hooks/Functions | X | X | 100% |
| State Modules | X | X | 100% |

### Unresolved Items
- `[type]` in `[file]` - [reason]

## Deep-Dive References
- [entity-contracts.json](./analysis/entity-contracts.json)
- [api-contracts.json](./analysis/api-contracts.json)
- [function-registry.json](./analysis/function-registry.json)
- [dependency-chains.md](./deep-dive/dependency-chains.md)
- [data-flow.md](./deep-dive/data-flow.md)
- [component-registry.md](./deep-dive/component-registry.md)
- [testing-strategy.md](./deep-dive/testing-strategy.md)

## Feature Patterns (MANDATORY for /engineering-agent Pattern Matching)

> **⛔ BLOCKING**: This section MUST be populated. Empty or placeholder Feature Patterns table blocks validation.

Group related code by feature domain to enable "similar feature to mimic" lookups:

| Feature | Components | Hooks | API Client | State Slice |
|---------|------------|-------|------------|-------------|
| [FeatureName] | [ComponentNames] | [hookNames] | [ApiClient] | [sliceName] |

**Generation Algorithm**:
1. For each `stateModule` in `function-registry.json`:
   - Find hooks that read this state (via `dependencies.state`)
   - Find API clients called by those hooks (via `calls.external`)
   - Find components that use those hooks (search for hook imports)
2. Group into coherent "Feature" rows
3. Name features by their primary domain (Search, Order, Payment, etc.)

**Minimum Requirement**: At least 3 feature patterns documented.

**Example**:
| Feature | Components | Hooks | API Client | State Slice |
|---------|------------|-------|------------|-------------|
| Search Flow | SearchEngine, SearchResults | useSearchDealsFlight | SearchApiClient | searchEngineSlice |
| Order Process | OrderForm, OrderSummary | useOrderProcess | PaymentApiClient | orderSlice |
| Deal Page | DealPage, DealDetails | useDealPage | DealApiClient | dealSlice |

**Purpose**: Enables /engineering-agent TechSpec § 2 "Pattern Reuse" to find similar existing features.

## Key Domain Types
| Type | File | Purpose |
|------|------|---------|

## State Modules
| Module | Key | Purpose |
|--------|-----|---------|

## Critical Rules

### MUST
- ✅ [Detected patterns]

### NEVER
- ❌ [Anti-patterns]
```

## Critical Rules
1. **Coverage section required**: Always include Extraction Coverage table
2. **Unresolved items visible**: Any `_unresolved` from previous phases must appear
3. **Cross-references**: Link to all analysis files
4. **Pattern enforcement**: MUST/NEVER rules based on discovered patterns
5. **All deep-dive files**: Generate all 5 required files

---

## Additional Deep-Dive Outputs

### `deep-dive/component-registry.md`
```markdown
# Component Registry

## [Category] (e.g., Forms)
| Component | File | Purpose |
|-----------|------|---------|
| [Name] | [path] | [description] |

## [Category] (e.g., Layouts, Cards, Modals)
...
```

**Component Props Extraction (MANDATORY for UI Projects)**:

> **⛔ BLOCKING** (UI projects only): If `source-structure.json.layers` includes "ui", component props MUST be extracted.

For EACH major component in the registry (prioritize components with >100 lines or >3 state reads):
1. Extract its **Props interface** (type definition or inline)
2. List **Redux state reads** (find `useAppSelector`, `useSelector`, `useStore` calls)
3. List **dispatched actions** (find `dispatch(...)` calls)
4. Note key **child components** imported

**Enhanced Format**:
```markdown
## [Category] Components

### [ComponentName]
- **File**: `[path]`
- **Props**: `{ prop1: string, prop2?: number, onSubmit: () => void }`
- **State Reads**: `siteId`, `searchEngine.searchFlight`
- **Dispatches**: `setSearchEngineValue`, `setSearchLoading`
- **Children**: `DatePicker`, `DestinationPicker`
```

**Minimum Requirement**: At least 10 major components documented with full metadata.

### `deep-dive/testing-strategy.md`
```markdown
# Testing Strategy

- **Framework**: [detected test framework]
- **Test Location**: [test file pattern, e.g., `*.test.ts` or `__tests__/`]
- **Coverage Tool**: [if detected]

## Mock Patterns
- [Description of common mocking approach]

## Test Utilities
| Utility | File | Purpose |
|---------|------|---------|
```

### `deep-dive/debugging-guide.md`
```markdown
# Debugging Guide

## Logger Patterns
- [Detected logging library or console patterns]

## Dev Mode Flags
| Flag | Location | Purpose |
|------|----------|---------|

## Error Boundaries
- [Detected error handling patterns]
```
