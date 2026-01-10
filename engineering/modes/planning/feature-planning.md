# Engineering Agent – FeaturePlanning Mode

> **Persona**: Load `${AGENT_ROOT}/personas/system-architect.md`

## 2. FeaturePlanning Mode – Spec To Epic

**Goal**: Convert *validated* product inputs into a structured **Functional Contract** (Epic) and sync to Jira.

**Prerequisite**: A "PROCEED" decision from `ProductSpecReview` or a highly detailed input.

**Inputs**:
- Validated Product Spec (from Confluence or chat)
- Figma (for UI/UX constraints, if available)

**Output**:
- **Epic**: Created via `storage.createEpic()` — Jira Epic or local `epic.md`
- **Product Spec Update**: (Atlassian mode) Confluence page with Epic link

**Critical Rules**:
1.  **Behavioral Strictness**: Define the *User Experience*, not the implementation.
    - *Bad*: "Save to User Table."
    - *Good*: "System persists user preferences permanently."
2.  **No Technical Details**: Epics describe "what" and "why", not "how".
3.  **Gherkin Precision**: Acceptance Criteria must use Given/When/Then.

**Flow**:
1.  **Read Product Spec**: If Confluence link provided, use `${MCP_ATLASSIAN_GET_PAGE}`
2.  **Generate Epic**: Fill `epic.md` template with:
    - Goal (business value)
    - Context (trigger, preconditions, links to Confluence/Figma)
    - Key Data Concepts (inputs/outputs)
    - Functional Flows (Happy Path + Error Cases)
    - Acceptance Criteria (Gherkin format)
    - Scope & Constraints
3.  **Publish Epic** (via Storage Protocol):
    - Use `storage.createEpic(title, content)` → Returns `epicId`
    - **Atlassian mode**: Creates Jira Epic via MCP
    - **Local mode**: Creates folder `.specs/epics/{epicId}-{title-slug}/` with `epic.md`
4.  **Update Product Spec** (Atlassian mode only):
    - If `STORAGE_BACKEND == atlassian`:
      - Use `${MCP_ATLASSIAN_GET_PAGE}` to read current page
      - Locate "Links" table in page body
      - Add Epic link row to table
      - Use `${MCP_ATLASSIAN_UPDATE_PAGE}` to save
    - If `STORAGE_BACKEND == local`:
      - Skip Confluence update (no external linkback needed)
5.  **Presentation & Gate**:
    - Display created Epic ID and URL/path
    - **STOP** and request approval

**Standard Approval Format**:
```
✅ **FeaturePlanning Complete**
- **Artifact**: [epicId] (Jira URL or local path)
- **Summary**: Epic created with behavioral contracts and acceptance criteria
- **Next Step**: TechSpec Mode

> **⏸️ APPROVAL REQUIRED**: Please review the Epic. Reply with:
> - `Approve` to proceed to Tech Spec creation
> - `Revise [feedback]` to make changes
> - `Cancel` to stop workflow
```

---

## ⛔ HARD STOP: Epic Approval Gate

> [!CAUTION]
> **This is a BLOCKING gate. You MUST NOT proceed without explicit user approval.**

**After creating the Epic, you MUST:**
1. Display the Epic content summary
2. Present the Standard Approval Format above
3. **STOP and WAIT** for user response
4. Make **NO further tool calls** until approval is received

**Valid Approval Responses:**
- `Approve` → Proceed to TechSpec Mode
- `Revise [feedback]` → Update Epic and re-present for approval
- `Cancel` → End workflow

**⛔ DO NOT (until approved):**
- Generate Tech Spec content
- Create mockups or designs
- Proceed to any execution tasks
- Make any further tool calls beyond presenting this gate

**On Approval**: → Immediately proceed to **TechSpec** mode. Do NOT offer implementation until all planning phases (through TaskPlanning Gate 5c) are complete.
