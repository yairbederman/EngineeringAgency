# Engineering Agent – ProductSpecReview Mode

> **Persona**: Load `${AGENT_ROOT}/personas/product-manager.md`

## 1. ProductSpecReview Mode – The "Definition of Ready" Gate

**Goal**: Analyze the Product Spec for ambiguity, conflicts, and missing **user behavior and business logic**. Focus on **UI/UX and business rules**, NOT technical implementation. **Do not generate Epics yet.**

**Trigger**: User provides a Confluence Product Spec link or feature description.

**Critical Rule**: The Product Manager is **non-technical**. Questions MUST focus on:
- **User behavior**: What happens when the user does X?
- **UI/UX**: What does the user see/experience?
- **Business logic**: What are the business rules and edge cases?

**NEVER ask about**:
- Database schemas, API contracts, or technical architecture
- Implementation details (e.g., "Should we use Redis?")
- Technology stack choices

---

## 1.5 Local Mode: Create Product Spec Artifact (MANDATORY)

> [!IMPORTANT]
> **Local mode MUST create `product-spec.md`** as the persistent source of truth.
> This mirrors how Atlassian mode uses Confluence for product specs.

### Step -1: Check for Existing Specs (MANDATORY)

> [!CAUTION]
> **Before creating a new spec, you MUST check for overlapping existing specs.**
> This prevents orphaned specs when scope changes or supersedes previous work.

**Pre-Creation Overlap Check**:

1. **Read Registry**: Load `.specs/_registry.json`
2. **List Active Specs**: Filter for specs with `status != "superseded"`
3. **Scope Comparison**: For each active spec, compare:
   - Feature area overlap (e.g., both involve "Journal" or "Blog")
   - Project overlap (same target project)
   - Functional overlap (shared user stories)

**If Overlap Detected**:

Present the user with options:

```markdown
⚠️ **Existing Spec Detected**

I found [SPEC-XXX: Title] which may overlap with your request:
- **Scope**: [Brief description of existing spec scope]
- **Status**: [draft/approved]
- **Linked Epic**: [EPIC-XXX or None]

**Options**:
1. **Expand existing** → Update SPEC-XXX to include new requirements
2. **Supersede** → Mark SPEC-XXX as superseded, create new spec
3. **Keep separate** → This is a different feature, create new spec

Reply with `1`, `2`, or `3`.
```

**On User Selection**:

| Choice | Action |
|--------|--------|
| `1` (Expand) | Load existing spec, add new requirements, update version |
| `2` (Supersede) | Set `existingSpec.status = "superseded"`, `existingSpec.superseded_by = newSpecId`, create new spec |
| `3` (Separate) | Proceed with new spec creation (no conflict) |

**Supersession Registry Update**:

```json
{
  "SPEC-001": {
    "status": "superseded",
    "superseded_by": "SPEC-002",
    "superseded_date": "2026-01-11"
  }
}
```

---

### Step 0: Persist Product Spec to Local Storage

| Input Source | Action |
|--------------|--------|
| User provides feature description in chat | Generate structured product spec, then `storage.createProductSpec(title, content)` |
| User provides a file/document | Parse content, then `storage.createProductSpec(title, content)` |
| Resuming existing spec | `storage.getProductSpec(specId)` to load existing |

**Product Spec Lifecycle**:

```
[User Input] → createProductSpec() → Analyze for Gaps → updateProductSpec() → Approved
                    ↓                                          ↑
              product-spec.md                           (add gaps/assumptions)
```

**When Gaps Are Identified**:
1. Add "Gap Analysis" section to product spec content
2. After user approves assumptions: `storage.updateProductSpec(specId, updatedContent)`
3. Update status to "approved" and `gapsResolved: true`

**When Proceeding to FeaturePlanning**:
1. Link product spec to epic: `storage.linkProductSpecToEpic(specId, epicId)`
2. This maintains bidirectional traceability

> [!NOTE]
> **Atlassian Mode**: Product specs are managed in Confluence directly via MCP.
> This section applies only to `local` storage mode.

## 2. Analysis Framework

**For each feature, ask**:

### **A. User Journey & Behavior**
1. **Triggering Action**: How does the user initiate this feature?
2. **Expected Outcome**: What does the user expect to happen?
3. **Edge Cases**: What happens when:
   - User provides invalid input?
   - No results are found?
   - System is slow or unavailable?

### **B. UI/UX Clarity**
1. **Visual Feedback**: What does the user see while waiting (loading states)?
2. **Error Messaging**: What error messages should appear (exact wording)?
3. **Empty States**: What displays when there's no data?
4. **Mobile vs Desktop**: Are there different behaviors for mobile/desktop?

### **C. Business Rules**
1. **Permissions**: Who can use this feature (all users, logged-in only, admins)?
2. **Data Scope**: What data is included/excluded (e.g., "Search only published pages, not drafts")?
3. **Priority/Ordering**: If multiple items match, how are they ranked?
4. **Timing**: When does data update (real-time, nightly, manual refresh)?

### **D. Design Reference Check (Figma)**

> **Purpose**: Quick validation that designs exist. Deep analysis happens in **DesignAnalysis** phase.

**Quick Checks Only**:
1. **Extract Figma Links**: List all Figma URLs found in the spec
2. **Verify Accessibility**: Confirm each link is reachable via MCP
3. **Coverage Check**: Note if any spec features lack Figma references
4. **Defer Deep Analysis**: Component extraction, tokens, and responsive review happen in DesignAnalysis

**Output**: Add to Gap Analysis Report:

```markdown
### Design References

| Feature Area | Figma Link | Accessible |
|--------------|------------|------------|
| Search Results | [node-id=123] | ✅ |
| GDS Modal | [node-id=456] | ✅ |
| Mobile View | Not provided | ❌ Flag |

> **Next Phase**: DesignAnalysis will extract tokens, components, and responsive specs.
```

**No-Figma Handling**:
- If spec has NO Figma links: Flag as 🟠 HIGH RISK
- User can choose to skip DesignAnalysis phase with acknowledgment

**Tool Failure Handling**:
- If Figma is unreachable: Ask user for screenshot
- If link is broken: Flag for PM to fix

---

## 3. Gap Analysis Output Format

**Severity Levels** (MANDATORY):

### **🔴 BLOCKER**
Questions that **prevent implementation** because core behavior is undefined.

**Examples**:
- "What should happen when the user clicks the search icon?"
- "Should search include blog posts, or only CMS pages?"
- "When does the search index update: immediately after content publish, or overnight?"

### **🟠 HIGH RISK**
Questions that, if wrong, **will cause rework or bad UX**.

**Examples**:
- "What should users see when search returns zero results?"
- "On mobile, does the search dropdown overlay the menu, or replace it?"
- "Should the search bar be visible on all pages, or only the homepage?"

### **🟢 LOW RISK**
Minor UX polish questions that can be defaulted but should be confirmed.

**Examples**:
- "Should hitting 'Enter' navigate to the first result, or do nothing?"
- "How many results should display before showing 'See More'?"
- "Should the search icon pulse/animate to draw attention?"

---

## 4. Example Question Template

```markdown
**[Feature Area] – [Specific Scenario]**:
- **Current Spec Says**: [Quote from spec]
- **Gap**: [What's unclear]
- **User Impact**: [Why this matters to the user]
- **Suggested Options**:
  - Option A: [Description]
  - Option B: [Description]
- **Recommendation**: [If you have one, based on UX best practices]
```

---

## 5. Output Decision

**After Analysis, you must**:

1. **If NO BLOCKER/HIGH RISK gaps**:
   - State: ✅ "Product Spec is ready for Epic creation."
   - **STOP** and request approval to proceed to FeaturePlanning.

2. **If BLOCKER/HIGH RISK gaps exist**:
   - Present Gap Analysis Report with severity-segmented questions
   - Offer **THREE options**:
     - **Option A**: Post questions to Confluence and wait for PM response
     - **Option B**: User provides answers directly in chat
     - **Option C**: Proceed with provisional assumptions (see § 6 below)
   - **STOP** until user selects an option.

3. **If LOW RISK gaps only**:
   - Document provisional assumptions (e.g., "Assuming 10 results max, can adjust later").
   - **PROCEED** to Epic creation with assumptions noted in "Assumptions Log" (see § 6).

---

## 6. Assumption Logging Protocol

**When to Use**: User chooses to proceed despite unresolved BLOCKER or HIGH RISK gaps (Option C above).

**Purpose**: Ensure all assumptions are explicitly documented for future validation and prevent "silent decisions" that cause rework.

### **Step 1: Create Assumptions Log Section in Epic**

When creating the Epic in FeaturePlanning mode, add this section **immediately after the Description**:

```markdown
---

## 🔔 Assumptions Log

> **Purpose**: Tracks provisional assumptions made due to gaps in the Product Spec. These MUST be validated before production release.

| ID | Gap Description | Provisional Assumption | Validation Plan | Risk | Status |
|---|---|---|---|---|---|
| A1 | [What's unclear in spec] | [What we're assuming] | [How we'll verify] | 🔴/🟠/🟢 | ⏳ Pending |

**Epic Label**: `needs-validation` (auto-applied)
```

### **Step 2: Populate Each Assumption**

For each unresolved gap, add a row:

**Example**:
```markdown
| A1 | Search scope not defined | Assuming search includes only published pages, not drafts | Product Manager to confirm in Sprint Review | 🟠 HIGH | ⏳ Pending |
| A2 | Mobile search behavior unclear | Assuming search dropdown overlays menu (follows industry standard) | QA to validate on mobile device | 🟢 LOW | ⏳ Pending |
```

### **Step 3: Label the Epic**

- Apply Jira label: `needs-validation`
- This flags the Epic for PM review before release

### **Step 4: Link Assumptions in Tech Spec**

When creating the Tech Spec (TechSpec mode), reference assumptions explicitly:

```markdown
## 3. Technical Assumptions

> **Cross-Reference**: See Epic [${JIRA_PROJECT_KEY}-XXX] Assumptions Log for product-level assumptions.

**Technical Decisions Based on Assumptions**:
- **Assumption A1** (Search scope) → Implementing filter: `status = 'published'` in search query
- **Assumption A2** (Mobile behavior) → Using `position: absolute` for search dropdown
```

### **Step 5: User Approval Required**

Before proceeding to FeaturePlanning, present the Assumptions Log draft and request explicit approval:

```markdown
⚠️ **PROCEEDING WITH ASSUMPTIONS**

I've identified [X] unresolved gaps. Here's the proposed Assumptions Log:

[Display table]

**Next Steps**:
1. I'll create the Epic with this Assumptions Log included
2. Epic will be labeled `needs-validation`
3. Product Manager should review before Sprint Planning

> **⏸️ APPROVAL REQUIRED**: Reply `Approve with assumptions` to proceed, or `Post questions to Confluence` to get PM clarification first.
```

---

## 7. Gate Rule

**You CANNOT proceed to FeaturePlanning (Epic creation) until**:
- All **🔴 BLOCKER** gaps are resolved by the Product Manager, **OR**
- User explicitly approves proceeding with assumptions (and Assumptions Log is prepared)
- All **🟠 HIGH RISK** gaps are either resolved **OR** documented in Assumptions Log with user approval

---

## ⛔ HARD STOP: Gap Analysis Approval Gate

> [!CAUTION]
> **This is a BLOCKING gate. You MUST NOT proceed without explicit user approval.**

**After completing Gap Analysis, you MUST:**
1. Present the Gap Analysis Report with severity-segmented questions
2. If no gaps: Present approval request to proceed to FeaturePlanning
3. If gaps exist: Present options (A, B, or C) and WAIT for user selection
4. **STOP and WAIT** for user response
5. Make **NO further tool calls** until approval is received

**Valid Approval Responses:**
- `Approve` or "Spec is ready" → Proceed to FeaturePlanning Mode
- `Option A/B/C` → Handle according to option selected
- `Revise [feedback]` → Update analysis and re-present

**⛔ DO NOT (until approved):**
- Create Epic content
- Generate mockups or designs
- Proceed to FeaturePlanning mode
- Make any further tool calls beyond presenting this gate

**On Approval**: → Immediately proceed to **FeaturePlanning** mode (or **DesignAnalysis** if Figma links exist). Do NOT offer implementation.

---

## 8. Product Increment / Adjustment Handling

> [!IMPORTANT]
> **Post-Approval Changes**: If user introduces changes AFTER ProductSpec is approved,
> use the Change Request Protocol to maintain documentation alignment.

### 8.1 Detecting Post-Approval Changes

After ProductSpec approval, if user says:
- "Actually, also..." / "Add..." / "Change..." / "Instead..."
- Introduces new requirements
- Modifies existing requirements

**Action**: Switch to ChangeRequest mode.

```
1. Load ${AGENT_ROOT}/modes/planning/change-request.md
2. Current Phase: PLANNING_SPEC (or beyond)
3. Execute classification and impact analysis
4. Return here after change is processed
```

### 8.2 Inline Minor Changes

For **Minor Adjustments** (typos, clarifications) during review:

```
1. Update product-spec.md directly via storage.updateProductSpec()
2. No formal change log entry needed
3. Continue with current gate
```

### 8.3 Scope Changes to Approved Spec

For **Scope Changes** after approval:

```
1. Archive current version: storage.archiveArtifact("product-spec", specId)
2. Update product-spec.md with new requirements
3. Log change: storage.logChange("product-spec", specId, {
     type: "increment" or "adjustment",
     source: sourceCode,
     priority: priority,
     description: "Added X requirement",
     impactedAreas: ["Epic", "TechSpec"] // what else needs update
   })
4. If Epic exists: cascade change via storage.cascadeChange()
5. Present updated spec for re-approval if needed
```

### 8.4 Change Log Section Template

ProductSpec files should include (at end):

```markdown
---

## Change Log

| Version | Date | Source | Type | Priority | Description |
|---------|------|--------|------|----------|-------------|
| v1.0 | 2026-01-10 | PM | initial | - | Initial product specification |

```

This section is automatically updated by `storage.logChange()`.
