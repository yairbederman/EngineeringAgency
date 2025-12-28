# Bug Pattern Record

> **Usage**: Complete this template after a successful bug fix (BugFix Mode Step 12) to document patterns for preventing similar bugs.
> **Purpose**: Build organizational knowledge to catch similar bugs in code review or automated checks.

---

## Bug Reference

**Bug Key**: [PROJ-XXX]
**Title**: [Bug title]
**Fixed Date**: [Date]

---

## Pattern Classification

### Category

Select the most appropriate category:

- [ ] **Null/Undefined Handling** – Missing null checks, optional chaining
- [ ] **Async/Concurrency** – Race conditions, missing await, state timing
- [ ] **Type Mismatch** – Wrong type assumptions, serialization issues
- [ ] **Validation Gap** – Missing input validation, boundary conditions
- [ ] **Error Boundary Gap** – Unhandled exceptions, missing error states
- [ ] **State Management** – Stale state, incorrect updates, cache invalidation
- [ ] **API Contract** – Request/response mismatch, schema drift
- [ ] **Database/Query** – N+1 queries, missing indexes, transaction issues
- [ ] **Configuration** – Environment mismatches, missing config values
- [ ] **Security** – Input sanitization, auth bypass, data exposure
- [ ] **Other** – [Describe category]

### Severity

- [ ] **Critical** – Data loss, security vulnerability, system crash
- [ ] **High** – Major feature broken, significant user impact
- [ ] **Medium** – Feature partially broken, workarounds exist
- [ ] **Low** – Minor inconvenience, cosmetic issue

---

## Pattern Description

### Summary
[One-line description of the bug pattern]

**Example**: "Async handler missing null check on optional API response field"

### Detailed Pattern

**What happened**:
[Describe the specific bug in detail]

**Why it happened**:
[Root cause analysis – what coding practice or oversight led to this]

**What masked it**:
[Why wasn't this caught earlier – testing gaps, edge case, etc.]

---

## Prevention Strategies

### Code-Level Prevention

**Fix pattern to apply**:
```typescript
// BAD (bug pattern)
const value = response.data.field;

// GOOD (prevention pattern)
const value = response.data?.field ?? defaultValue;
```

**Code review checklist item**:
- [ ] [Specific thing to check in code reviews]

### Automated Prevention

**Lint rule** (if applicable):
- Rule name: [e.g., `@typescript-eslint/no-unsafe-member-access`]
- Configuration: [how to enable]

**Type-level prevention** (if applicable):
- [ ] Stricter type definitions
- [ ] Required vs optional field review

**Test pattern** (if applicable):
```typescript
// Test to add for similar functionality
it('handles [edge case] gracefully', () => {
  // Test setup for edge case
});
```

---

## Similar Code Locations

### Files That May Have Same Pattern

| File | Location | Risk Level | Investigated? |
|------|----------|------------|---------------|
| [file path] | [function/method] | High / Medium / Low | [YES / NO] |
| [file path] | [function/method] | High / Medium / Low | [YES / NO] |

### Recommended Follow-up

- [ ] Search codebase for similar patterns
- [ ] Create follow-up Jira ticket for remediation
- [ ] Add lint rule to prevent future occurrences
- [ ] Update coding guidelines

---

## Learning & Documentation

### Team Knowledge Share

- [ ] Pattern added to team wiki/Confluence
- [ ] Discussed in team sync/retrospective
- [ ] Added to onboarding documentation

### Integration with Dev Process

- [ ] Added to code review checklist
- [ ] Added to PR template reminders
- [ ] Automated check implemented (lint/test)

---

## Metadata

**Documented by**: [Agent/Developer]
**Fix Branch**: `bugfix/[BugKey]-[summary]`
**Related PRs**: [PR links]
