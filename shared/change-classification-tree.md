# Change Classification Decision Tree

> **Purpose**: Reusable decision tree for classifying changes as Minor, Scope, or Major.
> **Used by**: `change-request.md` mode and gate handlers.

---

## Quick Decision Flow

```
                    ┌─────────────────────────────┐
                    │  User Request Arrives       │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │  Is this changing existing  │
                    │  scope/requirements?        │
                    └─────────────┬───────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │ NO                │                   │ YES
              ▼                   │                   ▼
    ┌─────────────────┐           │         ┌─────────────────┐
    │ New Feature?    │           │         │ Affects DB/API  │
    │ (Product        │           │         │ schema?         │
    │  Increment)     │           │         └────────┬────────┘
    └────────┬────────┘           │                  │
             │                    │         ┌────────┼────────┐
    ┌────────┼────────┐           │         │ YES    │        │ NO
    │ YES    │    NO  │           │         ▼        │        ▼
    ▼        │        ▼           │  ┌──────────┐    │  ┌──────────┐
┌────────┐   │  ┌──────────┐      │  │  MAJOR   │    │  │ <2 hours?│
│ SCOPE  │   │  │ MINOR    │      │  │  CHANGE  │    │  └────┬─────┘
└────────┘   │  │ (clarif) │      │  └──────────┘    │       │
             │  └──────────┘      │                  │  ┌────┼────┐
             │                    │                  │  │YES │    │NO
             │                    │                  │  ▼    │    ▼
             │                    │            ┌──────────┐  │ ┌──────────┐
             │                    │            │  MINOR   │  │ │  SCOPE   │
             │                    │            └──────────┘  │ └──────────┘
             │                    │                          │
             └────────────────────┴──────────────────────────┘
```

---

## Detailed Decision Matrix

### Step 1: Is this a change to existing work?

| Check | Result | Action |
|-------|--------|--------|
| No existing artifacts in `.specs/` | No → New work | Use standard planning flow |
| Existing artifacts found | Yes → Potential change | Continue to Step 2 |

---

### Step 2: What type of change?

| Question | Yes | No |
|----------|-----|-----|
| **Changes acceptance criteria?** | → Scope Change | Continue |
| **Adds new user-facing behavior?** | → Scope Change | Continue |
| **Removes planned functionality?** | → Scope Change | Continue |
| **Changes API contract?** | → Major Change | Continue |
| **Requires database schema change?** | → Major Change | Continue |
| **Affects security model?** | → Major Change | Continue |
| **Contradicts merged/deployed code?** | → Major Change | Continue |
| **Can be done in <2 hours?** | → Minor Adjustment | → Scope Change |
| **Is a typo/wording fix only?** | → Minor Adjustment | Continue |
| **Clarifies without changing intent?** | → Minor Adjustment | Continue |

---

## Classification Definitions

### Minor Adjustment

**Definition**: Changes that don't affect what we deliver or how it works.

**Examples**:
- Fixing typos in documentation
- Clarifying ambiguous wording
- Adding missing details that don't change scope
- Updating file paths or references
- Formatting improvements

**Characteristics**:
- No code changes required
- No test changes required
- Effort < 2 hours
- No stakeholder notification needed

**Approval**: Auto-apply, inform user.

---

### Scope Change

**Definition**: Changes that modify what we deliver or how it behaves.

**Examples**:
- Adding a new feature requirement
- Changing UI behavior
- Modifying acceptance criteria
- Adjusting workflow logic
- Adding/removing fields
- Changing validation rules

**Characteristics**:
- May require code changes
- May require test updates
- Effort 2-16 hours typically
- Stakeholders should be aware

**Approval**: User must confirm with "Acknowledge scope change".

---

### Major Change

**Definition**: Fundamental changes affecting architecture, data, or core assumptions.

**Examples**:
- Changing database schema
- Modifying API contracts
- Changing authentication model
- Pivoting core functionality
- Contradicting merged/deployed code
- Security model changes

**Characteristics**:
- Requires significant refactoring
- May require data migration
- High risk of regression
- Cross-team coordination needed

**Approval**: Block until PM + TechLead explicitly confirm.

---

## Override Rules

### User Override

If user explicitly states a classification, respect it with verification:

```
User: "This is just a minor tweak"

Agent: "I've classified this as a Scope Change because it modifies 
        acceptance criteria. Are you sure you want to treat it as Minor?
        
        ⚠️ Minor changes are auto-applied without logging.
        
        Confirm: `Override to Minor` or `Keep as Scope`"
```

### Escalation Rule

If uncertain, **escalate up**:

```
Uncertain between Minor and Scope → Treat as Scope
Uncertain between Scope and Major → Treat as Major
```

---

## Context Modifiers

### Phase Modifiers

| Current Phase | Modifier |
|---------------|----------|
| PLANNING_SPEC | All changes are easier, lean toward Minor |
| PLANNING_EPIC | Standard classification |
| PLANNING_TECH | Lean toward Scope for technical changes |
| EXECUTION | Any code-affecting change is at least Scope |
| COMPLETION | All changes are at least Scope, consider Major |

### Source Modifiers

| Source | Modifier |
|--------|----------|
| PM | Trust classification (they understand scope) |
| DEV | Verify technical impact (may underestimate) |
| STAKE | Escalate if unclear (high visibility) |
| QA | Treat bugs seriously (P0/P1 default) |

---

## Examples

### Example 1: Minor

```
User: "Can you fix the typo in the footer - it says 'Copyrigt' instead of 'Copyright'"

Classification: Minor
Reason: Typo fix, no behavior change, <2 hours
Action: Auto-apply, log informally
```

### Example 2: Scope

```
User: "Actually, add a contact form to the footer"

Classification: Scope Change
Reason: Adds new user-facing behavior (form), requires implementation
Action: Require "Acknowledge scope change", update all artifacts
```

### Example 3: Major

```
User: "We need to change from email authentication to OAuth"

Classification: Major Change
Reason: Changes security model, requires significant refactoring
Action: Block, require PM + TechLead explicit approval
```

### Example 4: Edge Case

```
User: "Add a loading spinner to the submit button"

Classification: Could be Minor or Scope
Questions to ask:
- Is this in the current spec? (No → Scope)
- Is this a standard pattern already used? (Yes → Minor)
- Effort estimate? (<2h → Minor, >2h → Scope)

Decision: Ask user for clarification or lean toward Scope
```

---

## Integration Points

### Used By

- `change-request.md` - Main classification logic
- `_gates.md` - Gate-level change detection
- `product-spec-review.md` - Inline change handling
- `engineering-agent.md` - Pre-check classification

### Calls To

- `storage.logChange()` - After classification confirmed
- `storage.archiveArtifact()` - Before applying Scope/Major changes
