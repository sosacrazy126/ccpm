---
name: epic-branch
description: PM integration for epic branch creation with GitHub issue linking
usage: /pm:epic-branch {name} {description?}
parameters:
  - name: name
    description: Epic name for branch and GitHub issue
    required: true
    type: string
  - name: description
    description: Optional epic description for GitHub issue
    required: false
    type: string
---

# /pm:epic-branch Command

## Purpose
Creates an epic branch and corresponding GitHub issue, integrating project management with git workflow.

## Implementation

```bash
#!/bin/bash

# Validate parameters
if [ -z "$1" ]; then
  echo "Error: Epic name is required"
  echo "Usage: /pm:epic-branch <epic-name> [description]"
  exit 1
fi

EPIC_NAME="$1"
DESCRIPTION="${2:-Auto-generated epic for $EPIC_NAME}"
BRANCH_NAME="epic/$EPIC_NAME"

echo "Creating epic: $EPIC_NAME"
echo "Branch: $BRANCH_NAME"
echo "Description: $DESCRIPTION"

# Create GitHub issue first
ISSUE_TITLE="Epic: $EPIC_NAME"
ISSUE_BODY="## Epic Overview

**Branch:** $BRANCH_NAME
**Status:** In Progress
**Created:** $(date -Iseconds)

### Description
$DESCRIPTION

### Tasks
- [ ] Branch creation and setup
- [ ] Implementation phases
- [ ] Testing and validation
- [ ] Documentation updates
- [ ] Merge to main"

echo "Creating GitHub issue: $ISSUE_TITLE"
ISSUE_NUMBER=$(gh issue create --title "$ISSUE_TITLE" --body "$ISSUE_BODY" --json number --jq '.[0].number' 2>/dev/null)

if [ $? -ne 0 ] || [ -z "$ISSUE_NUMBER" ]; then
  echo "Failed to create GitHub issue. Continuing with branch creation only."
  ISSUE_NUMBER="unknown"
else
  echo "Created GitHub issue #$ISSUE_NUMBER"
fi

# Create and push branch using git:epic-branch
echo "Creating branch $BRANCH_NAME"
bash /home/evilbastardxd/Desktop/tools/notes/workspace/ccpm/.claude/commands/git/epic-branch.md "$EPIC_NAME"

if [ $? -eq 0 ]; then
  # Link issue to branch
  if [ "$ISSUE_NUMBER" != "unknown" ]; then
    echo "#$ISSUE_NUMBER" >> .claude/context/linked-issues.md 2>/dev/null || touch .claude/context/linked-issues.md && echo "#$ISSUE_NUMBER" >> .claude/context/linked-issues.md
    gh issue comment "$ISSUE_NUMBER" --body "Created branch: $BRANCH_NAME

Next steps:
- Switch to branch: \`git checkout $BRANCH_NAME\`
- Start implementation
- Update issue with progress" 2>/dev/null || echo "Could not add branch comment to issue"
  fi
  
  echo "Epic setup complete!"
  echo "Branch: $BRANCH_NAME"
  echo "Issue: #$ISSUE_NUMBER"
  echo "Description: $DESCRIPTION"
else
  echo "Branch creation failed. Cleaning up GitHub issue if created."
  if [ "$ISSUE_NUMBER" != "unknown" ]; then
    gh issue close "$ISSUE_NUMBER" --comment "Epic cancelled due to branch creation failure" 2>/dev/null
  fi
  exit 1
fi
```

## Integration Features
- Creates GitHub issue with standardized epic template
- Links issue to branch creation via git:epic-branch command
- Maintains context linking in .claude/context/linked-issues.md
- Provides cleanup on failure (closes issue if branch creation fails)
- Compatible with existing PM workflow and hooks

## Dependencies
- **gh CLI**: For GitHub issue creation and management
- **git:epic-branch**: Underlying git operation command
- **Existing hooks**: Pre-commit, pre-push validation

## Error Recovery
- Continues branch creation if issue creation fails
- Closes GitHub issue if branch creation fails
- Maintains audit trail in context files
- Provides clear status messages for manual recovery
