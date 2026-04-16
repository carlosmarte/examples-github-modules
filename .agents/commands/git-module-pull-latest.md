# Git Module Pull Latest

Advance one or all git submodules to the latest commit on their tracked remote branch.

## Arguments

$ARGUMENTS

<!-- Argument format: [target-path] [--rebase] -->
<!-- Argument 1 (optional) — specific submodule path to update; if omitted, update ALL submodules -->
<!-- Argument 2 (optional) — `--rebase` to rebase local changes on top of upstream instead of merging -->

## Prerequisites

- Repo has at least one submodule registered in `.gitmodules`
- Network access to each submodule's remote
- Working tree of each submodule is clean (or user accepts `--rebase`)

## Instructions

### Step 1: Inspect current state

```bash
git submodule status
```

Note any submodules marked with `+` (modified) or `-` (uninitialized).

### Step 2: Update

For a single submodule:

```bash
git submodule update --remote --merge <target-path>
```

For all submodules:

```bash
git submodule update --remote --merge --recursive
```

Use `--rebase` instead of `--merge` only if the user explicitly passed it.

### Step 3: Verify the new pinned commit

```bash
git submodule status
git diff --submodule=log
```

`git diff --submodule=log` shows the upstream commits each submodule advanced through — share this with the user.

### Step 4: Stage parent-repo pointer updates

```bash
git add <updated-submodule-paths>
git status
```

Do NOT commit unless the user explicitly asks.

## Equivalent Make Target

```bash
make -f Makefile.git-module-repo pull-latest [TARGET_PATH=<path>]
```

## Failure Handling

- **Detached HEAD warnings**: expected — submodules check out a specific commit, not a branch
- **Merge conflicts inside a submodule**: stop and surface the conflict; do not auto-resolve
- **Network failures**: report which submodule failed and continue with the rest only if the user agrees

## Notes

- `--remote` is required to fetch new upstream commits; without it `update` only restores the pinned commit
- The branch tracked is whatever is set in `.gitmodules` (`submodule.<path>.branch`); see `/git-module-track-main` to change it
