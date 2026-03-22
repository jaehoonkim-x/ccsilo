---
name: pull
description: Pull latest changes from the Claude Code config silo repo (project or global)
argument-hint: "[--global]"
---

<Purpose>
Pull latest changes from the dedicated Claude Code config git repo ({project}-claude or ~/.claude/).
This plugin isolates Claude Code config files from the main project into a separate git repo.
</Purpose>

<Use_When>
- User requests "ccsilo pull", "silo pull", "config pull", "설정 pull"
- User wants to update Claude Code config from the remote silo repo
</Use_When>

## Flag Parsing

- `--global` → Pull from `~/.claude/` git repo
- No flag → Pull from `{project}-claude` repo

## Configuration

Load `.ccsilorc` from project root for `suffix` value. Defaults: suffix `"-claude"`.

## Steps

### Project Mode (default)

1. Determine current working directory and project name
2. Load `.ccsilorc` from project root (use defaults if not found)
3. Calculate silo repo path: `{project}{suffix}`
4. Verify the repo exists. If not, show error and stop
4. Run `git -C {project}-claude pull origin`
5. Show result

### Global Mode (`--global`)

1. Check if `~/.claude/` is a git repo (`git -C ~/.claude rev-parse --git-dir`)
2. If not a git repo, show error and stop
3. Run `git -C ~/.claude pull origin`
4. Show result

## Output Format

Detect user language and load messages from `i18n/{lang}.yaml` → `common.*` + `pull.*` keys.
Prefix all lines with `[ccsilo:pull]`.
