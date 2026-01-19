# Continual Learning

A self-evolving skill system for Claude. Skills improve through reflection on session outcomes and user feedback.

## Overview

Three core skills form a learning loop:

| Skill | Purpose |
|-------|---------|
| **Retrospective** | Analyzes task outcomes, proposes improvements |
| **Update** | Applies approved changes, handles git versioning |
| **Digest** | Summarizes activity for human review |

## How It Works

```
Task Completion
    ↓
[Retrospective] → Analyzes skills in skills_repo
    ↓
Pending Reflection (awaits user approval)
    ↓
[Update] → Applies approved changes
    ↓
Pending Update (awaits user approval)
    ↓
Git Commit + Tag
    ↓
[Digest] → Summarizes for human
```

All reflections and updates require user approval before being applied.

## Evolvable Skills

Only skills inside the `skills_repo` directory are evolvable. To add a new evolvable skill, place it in the `skills_repo` directory. To make a skill non-evolvable, move it outside.

## Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/dlin0320/continual-learning-core.git ~/Workspace/continual-learning-core
cd ~/Workspace/continual-learning-core
```

### 2. Symlink Skills

Create symlinks from `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
ln -s ~/Workspace/continual-learning-core/retrospective ~/.claude/skills/retrospective
ln -s ~/Workspace/continual-learning-core/update ~/.claude/skills/update
ln -s ~/Workspace/continual-learning-core/digest ~/.claude/skills/digest
```

### 3. Add Configuration

Copy `settings.template.json` to `~/.claude/continual-learning.json` (adjust `skills_repo` path as needed):

```bash
cp settings.template.json ~/.claude/continual-learning.json
```

Or create `~/.claude/continual-learning.json` with:

```json
{
  "skills_repo": "~/Workspace/continual-learning-core",
  "principles": {
    "safety_boundaries": [
      "Never evolve skills outside skills_repo"
    ]
  },
  "digest": {
    "interval": "daily",
    "notify": true
  }
}
```

### 4. Add CLAUDE.md Instructions

Append `CLAUDE.template.md` to `~/.claude/CLAUDE.md`:

```bash
cat CLAUDE.template.md >> ~/.claude/CLAUDE.md
```

### 5. Git Operations

This repo uses path-scoped git operations for skill-specific history:

```bash
git log --oneline -- retrospective/     # History for one skill
git tag -l "retrospective/*"            # Tags for one skill
```

## File Structure

```
continual-learning-core/       # ← This repo
├── CLAUDE.template.md         # Template to append to ~/.claude/CLAUDE.md
├── README.md
├── settings.template.json     # Reference config
├── retrospective/             # Post-session analysis skill
├── update/                    # Skill evolution engine
└── digest/                    # Daily summary generator

~/.claude/
├── CLAUDE.md              # Instructions (includes CLAUDE.template.md content)
├── continual-learning.json # Continual learning configuration
└── skills/                # Symlinks to this repo
    ├── retrospective -> ~/Workspace/continual-learning-core/retrospective
    ├── update -> ~/Workspace/continual-learning-core/update
    └── digest -> ~/Workspace/continual-learning-core/digest
```

## Adding Your Own Skills

Place additional skills in the same directory to make them evolvable:

```bash
# Clone your skill into the skills_repo
git clone <your-skill-repo> ~/Workspace/continual-learning-core/my-skill

# Symlink it
ln -s ~/Workspace/continual-learning-core/my-skill ~/.claude/skills/my-skill
```

## Safety

The model cannot:
- Evolve skills outside `skills_repo`
- Merge updates without user approval
- Propose new skill creation (user must request)

The model can:
- Modify `settings.json`
- Propose improvements to skills in `skills_repo`
- Note capability gaps (but not auto-propose new skills)
