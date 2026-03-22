---
name: push
description: Commit and push Claude Code config changes to the silo repo (project or global)
argument-hint: "[--global] [-m 'message']"
---

<Purpose>
Commit and push Claude Code config changes to the dedicated silo git repo ({project}-claude or ~/.claude/).
This plugin isolates Claude Code config files from the main project into a separate git repo.
</Purpose>

<Use_When>
- User requests "ccsilo push", "silo push", "config push", "설정 push"
- User wants to save Claude Code config changes to the remote silo repo
</Use_When>

## Flag Parsing

- `--global` → Push to `~/.claude/` git repo
- `-m 'message'` → Custom commit message (auto-generated if omitted)
- No flag → Push to `{project}-claude` repo

## Configuration

Load `.ccsilorc` from project root for `suffix` value. Defaults: suffix `"-claude"`.

## Steps

### Project Mode (default)

1. Determine current working directory and project name
2. Load `.ccsilorc` from project root (use defaults if not found)
3. Calculate silo repo path: `{project}{suffix}`
4. Verify the repo exists. If not, show error and stop
4. Run `git -C {project}-claude status --short` to check for changes
5. If no changes, stop
6. If changes exist:
   - Run `git -C {project}-claude add -A`
   - Generate commit message (user-specified or auto: `chore: sync claude config`)
   - Run `git -C {project}-claude commit -m "{message}"`
   - Run `git -C {project}-claude push origin`
7. Show result

### Global Mode (`--global`)

1. Check if `~/.claude/` is a git repo
2. If not a git repo, show error and stop
3. Run `git -C ~/.claude status --short` to check for changes
4. If no changes, stop
5. If changes exist:
   - Run `git -C ~/.claude add -A`
   - Generate commit message
   - Run `git -C ~/.claude commit -m "{message}"`
   - Run `git -C ~/.claude push origin`
6. Show result

## Security

- Verify `.env` and secret files are NOT included in the commit
- Verify `.gitignore` covers sensitive files
- Before push, show change summary and ask for confirmation

## Output Format

Detect user language and load messages from `i18n/{lang}.yaml` → `common.*` + `push.*` keys.
Prefix all lines with `[ccsilo:push]`.
