# Figma Token Extraction Automation

> **Purpose**: Ensure all Frontend tasks have pixel-perfect implementation guides by extracting design tokens from Figma before task creation.

---

## When to Trigger

**MANDATORY** for any task in TaskPlanning mode where:
- Task Layer = "Frontend" OR "Frontend Integration"
- Epic contains Figma link(s) in description or links section

---

## Automation Process

### Step 1: Detect Figma Links in Epic
- Read Epic using `mcp0_getJiraIssue`
- Search description and custom fields for Figma URLs
- Extract node IDs from URLs (format: `node-id=XXXXX-XXXXX`)

### Step 2: Extract Design Tokens
- For each Frontend task, call `mcp1_get_design_context` with node ID
- Extract:
  - Layout structure (Flex-col, Grid, etc.)
  - Color values (Background, Text, Border)
  - Spacing values (Padding, Margin, Gap)
  - Typography (Font family, size, weight)
  - Border/Shadow properties

### Step 3: Map to Project Tokens
- Read project's design system:
  - `tailwind.config.js` (for Tailwind projects)
  - `theme.ts` / `variables.scss` (for custom theme)
- Map Figma raw values to project token names:
  - Figma `#EF4444` → Project `bg-red-500` or `var(--color-danger)`
  - Figma `16px` → Project `p-4` or `spacing-md`

### Step 4: Populate UI Implementation Guide
- Insert into task description's "UI Implementation Guide" section:
  ```markdown
  ### UI Implementation Guide
  
  > **Strictness**: Pixel-perfect implementation required. Do not deviate from tokens.
  
  **Figma Reference**: [Node ID from Epic]
  
  **Structure**: [Extracted layout description]
  
  **Key Tokens**:
  - **Background**: [Mapped token]
  - **Spacing**: [Mapped tokens]
  - **Typography**: [Mapped tokens]
  - **Colors**: [Mapped tokens]
  - **Borders/Shadows**: [Mapped tokens]
  
  **Component Reuse** (from file-categorization.json):
  - [ ] Use [ComponentName] for [purpose]
  ```

---

## Validation

Before creating Frontend Jira task, verify:
- [ ] UI Implementation Guide section exists
- [ ] All Key Tokens are mapped (not raw Figma values)
- [ ] Component Reuse section populated from project components
- [ ] Figma Reference includes node ID

---

## Failure Handling

If `mcp1_get_design_context` fails:
- Add placeholder: `[TBD - Design] - Figma token extraction failed, manual design review required`
- Mark task with label: `needs-design-review`
- Inform user: "Figma extraction failed for Task X. Manual design review required before implementation."
