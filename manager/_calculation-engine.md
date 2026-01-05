# Manager Agent Calculation Engine

> **Purpose**: Concrete formulas for computing delivery metrics.
> **Usage**: Reference these calculations in all Manager Agent modes to ensure consistent, explainable outputs.

---

## 1. Delivery Confidence Score

### Formula

```
Delivery Confidence = 100 - (Scope Penalty + Rollover Penalty + Blocked Penalty + Review Penalty)
```

| Threshold | Confidence | Indicator |
|-----------|------------|-----------|
| ≥ 80 | High | 🟢 |
| 60-79 | Medium | 🟠 |
| < 60 | Low | 🔴 |

### Penalty Calculations

#### Scope Creep Penalty
```
scope_creep_ratio = added_after_start / planned_at_start
scope_penalty = 0                         if ratio < 0.10
scope_penalty = (ratio - 0.10) * 100      if 0.10 <= ratio < 0.25
scope_penalty = 15 + (ratio - 0.25) * 200 if ratio >= 0.25

Example:
- 8% scope creep  → 0 penalty
- 18% scope creep → (0.18 - 0.10) * 100 = 8 points
- 30% scope creep → 15 + (0.30 - 0.25) * 200 = 25 points
```

#### Rollover Candidate Penalty
```
rollover_ratio = rollover_candidates / total_sprint_items
rollover_penalty = 0                          if ratio < 0.15
rollover_penalty = (ratio - 0.15) * 100       if 0.15 <= ratio < 0.35
rollover_penalty = 20 + (ratio - 0.35) * 150  if ratio >= 0.35

Example:
- 10% rollover candidates → 0 penalty
- 25% rollover candidates → (0.25 - 0.15) * 100 = 10 points
- 40% rollover candidates → 20 + (0.40 - 0.35) * 150 = 27.5 points
```

#### Blocked Issue Penalty
```
blocked_weight = SUM over each blocked issue:
  - Base: 3 points per blocked item
  - Age multiplier: +2 points for each day blocked > BLOCKED_DAYS threshold
  - Critical path: +5 points if issue is on critical path (has dependents waiting)

blocked_penalty = MIN(blocked_weight, 30)  # Cap at 30 points

Example:
- 2 blocked issues (both at 1 day): 2 * 3 = 6 points
- 3 blocked issues (1 at 5 days, 2 at 2 days): 3*3 + 2*(5-2)*1 = 9 + 6 = 15 points
```

#### Review SLA Breach Penalty
```
review_breach_count = issues where (status = "In Review" AND age > REVIEW_SLA_HOURS)
review_penalty = review_breach_count * 2  # 2 points per breach
review_penalty = MIN(review_penalty, 15)  # Cap at 15 points

Example:
- 3 items in review > 48 hours: 3 * 2 = 6 points
- 10 items in review > 48 hours: MIN(20, 15) = 15 points (capped)
```

---

## 2. Burndown Velocity

### Current Velocity (Rolling)
```
# Use last 3 completed sprints for stability
completed_sprints = [sprint_n-1, sprint_n-2, sprint_n-3]
velocities = [sprint.done / sprint.planned for sprint in completed_sprints]
avg_velocity = MEAN(velocities)
velocity_std = STDEV(velocities)

# Confidence interval
velocity_range = (avg_velocity - velocity_std, avg_velocity + velocity_std)
```

### Projected Completion
```
remaining_work = total_sprint_items - done_items
days_remaining = sprint_end_date - today
required_daily_rate = remaining_work / days_remaining

# Compare to historical daily rate
historical_daily_rate = avg_velocity * planned / sprint_duration

completion_likelihood = 
  - 🟢 "On Track"     if required_daily_rate <= historical_daily_rate * 1.1
  - 🟠 "At Risk"      if required_daily_rate <= historical_daily_rate * 1.5
  - 🔴 "Off Track"    if required_daily_rate > historical_daily_rate * 1.5
```

### Burndown Chart Data Points
```
# Generate daily expected vs actual
for each day d in sprint:
  expected_remaining[d] = planned - (planned / sprint_duration * d)
  actual_remaining[d] = COUNT(issues WHERE status NOT IN done_statuses AND updated <= d)

# Trend calculation
burndown_delta = actual_remaining[today] - expected_remaining[today]
  - Positive delta = behind schedule
  - Negative delta = ahead of schedule
```

---

## 3. Risk Scoring

### Individual Risk Score
```
risk_score = likelihood * impact

Likelihood scale (1-5):
  1 = Unlikely (< 20%)
  2 = Possible (20-40%)
  3 = Likely (40-60%)
  4 = Very Likely (60-80%)
  5 = Almost Certain (> 80%)

Impact scale (1-5):
  1 = Minimal (1-2 items affected)
  2 = Minor (3-5 items affected)
  3 = Moderate (6-10 items affected, OR 1 epic affected)
  4 = Major (11-20 items affected, OR 2+ epics affected)
  5 = Critical (> 20 items affected, OR release milestone at risk)

Risk classification:
  - Score 1-6:   Low risk (monitor)
  - Score 7-12:  Medium risk (watchlist)
  - Score 13-19: High risk (mitigation required)
  - Score 20-25: Critical risk (escalate immediately)
```

### Automated Risk Detection

| Risk Type | Detection Logic | Likelihood Formula | Impact Formula |
|-----------|-----------------|-------------------|----------------|
| Scope Creep | `added_after_start / planned > SCOPE_CREEP_WARN` | `MIN(5, ratio / 0.10)` | Items added count / 5 |
| Rollover | `rollover_candidates / total > ROLLOVER_WARN` | `MIN(5, ratio / 0.15)` | Candidate count / 3 |
| Blocked Aging | `blocked_items.any(age > BLOCKED_DAYS * 2)` | `MAX(blocked_ages) / BLOCKED_DAYS` | Blocked count |
| Review Stall | `in_review.any(age > REVIEW_SLA_HOURS * 2)` | `breach_count / 3` | `MIN(5, breach_count)` |
| Dependency Drag | `blocked_by_external > 0` | 4 (external = high uncertainty) | External blocked count |
| Quality Drift | `bugs_created_in_sprint > planned_bugs * 1.5` | Bug ratio | Bug count / 5 |

---

## 4. Trend Analysis

### Week-over-Week Comparison
```
# For any metric M:
trend = (M_current_week - M_previous_week) / M_previous_week * 100

trend_indicator = 
  - "↑ +X%" if trend > 5%
  - "↓ -X%" if trend < -5%
  - "→ stable" if -5% <= trend <= 5%
```

### Sprint-over-Sprint Velocity Trend
```
velocity_trend = []
for i in range(1, len(completed_sprints)):
  delta = (sprints[i].velocity - sprints[i-1].velocity) / sprints[i-1].velocity
  velocity_trend.append(delta)

# Classification
if MEAN(velocity_trend) > 0.05:
  trend = "📈 Improving"
elif MEAN(velocity_trend) < -0.05:
  trend = "📉 Declining"
else:
  trend = "➡️ Stable"
```

### Rollover Pattern Detection
```
# Track rollover items across sprints
rollover_chain = {}
for issue in current_sprint:
  if issue in previous_sprint AND issue.status != DONE:
    rollover_chain[issue] = rollover_chain.get(issue, 0) + 1

# Flag chronic rollovers
chronic_rollovers = [issue for issue in rollover_chain if rollover_chain[issue] >= 2]

# This indicates systemic issues (sizing, dependencies, unclear requirements)
```

---

## 5. WIP Analysis

### Per-Developer WIP
```
for each assignee:
  wip_count = COUNT(issues WHERE assignee = dev AND status IN in_progress_statuses)
  wip_age = AVG(days_since_status_change for issues in wip)
  
  health = 
    - 🟢 if wip_count <= MAX_WIP AND wip_age <= AGING_WIP_DAYS
    - 🟠 if wip_count > MAX_WIP OR wip_age > AGING_WIP_DAYS
    - 🔴 if wip_count > MAX_WIP * 1.5 OR wip_age > AGING_WIP_DAYS * 2
```

### Team WIP Distribution
```
total_in_progress = COUNT(issues WHERE status IN in_progress_statuses)
team_size = COUNT(DISTINCT assignees WHERE status IN in_progress_statuses)
avg_wip = total_in_progress / team_size

wip_distribution = 
  - "Balanced" if STDEV(per_dev_wip) < 1
  - "Uneven" if STDEV(per_dev_wip) >= 1 AND < 2
  - "Overloaded" if STDEV(per_dev_wip) >= 2
```

---

## 6. Evidence Collection Protocol

### Required Evidence for Each Claim

When outputting any metric, ALWAYS include:

1. **Raw Numbers**: The actual counts/values from Jira
2. **Calculation**: How you derived the final number
3. **Comparison**: To threshold or historical baseline
4. **Trend**: Direction compared to last measurement

**Example Output Format**:
```markdown
## Scope Creep: 22% 🟠

**Evidence**:
- Planned at sprint start: 18 items
- Current sprint backlog: 22 items
- Added after start: 4 items
- Calculation: 4/18 = 22.2%
- Threshold: WARN at 15%, CRIT at 25%
- Trend: ↑ +8% from last week (was 14%)
- Impact: 8 penalty points applied to Delivery Confidence
```

---

## 7. Confidence Display Format

Always output Delivery Confidence in this format:

```markdown
## Delivery Confidence: 🟢 82/100

### Score Breakdown
| Factor | Value | Threshold | Penalty |
|--------|-------|-----------|---------|
| Scope Creep | 8% | WARN: 15% | 0 |
| Rollover Candidates | 18% | WARN: 20% | 0 |
| Blocked Issues | 2 (1 aging) | DAYS: 2 | 6 |
| Review SLA Breaches | 3 | SLA: 48h | 6 |
| **Total Penalty** | | | **12** |

### Primary Drivers
1. ✅ Scope under control (< 10%)
2. 🟠 3 items stuck in code review (> 48h)
3. 🟠 1 blocked item aging (5 days blocked)
```

---

## 8. Jira Query Library

### Sprint Health Queries (JQL)

```jql
# All sprint items
project = {TARGET} AND sprint in openSprints()

# Done items
project = {TARGET} AND sprint in openSprints() AND status = Done

# In Progress
project = {TARGET} AND sprint in openSprints() AND status = "In Progress"

# Blocked
project = {TARGET} AND sprint in openSprints() AND (status in ("Blocked", "On Hold") OR labels in ("blocked", "needs-input", "waiting"))

# Stale (no updates in X days)
project = {TARGET} AND sprint in openSprints() AND status != Done AND updated < -{STALE_DAYS}d

# In Review (over SLA)
project = {TARGET} AND sprint in openSprints() AND status = "In Code Review" AND updated < -{REVIEW_SLA_HOURS}h

# Added after sprint start
project = {TARGET} AND sprint in openSprints() AND created >= "{SPRINT_START_DATE}"

# Rollover candidates (in progress > X days)
project = {TARGET} AND sprint in openSprints() AND status = "In Progress" AND statusCategoryChangedDate < -{AGING_WIP_DAYS}d
```

### Historical Queries

```jql
# Last completed sprint
project = {TARGET} AND sprint in closedSprints() ORDER BY updated DESC

# Velocity history (done items per sprint)
project = {TARGET} AND sprint = "{SPRINT_ID}" AND status = Done
```

---

## 9. Thresholds Quick Reference

| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Delivery Confidence | ≥ 80 | 60-79 | < 60 |
| Scope Creep | < 10% | 10-25% | > 25% |
| Rollover Candidates | < 15% | 15-35% | > 35% |
| Blocked Aging | < 2 days | 2-4 days | > 4 days |
| Review SLA | < 48h | 48-72h | > 72h |
| WIP per Dev | ≤ 3 | 4-5 | > 5 |
| Stale Items | 0 | 1-2 | > 2 |
