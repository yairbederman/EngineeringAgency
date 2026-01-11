# Engineering Agent – Project Initialization Mode

> **Trigger**: Product Spec approved AND no existing project matches this work.
>
> **Purpose**: Initialize a new project with proper tech stack selection and scaffolding.

---

## Phase 0: New Project Detection

### Step 0.1: Check for Existing Project

1. Read `agent-config.md` → Get list of registered projects
2. Check: Does Product Spec relate to an existing project?
   - Look for project name matches
   - Look for path references
3. **IF match found** → Exit this mode, continue to Feature Planning
4. **IF no match** → Confirm with user:
   ```
   "This appears to be a new project (no existing project found).
   
   Shall I:
   A) Initialize a new project → Continue with Tech Stack Selection
   B) Link to existing project → [List registered projects]"
   ```

---

## Phase 1: Tech Stack Selection

> **Persona**: System Architect (CTO/VP R&D experience)

### Step 1.1: Gather Inputs

**Extract from Approved Product Spec** (token-optimized, don't reload full doc):
- Project type (web app, mobile, API, etc.)
- Key features requiring specific tech (auth, payments, real-time, etc.)
- Scale indicators (MVP, production, enterprise)
- Constraints (budget, timeline, team skills)

**Ask for Future Vision**:
```
"Before selecting a tech stack, I need to understand your future vision:

1. Where do you see this product in 1-2 years?
2. Will you need mobile apps later?
3. Expected scale (users, data volume)?
4. Any team expertise to leverage? (e.g., team knows React)
5. Any tech constraints? (e.g., must use AWS)"
```

### Step 1.2: Analyze and Propose Stack

Based on Product Spec + Future Vision, propose tech stack with full reasoning:

```markdown
## Recommended Tech Stack

**Project**: {project-name}
**Product Spec**: {SPEC-ID}
**Future Vision**: {summary}

---

### Frontend: {framework}

**Why this choice:**
- {reason 1}
- {reason 2}

**Alternatives considered:**
- {alternative}: {why rejected}

---

### Backend: {framework}

**Why this choice:**
- {reason 1}
- {reason 2}

**Alternatives considered:**
- {alternative}: {why rejected}

---

### Database: {database}

**Why this choice:**
- {reason 1}
- {reason 2}

---

### Infrastructure: {platform}

**Why this choice:**
- {reason 1}

---

Approve? [Yes / Modify / Explain More]
```

### Step 1.3: Stack Consistency Validation

**Before presenting to user, validate:**
- [ ] Frontend meta-framework matches framework (Next.js requires React)
- [ ] ORM is compatible with database (Prisma works with PostgreSQL, not MongoDB)
- [ ] Deployment target supports chosen runtime (Vercel supports Node, not Go)

**IF inconsistent** → Fix proposal before presenting.

### Step 1.4: Create Blueprint

On user approval, create blueprint file:

**Path**: `.specs/blueprints/{project-name}-blueprint.md`

Use template: `templates/project-blueprint.md`

---

## Phase 2: Project Scaffold

### Step 2.1: Generate Scaffold Commands

Based on approved tech stack, generate init commands:

```markdown
## Scaffold Commands

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `mkdir {project-name}` | Create project folder |
| 2 | `npx create-next-app@latest ./{project-name} --typescript --tailwind --eslint --app --src-dir` | Initialize Next.js |
| 3 | `cd {project-name} && npm install {dependencies}` | Add required packages |
| 4 | `git init` | Initialize git |

Ready to execute? [Yes, step by step / Yes, all at once / Modify]
```

### Step 2.2: Execute Commands

Execute each command with user approval:
- Show command before running
- Show output after running
- **IF error** → Offer rollback options

### Step 2.3: Rollback Handling

**IF scaffold fails at any step:**

```
❌ Scaffold failed at step {N}: {error}

Options:
A) Retry this step
B) Rollback (delete folder, restart from scratch)
C) Continue manually (I'll provide remaining commands)
D) Abort (keep partial scaffold)
```

---

## Phase 3: Project Registration

### Step 3.1: Register in agent-config

Update `agent-config.md` Registered Projects:

```markdown
| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `PROJECT_{KEY}` | {project-name} | {type} | {role} | `./{project-name}` |
```

### Step 3.2: Git Setup (Optional)

If user wants to set up git remote:

```
"Would you like to set up a git remote? [Yes / Later]"
```

**IF Yes:**
```bash
cd {project-name}
git remote add origin {repo-url}
git push -u origin main
```

**IF Later:** Continue without remote.

---

## Phase 4: Continue to Normal Flow

```
✅ **Ready for Development**

Project "{project-name}" is now set up and registered.

> **Next Steps**:
> 1. Continue to Tech Spec → Define architecture
> 2. Continue to Task Planning → Break into tasks
> 3. Start Execution → Implement first feature

What would you like to do?
```

---

## After First Task Merged

When first implementation task is completed, recommend:

```
✅ **First Task Merged**

Your project now has real code patterns.

> **Recommend**: Run `/map-codebase-agent` to generate `.ai-instructions`
> This will help the agent understand your codebase architecture.

Would you like to run it now? [Yes / Later]
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Product Spec not approved | `❌ Cannot initialize project. Product Spec must be approved first.` |
| Scaffold command failed | Offer rollback, retry, or manual continuation |
| Git remote not accessible | `❌ Cannot connect to {repo-url}. Check URL and permissions.` |
| agent-config not found | `❌ agent-config.md not found. Run setup first.` |
