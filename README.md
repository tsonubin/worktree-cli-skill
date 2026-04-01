# worktree-cli skill

A [Claude Code skill](https://skills.sh) for [@johnlindquist/worktree](https://github.com/johnlindquist/worktree-cli) (`wt`) -- a developer-friendly Git worktree management CLI.

## Install

```bash
npx skills add tsonubin/worktree-cli-skill
```

## What it does

Gives Claude Code knowledge of all `wt` commands, flags, workflows, and behavioral notes so it can help you:

- Create and manage Git worktrees (`wt new`, `wt setup`, `wt extract`)
- Review PRs in isolated worktrees (`wt pr`)
- Merge worktree branches back (`wt merge`)
- Clean up stale worktrees (`wt remove`, `wt purge`)
- Configure defaults (`wt config`)

## License

MIT
