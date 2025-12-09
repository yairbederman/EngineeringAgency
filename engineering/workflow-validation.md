# Lognet Workflow – Quality Validation Checklist

## Purpose

This document defines quality gates for each mode in the Lognet workflow. Use this checklist before completing any mode to ensure artifact quality and completeness.

---

## ProductSpecReview Mode – Quality Gate

### Before Declaring "PROCEED"
- [ ] All "Three C's" analyzed (Completeness, Consistency, Clarity)
- [ ] Happy Path explicitly defined
- [ ] At least one Unhappy Path/Error case identified
- [ ] No subjective terms left unqualified (e.g., "fast", "user-friendly")
- [ ] No contradictions with existing system (verified via Context7 or ${COPILOT_INSTRUCTIONS_PATH})
- [ ] All data inputs are explicit (not vague like "contact info")

### Gap Analysis Report Must Include
- [ ] **Confirmed Scope**: Clear list of what is understood
- [ ] **Critical Gaps**: Specific questions formatted as numbered list
- [ ] **Assumptions**: Explicit "I am assuming X; is this correct?" statements
- [ ] All Critical Gaps are answerable by Product Manager (not technical implementation details)

### Before Declaring "STOP"
- [ ] User has been offered option to post questions to Confluence (if link exists)
- [ ] Clear explanation of why planning cannot proceed without answers

---

## FeaturePlanning Mode – Quality Gate

### Epic Completeness (Before Publishing to Jira)
- [ ] **Goal**: Single sentence describing business value
- [ ] **Context**: Includes trigger, preconditions, and links (Jira/Confluence/Figma)
- [ ] **Key Data Concepts**: Inputs and Outputs listed (business view, not DB columns)
- [ ] **Functional Flows**: At least one Happy Path + one Error/Edge Case
- [ ] **Acceptance Criteria**: Written in Gherkin (Given/When/Then)
- [ ] **Scope & Constraints**: "Must Support" and "Out of Scope" defined

### Epic Quality Rules
- [ ] No technical implementation details (e.g., "Save to User Table")
- [ ] All flows describe user experience, not code behavior
- [ ] Error flows specify what user sees and why
- [ ] No assumptions about database, APIs, or frameworks
- [ ] Figma link included if UI work is involved

### MCP Actions Verified
- [ ] Epic created in Jira (${JIRA_PROJECT_KEY} project) - verify returned Epic key
- [ ] Confluence Product Spec updated with Epic link - verify MCP success
- [ ] If MCP failed, user informed and manual action requested

### Before STOP Gate
- [ ] Epic URL displayed to user
- [ ] Standard Approval Format used
- [ ] Next step (TechSpec Mode) clearly stated

---

## TechSpec Mode – Quality Gate

### Architecture Validation (CRITICAL)
- [ ] **Every** architecture decision references either:
  - Specific section from `${COPILOT_INSTRUCTIONS_PATH}`, OR
  - Existing pattern from Context7 with file path
- [ ] No "generic best practices" without project-specific validation
- [ ] Pattern matching completed: similar existing features identified

### Tech Spec Completeness
- [ ] **Architecture & Patterns**: Compliance explanation present
- [ ] **Data Model**: 
  - [ ] All entities defined as TypeScript interfaces
  - [ ] Migration strategy specified (Yes/No + description)
  - [ ] No invented database fields (use `[TBD]` if unknown)
- [ ] **API Contracts**:
  - [ ] All endpoints listed with METHOD and Path
  - [ ] Request/Response types defined in TypeScript
  - [ ] No invented API paths (use `[TBD]` if unknown)
- [ ] **Implementation Inventory**:
  - [ ] Frontend files listed with categories (from ${FILE_CATEGORIZATION_PATH})
  - [ ] Backend files listed with categories
  - [ ] Each file marked as [NEW] or [MODIFY]
- [ ] **Verification Strategy**:
  - [ ] Unit test approach defined
  - [ ] Integration test cases identified
  - [ ] Critical test scenarios listed

### TBD Validation
- [ ] All `[TBD]` items justified (truly unknown, not just lazy)
- [ ] All TBDs listed in "Assumptions / Missing Details" section
- [ ] No TBDs for core business logic or critical UX flows

### MCP Actions Verified
- [ ] Tech Spec published to Confluence (parent ID: ${TECH_SPECS_FOLDER_ID}) - verify returned URL
- [ ] Confluence Product Spec updated with Tech Spec link - verify MCP success
- [ ] If MCP failed, user informed and manual action requested

### Before STOP Gate
- [ ] Tech Spec URL displayed to user
- [ ] Standard Approval Format used
- [ ] Next step (TaskPlanning Mode) clearly stated

---

## TaskPlanning Mode – Quality Gate

### Task Decomposition Rules
- [ ] Each task is atomic (single testable unit of work)
- [ ] Complex UI work split: "Build Component" vs "Integrate Logic"
- [ ] Dependencies identified and ordered correctly
- [ ] No task exceeds reasonable implementation scope (max 1-2 days)

### Task Quality (For EACH Task)
- [ ] **Type**: Clearly marked as Backend | Frontend | Full-stack
- [ ] **Source**: References Tech Spec section number
- [ ] **Target Files**: 
  - [ ] Files to create listed
  - [ ] Files to modify listed
- [ ] **Implementation Context**:
  - [ ] Backend task: API signature copied from Tech Spec
  - [ ] Backend task (DB): Entity schema copied from Tech Spec
  - [ ] Frontend task: UI Implementation Guide populated
  - [ ] Full-stack task: BOTH API signature AND UI guide included
- [ ] **Steps**: Logical implementation steps listed
- [ ] **Test Plan**:
  - [ ] At least 2 test cases (Happy Path + Edge Case)
  - [ ] Test file path specified

### UI Implementation Guide (Frontend/Full-stack Tasks Only)
- [ ] Figma MCP called to extract design tokens (if Figma link exists)
- [ ] Raw Figma values (hex colors, pixel sizes) mapped to project tokens
- [ ] Structure specified (e.g., "Flex-col layout, centered items")
- [ ] Key Tokens listed:
  - [ ] Background
  - [ ] Spacing
  - [ ] Typography
- [ ] Component Reuse identified (from ${FILE_CATEGORIZATION_PATH})
- [ ] Pixel-perfect implementation requirement stated

### Context Injection Validation
Verify for each task type:

**Backend Task**:
- [ ] API endpoint signature present in "Implementation Context"
- [ ] If DB changes: Entity schema present
- [ ] Validation rules and constraints listed

**Frontend Task**:
- [ ] Figma link checked in Epic
- [ ] If Figma exists: `mcp1_get_design_context` called with correct node ID
- [ ] Figma tokens extracted and mapped to project tokens
- [ ] "UI Implementation Guide" section fully populated

**Full-stack Task**:
- [ ] API signature from Tech Spec included
- [ ] UI Implementation Guide populated
- [ ] Data flow between frontend and backend defined

### MCP Actions Verified
- [ ] All tasks created in Jira (${JIRA_PROJECT_KEY} project) - verify returned keys
- [ ] All tasks linked to Epic (parent field set)
- [ ] All tasks reference Tech Spec Confluence page
- [ ] If any MCP failed, user informed

### Before STOP Gate
- [ ] All task keys listed (${JIRA_PROJECT_KEY}-XXX, ${JIRA_PROJECT_KEY}-YYY, etc.)
- [ ] Task count displayed
- [ ] Standard Approval Format used
- [ ] Next step clearly stated:
  - Implementation Mode (select a specific task), subject to the Implementation/BugFix gate in core-rules.md, or
  - Return to TechSpec or FeaturePlanning if planning gaps remain


---
## Implementation/BugFix – Entry Validation

Before entering Implementation or BugFix:

- [ ] Confirm that all entry conditions in core-rules.md (Implementation/BugFix gate) are satisfied
- [ ] If any required input is missing, do not write or change code
- [ ] Explain to the user what is missing (spec, design, repo context, links, etc.)
- [ ] Propose the appropriate upstream mode:
  - ProductSpecReview, or
  - FeaturePlanning, or
  - TechSpec
- [ ] If entry conditions are satisfied, clearly state:
  - Which Epic / Product Spec you are implementing
  - Which Tech Spec you are following (if present)
  - Which Jira task(s) you are about to work on

---

## General Quality Rules (All Modes)

### Before ANY Mode Completion
- [ ] No invented data:
  - No made-up URLs, IDs, Jira keys
  - No invented schema fields, API paths
  - No invented Figma node IDs, design tokens
- [ ] All MCP tool calls verified for success
- [ ] All failures reported to user with clear next steps
- [ ] Standard Approval Format used consistently
- [ ] Mode Declaration present (first line of response)

### Placeholder Usage
When information is unknown, use explicit placeholders:
- `[TBD – requires input from Product]` for business logic/UX
- `[TBD – requires input from Design]` for visual/interaction design
- `[TBD – requires input from Backend]` for API/schema details

### Link-Back Traceability
- [ ] Product Spec Confluence page has link to Epic
- [ ] Product Spec Confluence page has link to Tech Spec
- [ ] All Tasks reference Epic (Jira parent field)
- [ ] All Tasks reference Tech Spec (Confluence link in description)

---

## Failure Recovery

### If ${COPILOT_INSTRUCTIONS_PATH} Missing
- [ ] STOP immediately
- [ ] Request user to run generate-instructions workflow first
- [ ] Do NOT guess project architecture or patterns

### If Context7 Unavailable
- [ ] Request relevant file paths or snippets from user
- [ ] Mark assumptions explicitly
- [ ] Reduce confidence in pattern matching

### If Figma MCP Fails
- [ ] Request screenshot from user
- [ ] Use `[TBD – Design]` for unknown layout/tokens
- [ ] Do NOT invent UI specifications

### If Atlassian MCP Fails
- [ ] Request pasted Jira/Confluence content from user
- [ ] Do NOT fabricate Epic keys, Tech Spec URLs
- [ ] Inform user that manual linking will be required

---

## Summary

Before completing any mode:
1. Run through the relevant quality gate checklist
2. Verify all required fields are present and complete
3. Confirm all MCP actions succeeded
4. Use Standard Approval Format
5. STOP and wait for user approval

**Never skip these checks to save time. Quality gates prevent rework and ensure LLM-ready outputs.**
