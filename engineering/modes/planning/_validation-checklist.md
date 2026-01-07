# Pre-Creation Validation Checklist

> Run BEFORE creating Jira tasks. Every task must pass ALL checks.

---

## 1. File Paths (BLOCKER if incomplete)

- [ ] ALL file paths are ABSOLUTE from project root (no `...` placeholders)
- [ ] Example: `my-project/src/ui/sites/base/components/SearchWidget/SearchWidget.tsx`
- [ ] NOT acceptable: `my-project/src/.../SearchWidget.tsx`
- [ ] Verify paths against project structure from `copilot-instructions.md`

---

## 2. Category Field (REQUIRED)

- [ ] Category field populated from `file-categorization.json`
- [ ] Examples: `controllers`, `services`, `react-components`, `migrations`

---

## 3. Context Injection (BLOCKER if missing)

### For Backend Tasks

- [ ] **Full TypeScript API Contract** with complete Request/Response interfaces
- [ ] **Service Method Signature** if applicable
- [ ] **Entity Schema** if touching database (from Tech Spec § 5.2)
- [ ] **Validation Rules** section (e.g., "field: Required, max 255 chars")
- [ ] Pattern Context with **Reference File** path and **Rationale**

### For Frontend Tasks

- [ ] **MANDATORY**: If Epic contains Figma link, extract tokens using Figma MCP
- [ ] **UI Implementation Guide** section populated with:
  - Structure description (e.g., "Flex-col layout")
  - Key Tokens: Background, Spacing, Typography, Colors, Borders/Shadows
  - Component Reuse list
- [ ] **API Endpoint to Call** section if component makes API calls
- [ ] Pattern Context with **Reference File** and **Rationale**

---

## 4. Test Plan (REQUIRED)

- [ ] **Mock Dependencies** specified (e.g., "Mock UserRepository → returns mockUser")
- [ ] At least 2 test cases: Happy Path + Edge/Error Case
- [ ] Each test case **Maps to Epic** scenario number
- [ ] Test file path specified

---

## 5. Dependencies Section (REQUIRED)

- [ ] Format: "Depends On: Task X (PROJECT-XXX): [Description] ([Layer])"
- [ ] OR "Depends On: None (first layer)" if no dependencies

---

## 6. Pattern Context (REQUIRED)

- [ ] **Pattern to Follow** specified with `copilot-instructions.md` section reference
- [ ] **Reference File** specified (absolute path) if reusing pattern
- [ ] **Rationale** explains why pattern is chosen

---

## 7. Acceptance Criteria (REQUIRED)

- [ ] Copy relevant Gherkin scenario from Epic that this task addresses
