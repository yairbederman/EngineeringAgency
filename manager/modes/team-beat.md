# Team Beat Mode (`/beat`)

**Goal**: Instant operational health check for the current sprint.
**Frequency**: Daily (Morning Standup prep).

## 1. Input parameters

*   `Target`: Project Key (e.g., `WG3`) or specific Board ID.
*   `Sprint`: Defaults to "Active Sprint".

## 2. Data Fetching Protocol

**JQL Query Strategy**:
```jql
project = {TARGET} AND sprint in openSprints() AND statusCategory in ("In Progress", "To Do")
```

**Required Fields**:
*   `status`: To group by lane.
*   `assignee`: To calculate load.
*   `updated`: To calculate staleness (Last Updated Date).
*   `issuelinks`: To identify Blockers (`blocks`, `is blocked by`).
*   `customfield_[dev_status]`: (If available) to check for commit activity.

## 3. Analysis Heuristics

The "Brain" of the Beat mode applies these rules to the raw data:

### 🟠 Stagnation Rule (The "Rot" Check)
*   **Logic**: IF `status` == "In Progress" AND `updated` < (Today - 3 days).
*   **Diagnosis**: "Stalled". The ticket is likely abandoned or stuck without visibility.
*   **Action**: Flag for "Immediate Update Required".

### 🔴 Overload Rule (The "Jam" Check)
*   **Logic**: IF `assignee` has > 3 tickets with `status` == "In Progress".
*   **Diagnosis**: "Context Switching Warning". Developer efficiency drops significantly after 2 concurrent tasks.
*   **Action**: Suggest moving lowest priority item to "To Do".

### 🔎 Review Rot Rule
*   **Logic**: IF `status` == "In Review" AND `updated` < (Today - 2 days).
*   **Diagnosis**: "Bottle-neck". Code is waiting for peers.
*   **Action**: Ping reviewers.

### ⛓️ Blocker Chain Rule
*   **Logic**: IF Task A `is blocked by` Task B, AND Task B is `blocked by` Task C.
*   **Diagnosis**: "Dependency Hell".
*   **Action**: Elevate to Team Lead immediately.

## 4. Output: Daily Pulse Report

**Output**: Present directly in Chat.

### Template

```markdown
# 💓 Daily Pulse: [Date]
**Sprint Goal**: [Goal Text] | **Days Left**: [N]

## 🚨 Immediate Attention (Red Flags)
| Issue | Owner | Problem | Recommendation |
|-------|-------|---------|----------------|
| [WG3-101] | @Sarah | Stalled (5d) | Ask for update or move to Backlog |
| [WG3-202] | @Mike | Overloaded (5 active) | Focus on [WG3-202], pause others |

## ⚠️ Potential Risks (Yellow Flags)
*   **Review Queue**: 4 items waiting in "Review" > 48hrs.
*   **Unassigned Work**: 2 "In Progress" tickets have no owner.

## 📊 Team Load (WIP Limits)
*   Sarah: [|||||] (5) - 🔴 High
*   David: [||] (2) - 🟢 Healthy
*   Jessica: [] (0) - ⚪ Free

## ✅ Recent Wins (Last 24h)
*   [WG3-305] Moved to Done
```

## 5. Execution Steps for Agent

1.  **Parse Request**: Identify project.
2.  **Jira Query**: Run `${MCP_ATLASSIAN_SEARCH_JQL}`.
3.  **Process Data**: Iterate through issues, applying heuristics in memory.
4.  **Format Output**: Generate Markdown table.
5.  **Present**: Display summary table in chat using the template below.
