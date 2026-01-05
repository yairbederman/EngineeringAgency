# Engineering Agent – Pull Request Mode

> **Persona**: Load `${AGENT_ROOT}/personas/system-architect.md`

## 12. Pull Request Mode – Code to Review

**Goal**: Generate a comprehensive, reviewer-friendly Pull Request with self-review checklist, context, and test evidence.

**Trigger**: After Implementation or BugFix mode completes (Jira status = "In Review")

**Inputs**:
- Completed Task(s) with implementation
- Git branch with commits
- Test results
- Visual verification (for Frontend)

**Outputs**:
- PR Description (Markdown)
- Self-Review Checklist (completed)
- Reviewer Context section
- Test Evidence summary

---

## Phase 0: Pre-flight

### Step 0.1: Gather Context

Execute in **PARALLEL**:
1. Get current branch: `git branch --show-current`
2. Get commit history: `git log main..HEAD --oneline`
3. Get changed files: `git diff main --name-only`
4. Fetch linked Jira issue(s) from branch name or commit messages

### Step 0.2: Classify Change Type

Based on changed files, determine PR type:

| File Patterns | PR Type | Review Focus |
|---------------|---------|--------------|
| `.controller.`, `.service.`, `.repository.`, `migrations/` | **Backend** | API contracts, data integrity, performance |
| `.tsx`, `.vue`, `.css`, `components/` | **Frontend** | Visual accuracy, accessibility, responsiveness |
| Both patterns | **Full-Stack** | Contract alignment, E2E flow |
| `*.test.*`, `__tests__/` only | **Test-Only** | Coverage, edge cases |
| `*.md`, `*.json` (config) | **Docs/Config** | Accuracy, no breaking changes |

---

## Phase 1: Self-Review (MANDATORY)

Before generating PR, complete self-review checklist:

### 1.1 Code Quality Checklist

```markdown
### Self-Review Checklist

#### General
- [ ] All tests pass locally
- [ ] No console.log / debug statements left
- [ ] No commented-out code (unless documented)
- [ ] No hardcoded values that should be config/env
- [ ] Error handling covers edge cases

#### Backend (if applicable)
- [ ] API contract matches Tech Spec
- [ ] Database queries are parameterized (no SQL injection)
- [ ] No N+1 query patterns
- [ ] Logging added for key operations
- [ ] Validation rules enforced

#### Frontend (if applicable)
- [ ] Uses design tokens, not hardcoded values
- [ ] Responsive behavior verified
- [ ] Accessibility: keyboard navigation works
- [ ] Loading/error states handled
- [ ] No layout shifts (CLS)

#### Testing
- [ ] Unit tests cover happy path + edge cases
- [ ] Test names describe expected behavior
- [ ] Mocks are realistic
```

### 1.2 Self-Review Execution

For each checklist item:
1. **Verify** against actual code
2. **Mark** as ✅ or ❌
3. **If ❌**: Either fix before PR or document as known issue

---

## Phase 2: Generate PR Description

### 2.1 PR Template

```markdown
## Summary

[1-2 sentences: What does this PR do and why?]

## Related Issues

- **Jira**: [PROJ-XXX](jira-url) - [Issue Title]
- **Epic**: [PROJ-YYY](epic-url) (if applicable)
- **Tech Spec**: [Link] (if applicable)

## Changes

### [Category 1: e.g., API Changes]
- [Specific change 1]
- [Specific change 2]

### [Category 2: e.g., UI Updates]
- [Specific change 1]

## Screenshots / Recordings

[For Frontend changes - embed before/after screenshots or recordings]

| Before | After |
|--------|-------|
| ![before](path) | ![after](path) |

## Testing

### Automated Tests
- ✅ [X] unit tests added/updated
- ✅ [Y] integration tests added/updated
- Total coverage: [X]%

### Manual Testing
- [ ] Tested locally with [browser/environment]
- [ ] Tested edge cases: [list]

## Self-Review Checklist

[Include completed checklist from Phase 1]

## Reviewer Notes

### Key Areas to Review
1. [Specific file/function that needs careful review]
2. [Complex logic that may need explanation]

### Questions for Reviewers
1. [Any architectural decisions you want feedback on]

## Rollback Plan

[If this PR causes issues, how to safely rollback]
- Revert commit: `git revert [commit-hash]`
- Feature flag: [flag name] (if applicable)
- Database: [No migrations / Migration is reversible / Requires manual intervention]
```

### 2.2 Populate Template

1. **Summary**: Extract from Jira issue description or commit messages
2. **Changes**: Parse `git diff --stat` and group by category
3. **Screenshots**: Reference any captured during Visual Verification
4. **Testing**: Summarize test run results from Implementation mode
5. **Reviewer Notes**: Highlight:
   - Files with most changes (`git diff --stat | head -5`)
   - New patterns introduced
   - Breaking changes (if any)

---

## Phase 3: Breaking Change Detection

### 3.1 API Breaking Changes

IF Backend changes exist:

```
Check for BREAKING changes:
1. Removed endpoints
2. Changed response structure (removed/renamed fields)
3. Changed request validation (stricter rules)
4. Changed HTTP status codes
```

**IF BREAKING**:
```markdown
> [!CAUTION]
> **Breaking Change Detected**
>
> This PR contains breaking API changes:
> - [List specific changes]
>
> **Migration Required**: [Yes/No]
> **Affected Consumers**: [List frontend/services that call this API]
> **Communication**: [Tag relevant teams]
```

### 3.2 Database Breaking Changes

IF migrations exist:

```
Check for BREAKING changes:
1. Column removal
2. Table removal
3. Non-nullable column added without default
4. Index removal
```

**IF BREAKING**:
```markdown
> [!WARNING]
> **Database Migration Warning**
>
> This PR contains database changes that may require:
> - Deployment coordination
> - Data backfill script
> - Rollback plan
```

---

## Phase 4: Generate Reviewer Context

### 4.1 File-Level Annotations

For each significantly changed file, add inline context:

```markdown
### File: `src/services/OrderService.ts`

**Purpose of changes**: [Brief explanation]
**Key logic**: Lines 45-67 contain the new validation
**Test coverage**: `OrderService.test.ts`
```

### 4.2 Sequence Diagram (Complex Changes)

If PR involves multi-step flow, generate sequence diagram:

```
Client          API             Service         Database
   |              |                |                |
   |-- request -->|                |                |
   |              |-- validate --->|                |
   |              |                |-- query ------>|
   |              |                |<-- result -----|
   |              |<-- response ---|                |
   |<-- 200 ------|                |                |
```

---

## Phase 5: Output & Gate

### 5.1 Present PR Draft

Display generated PR description to user.

### 5.2 User Options

```
✅ **Pull Request Draft Ready**

> **⏸️ ACTION REQUIRED**: Please review the PR description. Reply with:
> - `Review` to run AI Code Review (recommended)
> - `Create PR` - Open PR in browser (manual)
> - `Copy` - Copy PR description to clipboard
> - `Revise [feedback]` - Make changes to description
> - `Skip` - Do not create PR now
```

### 5.3 Post-PR Actions (Optional)

After PR is created:
1. **Update Jira**: Add PR link to issue
2. **Notify**: Mention specific reviewers based on file ownership
3. **Label**: Suggest labels based on PR type (breaking, frontend, backend, etc.)

---

## Rules

1. **Never fabricate test results** - Only report what was actually run
2. **Always include Jira link** - Traceability is mandatory
3. **Flag uncertainty** - If unsure about breaking changes, state so
4. **Keep summary concise** - Details go in sections, not summary
5. **Self-review is mandatory** - Do not skip checklist even for small PRs

---

## Quick Reference

### PR Title Format

```
[PROJ-XXX] Short description (50 chars max)
```

Examples:
- `[PROJ-123] Add order confirmation email`
- `[PROJ-456] Fix null crash in payment flow`
- `[PROJ-789] Refactor user service for performance`

### Label Suggestions

| PR Type | Suggested Labels |
|---------|------------------|
| New feature | `feature`, `[area]` |
| Bug fix | `bugfix`, `[severity]` |
| Breaking change | `breaking`, `needs-migration` |
| Refactor | `refactor`, `no-functional-change` |
| Docs only | `docs`, `skip-e2e` |
