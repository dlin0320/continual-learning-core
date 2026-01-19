# Reflection: {{session_id}}

**Date:** {{timestamp}}
**Status:** {{status}}

## Session Summary

{{task_summary}}

## Skills Used

| Skill | Evolvable | Success Rate | Notes |
|-------|-----------|--------------|-------|
{{#each used_skills}}
| {{name}} | {{#if evolvable}}✅{{else}}❌{{/if}} | {{success_rate}}% | {{notes}} |
{{/each}}

## What Worked

{{#each successes}}
- {{this}}
{{/each}}

## What Failed

{{#each failures}}
- {{this}}
{{/each}}

## Root Cause Analysis

{{#each root_causes}}
### {{issue}}

- **Category:** {{category}}
- **Skill:** {{skill}}
- **Confidence:** {{confidence}}%

{{/each}}

## Proposed Improvements

{{#each proposals}}
### [{{id}}] {{description}}

- **Skill:** {{skill}}
- **Type:** {{type}}
- **Confidence:** {{confidence}}%
- **Breaking:** {{breaking}}
- **Version Bump:** {{version_bump}}

{{/each}}

## Non-Evolvable Skills Used

{{#each non_evolvable_skills}}
- {{name}}: Used but outside skills_repo (not analyzed)
{{/each}}

## User Feedback

- **Rating:** {{feedback.rating}}
- **Comment:** "{{feedback.comment}}"
- **Corrections:** {{feedback.corrections}}

---

## Approval

- [ ] Approved
- [ ] Rejected

**Reviewer:**
**Notes:**
