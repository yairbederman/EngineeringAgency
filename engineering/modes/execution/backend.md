# Engineering Agent – Backend Implementation Track

> **When to use**: Task has labels `backend`, `api`, `service`, or description mentions API/database/service changes.

## Backend-Specific Pre-flight (Phase 0B)

### API Contract Validation

1. **Check Task Description** for:
   - API Contract section with endpoint, method, request/response schemas
   - OpenAPI/Swagger reference link
   - Database schema changes (if applicable)

2. **If Missing**:
   ```
   STOP. Return to TaskPlanning mode.
   "This backend task requires an API Contract. Please add:
   - Endpoint path and HTTP method
   - Request schema (body/params)
   - Response schema with status codes
   - Error scenarios"
   ```

3. **If Present**: Extract and validate:
   - Endpoint matches project routing conventions
   - Request/response types are defined or need creation
   - Database entities exist or need migration

### Database Migration Check

IF task mentions schema changes:
1. Verify migration strategy is documented
2. Check for rollback plan
3. Flag if production data transformation needed

---

## Backend Context Loading (Step 3B)

Load in PARALLEL:
- `${COPILOT_INSTRUCTIONS_PATH}` → Backend section
- Target service/controller files
- Related repository/DAO files
- Existing test files for the module
- `${SYSTEM_ARCH_OUTPUT}/cross-service-apis.json` (if cross-service)

**Skip**: Figma references, UI guides, visual verification

---

## Backend TDD Loop (Step 4B)

### Test Categories (Priority Order)

1. **Unit Tests** (MANDATORY):
   - Service method logic
   - Validation rules
   - Business rule edge cases
   - Error handling paths

2. **Integration Tests** (if infra exists):
   - Repository/DAO with test database
   - External service mocks
   - Cache behavior

3. **API Tests** (if controller touched):
   - Request validation (400 errors)
   - Success responses (200/201)
   - Error responses (404, 500)
   - Auth/permission checks (401, 403)

### Test Naming Convention

```
describe('[ServiceName]', () => {
  describe('[methodName]', () => {
    it('should [expected behavior] when [condition]', () => {});
    it('should throw [ErrorType] when [invalid condition]', () => {});
  });
});
```

---

## Backend Implementation (Step 5B)

### Implementation Order

1. **Types/Interfaces** – Define request/response DTOs
2. **Validation** – Input validation rules
3. **Repository/DAO** – Data access layer
4. **Service** – Business logic
5. **Controller** – HTTP layer
6. **Error Handling** – Custom exceptions

### Code Quality Checks

Before committing:
- [ ] No hardcoded values (use config/env)
- [ ] All database queries are parameterized
- [ ] Error messages are user-safe (no stack traces)
- [ ] Logging added for key operations
- [ ] No N+1 query patterns

---

## Backend Verification (Step 6B)

### Static Analysis Gate

Run in sequence:
1. **Type Check**: `tsc --noEmit` (TypeScript) or equivalent
2. **Lint Check**: Project linter (ESLint, etc.)
3. **Security Scan**: If available (npm audit, etc.)

### Test Execution

```bash
# Run unit tests for affected module
npm test -- --testPathPattern="[module-name]"

# Run integration tests if applicable
npm run test:integration -- --testPathPattern="[module-name]"
```

### API Contract Validation

IF OpenAPI spec exists:
1. Generate types from spec
2. Compare implementation against spec
3. Flag any mismatches

---

## Backend Performance Considerations

Check before completion:
- [ ] Database queries have appropriate indexes
- [ ] No N+1 query patterns in loops
- [ ] Large queries use pagination
- [ ] Cacheable data uses cache layer
- [ ] Background jobs for heavy operations

---

## Backend Completion (Phase 3B)

1. **Run Related Tests**: All tests in module + integration
2. **Commit**: `[TaskKey] Implement [summary]`
3. **Jira Update**: Transition to "In Review"
4. **Jira Comment**: 
   ```
   ### ✅ Backend Implementation Complete
   
   **Branch**: feature/[TaskKey]-[summary]
   **Tests**: [X] unit, [Y] integration passed
   **API**: [endpoint] implemented per contract
   
   Ready for code review.
   ```
