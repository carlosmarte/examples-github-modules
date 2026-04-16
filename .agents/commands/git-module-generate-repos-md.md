# Git Module Generate REPOS.md

Produce or refresh a `REPOS.md` at the repo root that lists every registered git submodule with its URL and a one-line description. Pre-existing descriptions in `REPOS.md` are reused as the source of truth.

## Arguments

$ARGUMENTS

<!-- Argument format: [output-path] -->
<!-- Argument 1 (optional, default `REPOS.md`) — output file path -->

## Prerequisites

- Inside a git repo with `.gitmodules` present
- Optional: existing `REPOS.md` at the repo root (parsed, not overwritten blindly)

## Instructions

### Step 1: Enumerate submodules from `.gitmodules`

```bash
git config --file .gitmodules --get-regexp '^submodule\..*\.(path|url)$'
```

Pair each `submodule.<name>.path` with its `submodule.<name>.url`. The `<name>` is the canonical submodule key.

### Step 2: Parse existing `REPOS.md` (if any) for descriptions

If `REPOS.md` exists, read it and extract the description for each submodule. Expected row format:

```markdown
| <name> | [<path>](<url>) | <description> |
```

Build a map `{ name → existing_description }`. Match by `<name>` first, then by `<path>` as a fallback (in case a submodule was renamed but kept its path).

**Treat existing descriptions as authoritative.** Never overwrite them, even if they look stale or empty-but-intentional (e.g., a single `-`).

### Step 3: Determine description for each submodule

For each submodule:

1. If a description exists in the parsed map → reuse it verbatim
2. Otherwise → generate one:
   - Prefer the first non-empty line of the submodule's own `README.md` (strip `#` headings, badges, HTML)
   - Fallback: derive from the repo name (e.g., `AI-Agent-Claude-MTA` → `AI Agent Claude MTA`) and clearly mark with a trailing ` _(auto)_` so the user can spot and curate it
   - If neither is available, write `_TBD_` and surface the gap to the user

Read the submodule README via:

```bash
test -f <path>/README.md && head -n 40 <path>/README.md
```

Do not invent facts about the repo; if the README is missing or unhelpful, prefer `_TBD_` over speculation.

### Step 4: Render `REPOS.md`

Write the file with this structure:

```markdown
# Repositories

This index is auto-maintained by `/git-module-generate-repos-md`.
Edit the **Description** column freely — your text is preserved on regeneration.

| Name | Repository | Description |
|---|---|---|
| <name> | [<path>](<url>) | <description> |
| ...    | ...            | ...           |
```

Sort rows alphabetically by `<name>` for stable diffs.

### Step 5: Report changes

Run `git diff -- REPOS.md` (or show a summary if the file is new) and tell the user:
- How many submodules were listed
- How many descriptions were reused vs newly generated
- Any rows still marked `_TBD_` that need their attention

Do NOT commit unless the user explicitly asks.

## Equivalent Make Target

```bash
make -f Makefile.git-module-repo repos-md [REPOS_MD=<path>]
```

## Failure Handling

- **No `.gitmodules`**: report that there are no submodules to index; do not create an empty `REPOS.md`
- **Malformed existing `REPOS.md` table**: do not silently discard descriptions — stop, show the parse failure, and ask the user how to proceed
- **Submodule directory uninitialized**: skip README inspection for that row and mark description `_TBD_`; suggest running `/git-module-pull-latest` first

## Notes

- The auto-generated marker (` _(auto)_`) is a signal, not a lock — once the user edits the description, they should remove the marker; subsequent runs preserve their edit regardless
- Match by submodule `<name>` (the `.gitmodules` section key), not by URL — URLs change when repos move between orgs but the submodule name typically does not
- Keep descriptions to a single line; multi-line descriptions break the markdown table
