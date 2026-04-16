---
name: git-module-init
description: This skill should be used when the user asks to "initialize a git submodule", "add a git module", "setup a new submodule", "register a repo as submodule", or wants to add a GitHub repository as a git submodule at a specified path within the project.
version: 1.0.0
---

# Git Module Init

Initial setup of a GitHub repository as a git submodule at a specified path.

## When This Skill Applies

- Adding a new external repository as a submodule for the first time
- Bootstrapping a project that needs to consume another repo's code
- Wiring `.gitmodules` and `.git/config` to track an external dependency

## Arguments

- `<repo-url>` — full GitHub URL (e.g., `https://github.com/org/repo`)
- `<target-path>` — relative directory to clone into (e.g., `./vendor/repo`)
- `<branch>` (optional) — branch to track; defaults to `main`

If any required argument is missing, ask the user before proceeding.

## Prerequisites

- Current directory is inside a git repository (`git rev-parse --is-inside-work-tree`)
- `<target-path>` does not already exist or conflict with a tracked path
- Network access to the remote

## Steps

### 1. Validate

```bash
git rev-parse --is-inside-work-tree
test ! -e <target-path> || echo "PATH EXISTS — abort or use --force"
```

### 2. Add submodule (tracking the requested branch)

```bash
git submodule add -b <branch> <repo-url> <target-path>
```

If no branch was provided, omit `-b <branch>` (defaults to remote HEAD), then pin tracking via:

```bash
git config -f .gitmodules submodule.<target-path>.branch main
```

### 3. Initialize and fetch

```bash
git submodule update --init --recursive <target-path>
```

### 4. Verify

```bash
git submodule status <target-path>
cat .gitmodules
```

### 5. Stage and report

```bash
git add .gitmodules <target-path>
git status
```

Report the staged paths to the user. Do NOT commit unless the user explicitly asks.

## Notes

- Use `--force` only if the user confirms an existing path should be overridden
- The submodule is pinned to the cloned commit; pair with `git-module-pull-latest` to advance it
- For nested submodules, `--recursive` ensures children are also initialized
