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

### Phase 2.5: Extract Database Schema (Backend Only)
**Read**: `${ARCHITECT_ROOT}/_phase-2.5-extract-database-schema.md`
**Output**: `analysis/database-schema.json`
- **Trigger**: Only for Backend projects with ORM/database layer (JPA, TypeORM, Prisma, etc.)
- Extract table definitions, columns, constraints, indexes
- Map entity-to-table relationships
- Document migration framework if present

### Phase 2.6: Extract Validation Schemas (Conditional)
**Read**: `${ARCHITECT_ROOT}/_phase-2.6-extract-validation-schemas.md`
**Output**: `analysis/validation-schemas.json`
- **Trigger**: Only when validation libraries detected (Zod, Yup, Joi, class-validator, etc.)
- Extract field-level validation rules
- Map schemas to entities
- Document custom validators

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

### Phase 3.7: Extract State Contracts (Frontend Only)
**Read**: `${ARCHITECT_ROOT}/_phase-3.7-extract-state-contracts.md`
**Output**: `analysis/state-contracts.json`
- **Trigger**: Only for Frontend projects with Redux/Zustand/Pinia/MobX/Recoil
- Extract full action payloads with typed parameters
- Document async thunk lifecycle (pending/fulfilled/rejected)
- Extract selector return types and dependencies

### Phase 4: Map Dependencies
**Read**: `${ARCHITECT_ROOT}/_phase-4-map-dependencies.md`
**Output**: `analysis/function-registry.json`, `deep-dive/*` (all deep-dive files)
- Map service → service dependencies
- Map controller → service → external service chains
- Document cross-project dependencies (calls to other APIs)
- Generate ALL deep-dive documentation files

### Phase 4.2: Extract Inter-Service Contracts (Backend Only)
**Read**: `${ARCHITECT_ROOT}/_phase-4.2-extract-inter-service-contracts.md`
**Output**: `analysis/inter-service-contracts.json`
- **Trigger**: Only for Backend projects calling other internal services
- Extract full request/response DTOs for inter-service calls
- Verify contracts against target service's api-contracts (if available)
- Document resilience patterns (circuit breaker, retry, timeout)

### Phase 4.3: Extract External Integrations (Conditional)
**Read**: `${ARCHITECT_ROOT}/_phase-4.3-extract-external-integrations.md`
**Output**: `analysis/external-integrations.json`
- **Trigger**: Only when third-party SDKs detected (Stripe, SendGrid, AWS, etc.)
- Extract wrapper methods with parameter and return types
- Document configuration requirements (env vars)
- Map webhook handlers if present

### Phase 4.4: Extract Error Taxonomy (MANDATORY - Universal)
**Read**: `${ARCHITECT_ROOT}/_phase-4.4-extract-error-taxonomy.md`
**Output**: `analysis/error-taxonomy.json`

> [!IMPORTANT]
> **This phase is ALWAYS executed** for ALL project types. Error handling patterns exist in every codebase and must be documented.

- Extract custom error classes with fields
- Document error codes and their meanings
- Map HTTP status code conventions
- Document error propagation patterns across layers

### Phase 4.5: Enforcement Gate (BLOCKING)
**Before proceeding to Phase 5**, check coverage thresholds dynamically:

**Dynamic Threshold Rules** (based on `source-structure.json.fileCount` keys):

| Category Pattern | Threshold | Validation Source |
|------------------|-----------|-------------------|
| `controllers`, `routes`, `endpoints` | 100% | `api-contracts.json` |
| `services`, `providers`, `handlers` | ≥50% | `function-registry.json` |
| `components`, `pages`, `views` | ≥70% | `api-contracts.json` |
| `hooks`, `composables` | ≥80% | `function-registry.json` |
| `state`, `stores`, `slices` | 100% | `function-registry.json` |
| `entities`, `models`, `types` | ≥30% | `entity-contracts.json` |

> For any `fileCount` key discovered in Phase 1: match the closest pattern above and apply its threshold.

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
**Read**: `${ARCHITECT_ROOT}/_phase-6-final-verification.md`
- Verify all required files exist
- Check for stale files
- Confirm coverage thresholds were met

### Validation
**Read**: `${ARCHITECT_ROOT}/_validation-rules.md`
- Run completeness checks
- Output validation summary

---

## Generated Artifacts Summary

**Read**: `${ARCHITECT_ROOT}/_artifacts-reference.md` for the complete list of required and conditional files.

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

