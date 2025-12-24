# Manager Agent Configuration

> **Purpose**: Defines manager-specific settings while inheriting global defaults.

---

## 1. Global Inheritance

**Read**: `../shared/configuration.md`

Inherits:
*   `${ATLASSIAN_CLOUD_ID}`
*   `${JIRA_PROJECT_KEY}` (Default target for Pulse checks)
*   `${CONFLUENCE_SPACE_KEY}`

---

## 2. Manager Output Paths

| Variable | Path | Description |
|----------|------|-------------|
| `${PULSE_REPORT_DIR}` | `./reports/pulse` | Daily Pulse reports output location |
| `${RISK_REPORT_DIR}` | `./reports/risk` | Strategic Risk analysis reports |

---

## 3. Threshold Configuration (Heuristics)

| Variable | Value | Description |
|----------|-------|-------------|
| `STALE_DAYS` | `4` | Days without update to flag as "Rotting" |
| `MAX_WIP` | `3` | Max concurrent tickets per dev before "Overload" flag |
| `REVIEW_SLA_HOURS` | `48` | Hours in "In Review" before nagging |
