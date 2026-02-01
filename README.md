# Claude Code Plugins Marketplace

A collection of Claude Code plugins for autonomous development workflows.

## Available Plugins

### go-autopilot

Autonomous Go development - from spec to working code.

**Commands:**
- `/feature [spec.md]` - Run full autonomous workflow
- `/go-refine` - Refine Go code

**Agents:**
- `code-simplifier` - Apply code simplification
- `test-fix-loop` - Iteratively test and fix until working
- `file-refactorer` - Split long files into focused modules

**Skills:**
- `autonomous-executing-plans` - Execute plans without stopping for review
- `bubbletea-tui` - Terminal UI development
- `cobra-cli` - Building CLI apps with Cobra
- `code-simplifier` - Clean up code for clarity
- `feature-workflow` - Documents the autonomous workflow
- `go-best-practices` - Idiomatic Go patterns
- `go-safety-critical` - NASA/JPL Power of 10 rules for Go
- `interview` - Requirements gathering through structured questions

## Installation

Add a plugin to your Claude Code configuration:

```bash
claude plugins install https://github.com/svenmalvik/my-claude-plugins/go-autopilot
```

Or manually copy the plugin directory to your `.claude/plugins/` folder.

## Repository Structure

```
marketplace.json           # Manifest listing all plugins
<plugin-name>/
  .claude-plugin/
    plugin.json            # Plugin metadata
  commands/                # Slash commands
  agents/                  # Subagents
  skills/                  # Skills
  hooks/                   # Event hooks (if any)
  CLAUDE.md                # Plugin-specific instructions
```

## Adding a New Plugin

1. Create a new directory for your plugin
2. Add `.claude-plugin/plugin.json` with metadata
3. Add your commands, agents, skills, and hooks
4. Update `marketplace.json` to include the new plugin

## License

MIT
