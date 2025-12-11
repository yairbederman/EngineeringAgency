# Phase 5: Generate System Documentation

> **Goal**: Generate the final `system-architecture.md` and deep-dive documents.

---

## Input

- `${ANALYSIS_DIR}/project-inventory.json` (Phase 1)
- `${ANALYSIS_DIR}/service-topology.json` (Phase 2)
- `${ANALYSIS_DIR}/cross-service-apis.json` (Phase 3)
- `${ANALYSIS_DIR}/unified-domain-model.json` (Phase 4)

## Output

- `${OUTPUT_ROOT}/system-architecture.md`
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

Generate `end-to-end-flows.md` with:

### Flow Template

```markdown
## Flow: <Flow Name>

### Overview
<Brief description of what this flow accomplishes>

### Service Chain
<service-a> → <service-b> → <service-c> → <service-d>

### Sequence
1. **<service-a>**: <user action or trigger>
2. **<service-b>**: `<METHOD> <endpoint>` <what it does>
3. **<service-c>**: `<METHOD> <endpoint>` <what it does>
4. **<service-d>**: `<METHOD> <endpoint>` <what it does>

### Data Flow
{Mermaid sequence diagram}
```

---

## Deep-Dive: Cross-Cutting Concerns

Generate `cross-cutting-concerns.md` documenting shared patterns.

---

## Validation

| Check | Requirement |
|-------|-------------|
| Mermaid valid | Diagram renders without errors |
| All services included | Count matches project inventory |
| Links valid | All entity/API references resolvable |
| Sections complete | No empty tables or placeholders |
