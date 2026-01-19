---
name: digest
description: "Daily summary generator for continual learning. Consolidates skill evolution activity including applied updates, pending approvals, and system health. Invoke daily or on-demand."
---

# Digest

Generates human-readable summaries of skill evolution activity. Consolidates changes, pending approvals, and system health into periodic reports.

## When to Invoke

- Scheduled interval (default: daily)
- Explicit user request ("show digest", "what changed")
- Pending approvals accumulate

## Data Sources

```yaml
sources:
  - retrospective/pending/    # Pending reflections
  - update/staged/            # Pending updates
  - git log -- {skill}/       # Path-scoped history (see update skill)
```

## Report Format

```markdown
# Skill Evolution Digest — {date}

**Updates:** {n} applied, {n} pending, {n} rejected
**Skills touched:** {skill} ({n}), {skill} ({n})

## Pending Approvals

- ⏳ `{skill}` — {description} (since {date})

## Action Items

- [ ] {action}

---
Details: `git log --oneline --since="{last_digest_date}"`
```

Keep it brief — details live in git history.

## Output

Save to `digest/logs/{YYYY-MM-DD}.md`

Track state in `digest/state.yaml`:
```yaml
last_digest:
  date: "2025-01-06"
  path: "digest/logs/2025-01-06.md"
next_scheduled: "2025-01-07"
```

## Example

```markdown
# Skill Evolution Digest — January 6, 2025

**Updates:** 3 applied, 1 pending, 1 rejected
**Skills touched:** retrospective (2), update (1)

## Pending Approvals

- ⏳ `scylladb-debugging` — Add query profiling (since 2025-01-05)

## Action Items

- [ ] Review pending: scylladb-debugging enhancement
- [ ] Consider: profiling skill (requested 3x)

---
Details: `git log --oneline --since="2025-01-05"`
```

## Self-Evolution

This skill can evolve:
- Report format
- Aggregation logic
- Pattern summaries

Cannot evolve:
- Data source locations
- Core metrics
