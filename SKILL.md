---
name: vault-entry
description: >-
  Find and navigate the user's AI-first Obsidian vault from ANY working directory — a repo,
  a worktree, an Orca worker spawned outside the vault. Load it whenever a task involves the
  vault: reading notes, project or knowledge notes, the automation inventory, meeting or
  research write-ups, or recording a result into the vault. It resolves the vault root
  machine-independently (when the vault is in use, this skill sits inside it as a submodule and
  is symlinked into each tool's skills path), then hands off to the vault's own entry documents,
  which are the SSOT. Triggers: "볼트", "옵시디언", "Obsidian vault", "노트에 정리해줘",
  "볼트에서 찾아줘", 볼트 노트 인용·검색, or any doubt about where the user's notes live.
---

# 볼트 진입 (vault-entry)

> 볼트 밖에서 뜬 세션이 볼트를 **찾아가서** 규칙을 제대로 로드하게 하는 진입 스킬.
> 규칙 본문은 여기 복제하지 않는다 — 볼트 안 `AGENTS.md` 와 `00_Vault/rules/` 의 규칙 노트가 SSOT다.
> (볼트 루트에서 시작한 세션은 이미 `AGENTS.md` 를 읽으므로 이 스킬이 필요 없다.)

## 1. 볼트 루트 찾기

볼트를 쓰는 기계에서 이 스킬은 볼트 안 `20_Agents/skills/vault-entry/` 에 서브모듈로 들어 있고,
도구 스킬 경로(`~/.claude/skills/` 등)는 그곳을 가리키는 **심볼릭링크**다. 링크를 풀고 위로 올라가며
`AGENTS.md` 와 `00_Vault/` 가 함께 있는 폴더를 찾는다 — 홈 절대경로나 고정 깊이를 가정하지 않는다.

```bash
skill_file="<이 SKILL.md 가 로드된 경로>"
d="$(cd "$(dirname "$(readlink -f "$skill_file")")" && pwd -P)"
while [ "$d" != "/" ] && ! { [ -f "$d/AGENTS.md" ] && [ -d "$d/00_Vault" ]; }; do d="$(dirname "$d")"; done
[ "$d" != "/" ] && vault_root="$d" && echo "$vault_root"
```

- `readlink -f` 가 없는 환경이면 `python3 -c 'import os,sys;print(os.path.realpath(sys.argv[1]))' "$skill_file"`.
- 로드 경로를 모를 때는 도구 스킬 경로에서 역추적한다:
  `for p in ~/.claude/skills ~/.agents/skills ~/.codex/skills ~/.cursor/skills; do readlink -f "$p/vault-entry" 2>/dev/null; done`
- 스킬을 볼트 밖에 단독으로 clone 해 둔 기계면 위 탐색이 실패한다. 그때나 파일시스템 접근이 없는
  표면이면 **추측하지 말고** 사용자에게 볼트 경로나 필요한 노트를 요청한다.

## 2. 찾은 다음 (규칙 로드 순서)

1. `$vault_root/AGENTS.md` — 에이전트 공통 진입점, 도메인 표와 절대 규칙.
2. `$vault_root/00_Vault/_index.md` — 볼트 규칙 노트의 목록. 노트를 쓰기 전에는 `노트 작성 규칙` 을 읽는다.
3. 작업 도메인의 `_index.md` 와 그 「규칙」이 가리키는 `rules/` 의 규칙 노트.

전체 트리를 읽지 않는다. 지식을 찾을 때는 검색이 먼저다 — 예: `rg -n "검색어" --glob '*.md' "$vault_root"`.
후보를 찾으면 소속 `_index.md` 로 맥락을 확인한다.

## 3. 읽기 전에 알아야 할 최소 규칙

원본을 읽기 전 실수를 막기 위한 요약이며, 충돌 시 `AGENTS.md` 가 우선한다.

- 볼트는 공개 저장소이고 일부 도메인만 추적한다. 로컬 전용 도메인(`AGENTS.md` 의 도메인 표에서 git
  「무시」)은 다른 기계에 없을 수 있다. 추적 문서에 비밀·식별자·개인정보·회사 내부 정보를 넣지 않는다.
- 모든 노트에 프론트매터가 필수다: `title`·`created`·`updated`·`tags`·`source`.
- 새 노트를 만들기 전에 같은 주제를 검색한다 — 있으면 갱신하고 `updated` 를 고친다.
- 노트 생성·삭제·이동은 소속 `_index.md` 갱신까지가 한 작업이다.
- 본문은 표준 마크다운 + 옵시디언 위키링크까지만. 도구 전용 문법 금지.
- 새 최상위 폴더를 만들지 않는다.

## 4. 볼트 밖 세션이 볼트에 쓸 때

- 사용자가 이번 지시에서 볼트 기록을 요청한 경우에만 쓴다. "정리해 두면 좋겠다" 수준의 추정으로
  볼트에 노트를 만들지 않는다 — 찾은 내용은 세션에서 보고하고 기록 여부는 사용자가 정한다.
- 쓸 때는 §2 로 규칙을 실제 로드한 뒤 §3 항목을 모두 지킨다. 개정 이력을 어디에 남기는지는 볼트의
  `변경 이력 규칙` 을 따른다.
- 반대 방향(볼트 안 세션이 볼트 밖을 수정)의 경계는 `vault-write-boundary` 스킬이 관장한다.
