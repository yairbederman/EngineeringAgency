# Root Cause Validation Checklist

> **Usage**: Complete this checklist during BugFix Mode (Step 5) before implementing any fix.
> **Rule**: If ANY item is marked NO, return to code inspection. Do NOT proceed with uncertain fixes.

---

## Bug Reference

**Bug Key**: [PROJ-XXX]
**Suspected Root Cause**: [Brief description]

---

## Validation Checklist

### 1. Reproducibility

- [ ] **Can I reproduce this bug with a failing test?**
  - Test file: [path]
  - Test name: [name]
  - Test fails: [YES / NO]
  
  > If NO: The test may not accurately capture the bug. Refine the test before proceeding.

---

### 2. Symptom Coverage

For each symptom reported in the bug:

| Symptom | Explained by Root Cause? | Explanation |
|---------|--------------------------|-------------|
| [Symptom 1 from bug report] | [YES / NO] | [How root cause explains this] |
| [Symptom 2 from bug report] | [YES / NO] | [How root cause explains this] |
| [Symptom 3 from bug report] | [YES / NO] | [How root cause explains this] |

- [ ] **Does my root cause explain ALL symptoms?**

  > If NO for any symptom: The root cause may be incomplete or incorrect. Investigate further.

---

### 3. Alternative Causes Ruled Out

List at least 2 alternative explanations and why they were ruled out:

| Alternative Cause | Ruled Out Because |
|-------------------|-------------------|
| [Alternative 1] | [Evidence/reasoning] |
| [Alternative 2] | [Evidence/reasoning] |

- [ ] **Have I ruled out at least 2 alternative causes?**

  > If NO: You may be jumping to conclusions. Consider other possibilities.

---

### 4. Root Cause vs Symptom Check

- [ ] **Am I fixing THE root cause, not A symptom?**

  **Test**: If I fix this, will the bug be PERMANENTLY resolved, or just masked?
  
  - [ ] Fix addresses the origin of the problem
  - [ ] Fix is not just a workaround (e.g., null check that hides underlying data issue)
  - [ ] No upstream issue that will cause this to recur

  > If treating a symptom: Trace back further to find the actual root cause.

---

### 5. Fix Approach Confidence

- [ ] **Am I confident this fix will work?**

  **Evidence for confidence**:
  1. [Evidence 1]
  2. [Evidence 2]

  **Remaining uncertainty**:
  - [Any remaining concerns]

---

## Validation Result

### All Checks Passed?

- [ ] **YES** → Proceed to Step 6 (Test Failure / Reproduction)
- [ ] **NO** → Return to Step 3 (Inspect) and gather more information

---

## Checkpoint

**Validated by**: [Agent session]
**Validated at**: [Timestamp]
**Confidence Level**: [High / Medium / Low]

> **Note**: If confidence is Low, consider escalating to user before proceeding with fix.
