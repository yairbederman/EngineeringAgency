# Sprint Retro Mode (`/retro`)

**Goal**: Analyze "What happened?" in the last sprint with managerial depth.
**Frequency**: End of Sprint / Review Prep.

## 1. Input parameters

*   `Target`: Project Key (e.g., `WG3`, `W0`).
*   `Sprint`: Defaults to "Active Sprint" (or mostly recently closed).

## 2. Dynamic Roster Discovery

Instead of a hardcoded list, we identify **Active Contributors** who touched the project recently.

**JQL for Roster**:
```jql
project = {TARGET} AND updated >= -30d
```
*   **Logic**: Collect all unique `Assignee` users.
*   **Filter**: Exclude "Automation for Jira" or generic system accounts.

## 3. Data Fetching Protocol

**Main JQL**:
```jql
project = {TARGET} AND sprint = {SPRINT_ID}
```

**Scope Creep JQL**:
```jql
project = {TARGET} AND sprint = {SPRINT_ID} AND created >= {SPRINT_START_DATE}
```

## 4. Analysis Heuristics

### 🔍 Scope Creep (The "Hidden Factory")
*   **Definition**: Tickets created *after* the sprint started.
*   **Impact Score**: Count of Unplanned Tickets / Total Tickets.
*   **Breakdown**: Were they Bugs (Quality issue) or Features (Planning issue)?

### 🏆 Accomplishments vs. Misses (Per Member)
For each user in the **Dynamic Roster**:

**Accomplishments (Likely Shipped)**:
*   Status: `Done`, `Verified`, `Ready for QA`.
*   *Note*: If `Ready for QA` > 3 days, flag as "Risk" but still credit dev effort.

**Misses (Carry Over / Stuck)**:
*   Status: `In Progress`, `To Do`, `Blocked`.
*   **Staleness**: If last updated > 5 days, flag as "Zombie".

## 5. Output: Managerial Retro Report

**Output**: Present directly in Chat.

### Template

```markdown
# 🏁 Sprint [N] Retro Analysis
**Window**: [Start] - [End]

## 1. 🔍 Scope Creep Analysis
**Volatility**: [High/Med/Low] ([N]% Unplanned)

| Key | Created | Type | Assignee | Impact |
|-----|---------|------|----------|--------|
| [${JIRA_PROJECT_KEY}-XXXX] | [Date] | Bug | [Assignee] | 🔴 Sprint Crash |

## 2. 👥 Team Performance (Accomplishments vs Misses)

### 👤 [User Name]
*   **🏆 Accomplishments**:
    *   `${JIRA_PROJECT_KEY}-XXX`: [Task Name] (Ready for QA)
*   **🥀 Misses**:
    *   `${JIRA_PROJECT_KEY}-XXX`: [Task Name] (Blocked by Env)
*   **🐛 Quality**: Fixed 2 bugs, Created 0.

### 👤 [Next User...]

## 3. 💡 Managerial Insights
*   **Resource Balance**: Shiran is handling 100% of bugs.
*   **Planning Health**: 50% of work was unplanned.
```

## 6. Execution Steps for Agent

1.  **Context**: Fetch Active Sprint to get ID and Start Date.
2.  **Roster**: Run 30-day lookback query to build team list.
3.  **Fetch Data**: Run Main JQL + Scope Creep logic.
4.  **Analyze**: Group by User, Categorize into Accomplishments/Misses.
5.  **Present**: Render Markdown report in chat.
