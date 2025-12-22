# Phase 3.6: Extract Component Registry (Frontend Only)

> **Purpose**: Build a mapping registry of reusable UI components for Figma-to-code matching.

---

## Trigger Condition

**Only execute for Frontend projects** where:
- `package.json` contains React/Vue/Angular/Svelte
- Component directory exists (e.g., `src/components/`, `src/ui/`)

**Skip this phase** for Backend APIs.

---

## Output

**File**: `analysis/component-registry.json`

---

## Extraction Process

### Step 1: Discover Component Directories

Scan for common component locations:
```
src/components/
src/ui/
src/shared/
components/
lib/components/
```

### Step 2: Extract Component Metadata

For each component file (`.tsx`, `.jsx`, `.vue`, `.svelte`):

1. **Identify exported component name**
2. **Extract props interface** (TypeScript) or propTypes (JavaScript)
3. **Detect variants** from:
   - Props with union types (e.g., `variant: 'primary' | 'secondary'`)
   - Size/state props (e.g., `size: 'sm' | 'md' | 'lg'`)
4. **Record import path** (relative from project root)

### Step 3: Build Variant Mapping

For props that look like Figma variants:

| Prop Pattern | Figma Naming Convention |
|-------------|------------------------|
| `variant: 'primary'` | `Button/Primary` |
| `size: 'lg'` | `Button/Large` |
| `state: 'disabled'` | `Button/Disabled` |
| `type: 'outline'` | `Button/Outline` |

### Step 4: Generate component-registry.json

**Schema**:

```json
{
  "generatedAt": "ISO timestamp",
  "framework": "react | vue | angular | svelte",
  "totalComponents": 42,
  "components": {
    "Button": {
      "filePath": "src/components/Button/Button.tsx",
      "importPath": "@/components/Button",
      "exportType": "named | default",
      "props": {
        "variant": {
          "type": "union",
          "values": ["primary", "secondary", "outline", "ghost"],
          "default": "primary"
        },
        "size": {
          "type": "union",
          "values": ["sm", "md", "lg"],
          "default": "md"
        },
        "disabled": {
          "type": "boolean",
          "default": false
        }
      },
      "figmaMapping": {
        "Button/Primary": { "variant": "primary" },
        "Button/Secondary": { "variant": "secondary" },
        "Button/Large": { "size": "lg" },
        "Button/Small": { "size": "sm" },
        "Button/Disabled": { "disabled": true }
      }
    },
    "Icon": {
      "filePath": "src/components/Icon/Icon.tsx",
      "importPath": "@/components/Icon",
      "exportType": "named",
      "props": {
        "name": {
          "type": "string",
          "description": "Icon name from icon set"
        },
        "size": {
          "type": "union",
          "values": ["sm", "md", "lg"],
          "default": "md"
        }
      },
      "figmaMapping": {
        "Icon/*": { "name": "$1" }
      }
    },
    "Avatar": {
      "filePath": "src/components/Avatar/Avatar.tsx",
      "importPath": "@/components/Avatar",
      "exportType": "default",
      "props": {
        "size": {
          "type": "union",
          "values": ["xs", "sm", "md", "lg", "xl"]
        },
        "src": {
          "type": "string"
        }
      },
      "figmaMapping": {
        "Avatar/Small": { "size": "sm" },
        "Avatar/Medium": { "size": "md" },
        "Avatar/Large": { "size": "lg" }
      }
    }
  },
  "aliases": {
    "CTA_Button": "Button",
    "ActionButton": "Button",
    "UserAvatar": "Avatar"
  }
}
```

---

## Figma Mapping Rules

### Automatic Mapping Generation

Generate `figmaMapping` entries by:

1. **Variant props** → `ComponentName/VariantValue`
   - `variant: 'primary'` → `Button/Primary`
   
2. **Size props** → `ComponentName/SizeName`
   - `size: 'lg'` → `Button/Large` (map lg→Large, sm→Small, md→Medium)
   
3. **Boolean state props** → `ComponentName/StateName`
   - `disabled: true` → `Button/Disabled`

4. **Wildcard for dynamic props**
   - `name: string` (on Icon) → `Icon/*` where `$1` is the wildcard value

### Size Normalization

| Code Value | Figma Alias |
|------------|-------------|
| `xs` | `XSmall`, `ExtraSmall` |
| `sm` | `Small` |
| `md` | `Medium`, `Default` |
| `lg` | `Large` |
| `xl` | `XLarge`, `ExtraLarge` |

---

## Verification

After extraction, verify:

- [ ] At least 5 components extracted (or flag "minimal component library")
- [ ] Each component has `importPath` specified
- [ ] Variant props have `figmaMapping` generated
- [ ] No duplicate component names

---

## Usage by Engineering Agent

The `/engineering-agent` uses this file in:

1. **TaskPlanning**: `figma-extraction-protocol.md` reads `component-registry.json` to match Figma instances
2. **Implementation**: LLM uses exact import paths and prop names

### Matching Algorithm (in figma-extraction-protocol.md)

```
Input: Figma instance "Button/Primary"

1. Parse: ComponentName = "Button", Variant = "Primary"
2. Lookup: component-registry.json → components["Button"]
3. Match: figmaMapping["Button/Primary"] → { "variant": "primary" }
4. Output: <Button variant="primary"> from @/components/Button
```

---

## Fallback Handling

If component not found in registry:

1. **Check aliases**: `CTA_Button` → `Button`
2. **Fuzzy match**: `ProductCard` might match `Card`
3. **If no match**: Flag as `[NEW COMPONENT]` with note to create or find existing

```markdown
**Component Instances** (REUSE REQUIRED):
- [x] `Button/Primary` → `<Button variant="primary">` from `@/components/Button`
- [ ] `ProductCard` → **[NOT FOUND]** - Verify if exists or create new
```
