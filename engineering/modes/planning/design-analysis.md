# Engineering Agent – DesignAnalysis Mode

> **Persona**: Load `${AGENT_ROOT}/personas/designer.md`

## 1. DesignAnalysis Mode – Design Deep-Dive

**Goal**: Extract comprehensive design context from Figma to enable pixel-perfect implementation. This phase produces a Design Review Report with tokens, components, and responsive specifications.

**Trigger**: 
- Run after ProductSpecReview
- Execute if Product Spec contains Figma links

**Skip Condition**: If no Figma links exist in the spec, present option to user:
- **Option A**: Proceed without DesignAnalysis (flag as design debt)
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

> **Reference**: Full extraction steps in [`figma-extraction-protocol.md`](../design/figma-extraction-protocol.md)

### Step 1: Node Selection

1. Parse Figma URLs from spec (extract node-id if present)
2. If no node-id, call `${MCP_FIGMA_GET_METADATA}` to list frames
3. Match frames to feature areas described in spec

### Step 2: Structural Discovery (CRITICAL)

> **Protocol**: The "2-Step Extraction Rule" applies here.

1. **Call `get_metadata(nodeId)`**:
   - Analyze the returned structure.
   - **Check**: Does it contain `<symbol>` (variants) or `<instance>` children?
   - **Decision**:
     - **YES** (Complex): You MUST extract *each* child node individually in Next Step.
     - **NO** (Simple): You may extract the parent node directly.

### Step 3: Deep-Dive Extraction

Execute based on Step 2 decision:

- **For Simple Frames**: Call `get_design_context(parentNodeId)`
- **For Component Sets**: Loop through child IDs and call `get_design_context(childNodeId)`

**Extract the following for the complete set**:

| Category | What to Extract |
|----------|-----------------|
| **Component Tree** | Semantic hierarchy, layer types, nesting structure |
| **Layout Properties** | Direction, gap, padding, alignment, sizing mode |
| **Visual Tokens** | Colors, borders, shadows, typography |
| **Component Instances** | Figma components that map to project components |
| **Interactive States** | Hover, active, disabled, focused variants |
| **Annotations** | `data-development-annotations` from *every* child variant |

### Step 4: Responsive Design Review

1. Identify viewport variants (Mobile, Tablet, Desktop)
2. Extract each variant's layout and token differences
3. Generate mobile-first responsive token mapping

### Step 5: Extract Variables & Tokens

1. Call `${MCP_FIGMA_GET_VARS}` for semantic tokens
2. Map Figma values to project design system:
   - Priority: Figma Variables > Style Names > Algorithmic match
3. Flag any tokens not in project system

### Step 6: Spatial Constraints Analysis

> **Purpose**: LLMs understand tokens but struggle with spatial reasoning. This step captures scroll behavior, pinned elements, z-index relationships, and overlay patterns that cannot be inferred from token tables.

1. **Identify Scroll Containers**:
   - Which frames have `overflow: scroll/auto`?
   - What content lives inside scroll areas vs. outside (pinned)?

2. **Detect Pinned/Sticky Elements**:
   - Headers that stick during scroll
   - Footers/action bars pinned to bottom
   - Floating action buttons (FABs)

3. **Map z-index Relationships**:
   - Overlays, modals, dropdowns (high z-index)
   - Main content (base z-index)
   - Background elements (low z-index)

4. **Document Viewport Constraints**:
   - Fixed position elements (relative to viewport)
   - Absolute position elements (relative to parent)
   - Sticky position elements (hybrid behavior)

**Detection Heuristics** (from Figma structure):
| Figma Pattern | Likely Constraint | CSS Implementation |
|---------------|-------------------|-------------------|
| Frame with `constraints.vertical = MAX` at bottom | Pinned footer | `mt-auto` or `sticky bottom-0` |
| Frame outside auto-layout with high layer order | Overlay/modal | `fixed inset-0 z-50` |
| Frame with `FILL` height containing scrollable content | Scroll container | `flex-1 overflow-auto` |
| Small frame at corner with high layer order | FAB | `fixed bottom-4 right-4 z-40` |

### Step 7: Content Constraints Analysis

> **Purpose**: Figma shows ideal content ("John Doe", 3 items). LLMs need to know how components behave with real-world variable content.

1. **Identify Text Elements**:
   - For each text layer, determine: What happens if text is 2x longer? 5x longer?
   - Define: truncate, wrap, clamp lines, or expand container?

2. **Identify List/Array Elements**:
   - For each repeating pattern, determine: What if 0 items? 100+ items?
   - Define: empty state, pagination, virtualization threshold

3. **Identify Dynamic Content**:
   - Images: What's the fallback if image fails to load?
   - Numbers: What's the max value? Locale formatting?
   - Dates: What format? Relative vs absolute?

4. **Document Required States**:
   - Empty states (0 items)
   - Loading states (skeleton/spinner)
   - Error states (failed to load)

**Common Overflow Patterns**:
| Content Type | Typical Constraint | CSS Implementation |
|--------------|-------------------|-------------------|
| User name | Max 30 chars, truncate | `truncate max-w-[200px]` |
| Description | Max 2 lines, clamp | `line-clamp-2` |
| Tag list | Max 5 visible, +N more | Custom logic |
| Long list | Virtualize after 50 | `react-virtual` or similar |

### Step 8: Semantic Intent Analysis

> **Purpose**: Figma shows *appearance*; Product Specs define *behavior*. LLMs need both. This step cross-references visual elements with their behavioral intent.

1. **Identify Interactive Elements**:
   - Scan extracted frame for: Buttons, Links, Inputs, Clickable cards, Toggles, etc.
   - For each, ask: "What happens when user interacts?"

2. **Cross-Reference with Product Spec**:
   - Match Figma element names to spec user stories/flows
   - Extract: Trigger action, Target state, Side effects

3. **Classify Behavioral Intent**:

| Intent Type | Examples | Implementation Pattern |
|-------------|----------|------------------------|
| **Navigation** | "View Details", breadcrumbs, tabs | `router.push()`, `<Link>` |
| **State Change** | Toggle, accordion, expand/collapse | `useState`, component state |
| **Data Mutation** | Save, Delete, Submit | API call + optimistic/confirmed update |
| **Data Fetch** | "Load More", search, filter | `useQuery`, infinite scroll, debounce |
| **Modal/Overlay** | "Add Item", confirmation dialogs | Modal open state, portal |
| **External** | Share, download, print | Browser APIs, external links |

4. **Document in Semantic Intent Map**:

```markdown
| Element (from Figma) | Visual Role | Behavioral Intent | Spec Reference | Implementation Pattern |
|----------------------|-------------|-------------------|----------------|------------------------|
| "View Details" button | CTA on card | Navigate to product detail page | § 3.2 Product Card | `router.push('/products/{id}')` |
| Heart icon | Top-right of card | Toggle wishlist (optimistic) | § 4.1 Wishlist | `useMutation` + optimistic update |
| "Load More" button | Bottom of grid | Fetch next page, append | § 3.1 Product Listing | `useInfiniteQuery` or intersection observer |
| Card container | Full card area | Navigate to product (same as View Details) | § 3.2 | `onClick` on card, `<Link>` wrapper |
```

5. **Handle Missing Behavioral Context**:

| Scenario | Action |
|----------|--------|
| Element in Figma but not in spec | Flag: `[TBD - behavior not specified in spec]` |
| Spec describes behavior, Figma doesn't show element | Flag in Gap Analysis, not here |
| Ambiguous intent (could be modal OR navigation) | List both options, ask for clarification |

6. **Detect Implicit Behaviors**:
   - Card click = same as "View Details" button? (common pattern)
   - Enter key in search input = click Search button?
   - Escape key in modal = click Close button?
   - Document these keyboard/accessibility equivalents

---

### Step 9: Asset Identification

1. Detect image fills, vector graphics, icons
2. Check against existing project assets
3. Generate asset manifest for required exports

### Step 10: Component Matching

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

## Visual Reference (Screenshots)

> **Purpose**: Provide visual grounding for LLM implementation. Token tables alone cannot convey visual hierarchy.

### Primary View (Desktop)
![Feature Name - Desktop](file:///path/to/screenshot-desktop.png)

### Responsive Variants (if available)
| Viewport | Screenshot |
|----------|------------|
| Mobile | ![Mobile](file:///path/to/screenshot-mobile.png) |
| Tablet | ![Tablet](file:///path/to/screenshot-tablet.png) |

### Critical Visual Notes
> - Button row is pinned to bottom (space-between, not gap-stacking)
> - Content area scrolls if overflow
> - Title has larger visual weight (24px margin-bottom vs 16px elsewhere)

---

## Token Mapping

| Category | Figma Value | Variable/Style | Project Token | Status |
|----------|-------------|----------------|---------------|--------|
| Background | #3B82F6 | color/primary/500 | bg-primary-500 | ✅ |
| Text | #111827 | color/neutral/900 | text-gray-900 | ✅ |
| Spacing | 24px | spacing/lg | gap-6 | ✅ |
| Border | #E5E7EB 1px | - | border-gray-200 | ⚠️ (closest) |

---

## Spatial Constraints

> **Purpose**: Capture scroll behavior, pinned elements, z-index, and positioning that LLMs cannot infer from tokens alone.

| Element | Constraint Type | Behavior | CSS Implementation |
|---------|-----------------|----------|-------------------|
| HeaderBar | Sticky | Sticks to top during content scroll | `sticky top-0 z-10` |
| ContentArea | Scroll Container | Scrolls when content overflows | `flex-1 overflow-auto` |
| ActionBar | Pinned Footer | Always visible at bottom of card | `mt-auto` or `sticky bottom-0` |
| ModalOverlay | Fixed Overlay | Covers viewport, blocks interaction | `fixed inset-0 z-50 bg-black/50` |
| FAB | Fixed Position | Always bottom-right of viewport | `fixed bottom-4 right-4 z-40` |

### Scroll & Overflow Behavior
```
┌─────────────────────────────┐
│ HeaderBar (sticky top)      │ ← Does NOT scroll
├─────────────────────────────┤
│                             │
│ ContentArea (overflow-auto) │ ← Scrolls independently
│                             │
├─────────────────────────────┤
│ ActionBar (mt-auto/sticky)  │ ← Does NOT scroll
└─────────────────────────────┘
```

### z-index Stack
| Layer | z-index | Elements |
|-------|---------|----------|
| Modal/Overlay | 50 | ModalOverlay, DialogBackdrop |
| Dropdown/Popover | 40 | Dropdown menus, tooltips |
| Floating | 30 | FAB, floating action buttons |
| Sticky | 10 | HeaderBar, sticky navigation |
| Base | 0 | Main content |

---

## Content Constraints

> **Purpose**: Figma shows ideal content. This section documents how components handle real-world variable content.

### Text Overflow Behavior
| Element | Content Type | Max Length | Overflow Behavior | CSS Implementation |
|---------|--------------|------------|-------------------|-------------------|
| User Name | Text | 30 chars | Truncate with ellipsis | `truncate max-w-[200px]` |
| Card Title | Text | 50 chars | Truncate | `truncate` |
| Description | Text | 150 chars | Clamp 2 lines | `line-clamp-2` |
| Button Label | Text | 20 chars | No wrap, min-width | `whitespace-nowrap min-w-[80px]` |

### List/Array Behavior
| Element | Min Items | Max Items | Empty State | Overflow Behavior |
|---------|-----------|-----------|-------------|-------------------|
| Product Grid | 0 | ∞ | "No products found" | Paginate (12/page) |
| Tag List | 0 | 10 | Hide section | Show +N after 5 |
| Notifications | 0 | ∞ | "All caught up!" | Virtualize after 50 |
| Search Results | 0 | ∞ | "No matches" | Infinite scroll |

### Required States Checklist
- [ ] **Empty State**: [Component] shows [message/illustration] when 0 items
- [ ] **Loading State**: [Component] shows [skeleton/spinner] while loading
- [ ] **Error State**: [Component] shows [error message] on failure
- [ ] **Partial State**: [Component] shows [fallback] for missing data (e.g., initials if no avatar)

### Dynamic Content Formatting
| Content | Format | Example |
|---------|--------|---------|
| Price | Locale currency | `$1,234.56` |
| Date | Relative < 7 days, absolute otherwise | "2 hours ago" / "Dec 15, 2024" |
| Numbers | Compact notation for large | `12.5K` |
| Phone | E.164 display | `+1 (555) 123-4567` |

---

## Interaction States

> **Purpose**: Document component states and transitions for micro-interactions.

### Button States
| State | Background | Border | Shadow | Transform | Transition |
|-------|------------|--------|--------|-----------|------------|
| Default | bg-primary-500 | - | shadow-sm | - | - |
| Hover | bg-primary-600 | - | shadow-md | scale(1.02) | 150ms ease-out |
| Pressed | bg-primary-700 | - | shadow-sm | scale(0.98) | 50ms ease-in |
| Focused | bg-primary-500 | ring-2 ring-primary-300 | shadow-sm | - | 100ms |
| Disabled | bg-gray-300 | - | - | - | - |

### Transition CSS
```css
.button {
  @apply transition-all duration-150 ease-out;
}
.button:hover {
  @apply bg-primary-600 shadow-md scale-102;
}
.button:active {
  @apply bg-primary-700 shadow-sm scale-98;
}
.button:focus-visible {
  @apply ring-2 ring-primary-300 ring-offset-2;
}
```

---

## Semantic Intent Map

> **Purpose**: Bridge visual design to behavioral implementation. Each interactive element maps to a specific action.

| Element (from Figma) | Visual Role | Behavioral Intent | Spec Reference | Implementation Pattern |
|----------------------|-------------|-------------------|----------------|------------------------|
| [Button/CTA name] | [Position/purpose] | [Navigation/Mutation/Fetch/etc.] | § [section] | [Pattern: router.push, useMutation, etc.] |
| [Icon/Toggle] | [Location] | [State change description] | § [section] | [useState, context, redux] |

### Keyboard/Accessibility Equivalents
| Visual Interaction | Keyboard Equivalent | Implementation |
|--------------------|---------------------|----------------|
| Click "Submit" button | Enter key in form | `onKeyDown` or form `onSubmit` |
| Click "Close" (modal) | Escape key | `useEffect` keyboard listener |
| Click card | Focus + Enter | `role="button" tabIndex={0}` |

### Missing Behavioral Context
> Flag any elements whose behavior is not specified in the Product Spec.

- [ ] [Element name]: `[TBD - behavior not specified in spec]`

---

## Component Reuse Checklist (Enhanced)

| Figma Instance | Project Component | Props | Events | A11y Required | Import |
|----------------|-------------------|-------|--------|---------------|--------|
| `Button/Primary` | `<Button>` | `variant="primary"` | onClick, onSubmit | aria-label (if icon-only) | `@/components/Button` |
| `Icon/Search` | `<Icon>` | `name="search"` | - | - | `@/components/Icon` |
| `Card/Default` | **[NEW]** | - | - | - | Create in `@/components/Card` |

### Component Usage Guide (with Behavioral Context)

#### Button (`@/components/Button`)

**Props**:
- **Required**: `variant="primary | secondary | ghost"`
- **For icons**: Use `leftIcon` or `rightIcon` props, NOT as children
- **For loading**: Set `isLoading={true}` (shows built-in spinner)

**Events**:
- `onClick`: Standard click handler `(e: MouseEvent) => void`
- `onSubmit`: **Preferred for async** - auto-manages loading state `() => Promise<void>`

**Accessibility**:
- Icon-only buttons MUST have `aria-label`
- Keyboard: Enter/Space triggers click
- Focus ring: Built-in via `focus-visible`

**❌ Anti-Patterns**:
- Don't wrap `<Icon>` in children - use `leftIcon`/`rightIcon` props
- Don't manage loading state manually when using `onSubmit`
- Don't omit `aria-label` on icon-only buttons

**Example**: `<Button variant="primary" leftIcon={<Icon name="search" />}>Search</Button>`

#### Compound Components (Dropdown, Modal, etc.)

> Components requiring context providers:

**Dropdown Pattern**:
```jsx
<DropdownProvider>
  <DropdownTrigger>Open Menu</DropdownTrigger>
  <DropdownMenu>
    <DropdownItem>Option 1</DropdownItem>
  </DropdownMenu>
</DropdownProvider>
```

**⚠️ Context Required**: All parts must be wrapped in provider.

**Accessibility**:
- Arrow keys navigate, Enter selects, Escape closes
- Role: `menu` with `menuitem` children

---

## Responsive Behavior

| Breakpoint | Width | Key Changes |
|------------|-------|-------------|
| Mobile (base) | < 768px | Single column, compact |
| Tablet (md:) | ≥ 768px | Two columns |
| Desktop (lg:) | ≥ 1024px | Three columns, full spacing |

### Responsive Classes Template
```
className="flex-col gap-2 p-4 md:flex-row md:gap-4 md:p-6 lg:gap-6 lg:p-8"
```

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
| Missing focus state | 🟠 HIGH | A11y risk | Add default focus ring |
| Font not in system | 🟢 LOW | Use fallback | Document substitution |
```

---

## 5. Failure Handling

### MCP Tool Failure

If `${MCP_FIGMA_GET_DESIGN}` fails:
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
