# Team Beat Mode (`/beat`)

Goal: Daily sprint health focused on **commitment accuracy** with explainable metrics.

> **Calculations**: All formulas defined in `../_calculation-engine.md`

---

## Data Fetch (JQL)

### Primary Query
```jql
project = {TARGET} AND sprint in openSprints()
```

### Secondary Queries (for metrics)

| Metric | JQL |
|--------|-----|
| Done | `... AND status = Done` |
| In Progress | `... AND status = "In Progress"` |
| Blocked | `... AND (status in ("Blocked", "On Hold") OR labels in ("blocked", "needs-input", "waiting"))` |
| Stale | `... AND status != Done AND updated < -{STALE_DAYS}d` |
| Review Breach | `... AND status = "In Code Review" AND updated < -{REVIEW_SLA_HOURS}h` |
| Added After Start | `... AND created >= "{SPRINT_START_DATE}"` |
| Aging WIP | `... AND status = "In Progress" AND statusCategoryChangedDate < -{AGING_WIP_DAYS}d` |

---

## Metric Calculations

### 1. Commitment Snapshot

```
PLANNED = COUNT(issues at sprint start)
DONE = COUNT(issues WHERE status = Done)
REMAINING = PLANNED - DONE
DAYS_LEFT = sprint_end_date - today

ADDED_AFTER_START = COUNT(issues WHERE created >= sprint_start_date)
SCOPE_CREEP_RATIO = ADDED_AFTER_START / PLANNED * 100
```

### 2. Rollover Candidates

An issue is a rollover candidate if ANY of:

| Condition | Threshold |
|-----------|-----------|
| In Progress too long | `statusCategoryChangedDate < today - AGING_WIP_DAYS` |
| Blocked too long | `blocked_days > BLOCKED_DAYS` |
| In Review too long | `review_hours > REVIEW_SLA_HOURS` |
| Has unresolved blocking link | `issuelinks.type = "is blocked by" AND linked.status != Done` |

```
ROLLOVER_CANDIDATES = COUNT(issues matching any condition)
ROLLOVER_RATIO = ROLLOVER_CANDIDATES / PLANNED * 100
```

### 3. Delivery Confidence

Use formula from `_calculation-engine.md` § 1:

```
confidence_score = 100 - (scope_penalty + rollover_penalty + blocked_penalty + review_penalty)

Display:
  🟢 if score >= 80
  🟠 if 60 <= score < 80
  🔴 if score < 60
```

### 4. Threat Identification

Automatically flag top 3 threats based on:

| Priority | Threat Type | Detection |
|----------|-------------|-----------|
| 1 | Critical Path Blocked | Blocked item has dependents in sprint |
| 2 | Scope Creep Surge | Added > 2 items in last 24h |
| 3 | Rollover Candidates | > 20% of remaining work |
| 4 | Review Bottleneck | > 3 items breaching SLA |
| 5 | WIP Overload | Any dev with > MAX_WIP items |

For each threat, calculate:
- **Evidence**: Specific issue keys and metrics
- **Impact**: How many items/days affected
- **Intervention**: Concrete action with owner

---

## Output Template

```markdown
# Team Beat — {TARGET} — Sprint {SPRINT}
**Generated**: {DATE_TIME}

---

## Delivery Confidence: {🟢/🟠/🔴} {SCORE}/100

### Score Breakdown
| Factor | Value | Threshold | Penalty |
|--------|-------|-----------|---------|
| Scope Creep | {PCT}% | WARN: 15% | {N} |
| Rollover Candidates | {PCT}% | WARN: 20% | {N} |
| Blocked Issues | {COUNT} ({AGING}) | DAYS: 2 | {N} |
| Review SLA Breaches | {COUNT} | SLA: 48h | {N} |
| **Total Penalty** | | | **{TOTAL}** |

### Primary Drivers
1. {driver_1}
2. {driver_2}
3. {driver_3}

---

## Commitment Snapshot

| Metric | Value | Status |
|--------|-------|--------|
| Planned | {PLANNED} | — |
| Done | {DONE} | {PCT}% |
| Remaining | {REMAINING} | {PCT}% |
| Days Left | {DAYS_LEFT} | — |
| Required Daily Rate | {RATE} items/day | {🟢/🟠/🔴} |

### Flow Health
| Metric | Count | Status |
|--------|-------|--------|
| Scope Creep | +{ADDED} ({PCT}%) | {🟢/🟠/🔴} |
| Rollover Candidates | {COUNT} ({PCT}%) | {🟢/🟠/🔴} |
| Blocked | {COUNT} | {🟢/🟠/🔴} |
| Review > SLA | {COUNT} | {🟢/🟠/🔴} |
| Stale (no updates) | {COUNT} | {🟢/🟠/🔴} |

---

## Top 3 Threats

### 1. {THREAT_NAME}
- **Evidence**: {issue_keys}, {metric_values}
- **Impact**: {N} items affected, {N} days at risk
- **Intervention**: {action}
- **Owner**: {name} | **Deadline**: {date}

### 2. {THREAT_NAME}
- **Evidence**: {facts}
- **Impact**: {assessment}
- **Intervention**: {action}
- **Owner**: {name} | **Deadline**: {date}

### 3. {THREAT_NAME}
- **Evidence**: {facts}
- **Impact**: {assessment}
- **Intervention**: {action}
- **Owner**: {name} | **Deadline**: {date}

---

## Today's Focus (Top 3)

| Priority | Action | Target | Owner |
|----------|--------|--------|-------|
| 1️⃣ Unblock | {item} | {reason} | {name} |
| 2️⃣ Finish | {item} | {reason} | {name} |
| 3️⃣ Review | {item} | {reason} | {name} |

---

## Wins (Last 24-48h)

- ✅ {completed_item_1}: {brief_impact}
- ✅ {completed_item_2}: {brief_impact}
- ✅ {completed_item_3}: {brief_impact}

---

## WIP Distribution

| Assignee | In Progress | Status |
|----------|-------------|--------|
| {name_1} | {count} | {🟢/🟠/🔴} |
| {name_2} | {count} | {🟢/🟠/🔴} |
| {name_3} | {count} | {🟢/🟠/🔴} |

**Team Average**: {avg_wip} | **Limit**: {MAX_WIP}
```

---

## Error Codes

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-BT-001` | 🔴 BLOCKING | Jira query failed | Check Jira connectivity |
| `MGR-BT-002` | 🔴 BLOCKING | No active sprint found | Verify sprint exists and is active |
| `MGR-BT-003` | 🟠 WARNING | Sprint data incomplete (missing story points) | Calculate by issue count |
| `MGR-BT-004` | 🟠 WARNING | No sprint start date available | Cannot calculate scope creep |
| `MGR-BT-005` | 🟡 INFO | Beat report generated | Output ready |
