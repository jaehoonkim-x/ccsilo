# ccsilo

Claude Code plugin to isolate and manage config files (`.claude/`, `docs/`, etc.) in dedicated git repos.

## Why?

Claude Code generates config and context files (`.claude/`, `docs/`, `AGENTS.md`, etc.) alongside your project code. These files are useful to version-control but can clutter your main repo's history. **ccsilo** moves them into a separate git repo and links them back via symlinks — keeping your main repo clean while preserving full version history for configs.

## Features

- **init** — Create a silo repo (`{project}-ccsilo`) and set up symlinks from your project
- **push** — Commit and push config changes to the silo repo
- **pull** — Pull latest config from the silo repo
- **status** — Show silo repo state (changes, sync, symlink health)
- **`.ccsilorc`** — Customize which files to include, exclude, and silo repo naming
- **i18n** — Korean / English output support
- **Global mode** — Version-control `~/.claude/` as a git repo (`--global` flag)

## Install

```bash
git clone https://github.com/jaehoonkim-x/ccsilo.git ~/.claude/plugins/ccsilo
```

## Usage

```
/ccsilo:init              # Initialize silo repo for current project
/ccsilo:init --global     # Initialize ~/.claude/ as a git repo
/ccsilo:push              # Push config changes
/ccsilo:push -m 'msg'     # Push with custom commit message
/ccsilo:pull              # Pull latest config
/ccsilo:status            # Show silo repo status
```

## How It Works

1. `init` scans your project for Claude-related files (`.claude/`, `docs/`, `AGENTS.md`, etc.)
2. Creates a sibling repo (`{project}-ccsilo`) and copies selected files into it
3. Replaces originals with symlinks pointing to the silo repo
4. Optionally creates a GitHub private repo

```
my-project/
  .claude/ → ../my-project-ccsilo/.claude/
  docs/    → ../my-project-ccsilo/docs/
  src/
  ...

my-project-ccsilo/    ← separate git repo
  .claude/
  docs/
  .ccsilorc
  setup-symlinks.sh
```

## Configuration

Create `.ccsilorc` in your project root to customize:

```yaml
include:
  - .claude/
  - docs/
  - .cursorrules

exclude:
  - CLAUDE.md

gitignore:
  - .env
  - "*.local"
  - node_modules/

suffix: "-ccsilo"
```

| Key | Default | Description |
|-----|---------|-------------|
| `include` | `.claude/`, `docs/` | Files/dirs to move into the silo |
| `exclude` | `CLAUDE.md` | Files to keep in the main project |
| `gitignore` | `.env`, `*.local`, `node_modules/` | Silo repo `.gitignore` entries |
| `suffix` | `-ccsilo` | Silo repo name suffix |

## License

MIT
