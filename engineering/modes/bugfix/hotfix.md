# Engineering Agent – Hotfix Mode

> **Persona**: Load based on issue type:
> - Backend issue → `${AGENT_ROOT}/personas/backend-developer.md`
> - Frontend issue → `${AGENT_ROOT}/personas/frontend-developer.md`

## Purpose

Enable **expedited** fixes for production incidents. This mode bypasses BugReport analysis when production is down and time is critical.

> [!CAUTION]
> **Hotfix mode is for PRODUCTION EMERGENCIES ONLY.** Use standard BugFix for all other bugs.

---

## When to Use

| Condition | Use Hotfix | Use Standard BugFix |
|-----------|------------|---------------------|
| Production is down | ✅ | ❌ |
| P0/Critical severity | ✅ | ❌ |
| Customer-facing incident | ✅ | ❌ |
| Bug found in testing | ❌ | ✅ |
| Minor UI issue | ❌ | ✅ |
| Performance degradation (not outage) | ❌ | ✅ |

---

## Eligibility Criteria

ALL must be true:

| # | Criterion | How to Verify |
|---|-----------|---------------|
| 1 | **Production Incident** | User confirms "production is down" or "P0" |
| 2 | **Jira Issue Exists** | Issue key provided with `Priority = Critical` or `P0` |
| 3 | **Known Repro** | User provides minimal reproduction steps |
| 4 | **Single Root Cause** | Issue is narrowed to specific behavior |

---

## Hotfix Flow (Streamlined)

### Phase 0: Emergency Context (2-3 min)

```
1. Fetch task via storage protocol: `getTask(taskId)`
2. Extract:
   - Summary and description
   - Affected component/service
   - User-provided repro steps
3. Load ${COPILOT_INSTRUCTIONS_PATH} (affected project only)
4. Skip: Full bug analysis, evidence gathering, pattern documentation
```

### Phase 1: Quick Root Cause (5-10 min)

**Minimal Investigation:**
1. Read the affected file(s) from user description
2. Identify the specific code path causing the issue
3. Formulate hypothesis (1 sentence)

**Validation Checklist (Abbreviated):**
- [ ] Can I explain the symptom with this root cause? (Y/N)
- [ ] Does fixing this have obvious side effects? (Y/N)

**IF "No" to first question** → Escalate, do not proceed.  
**IF "Yes" to second question** → Warn user, document side effect risk, proceed with caution.

### Phase 2: Emergency Fix (10-20 min)

#### Step 2.1: Branch
```bash
git checkout -b hotfix/[IssueKey]-emergency
```

#### Step 2.2: Write Minimal Test
- Write ONE failing test that reproduces the issue
- Run test → Must FAIL (proves bug exists)

#### Step 2.3: Implement Fix
- Apply minimal change to pass the test
- **FORBIDDEN**: Refactoring, cleanup, "while we're here" changes
- **REQUIRED**: Explain fix in 1-2 sentences

#### Step 2.4: Quick Verification
1. Run the reproduction test → Must PASS
2. Run tests in affected file → All must PASS
3. **Skip**: Full regression suite (too slow for emergency)

### Phase 3: Emergency Completion (2-5 min)

#### Step 3.1: Commit
```bash
git commit -m "[IssueKey][HOTFIX] <short description>"
```

#### Step 3.2: Status Update
```
updateTaskStatus(taskId, "in-review")
addTaskComment(taskId, comment) →
   "🚨 HOTFIX APPLIED
   - Branch: hotfix/[IssueKey]-emergency
   - Fix: [1-sentence explanation]
   - Test: [test file and method]
   - Full regression: PENDING (not run in emergency mode)"
```

#### Step 3.3: Present Completion
```
🚨 **HOTFIX Complete**
- **Issue**: [IssueKey] - [Summary]
- **Branch**: hotfix/[IssueKey]-emergency
- **Minimal test**: ✅ Passing
- **Full regression**: ⏳ Pending (run after deploy)

> **⏸️ NEXT STEP**: Reply with:
> - `Push` to push branch immediately
> - `Create PR` to generate expedited PR
> - `Rollback` if fix doesn't work
```

---

## Post-Hotfix Requirements (MANDATORY)

> [!IMPORTANT]
> After production is stable, these items MUST be completed:

| Item | Owner | Deadline |
|------|-------|----------|
| Full regression test run | Developer | Within 24 hours |
| Bug pattern documentation | Developer | Within 48 hours |
| Root cause analysis (full) | Developer | Within 1 week |
| Prevention action item | Tech Lead | Within 1 sprint |

---

## Rollback Protocol

If hotfix makes things worse:

```bash
# Immediate rollback
git revert HEAD --no-edit
git push origin hotfix/[IssueKey]-emergency

# Or if deployed, trigger rollback in CI/CD
```

**Jira Comment:**
```
❌ HOTFIX ROLLED BACK
- Reason: [why it failed]
- Status: Reverted to previous state
- Next: [escalation path]
```

---

## Comparison: Hotfix vs Standard BugFix

| Aspect | Hotfix | Standard BugFix |
|--------|--------|-----------------|
| **Duration** | 15-30 min | 1-2 hours |
| **Root Cause Analysis** | Minimal (1 question) | Full checklist (5 questions) |
| **Test Coverage** | Single repro test | Full regression |
| **Documentation** | Deferred | Inline |
| **Pattern Recording** | Post-incident | During fix |
| **Approval Gates** | Single | Multiple |

---

## Error Codes

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-HF-001` | 🔴 BLOCKING | Not a production incident | Use standard BugFix mode |
| `ENG-HF-002` | 🔴 BLOCKING | No reproduction steps provided | Request minimal repro |
| `ENG-HF-003` | 🟠 WARNING | Side effects identified | Document and proceed with caution |
| `ENG-HF-004` | 🟠 WARNING | Fix made things worse | Execute rollback protocol |
| `ENG-HF-005` | 🟡 INFO | Hotfix complete | Proceed to post-hotfix requirements |

---

## Quick Reference

```
HOTFIX FLOW
───────────────────────────────────────
[Production Down]
       │
       ▼
[Fetch Jira + Context] ─── 2-3 min
       │
       ▼
[Quick Root Cause] ─────── 5-10 min
       │
       ▼
[Write Test + Fix] ─────── 10-20 min
       │
       ▼
[Commit + Push] ────────── 2-5 min
       │
       ▼
[DEPLOY] ←─────── External (CI/CD)
       │
       ▼
[Post-Incident Work] ───── Within 24-48h
───────────────────────────────────────
```
