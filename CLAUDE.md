# Claude Code Plugins Marketplace

This repository is a marketplace for Claude Code plugins.

## Structure

- `marketplace.json` - Manifest listing all available plugins
- `<plugin-name>/` - Individual plugin directories
  - `.claude-plugin/plugin.json` - Plugin metadata
  - `commands/` - Slash commands
  - `agents/` - Subagents
  - `skills/` - Skills
  - `hooks/` - Event hooks (if any)
  - `CLAUDE.md` - Plugin-specific instructions

## Adding a New Plugin

1. Create a new directory for your plugin
2. Add `.claude-plugin/plugin.json` with metadata
3. Add your commands, agents, skills, and hooks
4. Update `marketplace.json` to include the new plugin

## Available Plugins

- **go-autopilot** - Autonomous Go development from spec to working code
