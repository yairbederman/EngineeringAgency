# Team Beat Mode (`/beat`)
Goal: daily sprint health focused on **commitment accuracy**.

## Data Fetch (JQL baseline)
```jql
project = {TARGET} AND sprint in openSprints()
```

Optional (if your Jira supports it well):

* Stale: `updated < -{STALE_DAYS}d`
* In review: `status in ({IN_REVIEW_STATUSES})`
* Blocked: `status in ("Blocked","On Hold") OR labels in ("blocked","needs-input","waiting") OR Flagged is not EMPTY`

## Rollover Candidates (definition)

An issue is a rollover candidate if any:

* In Progress > {AGING_WIP_DAYS} days
* Blocked > {BLOCKED_DAYS} days
* In Review > {REVIEW_SLA_HOURS} hours
* Has blocking link unresolved (if links used)

## Output Template

```markdown
# Team Beat — {TARGET} — Sprint {SPRINT}

## Delivery Confidence: {🟢/🟠/🔴}
Why:
- {evidence_1}
- {evidence_2}

## Commitment Snapshot
- Planned: {PLANNED} | Done: {DONE} | Remaining: {REMAINING} | Days left: {DAYS_LEFT}
- Scope creep: {ADDED_AFTER_START} ({SCOPE_CREEP_RATIO})
- Rollover candidates: {ROLLOVER_CANDIDATES} ({ROLLOVER_CANDIDATE_RATIO})
- Flow issues: Blocked {BLOCKED_COUNT} | Review > SLA {REVIEW_BREACH_COUNT} | Stale {STALE_COUNT}

## Top 3 Threats (with interventions)
1) {THREAT}
   - Evidence: {facts}
   - Intervention: {action} (Owner: {owner}, by: {date})
2) ...
3) ...

## Today’s Minimal Focus (max 3)
- Unblock: {item/area}
- Finish: {item/area}
- Review: {item/area}

## Wins (last 24–48h)
- {win_1}
- {win_2}
```
