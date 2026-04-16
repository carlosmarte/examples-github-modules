# Git Submodule Management Commands

User-invoked commands for managing git submodules. Each command is a standalone `.md` file with explicit `$ARGUMENTS` parsing — invoke deterministically rather than relying on context-based skill activation.

| Command | Purpose |
|---|---|
| [`git-module-init`](./git-module-init.md) | Initially set up a GitHub repo as a submodule at a target path |
| [`git-module-pull-latest`](./git-module-pull-latest.md) | Pull latest upstream commits for one or all submodules |
| [`git-module-cleanup`](./git-module-cleanup.md) | Audit / sync / remove submodule entries (mode arg gates destructive ops) |
| [`git-module-track-main`](./git-module-track-main.md) | Reconfigure all submodules to track `main` and fast-forward to latest |
| [`git-module-generate-repos-md`](./git-module-generate-repos-md.md) | Generate/refresh `REPOS.md` from `.gitmodules`; preserve existing descriptions |

## Why commands instead of skills?

Skills are model-invoked — Claude pulls them into context whenever the trigger description matches, which can fire unexpectedly during unrelated work. Commands are user-invoked: nothing runs unless you explicitly call it.

## To make these slash-command discoverable

By default Claude Code only auto-discovers commands from `.claude/commands/` (project) or `~/.claude/commands/` (user). To expose these as `/git-module-init` etc:

```bash
mkdir -p .claude/commands
ln -s ../../.agents/commands/git-module-init.md            .claude/commands/git-module-init.md
ln -s ../../.agents/commands/git-module-pull-latest.md     .claude/commands/git-module-pull-latest.md
ln -s ../../.agents/commands/git-module-cleanup.md         .claude/commands/git-module-cleanup.md
ln -s ../../.agents/commands/git-module-track-main.md      .claude/commands/git-module-track-main.md
ln -s ../../.agents/commands/git-module-generate-repos-md.md .claude/commands/git-module-generate-repos-md.md
```

## Equivalent Make targets

Each command also has a deterministic Make target — see [`Makefile.git-module-repo`](../../Makefile.git-module-repo) at the repo root.

## Typical Flow

1. `/git-module-init` — add a new dependency
2. `/git-module-track-main` — ensure it tracks the right branch
3. `/git-module-pull-latest` — routinely advance to upstream tip
4. `/git-module-cleanup` — periodically reconcile `.gitmodules`, `.git/config`, and `.git/modules/`
5. `/git-module-generate-repos-md` — keep the human-readable repo index in sync
