# Engineering Agent – Fast Track Mode

## Purpose

Enable rapid implementation of small, well-defined Jira Tasks without full planning overhead.

**Use When**: User says "Implement ${JIRA_PROJECT_KEY}-XXX" for an existing Task.

---

## Eligibility Criteria

| # | Criterion | Check | Auto-Detect |
|---|-----------|-------|-------------|
| 1 | **Issue Type** | Task or Sub-task (NOT Epic) | `issueType.name` |
| 2 | **Single Project** | All target files in 1 project | Parse file paths |
| 3 | **≤3 Files** | Modify/create max 3 files | Count in description |
| 4 | **Has Pattern** | References existing file or `${COPILOT_INSTRUCTIONS_PATH}` | Keyword scan |
| 5 | **No Schema** | No DB migrations | Keyword: "migration" |
| 6 | **No API Contract** | No new endpoints | Keyword: "new endpoint" |
| 7 | **Single Service** | No cross-service dependencies | Check workspace URIs |

### Eligibility Check Logic

```
1. IF issueType = Epic
   → REJECT: "Epics require standard workflow. Start with FeaturePlanning?"

2. IF files mentioned > 3
   → REJECT: "Task scope too large. Consider splitting into sub-tasks."

3. IF description contains "migration" OR "new table" OR "schema change"
   → REJECT: "Database changes require TechSpec."

4. IF description contains "new endpoint" OR "API contract"
   → REJECT: "API changes require TechSpec."

5. IF no pattern reference AND no "Reference File" section
   → WARN: "No pattern reference found. Recommend adding one."
   → PROCEED with warning

6. IF file paths span multiple workspace roots (e.g., wg-search-api AND wg-booking-api)
   → REJECT: "Cross-service changes require full planning workflow and TechSpec."

7. IF description contains "cross-service" OR "inter-service" OR "API call to [other-service]"
   → REJECT: "Cross-service integration requires TechSpec for contract alignment."

8. ELSE → PROCEED
```

---

## Fast Track Flow

### Step 1: Read Task

```
mcp0_getJiraIssue(key)
```

Extract:
- `summary` → Task title
- `description` → Parse for:
  - Target files (look for file paths)
  - Pattern reference (look for "Reference", "Following", "Similar to")
  - Test requirements
- `issuetype.name` → Must be "Task" or "Sub-task"

### Step 2: Eligibility Check

Apply criteria above. If any REJECT condition met, exit with guidance.

### Step 3: Context Load (Minimal)

```
1. Read ${COPILOT_INSTRUCTIONS_PATH} for affected project
2. Read target files (if MODIFY action)
3. Read pattern reference file (if specified)
```

**Skip**: Full pre-flight checks, Epic validation, Tech Spec lookup.

### Step 4: Implement

Standard TDD loop:
1. Write/update tests first
2. Implement changes
3. Run tests
4. Fix until green

### Step 5: Verify & Close

1. **Run tests**: All related tests must pass
2. **Commit**: `[${JIRA_PROJECT_KEY}-XXX] [Summary]`
3. **Jira**: Transition to "In Review" via `mcp0_transitionJiraIssue`
4. **Comment**: Post implementation summary via `mcp0_addCommentToJiraIssue`

---

## Entry Point

Fast Track is triggered when:
1. User provides a Jira key with "implement", "fix", or "build"
2. Issue type is Task or Sub-task
3. All eligibility criteria pass

Example triggers:
- `"Implement ${JIRA_PROJECT_KEY}-123"`
- `"Fix ${JIRA_PROJECT_KEY}-456"`
- `"Build the component in ${JIRA_PROJECT_KEY}-789"`

---

## Fallback to Standard Workflow

If eligibility fails, suggest appropriate mode:

| Failure Reason | Suggested Mode |
|----------------|----------------|
| Issue is Epic | FeaturePlanning or TechSpec |
| Too many files | TaskPlanning (split into sub-tasks) |
| Has migrations | TechSpec |
| Has API changes | TechSpec |
| Cross-service | TechSpec (contract alignment required) |
| Missing context | ProductSpecReview |
