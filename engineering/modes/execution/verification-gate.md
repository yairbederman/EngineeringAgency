# Live Verification Gate Protocol

> **Load When**: After code commit, before marking task "In Review"
> **Purpose**: Ensure implementation is verified via browser/MCP before task completion

---

## Scope Detection (MANDATORY)

Analyze changed files to determine verification type:

```bash
git diff --name-only HEAD~1
```

| File Pattern | Verification Type | Auto-Approve |
|--------------|-------------------|--------------|
| `*.md`, `README*`, `docs/*`, `*.txt` | DOC_ONLY | ✅ Yes |
| `*.env*`, `config/*`, `*.json` (non-package) | BUILD_CHECK | ⚠️ Build pass only |
| `*.controller.*`, `*.service.*`, `*.api.*`, `*.route.*` | API_TEST | ❌ No |
| `*.tsx`, `*.vue`, `*.jsx`, `*.css`, `*.scss`, `components/*` | BROWSER_VISUAL | ❌ No |
| Mixed frontend + backend | FULL_STACK | ❌ No |

**Detection Logic**:
```
IF all files match DOC_ONLY patterns → Type = DOC_ONLY
ELSE IF any file matches BROWSER_VISUAL patterns → Type = BROWSER_VISUAL (or FULL_STACK if API patterns also present)
ELSE IF any file matches API_TEST patterns → Type = API_TEST
ELSE → Type = BUILD_CHECK
```

---

## Tool Selection Protocol

### Priority Order

| Priority | Tool | Best For | Timeout |
|----------|------|----------|---------|
| 1 | `browser_subagent` | Visual verification, UI testing | 60s |
| 2 | MCP tools (Figma, API) | Design comparison, API validation | 30s |
| 3 | Build verification | Minimum bar for all changes | 120s |
| 4 | Manual fallback | When tools unavailable | N/A |

### Tool Availability Check

```
1. Check if browser_subagent is available
2. Check for relevant MCP tools (Figma, Postman, etc.)
3. Verify build commands exist in package.json
4. Prepare manual fallback instructions
```

---

## Verification Execution

### DOC_ONLY Verification
```
✅ Auto-approve: Documentation changes only
- No code impact
- Skip to completion
```

### BUILD_CHECK Verification
```bash
npm run build  # or project equivalent
npm run test   # if test script exists
```
- **Pass**: Build succeeds with exit code 0
- **Fail**: Present error, enter retry loop

### API_TEST Verification

1. **Identify Endpoints**: Parse changed controller/route files
2. **Execute Tests**: 
   - Run existing API tests: `npm run test:api`
   - Or use MCP/curl for endpoint validation
3. **Validate Response**:
   - Status code matches expected
   - Response schema matches contract
   - Error cases handled

**Checklist**:
- [ ] Endpoint returns expected status code
- [ ] Response schema matches contract
- [ ] Error cases handled (400, 401, 404, 500)
- [ ] Performance: Response < 500ms

### BROWSER_VISUAL Verification

1. **Start Dev Server** (if not running):
   ```bash
   npm run dev  # or detected dev command
   ```

2. **Browser Verification**:
   ```
   browser_subagent:
     Task: "Navigate to [component URL], verify rendering, capture screenshots of:
            1. Default state
            2. Hover/interactive states
            3. Mobile viewport (375px width)
            Return any console errors observed."
     RecordingName: "[taskkey]_verification"
   ```

3. **Evidence Capture**:
   - Screenshots: `[TaskKey]_impl_default.png`, `[TaskKey]_impl_mobile.png`
   - Recordings: `[TaskKey]_verification.webp`
   - Console logs: Any errors captured

**Checklist**:
- [ ] Component renders without errors
- [ ] All interactive states work (hover, click, focus)
- [ ] Responsive: Desktop, Tablet (768px), Mobile (375px)
- [ ] No console errors
- [ ] Loading states display correctly

### FULL_STACK Verification

Execute in sequence:
1. API_TEST verification
2. BROWSER_VISUAL verification
3. Integration check (frontend uses correct API responses)

---

## Evidence Storage

Store verification artifacts in project:

```
.verification/
└── [TaskKey]/
    ├── screenshots/
    │   ├── default.png
    │   ├── hover.png
    │   └── mobile.png
    ├── recordings/
    │   └── verification.webp
    ├── logs/
    │   ├── console.json
    │   └── test-results.json
    └── api/
        └── responses.json
```

**Retention**: 30 days or until Epic closed (whichever is sooner)

---

## Retry Logic

| Attempt | Action | Timeout |
|---------|--------|---------|
| 1 | Standard verification | 60s |
| 2 | Extended timeout, retry | 90s |
| 3 | User escalation | N/A |

**On Retry**:
- Preserve previous attempt evidence
- Log retry reason
- Reset timeout counter

### Session-Persistent Retry Tracking (MANDATORY)

> **Purpose**: Prevent infinite retry loops across session boundaries.

**Retry Log File**: `.verification/[TaskKey]/retry-log.json`

```json
{
  "taskKey": "[TaskKey]",
  "startedAt": "[ISO timestamp]",
  "attempts": [
    {
      "attemptNumber": 1,
      "timestamp": "[ISO timestamp]",
      "result": "FAIL | TIMEOUT | PASS",
      "reason": "[failure reason or null]",
      "evidenceFiles": ["[relative paths]"]
    }
  ],
  "currentAttempt": 1,
  "maxAttempts": 3,
  "status": "IN_PROGRESS | PASSED | ESCALATED | SKIPPED"
}
```

**Session Resume Protocol**:
1. On verification start, check for existing `retry-log.json`
2. If exists AND `status = IN_PROGRESS`:
   - Read `currentAttempt` value
   - If `currentAttempt >= maxAttempts` → Immediately escalate to user
   - Else → Increment and continue from last attempt
3. If not exists → Create new log with `attemptNumber: 1`

**FORBIDDEN**: Starting fresh without checking existing retry log.

---

## Timeout Handling

**On Timeout**:
1. Save partial evidence collected
2. Present fallback options:
   ```
   ⚠️ **Verification Timeout**
   
   Partial evidence captured. Options:
   A) `Retry` - Try again with extended timeout
   B) `Manual` - Provide URL for manual verification
   C) `Skip [reason]` - Bypass with justification (logged)
   ```
3. **NEVER auto-skip** - Always require user decision

---

## Approval Format

### Successful Verification

```markdown
✅ **Live Verification Complete**

| Attribute | Value |
|-----------|-------|
| **Type** | [BROWSER_VISUAL / API_TEST / BUILD_CHECK] |
| **Checklist** | [X/Y] items passed |
| **Evidence** | [Link to .verification/TaskKey/] |

**Screenshots/Recordings**:
[Embedded evidence carousel]

> **⏸️ APPROVAL REQUIRED**: 
> - `Approve` → Mark task complete, proceed to PR
> - `Retry` → Re-run verification
> - `Skip [reason]` → Bypass with logged justification
```

### Failed Verification

```markdown
❌ **Verification Failed**

| Issue | Details |
|-------|---------|
| **Failed Check** | [Specific checklist item] |
| **Error** | [Error message or screenshot] |

**Attempted Approaches**: [List of attempts]

> **Options**:
> - `Fix` → Return to implementation to address issue
> - `Retry` → Re-run verification after fix
> - `Skip [reason]` → Bypass with logged justification (adds `unverified` label)
```

---

## Skip Handling

When user provides `Skip [reason]`:

1. **Log Justification**: Add comment to task via storage adapter:
   ```
   storage.addTaskComment(taskId, `
   ⚠️ **Verification Skipped**
   - **Reason**: [user-provided reason]
   - **Skipped at**: [timestamp]
   - **Scope**: [verification type]
   `)
   ```

2. **Add Label**: Apply `unverified` label via storage adapter:
   ```
   storage.addLabel(taskId, 'unverified')
   ```
   > Backend-specific: Jira adds issue label, Local adds to registry.

3. **Continue**: Proceed to completion with warning in PR description

---

## CI/CD Integration

For automated pipelines:

```yaml
verification:
  mode: headless
  browser: puppeteer  # or playwright
  timeout: 120s
  auto_approve: false  # Attach evidence to PR for reviewer
  
  on_failure:
    action: block_pr
    notify: [reviewers]
```

When `auto_approve: false`:
- Verification runs automatically
- Evidence attached to PR
- Human reviewer approves verification as part of code review
