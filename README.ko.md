# ccsilo

Claude Code 설정 파일(`.claude/`, `docs/` 등)을 별도 git 저장소로 분리하여 관리하는 플러그인입니다.

## 왜 필요한가?

Claude Code는 프로젝트와 함께 설정 및 컨텍스트 파일(`.claude/`, `docs/`, `AGENTS.md` 등)을 생성합니다. 이 파일들은 버전 관리가 필요하지만 메인 저장소의 히스토리를 어지럽힐 수 있습니다. **ccsilo**는 이 파일들을 별도 git 저장소로 옮기고 symlink로 연결합니다 — 메인 저장소는 깔끔하게, 설정 히스토리는 온전하게 유지됩니다.

## 기능

- **init** — silo 저장소(`{프로젝트}-ccsilo`) 생성 및 symlink 설정
- **push** — 설정 변경사항을 silo 저장소에 커밋 & 푸시
- **pull** — silo 저장소에서 최신 설정 가져오기
- **status** — silo 저장소 상태 확인 (변경사항, 동기화, symlink 건강도)
- **`.ccsilorc`** — include, exclude, 저장소 이름 등 커스터마이즈
- **i18n** — 한국어 / 영어 출력 지원
- **글로벌 모드** — `~/.claude/`를 git 저장소로 관리 (`--global` 플래그)

## 설치

Claude Code에서 실행:

```
/plugin install ccsilo@ccsilo
```

## 사용법

```
/ccsilo:init              # 현재 프로젝트의 silo 저장소 초기화
/ccsilo:init --global     # ~/.claude/를 git 저장소로 초기화
/ccsilo:push              # 설정 변경사항 푸시
/ccsilo:push -m '메시지'   # 커밋 메시지 지정하여 푸시
/ccsilo:pull              # 최신 설정 가져오기
/ccsilo:status            # silo 저장소 상태 확인
```

## 동작 방식

1. `init`이 프로젝트에서 Claude 관련 파일(`.claude/`, `docs/`, `AGENTS.md` 등)을 스캔
2. 형제 디렉토리에 silo 저장소(`{프로젝트}-ccsilo`)를 생성하고 선택된 파일을 복사
3. 원본을 silo 저장소를 가리키는 symlink로 교체
4. 선택적으로 GitHub private 저장소 생성

```
my-project/
  .claude/ → ../my-project-ccsilo/.claude/
  docs/    → ../my-project-ccsilo/docs/
  src/
  ...

my-project-ccsilo/    ← 별도 git 저장소
  .claude/
  docs/
  .ccsilorc
  setup-symlinks.sh
```

## 설정

프로젝트 루트에 `.ccsilorc`를 생성하여 커스터마이즈:

```yaml
include:
  - .claude/
  - docs/
  - .cursorrules

exclude:
  - CLAUDE.md

suffix: "-ccsilo"
```

| 키 | 기본값 | 설명 |
|----|--------|------|
| `include` | `.claude/`, `docs/` | silo로 옮기고 symlink로 연결할 파일/디렉토리 |
| `exclude` | `CLAUDE.md` | 메인 프로젝트에 유지할 파일 |
| `suffix` | `-ccsilo` | silo 저장소 이름 접미사 |

## 라이선스

MIT
