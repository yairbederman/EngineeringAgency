# Engineering Agent – TechStackDecision Mode

> **Persona**: Load `${AGENT_ROOT}/personas/system-architect.md`

## Purpose

Determine the appropriate technology stack for implementation. This mode handles two distinct scenarios:
- **Existing Projects**: Inherit stack from project analysis
- **New Projects**: Interactive decision with user

---

## Trigger Conditions

| Condition | Trigger |
|-----------|---------|
| Epic creates **NEW project/codebase** | Gate 3.25 → TechStackDecision |
| Epic modifies **EXISTING project** | Skip → TechSpec (stack auto-detected) |

---

## Flow: Existing Project (Auto-Detection)

> **Skip Condition**: When Epic targets an existing codebase, tech stack is inherited.

### Step 1: Detect Project Stack

1. Read `${COPILOT_INSTRUCTIONS_PATH}` if available
2. Scan root configuration files:
   - `package.json` → Node.js/JavaScript ecosystem
   - `pom.xml` / `build.gradle` → Java ecosystem
   - `requirements.txt` / `pyproject.toml` → Python ecosystem
   - `go.mod` → Go ecosystem
   - `Cargo.toml` → Rust ecosystem
3. Document detected stack in Tech Spec

### Step 2: Confirm Detection

```markdown
✅ **Tech Stack Auto-Detected**

| Layer | Technology |
|-------|------------|
| **Runtime** | [Node.js 20 / Python 3.11 / etc.] |
| **Framework** | [Next.js / FastAPI / etc.] |
| **Styling** | [Tailwind / CSS Modules / etc.] |
| **Testing** | [Jest / Pytest / etc.] |

**Source**: [package.json / pom.xml / etc.]

> Proceeding to TechSpec with detected stack.
```

**No gate approval required** – proceed automatically to TechSpec.

---

## Flow: New Project (Interactive Decision)

> **Trigger**: Epic involves creating a new project/codebase.

### Step 1: Gather Context from ProductRoadmap

Read the ProductRoadmap summary from Gate 3.25:
- Future features planned (CMS, auth, e-commerce, etc.)
- Expected traffic scale
- Maintenance team expertise

### Step 2: Present Stack Options

Present 2-4 viable options based on roadmap context:

```markdown
## 🔧 Tech Stack Decision Required

Based on your roadmap, here are the recommended options:

### Option A: [Recommended Stack Name]
| Layer | Technology | Rationale |
|-------|------------|-----------|
| Runtime | [e.g., Node.js 22] | [Why this fits] |
| Framework | [e.g., Next.js 15] | [Why this fits] |
| Styling | [e.g., Tailwind CSS] | [Why this fits] |
| Database | [e.g., PostgreSQL] | [Why this fits] |

**Best for**: [Summary of ideal use case]
**Trade-offs**: [Any limitations]

---

### Option B: [Alternative Stack Name]
[Same format]

---

### Option C: [Lightweight Alternative]
[Same format]

---

> **⏸️ SELECT**: Reply with `A`, `B`, `C`, or request alternatives.
```

### Step 3: Capture Decision

On user selection:

1. **Document in Epic's Decisions Log**:
   ```markdown
   ## Decisions Log
   
   ### Tech Stack Decision (Gate 3.5)
   - **Date**: [timestamp]
   - **Selected**: Option [X] - [Stack Name]
   - **Rationale**: [User's stated reason or "per recommendation"]
   - **Alternatives Considered**: [List other options]
   ```

2. **Update ProductRoadmap** (if exists) with tech stack choice

3. **Proceed to TechSpec** with chosen technology configured

---

## Gate 3.5: Approval Format

```markdown
✅ **TechStackDecision Complete**

- **Selected Stack**: [Stack Name]
- **Documented In**: [Epic link/path] → Decisions Log
- **Next Step**: TechSpec

> **⏸️ APPROVAL REQUIRED**: Reply `Approve` to proceed to TechSpec.
```

---

## Stack Recommendation Guidelines

### For Static Sites / Marketing Pages
- **Recommend**: Next.js (static export) or Astro
- **Avoid**: Heavy frameworks without SSG capability

### For Full-Stack Web Apps
- **Recommend**: Next.js (App Router) or Vite + Backend
- **Consider**: User's team familiarity

### For API-First / Backend Services
- **Recommend**: Node.js (Express/Fastify) or Python (FastAPI)
- **Consider**: Existing infrastructure

### For Enterprise / Complex Apps
- **Recommend**: Monorepo with clear boundaries
- **Consider**: Long-term maintenance, team size

---

## Error Handling

| Scenario | Response |
|----------|----------|
| User requests unlisted stack | Present pros/cons, allow if viable |
| Conflicting roadmap requirements | Flag trade-offs, let user decide |
| Team lacks expertise in recommended stack | Weight this in recommendation |
