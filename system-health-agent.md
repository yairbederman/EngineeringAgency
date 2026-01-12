---
description: High-level System & Workspace Health Monitor
---

# System Health Agent

**Trigger**: `/system-health-agent`
**Role**: "The Overseer"
**Mandate**: Provide a holistic view of the Workspace, Project Status, and Configuration Health.

---

## 🏗️ Phase 1: Environment Health (Setup Check)

**Goal**: Ensure the "machine" is oiled and running.

### 1.1 Installation Verification
*   **Path Check**: Verifies `global_workflows` is in `.gemini/antigravity/`.
*   **Config Existence**: Checks for `Agent_Config/agent-config.md`.

### 1.2 Configuration Validation
*   **Fields**: Checks `SYSTEM_NAME`, `STORAGE_BACKEND`.
*   **Connectivity**: Tests Atlassian MCP connection (if applicable).

### 1.3 Agent Discovery
*   **Slash Commands**: Verifies `/engineering-agent`, `/map-codebase-agent`, `/manager-agent` are discoverable.

---

## 📊 Phase 2: Project Status (The "Pulse")

**Goal**: A high-level dashboard of *what is happening* right now.

### Input
*   **Source**: Reads `Agent_Config/agent-config.md` to find registered projects.
*   **Artifacts**: Scans `.specs/` and `.tasks/` (or Jira via Manager) for active items.

### Output: The Health Report
```markdown
# 🏥 System Health Report

## Workspace: [System Name]
**Backend**: [Local/Atlassian]

### 1. Active Projects
| Project | Role | Tech Stack | Last Activity |
|---------|------|------------|---------------|
| `api-service` | Backend | Node.js | 2h ago |
| `web-client` | Frontend | React | 1d ago |

### 2. Work in Progress
**Active Epics**:
*   [EPIC-001] User Authentication (Status: In Progress, 65% Complete)
*   [EPIC-002] Payment Gateway (Status: Planning, Gate 3)

**Blockers Detected**:
*   Task [TASK-104]: Missing Verification Evidence.
```

---

## 💡 Phase 3: Strategic Recommendations

**Goal**: Proactive advice based on gaps.

### Logic
The agent compares *Current State* vs *Ideal State* (Constitution).

| Condition | Recommendation |
|-----------|----------------|
| Project exists but no `.ai-instructions` | "Run `/map-codebase-agent` on [Project] to generate context." |
| Epic in progress but no recent Specs | "Review `specs/` folder; documentation may be stale." |
| High Verify Failure Rate | "Engineering Agent is failing Gate 5.9 often. Check test environment." |
| New Folder Detected | "Found `new-service/`; run Project Init to register it." |

---

## Usage

### Default Run
```
/system-health-agent
```
→ Runs full diagnostic: Setup + Status + Recommendations.

### Specific Checks
```
/system-health-agent --setup-only
/system-health-agent --status-only
```

---
