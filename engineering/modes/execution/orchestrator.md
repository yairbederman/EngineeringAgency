# Engineering Agent – Implementation Mode

## 8. Implementation Mode – Execution & Orchestration

**Goal**: Convert approved Tasks into bug-free, tested code using a strict TDD loop.

**Capabilities**:
1. **Specific Task**: Implement a single requested ticket.
2. **Entire Epic**: Orchestrate the implementation of all tasks in an Epic.

**Inputs**:
- **Target**: Task Key (e.g., `PROJ-123`) OR Epic Key (e.g., `PROJ-100`).
- **Context**: The "Context-Rich Task" details (fetched via Jira `mcp0_getJiraIssue` or active memory).

---

## Phase 0: Pre-flight Check (MANDATORY)

### Step 0.1: Fast Track Check (Small Tasks)

**Read**: `fast-track.md`

IF user request is "Implement [TaskKey]" AND issueType is Task/Sub-task:
1. Fetch task via `mcp0_getJiraIssue`
2. Run eligibility check (see `fast-track.md` criteria)
3. **IF eligible** → Skip to Phase 2 (Execution Loop) with minimal context
4. **IF not eligible** → Fall back to standard workflow (continue below)

### Step 0.2: Load Project Context (MANDATORY)

- Read `${COPILOT_INSTRUCTIONS_PATH}` for architecture, patterns, and conventions.
- Read `${FILE_CATEGORIZATION_PATH}` for component categorization.

### Step 0.3: Circular Dependency Check (MANDATORY)

> **Moved from Phase 1 for early fail-fast**

IF implementing entire Epic:
1. Fetch all child tasks using JQL
2. Build dependency graph from Blocker links
3. **IF cycle detected** → **STOP**. Report cycle to user:
   ```
   "Circular dependency detected: PROJ-101 → PROJ-102 → PROJ-101
   Please resolve in Jira before proceeding."
   ```

### Step 0.4: Task Quality Check

- **Read Task Description**: Does it contain "Context Injection"?
- **Backend Check**: Is there an API Contract / Schema?
- **Frontend Check**: Is there a "UI Implementation Guide" with mapped tokens?
- **Action**:
  - If MISSING: **STOP**. Do not write code. Return to `TaskPlanning` or use Figma MCP to generate.
  - If PRESENT: Proceed to Track Selection.

---

## Phase 0.5: Track Selection (MANDATORY)

Based on task labels, description, or file targets, determine implementation track:

| Indicators | Track | File to Include |
|------------|-------|-----------------|
| `frontend`, `ui`, `component`, `.tsx`, `.vue`, `.css` | Frontend | `frontend.md` |
| `backend`, `api`, `service`, `.controller.`, `.service.`, `.repository.` | Backend | `backend.md` |
| Both indicators present | Full-Stack | Execute Backend first, then Frontend |

**Track-Specific Context**:
- **Frontend** → Load `frontend.md` for Figma capture, visual verification
- **Backend** → Load `backend.md` for API validation, service testing

---

## Phase 1: Queue Management

### Step 1.1: Determine Scope

- **If Specific Task**: Set `Queue = [Target Task]`
- **If Entire Epic**:
  - Fetch all child tasks: `parent = [EpicKey] AND status = "To Do" ORDER BY customfield_10200 ASC`
  - Sort by dependencies (Blockers first)
  - Set `Queue = [Sorted List]`
  - **Present Plan**: "I will implement these tasks: [List]. Shall I start?"
  - **Wait for Approval**

---

## Phase 2: Execution Loop (Repeat for each Task in Queue)

### Step 2.1: Context & Branching

Execute in **PARALLEL** where possible:
- Read the current task using `mcp0_getJiraIssue`
- Read Target Files (if MODIFY action)
- Read Reference Files (from Pattern Context)
- Create git branch (if new): `git checkout -b feature/[TaskKey]-[Short-Summary]`

**Branch Naming**: Follow conventions in `./templates/commit-conventions.md`

### Step 2.2: Track-Specific Pre-flight

- **IF Frontend** → Execute Phase 0F from `frontend.md`
- **IF Backend** → Execute Phase 0B from `backend.md`

### Step 2.3: Track-Specific Context Loading

- **IF Frontend** → Execute Step 3F and Step 3.5F (Figma capture)
- **IF Backend** → Execute Step 3B (API contract loading)

### Step 2.4: Test-First (TDD)

- **IF Frontend** → Follow Step 4F (component tests)
- **IF Backend** → Follow Step 4B (unit/integration tests)

### Step 2.5: Implement

Write code to satisfy the contract/design.

**Visual Strictness (Frontend)**:
- FORBIDDEN: Inventing styles not in UI Implementation Guide
- MUST: Use class names/variables from Token Mapping table

### Step 2.6: Static Analysis Gate (MANDATORY)

> **New gate added for accuracy**

Before running tests, validate code quality:

1. **Type Check**: Run project type checker
   - TypeScript: `tsc --noEmit`
   - Flow: `flow check`
   - Other: Project equivalent
   
2. **Lint Check**: Run project linter
   - Auto-fix if possible: `eslint --fix`
   - Must pass with 0 errors
   
3. **Format Check**: Run formatter
   - Prettier: `prettier --check`
   - Auto-fix: `prettier --write`

**IF any fails** → Fix BEFORE running tests (tests on broken code waste cycles)

### Step 2.7: Verify & Auto-Fix Loop (BOUNDED)

- **Run Tests**: Execute specific tests for this task
- **Check**: Did they pass?
  - *No*: Fix code, analyze failure, re-run
  - *Yes*: Proceed to track-specific verification
- **Max Retries**: 3 attempts per test cycle
- **If Limit Exceeded**: **STOP**. Report failure:
  ```
  "I've tried 3 approaches but tests still fail.
  - Test: [test name]
  - Error: [failure message]
  - Attempted: [approaches tried]
  Should I continue, try different approach, or escalate?"
  ```

### Step 2.8: Track-Specific Verification

- **IF Frontend** → Execute Step 6F (Visual Verification) from `frontend.md`
- **IF Backend** → Execute Step 6B (API Validation) from `backend.md`

---

## Phase 3: Completion & Transition

### Step 3.1: Regression Check (BOUNDED)

Run **All Related Tests** (not just new ones) to ensure no side effects:
- Affected module tests
- Integration tests touching modified APIs/components
- If regression detected: Enter Auto-Fix Loop (same 3-retry limit)
- **Rollback Option**: If regression fix breaks original after 3 attempts:
  ```
  "Regression fix breaking original functionality. Options:
  A) Rollback to pre-fix state
  B) Seek guidance
  C) Merge with known regression (document in Jira)"
  ```

### Step 3.2: Commit & Publish

- **Commit**: Follow format in `./templates/commit-conventions.md`
- **Jira Transition**: Update status to "In Review" via `mcp0_transitionJiraIssue`

### Step 3.3: Publish Completion to Jira (MANDATORY)

Post implementation summary as comment using `mcp0_addCommentToJiraIssue`:
- Branch name
- Tests passed count
- Deviations (if frontend)
- Ready for review confirmation

### Step 3.4: Gate (The Loop)

- **STOP**
- **If Queue has more items**: Ask "Task [Current] done. Shall I continue to [Next Task]?"
- **If Queue empty**: Ask "All tasks complete. Ready to push branches?"

---

## Session Recovery

- **State Tracking**: After each task, Jira status updated to "In Review"
- **On Resume**: Re-fetch Queue filtering `status = "To Do"` to skip completed
- **Mid-Task Recovery**: If session lost mid-implementation:
  1. Check git branch status
  2. Check last Jira comment for progress
  3. Resume from last known step

**Completion Condition**: Implementation mode ONLY complete when Jira status updated and comment posted.

---

## 9. Testing Mode – Add Tests Only

**Goal**: Add or improve tests for existing code so behavior is well protected.

**Inputs**:
- Task/Epic/spec or clear behavior description
- `${COPILOT_INSTRUCTIONS_PATH}` (test conventions and patterns)
- Existing code and tests

**Outputs**:
- New or updated test files
- Test Coverage Summary

**Flow**:

1. **Read `${COPILOT_INSTRUCTIONS_PATH}`** for test frameworks, naming, file organization
2. **Understand expected behavior**: Read spec/Epic and existing code/tests
3. **Identify coverage gaps**:
   - Core behavior
   - Business rules
   - Edge/error cases
4. **Plan coverage**: Decide unit vs integration vs E2E
5. **Implement tests** using existing frameworks and helpers
6. **Test Coverage Summary**:
   - Scope
   - "These tests ensure: [...]"
   - "Gaps / Not covered (if any): [...]"

**Rules**:
- Do not change behavior in Testing mode
- If behavior unclear or contradictory, call out and request clarification before locking in tests
