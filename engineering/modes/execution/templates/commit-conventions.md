# Commit Conventions

Standardized commit message and branch naming conventions for the Engineering Agent.

## Branch Naming

```
feature/[TaskKey]-[short-kebab-summary]
```

**Examples**:
- `feature/PROJ-123-user-login`
- `feature/PROJ-456-search-widget`
- `bugfix/PROJ-789-null-pointer-fix`

**Rules**:
- Use `feature/` prefix for new features and enhancements
- Use `bugfix/` prefix for bug fixes
- Use `hotfix/` prefix for production emergencies
- Keep summary to 3-4 words max
- Use kebab-case (lowercase with hyphens)

## Commit Message Format

### Standard Commit

```
[PROJ-XXX] Short summary (imperative mood, max 50 chars)

Optional body with more details:
- What was changed
- Why it was changed
- Any notable decisions
```

### Commit with Deviations (Frontend)

```
[PROJ-XXX] Implement ComponentName

Deviations from Figma:
- [Deviation 1 with reason]
- [Deviation 2 with reason]
```

### Commit with Breaking Changes

```
[PROJ-XXX] Refactor API endpoint

BREAKING CHANGE: Response schema changed
- Old: { data: [] }
- New: { items: [], meta: {} }

Migration: Update all consumers to use `items` instead of `data`
```

## Commit Checklist

Before committing, verify:
- [ ] Message starts with Jira key in brackets
- [ ] Summary is imperative mood ("Add" not "Added")
- [ ] Summary is under 50 characters
- [ ] Body explains WHY, not just WHAT
- [ ] Deviations documented (if frontend)
- [ ] Breaking changes clearly marked

## Multi-File Commits

When a task touches multiple files, use a single commit with file list:

```
[PROJ-XXX] Add search functionality

Files changed:
- src/components/SearchWidget.tsx (new)
- src/hooks/useSearch.ts (new)
- src/pages/HomePage.tsx (modified)
- src/components/SearchWidget.test.tsx (new)
```
