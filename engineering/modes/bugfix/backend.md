# Engineering Agent – BugFix Backend Track

> **When to use**: Bug has API errors (4xx, 5xx), database issues, service failures, or affects `.controller.`/`.service.`/`.repository.` files.

## Context Loading (Backend-Specific)

**Load in PARALLEL**:
- `${COPILOT_INSTRUCTIONS_PATH}` → Backend section
- Target service/controller files
- Related repository/DAO files
- Existing test files for the module
- `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json` (if cross-service)
- `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json` (if cross-service)

**SKIP** (Token Optimization):
- Figma references
- `${DESIGN_TOKENS_PATH}`
- UI component files
- Visual verification steps

---

## Step 1B: Backend Layer Analysis (BugReport Phase)

### Error Evidence Collection

1. **Log Analysis**:
   - Parse stack traces
   - Identify error type and origin
   - Check for preceding warning logs

2. **API Response Inspection**:
   - Actual response vs expected response
   - HTTP status codes
   - Error message content

3. **Data Flow Tracing**:
   - Where does the bug's data originate?
   - Which service(s) touch this data?
   - Where is the data consumed?

### Codebase Inspection (Backend-Specific)

Use direct file inspection:
1. `view_file` on service/controller file to understand implementation
2. `grep_search` for error handling patterns related to the error type
3. Search test files for existing coverage and similar bugs

### Layer Mapping

Map bug to specific backend layer using `${FILE_CATEGORIZATION_PATH}`:
- `controllers` – HTTP layer
- `services` – Business logic
- `repositories` / `DAOs` – Data access
- `migrations` – Schema changes
- `utils` / `helpers` – Shared utilities

### Cross-Repository Check (Multi-Workspace)

IF bug involves API calls between services:

1. **Read Service Topology**:
   - `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json`
   - Identify `callsServices` and `calledBy` for affected service

2. **Check API Contracts**:
   - `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json`
   - Verify expected request/response schemas

3. **Determine Bug Location**:
   - Is issue in client (caller) or server (callee)?
   - Use network traces to identify which layer returns incorrect data

---

## Step 2B: Backend Code Inspection (BugFix Phase)

### Service/Controller Inspection

Use direct inspection:
1. `view_file` on service/controller file(s)
2. `view_code_item` for specific method implementations
3. `grep_search` for validation and error handling patterns

### Data Layer Analysis

- Check repository/DAO queries
- Verify database schema matches expectations
- Look for N+1 query patterns or missing indexes
- Check for transaction boundaries

### API Contract Analysis

IF bug involves API:
- Check request validation rules
- Verify response serialization
- Check for null/undefined handling in response
- Validate error response format

### Race Condition / Concurrency Analysis (If timing suspected)

**Symptoms suggesting race condition**:
- Bug appears intermittently under load
- Bug depends on request timing or order
- Bug involves concurrent database operations
- Bug shows data inconsistency between reads

**Inspection checklist**:
- [ ] Are there concurrent writes to the same resource?
- [ ] Is there missing transaction isolation for multi-step operations?
- [ ] Are there optimistic updates without proper conflict resolution?
- [ ] Is there a missing lock/mutex for shared resource access?
- [ ] Are cache invalidations racing with updates?

**Common patterns to check**:
```typescript
// BAD: Read-modify-write race condition
const user = await repo.findOne(id);
user.balance += amount;
await repo.save(user);

// GOOD: Atomic update
await repo.update(id, { $inc: { balance: amount } });

// GOOD: Pessimistic lock
await repo.findOne(id, { lock: { mode: 'pessimistic_write' } });
```

**Database-specific checks**:
- [ ] For INSERT: Is there a unique constraint race? Use `INSERT ... ON CONFLICT`
- [ ] For UPDATE: Is there a lost-update scenario? Use optimistic locking (version field)
- [ ] For DELETE: Are dependent entities handled atomically?

---

## Step 3B: API Verification (BugFix Phase)

> **Purpose**: Verify fix resolves API issue without breaking contract.

### Step 3B.1: Document Original Behavior

Before applying fix:
```markdown
**Original Behavior**
- Endpoint: [method] [path]
- Request: [sample request]
- Response: [buggy response]
- Status: [status code]
```

### Step 3B.2: Apply Fix and Verify

After implementing fix:
```markdown
**Fixed Behavior**
- Endpoint: [method] [path]
- Request: [sample request]
- Response: [fixed response]
- Status: [status code]
```

### Step 3B.3: Contract Integrity Check

Compare before/after:

| Field | Before Fix | After Fix | Change Type |
|-------|------------|-----------|-------------|
| Status code | [X] | [Y] | Breaking/Non-breaking |
| Response fields | [list] | [list] | Added/Removed/Modified |
| Error format | [format] | [format] | Breaking/Non-breaking |

**IF BREAKING change**:
- Document reason for breaking change
- Identify affected consumers (frontends, other services)
- Plan notification/migration

### Step 3B.4: Response Validation Checklist

After applying fix, verify:
- [ ] Correct HTTP status code returned
- [ ] Response body matches expected schema
- [ ] Error responses follow project error format
- [ ] No sensitive data leaked in error messages
- [ ] Response headers are correct (Content-Type, etc.)
- [ ] Null/undefined values handled appropriately

---

## Backend Test Categories

### Unit Tests (MANDATORY)

```typescript
describe('[ServiceName]', () => {
  describe('[methodName] - bug fix: [BugKey]', () => {
    it('should [expected behavior after fix]', () => {});
    it('should not [buggy behavior]', () => {});
  });
  
  describe('regression prevention', () => {
    it('maintains [related functionality]', () => {});
  });
});
```

### Integration Tests (If database/external service involved)

- Repository/DAO with test database
- External service mocks
- Cache behavior
- Transaction rollback scenarios

### API Tests (If controller touched)

- Request validation (400 errors)
- Success responses (200/201)
- Error responses (404, 500)
- Auth/permission checks (401, 403)

### Test Execution

Use test commands from `${COPILOT_INSTRUCTIONS_PATH}` → Testing section.

**Stack-Specific Defaults** (fallback if not defined in config):

| Stack | Unit Tests | Integration Tests |
|-------|------------|-------------------|
| Node/TS | `npm test -- --testPathPattern="[module]"` | `npm run test:integration` |
| Java | `mvn test -Dtest=[TestClass]` | `mvn verify -Pit` |
| Python | `pytest -k "[module]"` | `pytest -m integration` |
| Go | `go test ./... -run [Test]` | `go test -tags=integration ./...` |
| .NET | `dotnet test --filter [module]` | `dotnet test --filter Category=Integration` |

---

## Backend Performance Considerations

Check before completion:
- [ ] Database queries have appropriate indexes
- [ ] No N+1 query patterns in fix
- [ ] Large queries use pagination
- [ ] Cacheable data uses cache layer
- [ ] No new blocking operations in request path

---

## Backend Completion

1. **Run Related Tests**: All tests in module + integration
2. **API Verification**: Contract check completed
3. **Commit**: `[BugKey] Fix [description]`
4. **Jira Comment**:
   ```
   ### ✅ Backend Bug Fix Complete
   
   **Branch**: bugfix/[BugKey]-[summary]
   **Track**: Backend
   **Tests**: [X] unit, [Y] integration passed
   **API**: [endpoint] verified
   - **Evidence**: `curl` output or test result file

   
   **Fix Summary**: [Brief description of what was fixed]
   **Root Cause**: [What caused the bug]
   **Contract Changes**: [None / List breaking changes]
   
   Ready for code review.
   ```

---

## Backend-Specific Rules

- No hardcoded values (use config/env)
- All database queries must be parameterized
- Error messages must be user-safe (no stack traces to clients)
- Add logging for key operations touched by fix
- If fix touches shared code, verify all callers
- Do not change API contracts unless explicitly required by fix
