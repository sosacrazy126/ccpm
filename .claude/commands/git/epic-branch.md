---
name: epic-branch
description: Create and push epic branch to remote
usage: /git:epic-branch {name}
parameters:
  - name: name
    description: Epic name for branch (e.g., update-commit-to-remote-branch)
    required: true
    type: string
---

# /git:epic-branch Command

## Purpose
Creates a new epic branch with the format `epic/{name}` and pushes it to origin with upstream tracking.

## Implementation

```bash
#!/bin/bash

# Validate parameters
if [ -z "$1" ]; then
  echo "Error: Epic name is required"
  echo "Usage: /git:epic-branch <epic-name>"
  exit 1
fi

EPIC_NAME="$1"
BRANCH_NAME="epic/$EPIC_NAME"

# Check if branch already exists
if git show-ref --verify --quiet refs/heads/"$BRANCH_NAME"; then
  echo "Branch $BRANCH_NAME already exists locally"
  git checkout "$BRANCH_NAME"
  git push --set-upstream origin "$BRANCH_NAME" 2>/dev/null || echo "Branch already tracked remotely"
  exit 0
fi

# Check if remote branch exists
if git show-ref --verify --quiet refs/remotes/origin/"$BRANCH_NAME"; then
  echo "Remote branch $BRANCH_NAME exists, checking out locally"
  git checkout -b "$BRANCH_NAME" origin/"$BRANCH_NAME"
  exit 0
fi

# Create and switch to new branch
echo "Creating branch: $BRANCH_NAME"
git checkout -b "$BRANCH_NAME"

# Push to remote with upstream tracking
echo "Pushing to origin/$BRANCH_NAME"
git push -u origin "$BRANCH_NAME"

if [ $? -eq 0 ]; then
  echo "Successfully created and pushed branch: $BRANCH_NAME"
  echo "Tracking: origin/$BRANCH_NAME"
else
  echo "Failed to push branch. Cleaning up local branch."
  git checkout main
  git branch -D "$BRANCH_NAME"
  exit 1
fi
```

## Integration Notes
- Uses `git checkout -b` for local branch creation
- Uses `git push -u origin` for upstream tracking setup
- Includes validation for existing branches (local/remote)
- Provides cleanup on push failure
- Compatible with existing git hooks (pre-commit, pre-push)
- Works with current remote tracking configuration

## Hooks Integration
This command triggers standard git hooks:
- **pre-commit**: Code quality checks before any commits
- **pre-push**: Validation before pushing to remote
- **post-checkout**: Can be used for branch-specific setup

## Error Handling
- Validates epic name parameter
- Checks for existing local/remote branches
- Provides cleanup on push failures
- Maintains repository integrity
