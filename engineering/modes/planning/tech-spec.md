# Engineering Agent – TechSpec Mode

## 3. TechSpec Mode – Epic to Implementation Plan

**Goal**: Translate the functional Epic into a concrete, actionable implementation roadmap with architecture decisions, API contracts, and dependency-ordered file changes.

**Prerequisite**: Approved Epic with Jira link.

**Inputs**:
- Approved Epic (from Jira) - LLM-Ready Functional Contract
- Project(s) `${COPILOT_INSTRUCTIONS_PATH}` (architecture rules per project)
- `${FILE_CATEGORIZATION_PATH}` (component organization)
- Context7 (existing patterns to reuse)
- Figma (if UI-heavy feature)

**Output**:
- **Tech Spec** using the `tech-spec.md` template (Concrete Action Plan)
- **Confluence Page**: Published as child of Tech Specs folder (Parent ID: ${TECH_SPECS_FOLDER_ID})
- **Confluence Update**: Product Spec page updated with Tech Spec link

**Critical Rules**:
1.  **Validation**: Every architecture decision must reference either:
    - `${COPILOT_INSTRUCTIONS_PATH}` section, or
    - Existing pattern from Context7
2.  **API Contracts**: Must define strict TypeScript interfaces for ALL endpoints/services
3.  **Data Model**: Must define migration strategy if DB changes required
4.  **No Guessing**: Use `[TBD – requires input from Backend]` if schema/API is unknown
5.  **Multi-Project Awareness**: Identify which workspace project(s) are affected and review their specific architecture

**Flow**:

### Step 1: Read and Deconstruct Epic (Understand Requirements)
- Fetch Epic from Jira using `mcp0_getJiraIssue`
- **Deconstruct Epic into Components**:
  - Extract all **Functional Flows** (Happy Path, Error paths, Edge cases)
  - Extract **Acceptance Criteria** (Given/When/Then scenarios)
  - Extract **Data Concepts** (Inputs, Outputs, business entities)
  - Extract **Scope & Constraints** (What's in/out, platform requirements)
  - Extract **Figma Links** (if present for UI work)

### Step 2: Identify Affected Projects

> **⛔ CRITICAL**: Read system architecture files BEFORE finalizing project scope to discover transitive dependencies.

**Step 2.1: Read System Topology FIRST** (from `/system-architecture-agent` output):
1. **Read `${SYSTEM_ARCH_ROOT}/analysis/service-topology.json`**
2. Review all services and their `callsServices` + `calledBy` relationships
3. This context informs project identification below

**Step 2.2: Determine Initial Project Scope**: Based on Epic requirements, identify which workspace project(s) are directly impacted:
  - `${PROJECT_FRONTEND}` (Frontend/React)
  - `${PROJECT_CMS_API}` (CMS Backend)
  - `${PROJECT_DATA_API}` (Data Backend)
  - [Other projects from configuration.md]

**Step 2.3: Expand Scope via Cross-Project Impact Analysis**:
  1. For each identified project, check in `service-topology.json`:
     - Which services does it **call**? (`callsServices` - downstream dependencies)
     - Which services **call it**? (`calledBy` - upstream consumers that may be affected)
  2. If cross-service dependencies exist:
     - **Add those services to scope** (even if not initially identified)
     - **Read `${SYSTEM_ARCH_ROOT}/analysis/cross-service-apis.json`** for the specific API contracts
  3. Document in Tech Spec under "Cross-Project Impact" section

- **Project Selection Criteria**:
  - Frontend work (UI, forms, displays) → `${PROJECT_FRONTEND}`
  - CMS/Admin operations → `${PROJECT_CMS_API}`
  - Data processing, external APIs → `${PROJECT_DATA_API}`
  - Full-stack features → Multiple projects

> [!IMPORTANT]
> **If `service-topology.json` doesn't exist**:
> 1. **RECOMMEND**: "Run `/system-architecture-agent` first for complete cross-project visibility"
> 2. **IF PROCEEDING**: Document ALL assumed cross-project impacts in "Assumptions" section
> 3. **ADD LABEL**: Add `needs-system-architecture` label to Epic
> 4. **RISK**: Cross-project API changes may break consumers without proper impact analysis

### Step 3: Review Codebase Architecture (Per Project)
For **each affected project**:

- **Read Project Instructions**:
  - Read `[project-root]/${COPILOT_INSTRUCTIONS_PATH}`
  - Extract:
    - Architecture patterns (e.g., "All state management uses Redux", "Services live in /src/services")
    - Domain boundaries (e.g., "Canvas logic must use `useCanvas` hook")
    - Integration rules (e.g., "API calls via `ApiClient` class")
    - Testing conventions
    - File naming patterns
  
- **Read File Categorization**:
  - Read `[project-root]/${FILE_CATEGORIZATION_PATH}` (if exists)
  - Understand component categories: `react-components`, `services`, `controllers`, `repositories`, etc.
  - Identify where similar code lives (pattern location)

- **Pattern Matching** (Context7 / codebase_search):
  - Query: "What are the existing patterns for [Epic's core feature]?"
  - Examples:
    - "How do we handle form submissions in ${PROJECT_FRONTEND}?"
    - "Where are CRUD services implemented in ${PROJECT_CMS_API}?"
    - "How do we process external API responses in ${PROJECT_DATA_API}?"
  - Identify:
    - Reusable utilities (don't reinvent)
    - Shared services (extend vs create new)
    - Common components (reuse vs build)

### Step 4: Map Epic to Architecture (Requirements → Technical Design)

**For each Functional Flow in Epic**:
- Identify which **architectural layer** handles it:
  - **Frontend (${PROJECT_FRONTEND})**: UI components, state management, forms
  - **Backend API (${PROJECT_CMS_API}/${PROJECT_DATA_API})**: Controllers, business logic, data access
  - **Database**: Entity changes, new tables, migrations
- Determine if existing patterns can be **reused or extended**
- Define data flow: User → UI → API → Service → DB (and reverse)

**For each Data Concept**:
- Map to **database entities** (new or modify existing)
- Define **API contracts** if cross-service or frontend-backend communication needed
- Specify **validation rules** (where validated: frontend, backend, or both)

**For each Acceptance Criterion**:
- Define **testability approach**: unit, integration, E2E
- Map test to specific component/service

### Step 5: Generate Concrete Action Plan (Tech Spec Document)

Fill `tech-spec.md` template with:

#### 5.1 Architecture & Patterns
- **Per-Project Compliance**:
  - For each affected project, explain how Epic maps to its architecture
  - Reference specific sections from that project's `${COPILOT_INSTRUCTIONS_PATH}`
  - Example: "Following ${PROJECT_FRONTEND}'s state management pattern using Redux (§3.2)"
- **Pattern Reuse**:
  - List similar existing features found via Context7
  - Reference file paths: "Mimicking UserProfile component pattern from `src/components/UserProfile.tsx`"
- **Deviations** (if any):
  - Justify why deviating from current patterns
  - Explain alternative approach

#### 5.2 Data Model (Schema & Storage)
- **Define Entities** (TypeScript interfaces):
  - New entities: Full schema definition
  - Modified entities: Show before/after comparison
- **Migration Strategy**:
  - SQL migrations required? (Yes/No + description)
  - Data migration needed? (if modifying existing data)
- **Ownership**: Which database (CMS DB, Data DB, shared)

#### 5.3 API Contracts (The "Hard" Contract)
- **For each endpoint**:
  - Method + Path: `POST /api/v1/resource`
  - Request Type: TypeScript interface
  - Response Type: TypeScript interface
  - Which service owns it: `${PROJECT_CMS_API}` or `${PROJECT_DATA_API}`
- **Service Methods** (internal APIs):
  - Signature: `public async methodName(arg: Type): Promise<Type>`
  - Which project: `${PROJECT_FRONTEND}`, `${PROJECT_CMS_API}`, `${PROJECT_DATA_API}`

#### 5.4 Implementation Inventory (The Dependency-Ordered Roadmap)

**Purpose**: This section IS the concrete action plan - a step-by-step roadmap ordered by dependency.

**Structure by Dependency Layer**:

1. **Database Layer** (Do First):
   - [ ] `[Project]/migrations/YYYY-MM-DD-migration-name.sql` (NEW)
   - [ ] `[Project]/entities/EntityName.ts` (MODIFY) - Add field X

2. **Service Layer** (After Database):
   - [ ] `${PROJECT_CMS_API}/src/services/FeatureService.ts` (NEW) - Category: `services`
   - [ ] `${PROJECT_DATA_API}/src/repositories/FeatureRepository.ts` (NEW) - Category: `repositories`

3. **API Layer** (After Services):
   - [ ] `${PROJECT_CMS_API}/src/controllers/FeatureController.ts` (NEW) - Category: `controllers`
   - [ ] Change: Implements POST /api/v1/feature endpoint

4. **Frontend Layer** (After API):
   - [ ] `${PROJECT_FRONTEND}/src/components/FeatureComponent.tsx` (NEW) - Category: `react-components`
   - [ ] `${PROJECT_FRONTEND}/src/services/FeatureApiClient.ts` (NEW) - Category: `services`
   - [ ] `${PROJECT_FRONTEND}/src/pages/FeaturePage.tsx` (MODIFY) - Add new component

**Per-File Detail**:
- File path (absolute from project root)
- Action: NEW or MODIFY
- Category: from `${FILE_CATEGORIZATION_PATH}`
- Change summary: Brief description of what changes

#### 5.5 Verification Strategy
- **Unit Tests**:
  - Mock dependencies: `[Dependency]` → `[MockValue]`
  - Critical test cases: Map to Epic's Acceptance Criteria
- **Integration Tests**:
  - API endpoint tests (request/response validation)
  - Service integration tests
- **E2E Tests** (if applicable):
  - User flow tests: Map to Epic's Functional Flows

### Step 6: Publish to Confluence
- Use `mcp0_createConfluencePage` with:
  - `cloudId`: Extract from workspace context
  - `spaceId`: ${CONFLUENCE_SPACE_KEY} space ID
  - `parentId`: `${TECH_SPECS_FOLDER_ID}` (Tech Specs folder)
  - `title`: "Tech Spec: [Feature Name]"
  - `body`: Generated tech spec content (markdown)

**Failure Handling (Fallback)**:
- If Confluence publication fails (tool error, permission issue):
  1. Create a **Jira Task** under the Epic using `mcp0_createJiraIssue`.
     - `projectKey`: "${JIRA_PROJECT_KEY}"
     - `issueTypeName`: "Task"
     - `parent`: [Epic Key]
     - `summary`: "Tech Spec: [Feature Name]"
     - `description`: The full Tech Spec markdown content.
  2. Treat this Jira Task URL as the "Tech Spec Artifact" for subsequent steps.

### Step 7: Update Product Spec
- Update Links table with Tech Spec link (mandatory)
- Use `mcp0_createConfluenceFooterComment`

### Step 8: Presentation & Gate
- Display created Tech Spec URL
- **STOP** and request approval

**Standard Approval Format**:
```
✅ **TechSpec Complete**
- **Artifact**: [Tech Spec Confluence URL]
- **Summary**: Concrete action plan with [N] files across [X] project(s): dependency-ordered roadmap from DB → Services → API → UI
- **Next Step**: TaskPlanning Mode

> **⏸️ APPROVAL REQUIRED**: Please review the Tech Spec in Confluence. Reply with:
> - `Approve` to proceed to Task decomposition
> - `Revise [feedback]` to make changes
> - `Cancel` to stop workflow
```
