# Phase 5: Generate System Documentation

> **Goal**: Generate the final `system-architecture.md` and deep-dive documents.

---

## Input

- `${ANALYSIS_DIR}/project-inventory.json` (Phase 1)
- `${ANALYSIS_DIR}/service-topology.json` (Phase 2)
- `${ANALYSIS_DIR}/cross-service-apis.json` (Phase 3)
- `${ANALYSIS_DIR}/unified-domain-model.json` (Phase 4)

## Output

- `${OUTPUT_ROOT}/${SYSTEM_NAME}-system-architecture.md`
- `${DEEP_DIVE_DIR}/end-to-end-flows.md`
- `${DEEP_DIVE_DIR}/cross-cutting-concerns.md`

---

## Instructions

### Step 1: Generate Service Topology Diagram

Create a Mermaid diagram showing all services and dependencies:

```mermaid
graph TB
    subgraph Presentation
        CLIENT["<frontend-project><br/>Frontend App"]
    end
    
    subgraph API Layer
        SVC1["<backend-service-1><br/>Description"]
        SVC2["<backend-service-2><br/>Description"]
        SVC3["<backend-service-3><br/>Description"]
    end
    
    subgraph Integration
        EXT["<integration-service><br/>External/CMS"]
    end
    
    CLIENT --> SVC1
    CLIENT --> SVC2
    CLIENT --> SVC3
    SVC1 --> EXT
```

> [!CAUTION]
> **Diagram Edges MUST Match service-topology.json**
> - **ONLY** draw edges that exist in the `dependencies[]` array from Phase 2
> - Do **NOT** infer additional connections based on assumptions or documentation
> - Each edge in the diagram **MUST** have a corresponding entry in `dependencies[]`
> - If a connection is missing from `dependencies[]`, it means it was NOT verified against code and should NOT appear in the diagram

### Step 1.5: Validate Mermaid Diagram Against Source Data

Before finalizing the diagram, perform these checks:

| Check | Source | Diagram Requirement |
|-------|--------|---------------------|
| **Node Count** | `services.length` from `service-topology.json` | Must match node count (excluding subgraph labels) |
| **Edge Count** | `dependencies.length` from `service-topology.json` | Must match edge count exactly |
| **All Projects** | `project-inventory.json` (status=ready) | Every ready project must appear as a node |
| **No Phantom Edges** | `dependencies[]` array | Every `A --> B` must have `{from: A, to: B}` entry |

**Validation Procedure:**
1. For each edge `A --> B` you plan to draw:
   - Search `dependencies[]` for `{from: "A", to: "B"}`
   - If NOT found → **DO NOT draw this edge**
2. For each entry in `dependencies[]`:
   - Verify it appears as an edge in your diagram
   - If missing → **Add the edge**
3. Count nodes vs `services.length` - must match
4. Count edges vs `dependencies.length` - must match

**If validation fails**: Do NOT generate the diagram. Report the discrepancy.

### Step 1.6: Generate Per-Project Submodule Diagrams (MANDATORY for Libraries)

> [!IMPORTANT]
> **For projects with `internalModules[]` in `service-topology.json`, you MUST generate a detailed submodule diagram.**

For each project where `internalModules.length > 0`:

1. **Read `internalModules[]` from `service-topology.json`**
2. **Group modules by type** (Core, Client, Integration, etc.)
3. **Generate Mermaid subgraph**:

```mermaid
graph TB
    subgraph Modules["Submodules"]
        MOD1["<module-name-1>"]
        MOD2["<module-name-2>"]
        %% Include ALL modules from internalModules[]
    end
    
    subgraph Integrations["External Integrations"]
        EXT1["<external-module-1>"]
        %% Include ALL external/integration modules
    end
```

**Validation**: 
- Count nodes in diagram MUST equal `internalModules.length`
- If count doesn't match → diagram is INCOMPLETE

### Step 1.7: Generate ASCII Architecture Diagrams (MANDATORY)

> [!IMPORTANT]
> **ASCII diagrams provide text-based visualization that works everywhere—terminals, emails, plain text docs.**
> Generate BOTH system architecture AND software architecture ASCII diagrams.

#### 1.7.1: ASCII System Architecture Diagram

Generate a high-level system topology using box-drawing characters:

```
                            ┌─────────────────────────────────────────────────────────┐
                            │                    SYSTEM OVERVIEW                       │
                            └─────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────────────────────────┐
    │                              PRESENTATION LAYER                                      │
    │  ┌───────────────────────────────────────────────────────────────────────────────┐  │
    │  │  <frontend-project>                                                            │  │
    │  │  └── <description>                                                             │  │
    │  └───────────────────────────────────────────────────────────────────────────────┘  │
    └─────────────────────────────────────────────────────────────────────────────────────┘
                                              │
                                              ▼
    ┌─────────────────────────────────────────────────────────────────────────────────────┐
    │                                   API LAYER                                          │
    │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐                   │
    │  │ <api-service-1>  │  │ <api-service-2>  │  │ <api-service-3>  │                   │
    │  │ └── <desc>       │  │ └── <desc>       │  │ └── <desc>       │                   │
    │  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘                   │
    └───────────┼─────────────────────┼─────────────────────┼─────────────────────────────┘
                │                     │                     │
                ▼                     ▼                     ▼
    ┌─────────────────────────────────────────────────────────────────────────────────────┐
    │                              SHARED/INTEGRATION                                      │
    │  ┌──────────────────────────────────────────────────────────────────────────────┐   │
    │  │  <shared-lib>  ──────►  <integration-service>  ──────►  [External Systems]   │   │
    │  └──────────────────────────────────────────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────────────────────────┐
    │                                    LEGEND                                            │
    │  ───────►  Service call / dependency      [ ]  External system                       │
    │  └──       Description/role               │    Data flow direction                   │
    └─────────────────────────────────────────────────────────────────────────────────────┘
```

**Rules for ASCII System Diagram:**
1. Group services by layer (Presentation, API, Shared, Integration)
2. Use box-drawing characters: `┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼ ▼ ► ◄ ▲`
3. Show ALL services from `service-topology.json`
4. Indicate data flow direction with arrows
5. Include a legend

#### 1.7.2: ASCII Software Architecture Diagram

Generate a detailed component/module breakdown:

```
╔══════════════════════════════════════════════════════════════════════════════════════════╗
║                              SOFTWARE ARCHITECTURE                                        ║
╠══════════════════════════════════════════════════════════════════════════════════════════╣
║                                                                                           ║
║  CLIENT LAYER                                                                             ║
║  ════════════                                                                             ║
║  ┌────────────────────────────────────────────────────────────────────────────────────┐  ║
║  │ <FRONTEND-PROJECT>                                                                  │  ║
║  │                                                                                     │  ║
║  │   Components      Services        State           API Clients                       │  ║
║  │   ┌─────────┐    ┌───────────┐   ┌──────────┐    ┌──────────────────┐              │  ║
║  │   │ Pages   │    │ Auth      │   │ Redux/   │    │ <api-1>-client   │              │  ║
║  │   │ UI Comps│    │ Data      │   │ Context  │    │ <api-2>-client   │              │  ║
║  │   │ Layouts │    │ Util      │   │ Hooks    │    │ <api-3>-client   │              │  ║
║  │   └─────────┘    └───────────┘   └──────────┘    └──────────────────┘              │  ║
║  └────────────────────────────────────────────────────────────────────────────────────┘  ║
║                                         │                                                 ║
║                                         │ HTTP/REST                                       ║
║                                         ▼                                                 ║
║  API LAYER                                                                                ║
║  ═════════                                                                                ║
║  ┌───────────────────────┐  ┌───────────────────────┐  ┌───────────────────────┐        ║
║  │ <API-SERVICE-1>       │  │ <API-SERVICE-2>       │  │ <API-SERVICE-3>       │        ║
║  │                       │  │                       │  │                       │        ║
║  │ ┌───────────────────┐ │  │ ┌───────────────────┐ │  │ ┌───────────────────┐ │        ║
║  │ │ Controllers       │ │  │ │ Controllers       │ │  │ │ Controllers       │ │        ║
║  │ ├───────────────────┤ │  │ ├───────────────────┤ │  │ ├───────────────────┤ │        ║
║  │ │ Services          │ │  │ │ Services          │ │  │ │ Services          │ │        ║
║  │ ├───────────────────┤ │  │ ├───────────────────┤ │  │ ├───────────────────┤ │        ║
║  │ │ Repositories      │ │  │ │ Repositories      │ │  │ │ Repositories      │ │        ║
║  │ └───────────────────┘ │  │ └───────────────────┘ │  │ └───────────────────┘ │        ║
║  └───────────┬───────────┘  └───────────┬───────────┘  └───────────┬───────────┘        ║
║              │                          │                          │                     ║
║              └──────────────────────────┼──────────────────────────┘                     ║
║                                         ▼                                                 ║
║  SHARED LIBRARY                                                                           ║
║  ══════════════                                                                           ║
║  ┌────────────────────────────────────────────────────────────────────────────────────┐  ║
║  │ <SHARED-LIB>                                                                        │  ║
║  │                                                                                     │  ║
║  │   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌─────────────────────────┐   │  ║
║  │   │ Core DTOs  │   │ Utilities  │   │ Clients    │   │ Error Handling          │   │  ║
║  │   └────────────┘   └────────────┘   └────────────┘   └─────────────────────────┘   │  ║
║  └────────────────────────────────────────────────────────────────────────────────────┘  ║
║                                         │                                                 ║
║                                         ▼                                                 ║
║  EXTERNAL INTEGRATIONS                                                                    ║
║  ═════════════════════                                                                    ║
║  ┌────────────────────────────────────────────────────────────────────────────────────┐  ║
║  │ [CMS]  [Payment Gateway]  [Email Service]  [Search Engine]  [Database]             │  ║
║  └────────────────────────────────────────────────────────────────────────────────────┘  ║
║                                                                                           ║
╚══════════════════════════════════════════════════════════════════════════════════════════╝
```

**Rules for ASCII Software Diagram:**
1. Use double-line box for outer frame: `╔ ╗ ╚ ╝ ║ ═ ╠ ╣ ╬`
2. Use single-line box for components: `┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼`
3. Show internal structure of each project type
4. Include layer labels with `════════` underlines
5. Show data flow between layers

#### 1.7.3: Generate `deep-dive/ascii-architecture.md`

Create a dedicated file containing:

```markdown
# ASCII Architecture Diagrams

> Generated: {timestamp}
> Render correctly in: Terminal, Plain text editors, Markdown viewers

## System Architecture

{ASCII System Diagram from 1.7.1}

## Software Architecture  

{ASCII Software Diagram from 1.7.2}

## Dependency Matrix (ASCII Table)

| From ↓ / To → | <svc-1> | <svc-2> | <svc-3> | <lib> |
|---------------|---------|---------|---------|-------|
| <frontend>    |    ✓    |    ✓    |    ✓    |       |
| <svc-1>       |         |         |         |   ✓   |
| <svc-2>       |         |         |         |   ✓   |
| <svc-3>       |    ✓    |         |         |   ✓   |

## Legend

| Symbol | Meaning |
|--------|---------|
| ──────►| Data flow / API call |
| ┌─────┐| Service or component boundary |
| [ ]   | External system |
| ════  | Layer separator |
```

**Validation for Step 1.7:**
- [ ] ASCII System Diagram includes ALL services from `service-topology.json`
- [ ] ASCII Software Diagram shows internal layers for each project type
- [ ] All box-drawing characters render correctly (UTF-8)
- [ ] `deep-dive/ascii-architecture.md` generated
- [ ] Dependency matrix matches `service-topology.json` edges

### Step 2: Generate Project Responsibilities Table

| Project | Type | Role | Endpoints | Key Entities |
|---------|------|------|-----------|--------------|
| <project-name> | <Frontend/Backend> | <role from inventory> | <count> | <key entities> |
| ... | ... | ... | ... | ... |

### Step 3: Generate Cross-Service API Reference

For each service pair:

#### <caller-service> → <callee-service>

| Endpoint | Method | Request | Response | Used By |
|----------|--------|---------|----------|---------|
| `<path>` | <METHOD> | <RequestDTO> | <ResponseDTO> | <components> |
| ... | ... | ... | ... | ... |

### Step 4: Generate Domain Model Summary

#### Domain Areas

| Area | Owner | Key Entities |
|------|-------|--------------|
| <DomainArea> | <owning-project> | <EntityA>, <EntityB> |
| ... | ... | ... |

#### Canonical Entities

List entities that are defined once but used across services.

### Step 5: Document Cross-Cutting Concerns

Analyze projects for:

| Concern | Pattern | Projects |
|---------|---------|----------|
| Authentication | <pattern> | <projects> |
| Error Handling | <pattern> | <projects> |
| Logging | <pattern> | <projects> |
| Caching | <pattern> | <projects> |

---

## system-architecture.md Template

```markdown
# System Architecture

> Generated: {timestamp}
> Projects: {count}

## Service Topology

{Mermaid diagram}

## Project Responsibilities

{Table from Step 2}

## Cross-Service API Reference

{Sections from Step 3}

## Domain Model

### Domain Areas
{Table from Step 4}

### Canonical Entities
{List from Step 4}

### Entity Conflicts
{Warnings from unified-domain-model.json}

## Cross-Cutting Concerns

{Table from Step 5}

## How to Use This Document

### Integration with /engineering-agent

| Engineering Mode | System Architecture File | When to Read | Purpose |
|-----------------|-------------------------|--------------|---------|
| **TechSpec Step 2** | `service-topology.json` | BEFORE identifying projects | Discover all upstream/downstream dependencies |
| **TechSpec Step 2** | `service-topology.json` → `calledBy` | After identifying scope | Find consumers affected by changes |
| **TechSpec § 5.2** | `unified-domain-model.json` | When defining entities | Use canonical entity definitions |
| **TechSpec § 5.3** | `cross-service-apis.json` | When defining API contracts | Get existing cross-service signatures |
| **TaskPlanning** | `cross-service-apis.json` | Context injection step | Inject cross-service API context into tasks |
| **BugReport Step 6** | `service-topology.json` | Cross-repo analysis | Trace service chain for bug isolation |
| **BugFix Step 2** | `cross-service-apis.json` | Impact assessment | Verify fix doesn't break callers |

### Quick Reference for Multi-Project Features

1. **Before TechSpec**: Read `service-topology.json`
   - Check both `callsServices` AND `calledBy` for each service in scope
   - Add transitively affected services to Epic scope
2. **During TechSpec**: Read `unified-domain-model.json`
   - Use `canonicalSource` for entity definitions
   - Flag any `conflicting` fields
3. **During TaskPlanning**: Read `cross-service-apis.json`
   - Copy endpoint contracts into cross-service tasks

### Prerequisites (Data Source)
This document aggregates data from projects processed by `/map-codebase-agent`.

If project AI instructions are stale, re-run:
1. `/map-codebase-agent` on affected project(s)
2. `/system-architecture-agent` to refresh this document

### Updating This Document
Run `/system-architecture-agent` after:
- Adding a new project
- Significant API changes
- New cross-service integrations
```

---

## Deep-Dive: End-to-End Flows

> [!CAUTION]
> **Every flow step MUST be verified against actual code.**
> - Each "Service X calls Service Y" MUST have a matching entry in `cross-service-apis.json`
> - If no matching entry exists, the flow step is UNVERIFIED and must be flagged
> - Do NOT infer intermediate services from documentation - verify them in code
> - Do NOT assume direct calls exist - verify caller has API client for callee

Generate `end-to-end-flows.md` with:

### Flow Template

```markdown
## Flow: <Flow Name>

### Overview
<Brief description of what this flow accomplishes>

### Service Chain
<service-a> → <service-b> → <service-c> → <service-d>

### Verification (MANDATORY)
| Step | From | To | Verified In `cross-service-apis.json` | Code Evidence |
|------|------|----|---------------------------------------|---------------|
| 1 | <service-a> | <service-b> | ✅ Call ID: <call-id> | `<ApiClient>.ts:L<line>` |
| 2 | <service-b> | <service-c> | ✅ Call ID: <call-id> | `<Service>.java:L<line>` |

> **Validation Rule**: If ANY step cannot be verified against `cross-service-apis.json`, 
> mark it with ⚠️ UNVERIFIED and add to `_warnings` in JSON outputs.

### Sequence
1. **<service-a>**: <user action or trigger>
2. **<service-b>**: `<METHOD> <endpoint>` <what it does>
   - **Code Evidence**: `<file>:<line>` - `<method signature or call snippet>`
3. **<service-c>**: `<METHOD> <endpoint>` <what it does>
   - **Code Evidence**: `<file>:<line>` - `<method signature or call snippet>`
4. **<service-d>**: `<METHOD> <endpoint>` <what it does>
   - **Code Evidence**: `<file>:<line>` - `<method signature or call snippet>`

### Data Flow
{Mermaid sequence diagram - edges MUST match Verification table above}
```

### Flow Verification Gate (BLOCKING)

Before generating any flow in `end-to-end-flows.md`:

1. **For each step `A → B`**:
   - Search `cross-service-apis.json` for entry with `caller: A` and `callee: B`
   - If NOT found → **DO NOT document as direct call**
   - Check if `A → X → B` exists (intermediate service)

2. **Evidence Requirements**:
   - Every step MUST cite specific file and line number
   - Evidence must come from actual codebase grep/view, not from AI instructions

3. **Unverified Steps**:
   - If code evidence cannot be found, mark step as `⚠️ UNVERIFIED`
   - Add to `_warnings[]` in system documentation outputs

---

## Deep-Dive: Cross-Cutting Concerns

Generate `cross-cutting-concerns.md` documenting shared patterns.

## Step 6: Update ALL Existing Output Files (MANDATORY)

> [!IMPORTANT]
> **Before completing Phase 5, you MUST enumerate and update ALL existing files in `${OUTPUT_ROOT}`.**
> This includes the root directory AND all subdirectories (analysis/, deep-dive/, etc.).
> Do NOT assume you know what files exist - always enumerate dynamically.

### 6.1 Enumerate Entire Output Directory Tree

```
list_dir("${OUTPUT_ROOT}")
```

For each subdirectory found (e.g., `analysis/`, `deep-dive/`):
```
list_dir("${OUTPUT_ROOT}/<subdirectory>")
```

### 6.2 Process Each File Found

**For EVERY `.md`, `.json`, and `.html` file found:**

1. **Read the file** to understand its structure
2. **Identify project-related content**:
   - Project lists or counts
   - Service names in diagrams, tables, or code
   - Dependency lists
   - Coverage statistics
3. **Update with new project information** where applicable
4. **Update timestamps** if the file has a `generatedAt` field

### 6.3 Common Patterns to Check

| File Type | What to Update |
|-----------|----------------|
| `*.json` (analysis/) | Project counts, service arrays, coverage stats |
| `*.md` (root, deep-dive/) | Mermaid diagrams, project tables, service lists |
| `*.html` (viewers) | Embedded diagrams, navigation items |

### 6.4 Validation Checklist

Before marking Phase 5 complete:

- [ ] `list_dir("${OUTPUT_ROOT}")` executed
- [ ] `list_dir()` executed on EACH subdirectory
- [ ] **Every file** in the directory tree was read
- [ ] New project added to all relevant sections
- [ ] No file or directory was skipped

---

## Validation

| Check | Requirement |
|-------|-------------|
| Mermaid valid | Diagram renders without errors |
| All services included | Count matches project inventory |
| Links valid | All entity/API references resolvable |
| Sections complete | No empty tables or placeholders |
| **Full tree enumerated** | `list_dir()` on root AND all subdirectories |
| **All files processed** | Every file in tree was read and updated if needed |
| **Counts consistent** | Project counts match across all JSON/MD files |
