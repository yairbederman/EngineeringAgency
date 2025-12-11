# Figma Design Guidelines for AI-Powered Development

> **Purpose**: These guidelines ensure Figma designs can be automatically translated into implementation-ready code with maximum accuracy.

---

## Why This Matters

Our development workflow uses AI to extract design tokens directly from Figma. When designs follow these guidelines, developers get:
- ✅ Exact spacing and layout values
- ✅ Correct component references
- ✅ Proper typography and color tokens
- ✅ Interactive states (hover, disabled, etc.)

When designs don't follow these guidelines:
- ⚠️ Developers must estimate values
- ⚠️ Custom components get rebuilt instead of reused
- ⚠️ Implementation takes longer and requires more review cycles

---

## Required Practices

### 1. Always Use Auto-Layout

| ❌ Don't | ✅ Do |
|----------|-------|
| Position elements with absolute coordinates | Use Auto-Layout on all container frames |
| Manual spacing between elements | Set explicit `gap` values in Auto-Layout |
| Resize by dragging corners | Use `Hug contents` or `Fill container` |

**Why**: Auto-Layout provides explicit `gap`, `padding`, and `direction` values that map directly to CSS flexbox.

**Quick Check**: If you can't resize a frame by changing padding/gap values, it's not using Auto-Layout.

---

### 2. Use Component Instances (Not Detached Copies)

| ❌ Don't | ✅ Do |
|----------|-------|
| Copy-paste a button and detach it | Insert button from the component library |
| Create one-off variations | Use component variants |
| Draw icons as shapes | Use icon component instances |

**Why**: Component instances are detected and mapped to existing code components. Detached elements are treated as custom and rebuilt from scratch.

**Quick Check**: Select an element → Right panel should show "Instance of [ComponentName]"

---

### 3. Use Defined Text Styles

| ❌ Don't | ✅ Do |
|----------|-------|
| Set font size/weight manually per text | Apply a Text Style from the library |
| Use arbitrary font sizes (15px, 17px) | Use scale-based sizes (14px, 16px, 18px, 20px, 24px) |
| Mix font families | Stick to project's defined font(s) |

**Why**: Text styles map to typography tokens (e.g., `text-base font-semibold`). Custom values require manual mapping.

**Quick Check**: Select text → Right panel should show a linked Text Style name

---

### 4. Use Color Styles (Not Raw Hex Values)

| ❌ Don't | ✅ Do |
|----------|-------|
| Pick colors with the eyedropper | Apply a Color Style from the library |
| Use slightly different shades (#F3F3F3 vs #F4F4F4) | Use consistent color tokens |
| Create new colors without adding to library | Add new colors as Color Styles first |

**Why**: Color styles map to CSS variables or Tailwind classes. Raw hex values require manual matching to "closest" color.

**Quick Check**: Select element → Fill/Stroke should show a linked Color Style name, not a hex value

---

### 5. Name Layers Semantically

| ❌ Don't | ✅ Do |
|----------|-------|
| "Frame 1", "Rectangle 5" | "Header", "CardContainer", "ActionBar" |
| "Group 12" | "UserInfoSection" |
| Default auto-names | Descriptive names that indicate purpose |

**Why**: Layer names become hints for code structure. "ProductCard" tells developers more than "Frame 47".

---

### 6. Define Component Variants for States

| ❌ Don't | ✅ Do |
|----------|-------|
| Create separate components for each state | Use Variants on a single component |
| Describe hover state in comments | Create `State=Hover` variant |
| Leave disabled state undefined | Create `State=Disabled` variant |

**Required States for Interactive Elements**:
- **Default** – Normal state
- **Hover** – Mouse over
- **Focused** – Keyboard focus (important for accessibility)
- **Disabled** – Non-interactive state
- **Active/Pressed** – During click (optional)

**Why**: Variant properties are extracted and translated to CSS pseudo-classes and state-based styling.

---

### 7. Set Constraints for Responsive Behavior

| ❌ Don't | ✅ Do |
|----------|-------|
| Leave constraints at default | Set left/right constraints for horizontal stretch |
| Assume developers will figure it out | Use "Scale" for proportional elements |
| Design only for one viewport | Consider how elements should resize |

**Why**: Constraints hint at responsive behavior (fixed width vs. flexible).

---

## Quick Checklist Before Handoff

Before marking a design ready for development, verify:

- [ ] **Auto-Layout**: All containers use Auto-Layout with explicit gap/padding
- [ ] **Components**: All buttons, icons, inputs are component instances (not detached)
- [ ] **Text Styles**: All text uses linked Text Styles
- [ ] **Color Styles**: All fills/strokes use linked Color Styles
- [ ] **Layer Names**: No "Frame 1" or "Group 2" – all named semantically
- [ ] **Variants**: Interactive elements have hover/disabled/focused variants
- [ ] **Constraints**: Key elements have appropriate resize constraints

---

## What Happens If Guidelines Aren't Followed

| Missing Element | Developer Impact |
|-----------------|------------------|
| No Auto-Layout | Spacing marked as `[ESTIMATED]` – requires designer confirmation |
| Detached components | Built from scratch instead of reusing existing code |
| Raw hex colors | Mapped to "closest match" – may not be exact |
| Missing text styles | Typography inconsistent with design system |
| Poor layer names | Code structure harder to understand |
| No state variants | Developers must guess hover/disabled styling |

---

## Questions?

If you're unsure whether a design follows these guidelines, ask development:
- "Can you extract tokens from this frame?"
- "Are my components recognized?"

We can run a quick extraction test before full handoff to catch issues early.
