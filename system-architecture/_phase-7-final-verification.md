# Phase 7: Final Verification

> **Goal**: Verify all system architecture artifacts were generated correctly and the interactive viewer renders without errors.

---

## Required Artifacts

All files below MUST exist at the end of the workflow:

| Category | File | Path |
|----------|------|------|
| **Analysis** | Project Inventory | `${OUTPUT_ROOT}/analysis/project-inventory.json` |
| **Analysis** | Service Topology | `${OUTPUT_ROOT}/analysis/service-topology.json` |
| **Analysis** | Cross-Service APIs | `${OUTPUT_ROOT}/analysis/cross-service-apis.json` |
| **Analysis** | Unified Domain Model | `${OUTPUT_ROOT}/analysis/unified-domain-model.json` |
| **Documentation** | System Architecture | `${OUTPUT_ROOT}/${SYSTEM_NAME}-system-architecture.md` |
| **Deep Dive** | End-to-End Flows | `${OUTPUT_ROOT}/deep-dive/end-to-end-flows.md` |
| **Deep Dive** | Cross-Cutting Concerns | `${OUTPUT_ROOT}/deep-dive/cross-cutting-concerns.md` |
| **Deep Dive** | ASCII Architecture | `${OUTPUT_ROOT}/deep-dive/ascii-architecture.md` |
| **Interactive** | HTML Viewer | `${OUTPUT_ROOT}/${OUTPUT_FILE}` |

---

## Verification Steps

### Step 1: Enumerate Output Directory

```
list_dir("${OUTPUT_ROOT}")
list_dir("${OUTPUT_ROOT}/analysis")
list_dir("${OUTPUT_ROOT}/deep-dive")
```

### Step 2: Validate All Required Files Exist

For each file in the Required Artifacts table:
1. Verify file exists using `list_dir` results
2. If missing → **FAIL** - Report which file is missing and which phase should have generated it

### Step 3: Validate JSON Files Are Valid

For each `.json` file in `${OUTPUT_ROOT}/analysis/`:
1. Read file contents
2. Verify valid JSON structure
3. Check for required top-level keys:
   - `project-inventory.json`: `projects[]`, `_metadata`
   - `service-topology.json`: `services[]`, `dependencies[]`, `layers`
   - `cross-service-apis.json`: `calls[]`, `_verificationSummary`
   - `unified-domain-model.json`: `domains[]`, `canonicalEntities`

### Step 4: Validate Documentation Files

For `${SYSTEM_NAME}-system-architecture.md`:
- [ ] Contains Mermaid diagram (```mermaid block)
- [ ] Contains ASCII system architecture diagram (box-drawing characters)
- [ ] Contains Project Responsibilities table
- [ ] Contains Cross-Service API Reference section
- [ ] Contains Domain Model section

For `deep-dive/ascii-architecture.md`:
- [ ] Contains System Architecture ASCII diagram
- [ ] Contains Software Architecture ASCII diagram
- [ ] Contains Dependency Matrix
- [ ] Uses valid UTF-8 box-drawing characters

### Step 4.5: Validate Dependency Graph Integrity (BLOCKING)

> [!CAUTION]
> **⛔ BLOCKING**: Before testing the interactive viewer, perform these numeric validations.
> If any check fails, do NOT proceed to Step 5. Return to the failing phase to fix data quality issues.

**1. Service Count Consistency:**
```
project-inventory.json.readyProjects == service-topology.json.services.length
```
If mismatch → Return to Phase 1 or Phase 2

**2. Dependency Evidence Quality:**
For each dependency in `service-topology.json.dependencies[]`:
- [ ] `codeEvidence` does NOT contain: "inferred", "assumed", "pattern", "standard"
- [ ] `verified` is `true`
- [ ] Evidence contains file reference and line number

Count check: `dependencies with quality issues == 0`
If issues found → Return to Phase 2 to fix evidence

**3. Internal Module Coverage:**
```
services_with_modules = COUNT(services WHERE internalModules.length > 0)
total_services = services.length
coverage_ratio = services_with_modules / total_services
```
- Minimum acceptable: `coverage_ratio >= 0.3` (at least 30% of services have enumerated modules)
- Optimal: `coverage_ratio >= 0.7` (70%+ coverage)

If below minimum → Return to Phase 2 Step 1.5

**4. Cross-Service API Coverage:**
```
topology_dependencies = service-topology.json.dependencies.length
documented_calls = cross-service-apis.json.crossServiceCalls.length
```
- Every verified HTTP dependency should have at least one documented call
- If `documented_calls < http_dependencies` → Return to Phase 3

**Validation Report Format:**
```markdown
### Data Integrity Validation
| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| Service Count | <N> | <N> | ✅/❌ |
| Evidence Quality Issues | 0 | <N> | ✅/❌ |
| Module Coverage | ≥30% | <N>% | ✅/❌ |
| API Documentation | ≥<M> | <N> | ✅/❌ |
```

### Step 5: Validate Interactive Viewer

For `${OUTPUT_FILE}`:
1. Verify file exists
2. Open in browser using `${MCP_BROWSER_ACTION}`:
   - Navigate to `file:///${OUTPUT_ROOT}/${OUTPUT_FILE}`
   - Verify Mermaid diagrams render (no syntax errors)
   - Verify sidebar shows all projects
   - Verify at least one node is clickable for drill-down

### Step 6: Check for Stale Files

Compare files in `${OUTPUT_ROOT}` against Required Artifacts:
- Any file NOT in the Required Artifacts list → **WARNING** - May be stale from previous run
- Report stale files but do NOT auto-delete (user may have custom files)

---

## Verification Report

Generate a summary at the end:

```markdown
## System Architecture Verification Report

**Generated**: {timestamp}
**Output Root**: ${OUTPUT_ROOT}

### Artifact Status

| Artifact | Status | Notes |
|----------|--------|-------|
| project-inventory.json | ✅ Valid | {project count} projects |
| service-topology.json | ✅ Valid | {service count} services, {dependency count} dependencies |
| cross-service-apis.json | ✅ Valid | {call count} cross-service calls |
| unified-domain-model.json | ✅ Valid | {entity count} canonical entities |
| ${SYSTEM_NAME}-system-architecture.md | ✅ Valid | All sections present |
| end-to-end-flows.md | ✅ Valid | {flow count} flows documented |
| cross-cutting-concerns.md | ✅ Valid | Patterns documented |
| ascii-architecture.md | ✅ Valid | System & Software ASCII diagrams present |
| ${OUTPUT_FILE} | ✅ Renders | Diagrams visible, navigation works |

### Warnings
- {list any stale files or minor issues}

### Result
✅ **PASS** - All artifacts generated and validated
```

---

## Failure Handling

If verification fails:

| Failure Type | Action |
|--------------|--------|
| Missing file | Report which phase should have generated it, re-run that phase |
| Invalid JSON | Report parse error, re-run generating phase |
| Missing Mermaid diagram | Re-run Phase 5 |
| Interactive viewer doesn't render | Check for Mermaid syntax errors in Phase 6 output |
| Stale files detected | Warn user, suggest re-running with Phase 0 clean slate |

> [!CAUTION]
> **Do NOT mark workflow as complete if any required artifact is missing or invalid.**
