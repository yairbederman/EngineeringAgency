# Deviation Handling Protocol

When implementation differs from Figma design, follow this protocol.

## Step 1: Document in Comparison Table

Add a row to the visual comparison table with warning status:

```markdown
| Colors | bg-brand-500 expected | bg-blue-500 used | ⚠️ Token missing |
```

## Step 2: Categorize Deviation

| Category | Description | Action |
|----------|-------------|--------|
| **Token Missing** | Design uses value not in project tokens | Use closest match, document |
| **Intentional Override** | Design requires one-off value | Add inline style with comment |
| **Design System Gap** | Pattern needed but doesn't exist | Flag for design system update |
| **Implementation Limitation** | Technical constraint prevents match | Document workaround |

## Step 3: Add to Commit Message

Include deviations in commit message body:

```
[PROJ-XXX] Implement ComponentName

Deviations from Figma:
- Used bg-blue-500 (closest match for #1E40AF not in design-tokens)
- Spacing 13px rounded to gap-3 (12px) per token grid
- Shadow uses elevation-md instead of custom 0 4px 12px
```

## Step 4: Escalation Criteria

Add Jira label `needs-design-review` if:
- [ ] More than 3 deviations in single component
- [ ] Any color deviation (brand consistency)
- [ ] Typography deviation (font-family or weight)
- [ ] Layout structure differs from design tree

## Step 5: Jira Comment Format

```markdown
### 🎨 Design Deviations Detected

| Element | Figma Value | Implementation | Reason |
|---------|-------------|----------------|--------|
| Background | #1E40AF | bg-blue-500 | Token not available |
| Gap | 13px | 12px (gap-3) | Nearest token value |

**Recommendation**: [Add token / Accept deviation / Request design update]
```
