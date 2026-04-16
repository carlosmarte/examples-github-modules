---
name: gitmodules-repos-md
description: Use ONLY when the user explicitly asks to generate, refresh, or update REPOS.md from .gitmodules in this repo. Reads every registered submodule, emits a markdown table with name, repository link, and description, and preserves any descriptions already present in REPOS.md as the source of truth. Do NOT activate for general submodule management (use git-module-repo for that), unrelated documentation work, or generic git questions.
allowed-tools: Bash,Read,Write,Edit,Grep,Glob
argument-hint: "[output-path]"
disable-model-invocation: true
---

# Generate REPOS.md from .gitmodules

Build (or refresh) a `REPOS.md` index at the repo root from `.gitmodules`. Existing per-repo descriptions in `REPOS.md` are authoritative and preserved verbatim across runs — only new or missing rows get a freshly written description.

## Inputs

- `$1` (optional) — output path. Default: `REPOS.md` at the repo root.
- `.gitmodules` at the repo root — required. If absent, abort with a clear message; do not create an empty file.

## Output Format

```markdown
# Repositories

This index is auto-maintained by `/gitmodules-repos-md`.
Edit the **Description** column freely — your text is preserved on regeneration.

| Repo | Link | Description |
|---|---|---|
| <name> | [<path>](<url>) | <description> |
```

Rows are sorted alphabetically by `<name>` for stable diffs.

## Step 1: Confirm `.gitmodules` exists

```bash
test -f .gitmodules || { echo "no .gitmodules — nothing to index"; exit 1; }
```

## Step 2: Enumerate submodules

```bash
git config --file .gitmodules --get-regexp '^submodule\..*\.(path|url)$'
```

Pair each `submodule.<name>.path` with its `submodule.<name>.url`. The `<name>` (the section key in `.gitmodules`) is the canonical identifier — match on it, not on URL (URLs change when repos move orgs).

## Step 3: Parse existing REPOS.md descriptions (if any)

If the output file exists, read it and extract `{ name → description }` from rows shaped like:

```markdown
| <name> | [<path>](<url>) | <description> |
```

Matching rules:
1. Match by `<name>` first.
2. Fallback: match by `<path>` (handles a renamed submodule that kept its directory).
3. Treat the parsed description as authoritative — never overwrite it, even if it looks empty/stale (e.g. a single `-` or `_TBD_`).

If the existing table is malformed (column count off, missing header, etc.), STOP and surface the parse error to the user. Do not silently discard descriptions.

## Step 4: Resolve description for each submodule

For each submodule, in order:

1. **Reuse** the parsed existing description if one exists for this `<name>` (or `<path>` fallback). Use it verbatim.
2. Otherwise **generate** one:
   - Read the submodule's own `README.md` if present:
     ```bash
     test -f <path>/README.md && head -n 40 <path>/README.md
     ```
     Use the first non-empty prose line. Strip leading `#` heading marks, badge images, raw HTML, and surrounding whitespace.
   - If no README is available, derive a human-readable label from the repo name (e.g. `AI-Agent-Claude-MTA` → `AI Agent Claude MTA`) and append ` _(auto)_` so the user can spot and curate it later.
   - If the submodule directory is uninitialized AND the repo name is uninformative, write `_TBD_` and surface the gap in the final report.

Never invent facts about a repo. When unsure, prefer `_TBD_` over speculation.

## Step 5: Render the table

Write the output file with the structure shown above. Each row:

```
| <name> | [<path>](<url>) | <description> |
```

Keep descriptions to a single line — multi-line text breaks the markdown table. Sort rows alphabetically by `<name>`.

## Step 6: Report

After writing, run `git diff -- <output-path>` (or note "new file" if it didn't previously exist) and tell the user:

- Total submodules listed
- How many descriptions were **reused** vs **newly generated**
- Any rows still marked `_(auto)_` or `_TBD_` that need their attention

Do NOT commit the change unless the user explicitly asks.

## Edge Cases

- **No `.gitmodules`** — report and exit; do not create an empty `REPOS.md`.
- **Malformed existing `REPOS.md`** — stop and ask the user; do not destroy descriptions.
- **Uninitialized submodule** — skip README inspection, mark `_TBD_`, suggest the user run `git submodule update --init` first.
- **Duplicate submodule names** — abort and ask the user to disambiguate; the index needs unique keys.

## Notes

- The ` _(auto)_` marker is a hint, not a lock. Once the user edits a description, they remove the marker; subsequent runs preserve the edit either way (Step 3 always wins over Step 4).
- This skill only generates the index. For adding, removing, pulling, or retargeting submodules, defer to the `git-module-repo` skill.
