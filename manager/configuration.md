# Manager Agent Configuration (Predictable Delivery)

## 1) Inheritance
Read:
- `../shared/configuration.md`
- `../shared/mcp-config.md`

## 2) Default Target
If user doesn’t specify `Target`, use:
- `${JIRA_PROJECT_KEY}` (from shared config) or set manually here.

## 3) Heuristics (Predictability levers)
| Key | Default | Meaning |
|---|---:|---|
| STALE_DAYS | 4 | No meaningful updates → rotting |
| MAX_WIP | 3 | WIP limit per dev |
| REVIEW_SLA_HOURS | 48 | “In Review” too long |
| BLOCKED_DAYS | 2 | Blocked too long |
| SCOPE_CREEP_WARN | 0.15 | 15%+ added after sprint start → 🟠 |
| SCOPE_CREEP_CRIT | 0.25 | 25%+ → 🔴 |
| ROLLOVER_WARN | 0.20 | rollover candidates 20%+ → 🟠 |
| ROLLOVER_CRIT | 0.35 | 35%+ → 🔴 |
| AGING_WIP_DAYS | 4 | In Progress > 4 days → rollover candidate |

## 4) Jira Conventions (edit to match your board)
### Status buckets
- TODO categories: `To Do`
- IN_PROGRESS categories: `In Progress`
- DONE categories: `Done`

### “In Review” identification (any match)
- IN_REVIEW_STATUSES: `["In Code Review"]`
- IN_REVIEW_LABELS: `["review"]` (optional)

### Blocked source-of-truth (robust default)
Treat an issue as BLOCKED if any:
- Status in: `["Blocked", "On Hold"]`
- Labels include any: `["blocked", "needs-input", "waiting"]`
- Jira “Flagged” field is set (if available)

(If you want it strict, remove options you don’t use.)

## 5) Confidence Rules (explainable)
Start 🟢, then:
- 🟠 if any: scope creep > SCOPE_CREEP_WARN OR rollover candidates > ROLLOVER_WARN OR blocked count is rising
- 🔴 if any: scope creep > SCOPE_CREEP_CRIT OR rollover candidates > ROLLOVER_CRIT OR blocked items on critical path > BLOCKED_DAYS
