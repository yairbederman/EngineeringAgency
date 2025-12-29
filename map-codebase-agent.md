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
1. **Read**: `${ARCHITECT_ROOT}/../shared/configuration.md` (Global constants)
2. **Read**: `${ARCHITECT_ROOT}/configuration.md` (Agent specifics)

Where `ARCHITECT_ROOT` = `./mapcodebase`

---

## Project Validation (Prerequisite)

Before running any phases, validate the target project:

1. **Read** `${GLOBAL_WORKFLOWS_ROOT}/shared/configuration.md`
2. **Check**: Is the target project listed in the Registered Projects table?
   - **If YES**: Proceed to Execution
   - **If NO**: 
     - **STOP**: "Project not registered. Add it to `shared/configuration.md` first."
     - Provide guidance on how to add the project

> **Why**: Ensures all analyzed projects are tracked centrally, enabling `/system-architecture-agent` to discover them.

---

## Execution

> [!CAUTION]
> **⛔ BLOCKING: Phase 0 MUST be executed first.** Do NOT proceed to any other phase until the `.ai-instructions/` folder has been deleted.

---

### Phase 0: Clean Slate (BLOCKING GATE)

> [!WARNING]
> **THIS PHASE IS NOT OPTIONAL.** Skipping this phase will result in stale files remaining. The agent MUST execute the delete command before any analysis.

**Step 0: User Confirmation (MANDATORY)**

> [!CRITICAL]
> **STOP**: You are about to permanently delete files.
> **Ask the user**: "I am about to delete the `.ai-instructions` folder for [Project Name] to ensure a clean extraction. Do I have your approval to proceed?"
> **Wait** for explicit "Yes" or "Approved".

**Step 1: Delete existing artifacts**

For EACH target project, execute:

```bash
# Delete the entire .ai-instructions folder
rm -rf ${PROJECT_ROOT}/.ai-instructions/
```

**Step 2: Verify deletion**

After executing the delete command, verify the folder no longer exists:

```bash
# Verify deletion (should return "not found" or empty)
ls ${PROJECT_ROOT}/.ai-instructions/ 2>&1 || echo "Folder deleted successfully"
```

**Step 3: Confirmation gate**

> [!IMPORTANT]
> **DO NOT PROCEED** to Phase 1 until:
> - [ ] Delete command executed for ALL target projects
> - [ ] Verification confirms folders are deleted
> - [ ] If any folder still exists, re-execute delete

**Why this matters**: Old files that are no longer applicable will remain and cause confusion. Fresh regeneration ensures all artifacts are current and consistent.

---

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
**Output**: `analysis/function-registry.json`, `deep-dive/*` (all deep-dive files)
- Map service → service dependencies
- Map controller → service → external service chains
- Document cross-project dependencies (calls to other APIs)
- Generate ALL deep-dive documentation files

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

### Phase 6: Final Verification (BLOCKING GATE)

> [!WARNING]
> **THIS PHASE IS NOT OPTIONAL.** Do NOT report completion until all verification steps pass.

**Step 1: List all generated files**

For EACH target project, list the contents of `.ai-instructions/`:

```bash
find ${PROJECT_ROOT}/.ai-instructions/ -type f
```

**Step 2: Verify Required Files Exist**

Check EACH required file (see "Generated Artifacts Summary" below):

| File | Status |
|------|--------|
| `copilot-instructions.md` | [ ] Exists |
| `analysis/techstack.md` | [ ] Exists |
| `analysis/source-structure.json` | [ ] Exists |
| `analysis/entity-contracts.json` | [ ] Exists |
| `analysis/api-contracts.json` | [ ] Exists |
| `analysis/function-registry.json` | [ ] Exists |
| `analysis/file-categorization.json` | [ ] Exists |
| `deep-dive/dependency-chains.md` | [ ] Exists |
| `deep-dive/data-flow.md` | [ ] Exists |

**Step 3: Check for stale files**

If ANY file exists in `.ai-instructions/` that is NOT in the required or conditional lists:
- **Option A**: Regenerate it with current content
- **Option B**: Delete it (and document why in summary)

> [!IMPORTANT]
> **Since Phase 0 deleted all files**, there should be NO stale files. If stale files exist, Phase 0 was not executed correctly. Re-run from Phase 0.

**Step 4: Confirm completion**

> [!CAUTION]
> **DO NOT mark workflow complete** until:
> - [ ] All required files exist for ALL target projects
> - [ ] No stale files remain
> - [ ] Coverage thresholds were met (Phase 4.5)

### Validation
**Read**: `${ARCHITECT_ROOT}/_validation-rules.md`
- Run completeness checks
- Output validation summary

---

## Generated Artifacts Summary

After successful execution, the following files **MUST** exist in `${AI_INSTRUCTIONS_ROOT}`:

### Required Files (Always Generated)

```
.ai-instructions/
├── copilot-instructions.md       # Master AI instructions (entry point)
├── analysis/
│   ├── techstack.md              # Stack detection results
│   ├── source-structure.json     # Discovered locations + file counts
│   ├── entity-contracts.json     # Type definitions with fields
│   ├── api-contracts.json        # REST endpoints with validation
│   ├── function-registry.json    # Service dependencies
│   └── file-categorization.json  # Files grouped by layer
└── deep-dive/
    ├── dependency-chains.md      # Controller → Service → External chains
    └── data-flow.md              # Data object transformations
```

### Conditional Files (Generated When Applicable)

```
.ai-instructions/
├── analysis/
│   ├── design-tokens.json        # (Frontend only) CSS/Tailwind tokens
│   └── component-registry.json   # (Frontend only) React/Vue components
└── deep-dive/
    ├── component-registry.md     # (Frontend only) Component documentation
    ├── debugging-guide.md        # (If complex error handling exists)
    └── testing-strategy.md       # (If test files exist)
```

### Regeneration Rule

> [!WARNING]
> If ANY file in `.ai-instructions/` exists but is NOT in the lists above, either:
> 1. **Regenerate it** with fresh content and timestamp, OR
> 2. **Delete it** if no longer applicable (explain why)

---

## Success Criteria

- [ ] `source-structure.json` has exhaustive `discoveredLocations`
- [ ] `entity-contracts.json` has full field definitions + `_coverage`
- [ ] `api-contracts.json` documents ALL controllers + `_coverage`
- [ ] `function-registry.json` has ALL services/dependencies + `_coverage`
- [ ] `copilot-instructions.md` has Architecture Diagram + Critical Rules
- [ ] `file-categorization.json` groups files by layer/category
- [ ] `deep-dive/dependency-chains.md` has controller→service chains
- [ ] `deep-dive/data-flow.md` has data object transformations
- [ ] ALL existing files in `.ai-instructions/` have been regenerated
- [ ] No `"type": "any"` without `_unresolved` entry
- [ ] **Coverage thresholds met** (Controllers 100%, Services ≥50%)

