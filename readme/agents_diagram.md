# Agents Diagram & Explanations

## 🔄 Workflow Hierarchy

```
┌─────────────────────────┐       ┌─────────────────────────┐
│  /map-codebase-agent    │       │  /system-health-agent   │
└───────────┬─────────────┘       │      (The Overseer)     │
            ↓                     └─────────────────────────┘
┌─────────────────────────┐                    |
│/system-architecture-agent│                   | (Monitors All)
└───────────┬─────────────┘                    |
            ↓                                  v
┌─────────────────────────┐
│   /engineering-agent    │
└─────────────────────────┘
            ↓
┌─────────────────────────┐
│     /manager-agent      │  ← Oversight & Risk Management
└─────────────────────────┘
```

### When to Run Each Workflow

| Workflow | Trigger | Output |
|----------|---------|--------|
| `/map-codebase-agent` | Project structure changes significantly | `.ai-instructions/` in project |
| `/system-architecture-agent` | New project added or major API changes | `system-architecture/` docs |
| `/engineering-agent` | Any feature work, bug fixes | Jira tasks, code, tests |
| `/manager-agent` | Daily standup, Weekly sync, Executive reporting | Risk reports, status briefs |
| `/system-health-agent` | System setup, diagostics, project status | Health report, recommendations |

---

## ✅ Quick Start (Daily Use)

### Start Feature Work
```
/engineering-agent
```
Guides you through: ProductSpecReview → DesignAnalysis → FeaturePlanning → TechSpec → TaskPlanning → Implementation

### Approval Gates (7 Total)

| # | Gate | Artifact | Action |
|---|------|----------|--------|
| 1 | ProductSpecReview | Gap Analysis | User selects option (post/answer/assume) |
| 2 | DesignAnalysis | Design Report | Approve design tokens *(skip if no Figma)* |
| 3 | FeaturePlanning | Jira Epic | Approve Epic |
| 4 | TechSpec | Tech Spec → Jira Task | Approve + Inject to Jira |
| 5a | TaskPlanning: Presentation | Proposed task list | Approve before Jira creation |
| 5b | TaskPlanning: Pre-Impl | Traceability matrix | Confirm scope |
| 5c | TaskPlanning: Final | Summary + handoff | Select first task |

### Refresh Project Knowledge (When Needed)
```
/map-codebase-agent
```
Run on a specific project when its structure changes.

### Refresh System Architecture (When Needed)
```
/system-architecture-agent
```
Run after adding new projects or major cross-service changes.

### Manager Oversight (Daily/Weekly)
```
/manager-agent
```
Run to check team pulse (`/beat`) or assess delivery risk (`/risk`).
