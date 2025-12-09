## Task: [Name]

**Layer**: [Database | Service | API | Frontend | State]  
**Category**: [e.g., `migrations`, `services`, `controllers`, `react-components`]  
**Type**: [Backend | Frontend | Full-stack]  
**Source**: Tech Spec § [Section Number]

---

### Target Files

**Action: NEW**
- `[absolute/path/to/new/file.ts]`

**Action: MODIFY**
- `[absolute/path/to/existing/file.ts]`

---

### Dependencies

**Depends On**: [List task numbers/names that must complete first, or "None" if first layer]

**Example**:
- Task 1: Create migration for navigation_items table (Database layer)
- Task 2: Create NavigationEntity.ts (Database layer)

OR

- None (this is a Database layer task)

---

### Pattern Context (From Tech Spec § 2)

**Pattern to Follow**: [Architecture pattern from Tech Spec]  
**Example**: "Following ${PROJECT_FRONTEND} Redux state management pattern (§ 3.2 from .ai-instructions)"

**Reference File**: [Existing file to mimic, if applicable]  
**Example**: `src/components/UserProfile/UserProfile.tsx`

**Rationale**: [Why this pattern/file is being reused]  
**Example**: "UserProfile component uses the same layout structure and data fetching pattern"

---

### Implementation Context

#### For Backend Tasks:

**API Contract** (From Tech Spec § 5.3):
```typescript
// Endpoint: [METHOD] [Path]
// Owned by: [${PROJECT_CMS_API} | ${PROJECT_DATA_API}]

// Request
type RequestType = {
  field: string;
}

// Response
type ResponseType = {
  id: string;
  field: string;
}
```

**Service Method** (if applicable):
```typescript
// From Tech Spec § 5.3
class ServiceName {
  public async methodName(arg: ArgType): Promise<ReturnType> {
    // Contract signature from Tech Spec
  }
}
```

**Entity Schema** (From Tech Spec § 5.2, if touching database):
```typescript
// From Tech Spec § 5.2
interface EntityName {
  id: string;
  field1: string;
  field2: number;
  createdAt: Date;
}
```

**Before/After** (if MODIFY action):
```typescript
// Before (existing)
interface Entity {
  id: string;
  oldField: string;
}

// After (modified)
interface Entity {
  id: string;
  oldField: string;
  newField: boolean; // ADDED
}
```

**Validation Rules**:
- [List validation constraints from Tech Spec]
- Example: `field1`: Required, max 255 chars
- Example: `field2`: Required, positive integer

---

#### For Frontend Tasks:

**API Endpoint to Call** (From Tech Spec § 5.3, if applicable):
```typescript
// The component will call this endpoint
POST /api/v1/resource
Request: ResourceRequest
Response: ResourceResponse
```

**UI Implementation Guide** (Auto-Generated from Figma):

> **Strictness**: Pixel-perfect implementation required. Do not deviate from tokens.

**Figma Reference**: [Figma URL with node ID, if applicable]

**Structure**: [Layout description]  
**Example**: "Flex-col layout, items start-aligned, full width"

**Key Tokens**:
- **Background**: `[e.g., bg-surface-card, bg-white]`
- **Spacing**: `[e.g., gap-4, p-6, mb-8]`
- **Typography**: `[e.g., text-xl font-bold text-neutral-900]`
- **Colors**: `[e.g., text-primary-600, border-gray-300]`
- **Borders/Shadows**: `[e.g., rounded-lg, shadow-sm]`

**Component Reuse** (From ${FILE_CATEGORIZATION_PATH} or Tech Spec § 2):
- [ ] Use `[ComponentName]` component for `[purpose]`
- **Example**: Use `Avatar` component for user profile image
- **Example**: Use `Button variant="primary"` for submit action

---

### Implementation Steps

> Steps below are **layer-specific**. Follow the sequence for your task's layer.

#### Database Layer Tasks:
1. Review existing schema (if MODIFY action)
2. Create migration file with proper timestamp naming
3. Update entity definition with new fields/constraints
4. Test migration (up/down) locally
5. Verify entity changes with database inspection

#### Service Layer Tasks:
1. Review similar service from pattern reference (Tech Spec § 2)
2. Implement business logic methods following pattern
3. Add validation logic per Tech Spec constraints
4. Write unit tests with mocks (from Test Plan below)
5. Verify service behavior with test coverage

#### API Layer Tasks:
1. Review similar controller from pattern reference
2. Implement endpoint following API contract (Tech Spec § 5.3)
3. Add request validation middleware
4. Add error handling per Epic's error flows
5. Write integration tests for endpoint

#### Frontend Layer Tasks:
1. Review reference component (Tech Spec § 2 Pattern Context)
2. Implement UI structure following Figma tokens (UI Guide above)
3. Integrate API client calls (if applicable)
4. Add form validation (if form component)
5. Write component tests (from Test Plan below)

#### State Layer Tasks:
1. Review existing state management pattern
2. Create state slice/reducer following pattern
3. Define actions and selectors
4. Integrate with relevant components
5. Write state management unit tests

---

### Test Plan (From Tech Spec § 5.5)

**Test File**: `[path/to/test/file.test.ts]`

**Mock Dependencies** (From Tech Spec § 5.5):
- Mock `[DependencyName]` → returns `[MockValue]`
- **Example**: Mock `UserRepository` → returns `mockUser` object
- **Example**: Mock `ApiClient` → returns `mockResponse`

**Test Cases** (Map to Epic's Acceptance Criteria from Tech Spec § 5.5):

- [ ] **Happy Path**: [Test description from Tech Spec]
  - **Given**: [Precondition]
  - **When**: [Action]
  - **Then**: [Expected result]
  - **Maps to Epic**: Acceptance Criterion [#X]

- [ ] **Edge Case**: [Test description from Tech Spec]
  - **Given**: [Edge condition]
  - **When**: [Action]
  - **Then**: [Expected behavior]
  - **Maps to Epic**: Error Flow [#Y]

- [ ] **Error Case**: [Test description]
  - **Given**: [Error condition]
  - **When**: [Action]
  - **Then**: [Error handling]
  - **Maps to Epic**: Error Flow [#Z]

**Integration Tests** (if specified in Tech Spec § 5.5):
- [ ] [Integration test description]

---

### Acceptance Criteria (From Epic via Tech Spec)

Copy relevant acceptance criteria from Epic that this task addresses:

**Scenario**: [Name]
```gherkin
Given [precondition]
When [action]
Then [expected result]
```

---

### Change Summary

**What**: [Brief description of what changes in this task]  
**Example**: "Create NavigationService to handle CRUD operations for navigation items"

**Why**: [Purpose from Epic/Tech Spec]  
**Example**: "Enables admin users to manage site navigation structure"

**How**: [Technical approach]  
**Example**: "Implements RESTful endpoints following NestJS controller pattern"
