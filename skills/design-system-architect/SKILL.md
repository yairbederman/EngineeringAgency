---
name: design-system-architect
description: Build consistent, accessible UI with design tokens, component libraries, and automated testing. Use when establishing or evolving a design system.
---

# Design System Architect

Build and maintain consistent, accessible design systems.

## When to Use

- Creating new design system
- Adding components to existing system
- Defining design tokens
- Establishing component patterns
- Design-to-code automation

## Design Tokens

### Token Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    Global Tokens                            │
│  (Brand agnostic: colors, spacing, typography scales)       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Alias Tokens                             │
│  (Semantic: primary, success, text-primary, spacing-md)     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Component Tokens                         │
│  (Specific: button-bg, card-padding, input-border)          │
└─────────────────────────────────────────────────────────────┘
```

### Token Structure

```json
{
  "color": {
    "primitive": {
      "blue-500": { "value": "#3B82F6" },
      "gray-900": { "value": "#111827" }
    },
    "semantic": {
      "primary": { "value": "{color.primitive.blue-500}" },
      "text-primary": { "value": "{color.primitive.gray-900}" }
    }
  },
  "spacing": {
    "xs": { "value": "4px" },
    "sm": { "value": "8px" },
    "md": { "value": "16px" },
    "lg": { "value": "24px" },
    "xl": { "value": "32px" }
  },
  "typography": {
    "font-family": {
      "sans": { "value": "Inter, system-ui, sans-serif" },
      "mono": { "value": "JetBrains Mono, monospace" }
    },
    "font-size": {
      "xs": { "value": "12px" },
      "sm": { "value": "14px" },
      "base": { "value": "16px" },
      "lg": { "value": "18px" },
      "xl": { "value": "20px" }
    }
  }
}
```

## Atomic Design

### Component Hierarchy

| Level | Description | Examples |
|-------|-------------|----------|
| **Atoms** | Basic building blocks | Button, Input, Icon, Text |
| **Molecules** | Groups of atoms | SearchField, FormField, Card |
| **Organisms** | Complex components | Header, ProductCard, Form |
| **Templates** | Page layouts | DashboardLayout, AuthLayout |
| **Pages** | Specific instances | HomePage, SettingsPage |

### Component Structure

```
components/
├── atoms/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.styles.ts
│   │   ├── Button.test.tsx
│   │   └── Button.stories.tsx
│   └── Input/
│       └── ...
├── molecules/
│   └── FormField/
│       └── ...
└── organisms/
    └── Header/
        └── ...
```

## Component API Design

### Props Interface

```typescript
interface ButtonProps {
  // Variants
  variant: 'primary' | 'secondary' | 'ghost' | 'danger'
  size: 'sm' | 'md' | 'lg'
  
  // States
  disabled?: boolean
  loading?: boolean
  
  // Content
  children: React.ReactNode
  leftIcon?: React.ReactNode
  rightIcon?: React.ReactNode
  
  // Behavior
  onClick?: (event: React.MouseEvent) => void
  type?: 'button' | 'submit' | 'reset'
}
```

### Compound Components

```tsx
// Usage
<Card>
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
  <Card.Footer>Actions</Card.Footer>
</Card>
```

## Accessibility

### Requirements Checklist

- [ ] Color contrast ratios (WCAG AA minimum)
- [ ] Keyboard navigation
- [ ] Focus visible states
- [ ] Screen reader labels
- [ ] ARIA attributes where needed
- [ ] Reduced motion support

### Implementation

```tsx
const Button = ({ children, ...props }: ButtonProps) => {
  return (
    <button
      {...props}
      aria-disabled={props.disabled}
      aria-busy={props.loading}
      className={cn(styles.button, {
        [styles.loading]: props.loading
      })}
    >
      {props.loading ? (
        <>
          <Spinner aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  )
}
```

## Documentation

### Component Documentation

```tsx
/**
 * Primary action button for forms and CTAs.
 *
 * @example
 * <Button variant="primary" onClick={handleClick}>
 *   Submit
 * </Button>
 */
```

### Storybook Stories

```tsx
export default {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    variant: { control: 'select', options: ['primary', 'secondary'] },
    size: { control: 'select', options: ['sm', 'md', 'lg'] }
  }
}

export const Primary = { args: { variant: 'primary', children: 'Button' } }
export const Secondary = { args: { variant: 'secondary', children: 'Button' } }
```

## Design System Checklist

Before releasing component:

- [ ] Tokens used (no magic numbers)
- [ ] All variants documented
- [ ] Accessibility tested
- [ ] Keyboard navigation works
- [ ] Stories cover all states
- [ ] Responsive behavior defined
- [ ] Dark mode supported
- [ ] Animation respects reduced motion
