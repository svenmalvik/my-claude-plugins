---
name: autonomous-executing-plans
description: Use when executing an implementation plan fully autonomously without review checkpoints. Based on superpowers:executing-plans but removes all stopping points.
---

# Autonomous Plan Execution

## Overview

Load plan and execute ALL tasks autonomously without stopping for review.

**Core principle:** Complete autonomous execution. No checkpoints. No feedback loops. Make decisions and proceed.

**Announce at start:** "I'm using the autonomous-executing-plans skill to implement this plan without checkpoints."

## CRITICAL: Autonomous Operation Rules

**You MUST NOT:**
- Ask clarifying questions
- Wait for user feedback
- Present options for the user to choose
- Stop between batches
- Say "Ready for feedback"
- Request approval before continuing

**You MUST:**
- Make decisions independently
- Choose the simplest option when unclear
- Proceed through ALL tasks without stopping
- Handle failures by attempting fixes (up to 3 times per task)
- Skip tasks that cannot be completed after 3 attempts
- Complete the entire plan before returning

## The Process

### Step 1: Load Plan

1. Read the plan file
2. Create TodoWrite with all tasks from the plan
3. Proceed immediately to execution (no review phase)

### Step 2: Execute ALL Tasks

Execute every task in sequence:

For each task:
1. Mark as in_progress
2. Follow each step exactly as written in the plan
3. Run all verifications specified
4. If verification fails: attempt to fix (up to 3 attempts)
5. Mark as completed (or skipped if failed after 3 attempts)
6. **Continue immediately to next task** (no stopping)

### Step 3: Handle Blockers Autonomously

When you encounter issues:

| Issue | Autonomous Response |
|-------|---------------------|
| Missing dependency | Install it or skip task |
| Test fails | Analyze and fix (up to 3 attempts) |
| Instruction unclear | Use best judgment, choose simplest option |
| Verification fails | Fix the code or skip after 3 attempts |
| File not found | Create it or adjust path |
| Syntax error | Fix it |

**Never stop to ask. Always proceed.**

### Step 4: Complete

After ALL tasks executed:
1. Run final build/test verification
2. Log completion summary:
   - Tasks completed: X
   - Tasks skipped: Y (with reasons)
   - Final verification: PASS/FAIL
3. Return (do not invoke finishing-a-development-branch or ask about next steps)

## Decision Making Guidelines

When facing ambiguity:

1. **Choose simpler** - Straightforward over clever
2. **Use standard lib** - Avoid new dependencies when possible
3. **First option** - When multiple valid approaches, pick first
4. **Minimal change** - Don't over-engineer
5. **Working > perfect** - Ship it

## Differences from superpowers:executing-plans

| superpowers:executing-plans | autonomous-executing-plans |
|-----------------------------|---------------------------|
| Batch of 3 tasks | ALL tasks at once |
| Report between batches | No reports until done |
| Wait for feedback | Never wait |
| Stop when blocked | Try to fix, then skip |
| Ask for clarification | Use best judgment |
| Invoke finishing skill | Just report and return |

## Remember

- Execute the ENTIRE plan without stopping
- No checkpoints, no feedback requests
- Handle problems autonomously
- Skip what you can't fix after 3 attempts
- Report only at the very end
