---
description: Initialize a Claude Code config silo repo and set up symlinks (project or global)
argument-hint: "[--global]"
---

<Purpose>
Isolate Claude Code config files (.claude/, docs/) from the main project into a dedicated git repo ({project}-ccsilo),
or initialize ~/.claude/ as a git repo for global config versioning.
</Purpose>

<Use_When>
- User requests "ccsilo init", "silo init", "config init", "설정 초기화"
- User wants to start version-controlling Claude Code config separately
</Use_When>

## Flag Parsing

- `--global` → Initialize `~/.claude/` as a git repo
- No flag → Create `{project}-ccsilo` repo and set up symlinks

## Configuration (.ccsilorc)

Before executing, check for `.ccsilorc` in the project root. If not found, use defaults.

```yaml
# .ccsilorc — ccsilo configuration
# Directories/files to include in the silo and symlink back
# Supports gitignore-style paths: dirs, subdirs, files, globs
include:
  - .claude/
  - docs/api/
  - docs/guides/*.md
  - ANTIBOT.md

# Files to exclude from silo (kept in project git)
exclude:
  - CLAUDE.md

# Silo repo name suffix (default: -ccsilo)
suffix: "-ccsilo"
```

**Defaults** (when no `.ccsilorc` exists):
- `include`: `[".claude/", "docs/"]`
- `exclude`: `["CLAUDE.md"]`
- `suffix`: `"-ccsilo"`

## Interactive UX Rules

**All user-facing choices MUST use `AskUserQuestion` tool.** Never skip prompts or assume defaults silently.

## Steps

### Project Mode (default)

1. Determine current project path and name
2. Load `.ccsilorc` from project root
3. **If `.ccsilorc` not found** → Run [File Discovery](#file-discovery) flow
   - **If `.ccsilorc` found** → Show loaded config summary, proceed with its values
4. Calculate repo name: `{project}{suffix}` (suffix from config)
5. Check if silo directory already exists
   - If exists, stop
6. Use `AskUserQuestion` to confirm before proceeding:
   ```
   Question: "다음 설정으로 silo를 초기화합니다. 진행할까요?"
   Body:
     - Silo repo: {silo_name}
     - Include (silo + symlink): {include 목록}
     - Exclude: {exclude 목록}
   Options: ["진행", "설정 변경", "취소"]
   ```
   - "설정 변경" → Re-run [File Discovery](#file-discovery) flow
   - "취소" → Stop
7. Create silo directory
8. Run `git -C {silo} init`
9. Copy files/directories listed in `include` from project to silo repo
   - Skip items listed in `exclude`
   - Skip items that don't exist (with info log)
   - For sub-directory entries (e.g. `docs/api/`), preserve the parent directory structure in silo
10. Create `.gitignore` with sensible defaults (`.DS_Store`)
11. Save final config as `.ccsilorc` into the silo repo
12. Generate `setup-symlinks.sh` script:
    - Symlinks each `include` entry from silo into the original project
    - Backs up existing files as `.bak` before creating symlinks
13. Execute the symlink script
14. Use `AskUserQuestion`:
    ```
    Question: "GitHub에 private repo를 생성할까요?"
    Options: ["생성", "나중에"]
    ```
    - "생성" → `gh repo create {silo_name} --private --source=. --push`

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
5. Use `AskUserQuestion`:
   ```
   Question: "GitHub에 private repo를 생성할까요?"
   Options: ["생성", "나중에"]
   ```
   - "생성" → `gh repo create dotclaude --private --source=~/.claude --push`

---

## File Discovery

This flow runs when no `.ccsilorc` exists. It scans the project and lets the user choose what to include.

1. **Scan project root** for Claude-related files and directories:
   - Directories: `.claude/`, `docs/`
   - Root `.md` files: `AGENTS.md`, `ANTIBOT.md`, `CONVENTIONS.md`, or any other `*.md` files at project root (excluding `README.md`, `CHANGELOG.md`, `LICENSE.md`)
   - Other known configs: `.cursorrules`, `.windsurfrules`, `.github/copilot-instructions.md`

2. **Build candidate list** from scan results. Example:
   ```
   Detected:
     [1] .claude/          (directory)
     [2] docs/             (directory)
     [3] AGENTS.md         (file)
     [4] ANTIBOT.md        (file)
   ```

3. **Use `AskUserQuestion`** to let user select:
   ```
   Question: "silo에 포함할 항목을 선택하세요 (기본: .claude/, docs/)"
   Body: {detected 항목 목록 표시}
   Options: ["기본값 사용 (.claude/, docs/)", "전체 선택", "직접 선택"]
   ```
   - "기본값 사용" → `include: [".claude/", "docs/"]`
   - "전체 선택" → 감지된 모든 항목 포함
   - "직접 선택" → 추가 `AskUserQuestion`으로 개별 항목 y/n 확인

4. **Directory Drill-Down** — For each selected **directory**, ask granularity:
   ```
   Question: "{dir_name} 포함 범위를 선택하세요"
   Options: ["전체 디렉토리", "하위 항목 선택"]
   ```
   - "전체 디렉토리" → include as-is (e.g. `docs/`)
   - "하위 항목 선택" → List immediate children (subdirs and files) of that directory, then use `AskUserQuestion` with `multiSelect: true`:
     ```
     Question: "{dir_name} 에서 포함할 항목을 선택하세요"
     Body: {하위 항목 목록}
     Options: [{child1}, {child2}, {child3}, ...]
     ```
     Selected children replace the parent in `include` (e.g. `docs/` → `docs/api/`, `docs/guides/`)

5. **Use `AskUserQuestion`** for exclude:
   ```
   Question: "silo에서 제외할 파일이 있나요?"
   Body: "기본 제외: CLAUDE.md (프로젝트 git에 유지)"
   Options: ["기본값 사용", "직접 선택"]
   ```

6. Save selections and continue to Step 4 of Project Mode

## Output Format

Detect user language and load messages from `i18n/{lang}.yaml` → `common.*` + `init.*` keys.
Prefix all lines with `[ccsilo:init]`.
