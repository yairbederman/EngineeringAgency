# Tech Spec: [Feature Name]

**Reference**: Epic [Jira Key]  
**Projects Affected**: [${PROJECT_FRONTEND} | ${PROJECT_CMS_API} | ${PROJECT_DATA_API} | ...]  
**Validation**: All architecture decisions validated against project-specific `${COPILOT_INSTRUCTIONS_PATH}` and existing codebase patterns.

---

## 1. Epic Deconstruction (Requirements Summary)

### Functional Flows
- **Happy Path**: [Describe main user flow]
- **Error Cases**: [Describe error handling scenarios]

### Acceptance Criteria (From Epic)
```gherkin
Scenario: [Name]
  Given [precondition]
  When [action]
  Then [expected result]
```

### Data Concepts
- **Inputs**: [List business inputs from Epic]
- **Outputs**: [List expected outputs from Epic]

### Scope & Constraints
- **In Scope**: [What this spec covers]
- **Out of Scope**: [What is explicitly excluded]

---

## 2. Architecture & Patterns

### Per-Project Compliance

#### [Project Name] (e.g., ${PROJECT_FRONTEND})
- **Architecture Pattern**: [Pattern from `${COPILOT_INSTRUCTIONS_PATH} § X.X`]
- **Compliance**: [Explain how this spec follows the project's patterns]
- **Example**: "Following ${PROJECT_FRONTEND}'s state management pattern using Redux (§3.2)"

#### [Project Name] (e.g., ${PROJECT_CMS_API})
- **Architecture Pattern**: [Pattern from `${COPILOT_INSTRUCTIONS_PATH} § X.X`]
- **Compliance**: [Explain compliance]

### Pattern Reuse (From Context7)
- **Similar Feature**: [Name of existing feature to mimic]
- **Reference File**: `[path/to/similar/file.ts]`
- **Rationale**: [Why this pattern is being reused or extended]

### Deviations (If Any)
- **Deviation**: [What deviates from current patterns]
- **Justification**: [Why this deviation is necessary]
- **Alternative Approach**: [Proposed solution]

---

## 2.5 UI Design Specifications (If Figma Links Present)

> **Note**: This section is MANDATORY when the Epic contains Figma links. Remove if not applicable.

### Figma References

| State/Variant | Figma Link | Screenshot |
|---------------|------------|------------|
| [State 1 Name] | [Figma URL] | ![Screenshot](path/to/screenshot.png) |
| [State 2 Name] | [Figma URL] | ![Screenshot](path/to/screenshot.png) |

### Component Structure

```text
[ComponentName]
├── Header (optional)
│   └── [Title / Controls]
└── Body
    ├── [Row/Item] (repeated)
    │   ├── [Icon]
    │   └── [TextStack]
    └── Footer (optional)
```

### Design Token Mapping

| Property | Figma Value | Project Token |
|----------|-------------|---------------|
| Background Color | `#FFFFFF` | `--bg-white` |
| Primary Color | `[Hex]` | `--color-primary` |
| Success Color | `[Hex]` | `--color-success` |
| Warning Color | `[Hex]` | `--color-error` |
| Text Primary | `[Hex]` | `--text-primary` |
| Text Secondary | `[Hex]` | `--text-secondary` |
| Border Radius | `[Npx]` | `--radius-[size]` |
| Padding | `[Npx]` | `--spacing-[N]` |
| Shadow | `[CSS shadow]` | `--shadow-[size]` |

### Typography

| Element | Font Family | Size | Weight |
|---------|-------------|------|--------|
| Header Title | [Family] | [Npx] | [Weight] |
| Body Text | [Family] | [Npx] | [Weight] |
| Caption | [Family] | [Npx] | [Weight] |

### Icons Used

| Icon | Purpose | Color Variant | Project Component |
|------|---------|---------------|-------------------|
| [Icon name] | [Purpose] | [Color] | `<IconResolver type="[Type]" />` |

### RTL & Responsive Behavior

- **RTL Layout**: [Yes/No] - [Text alignment requirements]
- **Mobile Touch**: [Hover vs Click behavior on mobile]
- **Breakpoints**: [Any layout changes at specific breakpoints]

---

## 3. Data Model (Schema & Storage)


### New Entities

```typescript
// [Project]/entities/EntityName.ts
interface EntityName {
  id: string;
  field1: string;
  field2: number;
  createdAt: Date;
  updatedAt: Date;
}
```

### Modified Entities

**Before**:
```typescript
interface ExistingEntity {
  id: string;
  oldField: string;
}
```

**After**:
```typescript
interface ExistingEntity {
  id: string;
  oldField: string;
  newField: boolean; // ADDED
}
```

### Migration Strategy
- [ ] **SQL Migration Required**: [Yes/No]
  - **Description**: [e.g., "Add `is_active` column to `users` table, default true"]
  - **File**: `[Project]/migrations/YYYY-MM-DD-add-field.sql`
- [ ] **Data Migration Required**: [Yes/No if existing data needs transformation]

### Database Ownership
- **Database**: [CMS DB | Data DB | Shared]
- **Project**: [${PROJECT_CMS_API} | ${PROJECT_DATA_API}]

---

## 4. API & Interface Contracts (The "Hard" Contract)

### REST Endpoints

#### Endpoint 1
- **Method & Path**: `POST /api/v1/resource`
- **Owned By**: [${PROJECT_CMS_API} | ${PROJECT_DATA_API}]
- **Request**:
  ```typescript
  type CreateResourceRequest = {
    field1: string;
    field2: number;
  }
  ```
- **Response**:
  ```typescript
  type CreateResourceResponse = {
    id: string;
    field1: string;
    field2: number;
    createdAt: string;
  }
  ```
- **Validation Rules**:
  - `field1`: Required, max 255 chars
  - `field2`: Required, positive integer

#### Endpoint 2
- **Method & Path**: `GET /api/v1/resource/:id`
- **Owned By**: [${PROJECT_CMS_API} | ${PROJECT_DATA_API}]
- [... similar structure ...]

### Service Methods (Internal APIs)

#### [Project]/src/services/ServiceName.ts
```typescript
class ServiceName {
  public async methodName(arg: ArgType): Promise<ReturnType> {
    // Implementation contract
  }
}
```

### Frontend API Client

#### ${PROJECT_FRONTEND}/src/services/ApiClient.ts
```typescript
class ApiClient {
  async callEndpoint(data: RequestType): Promise<ResponseType> {
    // Frontend-to-backend contract
  }
}
```

---

## 5. Sequence Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                     SEQUENCE DIAGRAM                             │
│                   [Feature Name] Flow                            │
└──────────────────────────────────────────────────────────────────┘

     User              [Component1]            [Component2]        
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

---

## 6. Implementation Inventory (Concrete Action Plan)

**Purpose**: This is the dependency-ordered roadmap. Execute top-to-bottom.

### Layer 1: Database (Do First)

- [ ] **[NEW]** `${PROJECT_CMS_API}/migrations/2025-XX-XX-create-resource.sql`
  - **Category**: `migrations`
  - **Change**: Create `resources` table with fields: id, field1, field2, created_at, updated_at

- [ ] **[MODIFY]** `${PROJECT_CMS_API}/src/entities/ExistingEntity.ts`
  - **Category**: `entities`
  - **Change**: Add `newField: boolean` property

### Layer 2: Service (After Database)

- [ ] **[NEW]** `${PROJECT_CMS_API}/src/services/ResourceService.ts`
  - **Category**: `services`
  - **Change**: Implement business logic for resource CRUD operations

- [ ] **[NEW]** `${PROJECT_CMS_API}/src/repositories/ResourceRepository.ts`
  - **Category**: `repositories`
  - **Change**: Implement database access layer for resources table

### Layer 3: API (After Services)

- [ ] **[NEW]** `${PROJECT_CMS_API}/src/controllers/ResourceController.ts`
  - **Category**: `controllers`
  - **Change**: Implement `POST /api/v1/resource` and `GET /api/v1/resource/:id` endpoints

- [ ] **[MODIFY]** `${PROJECT_CMS_API}/src/routes/index.ts`
  - **Category**: `routes`
  - **Change**: Register ResourceController routes

### Layer 4: Frontend (After API)

- [ ] **[NEW]** `${PROJECT_FRONTEND}/src/services/ResourceApiClient.ts`
  - **Category**: `services`
  - **Change**: Implement API client methods for resource endpoints

- [ ] **[NEW]** `${PROJECT_FRONTEND}/src/components/ResourceForm.tsx`
  - **Category**: `react-components`
  - **Change**: Build form component for creating/editing resources
  - **Figma**: [Link to Figma frame if applicable]

- [ ] **[NEW]** `${PROJECT_FRONTEND}/src/components/ResourceList.tsx`
  - **Category**: `react-components`
  - **Change**: Build list component for displaying resources

- [ ] **[MODIFY]** `${PROJECT_FRONTEND}/src/pages/ResourcePage.tsx`
  - **Category**: `pages`
  - **Change**: Integrate ResourceForm and ResourceList components

### Layer 5: State Management (If Applicable)

- [ ] **[NEW]** `${PROJECT_FRONTEND}/src/store/slices/resourceSlice.ts`
  - **Category**: `redux-slices`
  - **Change**: Implement Redux slice for resource state management

---

## 7. Verification Strategy

### Unit Tests

#### Backend (${PROJECT_CMS_API})
- [ ] **File**: `${PROJECT_CMS_API}/src/services/__tests__/ResourceService.test.ts`
  - **Mock**: Repository → returns mock data
  - **Test Cases**:
    - ✅ Happy Path: Create resource successfully
    - ✅ Edge Case: Validation fails for invalid input
    - ✅ Error Case: Database connection failure

#### Frontend (${PROJECT_FRONTEND})
- [ ] **File**: `${PROJECT_FRONTEND}/src/components/__tests__/ResourceForm.test.tsx`
  - **Mock**: API client → returns mock response
  - **Test Cases**:
    - ✅ Happy Path: Form submission success
    - ✅ Edge Case: Form validation errors displayed
    - ✅ Acceptance Criteria: Maps to Epic's "User can create resource" scenario

### Integration Tests

- [ ] **File**: `${PROJECT_CMS_API}/tests/integration/resource.test.ts`
  - **Test Cases**:
    - ✅ API endpoint: POST /api/v1/resource returns 201
    - ✅ API endpoint: GET /api/v1/resource/:id returns resource
    - ✅ Database: Resource persisted correctly

### E2E Tests (If Applicable)

- [ ] **File**: `e2e/tests/resource-flow.spec.ts`
  - **User Flow**: Maps to Epic's "Happy Path" functional flow
  - **Test Steps**:
    1. User navigates to resource page
    2. User fills form with valid data
    3. User submits form
    4. System displays success message
    5. Resource appears in list

---

## 7. Summary

**Total Files**: [X files across Y project(s)]
- **New Files**: [N]
- **Modified Files**: [M]

**Dependency Order**: Database → Services → API → Frontend → State Management

**Projects Impacted**:
- [x] ${PROJECT_FRONTEND}: [count] files
- [x] ${PROJECT_CMS_API}: [count] files
- [ ] ${PROJECT_DATA_API}: [count] files (if applicable)

**Estimated Complexity**: [Low | Medium | High]

**Risks/Unknowns**:
- [TBD – requires input from Product]: [List any unknowns]
- [TBD – requires input from Design]: [List any design decisions needed]
- [TBD – requires input from Backend]: [List any backend decisions needed]
