---
template: task-backend
version: 1.0.0
contract: _template-contracts.md#task-backend-template-contract
used_by: TaskPlanning
required_sections:
  - header
  - metadata
  - target_files
  - dependencies
  - pattern_context
  - implementation
  - steps
  - test_plan
  - acceptance
  - summary
conditional_sections:
  - validation_rules: "If API/Entity task"
  - cross_service: "If calling other services"
  - before_after: "If MODIFY action"
---

## Task: [Name]

**Layer**: [Database | Service | API]  
**Category**: [e.g., `migrations`, `entities`, `services`, `controllers`]  
**Type**: Backend  
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

**Examples**:
- Task 1: Create migration for navigation_items table (Database layer)
- Task 2: Create NavigationEntity.ts (Database layer)

OR

- None (this is a Database layer task)

---

### Pattern Context (From Tech Spec § 2)

**Pattern to Follow**: [Architecture pattern from Tech Spec]  
**Example**: "Following ${PROJECT_CMS_API} NestJS service pattern (§ 3.2 from .ai-instructions)"

**Reference File**: [Existing file to mimic]  
**Example**: `src/services/UserService.ts`

**Rationale**: [Why this pattern/file is being reused]

---

### Implementation Context

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
- `field1`: Required, max 255 chars
- `field2`: Required, positive integer

**Cross-Service Dependencies** (if calling other services):
| Service Called | Endpoint | Purpose |
|----------------|----------|---------|
| ${PROJECT_DATA_API} | GET /api/v1/resource | Fetch resource data |

---

### Implementation Steps

> Steps are **layer-specific**. Follow the sequence for your task's layer.

#### Database Layer:
1. Review existing schema (if MODIFY action)
2. Create migration file with proper timestamp naming
3. Update entity definition with new fields/constraints
4. Test migration (up/down) locally
5. Verify entity changes with database inspection

#### Service Layer:
1. Review similar service from pattern reference (Tech Spec § 2)
2. Implement business logic methods following pattern
3. Add validation logic per Tech Spec constraints
4. Write unit tests with mocks (from Test Plan below)
5. Verify service behavior with test coverage

#### API Layer:
1. Review similar controller from pattern reference
2. Implement endpoint following API contract (Tech Spec § 5.3)
3. Add request validation middleware
4. Add error handling per Epic's error flows
5. Write integration tests for endpoint

---

### Test Plan (From Tech Spec § 5.5)

**Test File**: `[path/to/test/file.test.ts]`

**Mock Dependencies**:
- Mock `[DependencyName]` → returns `[MockValue]`
- **Example**: Mock `UserRepository` → returns `mockUser` object

**Test Cases** (Map to Epic's Acceptance Criteria):

- [ ] **Happy Path**: [Test description]
  - **Given**: [Precondition]
  - **When**: [Action]
  - **Then**: [Expected result]
  - **Maps to Epic**: Acceptance Criterion [#X]

- [ ] **Edge Case**: [Test description]
  - **Given**: [Edge condition]
  - **When**: [Action]
  - **Then**: [Expected behavior]
  - **Maps to Epic**: Error Flow [#Y]

- [ ] **Error Case**: [Test description]
  - **Given**: [Error condition]
  - **When**: [Action]
  - **Then**: [Error handling]
  - **Maps to Epic**: Error Flow [#Z]

**Integration Tests** (if applicable):
- [ ] [Integration test description]

---

### Acceptance Criteria (From Epic)

**Scenario**: [Name]
```gherkin
Given [precondition]
When [action]
Then [expected result]
```

---

### Change Summary

**What**: [Brief description of what changes]  
**Why**: [Purpose from Epic/Tech Spec]  
**How**: [Technical approach]
