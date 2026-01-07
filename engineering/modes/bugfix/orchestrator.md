# Engineering Agent – BugFix Mode – Orchestrator

> **Entry Point**: This is the main BugFix workflow. It routes to track-specific files based on bug type.

## Overview

This orchestrator handles both **BugReport Mode** (analysis) and **BugFix Mode** (implementation), routing to the appropriate track (Frontend, Backend, or Full-Stack) based on bug symptoms.

---

## Phase 0: Pre-flight & Track Selection (MANDATORY)

### Step 0.1: Fetch Bug Context (Source of Truth)

1. **Fetch Jira Issue**: Use `${MCP_ATLASSIAN_GET_ISSUE}` to get full bug details
2. **CRITICAL WARNING**: Do NOT rely on user summary. Jira issue is the ONLY source of truth.
3. **Validate Understanding**:
   - Is the bug title and description clear?
   - Can you determine current vs expected behavior?
   - Are reproduction steps provided?
   - **IF NO** → STOP and request clarification from user

### Step 0.2: Track Selection (MANDATORY)

Based on bug symptoms, affected files, and error evidence, determine the investigation track:

| Symptom Indicators | Track | Context to Load | Context to Skip |
|-------------------|-------|-----------------|-----------------|
| UI/Visual regression, CSS errors, Component render issues, `.tsx`/`.vue` files | **Frontend** | `frontend.md` | Service topology, cross-service APIs |
| API errors (4xx, 5xx), Database issues, Service failures, `.controller.`/`.service.` files | **Backend** | `backend.md` | Figma, design tokens, visual verification |
| Data mismatch client↔server, E2E failures, Network trace shows API + UI issue | **Full-Stack** | Both tracks | None (apply Backend first, then Frontend) |

**Track Detection Logic**:
```
IF bug mentions: UI, visual, component, CSS, render, display, layout
   OR affected files contain: .tsx, .vue, .css, .scss, hooks/, components/
   → SET Track = "Frontend"

ELSE IF bug mentions: API, database, service, 500, 404, timeout, migration
   OR affected files contain: .controller., .service., .repository., migrations/
   → SET Track = "Backend"

ELSE IF bug involves: data mismatch, network trace shows both layers, E2E
   → SET Track = "Full-Stack" (Backend first, then Frontend)

ELSE
   → ASK user: "Cannot determine bug track. Is this Frontend, Backend, or Full-Stack?"
```

### Step 0.3: Load Context (TRACK-SELECTIVE, CACHED)

> **Optimization**: Context is cached per Bug ID and only track-relevant sections are loaded.

- **IF `_BUGFIX_CONTEXT_LOADED == [BugKey]`** → Skip context reload
- **ELSE**:
  - Read `${COPILOT_INSTRUCTIONS_PATH}` → **Only [Track] section**
  - Query `${FILE_CATEGORIZATION_PATH}` for affected layers
  - **Frontend Track**: Load `./frontend.md`
  - **Backend Track**: Load `./backend.md`
  - **Full-Stack Track**: Load both (process Backend first)
  - Set `_BUGFIX_CONTEXT_LOADED = [BugKey]`

---

## 10. BugReport Mode – Bug To Analysis Report

**Goal**: Turn a bug description into a clear Bug Analysis Report that can drive a safe BugFix.

**Inputs**:
- Bug title and description (from Jira)
- Steps to reproduce
- Logs/stack traces/screenshots/network traces
- Optional: Related Jira tickets/specs/commits

**Outputs**:
- Bug Analysis Report (posted to Jira)

### Step 1: Tech Stack Identification (MANDATORY)

- Detect frameworks from `package.json`, `pom.xml`, `go.mod`, etc.
- Identify test framework (Jest, Vitest, Mocha, JUnit, etc.)
- Note build tools (Webpack, Turbopack, Maven, etc.)
- **Why**: Framework-specific bugs require framework-specific fixes.

### Step 2: Understand Reported Symptoms

- Scope, impact, severity
- User-facing vs. internal impact
- **Track-specific questions**:
  - **Frontend**: What visual/interactive behavior is wrong?
  - **Backend**: What API response/data is incorrect?
  - **Full-Stack**: Where is the data corrupted (source vs consumer)?

### Step 3: Codebase Discovery

**Direct Code Inspection**:
1. Use `view_file` to read affected file(s) identified from bug report
2. Use `view_code_item` to inspect specific functions/classes
3. Use `grep_search` to find related patterns and similar code

**Frontend Track**:
- Inspect component implementation and props/state
- Check design tokens/styles applied to affected elements
- Search for similar bugs or patterns in test files

**Backend Track**:
- Inspect service/controller implementation
- Check error handling patterns for the error type
- Search for similar bugs or patterns in test files

**Full-Stack Track**:
- Trace data flow from service to component
- Inspect both API and UI layers

### Step 4: Inspect Evidence

- Logs, traces, screenshots, network traces
- Map evidence to code findings from Step 3
- Identify exact failure points in the code
- **Track-specific analysis**:
  - **Frontend**: Load visual evidence screenshots
  - **Backend**: Parse API response/error logs
  - **Full-Stack**: Correlate network trace with both layers

### Step 5: Layer & Dependency Analysis

**Execute track-specific file**: 
- **Frontend** → `./frontend.md` Step 1F (Frontend Layer Analysis)
- **Backend** → `./backend.md` Step 1B (Backend Layer Analysis)
- **Full-Stack** → Execute both in sequence

### Step 6: Integration Rule Check

- Validate that suspected fix won't violate integration rules from `${COPILOT_INSTRUCTIONS_PATH}`
- Check architectural constraints and patterns
- Identify any cross-project or cross-layer impacts

### Step 7: Formulate Fix Strategy

Use template: `./templates/bug-analysis-report.md`

- Bug summary
- Impact and scope
- Reproduction understanding
- Suspected root cause
- Related evidence
- Suggested fix plan (track-specific)
- Verification approach
- Assumptions / Missing info

### Step 8: Publish Analysis to Jira (MANDATORY)

1. **Present Report**: Show Bug Analysis Report to user for review
2. **Request Approval**: "Do you approve this analysis for posting to Jira?"
3. **On Approval**: Post using `${MCP_ATLASSIAN_ADD_COMMENT}`
4. **Confirm Success**: Then ask: "Ready to proceed to BugFix?"

**Completion Condition**: BugReport mode is ONLY complete when analysis posted to Jira AND confirmed.

---

## 11. BugFix Mode – Implement Bug Fixes

**Goal**: Take a Bug Analysis Report and produce a safe, tested fix on a dedicated branch.

**Inputs**:
- Latest Bug Analysis Report
- Original bug context (Jira)
- Track selection from Phase 0

**Outputs**:
- Code changes on `bugfix/` branch
- Tests reproducing the bug
- BugFix Summary (posted to Jira)

### Step 1: Re-validate (CACHED)

> **Optimization**: Uses cached context from BugReport mode if same session.

- **IF `_BUGFIX_CONTEXT_LOADED == [BugKey]`** → Skip full reload
- **ELSE**: Fetch bug using `${MCP_ATLASSIAN_GET_ISSUE}` and reload context
- Review Bug Analysis Report
- Confirm track selection still valid

### Step 2: Git Action

- Construct branch name: `bugfix/[BugKey]-[Short-Summary]`
- Example: `bugfix/PROJ-99-fix-crash`
- Create/checkout branch

### Step 3: Inspect Implementation (Track-Specific)

**Execute track-specific file**: 
- **Frontend** → `./frontend.md` Step 2F (Frontend Code Inspection)
- **Backend** → `./backend.md` Step 2B (Backend Code Inspection)
- **Full-Stack** → Execute Backend first, then Frontend

### Step 4: Define Fix Plan

- Brief summary of changes
- Files to be modified
- Compliance with integration rules from `${COPILOT_INSTRUCTIONS_PATH}`
- Alternative approaches if applicable

### Step 5: Root Cause Validation (MANDATORY – STRUCTURED)

Use checklist: `./templates/root-cause-checklist.md`

- [ ] Can I reproduce the bug with a failing test? (Y/N)
- [ ] Does my root cause explain symptom #1? (Y/N)
- [ ] Does my root cause explain symptom #2? (Y/N for each)
- [ ] Have I ruled out at least 2 alternative causes?
- [ ] Am I confident this is THE root cause, not A symptom? (Y/N)

**IF any N** → Return to Step 3 (Inspect). Do NOT proceed with uncertain fixes.

### Step 6: Test Failure (Reproduction)

- Create test case that reproduces the bug
- **Run Test**: Confirm it FAILS (proves bug exists)
- If test doesn't fail: Re-evaluate root cause (Step 5)

### Step 7: Static Analysis Gate (MANDATORY)

> **Applies BEFORE running tests** – Catch type/lint errors early.

1. **Type Check**: 
   - TypeScript: `tsc --noEmit`
   - Flow: `flow check`
   - Other: Project equivalent

2. **Lint Check**: 
   - `eslint --fix` or project linter
   - Must pass with 0 errors

3. **Format Check**: 
   - `prettier --write` or project formatter

**IF any fails** → Fix code BEFORE running tests (tests on broken code waste cycles)

### Step 8: Implement Fix (BOUNDED)

- Apply minimal changes to pass the test
- Follow project patterns from `${COPILOT_INSTRUCTIONS_PATH}`
- **Run Test**: Confirm it PASSES
- **Max Retries**: 3 attempts per fix approach
- **If Limit Exceeded**: STOP. Report:
  ```
  "Fix approach not working after 3 attempts.
  - Test: [test name]
  - Error: [failure message]
  - Attempted: [approaches tried]
  Should I try alternative strategy or escalate?"
  ```

### Step 9: Track-Specific Verification

**Execute track-specific file**: 
- **Frontend** → `./frontend.md` Step 3F (Visual Verification)
- **Backend** → `./backend.md` Step 3B (API Verification)
- **Full-Stack** → Execute both

### Step 10: Regression Check (MANDATORY, BOUNDED)

- Run all tests in affected file(s)
- Run related integration tests
- If regression detected: Enter bounded auto-fix loop (3 attempts)
- **Rollback Option**: If regression persists after 3 attempts:
  ```
  "Regression fix breaking original functionality. Options:
  A) Rollback to pre-fix state
  B) Seek guidance
  C) Merge with known regression (document in Jira)"
  ```

### Step 11: Verify Fix Compliance

- Does the fix follow project patterns from `${COPILOT_INSTRUCTIONS_PATH}`?
- Does it violate any integration rules?
- Are all affected tests passing?
- Are there any cross-layer or cross-project impacts?

### Step 12: Bug Pattern Documentation (RECOMMENDED)

Use template: `./templates/bug-pattern-record.md`

Document to prevent recurrence:
- Pattern: [e.g., "Null check missing in async handler"]
- Root Cause Category: [e.g., "Error Boundary Gap"]
- Prevention: [e.g., "Add required field validation in service layer"]
- Similar Code Locations: [List files that may have same pattern]

### Step 13: Commit & Summary

- Commit with message: `[BugKey] Fix [short description]`
- Report: "Fixed on branch `[BranchName]`. Tests passed."
- Document any architectural considerations or follow-up items

### Step 14: Publish Fix to Jira (MANDATORY)

1. **Present Summary**: Show BugFix Summary to user for review
2. **Request Approval**: "Do you approve this fix summary for posting to Jira?"
3. **On Approval**: Post using `${MCP_ATLASSIAN_ADD_COMMENT}`
4. **Present Completion Options**:

```
✅ **BugFix Complete**
- **Bug**: [BugKey] - [Summary]
- **Branch**: bugfix/[BugKey]-[summary]
- **Tests**: Regression tests passing
- **Jira**: Status updated, comment posted

> **⏸️ NEXT STEP**: Reply with:
> - `Create PR` to generate PR description (recommended)
> - `Push` to push branch only
> - `Done` to end workflow
```

**Content**: Include Summary, Fix Strategy, Tests, Verification Results

**Completion Condition**: BugFix mode is complete when:
1. Fix summary posted to Jira
2. User chooses next action (Create PR / Push / Done)

---

## Full-Stack Coordination Protocol

> **When**: Track = "Full-Stack"

### Backend-to-Frontend Handoff (MANDATORY)

After completing Backend fix, before starting Frontend verification:

1. **Contract Integrity Check**:
   - Compare API response before/after fix
   - Check if response structure changed
   - **IF BREAKING change** → Update Frontend expectations

2. **Frontend Context Refresh**:
   - Re-verify Frontend still works with fixed Backend
   - If Frontend now broken by Backend fix → Enter Frontend fix loop

3. **Cross-Layer Verification**:
   - E2E test covering full flow
   - Verify fix doesn't introduce new symptoms in other layer

---

## Rules

- Do not rewrite QA's original description
- Be explicit about uncertainty and alternatives
- Do not silently change behavior in unrelated flows
- If expected behavior is unclear or has product/UX impact, escalate back to BugReport/spec clarification

---

## Quick Reference

### Track-Specific Files

| Track | File | When to Use |
|-------|------|-------------|
| **Frontend** | [frontend.md](./frontend.md) | UI/visual bugs, CSS issues, component render problems |
| **Backend** | [backend.md](./backend.md) | API errors (4xx, 5xx), database issues, service failures |
| **Full-Stack** | Both files in sequence | Data mismatch client↔server, E2E failures |

### Templates

| Template | Purpose |
|----------|---------|
| [bug-analysis-report.md](./templates/bug-analysis-report.md) | Structured output for BugReport mode |
| [root-cause-checklist.md](./templates/root-cause-checklist.md) | Validation before implementing fix |
| [bug-pattern-record.md](./templates/bug-pattern-record.md) | Document patterns to prevent recurrence |

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    BugFix Orchestrator                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Fetch Bug (Jira)                                            │
│  2. Track Selection ──────┬──────────────┬──────────────────────│
│                           │              │                      │
│                           ▼              ▼                      │
│              ┌─────────────────┐  ┌─────────────────┐           │
│              │   Frontend      │  │    Backend      │           │
│              │   Track         │  │    Track        │           │
│              │   frontend.md   │  │   backend.md    │           │
│              └────────┬────────┘  └────────┬────────┘           │
│                       │                    │                    │
│                       ▼                    ▼                    │
│  3. BugReport Mode (Analysis)                                   │
│     └── Output: Bug Analysis Report                             │
│                                                                 │
│  4. BugFix Mode (Implementation)                                │
│     ├── Root Cause Validation (Checklist)                       │
│     ├── Static Analysis Gate                                    │
│     ├── TDD Fix Loop (Bounded)                                  │
│     ├── Track-Specific Verification                             │
│     └── Bug Pattern Documentation                               │
│                                                                 │
│  5. Publish to Jira                                             │
└─────────────────────────────────────────────────────────────────┘
```
