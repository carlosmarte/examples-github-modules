---
name: git-module-pull-latest
description: This skill should be used when the user asks to "pull latest submodule", "update git modules", "fetch latest commits for submodules", "pull all submodules", or wants to advance one or all submodules to their latest upstream commit on the tracked branch.
version: 1.0.0
---

# Git Module Pull Latest

Advance one or all git submodules to the latest commit on their tracked remote branch.

## When This Skill Applies

- Refreshing a single submodule to its newest upstream commit
- Bulk-updating every submodule in the repo
- Resolving "submodule is behind" warnings after upstream changes

## Arguments

- `<target-path>` (optional) — specific submodule path to update; if omitted, update **all** submodules
- `--rebase` (optional) — rebase local changes inside the submodule on top of upstream

## Prerequisites

- Repo has at least one submodule registered in `.gitmodules`
- Network access to each submodule's remote
- Working tree of each submodule is clean (or user accepts `--rebase`)

## Steps

### 1. Inspect current state

```bash
git submodule status
```

Note any submodules marked with `+` (modified) or `-` (uninitialized).

### 2. Update

For a single submodule:

```bash
git submodule update --remote --merge <target-path>
```

For all submodules:

```bash
git submodule update --remote --merge --recursive
```

Use `--rebase` instead of `--merge` only if the user explicitly requests it.

### 3. Verify the new pinned commit

```bash
git submodule status
git diff --submodule=log
```

`git diff --submodule=log` displays the upstream commits each submodule advanced through — share this with the user.

### 4. Stage parent-repo pointer updates

```bash
git add <updated-submodule-paths>
git status
```

Do NOT commit unless the user explicitly asks.

## Failure Handling

- **Detached HEAD warnings**: expected — submodules check out a specific commit, not a branch
- **Merge conflicts inside a submodule**: stop and surface the conflict; do not auto-resolve
- **Network failures**: report which submodule failed and continue with the rest only if the user agrees

## Notes

- `--remote` is required to fetch new upstream commits; without it `update` only restores the pinned commit
- The branch tracked is whatever is set in `.gitmodules` (`submodule.<path>.branch`); see `git-module-track-main` to change it
