# Auto Branch Creation Update

## Implementation Details

### Command Created
- **Path**: `.claude/commands/git/epic-branch.md`
- **Functionality**:
  - Branch creation: `git checkout -b epic/{epic_name}`
  - Remote push: `git push -u origin epic/{epic_name}`
  - PR creation: `gh pr create` with auto-generated title and body

### Key Features
- **Remote Tracking**: Uses `-u` flag for upstream setup
- **PR Automation**: Creates draft PR even without issues enabled
- **Hook Compatibility**: Follows existing git command patterns in `.claude/commands/git/`
- **Epic Naming**: Uses `epic/{name}` convention for branch names

### Usage Example
```
/pm:epic-branch{epic_name="my-new-epic"}
```

This will create `epic/my-new-epic` branch, push it, and open a PR against main.