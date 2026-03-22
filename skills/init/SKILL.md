---
name: init
description: Initialize a Claude Code config silo repo and set up symlinks (project or global)
argument-hint: "[--global]"
---

<Purpose>
Isolate Claude Code config files (.claude/, docs/) from the main project into a dedicated git repo ({project}-claude),
or initialize ~/.claude/ as a git repo for global config versioning.
</Purpose>

<Use_When>
- User requests "ccsilo init", "silo init", "config init", "설정 초기화"
- User wants to start version-controlling Claude Code config separately
</Use_When>

## Flag Parsing

- `--global` → Initialize `~/.claude/` as a git repo
- No flag → Create `{project}-claude` repo and set up symlinks

## Configuration (.ccsilorc)

Before executing, check for `.ccsilorc` in the project root. If not found, use defaults.

```yaml
# .ccsilorc — ccsilo configuration
# Directories/files to include in the silo (symlinked)
include:
  - .claude/
  - docs/

# Files to exclude from silo (kept in project git)
exclude:
  - CLAUDE.md

# Additional .gitignore entries for the silo repo
gitignore:
  - .env
  - "*.local"
  - node_modules/

# Silo repo name suffix (default: -claude)
suffix: "-claude"
```

**Defaults** (when no `.ccsilorc` exists):
- `include`: `[".claude/", "docs/"]`
- `exclude`: `["CLAUDE.md"]`
- `gitignore`: `[".env", "*.local", "node_modules/"]`
- `suffix`: `"-claude"`

## Steps

### Project Mode (default)

1. Determine current project path and name
2. Load `.ccsilorc` from project root (use defaults if not found)
3. Calculate repo name: `{project}{suffix}` (suffix from config)
4. Check if silo directory already exists
   - If exists, stop
5. Create silo directory
6. Run `git -C {silo} init`
7. Copy files/directories listed in `include` from project to silo repo
   - Skip items listed in `exclude`
   - Skip items that don't exist (with info log)
8. Create `.gitignore` from config `gitignore` entries
9. Copy `.ccsilorc` into the silo repo (so config travels with it)
10. Generate `setup-symlinks.sh` script:
    - Symlinks each `include` entry from silo into the original project
    - Backs up existing files as `.bak` before creating symlinks
11. Execute the symlink script
12. Ask user about GitHub repo creation
    - If yes: `gh repo create {silo_name} --private --source=. --push`

### Global Mode (`--global`)

1. Check if `~/.claude/` is already a git repo
   - If already a git repo, stop
2. Run `git -C ~/.claude init`
3. Create `.gitignore`:
   ```
   .env
   *.local
   plugins/cache/
   credentials/
   ```
4. Create initial commit
5. Ask user about GitHub repo creation
   - If yes: `gh repo create dotclaude --private --source=~/.claude --push`

## Output Format

Detect user language and load messages from `i18n/{lang}.yaml` → `common.*` + `init.*` keys.
Prefix all lines with `[ccsilo:init]`.
