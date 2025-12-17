# Engineering Agent – DesignReview Mode

## 1. DesignReview Mode – Design Deep-Dive

**Goal**: Extract comprehensive design context from Figma to enable pixel-perfect implementation. This phase produces a Design Review Report with tokens, components, and responsive specifications.

**Trigger**: 
- Run after ProductSpecReview
- Execute if Product Spec contains Figma links

**Skip Condition**: If no Figma links exist in the spec, present option to user:
- **Option A**: Proceed without DesignReview (flag as design debt)
- **Option B**: Request Figma designs from PM/Designer before continuing

---

## 2. Prerequisites

Before starting, ensure:
1. ProductSpecReview is complete (Gap Analysis approved)
2. Figma links are extracted from Product Spec
3. Access to project's design system files:
   - `${COPILOT_INSTRUCTIONS_PATH}` for architecture patterns
   - `${DESIGN_TOKENS_PATH}` for token definitions
   - `${FILE_CATEGORIZATION_PATH}` for component registry

---

## 3. Deep-Dive Extraction Process

> **Reference**: Full extraction steps in [`figma-automation.md`](../design/figma-automation.md)

### Step 1: Node Selection

1. Parse Figma URLs from spec (extract node-id if present)
2. If no node-id, call `mcp_figma-dev-mode-mcp-server_get_metadata` to list frames
3. Match frames to feature areas described in spec

### Step 2: Extract Design Context

For each relevant Figma frame, call `mcp_figma-dev-mode-mcp-server_get_design_context` and extract:

| Category | What to Extract |
|----------|-----------------|
| **Component Tree** | Semantic hierarchy, layer types, nesting structure |
| **Layout Properties** | Direction, gap, padding, alignment, sizing mode |
| **Visual Tokens** | Colors, borders, shadows, typography |
| **Component Instances** | Figma components that map to project components |
| **Interactive States** | Hover, active, disabled, focused variants |

### Step 3: Responsive Design Review

1. Identify viewport variants (Mobile, Tablet, Desktop)
2. Extract each variant's layout and token differences
3. Generate mobile-first responsive token mapping

### Step 4: Extract Variables & Tokens

1. Call `mcp_figma-dev-mode-mcp-server_get_variable_defs` for semantic tokens
2. Map Figma values to project design system:
   - Priority: Figma Variables > Style Names > Algorithmic match
3. Flag any tokens not in project system

### Step 5: Asset Identification

1. Detect image fills, vector graphics, icons
2. Check against existing project assets
3. Generate asset manifest for required exports

### Step 6: Component Matching

1. List all Figma component instances
2. Cross-reference with project `file-categorization.json`
3. Output reuse checklist with import paths

---

## 4. Output: Design Review Report

Generate a structured report with these sections:

```markdown
# Design Review Report: [Feature Name]

**Date**: [Date]
**Figma Source**: [URL(s)]
**Status**: ✅ Complete | ⚠️ Partial | ❌ Issues Found

---

## Token Mapping

| Category | Figma Value | Project Token | Status |
|----------|-------------|---------------|--------|
| Background | #3B82F6 | bg-primary-500 | ✅ |
| Text | #111827 | text-gray-900 | ✅ |
| Spacing | 24px | gap-6 | ✅ |
| Border | #E5E7EB 1px | border-gray-200 | ⚠️ (closest) |

---

## Component Reuse Checklist

- [ ] `Button/Primary` → `<Button variant="primary">` from `@/components/Button`
- [ ] `Icon/Search` → `<Icon name="search">` from `@/components/Icon`
- [ ] `Card/Default` → **[NEW]** - Create following pattern in `@/components/Card`

---

## Responsive Behavior

| Breakpoint | Width | Key Changes |
|------------|-------|-------------|
| Mobile (base) | < 768px | Single column, compact |
| Tablet (md:) | ≥ 768px | Two columns |
| Desktop (lg:) | ≥ 1024px | Three columns, full spacing |

---

## Asset Manifest

| Asset | Type | Format | Target Path | Status |
|-------|------|--------|-------------|--------|
| hero-banner | Raster | PNG @2x | public/images/ | 🆕 Export |
| search-icon | Icon | SVG | - | ✅ Existing |

---

## Design Issues

| Issue | Severity | Impact | Action |
|-------|----------|--------|--------|
| No mobile variant | 🟠 HIGH | Responsive unclear | Request from designer |
| Missing error state | 🟠 HIGH | UX gap | Flag in Epic |
| Font not in system | 🟢 LOW | Use fallback | Document substitution |
```

---

## 5. Failure Handling

### MCP Tool Failure

If `mcp_figma-dev-mode-mcp-server_get_design_context` fails:
1. Request screenshot from user
2. Perform visual analysis instead
3. Mark report as `[MANUAL - from screenshot]`

### Incomplete Figma Structure

| Issue | Handling |
|-------|----------|
| No auto-layout | Estimate spacing, flag for designer confirmation |
| No component instances | Check project library before building custom |
| Missing font | Use project default, document substitution |

---

## 6. Gate Rule

**You CANNOT proceed to FeaturePlanning until**:
- Design Review Report is generated
- User explicitly approves the report
- All 🔴 BLOCKER design issues are resolved OR documented with mitigation plan
