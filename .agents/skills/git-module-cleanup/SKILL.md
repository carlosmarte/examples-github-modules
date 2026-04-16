---
name: git-module-cleanup
description: This skill should be used when the user asks to "cleanup stale submodules", "remove orphaned git modules", "sync submodules", "prune submodules", "fix dirty submodules", or wants to clear out removed/stale submodule directories and re-sync the remaining ones to their latest tracked commits.
version: 1.0.0
---

# Git Module Cleanup

Remove stale submodule artifacts (deleted from `.gitmodules` but lingering on disk / in `.git/config`) and re-sync the remaining submodules to their latest tracked commits.

## When This Skill Applies

- A submodule entry was removed from `.gitmodules` but the directory or `.git/modules/<name>` still exists
- `.git/config` has `submodule.*` entries that no longer match `.gitmodules`
- After upstream submodule URL changes that need to propagate locally
- Periodic hygiene: bring all submodules to a known clean, latest state

## Prerequisites

- Inside a git repo
- User has confirmed which (if any) submodules they intend to permanently remove — never delete on assumption

## Steps

### 1. Audit current state

```bash
git submodule status
cat .gitmodules 2>/dev/null
git config --file .git/config --get-regexp '^submodule\.' || true
ls -la .git/modules 2>/dev/null
```

Diff the three sources (`.gitmodules`, `.git/config`, `.git/modules/`) and report mismatches to the user before deleting anything.

### 2. Remove stale submodule entries (only with user confirmation)

For each submodule the user confirms should be removed:

```bash
git submodule deinit -f <target-path>
git rm -f <target-path>
rm -rf .git/modules/<target-path>
```

### 3. Sync URL/branch changes from `.gitmodules` into `.git/config`

```bash
git submodule sync --recursive
```

This is safe and non-destructive — it copies the current `.gitmodules` URLs/branches into `.git/config`.

### 4. Re-initialize and update remaining submodules

```bash
git submodule update --init --recursive
git submodule update --remote --merge --recursive
```

### 5. Verify

```bash
git submodule status
git status
```

Report:
- Submodules removed
- Submodules re-synced
- New pinned commits (from step 4)

Do NOT commit unless the user explicitly asks.

## Failure Handling

- **`deinit` fails because of local changes**: stop and surface to the user; never use `-f` to discard their work without confirmation
- **`.git/modules/<name>` removal fails**: usually a permissions issue — report the path, do not retry with `sudo`

## Notes

- This skill is destructive in step 2; treat the user's "cleanup" request as authorization to *audit*, not to delete. Always confirm the deletion list before running `git rm`
- `git submodule sync` alone solves most "URL changed upstream" issues without any deletion
