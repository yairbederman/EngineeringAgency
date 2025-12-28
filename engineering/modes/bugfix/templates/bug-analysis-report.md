# Bug Analysis Report Template

> **Usage**: Fill this template during BugReport Mode (Step 7).

---

## Bug Summary

**Bug Key**: [PROJ-XXX]
**Title**: [Bug title from Jira]
**Track**: [Frontend / Backend / Full-Stack]
**Severity**: [Critical / High / Medium / Low]

---

## Impact & Scope

**User Impact**:
- [ ] User-facing (customers affected)
- [ ] Internal (team members affected)
- [ ] Silent (data corruption, no visible symptom)

**Scope**:
- Affected feature(s): [List]
- Affected user segment: [All / Specific group]
- Frequency: [Always / Sometimes / Rare conditions]

---

## Reproduction Understanding

### Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

### Current Behavior
[Describe what actually happens]

### Expected Behavior
[Describe what should happen]

### Reproduction Confidence
- [ ] Reproduced locally
- [ ] Reproduced via test
- [ ] Inferred from logs/evidence
- [ ] Cannot reproduce (edge case)

---

## Suspected Root Cause

### Root Cause Analysis

**Primary Hypothesis**:
[Detailed explanation of what you believe is causing the bug]

**Evidence Supporting This Hypothesis**:
1. [Evidence 1]
2. [Evidence 2]

**Alternative Hypotheses Considered**:
1. [Alternative 1] - Ruled out because: [reason]
2. [Alternative 2] - Ruled out because: [reason]

### Affected Code

| File | Line(s) | Issue |
|------|---------|-------|
| [file path] | [lines] | [description] |

---

## Related Evidence

### Logs / Stack Traces
```
[Paste relevant log excerpts]
```

### Screenshots / Network Traces
[Attach or reference evidence files]

### Related Issues
- [PROJ-XXX] - [Relationship: duplicate / similar / related]

---

## Suggested Fix Plan

### Approach
[High-level description of the fix approach]

### Files to Modify

| File | Action | Change Description |
|------|--------|-------------------|
| [file path] | MODIFY / ADD / DELETE | [description] |

### Track-Specific Considerations

**Frontend Track**:
- Component(s) affected: [list]
- Visual regression risk: [High / Medium / Low]
- Design token changes: [Yes / No]

**Backend Track**:
- API contract changes: [Yes / No]
- Database migration needed: [Yes / No]
- Cross-service impact: [Yes / No]

---

## Verification Approach

### Tests to Add/Modify
1. [Test description]

### Manual Verification
1. [Verification step]

### Regression Areas to Check
1. [Area that might be affected by fix]

---

## Assumptions / Missing Info

### Assumptions Made
1. [Assumption 1]

### Information Needed
1. [Missing info that would help]

### Open Questions
1. [Question for QA/PM/User]

---

## Confidence Assessment

| Aspect | Confidence | Notes |
|--------|------------|-------|
| Root cause identification | High / Medium / Low | [reason] |
| Fix approach viability | High / Medium / Low | [reason] |
| Regression risk | High / Medium / Low | [reason] |
