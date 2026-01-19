---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work
---

# Writing Skills

## Overview

**Writing skills IS Test-Driven Development applied to process documentation.**

You write test cases (pressure scenarios), watch them fail (baseline behavior), write the skill (documentation), watch tests pass (agents comply), and refactor (close loopholes).

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.

## What is a Skill?

A **skill** is a reference guide for proven techniques, patterns, or tools. Skills help future agent instances find and apply effective approaches.

**Skills are:** Reusable techniques, patterns, tools, reference guides

**Skills are NOT:** Narratives about how you solved a problem once

## Skill File Structure

Every skill lives in `global_workflows/skills/<skill-name>/SKILL.md`:

```yaml
---
name: skill-name
description: When to use this skill (appears in skill discovery)
---

# Skill Title

## Overview
Brief explanation of what this skill does.

## When to Use
Clear conditions for when this skill applies.

## The Process
Step-by-step instructions.

## Common Mistakes
What NOT to do.

## Checklist
Pre-completion verification items.
```

## Creating a New Skill

### Step 1: Identify the Need

- What problem does this skill solve?
- When should an agent use this skill?
- What's the failure mode without this skill?

### Step 2: Write the Description

The `description` in YAML frontmatter is critical for skill discovery:

**Good:** "Use when implementing any feature or bugfix, before writing implementation code"

**Bad:** "TDD stuff"

### Step 3: Write Clear Instructions

- Use imperative mood ("Do X", not "You should do X")
- Include checklists for verification
- Add "Common Mistakes" section
- Keep it concise but complete

### Step 4: Test the Skill

1. Describe a scenario where the skill should apply
2. Verify the skill instructions lead to the correct outcome
3. Identify edge cases and add guidance

## Quality Checklist

- [ ] YAML frontmatter has `name` and `description`
- [ ] Description clearly states when to use the skill
- [ ] Instructions are step-by-step and actionable
- [ ] Common mistakes section exists
- [ ] Pre-completion checklist exists
- [ ] No narrative storytelling—just reference material

## Skill Categories

| Category | Examples |
|----------|----------|
| **Core** | using-superpowers, test-driven-development |
| **Planning** | brainstorming, writing-plans |
| **Execution** | executing-plans, subagent-driven-development |
| **Review** | requesting-code-review |
