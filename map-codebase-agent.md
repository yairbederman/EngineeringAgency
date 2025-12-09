---
description: Generate comprehensive AI instructions for any project type with deep engineering metadata extraction for Tech Spec readiness
---

# Map Codebase Agent

Produces `${AI_INSTRUCTIONS_ROOT}` with full entity contracts, API definitions, and dependency maps—enabling Tech Specs with zero `[TBD]` placeholders.

## Design Principles

| Principle | Description |
|-----------|-------------|
| **Zero Hardcoding** | Detect patterns dynamically, not specific frameworks |
| **Project Agnostic** | Works for client, backend, data, microservices, monolith |
| **Exhaustive** | Count files and track coverage; no silent omissions |
| **No `any`** | Resolve types or document in `_unresolved` |

## Configuration

First, read the configuration file for path variables:
**Read**: `${ARCHITECT_ROOT}/configuration.md`

Where `ARCHITECT_ROOT` = `./mapcodebase`

---

## Execution

Run phases in order. Each phase outputs to `${AI_INSTRUCTIONS_ROOT}` in the target project.

### Phase 1: Detect Stack
**Read**: `${ARCHITECT_ROOT}/_phase-1-detect-stack.md`
**Output**: `analysis/techstack.md`, `analysis/source-structure.json`
- Detect ecosystem (Node/JVM/Python/Go/Rust/.NET)
- Discover ALL relevant directories dynamically
- Count files for coverage tracking

### Phase 2: Extract Entities  
**Read**: `${ARCHITECT_ROOT}/_phase-2-extract-entities.md`
**Output**: `analysis/entity-contracts.json`
- Extract ALL types/interfaces/classes with full fields
- No `any` without resolution attempt → `_unresolved`
- Track coverage

### Phase 3: Extract APIs
**Read**: `${ARCHITECT_ROOT}/_phase-3-extract-apis.md`
**Output**: `analysis/api-contracts.json`
- **Exhaustive**: Document ALL directories from Phase 1
- Every skipped directory logged with reason
- Cross-reference types with entity-contracts

### Phase 4: Map Dependencies
**Read**: `${ARCHITECT_ROOT}/_phase-4-map-dependencies.md`
**Output**: `analysis/function-registry.json`, `deep-dive/dependency-chains.md`, `deep-dive/data-flow.md`
- Dynamic state detection (works for any library)
- **Exhaustive**: All hooks/services documented (≥80% coverage)
- Minimum 3 call-chain diagrams

### Phase 4.5: Enforcement Gate (BLOCKING)
**Before proceeding to Phase 5**, check coverage thresholds:

| Category | Source Count | Extracted Count | Required |
|----------|-------------|-----------------|----------|
| Classes | `source-structure.json.fileCount.classes` | `entity-contracts` with `kind: "class"` | ≥70% |
| Hooks | `source-structure.json.fileCount.hooks.total` | `function-registry.hooks` count | ≥80% |
| APIs | `source-structure.json.fileCount.apis` | `api-contracts` non-empty clients | 100% |
| State | `source-structure.json.fileCount.state.slices` | `function-registry.stateModules` | 100% |

**If ANY threshold not met**:
1. **DO NOT proceed** to Phase 5
2. List specific undocumented files
3. Return to the failing phase with targeted extraction
4. Re-check thresholds after re-extraction

**Only proceed to Phase 5 when all thresholds are met.**

### Phase 5: Generate Master
**Read**: `${ARCHITECT_ROOT}/_phase-5-generate-master.md`
**Output**: `copilot-instructions.md`, `analysis/file-categorization.json`, `deep-dive/component-registry.md`, `deep-dive/testing-strategy.md`, `deep-dive/debugging-guide.md`
- Include Coverage Summary section
- Surface all unresolved items
- Pattern enforcement rules

### Validation
**Read**: `${ARCHITECT_ROOT}/_validation-rules.md`
- Run completeness checks
- Output validation summary

---

## Success Criteria

- [ ] `source-structure.json` has exhaustive `discoveredLocations`
- [ ] `entity-contracts.json` has full field definitions + `_coverage`
- [ ] `api-contracts.json` documents ALL directories + `_coverage`
- [ ] `function-registry.json` has ALL hooks/state + `_coverage`
- [ ] `copilot-instructions.md` has Coverage Summary section
- [ ] No `"type": "any"` without `_unresolved` entry
- [ ] **Validation passes with coverage thresholds met** (Hooks ≥80%, Classes ≥70%, State 100%, APIs 100%)
- [ ] **All 5 deep-dive files present**: `dependency-chains.md`, `data-flow.md`, `component-registry.md`, `testing-strategy.md`, `debugging-guide.md`
