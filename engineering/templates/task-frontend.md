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
| Category | Figma Value | Project Token |
|----------|-------------|---------------|
| Background | #FFFFFF | bg-white / bg-surface-card |
| Border | 1px #E5E7EB | border border-gray-200 |
| Shadow | 0 1px 3px rgba(0,0,0,0.1) | shadow-sm |
| Text Primary | #111827 / 16px / 600 | text-gray-900 text-base font-semibold |
| Spacing Gap | 16px | gap-4 |

**Component Instances** (REUSE REQUIRED):
- [ ] `IconComponent` → Use `<Icon name="...">` from `src/components/Icon`
- [ ] `Button/Primary` → Use `<Button variant="primary">` from `src/components/Button`

**Interactive States** (if component has states):
| State | Background | Border | Text |
|-------|------------|--------|------|
| Default | bg-primary-600 | - | text-white |
| Hover | bg-primary-700 | - | text-white |
| Focused | bg-primary-600 | ring-2 ring-primary-400 | text-white |
| Disabled | bg-gray-300 | - | text-gray-500 |

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
