# Git Submodule Management Skills

One focused skill per git submodule operation. Each skill lives in its own directory with a `SKILL.md` definition.

| Skill | Purpose |
|---|---|
| [`git-module-init`](./git-module-init/SKILL.md) | Initially set up a GitHub repo as a submodule at a target path |
| [`git-module-pull-latest`](./git-module-pull-latest/SKILL.md) | Pull latest upstream commits for one or all submodules |
| [`git-module-cleanup`](./git-module-cleanup/SKILL.md) | Remove stale submodule artifacts and re-sync the remainder |
| [`git-module-track-main`](./git-module-track-main/SKILL.md) | Reconfigure all submodules to track `main` and fast-forward to latest |
| [`git-module-generate-repos-md`](./git-module-generate-repos-md/SKILL.md) | Generate/refresh `REPOS.md` from `.gitmodules`; preserve existing descriptions |

## Typical Flow

1. `git-module-init` — add a new dependency
2. `git-module-track-main` — ensure it tracks the right branch
3. `git-module-pull-latest` — routinely advance to upstream tip
4. `git-module-cleanup` — periodically reconcile `.gitmodules`, `.git/config`, and `.git/modules/`
5. `git-module-generate-repos-md` — keep the human-readable repo index in sync
