---
name: worktree-cli
description: >
  Git worktree management with the `wt` CLI (@johnlindquist/worktree).
  Use when the user mentions worktrees, `wt`, wants to work on multiple branches simultaneously,
  review PRs in isolation, extract branches, merge worktree branches, or manage parallel development workflows.
  Triggers on: "worktree", "wt new", "wt pr", "wt setup", "wt merge", "wt extract", "wt open",
  "wt remove", "wt purge", "wt list", "wt config", "parallel branches", "isolate branch work".
---

# worktree-cli (`wt`)

A CLI tool that wraps Git worktrees with a developer-friendly interface. Binary: `wt`, package: `@johnlindquist/worktree`.

## Installation

```bash
pnpm install -g @johnlindquist/worktree
# or: npm install -g @johnlindquist/worktree
```

## Commands Reference

### `wt new [branchName]` -- Create a new worktree

Creates a worktree for the given branch and opens it in the configured editor.

| Flag | Description |
|------|-------------|
| `-p, --path <path>` | Custom worktree path |
| `-c, --checkout` | Create the branch if it doesn't exist |
| `-i, --install <pm>` | Run package manager install (`npm`/`pnpm`/`bun`/`yarn`) |
| `-e, --editor <editor>` | Override default editor |

**Branch name is required** -- unlike other commands, `wt new` has no interactive selector.

### `wt setup [branchName]` -- Create worktree + run setup scripts

Same as `wt new` but also runs setup scripts from `.cursor/worktrees.json` or `worktrees.json`.

| Flag | Description |
|------|-------------|
| `-t, --trust` | Skip command confirmation (for CI/automation) |

Setup script format (either file):
```json
["pnpm install", "pnpm db:migrate"]
```
or:
```json
{ "setup-worktree": ["pnpm install", "pnpm db:migrate"] }
```

The `$ROOT_WORKTREE_PATH` env var points to the main repo root inside setup scripts.

### `wt pr [prNumber]` -- Create worktree from a GitHub PR or GitLab MR

Fetches the PR ref directly without switching the current branch.

| Flag | Description |
|------|-------------|
| `-p, --path <path>` | Custom path |
| `-i, --install <pm>` | Package manager |
| `-e, --editor <editor>` | Override editor |
| `-s, --setup` | Run setup scripts after creation |
| `-t, --trust` | Trust setup scripts without confirmation |

Without a PR number, shows an interactive selector. Requires `gh` CLI (GitHub) or `glab` CLI (GitLab), or `GITHUB_TOKEN`/`GITLAB_TOKEN` env vars as fallback.

### `wt open [pathOrBranch]` -- Open existing worktree in editor

Without args: shows interactive fuzzy-searchable list.

| Flag | Description |
|------|-------------|
| `-e, --editor <editor>` | Override editor |

### `wt list` (alias: `ls`) -- List all worktrees with status

### `wt remove [pathOrBranch]` (alias: `rm`) -- Remove a worktree

Without args: interactive selection.

| Flag | Description |
|------|-------------|
| `-f, --force` | Force removal |

### `wt purge` -- Multi-select removal of worktrees (excludes main)

### `wt extract [branchName]` -- Extract current branch into a separate worktree

Moves branch work into an isolated worktree.

| Flag | Description |
|------|-------------|
| `-p, --path <path>` | Custom path |
| `-i, --install <pm>` | Package manager |
| `-e, --editor <editor>` | Override editor |

### `wt merge <branchName>` -- Merge a worktree branch into current branch

| Flag | Description |
|------|-------------|
| `--auto-commit` | Auto-commit uncommitted changes in target worktree first |
| `-m, --message <msg>` | Custom commit message (requires `--auto-commit`) |
| `--remove` | Remove source worktree after merge |
| `-f, --force` | Force removal (with `--remove`) |

**Safety defaults**: Does NOT auto-commit or auto-remove. Both require explicit flags.

### `wt config` -- Configuration management

```bash
wt config set editor <cursor|code|webstorm|none>
wt config get editor
wt config set provider <gh|glab>
wt config get provider
wt config set worktreepath <path>    # Global worktree directory
wt config get worktreepath
wt config clear worktreepath         # Reset to sibling directory behavior
wt config path                       # Show config file location
```

## Path Resolution

Priority order:
1. `--path` flag (used as-is)
2. `defaultWorktreePath` config + repo namespace (e.g., `~/worktrees/myrepo/feature-auth`)
3. Sibling directory (default): `../myrepo-feature-auth`

Branch names with slashes are sanitized to dashes (`feature/auth` -> `feature-auth`).

## Dirty State Handling

If the working directory has uncommitted changes, `wt` prompts: **Stash**, **Abort**, or **Continue anyway**. Stashing uses `git stash create` for a unique hash (avoids stash stack race conditions). SIGINT handlers attempt stash restoration on interruption.

Bare repos (`core.bare=true`) skip dirty-state checks.

## Important Behavior Notes

- **Editor "none"**: `wt config set editor none` skips opening any editor -- useful for CI or when running inside Claude Code
- **No package manager auto-detection**: You must pass `-i pnpm` (or npm/bun/yarn) explicitly
- **Setup scripts only run with `wt setup` or `wt pr --setup`**, never with `wt new`
- **Atomic rollback**: If dependency install fails after worktree creation, the worktree is automatically cleaned up
- **Pushing from a PR worktree updates the PR directly**
- **Remote detection**: Uses multi-strategy approach (main branch tracking > common remote names > first available), not hardcoded "origin"

## Common Workflows

### Review a PR in isolation
```bash
wt pr 42 -i pnpm
```

### Start a new feature branch
```bash
wt new feature/auth -c -i pnpm
```

### Extract messy WIP into its own worktree
```bash
wt extract my-experiment
```

### Merge worktree work back and clean up
```bash
wt merge feature/auth --auto-commit --remove
```

### Clean up stale worktrees
```bash
wt purge
```
