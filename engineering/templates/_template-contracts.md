# Template Contracts

> **Purpose**: Define required sections for each template to prevent drift between templates and validation rules.
> **Usage**: Before completing any artifact generation mode, validate output against these contracts.

---

## Contract Format

Each template contract specifies:
- **required_sections**: Sections that MUST be present (BLOCKING if missing)
- **conditional_sections**: Sections required only under certain conditions
- **validation_file**: Links back to the validation rules that check this template

---

## Epic Template Contract

**Template**: [`epic.md`](./epic.md)  
**Used By**: FeaturePlanning mode  
**Validation**: [`workflow-validation.md`](../workflow-validation.md) § FeaturePlanning Mode

### Required Sections (BLOCKING)

| Section | Header Text | Purpose | Validation Check |
|---------|-------------|---------|------------------|
| `goal` | `**Goal**` | Business value statement | Non-empty, single sentence |
| `context` | `**Context**` | Trigger, preconditions, links | Must have Trigger and Links |
| `key_data` | `**Key Data Concepts**` | Inputs and Outputs | Both Inputs and Outputs present |
| `flows` | `**Functional Flows**` | Happy path + error paths | At least 2 flows |
| `acceptance` | `**Acceptance Criteria**` | Gherkin scenarios | At least 1 scenario in Given/When/Then |
| `scope` | `**Scope & Constraints**` | In scope / out of scope | Both Must Support and Out of Scope |

### Conditional Sections

| Section | Condition | Header Text |
|---------|-----------|-------------|
| `assumptions` | If provisional assumptions made | `**Assumptions Log**` |
| `figma_link` | If UI work involved | Link in Context section |

### Quality Rules

- [ ] No technical implementation details (database tables, API paths)
- [ ] All flows describe USER experience, not code behavior
- [ ] Figma link present if UI work is involved
- [ ] Labels include `needs-validation` if assumptions logged

### Error Codes

| Validation Failure | Error Code |
|--------------------|------------|
| Missing Goal section | `ENG-FP-ERR-001` |
| Missing Acceptance Criteria | `ENG-FP-ERR-002` |
| No Functional Flows defined | `ENG-FP-ERR-003` |
| UI work but no Figma link | `ENG-FP-ERR-004` |

---

## Tech Spec Template Contract

**Template**: [`tech-spec.md`](./tech-spec.md)  
**Used By**: TechSpec mode  
**Validation**: [`workflow-validation.md`](../workflow-validation.md) § TechSpec Mode

### Required Sections (BLOCKING)

| Section | Header Text | Purpose | Validation Check |
|---------|-------------|---------|------------------|
| `reference` | `**Reference**:` | Epic link | Valid Jira key |
| `projects` | `**Projects Affected**:` | Impacted projects | At least 1 project |
| `quick_ref` | `## 0. Quick Reference` | Component summary | Non-empty table |
| `epic_deconstruction` | `## 1. Epic Deconstruction` | Requirements summary | All Epic items covered |
| `architecture` | `## 2. Architecture & Patterns` | Compliance section | Pattern references present |
| `data_model` | `## 3. Data Model` | Entities & migrations | TypeScript interfaces |
| `api_contracts` | `## 4. API & Interface Contracts` | Endpoints | Full request/response types |
| `sequence` | `## 5. Sequence Diagram` | Flow diagram | ASCII or Mermaid diagram |
| `inventory` | `## 6. Implementation Inventory` | File list | All files listed with [NEW]/[MODIFY] |
| `verification` | `## 7. Verification Strategy` | Test plan | Unit + Integration tests |
| `summary` | `## 7. Summary` | Totals | File counts and projects |

### Conditional Sections

| Section | Condition | Header Text |
|---------|-----------|-------------|
| `ui_design` | If Epic has Figma links | `## 2.5 UI Design Specifications` |
| `migrations` | If database changes | `### Migration Strategy` in Data Model |

### Quality Rules

- [ ] Every architecture decision references `${COPILOT_INSTRUCTIONS_PATH}` or Context7
- [ ] All entities defined as TypeScript interfaces
- [ ] All endpoints have METHOD, Path, Request, Response
- [ ] All [TBD] items listed in Assumptions section
- [ ] No invented database fields, API paths, or Figma node IDs

### Error Codes

| Validation Failure | Error Code |
|--------------------|------------|
| Missing Epic reference | `ENG-TS-ERR-001` |
| No architecture validation | `ENG-TS-ERR-002` |
| Missing API contracts | `ENG-TS-ERR-003` |
| No verification strategy | `ENG-TS-ERR-004` |
| Core logic has [TBD] | `ENG-TS-ERR-005` |

---

## Task (Backend) Template Contract

**Template**: [`task-backend.md`](./task-backend.md)  
**Used By**: TaskPlanning mode (Backend layer)  
**Validation**: [`workflow-validation.md`](../workflow-validation.md) § TaskPlanning Mode

### Required Sections (BLOCKING)

| Section | Header Text | Purpose | Validation Check |
|---------|-------------|---------|------------------|
| `header` | `## Task:` | Task name | Non-empty name |
| `metadata` | `**Layer**:` / `**Category**:` / `**Type**:` / `**Source**:` | Classification | All 4 fields present |
| `target_files` | `### Target Files` | Files to create/modify | At least 1 file with absolute path |
| `dependencies` | `### Dependencies` | Task dependencies | "None" or valid task references |
| `pattern_context` | `### Pattern Context` | Architecture compliance | Pattern + Reference File + Rationale |
| `implementation` | `### Implementation Context` | Technical details | API Contract OR Entity Schema |
| `steps` | `### Implementation Steps` | Execution guide | At least 3 steps |
| `test_plan` | `### Test Plan` | Testing strategy | Test file + at least 2 test cases |
| `acceptance` | `### Acceptance Criteria` | Gherkin from Epic | At least 1 scenario |
| `summary` | `### Change Summary` | What/Why/How | All 3 fields |

### Conditional Sections

| Section | Condition | Header Text |
|---------|-----------|-------------|
| `validation_rules` | If API/Entity task | `**Validation Rules**:` |
| `cross_service` | If calling other services | `**Cross-Service Dependencies**:` |
| `before_after` | If MODIFY action | `**Before/After**:` code block |

### Quality Rules

- [ ] All file paths are ABSOLUTE (no `...` placeholders)
- [ ] Category matches `file-categorization.json` categories
- [ ] API Contract copied from Tech Spec § 4 (not invented)
- [ ] Test cases map to Epic acceptance criteria
- [ ] At least 1 Happy Path + 1 Edge/Error case

### Error Codes

| Validation Failure | Error Code |
|--------------------|------------|
| File path has placeholder | `ENG-TP-ERR-001` |
| Missing Category field | `ENG-TP-ERR-002` |
| No API Contract for API task | `ENG-TP-ERR-003` |
| Less than 2 test cases | `ENG-TP-ERR-004` |
| Missing Pattern Context | `ENG-TP-ERR-005` |

---

## Task (Frontend) Template Contract

**Template**: [`task-frontend.md`](./task-frontend.md)  
**Used By**: TaskPlanning mode (Frontend layer)  
**Validation**: [`workflow-validation.md`](../workflow-validation.md) § TaskPlanning Mode

### Required Sections (BLOCKING)

| Section | Header Text | Purpose | Validation Check |
|---------|-------------|---------|------------------|
| `header` | `## Task:` | Task name | Non-empty name |
| `metadata` | `**Layer**:` / `**Category**:` / `**Type**:` / `**Source**:` | Classification | All 4 fields present |
| `target_files` | `### Target Files` | Files to create/modify | At least 1 file with absolute path |
| `dependencies` | `### Dependencies` | Task dependencies | "None" or valid task references |
| `pattern_context` | `### Pattern Context` | Architecture compliance | Pattern + Reference Component + Rationale |
| `ui_guide` | `### UI Implementation Guide` | Design specifications | Token Mapping table present |
| `component_instances` | `**Component Instances**` | Reusable components | At least 1 instance if UI task |
| `steps` | `### Implementation Steps` | Execution guide | At least 3 steps |
| `test_plan` | `### Test Plan` | Testing strategy | Test file + at least 2 test cases |
| `acceptance` | `### Acceptance Criteria` | Gherkin from Epic | At least 1 scenario |
| `summary` | `### Change Summary` | What/Why/How | All 3 fields |

### Conditional Sections

| Section | Condition | Header Text |
|---------|-----------|-------------|
| `figma_reference` | If Figma link in Epic | `**Figma Reference**:` with URL |
| `visual_reference` | If Figma extracted | `**Visual Reference (Screenshot)**:` |
| `api_integration` | If calling APIs | `### API Integration` |
| `interactive_states` | If interactive component | `**Interactive States & Transitions**:` |
| `figma_issues` | If Figma extraction had issues | `### ⚠️ Figma Data Quality Issues` |

### Quality Rules

- [ ] All file paths are ABSOLUTE (no `...` placeholders)
- [ ] Figma link present if Epic has Figma link
- [ ] Token Mapping table maps Figma values to project tokens
- [ ] Component Instances table identifies reusable components
- [ ] At least 1 Happy Path + 1 Edge/Error case in test plan
- [ ] Interactive states documented if component is interactive

### Error Codes

| Validation Failure | Error Code |
|--------------------|------------|
| File path has placeholder | `ENG-TP-ERR-001` |
| Missing UI Implementation Guide | `ENG-TP-ERR-006` |
| No Component Instances for UI task | `ENG-TP-ERR-007` |
| Missing Token Mapping | `ENG-TP-ERR-008` |
| Figma link in Epic but not extracted | `ENG-TP-ERR-009` |

---

## Validation Protocol

### Before Completing FeaturePlanning Mode

```markdown
## Epic Contract Validation

- [ ] Goal section present and non-empty
- [ ] Context section has Trigger and Links
- [ ] Key Data Concepts has Inputs and Outputs
- [ ] At least 2 Functional Flows (Happy + Error)
- [ ] Acceptance Criteria in Gherkin format
- [ ] Scope & Constraints has Must Support and Out of Scope
- [ ] If UI work: Figma link in Context
- [ ] If assumptions made: Assumptions Log section added

**Result**: [ ] PASS / [ ] FAIL (list error codes)
```

### Before Completing TechSpec Mode

```markdown
## Tech Spec Contract Validation

- [ ] Epic reference with valid Jira key
- [ ] Projects Affected listed
- [ ] Quick Reference table populated
- [ ] Architecture section references copilot-instructions.md
- [ ] Data Model has TypeScript interfaces
- [ ] API Contracts have full request/response types
- [ ] If Figma in Epic: UI Design Specifications section present
- [ ] Implementation Inventory has all files with [NEW]/[MODIFY]
- [ ] Verification Strategy has Unit + Integration tests
- [ ] Summary has file counts

**Result**: [ ] PASS / [ ] FAIL (list error codes)
```

### Before Completing TaskPlanning Mode

```markdown
## Task Contract Validation (Per Task)

**For Backend Tasks**:
- [ ] Layer/Category/Type/Source all present
- [ ] Target Files have absolute paths (no `...`)
- [ ] Dependencies specified or "None"
- [ ] Pattern Context complete (Pattern + Reference + Rationale)
- [ ] Implementation Context has API Contract or Entity Schema
- [ ] Test Plan has ≥2 test cases
- [ ] Acceptance Criteria from Epic

**For Frontend Tasks**:
- [ ] All Backend requirements PLUS:
- [ ] UI Implementation Guide present
- [ ] Token Mapping table populated
- [ ] Component Instances identified
- [ ] If Figma in Epic: Figma Reference extracted

**Result**: [ ] PASS / [ ] FAIL (list error codes)
```

---

## Updating Contracts

When updating a template:

1. **Update the template file** (`epic.md`, `tech-spec.md`, etc.)
2. **Update this contract** (`_template-contracts.md`) to reflect new/removed sections
3. **Update validation rules** (`workflow-validation.md`) to match
4. **Update error codes** (`../../../shared/error-codes.md`) if new failures added

> [!WARNING]
> **Sync Requirement**: These three files MUST stay in sync:
> - Template file (structure)
> - Contract file (requirements)
> - Validation file (checks)
>
> Failure to sync causes template drift and incorrect validations.
