# Strategic Risk Mode (`/risk`)
Goal: weekly radar for threats to predictable delivery (next 1–2 weeks).

## Data Fetch (JQL)
- Active sprint + next sprint (if exists)
- Epics / initiatives in flight (if your org uses Epic Link)

## Signals to report
1) Scope creep trend (added after sprint start)
2) Rollover trend (carryover + aging WIP)
3) Dependency drag (blocked/links)
4) Review SLA breakdown (queueing)
5) Quality drag (bugs created / reopened) — only if data exists

## Output Template
```markdown
# Risk Radar — {TARGET} — Horizon {HORIZON}

## 🔴 Critical (commitment risk)
| Risk | Evidence | Impact | Mitigation | Owner |
|---|---|---|---|---|
| {risk_1} | {facts} | {impact} | {plan} | {owner} |

## 🟠 Watchlist
- {item}: {why trending}

## Leading Indicators
- Scope creep: {trend}
- Rollover candidates: {trend}
- Blocked aging: {trend}
- Review SLA breaches: {trend}

## Decisions / Asks
- {ask_1}
- {ask_2}
```
