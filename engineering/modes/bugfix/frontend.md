# Engineering Agent – BugFix Frontend Track

> **When to use**: Bug has UI/visual symptoms, CSS errors, component render issues, or affects `.tsx`/`.vue`/`.css` files.

## Context Loading (Frontend-Specific)

**Load in PARALLEL**:
- `${COPILOT_INSTRUCTIONS_PATH}` → Frontend section, design tokens
- `${DESIGN_TOKENS_PATH}` (if exists)
- Target component files
- Related component dependencies
- Existing test files for the component

**SKIP** (Token Optimization):
- `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json`
- `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json`
- Backend service/controller files
- Database migrations

---

## Step 1F: Frontend Layer Analysis (BugReport Phase)

### Visual Evidence Collection

1. **Screenshot Analysis**:
   - Current (buggy) state
   - Expected state (from Figma or previous working version)
   - Identify visual delta

2. **Component Tree Inspection**:
   - Which component(s) are affected?
   - Is the bug in the component itself or its parent?
   - Are props being passed correctly?

3. **Style Debugging**:
   - Check computed styles in evidence
   - Identify conflicting CSS rules
   - Check for missing/incorrect design tokens

### Codebase Inspection (Frontend-Specific)

Use direct file inspection:
1. `view_file` on component file to understand implementation and props/state
2. `grep_search` for design tokens/styles applied to affected element
3. Search test files for similar bugs or existing coverage

### Layer Mapping

Map bug to specific frontend layer using `${FILE_CATEGORIZATION_PATH}`:
- `react-components` / `vue-components`
- `hooks` / `composables`
- `redux` / `pinia` / `context`
- `api-client` / `fetchers`

---

## Step 2F: Frontend Code Inspection (BugFix Phase)

### Component Inspection

Use direct inspection:
1. `view_file` on component file(s)
2. `view_file` on related test files (`.test.tsx`, `.spec.tsx`)

### State Flow Analysis

- Is state being updated correctly?
- Are there race conditions in async updates?
- Is the component re-rendering when it shouldn't (or vice versa)?

### Style Inspection

- Check token usage matches design system
- Identify any hardcoded values that should be tokens
- Check responsive/RTL considerations

### Race Condition Analysis (If async/timing suspected)

**Symptoms suggesting race condition**:
- Bug appears intermittently or "sometimes works"
- Bug depends on network speed or component load order
- Bug involves async data fetching or state updates

**Inspection checklist**:
- [ ] Are there multiple `useEffect` hooks with shared dependencies?
- [ ] Is state being set after component unmount? (memory leak)
- [ ] Are async operations properly cancelled on unmount?
- [ ] Is there a missing loading state causing premature render?
- [ ] Are there competing `setState` calls from parallel fetches?

**Common patterns to check**:
```typescript
// BAD: Race condition on unmount
useEffect(() => {
  fetchData().then(setData);
}, []);

// GOOD: Abort controller pattern
useEffect(() => {
  const controller = new AbortController();
  fetchData({ signal: controller.signal }).then(setData);
  return () => controller.abort();
}, []);
```

---

## Step 3F: Visual Verification (BugFix Phase)

> **Purpose**: Verify fix resolves visual issue without introducing regressions.

### Step 3F.1: Capture Pre-Fix State (If not already captured)

IF visual evidence not already captured in BugReport:
```
browser_subagent:
  Task: "Navigate to [component URL], capture screenshot of buggy state"
  RecordingName: "[bugkey]_pre_fix"
```

Save to: `[BugKey]_pre_fix.png`

### Step 3F.2: Apply Fix and Capture Post-Fix State

After implementing fix:
```
browser_subagent:
  Task: "Navigate to [component URL], capture screenshot of fixed state"
  RecordingName: "[bugkey]_post_fix"
```

Save to: `[BugKey]_post_fix.png`

### Step 3F.3: Side-by-Side Comparison

Generate comparison:
```markdown
### Visual Verification

````carousel
![Pre-Fix (Bug State)](/path/to/[BugKey]_pre_fix.png)
<!-- slide -->
![Post-Fix (Fixed State)](/path/to/[BugKey]_post_fix.png)
<!-- slide -->
![Expected (Figma/Reference)](/path/to/[BugKey]_expected.png)
````
```

### Step 3F.4: Visual Regression Checklist

After applying fix, verify:
- [ ] Original bug is resolved
- [ ] No new visual regressions introduced
- [ ] Component states all render correctly:
  - [ ] Default state
  - [ ] Hover state
  - [ ] Focused state
  - [ ] Disabled state (if applicable)
  - [ ] Loading state (if applicable)
  - [ ] Error state (if applicable)
- [ ] Responsive behavior maintained (if applicable)
- [ ] RTL layout correct (if applicable)

### Step 3F.5: Deviation Handling

IF fix introduces intentional visual changes:
```markdown
**Visual Deviation Record**

| Aspect | Before Fix | After Fix | Reason |
|--------|------------|-----------|--------|
| [aspect] | [original] | [new] | [justification] |
```

---

## Frontend Test Categories

### Component Tests (MANDATORY)

```typescript
describe('<ComponentName />', () => {
  describe('bug fix: [BugKey]', () => {
    it('should [expected behavior after fix]', () => {});
    it('should not [buggy behavior]', () => {});
  });
  
  describe('regression prevention', () => {
    it('maintains [related functionality]', () => {});
  });
});
```

### Interaction Tests (If bug is interaction-related)

- Click handlers fire correctly
- Form inputs work
- Keyboard navigation
- Focus management

### State Tests (If bug is state-related)

- State updates correctly
- Re-renders happen appropriately
- Async state transitions work

---

## Frontend Completion

1. **Run Component Tests**: All tests for affected component(s)
2. **Visual Verification**: Comparison completed, checklist passed
3. **Commit**: `[BugKey] Fix [description]`
4. **Jira Comment**:
   ```
   ### ✅ Frontend Bug Fix Complete
   
   **Branch**: bugfix/[BugKey]-[summary]
   **Track**: Frontend
   **Tests**: [X] component tests passed
   **Visual**: Pre/post comparison verified
   
   **Fix Summary**: [Brief description of what was fixed]
   **Root Cause**: [What caused the bug]
   
   Ready for code review.
   ```

---

## Frontend-Specific Rules

- Do not invent styles not in the design system
- Use existing design tokens, not hardcoded values
- Preserve existing component behavior unless explicitly fixing it
- Test all component states, not just the buggy one
- Always capture visual evidence for UI bugs
