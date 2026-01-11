# Engineering Agent – ProductRoadmap Mode

> **Persona**: Load `${AGENT_ROOT}/personas/product-manager.md`

## Product Roadmap Analysis – Future Feature Discovery

**Goal**: Before selecting a tech stack, understand the product's future trajectory to make architecture decisions that scale appropriately.

**Prerequisite**: Approved Epic from FeaturePlanning mode.

**Trigger**: This step is **MANDATORY for new projects**. Skip if Epic only modifies an existing codebase with established architecture.

**Inputs**:
- Approved Epic (from FeaturePlanning)
- Product Spec (for context)
- User input on future plans

**Output**:
- **Product Roadmap Summary**: Document capturing future phases and their technical implications
- **Input to Tech Stack Decision**: Ensures tech stack fits long-term vision

---

## Flow

### Step 1: Present Future Feature Discovery Questions

Ask the user about planned future capabilities. This directly informs tech stack selection.

**Standard Question Set**:

```markdown
## 📋 Future Feature Discovery

To select the right tech stack, I need to understand where this product is headed:

### Content & Publishing
1. Will you need a **CMS or blog** for content updates?
   - [ ] Yes, client should edit content themselves
   - [ ] No, all content updates go through a developer
   - [ ] Undecided

### User Accounts & Data
2. Will you need **user accounts, login, or personalization**?
   - [ ] Yes, users log in
   - [ ] No, public site only
   - [ ] Future phase

### E-Commerce
3. Will there be **payments, subscriptions, or e-commerce**?
   - [ ] Yes
   - [ ] No
   - [ ] Future phase

### Integrations
4. Will you need **third-party integrations** (CRM, calendar, booking, chat)?
   - [ ] Yes: [list them]
   - [ ] No
   - [ ] Future phase

### Scale & Performance
5. How much **traffic** do you expect initially?
   - [ ] Low (< 1K visitors/month)
   - [ ] Medium (1K-50K/month)
   - [ ] High (> 50K/month)

### Maintenance
6. Who will **maintain** this after launch?
   - [ ] Developer/agency (you or similar)
   - [ ] Client's internal team
   - [ ] No ongoing maintenance planned

---

Please answer the questions above, or reply `Skip` if this is a simple MVP with no planned expansion.
```

### Step 2: Document Roadmap Summary

Based on user responses, create a Product Roadmap Summary:

```markdown
## Product Roadmap Summary

**Project**: [Project Name]
**Date**: [YYYY-MM-DD]

### Phase 1 (Current Epic)
[Summary of current scope]

### Planned Future Phases

| Phase | Features | Timeline | Technical Impact |
|-------|----------|----------|------------------|
| Phase 2 | [Feature] | [Q2 2026] | [Needs X capability] |
| Phase 3 | [Feature] | [Q4 2026] | [Requires Y stack] |

### Technical Implications for Tech Stack

| Future Need | Impact on Tech Stack Choice |
|-------------|----------------------------|
| CMS/Blog | Favor WordPress or headless CMS integration |
| User Accounts | Need backend framework, database |
| E-Commerce | Favor Shopify, WooCommerce, or custom backend |
| High Traffic | Need SSG/CDN optimization |
| Client Editing | Need CMS or visual builder |
| Developer-Only | Static site or code-first framework OK |

### Recommendation
Based on future plans, the recommended tech stack direction is: [Static / Framework / CMS / Full-stack]
```

### Step 3: Add to Epic Decisions Log

Update the Epic's Decisions Log with roadmap context:

```markdown
| Date | Decision | Rationale |
|------|----------|-----------|
| YYYY-MM-DD | Product Roadmap: [Static/CMS/Full-stack] | [Summary of future plans impacting tech choice] |
```

### Step 4: Proceed to Tech Stack Decision

With roadmap context established, present Tech Stack options (Gate 3.5) with explicit mapping to future needs.

---

## ⛔ HARD STOP: Roadmap Analysis Gate

> [!CAUTION]
> **This is a BLOCKING gate for new projects.**

**After presenting questions, you MUST:**
1. Wait for user to answer roadmap questions
2. Document the Product Roadmap Summary
3. **STOP and WAIT** before proceeding to Tech Stack Decision
4. Present roadmap summary for user review

**Valid Responses:**
- User answers questions → Document and proceed to Tech Stack Decision
- `Skip` → Proceed to Tech Stack Decision with assumption: "Simple MVP, no planned expansion"

**On Completion**: → Immediately proceed to **TechStackDecision** (Gate 3.5)

---

## Skip Conditions

This mode can be **skipped** when:
- Epic only modifies an existing project with established architecture
- User explicitly states "no future plans, simple MVP"
- Project is a bug fix or minor feature addition

When skipped, document in Epic: `Roadmap Analysis: Skipped (existing architecture / simple MVP)`
