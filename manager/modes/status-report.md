# Status Mode (`/status`)
Goal: Executive-ready status update (8-12 lines) for stakeholder communication.

## Audience Variants

### `/status team` (Default)
- Focus: What got done, what's next, blockers
- Tone: Informal, action-oriented
- Format: Bullet points

### `/status eng_manager`
- Focus: Delivery confidence, risks, asks
- Tone: Direct, metrics-driven
- Format: Structured with metrics table

### `/status exec`
- Focus: Business outcomes, timeline, dependencies
- Tone: High-level, impact-oriented
- Format: 3-5 key points only

---

## Data Fetch (JQL)
```jql
# Same as /beat mode
project = {TARGET} AND sprint in openSprints()
```

## Calculations Required

1. **Delivery Confidence Score** (from `_calculation-engine.md` § 1)
2. **Completion Rate**: `done / planned * 100`
3. **Remaining Work**: `remaining / total * 100`
4. **Key Blockers**: Top 3 blocked items with owners

---

## Output Templates

### Team Status

```markdown
# Sprint {SPRINT} Status — {DATE}

**Progress**: {DONE}/{PLANNED} complete ({COMPLETION_PCT}%)
**Confidence**: {🟢/🟠/🔴} {SCORE}/100

## ✅ Completed (last 48h)
- {completed_1}: [Brief description]
- {completed_2}: [Brief description]

## 🎯 In Flight
- {in_progress_1}: [Assignee] — ETA {date}
- {in_progress_2}: [Assignee] — ETA {date}

## ⚠️ Blockers ({COUNT})
- {blocker_1}: Waiting on {dependency} — Owner: {owner}

## 📅 Next 48h Focus
- {focus_1}
- {focus_2}
```

### Engineering Manager Status

```markdown
# Sprint {SPRINT} — Delivery Report

| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| Completion | {DONE}/{PLANNED} | 100% | {🟢/🟠/🔴} |
| Scope Creep | {PCT}% | <15% | {🟢/🟠/🔴} |
| Rollover Risk | {PCT}% | <20% | {🟢/🟠/🔴} |
| Blocked | {COUNT} | 0 | {🟢/🟠/🔴} |
| Review Stalled | {COUNT} | 0 | {🟢/🟠/🔴} |

## Delivery Confidence: {🟢/🟠/🔴} {SCORE}/100

**Drivers**:
1. {driver_1}
2. {driver_2}

## Top 3 Risks
1. **{Risk}**: {Impact} — Mitigation: {plan} — Owner: {name}
2. ...
3. ...

## Asks
- {ask_1}
- {ask_2}
```

### Executive Status

```markdown
# {PROJECT_NAME} — Week {N} Update

## 📊 Delivery: {🟢/🟠/🔴}

**Summary**: {One sentence: on track / at risk / behind}

## Key Accomplishments
- {Business outcome 1}
- {Business outcome 2}

## Risks to Timeline
- {Risk 1}: {Impact on milestone} — {Mitigation}

## Decisions Needed
- {Decision 1}

## Next Milestone
{Milestone name}: {Date} — {Confidence level}
```

---

## Evidence Rules

For each status update:

1. **Completion numbers MUST match Jira** — No rounding until final display
2. **Blockers MUST have owners** — If no owner, note "Unassigned ⚠️"
3. **ETAs MUST be grounded** — Based on velocity or explicit commitment
4. **Risks MUST have evidence** — At least one Jira issue or metric

---

## Error Codes

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-ST-001` | 🔴 BLOCKING | Jira query failed | Check Jira connectivity |
| `MGR-ST-002` | 🟠 WARNING | Sprint data incomplete | Mark as partial report |
| `MGR-ST-003` | 🟠 WARNING | No velocity history (first sprint) | Omit velocity comparisons |
| `MGR-ST-004` | 🟡 INFO | Status report generated | Output ready |
