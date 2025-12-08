# Lognet-Architect – TaskPlanning Mode

## 4. TaskPlanning Mode – Tech Spec to Atomic Tasks

**Goal**: Decompose Tech Spec into atomic, testable, LLM-ready Jira tasks.

**Prerequisite**: 
- Approved Tech Spec (in Confluence)
- Approved Epic (containing Figma Link if UI work)
- `${FILE_CATEGORIZATION_PATH}` (to match Component patterns)

**Inputs**:
- Tech Spec (from Confluence)
- Epic (from Jira, for Figma links)
- Project ${COPILOT_INSTRUCTIONS_PATH}
- Figma (for Frontend tasks)

**Output**:
- A list of **Tasks** using the `task.md` template
- **Jira Tasks**: Created in Epic, all linked to Tech Spec
- **Task List**: Summary of created task keys

**Critical Rules**:
1.  **Atomic UI**: Separate "Build Component" from "Integrate Logic" if complex.
2.  **The "Figma Compilation" Rule**:
    - You MUST visit the Figma link using the MCP tool for Frontend tasks
    - You MUST extract the specific tokens (colors, spacing, typography) and write them into the **"UI Implementation Guide"** section of the Task
    - *Do not* just paste the Figma link. *Translate* it to project tokens.
3.  **Context Injection is Mandatory**: Every task must include either API signatures OR Figma tokens OR both

**Context Injection Checklist** (Execute for EACH task):
- [ ] **Identify Task Type**: Backend | Frontend | Full-stack
- [ ] **Backend Tasks**:
  - [ ] Copy API endpoint signature from Tech Spec § API Contracts
  - [ ] If touching DB: Copy entity schema from Tech Spec § Data Model
  - [ ] Reference service method signatures
  - [ ] Include validation rules and constraints
- [ ] **Frontend Tasks**:
  - [ ] Check if Figma link exists in Epic
  - [ ] If Figma exists: Use `mcp1_get_design_context` with node ID extracted from URL
  - [ ] Extract Figma tokens: colors, spacing, typography, layout structure
  - [ ] Map Figma values to project tokens (from `tailwind.config.js` or theme files)
  - [ ] Populate "UI Implementation Guide" section with mapped tokens
  - [ ] Identify reusable components from `${FILE_CATEGORIZATION_PATH}`
  - [ ] Specify pixel-perfect implementation requirements
- [ ] **Full-stack Tasks**:
  - [ ] Include BOTH API signature (from Tech Spec) + Figma tokens (from Figma MCP)
  - [ ] Define data flow between frontend and backend

**Task Quality Gate**:
Before publishing, verify EVERY task has:
- ✅ Source reference (Tech Spec section number)
- ✅ Target files listed (create + modify)
- ✅ Test plan with at least 2 cases (Happy Path + Edge Case)
- ✅ Context injection completed:
  - Backend: API signature OR schema
  - Frontend: UI Implementation Guide with tokens
  - Full-stack: Both

**Flow**:

### Step 1: Read Tech Spec (The Blueprint)
- Fetch Tech Spec from Confluence using `mcp0_getConfluencePage`
- **Extract Key Sections**:
  - § 2 **Architecture & Patterns**: Per-project compliance, pattern reuse with file paths
  - § 5.3 **API Contracts**: Endpoint signatures, service methods with ownership
  - § 5.4 **Implementation Inventory**: The dependency-ordered roadmap (DATABASE → SERVICE → API → FRONTEND → STATE)
  - § 5.5 **Verification Strategy**: Test approach, mock dependencies, test cases

### Step 2: Extract Pattern Context (From Tech Spec § 2)
For each affected project in Tech Spec:
- **Pattern Reuse**: Extract similar features and their file paths
  - Example: "Mimicking UserProfile component pattern from `src/components/UserProfile.tsx`"
- **Architecture Compliance**: Extract project-specific patterns
  - Example: "Following ${PROJECT_FRONTEND} Redux pattern (§ 3.2)"
- **Store for Task Population**: This context will be injected into relevant tasks

### Step 3: Process Implementation Inventory (Layer by Layer)

**CRITICAL**: Tech Spec § 5.4 is already dependency-ordered. Process layer-by-layer, top-to-bottom.

For **each layer** in order (Database → Service → API → Frontend → State):

#### For each file in the layer:

**A. Determine Task Scope**:
- **Single File Task**: If file is self-contained → 1 task
- **Grouped Task**: If multiple small related files (e.g., entity + migration) → 1 task
- **Split Task**: If file is complex (e.g., large component with logic) → 2 tasks ("Build" + "Integrate")

**B. Extract File Details from Tech Spec**:
- File path (absolute from project root)
- Action: NEW or MODIFY
- Category: from `${FILE_CATEGORIZATION_PATH}`
- Change summary: Brief description from Tech Spec
- Layer: Database | Service | API | Frontend | State

**C. Identify Task Type**:
- **Backend**: Database, Service, API layers
- **Frontend**: Frontend, State layers
- **Full-stack**: If task spans multiple layers (rare, prefer atomic tasks)

**D. Context Injection**:

**For Backend Tasks**:
1. **Extract from Tech Spec § 5.3 (API Contracts)**:
   - If file is Controller/API: Copy endpoint signature (Method, Path, Request/Response types)
   - If file is Service: Copy service method signature
   - Extract which project owns it (${PROJECT_CMS_API}, ${PROJECT_DATA_API})

2. **Extract from Tech Spec § 5.2 (Data Model)**:
   - If touching database: Copy entity schema (TypeScript interface)
   - If migration: Copy migration description
   - Show before/after if MODIFY action

3. **Extract from Tech Spec § 2**:
   - Pattern to follow (e.g., "Following ${PROJECT_CMS_API} NestJS controller pattern")
   - Reference file (if pattern reuse mentioned)

**For Frontend Tasks**:
1. **Check Figma** (from Epic):
   - If Figma link exists in Epic: Use `mcp1_get_design_context` with node ID
   - Extract Figma tokens: colors, spacing, typography, layout structure
   - Map to project tokens (from `tailwind.config.js`)

2. **Populate UI Implementation Guide**:
   - Structure (e.g., "Flex-col layout, centered items")
   - Key Tokens: Background, Spacing, Typography
   - Component Reuse (from `${FILE_CATEGORIZATION_PATH}` or Tech Spec § 2)

3. **Extract from Tech Spec § 2**:
   - Pattern to follow (e.g., "Following ${PROJECT_FRONTEND} Redux state management")
   - Reference component (if reusing pattern)

4. **Extract from Tech Spec § 5.3** (if API calls):
   - API endpoint signature the component will call
   - Request/Response types

**For Full-stack Tasks**:
- Include BOTH backend API signature AND frontend Figma tokens
- Define data flow: Component → API Client → Endpoint

**E. Extract Test Plan** (from Tech Spec § 5.5):
- Unit test mock dependencies
- Test cases that map to Epic's Acceptance Criteria
- Integration test requirements (if specified)

**F. Define Implementation Steps** (based on layer):

**Database Layer**:
1. Review existing schema (if MODIFY)
2. Create migration file
3. Update entity definition
4. Test migration (up/down)

**Service Layer**:
1. Review similar service from pattern reference (Tech Spec § 2)
2. Implement business logic following pattern
3. Write unit tests with mocks (from Tech Spec § 5.5)

**API Layer**:
1. Review similar controller from pattern reference
2. Implement endpoint following API contract (Tech Spec § 5.3)
3. Write integration tests

**Frontend Layer**:
1. Review reference component (Tech Spec § 2)
2. Implement UI following Figma tokens (from UI Guide)
3. Integrate API client calls
4. Write component tests

**State Layer**:
1. Review existing state pattern
2. Implement state slice/reducer
3. Write state management tests

**G. Determine Dependencies**:
- **Depends On**: Which tasks in previous layers must complete first
  - Database tasks: None (first layer)
  - Service tasks: Depends on Database tasks touching same entities
  - API tasks: Depends on Service tasks they call
  - Frontend tasks: Depends on API tasks they consume
  - State tasks: Depends on Frontend components using the state

### Step 4: Order Tasks by Dependency
- Tasks are already layer-ordered from Tech Spec
- Within each layer, order by file dependency if applicable
- Assign order number: Task 1, Task 2, etc.

### Step 5: Populate Task Template
For each task, fill `task.md` template with all extracted context:
- **Layer** and **Category** from Tech Spec § 5.4
- **Target Files** with Action (NEW/MODIFY)
- **Pattern Context** from Tech Spec § 2
- **Implementation Context**:
  - Backend: API signatures, Entity schemas
  - Frontend: Figma tokens, Component reuse
- **Steps** based on layer (see Step 3F above)
- **Test Plan** from Tech Spec § 5.5
- **Dependencies** identified in Step 3G

### Step 6: Quality Gate Validation
Before publishing, verify EVERY task has:
- ✅ **Layer** specified (Database/Service/API/Frontend/State)
- ✅ **Category** from ${FILE_CATEGORIZATION_PATH}
- ✅ **Source reference**: Tech Spec § section number
- ✅ **Target files** with Action (NEW or MODIFY)
- ✅ **Pattern Context**: Reference file or architecture pattern from Tech Spec § 2
- ✅ **Implementation Context**:
  - Backend: API signature OR entity schema
  - Frontend: UI Implementation Guide with mapped tokens
  - Full-stack: Both
- ✅ **Dependencies**: Which tasks must complete first (or "None" if first layer)
- ✅ **Test Plan**: At least 2 cases, references Tech Spec § 5.5
- ✅ **Steps**: Layer-appropriate implementation steps

### Step 7: Publish to Jira
- Create Jira Issues using `mcp0_createJiraIssue` with:
  - `projectKey`: "${JIRA_PROJECT_KEY}"
  - `issueTypeName`: "Task" or "Story"
  - `parent`: Epic key (to link tasks to Epic)
  - `summary`: Task title (e.g., "Layer: Create FeatureService.ts")
  - `description`: Fully populated task.md content
- Add Tech Spec Confluence link to each task description

### Step 8: Verify Links
- Use `mcp0_getJiraIssueRemoteIssueLinks` to verify:
  - All tasks linked to Epic (parent field)
  - All tasks reference Tech Spec Confluence page

### Step 9: Presentation & Gate
- **Display Summary**:
  - Total tasks created: [N]
  - By layer breakdown (e.g., "Database: 2, Service: 3, API: 2, Frontend: 4")
  - Dependency order confirmed: Task 1 → Task 2 → ... → Task N
- **List All Tasks** with keys, titles, and layer
- **STOP** and request approval

**Standard Approval Format**:
```
✅ **TaskPlanning Complete**
- **Artifact**: [List of Task Keys: ${JIRA_PROJECT_KEY}-XXX, ${JIRA_PROJECT_KEY}-YYY, ${JIRA_PROJECT_KEY}-ZZZ]
- **Summary**: Created N atomic tasks with API signatures and UI guides
- **Next Step**: Implementation Mode (select a task to start)

> **⏸️ APPROVAL REQUIRED**: Please review the tasks in Jira. Reply with:
> - `Approve` to begin implementation (then select a task)
> - `Revise [feedback]` to adjust tasks
> - `Cancel` to stop workflow
```
