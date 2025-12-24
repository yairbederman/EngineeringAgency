# Strategic Risk Mode (`/risk`)

**Goal**: Predictive analysis for Engineering Managers/Directors.
**Frequency**: Weekly (or Ad-Hoc for Board Meetings).
**Scope**: Epics, Initiatives, and Cross-Project dependencies.

## 1. Input parameters

*   `Target`: Project Key (e.g., `WG3`, `W0`).
*   `Horizon`: "Quarter" (default) or specific targeted "Epic Link".

## 2. Analysis Heuristics

The "Risk Radar" applies these rules to long-running items (Epics):

### 📈 Scope Creep Rule
*   **Logic**: Compare `Total Story Points` at `Epic Start Date` vs. `current`.
*   **Threshold**: If growth > 20% post-start.
*   **Diagnosis**: "Scope Explosion". Feature is growing uncontrollably.
*   **Action**: Flag for "Change Control Review".

### 📉 Timeline Slippage (Monte Carlo Lite)
*   **Logic**: 
    1. Calculate `Avg Velocity` (last 3 sprints).
    2. Calculate `Remaining Points` in Epic.
    3. `Projected End` = Today + (Remaining / Velocity).
*   **Check**: Is `Projected End` > `Due Date`?
*   **Diagnosis**: "Miss Risk".
*   **Severity**: 
    *   < 1 Sprint slippage = 🟡 Low
    *   > 2 Sprints slippage = 🔴 Critical

### 🔗 Dependency Web
*   **Logic**: Scan active Epics for `is blocked by` links where the blocker is in a *different* project.
*   **Diagnosis**: "External Risk". We are not in control of our destiny.
*   **Action**: List external owners to ping.

### 🧟 Zombie Epic Rule
*   **Logic**: Epic status is "In Progress" but `Done Ratio` < 10% after 4 weeks.
*   **Diagnosis**: "Zombie Feature". Resources allocated but nothing shipping.
*   **Action**: Suggest "Kill or Rescue".

## 3. Output: Risk Radar Report

**File**: `${MANAGER_ROOT}/reports/risk-{date}.md`

### Template

```markdown
# 📡 Risk Radar: Q1 Analysis
**Project**: [Key] | **Velocity**: [N] pts/sprint

## 🔴 Critical Risks (Requires Intervention)

| Epic | Risk Type | Details | projected Delay |
|------|-----------|---------|-----------------|
| [WG3-900] Payments | **Scope Creep** | grew +40% (20 -> 28pts). | +1.5 Sprints |
| [WG3-850] Search | **Dependency** | Blocked by `LTS-CORE` API refactor. | Unknown |

## 🟡 Watchlist (Trending Negative)
*   **[WG3-700] Mobile UI**: Velocity slowed by 50% last sprint.
*   **Zombies**: [WG3-600] has been open 8 weeks with 5% completion.

## 📉 Timeline Projection
*   **Optimistic Finish**: Feb 15
*   **Likely Finish**: Mar 01 (⚠️ Misses Launch Date)
*   **Pessimistic Finish**: Mar 15

## 💡 Recommendations
1.  **De-scope [WG3-900]**: Cut the "Crypto" feature to save 8pts.
2.  **Escalate [WG3-850]**: Meeting needed with Core Team Lead.
```

## 4. Execution Steps

1.  **Parse Request**: Determine scope (Project or specific Epics).
2.  **Jira Query**: Fetch Epics, their children stats, and Sprint Velocity.
3.  **Process Data**: Run Slippage and Creep calculations.
4.  **Format Output**: Generate Markdown report.
