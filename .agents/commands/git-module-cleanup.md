# Git Module Cleanup

Remove stale submodule artifacts (deleted from `.gitmodules` but lingering on disk / in `.git/config`) and re-sync the remaining submodules to their latest tracked commits.

## Arguments

$ARGUMENTS

<!-- Argument format: [audit|sync|remove <target-path>|all] -->
<!-- Argument 1 — mode: -->
<!--   `audit`              read-only mismatch report (default if no args) -->
<!--   `sync`               sync URLs + re-init + pull remote -->
<!--   `remove <path>`      destructive: deinit + rm + clear .git/modules/<path> -->
<!--   `all`                audit + sync (non-destructive bundle) -->

If the mode is `remove`, the user MUST also pass the target path; never guess which submodule to remove.

## Prerequisites

- Inside a git repo
- For `remove`: user has explicitly named the submodule to delete

## Instructions

### Mode: audit (read-only)

```bash
echo "== git submodule status =="
git submodule status || true
echo
echo "== .gitmodules =="
cat .gitmodules 2>/dev/null || echo "(no .gitmodules)"
echo
echo "== .git/config submodule entries =="
git config --file .git/config --get-regexp '^submodule\.' || echo "(none)"
echo
echo "== .git/modules/ =="
ls -la .git/modules 2>/dev/null || echo "(none)"
```

Diff the three sources (`.gitmodules`, `.git/config`, `.git/modules/`) and report mismatches to the user. **Do not delete anything in audit mode.**

### Mode: remove (destructive — requires explicit target-path)

```bash
git submodule deinit -f <target-path>
git rm -f <target-path>
rm -rf .git/modules/<target-path>
git status
```

If the user named multiple paths, run the block once per path.

### Mode: sync (non-destructive)

Sync URL/branch changes from `.gitmodules` into `.git/config`:

```bash
git submodule sync --recursive
```

Re-initialize and update remaining submodules:

```bash
git submodule update --init --recursive
git submodule update --remote --merge --recursive
```

### Mode: all

Run audit, then sync. Never include `remove` here — destructive removal must be invoked explicitly.

### Verify

```bash
git submodule status
git status
```

Report:
- Submodules removed (if any)
- Submodules re-synced
- New pinned commits (from sync step)

Do NOT commit unless the user explicitly asks.

## Equivalent Make Targets

```bash
make -f Makefile.git-module-repo cleanup-audit
make -f Makefile.git-module-repo cleanup-sync
make -f Makefile.git-module-repo cleanup-remove TARGET_PATH=<path>
make -f Makefile.git-module-repo cleanup            # = audit + sync
```

## Failure Handling

- **`deinit` fails because of local changes**: stop and surface to the user; never use `-f` to discard their work without confirmation
- **`.git/modules/<name>` removal fails**: usually a permissions issue — report the path, do not retry with `sudo`

## Notes

- `git submodule sync` alone solves most "URL changed upstream" issues without any deletion
- If the user said "cleanup" without a mode, default to `audit` and ask before doing anything destructive
