# /pm:epic-sync Command

## Description
Creates a GitHub epic issue and sub-issues, then syncs to git branch with commit and push.

## Usage
```
/pm:epic-sync{epic_title="Title", description="Description", tasks="Task1\nTask2"}
```

## Implementation Logic
1. Parse epic_title, description, tasks from parameters
2. Use `gh issue create` to create epic issue
3. For each task, create sub-issue with `gh issue create --label epic:task`
4. Create branch: `git checkout -b feature/$(slugify epic_title)`
5. Commit staged changes: `git commit -m "feat: $(epic_title)"`
6. Push: `git push -u origin HEAD`
7. Log issue IDs in context bundle

## Validation
- Ensure gh CLI authenticated
- Check git remote origin exists
- Verify staged changes present