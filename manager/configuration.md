# Manager Agent Configuration (Predictable Delivery)

> **Calculation Formulas**: See `_calculation-engine.md` for all metric formulas.
> **Error Codes**: See `../shared/error-codes.md` § Manager Agent.

---

## 1) Inheritance

Read these files before executing:
- `${WORKSPACE_ROOT}/Agent_Config/agent-config.md` — Project registry, Atlassian settings
- `../shared/mcp-config.md` — MCP tool references
- `./_calculation-engine.md` — Calculation formulas

---

## 2) Default Target

If user doesn't specify `Target`, use:
- `${JIRA_PROJECT_KEY}` from `Agent_Config/agent-config.md`

---

## 3) Heuristics (Predictability Levers)

### Thresholds

| Key | Default | Unit | Purpose |
|-----|---------|------|---------|
| `STALE_DAYS` | 4 | days | No meaningful updates → item is rotting |
| `MAX_WIP` | 3 | items | WIP limit per developer |
| `REVIEW_SLA_HOURS` | 48 | hours | "In Review" too long threshold |
| `BLOCKED_DAYS` | 2 | days | Blocked too long threshold |
| `AGING_WIP_DAYS` | 4 | days | In Progress too long → rollover candidate |

### Warning/Critical Thresholds

| Key | Warn | Critical | Purpose |
|-----|------|----------|---------|
| `SCOPE_CREEP` | 0.15 (15%) | 0.25 (25%) | Items added after sprint start |
| `ROLLOVER` | 0.20 (20%) | 0.35 (35%) | Rollover candidate ratio |
| `BLOCKED_AGING` | 2× threshold | 3× threshold | Long-blocked items |
| `REVIEW_BREACH` | 3 items | 6 items | Items over SLA |

### Status Indicators

```
🟢 Green:  Below all warn thresholds
🟠 Yellow: At or above any warn threshold, below all critical
🔴 Red:    At or above any critical threshold
```

---

## 4) Jira Conventions

> **Customize these to match your Jira board configuration**

### Status Buckets

| Bucket | Status Names (comma-separated) |
|--------|-------------------------------|
| `TODO` | `To Do`, `Open`, `Backlog` |
| `IN_PROGRESS` | `In Progress`, `In Development` |
| `IN_REVIEW` | `In Code Review`, `In QA`, `Awaiting Review` |
| `BLOCKED` | `Blocked`, `On Hold`, `Waiting` |
| `DONE` | `Done`, `Closed`, `Resolved` |

### Blocked Detection (any match = blocked)

```yaml
BLOCKED_STATUSES:
  - "Blocked"
  - "On Hold"

BLOCKED_LABELS:
  - "blocked"
  - "needs-input"
  - "waiting"
  - "dependency"

BLOCKED_FIELD: "Flagged"  # If Jira 'Flagged' field is set
```

### Priority Mapping

| Jira Priority | Weight (for risk) |
|---------------|-------------------|
| `Highest`, `Blocker` | 5 |
| `High` | 4 |
| `Medium` | 3 |
| `Low` | 2 |
| `Lowest` | 1 |

---

## 5) Delivery Confidence Rules

### Score Calculation

Start at 100, subtract penalties:

| Factor | Calculation | Max Penalty |
|--------|-------------|-------------|
| Scope Creep | See `_calculation-engine.md` §1 | 35 |
| Rollover Candidates | See `_calculation-engine.md` §1 | 35 |
| Blocked Issues | 3 pts/item + 2 pts/day over threshold | 30 |
| Review SLA Breach | 2 pts/item | 15 |

### Confidence Bands

| Score | Level | Display |
|-------|-------|---------|
| 80-100 | High | 🟢 |
| 60-79 | Medium | 🟠 |
| 0-59 | Low | 🔴 |

---

## 6) Risk Scoring

### Likelihood Scale (1-5)

| Score | Description | Detection |
|-------|-------------|-----------|
| 1 | Unlikely (< 20%) | No current signals |
| 2 | Possible (20-40%) | Early warning signals |
| 3 | Likely (40-60%) | Multiple warning signals |
| 4 | Very Likely (60-80%) | Threshold breached |
| 5 | Almost Certain (> 80%) | Critical threshold breached |

### Impact Scale (1-5)

| Score | Description | Scope |
|-------|-------------|-------|
| 1 | Minimal | 1-2 items affected |
| 2 | Minor | 3-5 items affected |
| 3 | Moderate | 6-10 items OR 1 epic |
| 4 | Major | 11-20 items OR 2+ epics |
| 5 | Critical | > 20 items OR release at risk |

### Risk Classification

| Score (L×I) | Level | Action |
|-------------|-------|--------|
| 1-6 | Low | Monitor |
| 7-12 | Medium | Watchlist 🟠 |
| 13-19 | High | Mitigation required 🔴 |
| 20-25 | Critical | Escalate immediately 🔴🔴 |

---

## 7) Output Formatting

### Standard Metric Format

Every metric MUST show:
```markdown
**{Metric Name}**: {value} {🟢/🟠/🔴}
- Raw: {raw_value}
- Calculation: {formula_applied}
- Threshold: WARN at {X}, CRIT at {Y}
- Trend: {↑/↓/→} {pct}% from {comparison_period}
```

### Standard Intervention Format

Every risk/threat MUST show:
```markdown
**{Risk Name}**
- Evidence: {specific data points}
- Impact: {quantified}
- Intervention: {specific action}
- Owner: {name}
- Deadline: {date}
```

---

## 8) Velocity Baseline

### How to Calculate (if historical data exists)

```
# Use last 3 completed sprints
sprints = [sprint_n-1, sprint_n-2, sprint_n-3]
velocities = [sprint.done for sprint in sprints]

avg_velocity = MEAN(velocities)
velocity_std = STDEV(velocities)

# Recommended commitment = average - 10% buffer
recommended_commitment = avg_velocity * 0.9
```

### If No History

If this is the first sprint:
- Omit velocity trend analysis
- Use planned items as baseline
- Emit `MGR-*-003` warning
