---
description: Autonomous feature development from spec file
argument-hint: [spec-file.md]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Task
---

# Autonomous Feature Development

Execute the complete feature development workflow autonomously from a specification file.

**CRITICAL: This workflow runs 100% autonomously. Do not ask questions, do not wait for approval, do not present options. Make decisions and proceed.**

## Agent Execution Rules

- Use explicit language: "Launch sub-agent: X" before each agent invocation
- Use the Task tool with `subagent_type` parameter to launch agents
- Phases 4 and 5 are sequential (test-fix must complete before file splitting)
- For future parallel execution, use `superpowers:dispatching-parallel-agents` skill

## Input

Spec file: @$1

## Phase 1: Setup

1. Extract spec filename without extension to determine project name
2. Create project folder: `projects/{spec-name}/`
3. Log: "Starting autonomous feature development for {spec-name}"

## Phase 2: Planning

Use the superpowers:writing-plans skill to create an implementation plan based on the spec.

Requirements for the plan:
- All new code files must be created in `projects/{spec-name}/`
- Save plan to `docs/plans/{date}-{spec-name}.md`
- Do NOT ask which execution approach to use - proceed directly to implementation

After plan is written, do NOT stop for feedback. Proceed immediately to Phase 3.

## Phase 3: Implementation

Use the `autonomous-executing-plans` skill to implement the plan.

The skill will:
- Execute ALL tasks from docs/plans/{date}-{spec-name}.md
- Run without checkpoints or feedback requests
- Handle blockers autonomously (fix or skip after 3 attempts)
- Complete the entire plan before returning

All code must be created in `projects/{spec-name}/`.

## Phase 4: Test-Fix Loop

**Launch sub-agent: `test-fix-loop`**

Use the Task tool with:
- `subagent_type`: "test-fix-loop"
- `prompt`: "Test the feature in projects/{spec-name}/ from a user perspective. Fix any failures. Maximum 5 iterations. After fixes, apply: (1) Code Quality Review - run linters, fix all warnings, (2) Security Review - check OWASP top 10, injection, auth, (3) Logging - add structured logging, (4) Use the code-simplifier skill to clean up code."

Wait for sub-agent to complete before proceeding.

## Phase 5: File Length Check

**Launch sub-agent: `file-refactorer`**

Use the Task tool with:
- `subagent_type`: "file-refactorer"
- `prompt`: "Check all code files in projects/{spec-name}/ for files exceeding 300 lines. Split any long files into focused modules. Maximum 10 refactoring rounds."

Wait for sub-agent to complete before proceeding.

## Phase 6: Final Instructions

Generate clear instructions for the user on how to test and run the implemented feature:

1. Prerequisites or dependencies needed
2. Step-by-step commands to run the feature
3. Expected output or behavior
4. Any configuration or environment variables required

Write these instructions to `projects/{spec-name}/README.md`.

## Completion

Log: "Feature development complete. See projects/{spec-name}/ for implementation and README.md for usage instructions."
