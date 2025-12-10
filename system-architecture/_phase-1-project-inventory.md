# Phase 1: Project Inventory

> **Goal**: Verify all registered projects have AI instructions and extract project summaries.

---

## Input

- Project Registry from `configuration.md`

## Output

- `${ANALYSIS_DIR}/project-inventory.json`

---

## Instructions

### Step 1: Read Configuration

Load the project registry from `${GLOBAL_WORKFLOWS_ROOT}/shared/projects.md`.

### Step 2: Scan Each Project

For each project in the registry:

1. **Check AI Instructions Exist**

   > [!CAUTION]  
   > **Do NOT use** `find_by_name` to search for `.ai-instructions/` directories.  
   > These directories are typically gitignored and will NOT be found by glob-based search tools.
   >
   > **Use direct path access instead:**
   > - Construct path: `${PROJECT_PATH}/.ai-instructions/`
   > - Use `list_dir("${PROJECT_PATH}/.ai-instructions/")` to verify existence
   > - If the directory exists, check for specific files

   - Verify `.ai-instructions/` directory exists (via direct `list_dir`)
   - Verify `copilot-instructions.md` exists
   - Verify `analysis/entity-contracts.json` exists
   - Verify `analysis/api-contracts.json` exists

2. **Extract Project Summary**
   - Read `copilot-instructions.md`
   - Extract:
     - Project description (first paragraph or "Overview" section)
     - Tech stack (from "Tech Stack" or detect from content)
     - Architecture pattern (if documented)

3. **Record Status**
   - `status: "ready"` – All required files present
   - `status: "incomplete"` – Some files missing (list which)
   - `status: "missing"` – No `.ai-instructions/` at all

### Step 3: Generate Output

```json
{
  "generatedAt": "ISO timestamp",
  "totalProjects": 8,
  "readyProjects": 7,
  "incompleteProjects": 1,
  "projects": [
    {
      "name": "wg-client",
      "type": "Frontend",
      "role": "Next.js web application",
      "path": "C:\\My Projects\\WG3\\wg-client",
      "status": "ready",
      "summary": "Frontend application built with Next.js 14...",
      "techStack": {
        "framework": "Next.js 14",
        "language": "TypeScript",
        "stateManagement": "Zustand",
        "styling": "Tailwind CSS"
      },
      "aiInstructionsPath": ".ai-instructions/",
      "files": {
        "copilotInstructions": true,
        "entityContracts": true,
        "apiContracts": true,
        "fileCategorization": true
      }
    },
    {
      "name": "wg-data-api",
      "type": "Backend",
      "role": "Site data and configuration",
      "path": "C:\\My Projects\\WG3\\wg-data-api",
      "status": "ready",
      "summary": "Spring Boot 3.0 REST API...",
      "techStack": {
        "framework": "Spring Boot 3.0",
        "language": "Java 17",
        "database": "PostgreSQL"
      },
      "files": {
        "copilotInstructions": true,
        "entityContracts": true,
        "apiContracts": true
      }
    }
  ],
  "_issues": [
    {
      "project": "new-project",
      "status": "missing",
      "message": "Run /map-codebase-agent on this project first"
    }
  ]
}
```

---

## Validation

Before proceeding to Phase 2:

| Check | Requirement |
|-------|-------------|
| All projects scanned | Count matches registry |
| No `missing` projects | Or explicitly acknowledged |
| Ready projects ≥ 1 | At least one project to analyze |

**If validation fails**: Stop and report which projects need `/map-codebase-agent`.

---

## Error Handling

| Scenario | Action |
|----------|--------|
| Project path doesn't exist | Mark as `missing`, log error |
| `copilot-instructions.md` unreadable | Mark as `incomplete` |
| JSON files malformed | Mark as `incomplete`, note parse error |
