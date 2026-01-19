---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for the codebase and questionable taste. Document everything they need to know: which files to touch, testing approach, verification steps.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

## Plan Document Header

Every plan MUST start with this header:

```markdown
# [Feature Name] Implementation Plan

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**

1. "Write the failing test" — step
2. "Run it to make sure it fails" — step  
3. "Implement the minimal code to make the test pass" — step
4. "Run the tests and make sure they pass" — step
5. "Commit" — step

## Task Structure

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

[code block with test]

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

[code block with implementation]

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

`git commit -m "feat: add specific feature"`
```

## Key Principles

| Principle | Application |
|-----------|-------------|
| **DRY** | Don't repeat code—extract shared logic |
| **YAGNI** | Don't build features you don't need yet |
| **TDD** | Every task follows Red-Green-Refactor |
| **Frequent commits** | One commit per task |

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete. Two execution options:**

1. **This session** — Execute tasks with review between each
2. **Parallel session** — Batch execution with checkpoints

**Which approach?"**

## Common Mistakes

- Vague file paths ("in the utils folder")
- Large tasks that take > 10 minutes
- Skipping test steps
- Not specifying expected test output
- Leaving "TODO" placeholders in code snippets
