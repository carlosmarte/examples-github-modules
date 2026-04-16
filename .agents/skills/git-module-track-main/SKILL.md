---
name: git-module-track-main
description: This skill should be used when the user asks to "point all submodules to main", "track main branch for git modules", "switch submodules to main", "make submodules follow latest main", or wants every registered git submodule to track the `main` branch and advance to its latest commit.
version: 1.0.0
---

# Git Module Track Main

Reconfigure every registered git submodule to track the `main` branch and fast-forward to its latest commit.

## When This Skill Applies

- Standardizing all submodules to follow `main` after some were pinned to other branches/tags
- Migrating from `master` → `main` across submodules
- Catching up the entire dependency graph to the latest tip of `main`

## Arguments

- `<branch>` (optional, default `main`) — branch every submodule should track
- `<target-path>` (optional) — restrict to a single submodule; otherwise apply to all

## Prerequisites

- Inside a git repo with at least one submodule
- Each upstream submodule actually has a branch matching `<branch>` (verify in step 2)
- Working trees inside submodules are clean

## Steps

### 1. Enumerate submodules

```bash
git config --file .gitmodules --get-regexp 'submodule\..*\.path'
```

This yields one `submodule.<name>.path <path>` line per registered submodule.

### 2. Verify each upstream has the target branch

For each submodule path:

```bash
git -C <path> fetch origin
git -C <path> ls-remote --heads origin <branch>
```

If a submodule's upstream lacks the branch, surface it to the user before proceeding — do not silently skip or guess an alternative.

### 3. Update `.gitmodules` to track the branch

For each submodule:

```bash
git config -f .gitmodules submodule.<name>.branch <branch>
```

### 4. Propagate to `.git/config`

```bash
git submodule sync --recursive
```

### 5. Check out and fast-forward each submodule to the branch tip

For each submodule:

```bash
git -C <path> checkout <branch>
git -C <path> pull --ff-only origin <branch>
```

Use `--ff-only` so a divergent local history fails loudly instead of producing a merge commit.

### 6. Verify

```bash
git submodule status
git submodule foreach 'git rev-parse --abbrev-ref HEAD'
git diff --submodule=log
```

Confirm every submodule reports `<branch>` as its current ref and the parent repo shows the new pointers.

### 7. Stage parent-repo updates

```bash
git add .gitmodules <updated-submodule-paths>
git status
```

Do NOT commit unless the user explicitly asks.

## Failure Handling

- **`pull --ff-only` rejects**: a submodule has divergent local commits — stop and surface; do not force
- **Branch missing upstream**: report the submodule and its available branches; let the user decide
- **Detached HEAD after `update --remote`**: re-run step 5 explicitly to land on the branch ref, not just its commit

## Notes

- The combination of `.gitmodules` `branch =` setting + `git submodule update --remote` is what keeps submodules following `main` going forward — without it, a submodule re-pins to whatever commit was checked out
- For repos still on `master`, pass `<branch>` explicitly rather than assuming `main`
