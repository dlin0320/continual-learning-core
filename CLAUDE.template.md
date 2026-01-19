# Continual Learning

This system enables skills to evolve based on session outcomes and user feedback. Three core skills work together: retrospective, update, and digest.

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
[Digest] → Summarizes changes for human review
```

## Evolvable Skills

Only skills inside the `skills_repo` directory are evolvable. The retrospective skill filters by path:

```python
skills_repo = settings.continual_learning.skills_repo
skills_to_analyze = [s for s in used_skills if s.path.startswith(skills_repo)]
```

To add a new evolvable skill, place it in the `skills_repo` directory.

## After Task Completion

When a task used skills from `skills_repo`:

1. Invoke retrospective with session context:
   - User feedback (explicit + implicit)
   - Skills used and their outcomes
   - Task context and approach
   - Errors and capability gaps
2. Retrospective creates pending reflection
3. User reviews and approves reflection
4. Update applies changes and stages for approval
5. User reviews and approves update
6. Changes are committed and tagged
7. Changes appear in next digest

## Configuration

Settings are in `~/.claude/continual-learning.json`:

```json
{
  "skills_repo": "~/Workspace/continual-learning",
  "principles": {
    "safety_boundaries": [...],
    "quality_thresholds": {...},
    "governance": {...}
  },
  "digest": {
    "interval": "daily",
    "notify": true
  }
}
```

**skills_repo** = Path to the directory containing evolvable skills. Only skills in this directory can be evolved.

## Creating New Skills

To add a skill to the continual learning system:

1. Create the skill in the `skills_repo` directory
2. The skill is now evolvable (retrospective will analyze it)

To make a skill non-evolvable:
- Move it outside the `skills_repo` directory

## Principles (Immutable)

These constraints cannot be overridden:

1. Never evolve skills outside `skills_repo`
2. Major version updates always require human approval
3. Never propose new skill creation — only user can initiate ("create a skill for X")
4. All reflections and updates require user approval
