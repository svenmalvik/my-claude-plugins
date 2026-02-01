# Infinite Claude

A Claude Code configuration for fully autonomous feature development. Give it a spec, get working code.

## What This Is

This repository contains a `.claude/` folder with skills, agents, and commands that enable Claude Code to develop features without user interaction. The workflow handles planning, implementation, testing, fixing, and code cleanup automatically.

## Quick Start

1. Copy the `.claude/` folder to your project
2. Create a spec file describing your feature
3. Run:
   ```
   /feature path/to/your-spec.md
   ```
4. Wait for the autonomous workflow to complete
5. Find your code in `projects/{spec-name}/`

## How It Works

The `/feature` command triggers a 6-phase autonomous workflow:

| Phase | What Happens |
|-------|--------------|
| 1. Setup | Creates project folder from spec name |
| 2. Planning | Generates implementation plan |
| 3. Implementation | Executes plan without checkpoints |
| 4. Test-Fix Loop | Tests as a user would, fixes failures (max 5 iterations) |
| 5. File Length Check | Splits files > 300 LOC into modules |
| 6. Final | Generates README with usage instructions |

## Components

### Skills

| Skill | Purpose |
|-------|---------|
| `feature-workflow` | Documents the autonomous workflow |
| `autonomous-executing-plans` | Executes plans without stopping for review |
| `code-simplifier` | Cleans up code for clarity |
| `go-best-practices` | Idiomatic Go patterns |
| `go-safety-critical` | NASA/JPL Power of 10 rules for Go |
| `cobra-cli` | Building CLI apps with Cobra |
| `bubbletea-tui` | Terminal UI development |
| `interview` | Requirements gathering through structured questions |

### Agents

| Agent | Purpose |
|-------|---------|
| `test-fix-loop` | Iteratively test and fix until working |
| `file-refactorer` | Split long files into focused modules |
| `code-simplifier` | Apply code simplification skill |

### Commands

| Command | Purpose |
|---------|---------|
| `/feature [spec.md]` | Run full autonomous workflow |
| `/go-refine` | Refine Go code |

## Configuration

Constants that control the workflow:

| Constant | Value | Purpose |
|----------|-------|---------|
| MAX_ITERATIONS | 5 | Test-fix loop limit |
| MAX_LOC | 300 | Max lines per file |
| MAX_REFACTOR_ROUNDS | 10 | File splitting limit |

## Writing Specs

Create a markdown file describing what you want built. Include:

- Feature description
- Expected behavior
- Any constraints or requirements
- Example usage (if applicable)

The workflow will interpret the spec and make autonomous decisions when details are ambiguous.

## Decision-Making Rules

When facing ambiguity, the workflow:

1. Chooses the simpler approach
2. Prefers standard library over external dependencies
3. Prefers explicit over clever
4. Ships working code over perfect code
5. Picks the first valid option when multiple exist

## License

MIT
