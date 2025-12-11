# Engineering Agent – Implementation Modes

## 8. Implementation Mode – Execution & Orchestration

**Goal**: Convert approved Tasks into bug-free, tested code using a strict TDD loop.
**Capabilities**:
1. **Specific Task**: Implement a single requested ticket.
2. **Entire Epic**: Orchestrate the implementation of all tasks in an Epic.

**Inputs**:
- **Target**: Task Key (e.g., `PROJ-123`) OR Epic Key (e.g., `PROJ-100`).
- **Context**: The "Context-Rich Task" details (fetched via Jira `mcp0_getJiraIssue` or active memory).

**Flow**:

### Phase 0: Pre-flight Check (MANDATORY)
0. **Load Project Context** (MANDATORY):
   - Read `${COPILOT_INSTRUCTIONS_PATH}` for architecture, patterns, and conventions.
   - Read `${FILE_CATEGORIZATION_PATH}` for component categorization.

1. **Task Quality Check**:
   - **Read Task Description**: Does it contain "Context Injection"?
   - **Backend Check**: Is there an API Contract / Schema?
   - **Frontend Check**: Is there a "UI Implementation Guide" with mapped tokens?
   - **Action**:
     - If MISSING: **STOP**. Do not write code. Go back to `TaskPlanning` or use Figma MCP (`mcp1_get_design_context`) to generate it.
     - If PRESENT: Proceed to Phase 1.

### Phase 1: Queue Management
2. **Determine Scope**:
   - **If Specific Task**: Set `Queue = [Target Task]`.
   - **If Entire Epic**:
     - Fetch all child tasks linked to the Epic using `mcp0_searchJiraIssuesUsingJql`.
     - **JQL**: `parent = [EpicKey] AND status = "To Do" ORDER BY customfield_10200 ASC`
     - **Circular Dependency Check** (MANDATORY):
       - Before setting Queue, validate no cycles exist in task dependencies.
       - If cycle detected: **STOP**. Report cycle to user and request manual resolution.
     - Filter for status "To Do".
     - Sort by dependencies (Blockers first).
     - Set `Queue = [Sorted List]`.
     - **Present Plan**: "I will implement these tasks: [List]. Shall I start?"
     - **Wait for Approval**.

### Phase 2: Execution Loop (Repeat for each Task in Queue)
3. **Context & Branching**:
    - Read the **current task** using `mcp0_getJiraIssue`.
    - **Read Current Codebase (PRIORITY)**:
      - Read `Target Files` (if MODIFY action).
      - Read `Reference Files` (from Pattern Context).
    - **Strict Check (Backend)**: Does task have the API Contract?
    - **Strict Check (Frontend)**: Does task have the "UI Implementation Guide"?
        - *If No*: Stop. Go back to `TaskPlanning` or use Figma MCP (`mcp1_get_design_context`) *now* to generate it.
        - *If Yes*: Proceed.
    - **Git Action**: 
        - **Branch Name**: `feature/[TaskKey]-[Short-Summary]` (e.g., `feature/PROJ-123-user-login`).
        - **Command**: `git checkout -b [BranchName]` (if new) or `git checkout [BranchName]`.

3.5 **Capture Figma Reference** (Frontend Only - MANDATORY):
    > **Purpose**: Save the design reference BEFORE implementation for accurate visual comparison.
    
    - **Step A: Extract Figma Node ID**
      - Parse from UI Implementation Guide's "Figma Reference" field
      - Example: `https://figma.com/design/abc123/File?node-id=1-234` → nodeId: `1-234`
    
    - **Step B: Capture Design Screenshot**
      ```
      mcp1_get_screenshot(nodeId: "[node-id]")
      ```
      - Save to artifacts: `[TaskKey]_figma_reference.png`
    
    - **Step C: Extract Design Dimensions**
      - Note the bounding box from `mcp1_get_design_context`:
        - Width: `[X]px`
        - Height: `[Y]px`
      - This sets the expected viewport for browser comparison
    
    - **Fallback**: If `mcp1_get_screenshot` fails, note in walkthrough:
      ```markdown
      > ⚠️ Figma screenshot unavailable. Visual comparison based on Token Mapping table only.
      ```
    
4. **Test-First (TDD)**:
    - Create/Update test files.
5. **Implement**:
    - Write code to satisfy the contract.
    - **Visual Strictness**:
        - You are FORBIDDEN from inventing styles.
        - You MUST use the class names/variables listed in the Task's "UI Implementation Guide".
        - *Example*: If Task says `gap-4`, do not write `margin-top: 16px`.
6. **Verify & Auto-Fix Loop** (BOUNDED):
    - **Run Tests**: Execute the specific tests for this task.
    - **Check**: Did they pass?
        - *No*: **Fix Code**. Analyze failure, modify implementation, re-run tests.
        - *Yes*: Proceed.
    - **Max Retries**: 3 attempts per test cycle.
    - **If Limit Exceeded**: **STOP**. Report failure to user with:
        - Test failure details
        - Attempted fixes summary
        - Request: *"I've tried 3 approaches but tests still fail. Should I continue, try a different approach, or escalate?"*
    - (Frontend Only) **Visual Verification** (MANDATORY):
    
      #### Step 6A: Launch Component in Browser
      
      Use `browser_subagent` to:
      1. Start dev server if not running: `npm run dev`
      2. Navigate to component URL (Storybook, dev page, or isolated route)
      3. Wait for component to fully render
      
      #### Step 6B: Capture Implementation Screenshots
      
      **For walkthrough documentation:**
      - Use `browser_subagent` with recording to capture the implementation
      - Save recordings to artifacts directory
      - Name format: `[TaskKey]_implementation.png`
      
      **Screenshot checklist:**
      - [ ] Default state → `[TaskKey]_impl_default.png`
      - [ ] Hover state (if applicable) → `[TaskKey]_impl_hover.png`
      - [ ] Focused state (if applicable)
      - [ ] Disabled state (if applicable)
      - [ ] Mobile viewport (if responsive)
      
      #### Step 6C: Side-by-Side Comparison (MANDATORY for Frontend)
      
      **Compare Figma reference (from Step 3.5) with browser implementation:**
      
      1. **Load both images:**
         - Figma: `[TaskKey]_figma_reference.png` (captured in Step 3.5)
         - Browser: `[TaskKey]_impl_default.png` (captured above)
      
      2. **Generate comparison carousel for walkthrough:**
         ```markdown
         ````carousel
         ![Figma Design](/path/to/[TaskKey]_figma_reference.png)
         <!-- slide -->
         ![Implementation](/path/to/[TaskKey]_impl_default.png)
         ````
         ```
      
      3. **Visual Comparison Checklist:**
      
      | Check | Figma Reference | Implementation | Tolerance | Status |
      |-------|-----------------|----------------|-----------|--------|
      | Overall layout | Screenshot | Browser | Visual match | ✅/❌ |
      | Component tree structure | Figma tree vs DOM | - | Exact nesting | ✅/❌ |
      | Spacing (gap, padding) | Layout Properties table | Computed styles | ±2px | ✅/❌ |
      | Colors (bg, text, border) | Token Mapping table | Applied classes | Exact match | ✅/❌ |
      | Typography | Token Mapping table | Applied classes | Exact match | ✅/❌ |
      | Component reuse | Component Instances list | Imports used | All reused | ✅/❌ |
      | Interactive states | Interactive States table | Browser behavior | Visual match | ✅/❌ |
      
      > **Key Insight**: The Figma screenshot provides the definitive visual target. 
      > Token tables validate semantics; screenshots validate appearance.
      
      #### Step 6D: Deviation Handling
      
      **If implementation differs from Figma:**
      
      1. **Document in comparison table:**
         ```markdown
         | Colors | bg-brand-500 expected | bg-blue-500 used | ⚠️ Token missing |
         ```
      
      2. **Add deviation note to commit message:**
         ```
         [W0-123] Implement SearchWidget
         
         Deviations from Figma:
         - Used bg-blue-500 (closest match for #1E40AF not in design-tokens)
         - Spacing 13px rounded to gap-3 (12px)
         ```
      
      3. **Flag for design review if blocking:**
         - Add label `needs-design-review` to Jira task
         - Comment with specific deviation details
      
      #### Step 6E: Record for Walkthrough (Optional)
      
      For significant UI implementations, create demo recording:
      
      ```
      browser_subagent:
        Task: "Navigate to [URL], demonstrate [user flow], capture recording"
        RecordingName: "[taskkey]_user_flow"
      ```
      
      Recording should demonstrate:
      - Initial component render
      - Interactive behaviors (hover, click, input)
      - Loading/success/error states if applicable

### Phase 3: Completion & Transition
7. **Regression Check** (BOUNDED):
   - Before finishing, run **All Related Tests** (not just the new one) to ensure no side effects.
   - If regression detected: Enter **Auto-Fix Loop** (same 3-retry limit).
   - **Rollback Option**: If regression fix breaks original implementation after 3 attempts, offer:
     - *"Regression fix is breaking original functionality. Should I rollback to pre-fix state and seek guidance?"*
8. **Commit & Publish**:
   - **Git Action**: Commit changes with message `[TaskKey] Implement feature`.
   - **MCP Action**: Update Jira Status to "In Review" using `mcp0_transitionJiraIssue`.
9. **Publish Completion to Jira** (MANDATORY):
   - **Action**: Post implementation summary as a comment using `mcp0_addCommentToJiraIssue`.
   - **Content**: Branch name, tests passed, any notes.
10. **Gate (The Loop)**:
   - **STOP**.
   - **If Queue has more items**: Ask *"Task [Current] done. Tests passed (including regressions). Shall I switch context to [Next Task]?"*
   - **If Queue empty**: Ask *"All tasks complete. Ready to push branches?"*

### Session Recovery
- **State Tracking**: After each task completion, Jira status is updated to "In Review".
- **On Resume**: Re-fetch Queue filtering for `status = "To Do"` to automatically skip completed tasks.
- **Mid-Task Recovery**: If session is lost mid-implementation, on resume:
  1. Check git branch status
  2. Check last Jira comment for progress
  3. Resume from last known step

**Completion Condition**:
- Implementation mode is ONLY complete when Jira status is updated and comment is posted.

---

## 9. Testing Mode – Add Tests Only

Goal:
- Add or improve tests for existing code so behavior is well protected.

Inputs:
- Task/Epic/spec or clear behavior description (fetch using `mcp0_getJiraIssue` or `mcp0_getConfluencePage`)
- `${COPILOT_INSTRUCTIONS_PATH}` (test conventions and patterns)
- Existing code and tests

Outputs:
- New or updated test files
- Test Coverage Summary

Flow:
1. **Read `${COPILOT_INSTRUCTIONS_PATH}`** to understand test frameworks, naming conventions, and file organization
2. **Understand expected behavior**:
   - Read spec/Epic and existing code/tests.
3. **Identify coverage gaps**:
   - Core behavior
   - Business rules
   - Edge/error cases
4. **Plan coverage**:
   - Decide which tests are unit, integration, E2E.
5. **Implement tests** using existing frameworks and helpers.
6. **Test Coverage Summary**:
   - Scope
   - "These tests ensure: [...]"
   - "Gaps / Not covered (if any): [...]"

Rules:
- Do not change behavior in Testing mode.
- If behavior is unclear or contradictory, call it out and recommend clarification before locking in tests.
