---
description: Generate cross-project architecture documentation for multi-service systems
---

# System Architecture Agent

Produces centralized documentation mapping service dependencies, cross-service APIs, and unified domain models—enabling `/engineering-agent` to create complete Tech Specs for multi-project features.

## Design Principles

| Principle | Description |
|-----------|-------------|
| **Project Agnostic** | Works for any tech stack (Node, JVM, Python, Go, etc.) |
| **Exhaustive** | All registered projects must be scanned |
| **Cross-Reference** | Types referenced across services are linked |
| **Living Document** | Re-run when projects are added or significantly changed |

## Prerequisites

Each project MUST have been processed by `/map-codebase-agent` first, producing:
- `.ai-instructions/copilot-instructions.md`
- `.ai-instructions/analysis/entity-contracts.json`
- `.ai-instructions/analysis/api-contracts.json`

## Configuration

First, read the configuration file for path variables:
**Read**: `${SYSTEM_ARCH_ROOT}/configuration.md`

Where `SYSTEM_ARCH_ROOT` = `./system-architecture`

---

## Execution

Run phases in order. Each phase outputs to `${OUTPUT_ROOT}`.

### Phase 1: Project Inventory
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-1-project-inventory.md`
**Output**: `analysis/project-inventory.json`
- Scan all registered projects
- Verify `.ai-instructions/` exists for each
- Extract project summaries from `copilot-instructions.md`
- Flag projects missing AI instructions

### Phase 2: Service Topology
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-2-service-topology.md`
**Output**: `analysis/service-topology.json`
- Map service responsibilities
- Identify runtime dependencies (who calls whom)
- Generate dependency graph

### Phase 3: Cross-Service APIs
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-3-cross-service-apis.md`
**Output**: `analysis/cross-service-apis.json`
- Extract API calls FROM client projects TO backend services
- Document DTOs that cross service boundaries
- Map request/response types to source definitions

### Phase 4: Unified Domain Model
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-4-domain-model.md`
**Output**: `analysis/unified-domain-model.json`
- Merge entity definitions across projects
- Identify canonical entities (defined in one place, used in many)
- Flag entity duplicates or conflicts

### Phase 5: Generate System Documentation
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-5-generate-system-doc.md`
**Output**: `system-architecture.md`, `deep-dive/end-to-end-flows.md`, `deep-dive/cross-cutting-concerns.md`
- Generate Mermaid service topology diagram
- Document cross-service API contracts
- Surface unified domain model
- List cross-cutting concerns (auth, error handling, etc.)

### Phase 6: Generate Interactive Diagram Viewer
**Read**: `${SYSTEM_ARCH_ROOT}/_phase-6-interactive-viewer.md`
**Template**: `${SYSTEM_ARCH_ROOT}/WG3-Architecture-template.html`
**Output**: `WG3-Architecture-Interactive.html`
- Update the embedded Mermaid diagrams in the HTML viewer to reflect the latest architecture
- Ensure all projects from `project-inventory.json` are represented as clickable nodes in the System Overview diagram
- Validate that drill-down navigation links correctly to subsystem-specific diagrams
- The viewer provides:
  - Interactive pan/zoom for all diagrams
  - Breadcrumb navigation (System Overview > Subsystem)
  - Real-time sidebar search
  - Collapsible sidebar for full-screen viewing

---

## Success Criteria

- [ ] `project-inventory.json` lists ALL registered projects with status
- [ ] `service-topology.json` has dependency graph with no orphan services
- [ ] `cross-service-apis.json` documents all inter-service calls
- [ ] `unified-domain-model.json` identifies canonical entities
- [ ] `system-architecture.md` contains:
  - Service topology Mermaid diagram
  - Project responsibilities table
  - Cross-service API reference
  - Domain model summary
- [ ] All projects with missing AI instructions are flagged (not silently skipped)
- [ ] `WG3-Architecture-Interactive.html` renders all diagrams with working navigation and zoom
