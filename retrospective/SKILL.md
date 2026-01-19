---
name: retrospective
description: "Post-session reflection and analysis for continual learning. Invoke after task completion to analyze outcomes for skills in skills_repo, interpret user feedback, identify root causes of failures, and propose skill improvements."
---

# Retrospective

Post-execution analysis that generates structured feedback about what worked, what failed, and why. Transforms raw outcomes into actionable improvement signals for the update skill.

## When to Invoke

After any task where skills from `skills_repo` were used. Only skills inside `skills_repo` are analyzed for evolution.

## Input: Session Context

```yaml
session_context:
  session_id: "uuid-or-timestamp"
  timestamp: "ISO-8601"

  user_feedback:
    explicit:
      rating: -1 | 0 | 1
      comment: "optional"
    implicit:
      accepted_output: boolean
      manual_corrections: number
      follow_up_questions: number

  used_skills:
    - name: "skill-name"
      path: "/path/to/skill"  # Used for filtering
      version: "x.y.z"
      outcomes:
        - step: 1
          success: true
          notes: "..."

  task:
    original_request: "..."
    domain_signals: ["tag1", "tag2"]

  approach:
    initial_plan: "..."
    alternatives_considered: [...]

  outcome:
    status: "success" | "partial_success" | "failure"
    artifacts: [...]
    errors: [...]
    capability_gaps: ["skills that would have helped"]

  # From recent digest logs and git history
  # Use path-scoped git queries (see update skill for commands)
  digest_context:
    recent_updates:        # git log --since="7 days ago" -- {skill}/
      - skill: "skill-name"
        date: "ISO-8601"
        change: "summary"
        success: true | false
    recent_rejections:
      - proposal: "prop-id"
        skill: "skill-name"
        reason: "why rejected"
        date: "ISO-8601"
    churn_signals:         # Skills with >2 updates in 7d
      - skill: "skill-name"
        updates_7d: 3      # git log --since="7 days ago" --oneline -- {skill}/ | wc -l
```

## Process

### Step 1: Filter to Evolvable Skills

```python
# Read config from ~/.claude/continual-learning.json
config = load_json("~/.claude/continual-learning.json")

# Only analyze skills inside skills_repo
skills_repo = expand_path(config.skills_repo)
skills_to_analyze = [s for s in used_skills if s.path.startswith(skills_repo)]

# Note non-evolvable skills for reporting, but don't propose changes
non_evolvable = [s for s in used_skills if not s.path.startswith(skills_repo)]
```

### Step 2: Outcome Assessment

For each evolvable skill:
- Success rate across invocations
- Value delivered vs expectations
- Execution quality

### Step 3: Feedback Interpretation

Analyze user signals:
- Explicit rating and comments
- Implicit signals (corrections, follow-ups, acceptance)
- Infer satisfaction and issues

### Step 4: Root Cause Analysis

For failures or partial successes:
- Identify what went wrong
- Categorize (code_quality, process_gap, capability_gap, etc.)
- Determine confidence in diagnosis

### Step 5: Generate Proposals

For each identified improvement:
```yaml
proposal:
  id: "prop-001"
  skill: "skill-name"
  type: "enhancement" | "bugfix"  # NOT new_capability
  description: "What to improve"
  implementation_hints: [...]
  confidence: 0.0-1.0
  breaking: boolean
  version_bump: "major" | "minor" | "patch"
```

**Note:** Retrospective cannot propose new skill creation. It can note capability gaps in the reflection, but only the user can initiate new skill creation ("create a skill for X").

### Step 6: Save to Pending

All reflections are saved to `retrospective/pending/` for user review. No auto-approval.

## Output: Reflection Note

Save to `retrospective/pending/{session_id}.md`.

### Reflection Lifecycle

```
pending/    → Awaiting user review
    ↓
User approves proposals
    ↓
archive/    → All proposals processed
```

After all proposals in a reflection are processed:
1. Update the Approval section with decisions
2. Move from `pending/` to `archive/`

```bash
mv retrospective/pending/{session_id}.md retrospective/archive/
```

## Pattern Detection

Track across sessions using `digest_context`:

| Pattern | Condition | Action |
|---------|-----------|--------|
| Repeated failure | Same error 3+ times | Elevate confidence, prioritize |
| Capability gap | Same gap 3+ sessions | Note in reflection (user decides) |
| Success pattern | Approach works consistently | Document or promote |
| Regression | Performance dropped after update | Flag for rollback |
| **Re-proposal** | Same issue rejected within 7d | Block or require justification |
| **Churn** | >2 updates to same skill in 7d | Suggest stabilization period |
| **Oscillation** | Change reverts recent change | Block, require explicit override |
| **Failed retry** | Same fix attempted and failed before | Do not propose again |

### Example: Failed Retry Detection

```yaml
digest_context:
  recent_updates:
    - skill: "scylladb-debugging"
      date: "2025-01-05"
      change: "Add retry logic for timeouts"
      success: false  # User reported it didn't help
```

If retrospective is about to propose "Add retry logic for timeouts" again:

```
⛔ BLOCKED: Tried this update last time but still didn't work, don't try again.

Previous attempt: 2025-01-05
Result: Failed — did not resolve the issue

Consider alternative approaches or mark as known limitation.
```

## Example

### Input: Session Context

```yaml
session_context:
  session_id: "scylladb-debug-20250106"
  timestamp: "2025-01-06T10:30:00Z"

  user_feedback:
    explicit:
      rating: 1
      comment: "worked but could be faster"
    implicit:
      accepted_output: true
      manual_corrections: 1
      follow_up_questions: 0

  used_skills:
    - name: "scylladb-debugging"
      path: "~/Workspace/continual-learning/scylladb-debugging"
      version: "1.2.0"
      outcomes:
        - step: 1
          success: true
          notes: "Identified partition hotspot"
        - step: 2
          success: false
          notes: "Suggested fix had syntax error"

  task:
    original_request: "Debug why my ScyllaDB queries are timing out"
    domain_signals: ["scylladb", "performance", "timeout"]

  outcome:
    status: "partial_success"
    artifacts: ["query-analysis.md"]
    errors: ["CQL syntax error in fix script"]
    capability_gaps: ["profiling"]
```

### Output: Reflection Note (excerpt)

```markdown
# Reflection: scylladb-debug-20250106

**Status:** pending
**Overall Success:** 70%

## What Worked
- Successfully identified partition hotspot as root cause
- User accepted the analysis

## What Failed
- Fix script had CQL syntax error
- User had to manually correct

## Root Cause Analysis
### CQL Syntax Error
- **Category:** code_quality
- **Skill:** scylladb-debugging
- **Confidence:** 95%

## Proposed Improvements
### [prop-001] Add CQL syntax validation
- **Skill:** scylladb-debugging
- **Type:** enhancement
- **Description:** Validate CQL syntax before suggesting fix scripts
- **Confidence:** 85%
- **Version Bump:** minor
```

## Self-Evolution

This skill can evolve:
- Analysis heuristics
- Feedback interpretation
- Confidence scoring
- Pattern detection

Cannot evolve:
- Path-based filtering logic (settings-controlled)
- Core input/output schemas
