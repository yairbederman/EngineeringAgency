---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)
- Manager wants it fixed NOW (systematic is faster than thrashing)

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully**
   - Don't skip past errors or warnings
   - They often contain the exact solution
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce Consistently**
   - Can you trigger it reliably?
   - What are the exact steps?
   - Does it happen every time?
   - If not reproducible → gather more data, don't guess

3. **Check Recent Changes**
   - What changed that could cause this?
   - Git diff, recent commits
   - New dependencies, config changes
   - Environmental differences

4. **Gather Evidence in Multi-Component Systems**
   - Check logs from ALL relevant services
   - Trace requests through the system
   - Look for timing/race conditions
   - Check integration points

### Phase 2: Isolation

1. **Narrow Down the Scope**
   - Binary search through code/commits
   - Identify the smallest reproducible case
   - Remove unrelated components

2. **Create Minimal Reproduction**
   - Strip away everything not needed
   - Isolate the exact trigger condition

### Phase 3: Hypothesis Testing

1. **Form a Theory**
   - Based on evidence, not intuition
   - Be specific about what you expect

2. **Test Minimally**
   - One change at a time
   - Verify your hypothesis directly
   - Don't fix while testing

### Phase 4: Implementation

1. **Write Failing Test First**
   - Proves the bug exists
   - Prevents regression
   - Uses TDD principles

2. **Implement the Fix**
   - Address root cause, not symptoms
   - Minimal necessary change
   - Document why this fixes it

3. **Verify Completely**
   - Original test passes
   - No new regressions
   - Edge cases covered

## Phase Summary Table

| Phase | Actions | Output |
|-------|---------|--------|
| **1. Root Cause** | Read errors, reproduce, check changes | Understand why |
| **2. Isolation** | Narrow scope, create minimal examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## When Process Reveals "No Root Cause"

If systematic investigation reveals issue is truly environmental, timing-dependent, or external:

1. You've completed the process
2. Document what you investigated
3. Implement appropriate handling (retry, timeout, error message)
4. Add monitoring/logging for future investigation

**But:** 95% of "no root cause" cases are incomplete investigation.

## Real-World Impact

From debugging sessions:
- Systematic approach: 15-30 minutes to fix
- Random fixes approach: 2-3 hours of thrashing
- First-time fix rate: 95% vs 40%
- New bugs introduced: Near zero vs common
