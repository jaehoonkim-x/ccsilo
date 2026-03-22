---
description: Show the current state of the Claude Code config silo repo (project or global)
argument-hint: "[--global]"
---

<Purpose>
Show the current status of the silo repo: changed files, last commit, remote connection, and sync state.
</Purpose>

<Use_When>
- User requests "ccsilo status", "silo status", "config status", "설정 상태"
- User wants to check silo repo state before push/pull
</Use_When>

## Flag Parsing

- `--global` → Show status of `~/.claude/` git repo
- No flag → Show status of `{project}-ccsilo` repo

## Configuration

Load `.ccsilorc` from project root for `suffix` and `include` values. Defaults: suffix `"-ccsilo"`, include `[".claude/", "docs/"]`.

## Steps

### Project Mode (default)

1. Determine current working directory and project name
2. Load `.ccsilorc` from project root (use defaults if not found)
3. Calculate silo repo path: `{project}{suffix}`
4. Verify the repo exists. If not, show error and stop
5. Collect and display:
   - **Changed files**: `git -C {repo} status --short`
   - **Last commit**: `git -C {repo} log -1 --format="%h %s (%cr)"`
   - **Branch**: `git -C {repo} branch --show-current`
   - **Remote**: `git -C {repo} remote -v` (show if configured, warn if not)
   - **Sync state**: `git -C {repo} status -sb` (ahead/behind remote)
   - **Symlink health**: verify `include` items are correctly symlinked to the silo repo

### Global Mode (`--global`)

1. Check if `~/.claude/` is a git repo (`git -C ~/.claude rev-parse --git-dir`)
2. If not a git repo, show error and stop
3. Collect and display:
   - **Changed files**: `git -C ~/.claude status --short`
   - **Last commit**: `git -C ~/.claude log -1 --format="%h %s (%cr)"`
   - **Branch**: `git -C ~/.claude branch --show-current`
   - **Remote**: `git -C ~/.claude remote -v` (show if configured, warn if not)
   - **Sync state**: `git -C ~/.claude status -sb` (ahead/behind remote)

## Output Format

Detect user language and load messages from `i18n/{lang}.yaml` → `common.*` + `status.*` keys.
Prefix all lines with `[ccsilo:status]`.
Symlink health check uses `include` entries from `.ccsilorc` (or defaults) to verify each symlink target.
