# Infinite Claude - Autonomous Feature Development

## Project Overview

This project provides fully autonomous feature development from specification to working code.

## Autonomous Workflow Rules

**CRITICAL: All agents and commands in this project operate WITHOUT user interaction.**

When executing autonomous workflows:
- Make decisions independently - pick the simplest/first option when unclear
- Never ask clarifying questions - use reasonable defaults
- Never wait for user approval - proceed with best judgment
- Never present multiple options - choose and execute
- If something fails, try to fix it automatically before giving up

## Configuration Constants

These values match the original automate_feature.sh behavior:

| Constant | Value | Purpose |
|----------|-------|---------|
| MAX_ITERATIONS | 5 | Maximum test-fix loop iterations |
| MAX_LOC | 300 | Maximum lines of code per file |
| MAX_REFACTOR_ROUNDS | 10 | Maximum file splitting rounds |

## Project Structure

Feature implementations are created in `projects/{spec-name}/` folders.

## Workflow Phases

1. **Planning** - Read spec, create implementation plan using superpowers:writing-plans
2. **Implementation** - Execute plan autonomously without checkpoints
3. **Test-Fix Loop** - Test from user perspective, fix failures (max 5 iterations)
4. **File Length Check** - Split files exceeding 300 LOC (max 10 rounds)
5. **Final Instructions** - Generate user documentation for running the feature

## Skills to Apply During Development

After implementation, apply these to improve code quality:

**Actual Skills (invoke with Skill tool):**
- `code-simplifier` - Clean up code for clarity and maintainability

**Inline Quality Checks (execute directly, not skills):**
- **Code Quality Review** - Run linters, fix all compiler/linter warnings, check for unused code
- **Security Review** - Check for injection vulnerabilities, auth issues, data exposure, OWASP top 10
- **Logging/Tracing** - Add structured logging at key points, error logging with context
- **Verification** - Run build and tests, confirm everything passes before claiming done

## Decision-Making Guidelines

When facing choices without clear guidance:
1. Choose the simpler approach
2. Prefer standard library over external dependencies
3. Prefer explicit over clever
4. Prefer working over perfect
