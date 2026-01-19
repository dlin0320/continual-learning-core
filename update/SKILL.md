---
name: update
description: "Skill evolution engine for continual learning. Applies approved improvements from retrospective, handles git versioning (branch, commit, tag), validates changes, and supports rollback."
---

# Update

The evolution engine that takes retrospective proposals and produces improved skill versions. Handles git versioning, validation, and staged rollouts.

## Dependencies

- **git**: Required for versioning, branching, and rollback
  - Minimum version: 2.x
  - Each skill's git repo is auto-detected

- **skills_repo**: Path to the directory containing evolvable skills
  - Configured in `~/.claude/continual-learning.json` as `skills_repo`
  - Skills may be in nested git repos (e.g., `core/` has its own repo)
  - Git operations run from each skill's detected git root

### Without Git

If git is unavailable:
- Changes are still applied to files
- No branching or rollback capability
- Version tracking falls back to timestamp comments in files
- **Recommendation:** Initialize git before using continual learning

## When to Invoke

Triggered when:
- User approves a proposal from retrospective
- User explicitly requests an update

## Input: Approved Proposal

```yaml
proposal:
  id: "prop-001"
  session_id: "abc-123"
  skill: "skill-name"
  type: "enhancement" | "bugfix"
  description: "What to improve"
  implementation_hints:
    - "hint 1"
    - "hint 2"
  confidence: 0.85
  breaking: false
  version_bump: "minor"
```

**Note:** `new_capability` (new skill creation) is not permitted via continual learning. Only user can initiate new skills.

## Process

### Step 1: Locate Skill and Detect Git Root

1. Read `skills_repo` from `~/.claude/continual-learning.json`
2. Expand path (e.g., `~` → `/Users/user`)
3. Construct skill path: `{skills_repo}/{skill}`
4. Verify `{skill_path}/SKILL.md` exists
5. **Detect git root** by walking up from skill path:

```python
def find_git_root(skill_path):
    """Walk up from skill to find its git repo"""
    current = skill_path
    while current != '/':
        if os.path.exists(os.path.join(current, '.git')):
            return current
        current = os.path.dirname(current)
    return None

# Examples:
# skills_repo: ~/Workspace/continual-learning
#
# core/retrospective → git_root: ~/Workspace/continual-learning/core
# core/update        → git_root: ~/Workspace/continual-learning/core
# ccws               → git_root: ~/Workspace/continual-learning
```

6. Read the skill's SKILL.md to understand current state
7. Calculate `relative_skill_path` from git root to skill

All subsequent git operations use the detected `git_root` as the working directory.

### Step 2: Generate Changes

Based on proposal type:

| Type | Strategy |
|------|----------|
| bugfix | Targeted fix to specific issue |
| enhancement | Add capability or improve existing |

### Step 3: Validate Changes

```yaml
validation:
  - syntax: Markdown/YAML/code valid
  - structure: Required sections present
  - diff_size: Reasonable change size (<30% for minor)
  - confidence: Meets threshold from settings
```

### Step 4: Git Operations and Stage

Git commands run from the **detected git root** (not necessarily `skills_repo`):

```bash
cd {git_root}

# Ensure clean state
git status --porcelain || abort

# Create branch (skill name in branch)
git checkout -b cl-update/{skill}/{type}/{timestamp}

# Apply changes to skill folder only
# ... modifications to {relative_skill_path}/ ...

# Commit with structured message
git add {relative_skill_path}/
git commit -m "[continual-learning] {skill}: {summary}

Proposal: {id}
Session: {session_id}
Type: {type}
Breaking: {breaking}
Confidence: {confidence}%"

# Stage for approval (do not auto-merge)
# Save staged info to {skills_repo}/update/staged/{proposal_id}.yaml
```

### Path-Scoped Git Commands

Run from the skill's git root:

```bash
cd {git_root}

# History for specific skill
git log --oneline -- {relative_skill_path}/

# Recent changes (last 7 days)
git log --since="7 days ago" --oneline -- {relative_skill_path}/

# Diff for skill only
git diff HEAD~5 -- {relative_skill_path}/

# Tags for specific skill (using {skill}/v{x.y.z} convention)
git tag -l "{skill}/*"
```

### Step 5: Version Tagging

After user approves and merge completes:

```
{skill-name}/v{major}.{minor}.{patch}
```

Bump rules:
- **patch:** Bugfixes, typos, clarifications
- **minor:** New capabilities, enhancements (non-breaking)
- **major:** Breaking changes, significant restructuring

## Staged Changes

All changes are saved to `{skills_repo}/update/staged/{proposal_id}.yaml` for user approval:

```yaml
proposal_id: "prop-001"
skill: "skill-name"
git_root: "/Users/user/Workspace/continual-learning/core"
relative_skill_path: "retrospective"
branch: "cl-update/retrospective/enhancement/20250106"
status: "pending_approval"

changes:
  - file: "retrospective/SKILL.md"
    diff_summary: "+15 lines, -3 lines"

validation_results:
  all_passed: true

merge_instructions: |
  cd {git_root}
  git diff main..{branch}  # Review
  git checkout main && git merge {branch}
  git tag {skill}/v{version}
```

## Approval and Merge

When user approves:

```bash
cd {git_root}  # From staged yaml

# Review the diff
git diff main..{branch}

# Merge
git checkout main && git merge {branch}

# Tag the new version
git tag {skill}/v{version}

# Clean up staged file
rm {skills_repo}/update/staged/{proposal_id}.yaml
```

## Rollback

When regression detected:

```bash
cd {git_root}

git checkout -b cl-rollback/{skill}/{timestamp}
git revert {commit}  # or checkout previous tag
git commit -m "[continual-learning] {skill}: Rollback to v{version}

Reason: {regression_description}"
git checkout main && git merge cl-rollback/{skill}/{timestamp}
git tag {skill}/v{version}-rollback
```

## Output

```yaml
update_result:
  proposal_id: "prop-001"
  status: "staged" | "failed"
  skill: "skill-name"
  git_root: "/path/to/git/root"
  relative_skill_path: "path/from/git_root"
  previous_version: "1.2.0"
  new_version: "1.3.0"
  git:
    branch: "cl-update/..."
    commit: "abc123"
    staged: true
```

## Example: Core Skill (Nested Repo)

### Input: Approved Proposal

```yaml
proposal:
  id: "prop-001"
  session_id: "debug-20250106"
  skill: "retrospective"
  type: "enhancement"
  description: "Improve pattern detection for repeated failures"
  implementation_hints:
    - "Add sliding window for failure tracking"
  confidence: 0.85
  breaking: false
  version_bump: "minor"
```

### Process Execution

```bash
# Step 1: Locate skill and detect git root
# skills_repo: ~/Workspace/continual-learning
# skill_path: ~/Workspace/continual-learning/core/retrospective
# git_root: ~/Workspace/continual-learning/core  (found .git here)
# relative_skill_path: retrospective

cd ~/Workspace/continual-learning/core

# Step 2-3: Generate and validate changes
# ...

# Step 4: Git operations (in core/ repo)
git checkout -b cl-update/retrospective/enhancement/20250106
# ... apply changes ...
git add retrospective/SKILL.md
git commit -m "[continual-learning] retrospective: Improve pattern detection

Proposal: prop-001
Session: debug-20250106
Type: enhancement
Breaking: no
Confidence: 85%"

# Save to ~/Workspace/continual-learning/update/staged/prop-001.yaml
```

### Output: Update Result

```yaml
update_result:
  proposal_id: "prop-001"
  status: "staged"
  skill: "retrospective"
  git_root: "/Users/user/Workspace/continual-learning/core"
  relative_skill_path: "retrospective"
  previous_version: "1.2.0"
  new_version: "1.3.0"
  git:
    branch: "cl-update/retrospective/enhancement/20250106"
    commit: "a1b2c3d"
    staged: true
```

## Example: Private Skill (Parent Repo)

### Input: Approved Proposal

```yaml
proposal:
  id: "prop-002"
  skill: "ccws"
  type: "bugfix"
  # ...
```

### Process Execution

```bash
# Step 1: Locate skill and detect git root
# skills_repo: ~/Workspace/continual-learning
# skill_path: ~/Workspace/continual-learning/ccws
# git_root: ~/Workspace/continual-learning  (found .git here, not in ccws/)
# relative_skill_path: ccws

cd ~/Workspace/continual-learning

# Step 4: Git operations (in parent repo)
git checkout -b cl-update/ccws/bugfix/20250106
git add ccws/SKILL.md
git commit -m "[continual-learning] ccws: Fix task cleanup

Proposal: prop-002
..."
```

## Self-Evolution

This skill can evolve:
- Validation pipeline
- Change generation heuristics
- Diff analysis

Cannot evolve:
- Git root detection logic
- Git workflow structure
- Version bump rules
