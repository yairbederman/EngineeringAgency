# Engineering Agent – TechSpec Mode

> **Persona**: Load `${AGENT_ROOT}/personas/system-architect.md`

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
- **Tech Spec**: Detailed Markdown content for user review.
- **Jira Task**: Created as Epic child after user approval of the content.

**Critical Rules**:
1.  **Validation**: Every architecture decision must reference either:
    - A specific section from `${COPILOT_INSTRUCTIONS_PATH}`
    - An existing pattern from Context7 (provide file path)
    
    > **Evidence Required**:
    > - File paths consulted
    > - Code snippets verified
2.  **API Contracts**: Must define strict TypeScript interfaces for ALL endpoints/services
3.  **Data Model**: Must define migration strategy if DB changes required
4.  **No Guessing**: Use `[TBD – requires input from Backend]` if schema/API is unknown
5.  **Multi-Project Awareness**: Identify which workspace project(s) are affected and review their specific architecture
6.  **No Auto-Creation**: NEVER create Jira Tasks or Confluence Pages without explicit user approval.

**Flow**:

### Step 1: Read and Deconstruct Epic (Understand Requirements)
- Fetch Epic from Jira using `${MCP_ATLASSIAN_GET_ISSUE}`
- **Deconstruct Epic into Components**:
  - Extract all **Functional Flows** (Happy Path, Error paths, Edge cases)
  - Extract **Acceptance Criteria** (Given/When/Then scenarios)
  - Extract **Data Concepts** (Inputs, Outputs, business entities)
  - Extract **Scope & Constraints** (What's in/out, platform requirements)
  - Extract **Figma Links** (if present for UI work)

### Step 2: Identify Affected Projects

> **⛔ CRITICAL**: Read system architecture files BEFORE finalizing project scope to discover transitive dependencies.

**Step 2.1: Read System Topology FIRST** (from `/system-architecture-agent` output):
1. **Read `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json`**
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
     - **Read `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json`** for the specific API contracts
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

> [!CAUTION]
> **Data Availability Verification (MANDATORY)**
> For EACH required data field in the Epic, you MUST:
> 1. **Search** for the field in frontend types/classes (e.g., `grep_search` for `fieldName`)
> 2. **Verify** it EXISTS in the data model (class properties, type definitions)
> 3. **Trace** the data path: API → Redux → Component
> 
> If field is **MISSING**:
> - Mark as **BLOCKER** in Tech Spec
> - Identify required change: Backend API addition OR frontend extraction
> - Add to **Assumptions Log** with High Risk
> 
> Do NOT leave as `[INVESTIGATE]` TODO. Complete the investigation NOW.

**For each Acceptance Criterion**:
- Define **testability approach**: unit, integration, E2E
- Map test to specific component/service

### Step 4.5: UI Design Extraction (MANDATORY if Figma Links Present)

> [!IMPORTANT]
> **If the Epic contains Figma links**, you MUST extract and document all UI design specifications.
> This ensures Tasks have all design context needed for pixel-perfect implementation.

> [!TIP]
> **Tool Priority**: Always prioritize **Figma MCP tools** (`${MCP_FIGMA_GET_DESIGN}`, etc.) over browser subagent.
> MCP provides structured data extraction (tokens, variables, component properties) directly.
> Use browser subagent only as fallback when MCP is unavailable or for screenshot capture.

**For EACH Figma Link in Epic**:

1. **Extract via Figma MCP** (PREFERRED):
   - Use `${MCP_FIGMA_GET_DESIGN}` to get component structure and tokens
   - Use `${MCP_FIGMA_GET_VARS}` for design system variables
   - Use `${MCP_FIGMA_GET_SCREENSHOT}` for visual reference
   - **Fallback**: If MCP unavailable, use browser subagent to navigate and capture

2. **Extract Component Structure**:

   ```text
   ComponentName
   ├── Header (optional)
   │   └── Title / Controls
   └── Body
       ├── Row/Item (repeated)
       │   ├── Icon
       │   └── TextStack
       └── Footer (optional)
   ```

3. **Extract Design Tokens** (map Figma values → project tokens):
   - Background colors, text colors, accent colors
   - Border radius, shadows
   - Spacing (padding, margins, gaps)

4. **Extract Typography**:
   - Font family, sizes, weights for each text element

5. **Extract Icons**:
   - List icons with color variants
   - Cross-reference with project icon library

6. **Document RTL/Responsive Behavior**:
   - RTL text alignment requirements
   - Mobile/touch interaction differences

7. **Capture Screenshots** (MANDATORY for UI implementation accuracy):

   > [!TIP]
   > **Screenshot Quality Checklist** - Screenshots should enable pixel-perfect implementation:
   
   **Via Figma MCP** (PREFERRED):
   - Use `${MCP_FIGMA_GET_SCREENSHOT}` with specific node ID
   - This captures the exact component at proper resolution
   
   **Via Browser Subagent** (FALLBACK):
   - **Zoom to 100%** before capturing (avoid zoomed-out canvas views)
   - **Dismiss login/modal popups** before screenshot
   - **Select the specific component** to focus the view
   - **Capture each state separately** (e.g., hover, default, error)
   
   **Screenshot Requirements**:
   - [ ] Component fills most of the frame (not tiny in large canvas)
   - [ ] All text is readable
   - [ ] Color accuracy preserved (no compression artifacts)
   - [ ] Each variant/state has its own screenshot
   - [ ] Save to artifacts directory with descriptive names (e.g., `baggage_tooltip_included.png`, `baggage_tooltip_paid.png`)
   
   **Embed in Tech Spec**:
   ```markdown
   ![Component Name - State](path/to/screenshot.png)
   ```

**Output**: Add "UI Design Specifications" section to Tech Spec document.


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

#### 5.4 Sequence Diagram (MANDATORY)

**Purpose**: Visualize the proposed solution to enable quick understanding of the flow.

> [!IMPORTANT]
> Every Tech Spec MUST include an ASCII sequence diagram showing:
> 1. **Participants**: User, UI Components, Services, APIs involved
> 2. **Happy Path Flow**: Main success scenario interactions
> 3. **Conditional Branches**: ALT/ELSE blocks for different states
> 4. **Data Flow**: What data moves between participants

**Format**: Use ASCII art in a code block (renders correctly in Jira/Confluence):

```
┌──────────────────────────────────────────────────────────────────┐
│                     SEQUENCE DIAGRAM                             │
│                   [Feature Name] Flow                            │
└──────────────────────────────────────────────────────────────────┘

     User              Component1              Component2         
       │                    │                       │              
       │   [Action]         │                       │              
       │───────────────────>│                       │              
       │                    │                       │              
       │                    │   [Process]           │              
       │                    │──────────────────────>│              
       │                    │                       │              
       │ ┌──────────────────┼───────────────────────┤              
       │ │ ALT [Condition = true]                   │              
       │ │<─────────────────┼─── [Success response]─│              
       │ ├──────────────────┼───────────────────────┤              
       │ │ ELSE [Condition = false]                 │              
       │ │<─────────────────┼─── [Error response]  ─│              
       │ └──────────────────┼───────────────────────┤              
       │                    │                       │              
```

**ASCII Symbols**:
- `│` Vertical line (lifeline)
- `─` Horizontal line (message)
- `>` Arrow direction
- `┌┐└┘` Box corners
- `├┤` Branch points

---

#### 5.5 Implementation Inventory (The Dependency-Ordered Roadmap)

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

### Step 6: Present Tech Spec for Review

> [!IMPORTANT]
> **DO NOT** create Confluence pages or update Jira issues yet.
> You must first present the generated Tech Spec content to the user for review.

1. Display the full Tech Spec markdown in the chat.
2. Explicitly ask for approval of the content.
3. **MANDATORY**: Ask if the user wants to "Inject to Jira" (create as Epic child Task).

### Step 7: Inject to Jira (ONLY IF AUTHORIZED)

**Condition**: Only proceed if the user explicitly confirms "Inject to Jira".

**Create Jira Task (Epic Child)**:
1. Create a **Jira Task** under the Epic using `${MCP_ATLASSIAN_CREATE_ISSUE}`:
   - `projectKey`: "${JIRA_PROJECT_KEY}"
   - `issueTypeName`: "Task"
   - `parent`: [Epic Key]
   - `summary`: "Tech Spec: [Feature Name]"
   - `description`: The full Tech Spec markdown content
   - `additional_fields`: Include all custom fields defined in `configuration.md` → "Jira Required Custom Fields" table. Example: `{"customfield_XXXXX": {"id": "VALUE_ID"}}`
2. **Verify** the Task is linked as a child of the Epic.

**Link Back (Bidirectional Traceability)**:
- Update Product Spec Links table with Tech Spec Jira link via `${MCP_ATLASSIAN_ADD_FOOTER_COMMENT}`.
- Update Epic description using `${MCP_ATLASSIAN_EDIT_ISSUE}` to replace "[TBD - Will be added after TechSpec phase]" with actual Tech Spec Task link.

### Step 8: Standard Approval & Gate

Display the status and next steps using the standard format.

**Standard Approval Format**:
```
✅ **TechSpec Content Generated**
- **Status**: [Content Ready | Injected to Atlassian]
- **Artifact**: [Tech Spec URL or "Local Markdown Only"]
- **Summary**: Concrete action plan with [N] files across [X] project(s).
- **Next Step**: TaskPlanning Mode

> **⏸️ APPROVAL REQUIRED**: Please review the Tech Spec.
>
> **Decision: Should this be injected to Jira?**
> - `Yes, Inject & Proceed`: Create Jira Task (Source of Truth) and move to Task Breakdown.
> - `No, Local Only`: Keep local `implementation_plan.md` and move to Task Breakdown.
> - `Revise [feedback]`: Request changes to the content.
```

