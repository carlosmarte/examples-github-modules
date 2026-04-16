# Git Module Track Main

Reconfigure every registered git submodule to track the `main` branch and fast-forward to its latest commit.

## Arguments

$ARGUMENTS

<!-- Argument format: [branch] [target-path] -->
<!-- Argument 1 (optional, default `main`) — branch every submodule should track -->
<!-- Argument 2 (optional) — restrict to a single submodule path; otherwise apply to all -->

## Prerequisites

- Inside a git repo with at least one submodule
- Each upstream submodule actually has a branch matching `<branch>` (verify in step 2)
- Working trees inside submodules are clean

## Instructions

### Step 1: Enumerate submodules

```bash
git config --file .gitmodules --get-regexp 'submodule\..*\.path'
```

Yields one `submodule.<name>.path <path>` line per registered submodule.

### Step 2: Verify each upstream has the target branch

For each submodule path:

```bash
git -C <path> fetch origin
git -C <path> ls-remote --heads origin <branch>
```

If a submodule's upstream lacks the branch, surface it to the user before proceeding — do not silently skip or guess an alternative.

### Step 3: Update `.gitmodules` to track the branch

For each submodule:

```bash
git config -f .gitmodules submodule.<name>.branch <branch>
```

### Step 4: Propagate to `.git/config`

```bash
git submodule sync --recursive
```

### Step 5: Check out and fast-forward each submodule to the branch tip

For each submodule:

```bash
git -C <path> checkout <branch>
git -C <path> pull --ff-only origin <branch>
```

`--ff-only` so a divergent local history fails loudly instead of producing a merge commit.

### Step 6: Verify

```bash
git submodule status
git submodule foreach 'git rev-parse --abbrev-ref HEAD'
git diff --submodule=log
```

Confirm every submodule reports `<branch>` as its current ref and the parent repo shows the new pointers.

### Step 7: Stage parent-repo updates

```bash
git add .gitmodules <updated-submodule-paths>
git status
```

Do NOT commit unless the user explicitly asks.

## Equivalent Make Target

```bash
make -f Makefile.git-module-repo track-main [BRANCH=<branch>] [TARGET_PATH=<path>]
```

## Failure Handling

- **`pull --ff-only` rejects**: a submodule has divergent local commits — stop and surface; do not force
- **Branch missing upstream**: report the submodule and its available branches; let the user decide
- **Detached HEAD after `update --remote`**: re-run step 5 explicitly to land on the branch ref, not just its commit

## Notes

- The combination of `.gitmodules` `branch =` setting + `git submodule update --remote` keeps submodules following `main` going forward — without it, a submodule re-pins to whatever commit was checked out
- For repos still on `master`, pass the branch explicitly rather than assuming `main`
