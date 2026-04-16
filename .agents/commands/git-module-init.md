# Git Module Init

Initial setup of a GitHub repository as a git submodule at a specified path.

## Arguments

$ARGUMENTS

<!-- Argument format: <repo-url> <target-path> [branch] -->
<!-- Argument 1 — full GitHub repository URL (e.g., https://github.com/org/repo) -->
<!-- Argument 2 — relative path where the submodule should be cloned (e.g., ./vendor/repo) -->
<!-- Argument 3 (optional) — branch to track; defaults to `main` -->

If any required argument is missing, ask the user to provide it before continuing.

## Prerequisites

- Inside a git repository (`git rev-parse --is-inside-work-tree`)
- `<target-path>` does not already exist or conflict with a tracked path
- Network access to the remote

## Instructions

### Step 1: Validate

```bash
git rev-parse --is-inside-work-tree
test ! -e <target-path> || echo "PATH EXISTS — abort or use --force"
```

### Step 2: Add submodule (tracking the requested branch)

```bash
git submodule add -b <branch> <repo-url> <target-path>
```

If no branch was provided, omit `-b <branch>` (defaults to remote HEAD), then pin tracking via:

```bash
git config -f .gitmodules submodule.<target-path>.branch main
```

### Step 3: Initialize and fetch

```bash
git submodule update --init --recursive <target-path>
```

### Step 4: Verify

```bash
git submodule status <target-path>
cat .gitmodules
```

### Step 5: Stage and report

```bash
git add .gitmodules <target-path>
git status
```

Report the staged paths to the user. Do NOT commit unless the user explicitly asks.

## Equivalent Make Target

```bash
make -f Makefile.git-module-repo init REPO_URL=<url> TARGET_PATH=<path> [BRANCH=main]
```

## Notes

- Use `--force` only if the user confirms an existing path should be overridden
- The submodule is pinned to the cloned commit; pair with `/git-module-pull-latest` to advance it
- For nested submodules, `--recursive` ensures children are also initialized
