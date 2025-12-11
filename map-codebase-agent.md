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

## Project Validation (Prerequisite)

Before running any phases, validate the target project:

1. **Read** `${GLOBAL_WORKFLOWS_ROOT}/shared/projects.md`
2. **Check**: Is the target project listed in the Registered Projects table?
   - **If YES**: Proceed to Execution
   - **If NO**: 
     - **STOP**: "Project not registered. Add it to `shared/projects.md` first."
     - Provide guidance on how to add the project

> **Why**: Ensures all analyzed projects are tracked centrally, enabling `/system-architecture-agent` to discover them.

---

## Execution

> [!IMPORTANT]
> **Always regenerate ALL artifacts fresh**, even if the `.ai-instructions/` folder already exists. Do NOT skip phases just because files are present – the goal is to produce updated artifacts with current timestamps.

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

### Phase 3.5: Extract Design Tokens (Frontend Only)
**Read**: `${ARCHITECT_ROOT}/_phase-3.5-extract-design-tokens.md`
**Output**: `analysis/design-tokens.json`
- **Trigger**: Only for Frontend projects with Tailwind/CSS variables
- Extract colors, spacing, fontSize, fontWeight, borderRadius, boxShadow
- Used by `/engineering-agent` for Figma-to-code token mapping

### Phase 3.6: Extract Component Registry (Frontend Only)
**Read**: `${ARCHITECT_ROOT}/_phase-3.6-extract-component-registry.md`
**Output**: `analysis/component-registry.json`
- **Trigger**: Only for Frontend projects with React/Vue/Angular/Svelte
- Extract component props, variants, and import paths
- Generate Figma-to-component mappings automatically
- Used by `/engineering-agent` for component instance matching

### Phase 4: Map Dependencies
**Read**: `${ARCHITECT_ROOT}/_phase-4-map-dependencies.md`
**Output**: `analysis/function-registry.json`, `deep-dive/dependency-chains.md`
- Map service → service dependencies
- Map controller → service → external service chains
- Document cross-project dependencies (calls to other APIs)

### Phase 4.5: Enforcement Gate (BLOCKING)
**Before proceeding to Phase 5**, check coverage thresholds based on project type:

**For Backend APIs (JVM/Spring Boot):**
| Category | Source Count | Extracted Count | Required |
|----------|-------------|-----------------|----------|
| Controllers | `source-structure.json.fileCount.controllers` | `api-contracts` controller count | 100% |
| Services | `source-structure.json.fileCount.services` | `function-registry.services` count | ≥50% |
| Classes | `source-structure.json.fileCount.classes` | `entity-contracts` with `kind: "class"` | ≥30% |

**For Frontend (Node/TypeScript):**
| Category | Source Count | Extracted Count | Required |
|----------|-------------|-----------------|----------|
| Hooks | `source-structure.json.fileCount.hooks.total` | `function-registry.hooks` count | ≥80% |
| Components | `source-structure.json.fileCount.components` | `api-contracts` pages/components | ≥70% |
| State | `source-structure.json.fileCount.state.slices` | `function-registry.stateModules` | 100% |

**If ANY threshold not met**:
1. **DO NOT proceed** to Phase 5
2. List specific undocumented files
3. Return to the failing phase with targeted extraction
4. Re-check thresholds after re-extraction

**Only proceed to Phase 5 when all thresholds are met.**

### Phase 5: Generate Master
**Read**: `${ARCHITECT_ROOT}/_phase-5-generate-master.md`
**Output**: `copilot-instructions.md`, `analysis/file-categorization.json`
- Include Coverage Summary section
- Surface all unresolved items
- Pattern enforcement rules (MUST/NEVER)
- Architecture diagrams

### Validation
**Read**: `${ARCHITECT_ROOT}/_validation-rules.md`
- Run completeness checks
- Output validation summary

---

## Generated Artifacts Summary

After successful execution, the following files should exist in `${AI_INSTRUCTIONS_ROOT}`:

```
.ai-instructions/
├── copilot-instructions.md     # Master AI instructions (entry point)
├── analysis/
│   ├── techstack.md            # Stack detection results
│   ├── source-structure.json   # Discovered locations + file counts
│   ├── entity-contracts.json   # Type definitions with fields
│   ├── api-contracts.json      # REST endpoints with validation
│   ├── function-registry.json  # Service dependencies
│   ├── file-categorization.json # Files grouped by layer
│   └── design-tokens.json      # (Frontend only) Colors, spacing, typography for Figma mapping
└── deep-dive/
    └── dependency-chains.md    # Controller → Service → External chains
```

---

## Success Criteria

- [ ] `source-structure.json` has exhaustive `discoveredLocations`
- [ ] `entity-contracts.json` has full field definitions + `_coverage`
- [ ] `api-contracts.json` documents ALL controllers + `_coverage`
- [ ] `function-registry.json` has ALL services/dependencies + `_coverage`
- [ ] `copilot-instructions.md` has Architecture Diagram + Critical Rules
- [ ] `file-categorization.json` groups files by layer/category
- [ ] `deep-dive/dependency-chains.md` has controller→service chains
- [ ] No `"type": "any"` without `_unresolved` entry
- [ ] **Coverage thresholds met** (Controllers 100%, Services ≥50%)
