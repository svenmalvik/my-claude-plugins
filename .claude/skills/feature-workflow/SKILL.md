---
name: feature-workflow
description: This skill should be used when the user asks to "develop a feature autonomously", "run the feature workflow", "implement from spec", "automate feature development", or needs guidance on the autonomous feature development process including planning, implementation, testing, and refactoring phases.
---

# Autonomous Feature Workflow

## Overview

This skill documents the fully autonomous feature development workflow that takes a specification file and produces working, tested, clean code without any user interaction.

**Key principle:** 100% autonomous operation. No questions, no approvals, no options presented. Make decisions and proceed.

## The Workflow

### Phase 1: Setup

1. Read the specification file
2. Extract project name from spec filename
3. Create project folder: `projects/{spec-name}/`

### Phase 2: Planning

Use `superpowers:writing-plans` to create an implementation plan.

**Autonomous override:** Do not ask which execution approach to use. Do not wait for feedback. Proceed directly to implementation after plan is written.

Plan requirements:
- All code files go in `projects/{spec-name}/`
- Save plan to `docs/plans/YYYY-MM-DD-{spec-name}.md`
- Include bite-sized tasks with exact file paths
- Include test commands and expected outputs

### Phase 3: Implementation

Use the `autonomous-executing-plans` skill to implement the plan.

The skill:
- Executes ALL tasks from the plan without stopping
- Runs verifications as specified
- Handles failures autonomously (fix or skip after 3 attempts)
- Does NOT stop for review checkpoints
- Completes entire plan before returning

### Phase 4: Test-Fix Loop

**Launch sub-agent: `test-fix-loop`** to test and fix the feature.

Configuration:
- MAX_ITERATIONS: 5

**User-Perspective Testing (CRITICAL):**

Actually USE the feature as a real user would - not just unit tests:
- **CLI tools:** Run commands, provide input, verify output
- **Calculators:** Press keys, enter numbers, check results
- **TUI apps:** Navigate menus, use hotkeys, interact with elements
- **macOS menubar:** Verify menu items exist, test hotkeys work
- **APIs:** Make real HTTP requests, verify responses
- **Scripts:** Run with real inputs, check outputs

All interactive elements must work (buttons, keys, hotkeys, menus).

After fixes, apply quality improvements:
- Code Quality Review (fix all warnings, run linters)
- Security Review (check OWASP top 10, injection, auth)
- Logging/Tracing (add structured logging)
- `code-simplifier` skill (clean up code)

Loop continues until:
- All user interactions work correctly, OR
- MAX_ITERATIONS reached

### Phase 5: File Length Check

**Launch sub-agent: `file-refactorer`** to enforce code organization.

Configuration:
- MAX_LOC: 300 lines per file
- MAX_REFACTOR_ROUNDS: 10

The agent:
- Finds files exceeding 300 LOC
- Splits them into focused modules
- Updates imports/exports
- Verifies code still works
- Repeats until all files under limit

### Phase 6: Final Instructions

Generate user documentation:
- Prerequisites/dependencies
- Step-by-step run commands
- Expected output/behavior
- Configuration/environment variables

Save to `projects/{spec-name}/README.md`.

## Entry Point

Use the `/feature` command:

```
/feature path/to/spec.md
```

This orchestrates all phases automatically.

## Configuration Constants

| Constant | Value | Purpose |
|----------|-------|---------|
| MAX_ITERATIONS | 5 | Test-fix loop limit |
| MAX_LOC | 300 | Max lines per file |
| MAX_REFACTOR_ROUNDS | 10 | File splitting limit |

## Decision-Making Rules

When agents face ambiguity:

1. **Choose simpler** - Prefer straightforward approaches
2. **Use standard lib** - Avoid external dependencies when possible
3. **Be explicit** - Clear code over clever code
4. **Working over perfect** - Ship, then improve
5. **First option** - When multiple valid choices, pick first

## Agent Execution Rules

**Explicit launch language:** Always say "Launch sub-agent: X" before invoking an agent.

**Use Task tool with `subagent_type` parameter:**
```
Task tool:
  subagent_type: "test-fix-loop"
  prompt: "Test the feature..."
```

**Sequential vs Parallel:**
- Phases 4 and 5 run **sequentially** (test-fix must pass before file splitting)
- For parallel agent execution, use `superpowers:dispatching-parallel-agents` skill
- In this workflow, parallelism is limited because phases depend on each other

## Skills and Agents

| Component | Type | Purpose | Phase |
|-----------|------|---------|-------|
| `superpowers:writing-plans` | Skill | Create implementation plan | 2 |
| `autonomous-executing-plans` | Skill | Execute plan without checkpoints | 3 |
| `test-fix-loop` | Agent | Test and fix iteratively | 4 |
| `file-refactorer` | Agent | Split long files | 5 |
| `code-simplifier` | Skill | Clean up code | 4 |

## Quality Improvements Applied

During test-fix loop, these improve code quality:

**Actual Skills (invoked via Skill tool):**
- `code-simplifier` - Clean up code for clarity

**Inline Quality Checks (executed directly):**
- **Code Quality Review** - Run linters, fix all warnings
- **Security Review** - OWASP top 10, injection, auth checks
- **Logging/Tracing** - Add structured logging at key points
- **Verification** - Run build and tests, confirm passes before done

## Output Structure

After workflow completes:

```
projects/{spec-name}/
├── README.md           # Usage instructions
├── src/                # Source code (or language-appropriate structure)
├── tests/              # Test files
└── ...                 # Other project files
```
