---
name: executing-plans
description: Use when you have a written implementation plan to execute with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute tasks in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints for review.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

## The Process

### Step 1: Load and Review Plan

1. Read plan file
2. Review critically—identify any questions or concerns
3. If concerns: Raise them before starting
4. If no concerns: Proceed

### Step 2: Execute Batch

**Default: First 3 tasks**

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Report

When batch complete:
- Show what was implemented
- Show verification output
- Say: "Ready for feedback."

### Step 4: Continue

Based on feedback:
- Apply changes if needed
- Execute next batch
- Repeat until complete

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## Checklist

Before reporting batch complete:

- [ ] All tasks in batch are marked complete
- [ ] All tests pass
- [ ] All verification steps ran successfully
- [ ] Changes committed to git
- [ ] No outstanding questions

## Common Mistakes

- Skipping verification steps
- Guessing when blocked instead of asking
- Proceeding when tests fail
- Not committing after each task
- Rushing through steps without reading carefully
