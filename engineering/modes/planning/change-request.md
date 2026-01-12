# Engineering Agent – ChangeRequest Mode

> **Trigger**: User makes a request that modifies existing planned/in-progress work.
> **Purpose**: Maintain documentation alignment when product changes occur mid-flow.

---

## 1. Context Gathering (MANDATORY FIRST STEP)

### 1.1 Determine Workflow Phase

Check current artifacts to determine phase:

| Phase | Indicator | Change Impact Level |
|-------|-----------|---------------------|
| `PLANNING_SPEC` | ProductSpec exists, no Epic | Low |
| `PLANNING_EPIC` | Epic exists, no TechSpec | Medium |
| `PLANNING_TECH` | TechSpec exists, no Tasks | High |
| `PLANNING_TASKS` | Tasks exist, none started | High |
| `EXECUTION` | Task(s) in progress/done | **⚠️ CRITICAL** |
| `COMPLETION` | PR submitted | **🔴 SCOPE CREEP** |

**How to Detect Phase**:
```
1. Check ${WORKSPACE_ROOT}/.specs/ exists
   - NO → Standard linear flow (not a change request)
   
2. Check for existing artifacts:
   a. product-specs/SPEC-* exists → At least PLANNING_SPEC
   b. epics/EPIC-* exists → At least PLANNING_EPIC
   c. *-tech-spec.md exists → At least PLANNING_TECH
   d. tasks/*.md exists → PLANNING_TASKS or beyond
   
3. Check task status in _registry.json:
   a. Any task with status "in-progress" or "done" → EXECUTION
   b. All tasks "open" → PLANNING_TASKS
```

### 1.2 Identify Active Feature

If multiple epics exist, **disambiguate target** before proceeding:

```
Check: ${WORKSPACE_ROOT}/.specs/epics/

If count(EPIC-*) > 1:
  Present to user:
  
  "I see multiple features in progress:
   - 🟢 EPIC-001: {title} ({phase})
   - 🟡 EPIC-002: {title} ({phase})
   
   Which feature does this change apply to?"
   
  Wait for user response.
```

### 1.3 Capture Change Source

Ask or infer the source of the change:

| Source | Code | Examples |
|--------|------|----------|
| Product Manager | `PM` | Spec update, priority shift |
| User/Developer | `DEV` | Implementation discovery |
| Stakeholder | `STAKE` | Demo feedback, executive request |
| QA/Testing | `QA` | Bug found during testing |
| Technical Debt | `TECH` | Refactor need discovered |
| External | `EXT` | API change, vendor update |

**Default**: If not explicitly stated, assume `DEV` for developer requests.

---

## 2. Change Classification (DECISION TREE)

> **Reference**: See `${AGENT_ROOT}/shared/change-classification-tree.md` for full decision tree.

### Quick Classification

| Question | Yes → | No → |
|----------|-------|------|
| Does this change acceptance criteria? | **Scope Change** | Continue |
| Does this add new user-facing behavior? | **Scope Change** | Continue |
| Does this require DB/API schema changes? | **Major Change** | Continue |
| Can this be done in <2 hours? | **Minor Adjustment** | **Scope Change** |

### Classification Definitions

| Classification | Definition | Approval Required |
|----------------|------------|-------------------|
| **Minor** | Clarification, typo fix, small tweak | None (auto-apply) |
| **Scope** | Changes what we deliver or how it works | User must confirm: "Acknowledge scope change" |
| **Major** | Fundamental change to architecture or goals | Block until explicit PM/TechLead confirm |

---

## 3. Priority Assignment

| Priority | Definition | SLA | Action |
|----------|------------|-----|--------|
| 🔴 **P0 - Blocker** | Cannot ship without this | Immediate | Pause current work, address now |
| 🟠 **P1 - High** | Significantly affects value | Within current sprint | Add to current epic, prioritize |
| 🟡 **P2 - Medium** | Enhances experience | Next sprint | Log in backlog with epic link |
| 🟢 **P3 - Low** | Nice to have | Future | Log as enhancement |

### Priority Determination

```
Ask (or infer from context):

1. "Can we ship the MVP without this change?"
   - Yes → P2 or P3
   - No → P0 or P1
   
2. "Will users/stakeholders notice if this is missing?"
   - Yes → Bump up one level
   - No → Keep current level
```

### Default Priority by Change Type

| Change Type | Default Priority |
|-------------|------------------|
| Refinement (clarification) | P3 |
| Increment (new feature) | P2 |
| Adjustment (fixing what's planned) | P1 |
| Blocker/Bug | P0 |

---

## 4. Impact Analysis

### 4.1 Artifact Impact

For each artifact type, determine if update is needed:

| Artifact | Check | Update If |
|----------|-------|-----------|
| ProductSpec | Does change affect requirements? | Any scope change |
| Epic | Does change affect acceptance criteria? | Scope or increment |
| TechSpec | Does change affect architecture? | Technical scope changes |
| Tasks | Does change affect task scope? | Any scope change |

### 4.2 Code Impact (EXECUTION Phase Only)

If phase is `EXECUTION`, analyze code impact:

```
1. Get task status from _registry.json:
   
   | Task Status | Impact Level | Action |
   |-------------|--------------|--------|
   | "open" | None | Update task description |
   | "in-progress" | Partial | Notify developer, add rework note |
   | "done" (local) | Full | Mark for refactor, estimate rework |
   | "done" (merged) | **Breaking** | STOP - Escalate to PM |

2. List affected files (from TechSpec mappings):
   - Extract file paths from implementation notes
   - Estimate LOC impact (rough)

3. Present "Change Cost Summary" to user
```

### 4.3 Test Impact

If change affects code or acceptance criteria:

```
1. Identify affected tests from TechSpec test matrix
2. Flag:
   - Tests needing modification
   - New tests needed
   - Tests that may be irrelevant
3. Include in impact summary
```

---

## 5. Version & Archive

Before applying any change:

### 5.1 Archive Current Versions

```
For each affected artifact:
  1. Call storage.archiveArtifact(type, id)
  2. This copies current file to .versions/{type}_v{version}.md
  3. Increments version number:
     - Scope change: Minor bump (1.0 → 1.1)
     - Major change: Major bump (1.0 → 2.0)
```

### 5.2 Apply Changes

```
1. Update each artifact with new content
2. Add change log entry via storage.logChange()
3. Update cross-links between artifacts
```

---

## 6. Approval Flow

| Classification | Approval Required | User Action |
|----------------|-------------------|-------------|
| **Minor** | None | Auto-apply, inform user |
| **Scope** | User confirmation | Must type: "Acknowledge scope change" |
| **Major** | Explicit approval | Must type: "Approve major change" with stakeholder mention |

### Scope Change Dialog

```markdown
┌────────────────────────────────────────────────────────────────┐
│ ⏸️ SCOPE CHANGE APPROVAL REQUIRED                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ **Change Classification**: Scope Change                        │
│ **Priority**: {priority}                                       │
│ **Source**: {source}                                           │
│                                                                │
│ **Summary**: {description}                                     │
│                                                                │
│ **Impact**:                                                    │
│ - ProductSpec: Update required                                 │
│ - Epic: Add acceptance criteria                                │
│ - TechSpec: {impact}                                           │
│ - Tasks: {new/modified count}                                  │
│                                                                │
│ **Estimated Effort**: {rework estimate if applicable}          │
│                                                                │
│ Reply with:                                                    │
│ • `Acknowledge scope change` - Apply updates, continue         │
│ • `Defer to backlog` - Log for later, continue current work    │
│ • `Cancel` - Discard change request                            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. Cascade & Sync

After approval, update all downstream artifacts:

### 7.1 Cascade Logic

```
1. Call storage.cascadeChange(rootArtifact, change)

2. Cascade order:
   ProductSpec
     └── Epic (update acceptance criteria)
           └── TechSpec (update architecture sections)
                 └── Tasks (mark affected, create new if needed)

3. For each cascaded artifact:
   - Add cross-reference in change log
   - If task: Add "needs-review" marker if appropriate
```

### 7.2 Verify Cross-Links

```
After cascade:
1. Verify all links in artifacts are valid
2. Update Links section in each affected document
3. Confirm version numbers are consistent
```

---

## 8. Stakeholder Report (Optional)

On user request, generate Change Summary Report:

```
User says: "Generate change report" or "Show what changed"

1. Call storage.generateChangeReport(epicId, dateRange)
2. Present formatted report
3. Optionally save to epic folder
```

---

## Resume Protocol

After change is processed:

```
1. Display summary:
   "✅ Change applied to: ProductSpec (v1.1), Epic, TechSpec"
   
2. Show current task status:
   "Current tasks:
    - 🟢 TASK-001: Hero (done)
    - 🟡 TASK-002: Services (in progress) 
    - 🔴 TASK-003: Footer (needs-review)
    - 🆕 TASK-004: Blog Schema (new)"
    
3. Prompt for next action:
   "Which task would you like to work on?"
```

---

## Error Handling

| Scenario | Response |
|----------|----------|
| No existing artifacts | "No artifacts found. Starting standard planning flow." |
| User rejects classification | "Please specify if this is: Minor / Scope / Major" |
| Cascade partially fails | "⚠️ Updated X, Y. Failed to update Z. Please verify manually." |
| Rollback requested mid-change | Abort current change, offer rollback options |

---

## Quick Reference

```
1. DETECT  → Is this a change to existing work?
2. CONTEXT → What phase? Which feature? What source?
3. CLASSIFY → Minor / Scope / Major?
4. ANALYZE → What's impacted? Code? Tests?
5. ARCHIVE → Save current versions
6. APPROVE → Get user confirmation if needed
7. CASCADE → Update all downstream artifacts
8. RESUME  → Return to current workflow position
```
