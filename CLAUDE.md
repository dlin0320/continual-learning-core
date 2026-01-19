# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the public core of the continual learning skill system. It contains the three core skills that enable Claude to evolve based on session outcomes and user feedback.

## Repository Structure

```
retrospective/           # Post-session analysis skill
  SKILL.md              # Skill definition
  pending/              # Reflections awaiting review
  archive/              # Processed reflections
  templates/            # Session context and reflection templates
update/                 # Skill evolution engine
  SKILL.md              # Skill definition
  staged/               # Pending updates awaiting approval
digest/                 # Daily summary generator
  SKILL.md              # Skill definition
  logs/                 # Daily digest files (YYYY-MM-DD.md)
  state.yaml            # Digest scheduling state
CLAUDE.template.md      # Template for ~/.claude/CLAUDE.md
settings.template.json  # Reference config for ~/.claude/settings.json
```

## Key Files

- `{skill}/SKILL.md` - Skill definition with frontmatter and implementation
- `CLAUDE.template.md` - Usage instructions users append to their ~/.claude/CLAUDE.md
- `settings.template.json` - Reference configuration
- `retrospective/templates/` - Templates for session context and reflection output

## Git Workflow

Path-scoped operations for skill-specific history:

```bash
# Skill-specific history
git log --oneline -- retrospective/

# Skill-specific tags (format: {skill}/v{major}.{minor}.{patch})
git tag -l "retrospective/*"

# Update branches: cl-update/{skill}/{type}/{timestamp}
```

## Development Guidelines

When modifying skills:
- Follow existing frontmatter format (`name`, `description`)
- Update version tags when making releases
- Test changes locally before committing

Skill SKILL.md structure:
1. YAML frontmatter (name, description)
2. Purpose and when to invoke
3. Implementation details
4. Self-evolution boundaries (what can/cannot evolve)

## Core Flow

```
Task Completion
    ↓
[Retrospective] → Filters skills by skills_repo path → Pending Reflection
    ↓
User Approval
    ↓
[Update] → Applies changes → Staged for Approval
    ↓
User Approval → Git Commit + Tag
    ↓
[Digest] → Summarizes activity
```

All changes require user approval. No auto-merge.
