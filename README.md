# ccsilo

Claude Code plugin to isolate and manage config files (`.claude/`, `docs/`, etc.) in dedicated git repos.

## Features

- **init** — Create a silo repo and set up symlinks from your project
- **push** — Commit and push config changes to the silo repo
- **pull** — Pull latest config from the silo repo
- **status** — Show silo repo state (changes, sync, symlink health)
- **`.ccsilorc`** — Customize which files to include, exclude, and silo repo naming
- **i18n** — Korean / English output support

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

suffix: "-claude"
```

## License

MIT
