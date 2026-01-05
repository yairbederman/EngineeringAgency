---
description: Validation rules to ensure extraction completeness
---

# Validation Rules

Run these checks after all phases complete to ensure extraction quality.

> **Error Codes**: When validations fail, emit structured error codes from `shared/error-codes.md`.
> Use format: `MCB-P45-XXX` for enforcement gate failures.

## Minimum Coverage Thresholds

| Category | Minimum | Action if Below | Error Code | Condition |
|----------|---------|-----------------|------------|-----------|
| Hooks | 80% | WARN and list undocumented files | `MCB-P45-003` | IF `detectedStack.hasFrontend` |
| Classes | 70% | WARN and list undocumented files | `MCB-P2-001` | IF `detectedStack.hasBackend` |
| State Modules | 100% | FAIL extraction | `MCB-P45-004` | IF `detectedStack.hasStateManagement` |
| API Endpoints | 100% | FAIL extraction | `MCB-P45-001` | IF `detectedStack.hasAPI` |
| Services | 50% | WARN and list undocumented | `MCB-P45-002` | IF `detectedStack.hasBackend` |

If threshold not met, output:
- **Error code** with severity and resolution
- List of undocumented files
- Reason for skip (utility, test, index-only, deprecated)

---

## Completeness Checks

### 1. API Coverage
```
directories in source-structure.json.discoveredLocations.apis
  == directories documented in api-contracts.json._coverage
```

### 2. Hook Coverage
```
files in each discoveredLocations.hooks directory
  == documented count in function-registry.json._coverage.hookDirectories
```

### 3. Type Coverage
```
files in discoveredLocations.types
  ~= types extracted in entity-contracts.json
```

### 4. State Coverage
```
files in discoveredLocations.state
  == modules in function-registry.json.stateModules
```

### 5. Database Schema Coverage (Backend Only)
```
IF detectedCapabilities.hasORM:
  entities in entity-contracts.json with ORM annotations
    == tables in database-schema.json
```

### 6. Validation Schema Coverage (Conditional)
```
IF detectedCapabilities.hasValidation:
  schemas in validation-schemas.json
    >= 80% of DTOs in entity-contracts.json with validation decorators
```

### 7. Inter-Service Contract Coverage (Backend Only)
```
IF detectedCapabilities.callsExternalServices:
  targets in inter-service-contracts.json
    == unique services called in function-registry.json.crossProjectDependencies
```

### 8. External Integration Coverage (Conditional)
```
IF detectedCapabilities.hasThirdPartySDKs:
  integrations in external-integrations.json
    == third-party packages in package.json/pom.xml with wrapper usage
```

### 9. Error Taxonomy Completeness (Universal)
```
error classes in error-taxonomy.json
  >= custom error/exception classes in codebase
error codes documented
  >= error codes used in error responses
```

## Quality Checks

| Check | Rule | Fail Action |
|-------|------|-------------|
| No `any` leakage | Zero `"type": "any"` without `_unresolved` entry | Log and continue |
| Enum values | All enums have `values[]`, not just names | Extract values |
| Resolved types | Request/response types have field breakdown | Cross-ref entity-contracts |
| Call chains | At least 3 chains in dependency-chains.md | Add more flows |
| **Validation rules** | All `requestFields` have `validation` object | **BLOCKING** |
| **Entity usage** | `_entityUsage` section exists in function-registry.json | **BLOCKING** |
| **No invalid skips** | Zero "Pending", "not fully extracted" in skip reasons | **BLOCKING** |

## Cross-Reference Checks

1. **API → Entity**: All types in `api-contracts.json` exist in `entity-contracts.json`
2. **Hook → State**: All state reads in `function-registry.json` reference valid state modules
3. **Hook → API**: All API calls in hooks reference documented API clients
4. **State → API** (NEW): All async thunks in `state-contracts.json` reference valid API clients
5. **DB → Entity** (NEW): All tables in `database-schema.json` map to entities in `entity-contracts.json`
6. **Validation → Entity** (NEW): All schemas in `validation-schemas.json` link to entities
7. **InterService → Config** (NEW): All targets in `inter-service-contracts.json` exist in `shared/configuration.md`
8. **Error → API** (NEW): Error codes in `error-taxonomy.json` appear in API error responses

## Code Evidence Requirement (For Cross-Service Accuracy)

> [!IMPORTANT]
> **Every external call and cross-service dependency MUST have code evidence.**
> This prevents assumption-based documentation that leads to incorrect Tech Specs.

**Every `calls.external` entry MUST have:**
1. **Client Name**: Which API client class is used
2. **Method Name**: Specific method being called
3. **Verification Reference**: Link to `api-contracts.json` entry
4. **Code Evidence**: File and line number where the call is made

**Invalid Outputs (BLOCKING)**:
| Pattern | Why Invalid | Resolution |
|---------|-------------|------------|
| `calls.external: ["<ApiName>"]` | No method specified | Extract specific method names |
| Missing `verifiedIn` field | Not cross-referenced | Verify against api-contracts.json |
| Missing `codeEvidence` field | No code location | Grep for actual call site |
| Assumed dependencies | Not verified in code | Search codebase for actual usage |

**Validation Check** (add to Phase 4.5 gate):
```bash
# Every external call should have verifiedIn field
grep -A3 '"external"' function-registry.json | grep -c 'verifiedIn'
```
If count doesn't match external calls count → FAIL

## Cross-Phase Consistency Check (Required)

Compare `source-structure.json` counts against extraction results:

| Source (Phase 1) | Extraction Output | Required Match | Condition |
|------------------|-------------------|----------------|-----------|
| `fileCount.classes` | `entity-contracts` entities with `kind: "class"` | ≥70% | IF `stack.isObjectOriented` |
| `fileCount.hooks.total` | `function-registry.hooks` count | ≥80% | IF `stack.hasHooks` |
| `fileCount.apis` | `api-contracts` endpoints (non-empty clients) | 100% | IF `stack.hasAPI` |
| `fileCount.state.slices` | `function-registry.stateModules` count | 100% | IF `stack.hasState` |

**On Mismatch Detected**:
1. DO NOT proceed to Phase 5
2. Log specific undocumented files
3. Return to the failing phase with targeted extraction
4. Only proceed when thresholds met

## Failure Handling

**For Coverage Threshold Failures (BLOCKING)**:
- Hook < 80%, Class < 70%, State < 100%, API < 100%
- **DO NOT proceed to Phase 5**
- Return to failing phase and complete extraction

**For Quality Check Failures (NON-BLOCKING)**:
- Unresolved `any` types, missing enum values, missing call chains
1. **Log** in the relevant `_coverage` or `_unresolved` section
2. **Continue** extraction—these don't block completion
3. **Surface** in `copilot-instructions.md` Coverage section

---

## Automated Validation Commands (MANDATORY)

Before proceeding to Phase 5, execute these grep checks:

### Check 1: Invalid Skip Reasons
```bash
# Must return ZERO matches to pass
grep -i -E "pending|not fully extracted|later" api-contracts.json function-registry.json
```
**If matches found**: HALT, return to failing phase, extract the skipped items.

### Check 2: Missing Validation Rules  
```bash
# Search for requestFields without validation object
# Any requestFields entry missing "validation" = FAIL
grep -A2 '"requestFields"' api-contracts.json | grep -v validation
```
**If matches found**: HALT, return to Phase 3, add validation metadata.

### Check 3: Missing _entityUsage
```bash
# Must find _entityUsage section
grep '_entityUsage' function-registry.json
```
**If NO match found**: HALT, return to Phase 4, build entity usage map.

### Check 4: Feature Patterns Table
```bash
# Must find Feature Patterns section in copilot-instructions.md
grep 'Feature Patterns' copilot-instructions.md
```
**If NO match found**: HALT, return to Phase 5, generate feature patterns table.

### Check 5: Contract Artifacts Verification

**Universal (ALWAYS Required)**:
```bash
# Error taxonomy is MANDATORY for ALL projects
ls error-taxonomy.json || FAIL "Phase 4.4 did not generate error-taxonomy.json"
```

**Conditional (Based on detectedCapabilities)**:
```bash
# For Backend projects with ORM
IF detectedCapabilities.hasORM: ls database-schema.json || FAIL

# For projects with validation libraries
IF detectedCapabilities.hasValidation: ls validation-schemas.json || FAIL

# For Frontend with state management  
IF detectedCapabilities.hasStateManagement: ls state-contracts.json || FAIL

# For Backend calling other services
IF detectedCapabilities.callsExternalServices: ls inter-service-contracts.json || FAIL

# For projects with third-party SDKs
IF detectedCapabilities.hasThirdPartySDKs: ls external-integrations.json || FAIL
```

**If any artifact missing when condition is met**: HALT and return to appropriate phase.

## Self-Check Command

Calculate coverage percentage:
```
coverage = (documented / found) × 100
status = coverage >= threshold ? "✅" : "⚠️"
```

At end of extraction, output:
```markdown
## Extraction Validation

| Check | Found | Documented | Coverage | Status |
|-------|-------|------------|----------|--------|
| Hooks | 62 | 50 | 81% | ✅ ≥80% |
| Classes | 164 | 120 | 73% | ✅ ≥70% |
| State Modules | 16 | 16 | 100% | ✅ Required |
| API Endpoints | 52 | 52 | 100% | ✅ Required |
| Types | 42 | 42 | 100% | ✅ |
| No `any` | - | - | 3 unresolved | ⚠️ See _unresolved |
| Call Chains | - | - | 4 flows | ✅ ≥3 required |
| Deep-Dive Files | 5 | 5 | 100% | ✅ All present |
```
