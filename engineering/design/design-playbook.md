# Design Playbook for Engineering Handoff

> **Purpose**: Guidelines for designers to ensure Figma designs are fully annotated and ready for the `/engineering-agent` to process without gaps.

---

## 1. Annotation Placement

### ✅ Where to Add Annotations

| Target | Example | Why |
|--------|---------|-----|
| **Component Definition** (parent with all variants) | `Primary bt_56_BF` | Applies to all instances |
| **Default Variant** | `Property 1=Default` | Click behavior originates here |
| **Interactive Elements** | Buttons, links, toggles | User actions need specification |

### ❌ Where NOT to Add

| Avoid | Reason |
|-------|--------|
| Page/Frame containers | Too broad, not picked up |
| Individual SVG/icon assets | These are visual only |
| Regular Figma comments | Not visible to MCP tools |

---

## 2. How to Add Dev Mode Annotations

1. Toggle to **Dev Mode** (top right of Figma UI)
2. Select the target component
3. In the right panel, find **"Annotations"** section
4. Click **"Add annotation"**
5. Enter structured annotation text (see format below)

> **Important**: Only Dev Mode annotations appear as `data-annotations` in the extracted code.

---

## 3. Annotation Format for Interactive Elements

### Standard Schema

```yaml
onClick: [actionName]
params: { [key]: [value] }
targets:
  - [condition1]: "[nodeId1]"
  - [condition2]: "[nodeId2]"
stateChange: [fromState] → [toState]
```

### Example: Button Opening Modal

```yaml
onClick: openFlightDetailsModal
params: { flightRowIndex: parentRowIndex }
targets:
  - firstRow: "21354:120266"
  - secondRow: "21354:120611"
stateChange: Default → select
```

### Common Action Names

| Action | Use For |
|--------|---------|
| `openModal` | Triggering modal/dialog |
| `navigate` | Page/route changes |
| `toggle` | On/off states |
| `submit` | Form submissions |
| `expand` / `collapse` | Accordion/dropdown |
| `select` | Selection from list |

---

## 4. Component State Naming

Use consistent state names across all components:

| State | Purpose | Visual Indicator |
|-------|---------|------------------|
| `Default` | Idle/initial state | Normal appearance |
| `hover` | Mouse over | Subtle highlight |
| `select` | Clicked/active | Strong visual feedback |
| `disable` | Non-interactive | Grayed out |
| `focus` | Keyboard focus | Outline ring |
| `loading` | Processing | Spinner/skeleton |
| `error` | Validation failed | Red border/text |

---

## 5. Required Information for Each Interactive Element

| Field | Description | Example |
|-------|-------------|---------|
| **Action** | What happens on click | `openModal` |
| **Target(s)** | Destination node IDs | `"21354:120266"` |
| **Context** | What data determines behavior | `parentRowIndex` |
| **State Change** | Visual feedback | `Default → select` |

---

## 6. Linking Modals and Targets

When a button opens a modal:

1. **In the button annotation**, reference the modal node ID
2. **In the modal frame**, add annotation describing:
   - How to close (X button, click outside, ESC)
   - Data bindings (what populates the content)

### Modal Annotation Example

```yaml
type: modal
trigger: "21353:118182" (button that opens this)
closeOn: ["clickOutside", "escKey", "closeButton"]
dataBinding: flightRowData
```

---

## 7. Pre-Handoff Checklist

Before marking a design ready for engineering:

### Structure
- [ ] All interactive elements have component definitions
- [ ] Components have all required state variants (Default, hover, select, etc.)
- [ ] Component naming follows convention (`ComponentName_size_variant`)

### Annotations
- [ ] onClick annotations on all buttons/interactive elements
- [ ] Target node IDs are specified for navigation/modals
- [ ] State transitions are documented

### Assets
- [ ] Icons exported as SVG
- [ ] Images optimized and named descriptively
- [ ] Design tokens consistent with project theme

### Verification
- [ ] Annotations visible in Dev Mode
- [ ] All target node IDs are valid (not deleted/renamed)
- [ ] Cross-referenced with Product Spec requirements

---

## 8. Node ID Reference

To find a node ID in Figma:

1. Select the element
2. Look at URL: `?node-id=21353-118182`
3. Convert to colon format: `21353:118182`

Use this ID format in all annotations.

---

## 9. MCP Tool Compatibility

The engineering agent uses these Figma MCP tools:

| Tool | Reads | Picks Up Annotations? |
|------|-------|----------------------|
| `get_design_context` | Component code + tokens | ✅ Yes (`data-annotations`) |
| `get_metadata` | Structure overview | ❌ No |
| `get_screenshot` | Visual reference | ❌ No |
| `get_variable_defs` | Design variables | ❌ No |

> **Only Dev Mode annotations** appear in `get_design_context` output.

---

## 10. Common Issues & Fixes

| Issue | Symptom | Fix |
|-------|---------|-----|
| Annotation not visible | No `data-annotations` in output | Use Dev Mode annotation, not comments |
| Wrong component annotated | Behavior not extracted | Annotate the component set, not the page |
| Missing target | Engineer asks "where does this go?" | Add target node IDs |
| Unclear action | Engineer asks "what should happen?" | Use standard action names |

---

## Related Files

- [figma-extraction-protocol.md](figma-extraction-protocol.md) - Token extraction automation
- [figma-design-guidelines.md](figma-design-guidelines.md) - Visual design standards
- [token-mapping-rules.md](token-mapping-rules.md) - Token to code mapping
