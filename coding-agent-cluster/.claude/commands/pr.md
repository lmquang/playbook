# Push working state to a PR

Our goal is to take the current state of our git diff (ALL The files), commit the files (with a reasonable commit message), push them (to a branch), open a PR,and surface the link to the PR back to me so I can click it.

## Workflow Details

For detailed git workflow information including branch naming conventions and recovery procedures, see [docs/GIT_WORKFLOW.md](../../docs/GIT_WORKFLOW.md).

## Quick Reference

### If on wrong branch

- **On main**: `git reset --soft` to origin/main, stash, create feature branch, pop stash
- **On unrelated branch**: Same process - reset to origin, stash, checkout main, pull, create new branch

### Pre-PR checklist

  1. Run `pnpm run lint`, `pnpm run lint:fix`, and analyze if any fixes are needed
  2. Run tests with `pnpm run test` if changes might impact them
  3. Then check for other rules in cicd pipeline to make sure everything is ready for PR

### Post-PR

- **Regular repo**: Checkout main and pull latest (while showing PR URL)
- **Git worktree**: Leave as-is (user will delete when PR is merged)
