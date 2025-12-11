# Phase 3.5: Extract Design Tokens (Frontend Only)

> **Purpose**: Extract project-specific design tokens for Figma-to-code mapping.

---

## Trigger Condition

**Only execute for Frontend projects** where one of these exists:
- `tailwind.config.js` or `tailwind.config.ts`
- `theme.ts` or `theme.js`
- `globals.css` or `variables.scss` with CSS custom properties

**Skip this phase** for Backend APIs (JVM, Python, Go, etc.)

---

## Output

**File**: `analysis/design-tokens.json`

---

## Extraction Process

### Step 1: Detect Design Token Source

Check these files in order:

| Priority | File Pattern | Token Type |
|----------|--------------|------------|
| 1 | `tailwind.config.{js,ts}` | Tailwind theme |
| 2 | `theme.{ts,js}` | Custom theme object |
| 3 | `*.{css,scss}` with `:root` | CSS variables |
| 4 | `styled-components` theme | Styled-components |

### Step 2: Extract Tailwind Tokens (if applicable)

Read `tailwind.config.js` → `theme.extend` section:

```javascript
// Extract from
module.exports = {
  theme: {
    extend: {
      colors: { ... },
      spacing: { ... },
      fontSize: { ... },
      // etc.
    }
  }
}
```

**Merge with defaults**: If `theme.extend` exists, merge with Tailwind defaults.

### Step 3: Extract CSS Variables (if applicable)

Find `:root` declarations:

```css
:root {
  --color-primary: #3B82F6;
  --spacing-md: 16px;
}
```

**Parse pattern**: `--{category}-{name}: {value}`

### Step 4: Generate design-tokens.json

**Schema**:

```json
{
  "generatedAt": "ISO timestamp",
  "source": "tailwind.config.js | theme.ts | globals.css",
  "framework": "tailwind | css-variables | styled-components",
  "colors": {
    "primary-500": {
      "hex": "#3B82F6",
      "rgb": [59, 130, 246],
      "class": "bg-primary-500",
      "variable": "var(--color-primary-500)"
    },
    "gray-100": {
      "hex": "#F3F4F6",
      "rgb": [243, 244, 246],
      "class": "bg-gray-100",
      "variable": null
    }
  },
  "spacing": {
    "0": { "px": 0, "class": "p-0" },
    "1": { "px": 4, "class": "p-1" },
    "2": { "px": 8, "class": "p-2" },
    "4": { "px": 16, "class": "p-4" },
    "6": { "px": 24, "class": "p-6" },
    "8": { "px": 32, "class": "p-8" }
  },
  "fontSize": {
    "xs": { "px": 12, "class": "text-xs" },
    "sm": { "px": 14, "class": "text-sm" },
    "base": { "px": 16, "class": "text-base" },
    "lg": { "px": 18, "class": "text-lg" },
    "xl": { "px": 20, "class": "text-xl" },
    "2xl": { "px": 24, "class": "text-2xl" }
  },
  "fontWeight": {
    "normal": { "value": 400, "class": "font-normal" },
    "medium": { "value": 500, "class": "font-medium" },
    "semibold": { "value": 600, "class": "font-semibold" },
    "bold": { "value": 700, "class": "font-bold" }
  },
  "borderRadius": {
    "none": { "px": 0, "class": "rounded-none" },
    "sm": { "px": 2, "class": "rounded-sm" },
    "DEFAULT": { "px": 4, "class": "rounded" },
    "md": { "px": 6, "class": "rounded-md" },
    "lg": { "px": 8, "class": "rounded-lg" },
    "full": { "px": 9999, "class": "rounded-full" }
  },
  "boxShadow": {
    "sm": { 
      "value": "0 1px 2px rgba(0,0,0,0.05)", 
      "class": "shadow-sm" 
    },
    "DEFAULT": { 
      "value": "0 1px 3px rgba(0,0,0,0.1)", 
      "class": "shadow" 
    },
    "md": { 
      "value": "0 4px 6px rgba(0,0,0,0.1)", 
      "class": "shadow-md" 
    }
  }
}
```

---

## Custom Token Handling

### Tailwind Extended Colors

If project defines custom colors:

```javascript
// tailwind.config.js
colors: {
  brand: {
    50: '#EFF6FF',
    500: '#3B82F6',
    900: '#1E3A8A'
  }
}
```

Extract as:
```json
{
  "brand-50": { "hex": "#EFF6FF", "class": "bg-brand-50" },
  "brand-500": { "hex": "#3B82F6", "class": "bg-brand-500" }
}
```

### CSS Variable Patterns

Parse common patterns:

| Pattern | Example | Extracted As |
|---------|---------|--------------|
| `--color-{name}` | `--color-primary` | `colors.primary` |
| `--spacing-{name}` | `--spacing-lg` | `spacing.lg` |
| `--font-size-{name}` | `--font-size-lg` | `fontSize.lg` |
| `--radius-{name}` | `--radius-md` | `borderRadius.md` |

---

## Verification

After extraction, verify:

- [ ] At least 5 colors extracted (or flags "minimal palette")
- [ ] Spacing scale has at least 6 values
- [ ] Font sizes have at least 4 values
- [ ] Source file correctly identified

---

## Usage by Engineering Agent

The `/engineering-agent` uses this file in:

1. **TaskPlanning**: `figma-automation.md` Step 3 reads `design-tokens.json` for token mapping
2. **Implementation**: LLM validates generated CSS against project tokens
