---
allowed-tools:
  - Read
  - Edit
  - Write
  - Grep
  - Glob
  - Bash
  - Task
---

# Go Code Refinement

You are refining Go code in this project. Apply all relevant project skills to improve code quality.

## Required Skills

Load and apply these skills in order:

1. **go-best-practices** - Idiomatic Go patterns, error handling, package design
2. **go-safety-critical** - Bounded loops, error discipline, memory predictability
3. **cobra-cli** - CLI patterns, command structure, flag handling (if touching CLI code)
4. **bubbletea-tui** - TUI patterns, Elm architecture, Lip Gloss styling (if touching TUI code)
5. **code-simplifier** - Clarity over brevity, avoid premature abstractions

## Workflow

1. **Identify scope** - What files/packages need refinement? Check recent changes with `git diff` or focus on user-specified files
2. **Read the code** - Understand current implementation before suggesting changes
3. **Apply skills systematically** - Go through each applicable skill's checklist
4. **Propose changes** - Explain which skill principle each change addresses
5. **Implement** - Make changes one skill/concern at a time
6. **Verify** - Run `go build ./...` and `go test ./...` after changes

## What to Look For

From **go-best-practices**:
- Error handling patterns (wrap with context, use sentinel errors)
- Package organization and naming
- Interface design (accept interfaces, return structs)

From **go-safety-critical**:
- Bounded loops and recursion
- Resource cleanup (defer patterns)
- Nil checks and defensive coding

From **cobra-cli** (if CLI code):
- Command structure and hierarchy
- Flag definitions and validation
- Help text quality

From **bubbletea-tui** (if TUI code):
- Model/Update/View separation
- Shared styles from tui package
- Keyboard handling patterns

From **code-simplifier**:
- Remove dead code
- Flatten nested conditionals
- Avoid premature abstractions (2-3 copies is OK)

## Important

- **Don't over-engineer** - A little copying is better than a little dependency
- **Preserve behavior** - Refine, don't refactor
- **One concern at a time** - Separate commits for separate improvements
