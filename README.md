---
title: vault-entry
created: 2026-10-16
updated: 2026-10-16
tags: [domain/지식관리, area/문서관리]
source: ai
---

# vault-entry

> 볼트 밖에서 뜬 에이전트 세션(다른 저장소, 워크트리, 오케스트레이션 워커)이 사용자의 AI-first Obsidian 볼트를 찾아가 볼트 자체의 규칙 문서를 로드하게 하는 진입 스킬이다.

## 하는 일

- 볼트 루트를 홈 경로나 고정 깊이에 기대지 않고 스킬 파일의 실제 경로에서 위로 올라가며 찾는다 (`AGENTS.md` + `00_Vault/`).
- 규칙은 복제하지 않고 볼트의 `AGENTS.md` · `00_Vault/` 로 넘긴다. 실수를 막는 최소 규칙만 요약한다.

## 설치

에이전트 도구의 스킬 경로에 clone 하거나 링크한다.

```bash
git clone https://github.com/s12171934/vault-entry.git ~/.claude/skills/vault-entry
# 다른 도구는 그 도구의 스킬 경로에 링크한다. 예: ln -s ~/.claude/skills/vault-entry ~/.codex/skills/vault-entry
```

- 볼트 안 `20_Agents/skills/vault-entry/` 에 서브모듈로 두고 도구 스킬 경로에 링크하면 볼트 루트를 스스로 찾는다. 볼트 밖에 단독으로 clone 하면 볼트 경로를 사용자에게 묻는다.

## 구성과 공개 면

- [[20_Agents/skills/vault-entry/SKILL|SKILL.md]] — 스킬 본문. 형식은 에이전트 도구가 정한다.
- 공개 면은 스킬 이름(`vault-entry`)뿐이다. 다른 스킬은 이 이름으로만 이 스킬을 가리킨다.
