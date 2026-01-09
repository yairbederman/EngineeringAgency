---
description: Validates global_workflows setup and configuration before running agents
---

# Verify Setup Workflow

**Purpose**: Validate that `global_workflows` is correctly installed and `Agent_Config/agent-config.md` is configured.

---

## Step 1: Check Installation Path

Verify workflows are in the correct directory for auto-discovery:

**Expected paths:**
- macOS/Linux: `~/.gemini/antigravity/global_workflows/`
- Windows: `%USERPROFILE%\.gemini\antigravity\global_workflows\`

**Validation:**
- [ ] `README.md` exists in the workflows directory
- [ ] `engineering-agent.md` exists and is readable
- [ ] `readme/agent-config.template.md` exists

> [!CAUTION]
> If workflows are not in `.gemini/antigravity/global_workflows`, slash commands will NOT be discovered. Move the directory to the correct path.

---

## Step 2: Check Workspace Configuration

> [!IMPORTANT]
> **Configuration lives in your workspace**, not in `global_workflows`.

Check if `agent-config.md` exists in current workspace:

**Validation checklist:**
- [ ] `Agent_Config/agent-config.md` exists in workspace
- [ ] `SYSTEM_NAME` is set (not `<YOUR_SYSTEM_NAME>`)
- [ ] `STORAGE_BACKEND` is set (`local` or `atlassian`)

**Commands to locate config:**
```bash
# macOS/Linux:
cat ./Agent_Config/agent-config.md | head -30

# Windows (PowerShell):
Get-Content .\Agent_Config\agent-config.md | Select-Object -First 30
```

**If `agent-config.md` is missing:**
```bash
# Copy template to workspace:
# macOS/Linux:
mkdir -p Agent_Config
cp ~/.gemini/antigravity/global_workflows/readme/agent-config.template.md ./Agent_Config/agent-config.md

# Windows (PowerShell):
New-Item -ItemType Directory -Force -Path Agent_Config
Copy-Item $env:USERPROFILE\.gemini\antigravity\global_workflows\readme\agent-config.template.md .\Agent_Config\agent-config.md
```

---

## Step 3: Validate Atlassian Configuration (Optional)

Skip this step if `STORAGE_BACKEND = local` in your `agent-config.md`.

Check `agent-config.md` Atlassian section:

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

Check that workspace is properly configured:

**Validation:**
- [ ] Current directory is a workspace root (contains projects)
- [ ] At least one project folder exists
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

```
✅ Setup Verification Checklist
================================

[ ] Installation Path
    └── global_workflows in .gemini/antigravity/

[ ] Workspace Configuration
    ├── Agent_Config/agent-config.md exists
    ├── SYSTEM_NAME set (not placeholder)
    └── STORAGE_BACKEND set (local/atlassian)

[ ] Atlassian Configuration (if STORAGE_BACKEND=atlassian)
    ├── Cloud ID set
    ├── Jira Project Key set
    ├── Confluence Space Key set
    └── MCP server authenticated

[ ] Workspace Structure
    └── Projects in IDE workspace

[ ] Agent Discovery
    └── /engineering-agent recognized
```

---

## Troubleshooting Quick Reference

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| "Workspace Configuration Required" | No `agent-config.md` | Copy template to Agent_Config/ (see Step 2) |
| Slash commands not found | Wrong install path | Move to `.gemini/antigravity/global_workflows/` + restart IDE |
| "Placeholder value" errors | Config not updated | Edit `Agent_Config/agent-config.md` |
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
