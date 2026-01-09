---
description: Validates global_workflows setup and configuration before running agents
---

# Verify Setup Workflow

**Purpose**: Validate that `global_workflows` is correctly installed and configured.

---

## Step 1: Check Installation Path

Verify workflows are in the correct directory for auto-discovery:

**Expected paths:**
- macOS/Linux: `~/.gemini/antigravity/global_workflows/`
- Windows: `%USERPROFILE%\.gemini\antigravity\global_workflows\`

**Validation:**
- [ ] `README.md` exists in the workflows directory
- [ ] `engineering-agent.md` exists and is readable
- [ ] `shared/configuration.md` exists

> [!CAUTION]
> If workflows are not in `.gemini/antigravity/global_workflows`, slash commands will NOT be discovered. Move the directory to the correct path.

---

## Step 2: Validate Core Configuration

Check `shared/configuration.md` for required values:

**Required settings:**
| Setting | Status Check |
|---------|--------------|
| `SYSTEM_NAME` | Must NOT be `<YOUR_SYSTEM_NAME>` |
| `WORKSPACE_ROOT` | Must be a valid, existing path |

**Validation script (run mentally or via command):**
```bash
# macOS/Linux:
cat ~/.gemini/antigravity/global_workflows/shared/configuration.md | grep -E "(SYSTEM_NAME|WORKSPACE_ROOT)"

# Windows (PowerShell):
Get-Content $env:USERPROFILE\.gemini\antigravity\global_workflows\shared\configuration.md | Select-String -Pattern "SYSTEM_NAME|WORKSPACE_ROOT"
```

**Expected output:**
- `SYSTEM_NAME` should be your actual system name (e.g., `MySystem`)
- `WORKSPACE_ROOT` should be your actual projects path (e.g., `C:/Projects/MySystem`)

> [!WARNING]
> Placeholder values like `<YOUR_SYSTEM_NAME>` will cause agent errors. Replace all placeholders before proceeding.

---

## Step 3: Validate Atlassian Configuration (Optional)

Skip this step if working in **local-only mode** (no Jira/Confluence integration).

Check `shared/atlassian-config.md`:

| Setting | Status Check |
|---------|--------------|
| `ATLASSIAN_CLOUD_ID` | Must NOT be `<YOUR_ORG>.atlassian.net` |
| `JIRA_PROJECT_KEY` | Must NOT be `<PROJECT_KEY>` |
| `CONFLUENCE_SPACE_KEY` | Must NOT be `<SPACE_KEY>` |

**MCP Server Test:**
```
# In your AI assistant, run:
mcp0_getAccessibleAtlassianResources
```

Expected: Returns your Atlassian Cloud ID without errors.

---

## Step 4: Validate Workspace Structure

Check that project folders are accessible:

**Validation:**
- [ ] `WORKSPACE_ROOT` directory exists
- [ ] At least one project folder exists inside `WORKSPACE_ROOT`
- [ ] Project folders are added to VS Code/Cursor workspace

---

## Step 5: Validate Agent Discovery

After IDE restart, verify agents are available:

**Test commands:**
- [ ] `/engineering-agent` — should be recognized
- [ ] `/map-codebase-agent` — should be recognized
- [ ] `/system-architecture-agent` — should be recognized
- [ ] `/manager-agent` — should be recognized

> [!TIP]
> If commands are not recognized, restart your IDE. The AI assistant discovers workflows on startup.

---

## Verification Summary

Run this checklist to confirm setup is complete:

```
✅ Setup Verification Checklist
================================

[ ] Installation Path
    └── global_workflows in .gemini/antigravity/

[ ] Core Configuration (shared/configuration.md)
    ├── SYSTEM_NAME set (not placeholder)
    └── WORKSPACE_ROOT set (valid path)

[ ] Atlassian Configuration (optional)
    ├── Cloud ID set
    ├── Jira Project Key set
    ├── Confluence Space Key set
    └── MCP server authenticated

[ ] Workspace Structure
    ├── WORKSPACE_ROOT exists
    └── Projects in IDE workspace

[ ] Agent Discovery
    └── /engineering-agent recognized
```

---

## Troubleshooting Quick Reference

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Slash commands not found | Wrong install path | Move to `.gemini/antigravity/global_workflows/` + restart IDE |
| "Placeholder value" errors | Config not updated | Edit `shared/configuration.md` |
| MCP authentication errors | MCP server not configured | Re-authenticate in IDE settings |
| "Project not found" errors | Project not registered | Run `/map-codebase-agent` on the project |

---

## All Clear?

If all checks pass:
```
🎉 Setup verified! You're ready to use:

/engineering-agent           — Feature development lifecycle
/map-codebase-agent          — Generate project AI context  
/system-architecture-agent   — Cross-project architecture
/manager-agent               — Sprint health & delivery oversight
```
