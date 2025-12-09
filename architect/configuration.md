# Architect Agent Configuration

> **Purpose**: Single source of truth for architect agent paths.
> When migrating, update `ARCHITECT_ROOT` only.

---

## Agent Paths

| Variable | Value | Description |
|----------|-------|-------------|
| `ARCHITECT_ROOT` | `c:\Users\YairBederman\.gemini\antigravity\global_workflows\architect\` | Base path for architect files |

### Phase Files (relative to ARCHITECT_ROOT)

| Phase | File | Output |
|-------|------|--------|
| Phase 1: Detect Stack | `_phase-1-detect-stack.md` | `analysis/techstack.md`, `analysis/source-structure.json` |
| Phase 2: Extract Entities | `_phase-2-extract-entities.md` | `analysis/entity-contracts.json` |
| Phase 3: Extract APIs | `_phase-3-extract-apis.md` | `analysis/api-contracts.json` |
| Phase 4: Map Dependencies | `_phase-4-map-dependencies.md` | `analysis/function-registry.json`, `deep-dive/dependency-chains.md` |
| Phase 5: Generate Master | `_phase-5-generate-master.md` | `copilot-instructions.md`, `analysis/file-categorization.json` |
| Validation | `_validation-rules.md` | Completeness checks |

---

## Output Paths (relative to target project root)

| Variable | Path | Description |
|----------|------|-------------|
| `${AI_INSTRUCTIONS_ROOT}` | `.ai-instructions/` | Root for all generated files |
| `${ANALYSIS_DIR}` | `.ai-instructions/analysis/` | JSON analysis files |
| `${DEEP_DIVE_DIR}` | `.ai-instructions/deep-dive/` | Detailed documentation |

---

## Installation

After cloning, update `ARCHITECT_ROOT` to match your local path:
- **Windows**: `c:\Users\{username}\.gemini\antigravity\global_workflows\architect\`
- **Mac/Linux**: `~/.gemini/antigravity/global_workflows/architect/`
