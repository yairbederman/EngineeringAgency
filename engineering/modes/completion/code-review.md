# Engineering Agent – Code Review Mode

> **Persona**: Load `${AGENT_ROOT}/personas/system-architect.md`

## 13. Code Review Mode – Reviewer Perspective

**Goal**: Review code changes from an external reviewer's perspective, catching issues before human review.

**Trigger**: 
- After `PullRequest` mode (AI self-reviews its own work from a different perspective)
- Or manually invoked on any branch/PR

**Inputs**:
- Branch name or PR reference
- Changed files (`git diff main --name-only`)
- Tech Spec (if available)
- Epic acceptance criteria (if available)

**Outputs**:
- Code Review Report with categorized findings
- Suggested improvements
- Approval recommendation

---

## Phase 0: Pre-flight

### Step 0.1: Gather Context

Execute in **PARALLEL**:
1. Get changed files: `git diff main --name-only`
2. Get diff content: `git diff main`
3. Fetch linked Jira issue from branch name
4. Load Tech Spec if referenced in Jira

### Step 0.2: Load Review Standards

Read from `${COPILOT_INSTRUCTIONS_PATH}`:
- Coding standards
- Architectural patterns
- Security requirements
- Performance guidelines

---

## Phase 1: Automated Checks (PARALLEL)

Run all automated checks before manual review:

| Check | Command | Blocking? |
|-------|---------|-----------|
| Type Check | `tsc --noEmit` | ✅ Yes |
| Lint | `eslint .` | ✅ Yes |
| Tests | `npm test` | ✅ Yes |
| Security Scan | `npm audit` (if available) | ⚠️ Warning |
| Bundle Size | (if configured) | ⚠️ Warning |

**IF any blocking check fails** → Report as BLOCKER before continuing

---

## Phase 2: Code Analysis

### 2.1 Architectural Compliance

Review against Tech Spec and `copilot-instructions.md`:

```markdown
### Architectural Review

| Requirement | Status | Evidence |
|-------------|--------|----------|
| Follows [Pattern] from Tech Spec | ✅/❌ | [File:Line] |
| Uses correct service layer | ✅/❌ | [File:Line] |
| Proper separation of concerns | ✅/❌ | [File:Line] |
| No circular dependencies | ✅/❌ | [File:Line] |
```

### 2.2 API Contract Validation (Backend)

IF backend changes exist:

- Compare implementation to Tech Spec API contract
- Verify request validation
- Check response structure
- Validate error handling

```markdown
### API Contract Review

| Endpoint | Tech Spec Match | Issues |
|----------|-----------------|--------|
| `POST /api/...` | ✅/❌ | [Details] |
```

### 2.3 Design Compliance (Frontend)

IF frontend changes exist:

- Compare implementation to Figma tokens (from Task)
- Check component reuse
- Verify responsive behavior
- Check accessibility

```markdown
### Design Compliance Review

| Component | Token Usage | Issues |
|-----------|-------------|--------|
| `ComponentName` | ✅/❌ | [Details] |
```

### 2.4 Test Coverage Analysis

- Are new code paths tested?
- Do tests cover edge cases from Epic?
- Are mocks realistic?

```markdown
### Test Coverage Review

| File Changed | Test Coverage | Missing Tests |
|--------------|---------------|---------------|
| `file.ts` | ✅ 80% | Edge case X |
```

---

## Phase 3: Issue Classification

Categorize all findings by severity:

### Severity Levels

| Level | Description | Action Required |
|-------|-------------|-----------------|
| 🔴 **BLOCKER** | Prevents merge - broken tests, security issues, breaking changes | Must fix |
| 🟠 **MAJOR** | Should fix before merge - architectural violations, missing tests | Strongly recommended |
| 🟡 **MINOR** | Nice to have - code style, optimizations | Optional |
| 💬 **SUGGESTION** | Food for thought - alternative approaches | Informational |

### Issue Template

```markdown
#### 🔴 BLOCKER: [Title]

**Location**: `src/path/to/file.ts:45`
**Category**: [Security | Architecture | Testing | Performance | Style]
**Description**: [What's wrong]
**Suggestion**: [How to fix]

```diff
- // Current code
- const user = await getUser(id);
+ // Suggested fix
+ const user = await getUser(id);
+ if (!user) throw new NotFoundError('User not found');
```
```

---

## Phase 4: Security Review

### 4.1 Security Checklist

- [ ] No hardcoded secrets/credentials
- [ ] Input validation on all user inputs
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (output encoding)
- [ ] CSRF protection (if applicable)
- [ ] Proper authentication/authorization checks
- [ ] Sensitive data not logged
- [ ] Error messages don't leak internal details

### 4.2 Dependency Review

- Check for known vulnerabilities in new dependencies
- Verify dependency necessity (no unused deps)
- Check license compatibility

---

## Phase 5: Performance Review

### 5.1 Backend Performance

- [ ] No N+1 query patterns
- [ ] Appropriate database indexes exist
- [ ] Large queries use pagination
- [ ] Expensive operations are async/cached
- [ ] No blocking operations in request path

### 5.2 Frontend Performance

- [ ] No unnecessary re-renders
- [ ] Large lists virtualized
- [ ] Images optimized
- [ ] Bundle size impact acceptable
- [ ] No memory leaks (event listeners cleaned up)

---

## Phase 6: Generate Review Report

### Report Template

```markdown
# Code Review Report

**Branch**: [branch-name]
**Reviewer**: Engineering Agent (System Architect)
**Date**: [timestamp]
**Jira**: [PROJ-XXX]

## Summary

| Category | Blockers | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| Architecture | X | X | X | X |
| Security | X | X | X | X |
| Performance | X | X | X | X |
| Testing | X | X | X | X |
| **Total** | **X** | **X** | **X** | **X** |

## Recommendation

- 🟢 **APPROVE** - Ready for human review
- 🟡 **APPROVE WITH COMMENTS** - Minor issues to address
- 🔴 **CHANGES REQUESTED** - Must fix blockers before proceeding

## Detailed Findings

[Categorized issues from Phase 3]

## Positive Highlights

- ✅ [Good pattern usage]
- ✅ [Well-tested code]
- ✅ [Clean architecture]
```

---

## Phase 7: Output & Gate

### 7.1 Present Review

Display Code Review Report to user.

### 7.2 User Options

```
📋 **Code Review Complete**

**Result**: [APPROVE / APPROVE WITH COMMENTS / CHANGES REQUESTED]
**Blockers**: [X] | **Major**: [X] | **Minor**: [X]

> **⏸️ NEXT STEP**: Reply with:
> - `Fix [issue]` to address specific finding
> - `Fix all` to address all issues
> - `Proceed anyway` to continue despite issues
> - `Done` to end review
```

### 7.3 Iteration Loop

IF user requests fixes:
1. Switch to Implementation mode (same branch)
2. Address specific issues
3. Return to CodeReview mode
4. Re-run review on fixed code

---

## Rules

1. **Be constructive** - Explain why something is an issue, not just what
2. **Provide alternatives** - Show how to fix, not just what's wrong
3. **Acknowledge good work** - Highlight positive patterns too
4. **Context matters** - Consider trade-offs, don't be dogmatic
5. **Prioritize** - Focus on blockers first, don't overwhelm with minor issues

---

## Integration with Human Review

This mode **precedes** human code review:

```
Implementation → PullRequest → CodeReview → Human Review → Merge
                               ↑
                          We're here
                     (AI pre-review)
```

The goal is to catch issues before human reviewers spend time, not to replace human judgment.
