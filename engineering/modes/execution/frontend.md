# Engineering Agent – Frontend Implementation Track

> **Persona**: Load `${AGENT_ROOT}/personas/frontend-developer.md`

> **When to use**: Task has labels `frontend`, `ui`, `component`, or description mentions UI/component/design changes.

## Frontend-Specific Pre-flight (Phase 0F)

### UI Implementation Guide Validation

1. **Check Task Description** for:
   - UI Implementation Guide section
   - Figma Reference link
   - Token Mapping table
   - Component Instances list

2. **If Missing**:
   ```
   STOP. Return to TaskPlanning mode or generate now:
   "This frontend task requires a UI Implementation Guide. Options:
    A) Run Figma extraction: `${MCP_FIGMA_GET_DESIGN}`
   B) Return to TaskPlanning to add UI context"
   ```

3. **If Present**: Validate completeness:
   - [ ] Figma link is accessible
   - [ ] Token mapping covers all visual properties
   - [ ] Component instances are identified
   - [ ] Interactive states documented

---

## Figma Reference Capture (Step 3.5F)

> **Purpose**: Save design reference BEFORE implementation for accurate visual comparison.

### Step A: Extract Figma Node ID

Parse from UI Implementation Guide's "Figma Reference" field:
- URL format: `https://figma.com/design/abc123/File?node-id=1-234`
- Extract: `nodeId: "1-234"` (replace `-` with `:` if needed)

### Step B: Capture Design Screenshot

```
`${MCP_FIGMA_GET_SCREENSHOT}`(nodeId: "[node-id]")
```

Save to artifacts: `[TaskKey]_figma_reference.png`

### Step C: Extract Design Dimensions

From `${MCP_FIGMA_GET_DESIGN}`:
- Width: `[X]px`
- Height: `[Y]px`
- Sets expected viewport for browser comparison

### Fallback

If `${MCP_FIGMA_GET_SCREENSHOT}` fails:
```markdown
> ⚠️ Figma screenshot unavailable. Visual comparison based on Token Mapping table only.
```

---

## Frontend Context Loading (Step 3F)

Load in PARALLEL:
- `${COPILOT_INSTRUCTIONS_PATH}` → Frontend section, design tokens
- `${DESIGN_TOKENS_PATH}` (if exists)
- Target component files
- Related component dependencies
- Existing Storybook stories or test files
- Figma screenshot (from Step 3.5F)

---

## Frontend TDD Loop (Step 4F)

### Test Categories (Priority Order)

1. **Component Tests** (MANDATORY):
   - Renders without crashing
   - Displays expected content
   - Handles all props correctly
   - Accessibility basics (aria-labels, roles)

2. **Interaction Tests**:
   - Click handlers fire
   - Form inputs work
   - Keyboard navigation
   - Focus management

3. **State Tests**:
   - Loading state renders correctly
   - Error state displays message
   - Empty state shown when appropriate
   - Disabled state prevents interaction

### Test Naming Convention

```typescript
describe('<ComponentName />', () => {
  describe('rendering', () => {
    it('renders with default props', () => {});
    it('renders [variant] variant correctly', () => {});
  });
  
  describe('interactions', () => {
    it('calls onClick when clicked', () => {});
    it('is focusable via keyboard', () => {});
  });
  
  describe('states', () => {
    it('shows loading spinner when isLoading', () => {});
    it('shows error message when error prop set', () => {});
  });
});
```

---

## Frontend Implementation (Step 5F)

### Visual Strictness Rules

> **FORBIDDEN**: Inventing styles not specified in the UI Implementation Guide.

**DO**:
- Use class names/variables from Token Mapping table
- Example: Task says `gap-4` → Use `gap-4`

**DO NOT**:
- Use raw values: `margin-top: 16px` instead of token
- Invent colors not in design system
- Guess spacing values

### Implementation Order

1. **Component Structure** – JSX/template matching Figma tree
2. **Styling** – Apply tokens from mapping table
3. **Props Interface** – Define component API
4. **State Management** – Local state or hook integration
5. **Event Handlers** – Interactions per spec
6. **Accessibility** – aria-labels, roles, keyboard

### Component Reuse Check

Before creating new elements:
1. Check `${FILE_CATEGORIZATION_PATH}` for existing components
2. Match Figma instance names to project components
3. Extend existing if partial match; create new only if necessary

---

## Visual Verification (Step 6F)

### Step 6A: Launch Component in Browser

**Detect Available Environment** (in priority order):

1. **Check for Storybook**:
   - Look for `storybook` in package.json scripts
   - If found: `npm run storybook` → Navigate to component story URL
   
2. **Check for Dev Server**:
   - Look for `dev` or `start` in package.json scripts
   - If found: Start server → Navigate to component route (from task description)
   
3. **Check for Demo/Examples**:
   - Look for `demo/`, `examples/`, or `playground/` folder
   - If found: Serve static files or use existing demo page
   
4. **Fallback**:
   - If none found: STOP and ask user:
     ```
     "Cannot auto-detect component preview environment.
     Options:
     A) Provide URL where component is visible
     B) Provide command to start preview server
     C) Skip visual verification (document as limitation)"
     ```

**Once environment running**:
- Navigate to component URL
- Wait for component to fully render (check for loading indicators)

### Step 6B: Capture Implementation Screenshots

**Screenshot Checklist**:
- [ ] Default state → `[TaskKey]_impl_default.png`
- [ ] Hover state → `[TaskKey]_impl_hover.png`
- [ ] Focused state (if applicable)
- [ ] Disabled state (if applicable)
- [ ] Mobile viewport (if responsive)

### Step 6C: Side-by-Side Comparison

**Compare**:
- Figma: `[TaskKey]_figma_reference.png` (captured in 3.5F)
- Browser: `[TaskKey]_impl_default.png` (captured in 6B)

**Generate Comparison Carousel**:
```markdown
````carousel
![Figma Design](/path/to/[TaskKey]_figma_reference.png)
<!-- slide -->
![Implementation](/path/to/[TaskKey]_impl_default.png)
````
```

**Run Visual Comparison Checklist**:
- Use table format from `./templates/visual-comparison-table.md`

### Step 6D: Deviation Handling

If implementation differs from Figma:
- Follow protocol in `./templates/deviation-handling.md`

### Step 6E: Demo Recording (Optional)

For significant UI implementations:
```
`${MCP_BROWSER_ACTION}`:
  Task: "Navigate to [URL], demonstrate [user flow], capture recording"
  RecordingName: "[taskkey]_user_flow"
```

Recording should show:
- Initial component render
- Interactive behaviors (hover, click, input)
- Loading/success/error states if applicable

---

## Frontend Accessibility Checks

Before completion:
- [ ] All interactive elements are keyboard accessible
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 UI)
- [ ] Focus indicators are visible
- [ ] Form inputs have associated labels
- [ ] Images have alt text (or aria-hidden if decorative)
- [ ] Screen reader tested (if available)

---

## Frontend Completion (Phase 3F)

1. **Run Component Tests**: All tests for this component
2. **Visual Verification**: Comparison table completed
3. **Commit**: Follow format in `./templates/commit-conventions.md`
4. **Status Update**: `updateTaskStatus(taskId, "in-review")`
5. **Completion Comment**: `addTaskComment(taskId, comment)`
   ```
   ### ✅ Frontend Implementation Complete
   
   **Branch**: feature/[TaskKey]-[summary]
   **Tests**: [X] component tests passed
   **Visual**: Comparison vs Figma completed
   
   **Deviations**: [None / List with reasons]
   
   Ready for code review.
   ```
