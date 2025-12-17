# Figma Token Extraction Automation

> **Purpose**: Ensure all Frontend tasks have pixel-perfect implementation guides by extracting design tokens from Figma before task creation.

---

## When to Trigger

**Primary Trigger**: During **DesignReview** mode when:
- Product Spec contains Figma links
- User has not opted to skip DesignReview

**Secondary Trigger**: During **TaskPlanning** mode for:
- Frontend tasks requiring additional detail
- Tasks where designs changed since DesignReview

---

## MCP Response Schema

> **Purpose**: Document the expected response structure from Figma MCP tools to ensure accurate parsing.

### `mcp_figma-dev-mode-mcp-server_get_design_context` Response

```json
{
  "metadata": {
    "fileKey": "string",
    "nodeId": "string",
    "nodeName": "string",
    "nodeType": "FRAME | COMPONENT | INSTANCE | GROUP | TEXT | VECTOR | ..."
  },
  "structure": {
    "name": "string (layer name)",
    "type": "string (node type)",
    "children": [
      {
        "name": "string",
        "type": "string",
        "isInstance": "boolean (true if component instance)",
        "componentName": "string (if isInstance, e.g., 'Button/Primary')",
        "children": []
      }
    ]
  },
  "layout": {
    "mode": "HORIZONTAL | VERTICAL | NONE (none = absolute positioning)",
    "primaryAxisAlignItems": "MIN | CENTER | MAX | SPACE_BETWEEN",
    "counterAxisAlignItems": "MIN | CENTER | MAX | BASELINE",
    "itemSpacing": "number (gap in pixels)",
    "paddingTop": "number",
    "paddingRight": "number",
    "paddingBottom": "number",
    "paddingLeft": "number",
    "layoutSizingHorizontal": "HUG | FILL | FIXED",
    "layoutSizingVertical": "HUG | FILL | FIXED"
  },
  "styles": {
    "fills": [
      {
        "type": "SOLID | GRADIENT_LINEAR | IMAGE",
        "color": { "r": 0-1, "g": 0-1, "b": 0-1, "a": 0-1 },
        "styleName": "string (if linked to style, e.g., 'Primary/500')"
      }
    ],
    "strokes": [
      {
        "color": { "r": 0-1, "g": 0-1, "b": 0-1, "a": 0-1 },
        "weight": "number (border width)",
        "styleName": "string"
      }
    ],
    "effects": [
      {
        "type": "DROP_SHADOW | INNER_SHADOW | LAYER_BLUR",
        "offset": { "x": "number", "y": "number" },
        "radius": "number (blur)",
        "spread": "number",
        "color": { "r": 0-1, "g": 0-1, "b": 0-1, "a": 0-1 }
      }
    ],
    "cornerRadius": "number | { topLeft, topRight, bottomRight, bottomLeft }"
  },
  "typography": {
    "fontFamily": "string (e.g., 'Inter')",
    "fontSize": "number",
    "fontWeight": "number (100-900)",
    "lineHeight": { "value": "number", "unit": "PIXELS | PERCENT | AUTO" },
    "letterSpacing": { "value": "number", "unit": "PIXELS | PERCENT" },
    "textAlignHorizontal": "LEFT | CENTER | RIGHT | JUSTIFIED",
    "styleName": "string (if linked to text style)"
  },
  "variants": {
    "isComponentInstance": "boolean",
    "componentProperties": {
      "PropertyName": "value (e.g., { 'State': 'Hover', 'Size': 'Large' })"
    },
    "availableVariants": ["Default", "Hover", "Disabled", "Focused"]
  },
  "constraints": {
    "horizontal": "MIN | CENTER | MAX | STRETCH | SCALE",
    "vertical": "MIN | CENTER | MAX | STRETCH | SCALE"
  },
  "boundingBox": {
    "width": "number",
    "height": "number",
    "x": "number (position)",
    "y": "number (position)"
  }
}
```

### Key Parsing Notes

| Field | Parsing Logic |
|-------|---------------|
| `layout.mode = NONE` | Frame uses absolute positioning; spacing values are unreliable |
| `structure.isInstance = true` | Map to existing project component; check `componentName` |
| `styles.fills[].styleName` | Prefer style name over raw color for token mapping |
| `typography.styleName` | Prefer style name; if missing, map raw values |
| `variants.availableVariants` | Generate CSS states for each variant |
| `constraints.horizontal = STRETCH` | Use `w-full` or `flex-1` |

### Color Conversion Formula

Figma returns colors as 0-1 floats. Convert to hex:

```
R = Math.round(color.r * 255)
G = Math.round(color.g * 255)  
B = Math.round(color.b * 255)
Hex = #${R.toString(16).padStart(2,'0')}${G.toString(16).padStart(2,'0')}${B.toString(16).padStart(2,'0')}
```

---

## Automation Process

### Step 1: Node Selection Protocol

> **Goal**: Identify the correct Figma frame(s) to extract, even when the URL is incomplete.

#### Step 1A: Parse Figma URL

Extract from Epic description:
- **File key**: From URL path (e.g., `figma.com/design/{fileKey}/...`)
- **Node ID**: From query param `node-id=XXXXX-XXXXX` (may be missing)

**URL Patterns**:
| Pattern | Example | Has Node ID? |
|---------|---------|--------------|
| Full node URL | `figma.com/design/abc123/File?node-id=1-234` | ✅ Yes |
| Page URL | `figma.com/design/abc123/File` | ❌ No |
| Branch URL | `figma.com/design/abc123/branch/xyz/File?node-id=1-234` | ✅ Yes |

#### Step 1B: Decision Tree

```
Does URL contain node-id?
├── YES → Use node ID directly → Go to Step 2
└── NO
    └── Call mcp_figma-dev-mode-mcp-server_get_metadata to list all frames on the page
        └── How many top-level frames?
            ├── 1 frame → Use that frame → Go to Step 2
            ├── 2-10 frames → Try automatic matching (Step 1C)
            └── 10+ frames → Ask user to specify
```

#### Step 1C: Automatic Frame Matching

When multiple frames exist, match by name:

1. **Extract task component name** from task title
   - E.g., "Implement SearchWidget" → look for "SearchWidget"

2. **Search frame names** for matches:
   ```
   Frame names from mcp_figma-dev-mode-mcp-server_get_metadata:
   - "SearchWidget" ← MATCH
   - "SearchWidget/Desktop"
   - "SearchWidget/Mobile"
   - "Header"
   - "Footer"
   ```

3. **Selection rules**:
   | Scenario | Action |
   |----------|--------|
   | Exact match found | Use that frame |
   | Multiple matches (Desktop/Mobile) | Extract primary (Desktop), note variants |
   | Partial match (e.g., "Search" vs "SearchWidget") | Use with confirmation |
   | No match | Ask user to clarify |

#### Step 1D: Handle Variants

If design has viewport variants:

```markdown
**Detected Figma Variants**:
- `SearchWidget/Desktop` (node-id: 1-234) ← PRIMARY
- `SearchWidget/Mobile` (node-id: 1-567)
- `SearchWidget/Tablet` (node-id: 1-890)

> Extracting Desktop variant. Mobile/Tablet tokens available if responsive behavior needed.
```

#### Step 1E: Fallback - Request Clarification

If automatic matching fails:

```markdown
⚠️ **Figma Frame Selection Required**

Found multiple frames, unable to auto-match:
1. `Homepage` (node-id: 1-100)
2. `SearchResults` (node-id: 1-200)
3. `ProductDetail` (node-id: 1-300)

Please specify which frame to use for task "[TaskName]":
- Provide node-id, OR
- Provide frame name
```

### Step 2: Extract Design Context (Enhanced)

For each Frontend task, call `mcp_figma-dev-mode-mcp-server_get_design_context` with node ID and extract ALL of the following:

#### 2A. Component Tree (Structure)
- **Root node**: Type (Frame/Component/Instance) and layer name
- **Child nodes**: List with types, names, and nesting depth (up to 3 levels)
- **Semantic hints**: Infer purpose from layer names (e.g., "Header", "CardContainer", "ActionBar")
- **Component Instances**: Identify reusable component references (e.g., `Button/Primary`, `Icon/Search`)

#### 2B. Auto-Layout Properties
Extract for each auto-layout frame:
- **Direction**: `row` | `column` | `wrap`
- **Primary axis align**: `start` | `center` | `end` | `space-between` | `space-around`
- **Cross axis align**: `start` | `center` | `end` | `stretch` | `baseline`
- **Gap**: Horizontal and vertical spacing between items
- **Padding**: Top, right, bottom, left values
- **Sizing mode**: `hug` | `fill` | `fixed` for width and height

#### 2C. Visual Tokens
- **Background**: Solid color, gradient, or image fill
- **Border**: Width, radius (per corner if different), color
- **Shadow**: Type (drop/inner), offset X/Y, blur, spread, color
- **Text styles**: Font family, size, weight, line height, letter spacing
- **Colors**: All color values with opacity

#### 2D. Component Variants (if component/instance)
- **Variant properties**: List all variant axes and values
- **Current state**: Which variant is currently displayed
- **Key states to implement**: default, hover, active, disabled, focused

#### 2E. Responsive Design Extraction

> **Goal**: Extract all viewport variants and generate responsive token mappings.

**Step 2E.1: Identify Viewport Variants**

From Step 1D, if multiple variants were detected:

| Variant Pattern | Breakpoint Mapping |
|-----------------|-------------------|
| `*/Mobile` or `*/Phone` | Default (mobile-first) |
| `*/Tablet` or `*/iPad` | `md:` (768px+) |
| `*/Desktop` or `*/Web` | `lg:` (1024px+) |
| `*/Wide` or `*/1440` | `xl:` (1280px+) |

**Step 2E.2: Extract Each Variant**

For each detected variant, run Step 2A-2D extraction:

```markdown
Extracting: SearchWidget/Mobile (375px) → Base styles
Extracting: SearchWidget/Tablet (768px) → md: overrides
Extracting: SearchWidget/Desktop (1024px) → lg: overrides
```

**Step 2E.3: Generate Responsive Diff**

Compare tokens between variants to identify what changes:

```markdown
**Responsive Token Mapping**:

| Property | Mobile (base) | Tablet (md:) | Desktop (lg:) |
|----------|---------------|--------------|---------------|
| Container width | w-full | max-w-2xl | max-w-4xl |
| Gap | gap-2 | gap-4 | gap-6 |
| Padding | p-4 | p-6 | p-8 |
| Font size (heading) | text-lg | text-xl | text-2xl |
| Layout direction | flex-col | flex-row | flex-row |
| Visibility | hidden | block | block |
```

**Step 2E.4: Output Responsive Classes**

Convert to mobile-first Tailwind pattern:

```css
/* Generated responsive classes */
.container {
  @apply w-full p-4 gap-2 flex-col;        /* Mobile base */
  @apply md:max-w-2xl md:p-6 md:gap-4 md:flex-row;  /* Tablet */
  @apply lg:max-w-4xl lg:p-8 lg:gap-6;     /* Desktop */
}
```

**Step 2E.5: Handle Missing Variants**

| Scenario | Action |
|----------|--------|
| Only Desktop provided | Use as-is, note: "Responsive variants not designed" |
| Only Mobile provided | Use as base, extrapolate Desktop with confirmation |
| Partial variants | Extract available, flag missing with `[TBD - Design]` |

**Responsive Output in UI Implementation Guide**:

```markdown
**Responsive Behavior**:

| Breakpoint | Width | Key Changes |
|------------|-------|-------------|
| Mobile (base) | < 768px | Single column, compact spacing |
| Tablet (md:) | ≥ 768px | Two columns, increased padding |
| Desktop (lg:) | ≥ 1024px | Three columns, full width |

> ⚠️ Only Desktop variant provided. Mobile behavior needs designer input.
```

#### 2F. Asset Extraction

> **Goal**: Identify images, icons, and illustrations that need to be exported and referenced.

**Step 2F.1: Detect Asset Types**

Scan the extracted frame for:

| Fill/Node Type | Asset Type | Detection |
|---------------|------------|-----------|
| `fills[].type = IMAGE` | Raster image | Background/foreground images |
| `type = VECTOR` | Illustration | Custom vector graphics |
| `isInstance = true` + `componentName` starts with `Icon/` | Icon | Existing icon component |
| `isInstance = true` + `componentName` starts with `Illustration/` | Illustration | Reusable illustration |

**Step 2F.2: Classify and Recommend Export**

| Asset Type | Recommended Format | Resolution |
|------------|-------------------|------------|
| Photo/background image | PNG or WebP | @1x and @2x |
| Illustration (simple) | SVG | N/A (vector) |
| Illustration (complex gradients) | PNG | @2x |
| Icon | SVG or use existing icon font | N/A |
| Logo | SVG | N/A |

**Step 2F.3: Check Against Existing Assets**

Before recommending export, check if asset already exists:

1. **Icons**: Check `component-registry.json` → `Icon` component for matching `name` prop
2. **Images**: Check `public/images/` or `src/assets/` for similar filenames
3. **Illustrations**: Check `public/icons/` or `src/assets/illustrations/`

**Step 2F.4: Generate Asset Manifest**

Output in UI Implementation Guide:

```markdown
**Required Assets**:

| Asset | Source | Type | Export Format | Target Path | Status |
|-------|--------|------|---------------|-------------|--------|
| hero-banner | Figma fill | Raster | PNG @2x | public/images/hero-banner.png | 🆕 Export needed |
| product-illustration | Figma vector | SVG | SVG | public/icons/product-illustration.svg | 🆕 Export needed |
| facebook | Icon instance | Icon | - | Use `<Icon name="facebook">` | ✅ Existing |
| search | Icon instance | Icon | - | Use `<Icon name="search">` | ✅ Existing |

> **Designer Action**: Export assets marked 🆕 from Figma and place in target paths.
```

**Step 2F.5: Asset Naming Convention**

Generate file names using:

| Pattern | Example |
|---------|---------|
| Component + purpose | `search-widget-hero.png` |
| kebab-case | `product-card-illustration.svg` |
| No spaces or special chars | ✅ `user-avatar.png` ❌ `User Avatar (1).png` |

**Step 2F.6: Handle Missing Assets**

If Figma MCP can't access image data:

```markdown
**Required Assets**:

| Asset | Source | Type | Export Format | Target Path | Status |
|-------|--------|------|---------------|-------------|--------|
| [IMAGE FILL] | Node 1-234 | Raster | PNG @2x | [TBD] | ⚠️ Manual export required |

> ⚠️ Image fill detected but cannot be auto-extracted. Designer must export from Figma.
```

#### 2G. Extract Figma Variables (Token Priority)

> **Purpose**: Figma Variables represent designer-defined semantic tokens. When available, they provide the most accurate source for token mapping.

**Step 2G.1: Call Variable Extraction**

After extracting design context, call `mcp_figma-dev-mode-mcp-server_get_variable_defs` with the same node ID:

```
mcp_figma-dev-mode-mcp-server_get_variable_defs(nodeId: "[node-id]")
```

**Step 2G.2: Parse Variable Response**

Expected response structure:
```json
{
  "variables": {
    "color/primary/500": "#3B82F6",
    "color/neutral/100": "#F3F4F6",
    "spacing/md": "16px",
    "radius/lg": "8px"
  }
}
```

**Step 2G.3: Variable-to-Token Mapping**

| Figma Variable Pattern | Maps To | Example |
|------------------------|---------|---------|
| `color/{category}/{shade}` | `{category}-{shade}` | `color/primary/500` → `primary-500` |
| `color/{name}` | Direct name | `color/danger` → `danger` |
| `spacing/{size}` | Spacing scale | `spacing/md` → `4` (16px) |
| `radius/{size}` | Border radius | `radius/lg` → `rounded-lg` |
| `font/{property}/{value}` | Typography | `font/size/lg` → `text-lg` |

**Step 2G.4: Token Resolution Priority**

When mapping Figma values to project tokens, use this priority order:

| Priority | Source | Confidence | Annotation |
|----------|--------|------------|------------|
| 1 | Figma Variable (from 2G) | ✅ High | None needed |
| 2 | Figma Style Name (from 2C) | ✅ High | None needed |
| 3 | `design-tokens.json` match | ⚡ Medium | None if exact |
| 4 | Algorithmic closest match | ⚠️ Low | Add `⚠️ (closest match)` |

**Step 2G.5: Merge Variable Data**

Enhance the Token Mapping table with variable source:

```markdown
**Token Mapping** (Figma → Project):
| Category | Figma Value | Variable | Project Token |
|----------|-------------|----------|---------------|
| Background | #3B82F6 | color/primary/500 | bg-primary-500 |
| Text | #111827 | color/neutral/900 | text-gray-900 |
| Spacing | 16px | spacing/md | gap-4 |
| Background | #F0F1F3 | - | bg-gray-100 ⚠️ (no variable) |
```

**Step 2G.6: Handle Variable Unavailability**

| Scenario | Action |
|----------|--------|
| `mcp_figma-dev-mode-mcp-server_get_variable_defs` fails | Proceed with Style Names and algorithmic matching |
| No variables defined in Figma | Note: "Figma Variables not configured" |
| Partial variables | Use available, fallback for rest |

```markdown
> ℹ️ **Figma Variables**: Not configured for this file. Using style names and algorithmic matching.
```

### Step 2H: Screenshot Extraction (Visual Grounding)

> **Purpose**: LLMs are language models but UI is visual. Screenshots provide critical visual context that token tables alone cannot convey.

**Step 2H.1: Capture Design Screenshot**

For each extracted frame, call `mcp_figma-dev-mode-mcp-server_get_screenshot`:

```
mcp_figma-dev-mode-mcp-server_get_screenshot(nodeId: "[node-id]")
```

**Step 2H.2: Screenshot Capture Strategy**

| Context | What to Capture |
|---------|-----------------|
| Full component | Primary frame at 1x scale |
| Responsive variants | Each breakpoint variant (Mobile, Tablet, Desktop) |
| Interactive states | Separate screenshots for Default, Hover, Disabled if visually distinct |
| Empty/Error states | Separate screenshots if they exist in design |

**Step 2H.3: Screenshot Embedding Format**

Embed screenshots in markdown using:

```markdown
### Visual Reference

![SearchWidget - Desktop](file:///path/to/screenshot-desktop.png)

> **Critical Visual Notes**:
> - Button row is pinned to bottom (space-between, not gap-stacking)
> - Content area scrolls if overflow
> - Title has larger visual weight (24px margin-bottom vs 16px elsewhere)
```

**Step 2H.4: Visual Annotations**

When capturing, add inline annotations for:

| Pattern | Annotation |
|---------|------------|
| Pinned elements | "Footer pinned to bottom with `mt-auto`" |
| Scroll areas | "Content area has `overflow-auto`" |
| Optical alignment | "Logo offset 4px left for optical center" |
| Visual rhythm | "Larger gap (24px) before CTA section" |

**Step 2H.5: Handle Screenshot Failure**

| Scenario | Action |
|----------|--------|
| MCP screenshot fails | Request screenshot from user |
| Very large frame | Capture at 0.5x scale, note: "Low-res preview" |
| Multiple pages | Capture each page section separately |

---

### Step 2I: Interaction & Animation Context

> **Purpose**: UI is not static. Components have states, transitions, and micro-interactions that must be explicitly documented.

**Step 2I.1: Extract Component Variants (States)**

From `mcp_figma-dev-mode-mcp-server_get_design_context` response, parse `variants` field:

```json
"variants": {
  "availableVariants": ["Default", "Hover", "Pressed", "Disabled", "Focused"]
}
```

**Step 2I.2: State Extraction Table**

For each available variant, extract visual differences:

```markdown
### Interaction States

| State | Background | Border | Shadow | Transform | Transition |
|-------|------------|--------|--------|-----------|------------|
| Default | bg-primary-500 | - | shadow-sm | - | - |
| Hover | bg-primary-600 | - | shadow-md | scale(1.02) | 150ms ease-out |
| Pressed | bg-primary-700 | - | shadow-sm | scale(0.98) | 50ms ease-in |
| Focused | bg-primary-500 | ring-2 ring-primary-300 | shadow-sm | - | 100ms |
| Disabled | bg-gray-300 | - | - | - | - |
```

**Step 2I.3: Transition Timing Defaults**

If Figma doesn't specify transition timing, use these defaults:

| Interaction | Duration | Easing |
|-------------|----------|--------|
| Hover effects | 150ms | ease-out |
| Active/Pressed | 50ms | ease-in |
| Focus ring | 100ms | ease-in-out |
| Modal/Drawer open | 200ms | ease-out |
| Modal/Drawer close | 150ms | ease-in |
| Skeleton shimmer | 1500ms | linear (infinite) |

**Step 2I.4: Micro-Interaction Patterns**

Identify and document common micro-interactions:

| Pattern | Detection | Output |
|---------|-----------|--------|
| Button press | Hover + Pressed variants exist | `active:scale-95 transition-transform` |
| Input focus | Focus variant with ring | `focus:ring-2 focus:ring-primary-300` |
| Checkbox toggle | Checked/Unchecked variants | `transition-colors duration-150` |
| Skeleton loading | Gradient fill with animation | `animate-pulse` or custom shimmer |
| Success feedback | Success state variant | `animate-bounce` or checkmark transition |

**Step 2I.5: Output Format in UI Implementation Guide**

```markdown
### Interaction Behavior

**State Machine**:
```
Default → Hover (on mouseenter) → Pressed (on mousedown) → Default (on mouseup)
         ↓
      Focused (on keyboard focus)
         ↓
      Disabled (when isDisabled=true)
```

**CSS Transitions**:
```css
.button {
  @apply transition-all duration-150 ease-out;
}
.button:hover {
  @apply bg-primary-600 shadow-md scale-102;
}
.button:active {
  @apply bg-primary-700 shadow-sm scale-98 duration-50 ease-in;
}
.button:focus-visible {
  @apply ring-2 ring-primary-300 ring-offset-2;
}
```

**Tailwind Classes**:
```
transition-all duration-150 ease-out hover:bg-primary-600 hover:shadow-md hover:scale-102 active:scale-98 active:duration-50 focus-visible:ring-2 focus-visible:ring-primary-300
```
```

**Step 2I.6: Handle Missing Interaction States**

| Scenario | Action |
|----------|--------|
| No variants defined | Use industry-standard defaults, mark as `[ASSUMED]` |
| Only Default + Hover | Extrapolate Pressed = darker shade, mark as `[EXTRAPOLATED]` |
| No Disabled state | Flag: "⚠️ Disabled state not designed" |
| No Focus state | Flag: "⚠️ Focus state missing - a11y risk" |

---

### Step 2J: Spatial Constraints Analysis

> **Purpose**: LLMs understand tokens but struggle with spatial reasoning. This step captures scroll behavior, pinned elements, z-index relationships, and positioning patterns that cannot be inferred from token tables.

**Step 2J.1: Identify Scroll Containers**

Analyze frame structure for overflow behavior:

| Figma Signal | Detection | Constraint Type |
|--------------|-----------|-----------------|
| Frame with `layoutSizingVertical = FILL` + children exceed height | Likely scroll container | `overflow-auto` |
| Frame with `clipsContent = true` | Clips overflow | `overflow-hidden` or `overflow-auto` |
| Frame name contains "Scroll", "List", "Content" | Designer intent | `overflow-auto` |

**Step 2J.2: Detect Pinned/Sticky Elements**

Analyze positioning within parent containers:

| Figma Signal | Detection | CSS Implementation |
|--------------|-----------|-------------------|
| Last child in vertical layout with `constraints.vertical = MAX` | Pinned footer | `mt-auto` or `sticky bottom-0` |
| First child with `constraints.vertical = MIN` | Pinned header | `sticky top-0` |
| Frame outside auto-layout, positioned at corner | FAB/floating | `fixed` or `absolute` |
| Frame with `layoutPositioning = ABSOLUTE` | Overlay element | `absolute` or `fixed` |

**Step 2J.3: Map z-index Relationships**

Determine layer stacking from Figma structure:

```
Layer Order (from Figma):
1. Background elements (lowest layer) → z-0
2. Main content → z-0 (base)
3. Sticky headers → z-10
4. Floating elements (FAB) → z-30
5. Dropdowns/Popovers → z-40
6. Modals/Overlays (highest layer) → z-50
```

**Detection Heuristics**:
| Figma Pattern | Likely Constraint | CSS Implementation |
|---------------|-------------------|-------------------|
| Frame outside auto-layout at top of layer list | Overlay/modal | `fixed inset-0 z-50` |
| Frame with dark/blur fill covering full bounds | Backdrop | `fixed inset-0 z-40 bg-black/50` |
| Small button frame at corner, high in layer order | FAB | `fixed bottom-4 right-4 z-30` |
| Frame with `constraints = STRETCH` in both axes | Full-bleed | `absolute inset-0` or `w-full h-full` |

**Step 2J.4: Viewport vs Container Positioning**

Determine positioning context:

| Positioning Type | When to Use | Detection |
|------------------|-------------|-----------|
| `fixed` | Element relative to viewport (stays during page scroll) | FABs, modals, toasts, global navs |
| `sticky` | Element relative to scroll container (stays during section scroll) | Section headers, action bars |
| `absolute` | Element relative to positioned parent | Badges, overlays within cards |
| `static` (default) | Normal document flow | Most elements |

**Step 2J.5: Output Format**

Add to UI Implementation Guide:

```markdown
## Spatial Constraints

> **Purpose**: Scroll, pinned, and z-index behavior not inferable from tokens.

| Element | Constraint Type | Behavior | CSS Implementation |
|---------|-----------------|----------|-------------------|
| HeaderBar | Sticky | Sticks to top during content scroll | `sticky top-0 z-10` |
| ContentArea | Scroll Container | Scrolls when content overflows | `flex-1 overflow-auto` |
| ActionBar | Pinned Footer | Always visible at bottom | `mt-auto` |
| ModalOverlay | Fixed Overlay | Covers entire viewport | `fixed inset-0 z-50 bg-black/50` |
| FAB | Fixed Position | Bottom-right of viewport | `fixed bottom-4 right-4 z-40` |

### Scroll & Overflow Structure
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
| Floating | 30 | FAB, floating buttons |
| Sticky | 10 | Headers, sticky navigation |
| Base | 0 | Main content |
```

**Step 2J.6: Handle Ambiguous Cases**

| Scenario | Action |
|----------|--------|
| Can't determine if sticky vs fixed | Default to `sticky`, flag for designer confirmation |
| Multiple scroll containers nested | Document each level, warn about complexity |
| No clear pinning in Figma | Mark as `[TBD - confirm scroll behavior with designer]` |

---

### Step 2K: Content Constraints Analysis

> **Purpose**: Figma shows ideal content ("John Doe", 3 items). LLMs need to know how components behave with real-world variable content (long text, 0 items, 100+ items).

**Step 2K.1: Identify Text Overflow Patterns**

For each text layer in the extracted frame:

| Detection | Question | Constraint Types |
|-----------|----------|------------------|
| Text within constrained width frame | What if text is 2-5x longer? | Truncate, wrap, or expand? |
| Single-line text with fixed height | Long text will overflow | Truncate with ellipsis |
| Multi-line text | How many lines before cutoff? | Line clamp (2-3 lines typical) |
| Button/label text | Should never wrap | `whitespace-nowrap min-w-[X]` |

**Common Patterns by Element Type**:
| Element | Max Length | Overflow Behavior | CSS Implementation |
|---------|------------|-------------------|-------------------|
| User Name | 30 chars | Truncate | `truncate max-w-[200px]` |
| Email | 50 chars | Truncate | `truncate` |
| Card Title | 50 chars | Truncate | `truncate` |
| Description | 150 chars | Clamp 2-3 lines | `line-clamp-2` |
| Button Label | 20 chars | No wrap, min-width | `whitespace-nowrap min-w-[80px]` |
| Table Cell | Varies | Truncate or wrap | Context-dependent |

**Step 2K.2: Identify List/Array Patterns**

For each repeating pattern (grid, list, cards):

| Detection | Questions to Answer |
|-----------|---------------------|
| Repeating child elements | What if 0 items? (empty state) |
| Showing N items | What if 100+ items? (pagination/virtualization) |
| Tag/chip list | What if 10+ tags? (show +N more) |

**Common Patterns**:
| List Type | Empty State | Overflow Behavior | Threshold |
|-----------|-------------|-------------------|-----------|
| Product Grid | "No products found" + CTA | Paginate | 12-24 per page |
| Notification List | "All caught up!" illustration | Virtualize | After 50 items |
| Search Results | "No matches for [query]" | Infinite scroll | Batch of 20 |
| Tag List | Hide section | "+N more" pill | After 5 visible |
| Dropdown Options | "No options" | Virtual scroll | After 100 |

**Step 2K.3: Required State Checklist**

For each component, verify these states are accounted for:

```markdown
### Required States
- [ ] **Empty State**: What to show when list has 0 items
- [ ] **Loading State**: Skeleton/spinner while data loads
- [ ] **Error State**: Message when API/data fetch fails
- [ ] **Partial State**: Fallback for missing optional data (e.g., avatar → initials)
```

**Detection from Figma**:
| Figma Signal | State Type |
|--------------|------------|
| Frame named "*Empty*", "*No Data*", "*Zero State*" | Empty State |
| Frame with skeleton/pulse animation | Loading State |
| Frame named "*Error*", "*Failed*" | Error State |
| Avatar with initials instead of image | Partial/Fallback State |

**Step 2K.4: Dynamic Content Formatting**

Document formatting expectations for dynamic data:

| Content Type | Format | Example | Notes |
|--------------|--------|---------|-------|
| Price/Currency | Locale-aware | `$1,234.56` | Use `Intl.NumberFormat` |
| Date < 7 days | Relative | "2 hours ago" | Use `date-fns/formatDistance` |
| Date >= 7 days | Absolute | "Dec 15, 2024" | Use locale format |
| Large numbers | Compact | `12.5K`, `1.2M` | Use `Intl.NumberFormat` compact |
| Phone | E.164 display | `+1 (555) 123-4567` | Format after validation |
| File size | Human readable | `2.4 MB` | Use appropriate unit |

**Step 2K.5: Output Format**

Add to UI Implementation Guide:

```markdown
## Content Constraints

> **Purpose**: Handle real-world variable content, not just Figma's ideal mockup.

### Text Overflow
| Element | Max | Overflow | CSS |
|---------|-----|----------|-----|
| User Name | 30 chars | Truncate | `truncate max-w-[200px]` |
| Description | 150 chars | Clamp 2 lines | `line-clamp-2` |

### List Handling
| Element | Empty State | Pagination/Virtualization |
|---------|-------------|---------------------------|
| Products | "No products found" | Paginate 12/page |
| Comments | "No comments yet" | Load more button |

### Required States
- [ ] Empty: `<EmptyState icon="box" message="..." />`
- [ ] Loading: `<Skeleton count={3} />`
- [ ] Error: `<ErrorMessage retry={refetch} />`
```

**Step 2K.6: Handle Missing Content Specs**

| Scenario | Action |
|----------|--------|
| Figma shows no empty state frame | Mark as `[TBD - empty state design needed]` |
| No max length defined | Estimate from container width, flag for product confirmation |
| List without pagination | Recommend virtualization threshold, flag for product decision |

---

### Step 2L: Semantic Intent Extraction

> **Purpose**: Figma shows *appearance*; behavior lives in Product Specs. LLMs need explicit intent to generate correct event handlers, not just correct JSX structure.

**Step 2L.1: Scan for Interactive Elements**

From the extracted frame structure, identify all interactive elements:

| Element Type | Detection Signal | Needs Intent? |
|--------------|------------------|---------------|
| Button | `isInstance = true` + componentName contains "Button" | ✅ Yes |
| Link | Text with underline OR componentName contains "Link" | ✅ Yes |
| Icon (standalone) | Small vector/icon outside of other components | ⚠️ Maybe (could be decorative) |
| Card (clickable) | Frame with hover variant OR cursor pointer | ✅ Yes |
| Input/Form field | Input component instance | ✅ Yes (for submit behavior) |
| Toggle/Checkbox | Switch or checkbox component | ✅ Yes |
| Tab/Accordion | Repeated similar elements with selection state | ✅ Yes |

**Step 2L.2: Cross-Reference with Product Spec**

For each interactive element, search the Product Spec (Confluence/Jira) for:

1. **User Story Match**: Does a user story mention this element?
   - "As a user, I can click 'Add to Cart' to add item to my cart"
   
2. **Flow Reference**: Is this element part of a documented flow?
   - "Step 3: User clicks 'Continue' → navigates to checkout"

3. **Acceptance Criteria**: Does an AC mention the behavior?
   - "GIVEN user on product page WHEN clicks 'Add to Cart' THEN item added and toast shown"

**Step 2L.3: Classify Intent Type**

| Intent Type | Characteristics | Implementation Pattern |
|-------------|-----------------|------------------------|
| **Navigation** | Changes URL, shows different view | `router.push()`, `<Link>`, `navigate()` |
| **State Toggle** | Flips boolean, no server call | `useState`, `useReducer`, prop change |
| **Data Mutation** | Creates/Updates/Deletes data | `useMutation`, API call, optimistic update |
| **Data Fetch** | Loads more data, filters, searches | `useQuery`, `useInfiniteQuery`, debounce |
| **Modal/Overlay** | Opens overlay without navigation | State for isOpen, portal rendering |
| **Form Submit** | Validates and sends form data | Form library submit, validation trigger |
| **External Action** | Browser API, external link | `window.open()`, clipboard, share API |
| **Disclosure** | Expand/collapse content in place | Accordion state, height animation |

**Step 2L.4: Document Intent Map**

For each interactive element, create an entry:

```markdown
| Element | Visual Role | Intent Type | Behavior Description | Spec Ref | Pattern |
|---------|-------------|-------------|----------------------|----------|---------|
| "Add to Cart" btn | Primary CTA below price | Data Mutation | POST to cart API, optimistic add, show toast | AC-3.2.1 | `useMutation` + toast |
| Heart icon | Top-right of card | State Toggle | Toggle wishlist, persist to API | US-4.1 | `useState` + debounced API |
| Product card | Entire clickable area | Navigation | Go to /products/{slug} | Flow 3.1 | `<Link>` wrapper or router |
| "Load More" | Bottom of list | Data Fetch | Fetch next page, append | US-2.5 | `useInfiniteQuery` |
| Size selector | Below product image | State Change | Update selected variant (local) | AC-3.2.3 | `useState` for selection |
```

**Step 2L.5: Detect Implicit Behaviors**

Many behaviors are implied, not explicit:

| Implicit Pattern | How to Detect | What to Document |
|------------------|---------------|------------------|
| Card = link to detail | Card has hover state, same destination as "View" button | Document: "Card click = same as View Details" |
| Enter = Submit | Form with submit button | Document: "Enter in last field triggers submit" |
| Escape = Close | Modal/drawer component | Document: "Escape key closes modal" |
| Click outside = Close | Modal with backdrop | Document: "Backdrop click closes modal" |
| Swipe = Dismiss | Mobile card with gesture hints | Document: "Swipe left triggers delete" |

**Step 2L.6: Handle Missing/Ambiguous Intent**

| Scenario | Action |
|----------|--------|
| Element visible in Figma, no spec reference | Add to Missing Context: `[TBD - "{element}" behavior not in spec]` |
| Spec reference ambiguous | List options: "Could be: (A) modal, (B) page navigation. Clarify." |
| Multiple elements → same action | Document: "Both [X] and [Y] trigger [action]" |
| Decorative element (no interaction) | Skip from intent map, note as "decorative" |

**Step 2L.7: Output for Design Review Report**

Add to report:

```markdown
## Semantic Intent Map

> **Purpose**: Maps visual elements to behavioral implementation patterns.

| Element | Visual Role | Intent | Spec Ref | Implementation |
|---------|-------------|--------|----------|----------------|
| "View Details" | Card CTA | Navigation | § 3.2 | `<Link to="/products/{id}">` |
| Heart icon | Wishlist toggle | Mutation | § 4.1 | `useMutation('wishlist')` |
| Product card | Clickable area | Navigation | § 3.2 | Same as "View Details" |
| "Load More" | Pagination | Fetch | § 3.1 | `useInfiniteQuery` |

### Keyboard Equivalents
| Element | Mouse | Keyboard |
|---------|-------|----------|
| Submit form | Click "Submit" | Enter in last field |
| Close modal | Click X or backdrop | Escape key |
| Navigate card | Click anywhere | Focus + Enter |

### Missing Behavioral Context
> ⚠️ These elements need spec clarification before implementation:

- [ ] "Share" button: `[TBD - opens native share? copies link? shows modal?]`
- [ ] Filter chips: `[TBD - single-select or multi-select?]`
```

---

### Step 3: Map to Project Design System

> **MANDATORY**: Use [`token-mapping-rules.md`](./token-mapping-rules.md) for all token conversions.

**Read project design tokens**:
- For Tailwind: `tailwind.config.js` (theme.extend.colors, spacing, etc.)
- For CSS variables: `theme.ts`, `variables.scss`, or `:root` definitions
- For component library: `.ai-instructions/analysis/file-categorization.json`

**Apply Token Mapping Rules** (from `token-mapping-rules.md`):

| Category | Rule Reference |
|----------|----------------|
| Colors | § 1 - RGB distance algorithm, threshold 30 for flagging |
| Spacing | § 2 - Round to Tailwind 4px scale |
| Typography | § 3 - Font size, weight, line-height mappings |
| Border Radius | § 4 - Pixel to rounded-* conversion |
| Shadows | § 5 - Match by blur radius primarily |
| Layout | § 6 - Flex direction, alignment, sizing |
| Fallbacks | § 7 - Annotation format for approximate matches |

### Step 3.5: Match Component Instances (Enhanced Schema)

> **MANDATORY**: Use project's `component-registry.json` for deterministic matching WITH full behavioral context.

**Read component registry**: `.ai-instructions/analysis/component-registry.json`

**Enhanced Registry Schema** (generated by `/map-codebase-agent`):

```json
{
  "Button": {
    "path": "src/components/Button/Button.tsx",
    "figmaMapping": {
      "Button/Primary": { "variant": "primary" },
      "Button/Secondary": { "variant": "secondary" },
      "Button/Ghost": { "variant": "ghost" }
    },
    "props": {
      "variant": { "type": "primary | secondary | ghost", "required": true },
      "size": { "type": "sm | md | lg", "default": "md" },
      "isLoading": { "type": "boolean", "default": false, "description": "Shows spinner, disables click" },
      "isDisabled": { "type": "boolean", "default": false }
    },
    "slots": {
      "children": { "type": "ReactNode", "description": "Button label text" },
      "leftIcon": { "type": "ReactNode", "description": "Icon before label" },
      "rightIcon": { "type": "ReactNode", "description": "Icon after label" }
    },
    "events": {
      "onClick": { 
        "type": "(e: MouseEvent) => void", 
        "description": "Standard click handler" 
      },
      "onSubmit": { 
        "type": "() => Promise<void>", 
        "description": "Async submit handler - shows loading state automatically",
        "preferred": true,
        "note": "Use this instead of onClick + manual loading state"
      }
    },
    "a11y": {
      "role": "button",
      "requiredAria": [
        "aria-label (required if icon-only, no children text)"
      ],
      "keyboardNav": "Enter/Space triggers onClick",
      "focusVisible": "Built-in focus ring via focus-visible"
    },
    "composition": {
      "contextRequired": null,
      "validChildren": ["text", "Icon"],
      "forbiddenChildren": ["Button", "Link", "a"],
      "validParents": ["form", "div", "ButtonGroup", "CardActions"]
    },
    "stateManagement": {
      "type": "uncontrolled",
      "internalState": ["isLoading (auto-managed when using onSubmit)"],
      "note": "Loading state is automatically managed when using onSubmit prop"
    },
    "antiPatterns": [
      "❌ Don't put <Icon> as direct child - use leftIcon/rightIcon props",
      "❌ Don't manage loading state manually when using onSubmit",
      "❌ Don't nest interactive elements (buttons, links) inside Button",
      "❌ Don't omit aria-label on icon-only buttons"
    ],
    "stateProps": ["isLoading", "isDisabled"],
    "usage": "<Button variant=\"primary\" leftIcon={<Icon name=\"search\" />}>Search</Button>"
  },
  
  "Input": {
    "path": "src/components/Input/Input.tsx",
    "props": {...},
    "events": {
      "onChange": { "type": "(e: ChangeEvent<HTMLInputElement>) => void" },
      "onBlur": { "type": "(e: FocusEvent) => void" }
    },
    "a11y": {
      "requiredAria": ["aria-label or associated label element"],
      "keyboardNav": "Standard input navigation"
    },
    "stateManagement": {
      "type": "controlled | uncontrolled",
      "controlled": "Pass value + onChange",
      "uncontrolled": "Pass defaultValue + use ref",
      "note": "Do NOT mix value and defaultValue"
    },
    "antiPatterns": [
      "❌ Don't pass both value and defaultValue",
      "❌ Don't forget to associate a label (a11y)"
    ]
  },
  
  "Dropdown": {
    "path": "src/components/Dropdown/Dropdown.tsx",
    "composition": {
      "contextRequired": "DropdownProvider",
      "structure": [
        "DropdownProvider (wrapper)",
        "  DropdownTrigger (button that opens menu)",
        "  DropdownMenu (the menu container)",
        "    DropdownItem (menu items)"
      ],
      "note": "All Dropdown parts MUST be wrapped in DropdownProvider"
    },
    "a11y": {
      "role": "menu",
      "keyboardNav": "Arrow keys to navigate, Enter to select, Escape to close"
    },
    "antiPatterns": [
      "❌ Don't use DropdownTrigger without DropdownProvider",
      "❌ Don't forget keyboard navigation testing"
    ]
  }
}
```

### Component API Contract Fields

| Field | Purpose | LLM Usage |
|-------|---------|-----------|
| `events` | Available callbacks with signatures | Know which handlers exist, prefer built-in over manual |
| `a11y.requiredAria` | Mandatory accessibility attributes | Never forget aria-label on icon buttons |
| `a11y.keyboardNav` | Expected keyboard behavior | Ensure implementation supports keyboard |
| `composition.contextRequired` | Context providers needed | Avoid runtime "context not found" errors |
| `composition.validChildren` | What can be nested inside | Avoid invalid DOM nesting |
| `stateManagement` | Controlled vs uncontrolled behavior | Don't mix patterns |
| `antiPatterns` | Common mistakes to avoid | Explicit "don't do this" guidance |

**Matching Algorithm**:

1. **Parse Figma instance name**: `Button/Primary` → ComponentName: `Button`, Variant: `Primary`

2. **Lookup in registry**:
   ```
   component-registry.json → components["Button"] → figmaMapping["Button/Primary"]
   ```

3. **Extract FULL context**:
   - `path`: Import path
   - `props`: Available props with types and defaults
   - `slots`: Named slots for children, icons, etc.
   - `events`: Callbacks with signatures and preferences
   - `a11y`: Accessibility requirements
   - `composition`: Context and nesting rules
   - `stateManagement`: Controlled/uncontrolled guidance
   - `antiPatterns`: Mistakes to avoid
   - `usage`: Example code snippet

4. **Handle wildcards**: `Icon/*` → `name` prop gets the wildcard value
   - `Icon/Search` → `<Icon name="search">`

5. **Fallback if not found**:
   - Check `aliases` section (e.g., `CTA_Button` → `Button`)
   - If still not found: Mark as `[NOT FOUND - verify or create]`

**Enhanced Output Format**:
```markdown
**Component Instances** (REUSE REQUIRED):

| Figma Instance | Project Component | Props | Slots | Import |
|----------------|-------------------|-------|-------|--------|
| `Button/Primary` | `<Button>` | `variant="primary"` | children, leftIcon, rightIcon | `@/components/Button` |
| `Icon/Search` | `<Icon>` | `name="search"` | - | `@/components/Icon` |
| `ProductCard` | **[NEW]** | - | - | Create in `@/components/ProductCard` |

**Component Usage Guide** (for LLM implementation):

#### Button (`@/components/Button`)
**Props**:
- **Required**: `variant="primary | secondary | ghost"`
- **For icons**: Use `leftIcon` or `rightIcon` props, NOT as children
- **For loading**: Set `isLoading={true}` (shows built-in spinner)

**Events**:
- `onClick`: Standard click handler
- `onSubmit`: **Preferred for async** - auto-manages loading state

**Accessibility**:
- Icon-only buttons MUST have `aria-label`
- Keyboard: Enter/Space triggers click

**❌ Anti-Patterns**:
- Don't wrap Icon in children, use leftIcon/rightIcon prop
- Don't manage loading manually when using onSubmit

**Example**: `<Button variant="primary" leftIcon={<Icon name="search" />}>Search</Button>`

#### Icon (`@/components/Icon`)
- **Required**: `name="icon-name"` (matches Figma icon name, lowercase)
- **Optional**: `size="sm | md | lg"`, `className` for color
- **Example**: `<Icon name="search" size="md" className="text-gray-500" />`

#### Dropdown (`@/components/Dropdown`)
**⚠️ Requires Context Provider**:
```jsx
<DropdownProvider>
  <DropdownTrigger>Open Menu</DropdownTrigger>
  <DropdownMenu>
    <DropdownItem>Option 1</DropdownItem>
  </DropdownMenu>
</DropdownProvider>
```
**Accessibility**: Arrow keys navigate, Enter selects, Escape closes
```

**Anti-Pattern Detection**:

| Pattern | Detection | Correction |
|---------|-----------|------------|
| Icon as child | `<Button><Icon/> Label</Button>` | Use `leftIcon` prop instead |
| Hardcoded color | `style={{ color: '#3B82F6' }}` | Use className with design tokens |
| Custom loading | Custom spinner inside Button | Use `isLoading` prop |
| Missing variant | `<Button>` without variant | Add required `variant` prop |

### Step 4: Populate UI Implementation Guide

Insert into task description using this **structured format**:

```markdown
### UI Implementation Guide

> **Strictness**: Pixel-perfect implementation required. Do not deviate from tokens.

**Figma Reference**: [Figma URL with node-id parameter]

---

**Component Tree** (Semantic Structure):
```
RootContainer (Frame)
├── HeaderSection (Frame, auto-layout row)
│   ├── Icon (Instance: IconComponent)
│   └── Title (Text)
├── ContentArea (Frame, auto-layout column)
│   └── [Children...]
└── ActionBar (Frame, auto-layout row)
    └── SubmitButton (Instance: Button/Primary)
```

**Layout Properties**:
| Container | Direction | Gap | Padding | Align |
|-----------|-----------|-----|---------|-------|
| Root | column | 16px → gap-4 | 24px → p-6 | stretch |
| HeaderSection | row | 8px → gap-2 | 0 | center |
| ActionBar | row | 12px → gap-3 | 16px x, 12px y | end |

**Token Mapping** (Figma → Project):
| Category | Figma Value | Project Token |
|----------|-------------|---------------|
| Background | #FFFFFF | bg-white |
| Border | 1px #E5E7EB | border border-gray-200 |
| Shadow | 0 1px 3px rgba(0,0,0,0.1) | shadow-sm |
| Text Primary | #111827 / 16px / 600 | text-gray-900 text-base font-semibold |

**Component Instances** (REUSE REQUIRED):
- [ ] `Button/Primary` → Use `<Button variant="primary">` from `src/components/Button`
- [ ] `Icon/Search` → Use `<Icon name="search">` from `src/components/Icon`

**Interactive States** (if applicable):
| State | Background | Border | Text |
|-------|------------|--------|------|
| Default | bg-primary-600 | - | text-white |
| Hover | bg-primary-700 | - | text-white |
| Disabled | bg-gray-300 | - | text-gray-500 |
```

---

## Validation

Before creating Frontend Jira task, verify ALL sections are populated:

**Structure & Layout**:
- [ ] Component Tree section shows semantic hierarchy (not just "Container")
- [ ] Layout Properties table has all auto-layout containers
- [ ] Directions, gaps, and padding are mapped to project tokens

**Token Mapping**:
- [ ] Token Mapping table complete (NO raw hex values in project column)
- [ ] All spacing values mapped to Tailwind classes or CSS variables
- [ ] Typography mapped with font-size, font-weight, and color

**Component Reuse**:
- [ ] Component Instances section lists ALL Figma component references
- [ ] Each instance mapped to existing project component with import path
- [ ] If no match exists, note: "NEW - Create component following [pattern]"

**States & Variants**:
- [ ] Interactive States table included (if component has states)
- [ ] All variant properties documented

---

## Failure Handling

### 1. MCP Tool Failure

**If `mcp_figma-dev-mode-mcp-server_get_design_context` fails** (timeout, auth, unreachable):

1. Add placeholder sections:
   ```markdown
   **Component Tree**: [TBD - Design] - Figma extraction failed
   **Layout Properties**: [TBD - Design]
   **Token Mapping**: [TBD - Design]
   ```
2. Mark task with label: `needs-design-review`
3. Inform user: "Figma extraction failed for Task X. Provide screenshot or manual design specs."
4. **Share with designer**: Reference [`figma-design-guidelines.md`](./figma-design-guidelines.md) to prevent future issues

---

### 2. Incomplete Figma Data (Graceful Degradation)

> **Developer Note**: Figma designs vary in quality. Some frames may lack proper structure. The following rules ensure you can still implement while flagging issues.

#### **No Auto-Layout Detected**

| Symptom | Handling |
|---------|----------|
| Frame uses absolute positioning | Describe layout as: "Absolute positioning - structure from layer order" |
| No gap/padding values | Use visual estimation with flag: `[ESTIMATED]` |

```markdown
**Layout Properties**:
| Container | Direction | Gap | Padding | Align |
|-----------|-----------|-----|---------|-------|
| Root | absolute | N/A | [ESTIMATED] 16px | N/A |

> ⚠️ **FIGMA ISSUE**: Frame lacks auto-layout. Layout values are estimated from visual inspection.
```

**Developer Action**: Confirm spacing with designer before finalizing.

---

#### **No Component Instances Found**

| Symptom | Handling |
|---------|----------|
| All elements are raw shapes/text | Mark as "all custom" |
| Buttons/icons are not component instances | Check project library for matches |

```markdown
**Component Instances**:
> ⚠️ **FIGMA ISSUE**: No component instances detected. All elements appear custom.
- [ ] Verify with designer if existing components should be used
- [ ] If truly custom, follow pattern from `[reference component path]`
```

**Developer Action**: Before building custom components, cross-check with `file-categorization.json` for existing matches.

---

#### **Missing or Incomplete Text Styles**

| Symptom | Handling |
|---------|----------|
| Text has no linked style | Extract raw values, map to closest project token |
| Font not in project | Flag and use project default |

```markdown
**Token Mapping** (Figma → Project):
| Category | Figma Value | Project Token |
|----------|-------------|---------------|
| Text | Roboto / 14px / 400 | text-sm font-normal [FONT MISSING - using Inter] |

> ⚠️ **FIGMA ISSUE**: Font "Roboto" not in project. Using project default "Inter".
```

**Developer Action**: Confirm font substitution is acceptable.

---

#### **Empty or Shallow Frame**

| Symptom | Handling |
|---------|----------|
| Selected frame has no children | Request different node ID |
| Only 1-2 elements | Warn that extraction is incomplete |

```markdown
> ⚠️ **FIGMA ISSUE**: Selected frame contains insufficient structure.

**Component Tree**: 
```
Root (Frame) - SHALLOW
└── SingleElement (Text)
```

**Developer Action**: Request complete frame node-id from designer, or provide screenshot.
```

---

### 3. Token Mapping Failures

**If Figma value doesn't exist in project design system**:

1. Use closest match with inline comment:
   ```css
   /* Closest: bg-gray-100, Figma: #F3F4F6 */
   ```
2. Document in Token Mapping table:
   ```markdown
   | Background | #F3F4F6 | bg-gray-100 ⚠️ (closest match) |
   ```
3. If pattern repeats (3+ occurrences): Flag for design system update

---

### 4. Developer Transparency Summary

Every task with Figma issues MUST include this summary at the top of the UI Implementation Guide:

```markdown
### ⚠️ Figma Data Quality Issues

| Issue | Impact | Action Required |
|-------|--------|-----------------|
| No auto-layout | Spacing estimated | Confirm with designer |
| No component instances | All custom build | Check project library first |
| Missing font | Substituted | Verify substitution OK |
| Token not in system | Closest match used | Consider design system update |

> Review flagged items before implementation. Contact designer if blocking.
```

**Labels to Apply**:
- `needs-design-review` – Critical issues, blocks implementation
- `figma-incomplete` – Minor issues, can proceed with caution
