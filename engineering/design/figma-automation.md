# Figma Token Extraction Automation

> **Purpose**: Ensure all Frontend tasks have pixel-perfect implementation guides by extracting design tokens from Figma before task creation.

---

## When to Trigger

**MANDATORY** for any task in TaskPlanning mode where:
- Task Layer = "Frontend" OR "Frontend Integration"
- Epic contains Figma link(s) in description or links section

---

## MCP Response Schema

> **Purpose**: Document the expected response structure from Figma MCP tools to ensure accurate parsing.

### `mcp1_get_design_context` Response

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
    └── Call mcp1_get_metadata to list all frames on the page
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
   Frame names from mcp1_get_metadata:
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

For each Frontend task, call `mcp1_get_design_context` with node ID and extract ALL of the following:

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

After extracting design context, call `mcp1_get_variable_defs` with the same node ID:

```
mcp1_get_variable_defs(nodeId: "[node-id]")
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
| `mcp1_get_variable_defs` fails | Proceed with Style Names and algorithmic matching |
| No variables defined in Figma | Note: "Figma Variables not configured" |
| Partial variables | Use available, fallback for rest |

```markdown
> ℹ️ **Figma Variables**: Not configured for this file. Using style names and algorithmic matching.
```

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

### Step 3.5: Match Component Instances

> **MANDATORY**: Use project's `component-registry.json` for deterministic matching.

**Read component registry**: `.ai-instructions/analysis/component-registry.json`

**Matching Algorithm**:

1. **Parse Figma instance name**: `Button/Primary` → ComponentName: `Button`, Variant: `Primary`

2. **Lookup in registry**:
   ```
   component-registry.json → components["Button"] → figmaMapping["Button/Primary"]
   ```

3. **Extract result**:
   - `importPath`: Where to import from
   - `props`: What props to pass (e.g., `{ "variant": "primary" }`)

4. **Handle wildcards**: `Icon/*` → `name` prop gets the wildcard value
   - `Icon/Search` → `<Icon name="search">`

5. **Fallback if not found**:
   - Check `aliases` section (e.g., `CTA_Button` → `Button`)
   - If still not found: Mark as `[NOT FOUND - verify or create]`

**Output Format**:
```markdown
**Component Instances** (REUSE REQUIRED):
- [x] `Button/Primary` → `<Button variant="primary">` from `@/components/Button`
- [x] `Icon/Search` → `<Icon name="search">` from `@/components/Icon`
- [ ] `ProductCard` → **[NOT FOUND]** - Verify in component-registry.json or create new
```

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

**If `mcp1_get_design_context` fails** (timeout, auth, unreachable):

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
