# Sprint Retro Mode (`/retro`)

Goal: Explain predictability outcomes with data and propose 3 improvements with accountable owners.

> **Calculations**: All formulas defined in `../_calculation-engine.md`

---

## Data Fetch (JQL)

### Closed Sprint Data
```jql
# All items in the closed sprint
project = {TARGET} AND sprint = "{CLOSED_SPRINT_ID}"

# Done items
project = {TARGET} AND sprint = "{CLOSED_SPRINT_ID}" AND status = Done

# Rolled over (in closed sprint but not done)
project = {TARGET} AND sprint = "{CLOSED_SPRINT_ID}" AND status != Done
```

### Historical Context
```jql
# Previous 3 sprints for velocity trend
project = {TARGET} AND sprint in closedSprints() ORDER BY "Sprint" DESC
```

---

## Metrics to Compute

### 1. Predictability Scorecard

| Metric | Formula | Good | Warning | Poor |
|--------|---------|------|---------|------|
| **Completion Rate** | `done / planned * 100` | ≥ 90% | 70-89% | < 70% |
| **Scope Creep** | `added_after_start / planned * 100` | < 10% | 10-25% | > 25% |
| **Rollover Rate** | `rolled_over / planned * 100` | < 10% | 10-25% | > 25% |
| **Commitment Accuracy** | `done / (planned + added) * 100` | ≥ 85% | 65-84% | < 65% |

### 2. Flow Health Analysis

| Metric | Formula |
|--------|---------|
| **Avg Cycle Time** | `MEAN(done_date - started_date for each done item)` |
| **Blocked Time Lost** | `SUM(blocked_days for all items)` |
| **Review Queue Time** | `SUM(review_days for all items)` |
| **Stale Items** | `COUNT(items with no updates > STALE_DAYS during sprint)` |

### 3. Velocity Trend

```
# Using _calculation-engine.md § 2
velocities = [sprint.done for sprint in last_4_sprints]
velocity_trend = 
  - "📈 Improving" if current > MEAN(previous_3) * 1.05
  - "📉 Declining" if current < MEAN(previous_3) * 0.95
  - "➡️ Stable" otherwise

# Predict next sprint capacity
recommended_commitment = MEAN(velocities) * 0.9  # 10% buffer
```

### 4. Theme Analysis

Categorize issues into themes:

| Theme | Detection |
|-------|-----------|
| **Dependency Blocked** | `labels in ("blocked", "waiting") OR linked to external team` |
| **Unclear Requirements** | `labels in ("needs-clarification", "spec-unclear") OR comments mention "unclear"` |
| **Technical Complexity** | `estimate increased OR multiple rework cycles` |
| **Resource Availability** | `assignee changed OR unassigned for > 2 days` |
| **Environment/Infra** | `labels in ("environment", "infra", "deployment")` |
| **Scope Creep** | `created >= sprint_start_date` |

### 5. Chronic Rollover Detection

```
# Items that rolled over 2+ sprints
chronic_rollovers = [
  item for item in rolled_over
  if item appeared in previous_sprint AND previous_sprint.status != Done
]

# These indicate systemic issues
if len(chronic_rollovers) > 0:
  flag for root cause analysis
```

---

## Output Template

```markdown
# Sprint Retro — {TARGET} — Sprint {SPRINT}
**Period**: {START_DATE} → {END_DATE}
**Generated**: {DATE_TIME}

---

## 📊 Predictability Scorecard

| Metric | Result | Target | Status |
|--------|--------|--------|--------|
| Completion Rate | {DONE}/{PLANNED} ({PCT}%) | ≥ 90% | {🟢/🟠/🔴} |
| Scope Creep | +{ADDED} ({PCT}%) | < 10% | {🟢/🟠/🔴} |
| Rollover Rate | {ROLLED}/{PLANNED} ({PCT}%) | < 10% | {🟢/🟠/🔴} |
| Commitment Accuracy | {DONE}/{TOTAL} ({PCT}%) | ≥ 85% | {🟢/🟠/🔴} |

### Overall Grade: {A/B/C/D/F}

**Summary**: {One sentence: e.g., "Strong execution but scope creep eroded predictability"}

---

## 📈 Velocity Trend

| Sprint | Planned | Done | Completion | Trend |
|--------|---------|------|------------|-------|
| {N-3} | {X} | {Y} | {PCT}% | — |
| {N-2} | {X} | {Y} | {PCT}% | {↑/↓/→} |
| {N-1} | {X} | {Y} | {PCT}% | {↑/↓/→} |
| **{N} (this)** | **{X}** | **{Y}** | **{PCT}%** | **{↑/↓/→}** |

**Trend**: {📈 Improving / ➡️ Stable / 📉 Declining}
**Recommended next sprint commitment**: {N} items (based on 3-sprint average with 10% buffer)

---

## ⏱️ Flow Health

| Metric | This Sprint | Previous Sprint | Change |
|--------|-------------|-----------------|--------|
| Avg Cycle Time | {X} days | {Y} days | {+/-}% |
| Blocked Time Lost | {X} days | {Y} days | {+/-}% |
| Review Queue Time | {X} days | {Y} days | {+/-}% |
| Stale Items | {X} | {Y} | {+/-} |

---

## ✅ What Helped Predictability

### 1. {Positive Theme}
- **Evidence**: {specific examples}
- **Impact**: {quantified if possible}
- **Keep Doing**: {recommendation}

### 2. {Positive Theme}
- **Evidence**: {specific examples}
- **Impact**: {quantified if possible}
- **Keep Doing**: {recommendation}

### 3. {Positive Theme}
- **Evidence**: {specific examples}
- **Impact**: {quantified if possible}
- **Keep Doing**: {recommendation}

---

## ❌ What Hurt Predictability

### 1. {Negative Theme}: {Impact Score}

**Evidence**:
- {Issue keys} — {description}
- Total impact: {X} items affected, {Y} days lost

**Root Cause**: {analysis}

### 2. {Negative Theme}: {Impact Score}

**Evidence**:
- {Issue keys} — {description}
- Total impact: {X} items affected, {Y} days lost

**Root Cause**: {analysis}

### 3. {Negative Theme}: {Impact Score}

**Evidence**:
- {Issue keys} — {description}
- Total impact: {X} items affected, {Y} days lost

**Root Cause**: {analysis}

---

## 🔄 Chronic Rollovers (2+ Sprints)

| Issue | Age (Sprints) | Root Cause | Recommendation |
|-------|---------------|------------|----------------|
| {KEY} | {N} | {reason} | {action} |
| {KEY} | {N} | {reason} | {action} |

> ⚠️ **Pattern**: {If pattern detected, describe systemic issue}

---

## 🎯 3 Improvements (With Accountability)

### Improvement 1: {Title}

| Attribute | Value |
|-----------|-------|
| **Problem** | {What hurt predictability} |
| **Root Cause** | {Why it happened} |
| **Action** | {Specific change to make} |
| **Success Metric** | {How we'll know it worked} |
| **Owner** | {Name} |
| **Deadline** | {Date} |
| **Check-in** | {When to review progress} |

### Improvement 2: {Title}

| Attribute | Value |
|-----------|-------|
| **Problem** | {What hurt predictability} |
| **Root Cause** | {Why it happened} |
| **Action** | {Specific change to make} |
| **Success Metric** | {How we'll know it worked} |
| **Owner** | {Name} |
| **Deadline** | {Date} |
| **Check-in** | {When to review progress} |

### Improvement 3: {Title}

| Attribute | Value |
|-----------|-------|
| **Problem** | {What hurt predictability} |
| **Root Cause** | {Why it happened} |
| **Action** | {Specific change to make} |
| **Success Metric** | {How we'll know it worked} |
| **Owner** | {Name} |
| **Deadline** | {Date} |
| **Check-in** | {When to review progress} |

---

## 📋 Previous Improvement Tracker

| Improvement (from last retro) | Owner | Status | Outcome |
|-------------------------------|-------|--------|---------|
| {improvement_1} | {name} | {Done/In Progress/Not Started} | {result} |
| {improvement_2} | {name} | {Done/In Progress/Not Started} | {result} |
| {improvement_3} | {name} | {Done/In Progress/Not Started} | {result} |

---

## 🔮 Next Sprint Recommendations

1. **Commit to**: {N} items (based on velocity trend)
2. **Reserve capacity for**: {buffer_reason}
3. **Watch for**: {risk from this sprint likely to recur}
```

---

## Grading Scale

| Grade | Criteria |
|-------|----------|
| **A** | Completion ≥ 95%, Scope Creep < 5%, Rollover < 5% |
| **B** | Completion ≥ 85%, Scope Creep < 15%, Rollover < 15% |
| **C** | Completion ≥ 75%, Scope Creep < 25%, Rollover < 25% |
| **D** | Completion ≥ 60%, Scope Creep < 35%, Rollover < 35% |
| **F** | Completion < 60% OR Scope Creep ≥ 35% OR Rollover ≥ 35% |

---

## Error Codes

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-RT-001` | 🔴 BLOCKING | No closed sprint found | Verify sprint is closed |
| `MGR-RT-002` | 🔴 BLOCKING | Sprint has no items | Check sprint filter |
| `MGR-RT-003` | 🟠 WARNING | No velocity history (first sprint) | Omit trend analysis |
| `MGR-RT-004` | 🟠 WARNING | Sprint dates unavailable | Cannot calculate cycle time |
| `MGR-RT-005` | 🟡 INFO | Retro insights generated | Output ready |
