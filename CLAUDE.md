# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent OS is a spec-driven agentic development framework that transforms AI coding agents into productive developers through structured workflows. It provides commands, agents, and standards that can be installed into any project to enable AI-assisted development with Claude Code, Cursor, Windsurf, or other AI coding tools.

**Language:** Bash scripts, YAML configuration, Markdown instruction files
**Version:** 2.1.1

## Common Commands

### Install Agent OS into a project
```bash
# From target project directory
~/agent-os/scripts/project-install.sh

# With specific profile
~/agent-os/scripts/project-install.sh --profile [name]

# With configuration overrides
~/agent-os/scripts/project-install.sh --claude-code-commands true --use-claude-code-subagents true

# Preview changes without installing
~/agent-os/scripts/project-install.sh --dry-run
```

### Update existing installation
```bash
~/agent-os/scripts/project-update.sh
```

### Create custom profile
```bash
~/agent-os/scripts/create-profile.sh [profile-name]
```

### Test bash scripts
```bash
bash -n scripts/project-install.sh  # Syntax check
```

## Architecture

### Core Structure
- `config.yml` - Global configuration (version, feature flags, default profile)
- `scripts/` - Installation and setup bash scripts
- `profiles/` - Workflow profiles containing commands, agents, standards, and workflows

### Profile Anatomy (`profiles/default/`)
```
profiles/default/
├── commands/           # Agent OS commands (6 phases)
│   ├── plan-product/   # Product planning
│   ├── shape-spec/     # Feature shaping before spec writing
│   ├── write-spec/     # Spec formalization
│   ├── create-tasks/   # Task list generation
│   ├── implement-tasks/# Single-agent implementation
│   └── orchestrate-tasks/ # Multi-agent orchestration
├── agents/             # Claude Code subagent definitions
├── standards/          # Coding standards (global, frontend, backend, testing)
└── workflows/          # Step-by-step workflow instructions
```

### Command Variants
Each command has two variants:
- `multi-agent/` - Delegates to Claude Code subagents
- `single-agent/` - Executes directly in main agent (numbered sequence files)

### Template Syntax
Markdown files use a custom template syntax:
- `{{workflows/implementation/implement-tasks}}` - Include another file
- `{{standards/*}}` - Include all files from a directory
- `{{UNLESS condition}}...{{ENDUNLESS condition}}` - Conditional blocks
- `{{IF condition}}...{{ENDIF condition}}` - Conditional blocks
- `@agent-os/path` - Project-relative path reference

### Agent Definition Format
Agents use YAML frontmatter:
```yaml
---
name: implementer
description: Use proactively to implement a feature...
tools: Write, Read, Bash, WebFetch, Playwright, Skill
color: red
model: inherit
---
```

### Configuration Options (config.yml)
- `claude_code_commands` - Install to `.claude/commands/agent-os/`
- `use_claude_code_subagents` - Enable subagent delegation
- `agent_os_commands` - Install to `agent-os/commands/`
- `standards_as_claude_code_skills` - Convert standards to Claude Code Skills
- `profile` - Default profile to use

## Development Workflow

### The 6 Development Phases
1. **plan-product** - Establish product mission, roadmap, tech stack
2. **shape-spec** - Shape rough ideas into requirements.md
3. **write-spec** - Formalize requirements into spec.md
4. **create-tasks** - Break spec into prioritized tasks.md
5. **implement-tasks** - Execute tasks (single-agent)
6. **orchestrate-tasks** - Execute with multi-agent control

### Script Conventions
- All scripts use `set -e` for exit-on-error
- Common utilities in `scripts/common-functions.sh`
- Flags accept both hyphens and underscores (`--use-claude-code-subagents` or `--use_claude_code_subagents`)
- `--dry-run` and `--verbose` flags available on install scripts

## Contributing

This project uses a Discussions-first workflow:
- Bug reports → GitHub Discussions (Bugs category)
- Feature ideas → GitHub Discussions (Ideas category)
- PRs for bug fixes should include `[bug fix]` in title
- Feature PRs require prior Discussion approval
