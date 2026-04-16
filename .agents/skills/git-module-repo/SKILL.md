---
name: git-module-repo
description: Use ONLY when the user explicitly asks to manage git submodules in this repo — adding/initializing a submodule, pulling latest commits for one or all submodules, auditing/cleaning up stale submodule entries, retargeting submodules to a branch (e.g. `main`), or (re)generating `REPOS.md`. Do NOT activate for general git operations, repo navigation, project setup, or unrelated work. When active, delegate execution to `Makefile.git-module-repo` rather than running raw `git submodule` commands.
version: 1.0.0
---

# Git Module Repo (Makefile-backed)

Single source of truth for git-submodule operations in this repo: every action is a target in [`Makefile.git-module-repo`](../../../Makefile.git-module-repo) at the repo root. **Always invoke via `make -f Makefile.git-module-repo <target>`** — do not reimplement the underlying `git submodule` commands inline.

## Activation Boundary (read carefully)

Activate **only** for explicit submodule-management requests. Examples that activate:

- "add `https://github.com/foo/bar` as a submodule under `vendor/bar`"
- "pull the latest for all submodules"
- "audit my submodules — anything stale?"
- "make every submodule track main"
- "regenerate REPOS.md"

Examples that do **NOT** activate:

- "what's in this repo" / "explain this code" / "init a project" / generic `git` questions
- Editing files inside a submodule directory (that's just normal file editing)
- Cloning the parent repo

If the request is ambiguous, ask the user before running anything.

## Target ↔ Intent Map

| User intent | Run |
|---|---|
| Add a new submodule | `make -f Makefile.git-module-repo init REPO_URL=<url> TARGET_PATH=<path> [BRANCH=main]` |
| Pull latest for one submodule | `make -f Makefile.git-module-repo pull-latest TARGET_PATH=<path>` |
| Pull latest for all submodules | `make -f Makefile.git-module-repo pull-latest` |
| Read-only audit (no changes) | `make -f Makefile.git-module-repo cleanup-audit` |
| Sync URLs + re-init + pull (non-destructive) | `make -f Makefile.git-module-repo cleanup-sync` |
| Audit + sync bundle | `make -f Makefile.git-module-repo cleanup` |
| Permanently remove a submodule (destructive) | `make -f Makefile.git-module-repo cleanup-remove TARGET_PATH=<path>` |
| Retarget every submodule to `main` | `make -f Makefile.git-module-repo track-main` |
| Retarget to a different branch / one path | `make -f Makefile.git-module-repo track-main BRANCH=<b> [TARGET_PATH=<p>]` |
| Generate / refresh `REPOS.md` (preserves descriptions) | `make -f Makefile.git-module-repo repos-md` |

`make -f Makefile.git-module-repo help` prints the same surface.

## Operating Rules

1. **Always run from the repo root** so the `-f Makefile.git-module-repo` path resolves. If the user is elsewhere, `cd` to the repo root first or pass an absolute `-f` path.
2. **Confirm before destructive targets.** `cleanup-remove` requires explicit user authorization for the specific `TARGET_PATH`. Never infer which submodule to delete.
3. **Never commit** the resulting changes unless the user explicitly asks — these targets stage but do not commit.
4. **Surface the output verbatim.** The Makefile already prints `git submodule status`, diff summaries, and counts; relay them rather than re-summarizing inaccurately.
5. **Don't reimplement.** If a step seems missing, edit the Makefile (and the matching command file in `.agents/commands/`) — don't shell out to raw `git submodule` from inside this skill.
6. **Required arguments:** `init` needs `REPO_URL` and `TARGET_PATH`; `cleanup-remove` needs `TARGET_PATH`. If absent, ask the user before invoking.

## Reference Documents

- Per-action playbooks (with full step-by-step rationale): [`.agents/commands/`](../../commands/)
- Make target source: [`Makefile.git-module-repo`](../../../Makefile.git-module-repo)
