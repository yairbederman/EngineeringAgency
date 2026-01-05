---
template: task-frontend
version: 1.0.0
contract: _template-contracts.md#task-frontend-template-contract
used_by: TaskPlanning
required_sections:
  - header
  - metadata
  - target_files
  - dependencies
  - pattern_context
  - ui_guide
  - component_instances
  - steps
  - test_plan
  - acceptance
  - summary
conditional_sections:
  - figma_reference: "If Figma link in Epic"
  - visual_reference: "If Figma extracted"
  - api_integration: "If calling APIs"
  - interactive_states: "If interactive component"
  - figma_issues: "If Figma extraction had issues"
---

## Task: [Name]

**Layer**: [Frontend | State]  
**Category**: [e.g., `react-components`, `hooks`, `redux-slices`, `pages`]  
**Type**: Frontend  
**Source**: Tech Spec § [Section Number]

---

### Target Files

**Action: NEW**
- `[absolute/path/to/new/Component.tsx]`

**Action: MODIFY**
- `[absolute/path/to/existing/Component.tsx]`

---

### Dependencies

**Depends On**: [List task numbers/names that must complete first, or "None"]

**Examples**:
- Task 3: Create UserService API endpoint (API layer)
- Task 4: Create UserState redux slice (State layer)

OR

- None (standalone component)

---

### Pattern Context (From Tech Spec § 2)

**Pattern to Follow**: [Architecture pattern from Tech Spec]  
**Example**: "Following ${PROJECT_FRONTEND} Redux state management pattern (§ 3.2 from .ai-instructions)"

**Reference Component**: [Existing component to mimic]  
**Example**: `src/components/UserProfile/UserProfile.tsx`

**Rationale**: [Why this pattern/component is being reused]

---

### API Integration (if applicable)

**Endpoint to Call** (From Tech Spec § 5.3):
```typescript
// The component will call this endpoint
POST /api/v1/resource
Request: ResourceRequest
Response: ResourceResponse
```

**API Client**: `src/api/resourceApi.ts`

---

### UI Implementation Guide

> **Strictness**: Pixel-perfect implementation required. Do not deviate from tokens.

**Figma Reference**: [Figma URL with node-id parameter]

---

**Visual Reference** (Screenshot):

![Component Name - Desktop](file:///path/to/screenshot.png)

> **Critical Visual Notes**:
> - [Note any pinned elements, scroll areas, visual rhythm]
> - [Note any non-obvious layout details visible in screenshot]

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
| ActionBar | row | 12px → gap-3 | 16px h, 12px v | end |

**Token Mapping** (Figma → Project):
| Category | Figma Value | Variable/Style | Project Token |
|----------|-------------|----------------|---------------|
| Background | #FFFFFF | color/neutral/50 | bg-white / bg-surface-card |
| Border | 1px #E5E7EB | color/neutral/200 | border border-gray-200 |
| Shadow | 0 1px 3px rgba(0,0,0,0.1) | shadow/sm | shadow-sm |
| Text Primary | #111827 / 16px / 600 | - | text-gray-900 text-base font-semibold |
| Spacing Gap | 16px | spacing/md | gap-4 |

---

**Spatial Constraints** (Scroll, Pinned, z-index):

| Element | Constraint | Behavior | CSS Implementation |
|---------|------------|----------|-------------------|
| HeaderSection | Sticky | Sticks to top during scroll | `sticky top-0 z-10` |
| ContentArea | Scroll | Scrolls when content overflows | `flex-1 overflow-auto` |
| ActionBar | Pinned | Always visible at container bottom | `mt-auto` |

> **Scroll Behavior**: ContentArea scrolls independently; Header and ActionBar remain visible.

---

**Content Constraints** (Overflow, Empty States, Lists):

| Element | Content Type | Max | Overflow Behavior | CSS |
|---------|--------------|-----|-------------------|-----|
| User Name | Text | 30 chars | Truncate | `truncate max-w-[200px]` |
| Description | Text | 150 chars | Clamp 2 lines | `line-clamp-2` |
| Item List | Array | ∞ | Virtualize after 50 | `react-virtual` |

**Required States**:
- [ ] **Empty**: Show `<EmptyState message="No items found" />` when 0 items
- [ ] **Loading**: Show skeleton/spinner while fetching
- [ ] **Error**: Show error message on API failure

---

### Semantic Intent Map

> **Purpose**: What each interactive element DOES, not just how it looks. Cross-referenced with Product Spec.

| Element | Visual Role | Intent Type | Behavior | Spec Ref | Implementation |
|---------|-------------|-------------|----------|----------|----------------|
| [Button name] | [Position/purpose] | [Navigation/Mutation/Fetch/etc.] | [What happens on click] | § [X.X] | [Pattern] |
| [Icon/Toggle] | [Location] | [State Toggle/Mutation] | [Detailed behavior] | § [X.X] | [Pattern] |

**Keyboard Equivalents**:
| Visual Action | Keyboard | Implementation |
|---------------|----------|----------------|
| Click form submit | Enter key | Form `onSubmit` handler |
| Click close button | Escape key | `useEffect` keyboard listener |

**Missing Behavioral Context** (if any):
> ⚠️ These elements need spec clarification:
- [ ] [Element]: `[TBD - behavior not specified]`

---

**Component Instances** (Enhanced - REUSE REQUIRED):

| Figma Instance | Project Component | Props | Slots | Import |
|----------------|-------------------|-------|-------|--------|
| `Icon/Search` | `<Icon>` | `name="search"` | - | `@/components/Icon` |
| `Button/Primary` | `<Button>` | `variant="primary"` | children, leftIcon, rightIcon | `@/components/Button` |

**Component Usage Guide** (with behavioral context):

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
- Don't wrap `<Icon>` in children - use `leftIcon`/`rightIcon` props
- Don't manage loading manually when using `onSubmit`

**Example**: `<Button variant="primary" leftIcon={<Icon name="search" />}>Search</Button>`

#### Icon (`@/components/Icon`)
- **Required**: `name="icon-name"` (matches Figma icon name, lowercase)
- **Optional**: `size="sm | md | lg"`, `className` for color
- **Example**: `<Icon name="search" size="md" className="text-gray-500" />`

#### [Compound Components] (e.g., Dropdown, Modal)

> If a component requires a context provider, document it here:

```jsx
<DropdownProvider>
  <DropdownTrigger>Open Menu</DropdownTrigger>
  <DropdownMenu>
    <DropdownItem>Option 1</DropdownItem>
  </DropdownMenu>
</DropdownProvider>
```

**⚠️ Context Required**: All parts must be wrapped in provider.

---

**Interactive States & Transitions**:

| State | Background | Border | Shadow | Transform | Transition |
|-------|------------|--------|--------|-----------|------------|
| Default | bg-primary-600 | - | shadow-sm | - | - |
| Hover | bg-primary-700 | - | shadow-md | scale(1.02) | 150ms ease-out |
| Pressed | bg-primary-800 | - | shadow-sm | scale(0.98) | 50ms ease-in |
| Focused | bg-primary-600 | ring-2 ring-primary-400 | shadow-sm | - | 100ms |
| Disabled | bg-gray-300 | - | - | - | - |

**Transition CSS**:
```css
.component {
  @apply transition-all duration-150 ease-out;
}
.component:hover {
  @apply bg-primary-700 shadow-md scale-102;
}
.component:active {
  @apply scale-98 duration-50;
}
.component:focus-visible {
  @apply ring-2 ring-primary-400 ring-offset-2;
}
```

---

### ⚠️ Figma Data Quality Issues (if applicable)

> This section appears ONLY when Figma extraction encountered issues.

| Issue | Impact | Developer Action |
|-------|--------|------------------|
| No auto-layout | Spacing values are `[ESTIMATED]` | Confirm with designer before finalizing |
| No component instances | All elements appear custom | Cross-check `file-categorization.json` for matches |
| Missing font | Substituted with project default | Verify substitution is acceptable |
| Token not in system | Using closest match (marked ⚠️) | Consider design system update |
| Shallow frame | Incomplete structure extracted | Request full frame node-id or screenshot |

**Labels Applied**:
- `needs-design-review` – Critical issues that block implementation
- `figma-incomplete` – Minor issues, proceed with caution

> **Developer Responsibility**: If ANY issue above is marked, you MUST:
> 1. Review the flagged sections before coding
> 2. Contact designer if spacing/styling is ambiguous
> 3. Document any deviations in your commit message
> 4. Share [`figma-design-guidelines.md`](../design/figma-design-guidelines.md) with designer to prevent future issues

---

### Implementation Steps

#### Frontend Component:
1. Review reference component (Tech Spec § 2 Pattern Context)
2. Implement UI structure following Figma tokens (UI Guide above)
3. Integrate API client calls (if applicable)
4. Add form validation (if form component)
5. Write component tests (from Test Plan below)

#### State Layer:
1. Review existing state management pattern
2. Create state slice/reducer following pattern
3. Define actions and selectors
4. Integrate with relevant components
5. Write state management unit tests

---

### Test Plan (From Tech Spec § 5.5)

**Test File**: `[path/to/Component.test.tsx]`

**Mock Dependencies**:
- Mock `ApiClient` → returns `mockResponse`
- Mock `useRouter` → returns `mockRouter`

**Test Cases**:

- [ ] **Renders correctly**: Component mounts without errors
  - **Maps to Epic**: Basic functionality

- [ ] **Happy Path**: [User flow description]
  - **Given**: [Precondition]
  - **When**: [User action]
  - **Then**: [Expected UI state]
  - **Maps to Epic**: Acceptance Criterion [#X]

- [ ] **Loading State**: Shows loading indicator
  - **Given**: API call in progress
  - **When**: Component renders
  - **Then**: Loading spinner visible

- [ ] **Error State**: Handles API errors gracefully
  - **Given**: API returns error
  - **When**: Component renders
  - **Then**: Error message displayed
  - **Maps to Epic**: Error Flow [#Y]

- [ ] **Interactive States**: Hover/focus work correctly
  - **Given**: Component rendered
  - **When**: User hovers/focuses
  - **Then**: Correct visual state applied

---

### Acceptance Criteria (From Epic)

**Scenario**: [Name]
```gherkin
Given [precondition]
When [action]
Then [expected result]
```

---

### Change Summary

**What**: [Brief description of what changes]  
**Why**: [Purpose from Epic/Tech Spec]  
**How**: [Technical approach - component structure, state management]
