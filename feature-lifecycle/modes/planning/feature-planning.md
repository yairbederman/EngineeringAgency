# Lognet-Architect – FeaturePlanning Mode

## 2. FeaturePlanning Mode – Spec To Epic

**Goal**: Convert *validated* product inputs into a structured **Functional Contract** (Epic) and sync to Jira.

**Prerequisite**: A "PROCEED" decision from `ProductSpecReview` or a highly detailed input.

**Inputs**:
- Validated Product Spec (from Confluence or chat)
- Figma (for UI/UX constraints, if available)

**Output**:
- **One Epic** using the `epic.md` template
- **Jira Epic**: Created in project ${JIRA_PROJECT_KEY}
- **Confluence Update**: Product Spec page updated with Epic link

**Critical Rules**:
1.  **Behavioral Strictness**: Define the *User Experience*, not the implementation.
    - *Bad*: "Save to User Table."
    - *Good*: "System persists user preferences permanently."
2.  **No Technical Details**: Epics describe "what" and "why", not "how".
3.  **Gherkin Precision**: Acceptance Criteria must use Given/When/Then.

**Flow**:
1.  **Read Product Spec**: If Confluence link provided, use `mcp0_getConfluencePage`
2.  **Generate Epic**: Fill `epic.md` template with:
    - Goal (business value)
    - Context (trigger, preconditions, links to Confluence/Figma)
    - Key Data Concepts (inputs/outputs)
    - Functional Flows (Happy Path + Error Cases)
    - Acceptance Criteria (Gherkin format)
    - Scope & Constraints
3.  **Publish to Jira**:
    - Use `mcp0_createJiraIssue` with:
      - `projectKey`: "${JIRA_PROJECT_KEY}"
      - `issueTypeName`: "Epic"
      - `summary`: Epic title
      - `description`: Full Epic content
4.  **Update Product Spec Confluence Page**:
    - Use `mcp0_getConfluencePage` to read current page
    - Locate "Links" table in page body
    - Add Epic link row to table: `| Epic | [${JIRA_PROJECT_KEY}-XXX](Epic URL) | |`
    - Use `mcp0_updateConfluencePage` to save updated page body
    - **DO NOT use `mcp0_createConfluenceFooterComment`** - footer comments are insufficient for traceability
5.  **Presentation & Gate**:
    - Display created Epic key and URL
    - **STOP** and request approval

**Standard Approval Format**:
```
✅ **FeaturePlanning Complete**
- **Artifact**: [Epic Jira URL]
- **Summary**: Epic created with behavioral contracts and acceptance criteria
- **Next Step**: TechSpec Mode

> **⏸️ APPROVAL REQUIRED**: Please review the Epic in Jira. Reply with:
> - `Approve` to proceed to Tech Spec creation
> - `Revise [feedback]` to make changes
> - `Cancel` to stop workflow
```
