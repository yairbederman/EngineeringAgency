# Epic: [Action-Oriented Name]

**Goal**
- [One sentence: What business value does this deliver to the user?]

**Context**
- **Trigger**: [What user action or event starts this?]
- **Preconditions**: [What must be true beforehand? e.g., "User is logged in"]
- **Links**: [Jira/Confluence/Figma]

**Key Data Concepts (Business View)**
* *Note: List the information required, not database columns.*
* **Inputs**: [e.g., "Email, Phone Number, Selected Plan"]
* **Outputs**: [e.g., "Confirmation Email, Receipt"]

**Functional Flows (The "What", not the "How")**
* **Flow 1: [Happy Path Name]**
  1. **User** [takes action] (e.g., clicks "Upgrade")
  2. **System** checks [rule] (e.g., "is payment method valid?")
  3. **System** [performs business logic] (e.g., "upgrades the account tier")
  4. **System** responds with [result] (e.g., "shows success message")

* **Flow 2: [Error/Edge Case]**
  1. **User** [takes invalid action]
  2. **System** detects [business rule violation]
  3. **System** blocks action and explains why.

**Acceptance Criteria (Gherkin - Behavior Only)**
* *Focus on the user experience, not the code.*
* **Scenario: Successful Completion**
  - **Given** [User is on the page]
  - **When** [User completes flow]
  - **Then** [User sees success] AND [Account is updated]

**Scope & Constraints**
- **Must Support**: [e.g., "Mobile devices", "Guest users"]
- **Out of Scope**: [e.g., "Admin panel changes", "Payment processing"]
