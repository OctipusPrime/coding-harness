---
name: my-review
description: Instructions for how to perform code review
---

## Instructions

Please analyze the changes in this $ARGUMENTS and focus on identifying issues related to:

- Potential bugs or issues
- Performance
- Security
- Correctness
- Code duplication
- Overcomplicated solutions
- README was updated with necessary information, not everything has to be in README but important new features, settings and code bases changes should be reflected.

If critical issues are found, list them in a few short bullet points. If no critical issues are found, provide a simple approval.
Sign off with a checkbox emoji: (approved) or (issues found).

Keep your response concise. Only highlight critical issues that must be addressed before merging. Skip detailed style or minor suggestions unless they impact performance, security, or correctness.

## Changes

**Unstaged changes:**
!`git diff`

**Staged changes:**
!`git diff --staged`

**Last commit:**

!`git show HEAD`
