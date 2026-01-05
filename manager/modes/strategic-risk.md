# Strategic Risk Mode (`/risk`)

Goal: Weekly radar for threats to predictable delivery with quantified risk scores.

> **Calculations**: Risk scoring formula defined in `../_calculation-engine.md` § 3

---

## Data Fetch (JQL)

### Sprint Context
```jql
# Current sprint
project = {TARGET} AND sprint in openSprints()

# Next sprint (planning)
project = {TARGET} AND sprint in futureSprints() ORDER BY "Sprint" ASC

# Related epics
project = {TARGET} AND issuetype = Epic AND status != Done
```

### Risk Signals Query
```jql
# Scope creep (added after sprint start)
project = {TARGET} AND sprint in openSprints() AND created >= "{SPRINT_START_DATE}"

# Rollover candidates (aging WIP)
project = {TARGET} AND sprint in openSprints() AND status = "In Progress" AND statusCategoryChangedDate < -{AGING_WIP_DAYS}d

# Dependency blockers
project = {TARGET} AND sprint in openSprints() AND (labels in ("blocked", "waiting", "needs-input") OR status in ("Blocked", "On Hold"))

# Review bottleneck
project = {TARGET} AND sprint in openSprints() AND status = "In Code Review" AND updated < -{REVIEW_SLA_HOURS}h

# Quality signals (bugs)
project = {TARGET} AND issuetype = Bug AND created >= -{HORIZON}d
```

---

## Risk Categories

### 1. Commitment Risks (Direct Impact on Sprint)

| Risk Type | Detection Formula | Likelihood | Impact |
|-----------|-------------------|------------|--------|
| **Scope Creep** | `added / planned > SCOPE_CREEP_WARN` | `MIN(5, ratio / 0.10)` | `added_count / 5` |
| **Rollover Surge** | `rollover_candidates / remaining > ROLLOVER_WARN` | `MIN(5, ratio / 0.15)` | `candidate_count / 3` |
| **Blocked Aging** | `any(blocked.age > BLOCKED_DAYS * 2)` | `MAX(ages) / BLOCKED_DAYS` | `blocked_count` |
| **Review Stall** | `review_breaches > 3` | `breach_count / 3` | `MIN(5, breach_count)` |

### 2. Downstream Risks (Next Sprint / Quarter)

| Risk Type | Detection Formula | Likelihood | Impact |
|-----------|-------------------|------------|--------|
| **Dependency Drag** | `external_blockers > 0` | 4 (external = uncertain) | `external_count` |
| **Tech Debt Accumulation** | `tech_debt_items / sprint_velocity > 0.3` | 3 | 3 |
| **Resource Availability** | `upcoming_pto / team_capacity > 0.2` | If mentioned in context | Team capacity impact |
| **Epic Delay Cascade** | `epic_items_remaining / velocity > sprints_to_deadline` | Epic completion risk | Epic business value |

### 3. Quality Risks

| Risk Type | Detection Formula | Likelihood | Impact |
|-----------|-------------------|------------|--------|
| **Bug Inflow** | `bugs_created > bugs_resolved * 1.5` | Bug ratio | Business-critical bugs |
| **Regression Rate** | `reopened_issues / done_issues > 0.1` | Reopen ratio | Quality perception |
| **Test Coverage Gap** | If flagged in PRs/context | Context-based | Release confidence |

---

## Risk Scoring (from `_calculation-engine.md`)

```
risk_score = likelihood * impact

Classification:
  Score 1-6:   Low (monitor)
  Score 7-12:  Medium (watchlist) 🟠
  Score 13-19: High (mitigation required) 🔴
  Score 20-25: Critical (escalate immediately) 🔴🔴
```

For each risk, compute:
- **Likelihood (1-5)**: How probable based on current data
- **Impact (1-5)**: How many items/days/milestones affected
- **Trend**: Compared to last week (improving/stable/worsening)
- **Mitigation**: Concrete action with owner

---

## Trend Analysis

### Week-over-Week Comparison

For each metric, calculate:
```
trend_pct = (current - previous) / previous * 100

Display:
  ↑ +X%  if trend > 5%
  ↓ -X%  if trend < -5%
  → stable  if -5% <= trend <= 5%
```

### Pattern Detection

| Pattern | Detection | Action |
|---------|-----------|--------|
| Chronic Rollover | Same items rolled 2+ sprints | Escalate for rescoping |
| Blocking Chain | 3+ items blocked by same dependency | Coordinate with dependency owner |
| Review Bottleneck | Same reviewer on 3+ stalled PRs | Load balance reviews |
| Scope Creep Source | 50%+ additions from same requester | Stakeholder conversation |

---

## Output Template

```markdown
# Risk Radar — {TARGET}
**Horizon**: {HORIZON} | **Generated**: {DATE_TIME}

---

## Overall Risk Level: {🟢/🟠/🔴}

**Assessment**: {One sentence summary of risk posture}

---

## 🔴 Critical Risks (Commitment Threat)

| # | Risk | Score | Evidence | Impact | Trend |
|---|------|-------|----------|--------|-------|
| 1 | {Risk Name} | {L}×{I}={S} | {specific data} | {items/days} | {↑/↓/→} |
| 2 | {Risk Name} | {L}×{I}={S} | {specific data} | {items/days} | {↑/↓/→} |

### Mitigations Required

**Risk 1: {Name}**
- **Root Cause**: {analysis}
- **Mitigation**: {concrete action}
- **Owner**: {name}
- **Deadline**: {date}
- **Escalation**: {if not resolved by deadline, then...}

**Risk 2: {Name}**
- **Root Cause**: {analysis}
- **Mitigation**: {concrete action}
- **Owner**: {name}
- **Deadline**: {date}

---

## 🟠 Watchlist (Trending Negative)

| Risk | Score | Current | Last Week | Trend |
|------|-------|---------|-----------|-------|
| {Risk 1} | {S} | {value} | {value} | {↑ +X%} |
| {Risk 2} | {S} | {value} | {value} | {↑ +X%} |
| {Risk 3} | {S} | {value} | {value} | {→ stable} |

**Action**: Monitor daily. Escalate if crosses threshold.

---

## 📊 Leading Indicators

| Indicator | Current | Threshold | Status |
|-----------|---------|-----------|--------|
| Scope Creep | {PCT}% | WARN: 15%, CRIT: 25% | {🟢/🟠/🔴} |
| Rollover Candidates | {PCT}% | WARN: 20%, CRIT: 35% | {🟢/🟠/🔴} |
| Blocked Aging | {MAX_DAYS}d | WARN: 2d, CRIT: 4d | {🟢/🟠/🔴} |
| Review SLA Breaches | {COUNT} | WARN: 3, CRIT: 6 | {🟢/🟠/🔴} |
| Bug Inflow/Outflow | {RATIO} | WARN: 1.5, CRIT: 2.0 | {🟢/🟠/🔴} |

### Trend Summary
```
Scope:    {7d ago} → {today} ({trend})
Rollover: {7d ago} → {today} ({trend})
Blocked:  {7d ago} → {today} ({trend})
Review:   {7d ago} → {today} ({trend})
```

---

## 🔗 Dependency Map

| Dependency | Type | Blocked Items | Owner | Status |
|------------|------|---------------|-------|--------|
| {Team/Service} | External | {count} | {name} | {ETA or blocker} |
| {Team/Service} | Internal | {count} | {name} | {ETA or blocker} |

---

## ✋ Decisions / Asks

| # | Decision Needed | Impact If Delayed | Owner | Deadline |
|---|-----------------|-------------------|-------|----------|
| 1 | {decision} | {impact} | {name} | {date} |
| 2 | {decision} | {impact} | {name} | {date} |

---

## 📈 Velocity Context (if historical data available)

| Metric | Last 3 Sprints Avg | Current Sprint | Trend |
|--------|-------------------|----------------|-------|
| Planned | {avg} | {current} | {+/-}% |
| Completed | {avg} | {projected} | {+/-}% |
| Rollover | {avg}% | {current}% | {+/-}% |

**Velocity Trend**: {📈 Improving / ➡️ Stable / 📉 Declining}
```

---

## Error Codes

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-RS-001` | 🔴 BLOCKING | Jira query failed | Check Jira connectivity |
| `MGR-RS-002` | 🟠 WARNING | No historical data (first sprint) | Omit trend analysis |
| `MGR-RS-003` | 🟠 WARNING | Sprint start date unavailable | Cannot calculate scope creep |
| `MGR-RS-004` | 🟠 WARNING | External dependencies not tracked in Jira | Note in dependency section |
| `MGR-RS-005` | 🟡 INFO | Risk radar generated | Output ready |
