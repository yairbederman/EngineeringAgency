# Engineering Agent – BugFix Modes

## 10. BugReport Mode – Bug To Analysis Report

Goal:
- Turn a bug description into a clear Bug Analysis Report that can drive a safe BugFix.

Inputs:
- Bug title and description
- Steps to reproduce (if provided)
- Logs/stack traces/screenshots/network traces
- Optional: links to related jira tickets/specs/commits

Output:
- Bug Analysis Report

Report structure:
- Bug summary
- Impact and scope
- Reproduction understanding
- Suspected root cause
- Related evidence
- Suggested fix plan
- Checks after fix
- Assumptions / Missing info

Flow:
0. **Fetch Jira Issue (Source of Truth)** (MANDATORY):
   - Use `mcp0_getJiraIssue` to fetch the full issue details.
   - **CRITICAL WARNING**: Do NOT rely on the user's summary or previous context. The Jira issue is the ONLY source of truth.
   - Verify the bug title, description, and expected behavior match your understanding.
   - If the bug is not clear, you cannot determine current vs expected behaviour, STOP and alert the user.

1. **Load Project Context** (MANDATORY):
   - Read `${COPILOT_INSTRUCTIONS_PATH}` to understand:
     - Architectural boundaries and domain structure
     - Integration rules and constraints
     - Domain-specific patterns and error handling
   - Query `${FILE_CATEGORIZATION_PATH}` to identify affected layer(s)

2. **Tech Stack Identification** (MANDATORY):
   - Detect frameworks from `package.json`, `pom.xml`, `go.mod`, etc.
   - Identify test framework (Jest, Vitest, Mocha, JUnit, etc.)
   - Note build tools (Webpack, Turbopack, Maven, etc.)
   - **Why**: Framework-specific bugs require framework-specific fixes.

3. **Understand Reported Symptoms**:
   - Scope, impact, severity
   - User-facing vs. internal impact

4. **Codebase Discovery (Context7)**:
   Query Context7 with specific questions:
   - "How is [affected entity/feature] implemented?"
   - "Where is [error symptom] handled in the codebase?"
   - "Are there similar bugs or error cases already fixed?"
   - "What are existing patterns for [affected component/layer]?"
   - "What tests cover this functionality?"

5. **Inspect Evidence**:
   - Logs, traces, screenshots, network traces
   - Map evidence to code findings from step 4
   - Identify exact failure points in the code

6. **Layer & Dependency Analysis**:
   - Map bug to specific layer using `${FILE_CATEGORIZATION_PATH}`:
     - Frontend: `react-components`, `hooks`, `redux`, `api-client`
     - Backend: `controllers`, `services`, `repositories`, `migrations`
   - Identify data flow: Where does the bug's data originate? Where is it consumed?
   - Find related files using Context7
   - Understand cross-layer dependencies
   - Assess blast radius of potential fix
   - **Cross-Repository Check** (Multi-Workspace):
     - **Read `${SYSTEM_ARCH_ROOT}/analysis/service-topology.json`** to understand the full service chain
     - Use `callsServices` and `calledBy` to trace the data path
     - If bug involves API calls, check `${SYSTEM_ARCH_ROOT}/analysis/cross-service-apis.json` for the exact endpoint contract
     - Determine if issue is in client (`${PROJECT_FRONTEND}`) or server (`${PROJECT_CMS_API}`, `${PROJECT_DATA_API}`)
     - Use network traces to identify which layer is returning incorrect data
     - Check if upstream/downstream services might be contributing to the issue

7. **Integration Rule Check**:
   - Validate that suspected fix won't violate integration rules from `${COPILOT_INSTRUCTIONS_PATH}`
   - Check architectural constraints and patterns
   - Identify any cross-project or cross-layer impacts

8. **Formulate Fix Strategy** (high level):
   - Proposed approach
   - Affected components and layers
   - Compliance with architectural rules
   - Alternative approaches if applicable

9. **Publish Analysis to Jira** (MANDATORY):
   - **Step 9a**: Present the Bug Analysis Report to the user for review.
   - **Step 9b**: Ask explicitly: "Do you approve this analysis for posting to Jira?"
   - **Step 9c**: On user approval, IMMEDIATELY post using `mcp0_addCommentToJiraIssue`.
   - **Step 9d**: Confirm post success, THEN ask: "Ready to proceed to BugFix?"
   - **Content**: Include the full analysis (Summary, Root Cause, Fix Strategy, Verification).

**Completion Condition**:
- BugReport mode is ONLY complete when the analysis has been posted to Jira AND confirmed.

Rules:
- Do not rewrite QA's original description.
- Be explicit about uncertainty and alternatives.
- No code changes in this mode.

---

## 11. BugFix Mode – Implement Bug Fixes

**Goal**: Take a Bug Analysis Report and produce a safe, tested fix on a dedicated branch.

**Inputs**:
- Latest Bug Analysis Report
- Original bug context (Jira/QA description)
- Relevant code

**Outputs**:
- Code changes on a `bugfix/` branch
- Tests reproducing the bug
- BugFix Summary

**Flow**:
1. **Re-validate**:
   - Fetch bug using `mcp0_getJiraIssue` to confirm understanding
   - Re-read `${COPILOT_INSTRUCTIONS_PATH}` for architectural constraints
   - Review Bug Analysis Report from BugReport mode
   - Identify tech stack and test framework from analysis

2. **Inspect (Context7)**:
   Query Context7 with specific questions:
   - "Show me the implementation of [affected component]"
   - "What tests cover this area?"
   - "How are similar error cases handled in the codebase?"
   - "What are the existing patterns for [affected functionality]?"

3. **Git Action**:
   - Construct branch name: `bugfix/[BugKey]-[Short-Summary]` (e.g., `bugfix/PROJ-99-fix-crash`)
   - **Create/Checkout Branch**

4. **Define Fix Plan**:
   - Brief summary of changes
   - Files to be modified
   - Compliance with integration rules from `${COPILOT_INSTRUCTIONS_PATH}`

5. **Root Cause Validation** (MANDATORY):
   - Can you reproduce the bug with a reproduction test?
   - Does the suspected root cause explain ALL symptoms?
   - **If NO**: Return to step 2 (Inspect) and re-analyze. Do NOT proceed with a fix you're uncertain about.

6. **Test Failure (Reproduction)**:
   - Create a test case that reproduces the bug
   - **Run Test**: Confirm it FAILS (This proves the bug exists)
   - If test doesn't fail: Re-evaluate root cause (step 5)

7. **Implement Fix** (BOUNDED):
   - Apply minimal changes to pass the test
   - Follow project patterns from `${COPILOT_INSTRUCTIONS_PATH}`
   - **Run Test**: Confirm it PASSES
   - **Max Retries**: 3 attempts per fix approach.
   - **If Limit Exceeded**: STOP. Report: *"Fix approach not working after 3 attempts. Should I try alternative strategy or escalate?"*

8. **Regression Check** (MANDATORY):
   - Run all tests in affected file(s)
   - Run related integration tests
   - If regression detected: Enter bounded auto-fix loop (3 attempts)
   - **Rollback Option**: If regression persists after 3 attempts, offer: *"Regression fix breaking original functionality. Should I rollback and seek guidance?"*

9. **Verify Fix Compliance**:
   - Does the fix follow project patterns from `${COPILOT_INSTRUCTIONS_PATH}`?
   - Does it violate any integration rules?
   - Are all affected tests passing?
   - Are there any cross-layer or cross-project impacts?

10. **BugFix Summary**:
    - Commit with message `[BugKey] Fix bug`
    - Report: "Fixed on branch `[BranchName]`. Tests passed."
    - Document any architectural considerations or follow-up items

11. **Publish Fix to Jira** (MANDATORY):
    - **Step 11a**: Present the BugFix Summary to the user for review.
    - **Step 11b**: Ask explicitly: "Do you approve this fix summary for posting to Jira?"
    - **Step 11c**: On user approval, IMMEDIATELY post using `mcp0_addCommentToJiraIssue`.
    - **Step 11d**: Confirm post success. BugFix mode is now complete.
    - **Content**: Include the full fix summary (Summary, Fix Strategy, Verification).

**Completion Condition**:
- BugFix mode is ONLY complete when the fix summary has been posted to Jira AND confirmed.

Rules:
- Do not silently change behavior in unrelated flows.
- If expected behavior is unclear or has product/UX impact, escalate back to BugReport/spec clarification.
