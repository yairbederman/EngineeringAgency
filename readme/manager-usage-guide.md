# Manager Agent — Usage Guide

> **Role**: Engineering Lead's Co-Pilot for Predictable Delivery  
> **Trigger**: `/manager-agent`

---

## Quick Start

| What You Need | Command |
|---------------|---------|
| Daily sprint health | `/manager-agent` |
| Weekly risk radar | `/manager-agent /risk` |
| Stakeholder status | `/manager-agent /status` |
| Sprint retrospective | `/manager-agent /retro Sprint: last_closed` |

---

## Modes Overview

### `/beat` — Daily Sprint Health (Default)

**Purpose**: Commitment accuracy check for daily standups or async updates.

**Output includes**:
- 📊 **Delivery Confidence Score** (0-100 with breakdown)
- 📈 **Commitment Snapshot** (done/planned/remaining/days left)
- ⚠️ **Top 3 Threats** (with owner + intervention + deadline)
- 🎯 **Today's Focus** (unblock, finish, review)
- ✅ **Wins** (last 24-48h)
- 👥 **WIP Distribution** (per developer)

**Example**:
```
/manager-agent
Target: PROJ
```

---

### `/risk` — Weekly Risk Radar

**Purpose**: Identify threats to predictable delivery with quantified risk scores.

**Output includes**:
- 🔴 **Critical Risks** (score ≥ 13, with mitigations)
- 🟠 **Watchlist** (trending negative)
- 📊 **Leading Indicators** (scope creep, rollover, blocked, review trends)
- 🔗 **Dependency Map** (external blockers)
- ✋ **Decisions Needed** (with deadlines)

**Risk Scoring**: `Likelihood (1-5) × Impact (1-5)`
- 1-6: Low (monitor)
- 7-12: Medium 🟠
- 13-19: High 🔴
- 20-25: Critical 🔴🔴

**Example**:
```
/manager-agent /risk
Target: PROJ
Horizon: 14d
```

---

### `/status` — Stakeholder Status Updates

**Purpose**: Right-sized updates calibrated to audience.

**Audience Variants**:

| Audience | Length | Focus |
|----------|--------|-------|
| `team` (default) | 8-12 lines | Done, in flight, blockers, next focus |
| `eng_manager` | 20-30 lines | Metrics table, risks, asks |
| `exec` | 5 lines | Business outcomes, timeline, decisions |

**Examples**:
```
# Team update (default)
/manager-agent /status
Target: PROJ

# Engineering manager update
/manager-agent /status
Target: PROJ
Audience: eng_manager

# Executive update
/manager-agent /status
Target: PROJ
Audience: exec
```

---

### `/retro` — Sprint Retrospective

**Purpose**: Explain predictability outcomes and propose 3 accountable improvements.

**Output includes**:
- 📊 **Predictability Scorecard** (completion, scope creep, rollover, accuracy)
- 📈 **Velocity Trend** (last 4 sprints with recommendation)
- ✅ **What Helped Predictability** (3 themes with evidence)
- ❌ **What Hurt Predictability** (3 themes with root cause)
- 🔄 **Chronic Rollovers** (items rolled 2+ sprints)
- 🎯 **3 Improvements** (with owner, deadline, success metric)
- 📋 **Previous Improvement Tracker** (status of last retro's actions)

**Grading Scale**:
- **A**: Completion ≥ 95%, Scope Creep < 5%, Rollover < 5%
- **B**: Completion ≥ 85%, Scope Creep < 15%, Rollover < 15%
- **C**: Completion ≥ 75%, Scope Creep < 25%, Rollover < 25%
- **D**: Completion ≥ 60%, Scope Creep < 35%, Rollover < 35%
- **F**: Below D thresholds

**Example**:
```
/manager-agent /retro
Target: PROJ
Sprint: last_closed
```

---

## Delivery Confidence Score

Every output shows an **explainable** confidence score:

```
Delivery Confidence: 🟠 73/100

| Factor | Value | Penalty |
|--------|-------|---------|
| Scope Creep | 18% | 8 pts |
| Rollover Candidates | 22% | 7 pts |
| Blocked Issues | 3 | 9 pts |
| Review SLA Breaches | 2 | 4 pts |
| **Total Penalty** | | **28 pts** |
```

**Score Bands**:
- 🟢 **80-100**: High confidence (on track)
- 🟠 **60-79**: Medium confidence (at risk)
- 🔴 **0-59**: Low confidence (off track)

---

## Input Parameters

### Required

| Parameter | Description | Examples |
|-----------|-------------|----------|
| `Target` | Jira project key or board ID | `PROJ`, `board=123` |

### Optional

| Parameter | Default | Options | Purpose |
|-----------|---------|---------|---------|
| `Sprint` | `active` | `active`, `last_closed`, `sprint=456` | Which sprint to analyze |
| `Audience` | `team` | `team`, `eng_manager`, `exec` | Output calibration |
| `Focus` | `delivery` | `delivery`, `quality`, `deps`, `people` | What to emphasize |
| `Horizon` | `7d` | `7d`, `14d`, `quarter` | Lookahead window |

---

## Customizing Thresholds

Edit `manager/configuration.md`:

| Setting | Default | When to Adjust |
|---------|---------|----------------|
| `SCOPE_CREEP_WARN` | 15% | Your team tolerates different scope changes |
| `SCOPE_CREEP_CRIT` | 25% | Adjust critical threshold |
| `ROLLOVER_WARN` | 20% | Different rollover tolerance |
| `ROLLOVER_CRIT` | 35% | Adjust critical threshold |
| `MAX_WIP` | 3 | Developers handle more/fewer items |
| `REVIEW_SLA_HOURS` | 48h | Different code review cadence |
| `BLOCKED_DAYS` | 2 | Different blocked tolerance |
| `STALE_DAYS` | 4 | Different activity expectations |

---

## Jira Configuration

### Status Mapping

In `manager/configuration.md`, map your Jira statuses:

```yaml
# TODO bucket
TODO: ["To Do", "Open", "Backlog"]

# IN_PROGRESS bucket  
IN_PROGRESS: ["In Progress", "In Development"]

# IN_REVIEW bucket
IN_REVIEW: ["In Code Review", "In QA"]

# BLOCKED bucket
BLOCKED: ["Blocked", "On Hold"]

# DONE bucket
DONE: ["Done", "Closed", "Resolved"]
```

### Blocked Detection

Configure how blocked issues are detected:

```yaml
# Any of these = blocked
BLOCKED_STATUSES: ["Blocked", "On Hold"]
BLOCKED_LABELS: ["blocked", "needs-input", "waiting"]
BLOCKED_FIELD: "Flagged"  # If Jira's Flagged field is set
```

---

## Example Session

```
User: /manager-agent
       Target: PROJ

Agent: 
# Team Beat — PROJ — Sprint 14
**Generated**: 2026-01-06 00:25

---

## Delivery Confidence: 🟢 85/100

### Score Breakdown
| Factor | Value | Threshold | Penalty |
|--------|-------|-----------|---------|
| Scope Creep | 8% | WARN: 15% | 0 |
| Rollover Candidates | 12% | WARN: 20% | 0 |
| Blocked Issues | 2 (0 aging) | DAYS: 2 | 6 |
| Review SLA Breaches | 2 | SLA: 48h | 4 |
| **Total Penalty** | | | **10** |

### Primary Drivers
1. ✅ Scope under control (< 10%)
2. ✅ Rollover candidates below warning
3. 🟠 2 items stuck in code review

---

## Commitment Snapshot

| Metric | Value | Status |
|--------|-------|--------|
| Planned | 22 | — |
| Done | 14 | 64% |
| Remaining | 8 | 36% |
| Days Left | 4 | — |
| Required Daily Rate | 2.0 items/day | 🟢 |

...
```

---

## Error Handling

The agent uses structured error codes:

| Code | Meaning |
|------|---------|
| `MGR-BT-001` | Jira query failed in /beat |
| `MGR-RS-001` | Jira query failed in /risk |
| `MGR-ST-001` | Jira query failed in /status |
| `MGR-RT-001` | No closed sprint found in /retro |
| `MGR-*-002` | Sprint data incomplete |
| `MGR-*-003` | No velocity history available |

If an error occurs, the agent will:
1. Emit the error code
2. Explain what data is missing
3. Ask for manual input or alternative

---

## Related Files

| File | Purpose |
|------|---------|
| [`manager-agent.md`](../manager-agent.md) | Main agent definition |
| [`manager/configuration.md`](../manager/configuration.md) | Thresholds & Jira config |
| [`manager/_calculation-engine.md`](../manager/_calculation-engine.md) | All metric formulas |
| [`manager/modes/team-beat.md`](../manager/modes/team-beat.md) | /beat template |
| [`manager/modes/strategic-risk.md`](../manager/modes/strategic-risk.md) | /risk template |
| [`manager/modes/status-report.md`](../manager/modes/status-report.md) | /status template |
| [`manager/modes/sprint-retro.md`](../manager/modes/sprint-retro.md) | /retro template |
| [`shared/error-codes.md`](../shared/error-codes.md) | Error code reference |
