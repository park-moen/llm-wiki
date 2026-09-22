# AI 개발에서 Markdown 편집과 Git Diff 검토 워크플로우

> Sources: [Orca 리뷰·편집·브라우저 작업 흐름](../orca/orca-review-editing-and-browser-workflow.md); [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md)
> Origin: 사용자가 제공한 ChatGPT 응답 `markdown-editor-workflow.md` (not stored in `raw/`)
> Archived: 2026-09-21

## Overview

Claude Code·Codex가 수정한 `README.md`, `CLAUDE.md`, `AGENTS.md`, `docs/**/*.md`를 사람이 자연스럽게 읽고, Git diff로 변경 의미를 검토한 뒤 필요한 부분만 stage하는 workflow 제안이다. Markdown 작성 경험과 Git 검토 경험을 한 도구에 억지로 합치기보다, Markdown 중심 도구와 Git·code review에 강한 IDE의 역할을 나누는 방식을 출발점으로 둔다. 이 문서는 제품 기능을 검증한 보고서가 아니라, 실제 repository에서 검증할 운영 가설을 보관한 snapshot이다.

## 목표와 역할 분리

핵심 요구는 Markdown을 편하게 읽고 직접 고치면서도, AI가 만든 변경을 Git diff로 빠르게 검토하고 hunk 단위로 선별하는 것이다. 제안된 기본 구성은 다음과 같다.

| 역할 | 우선 도구 | 맡길 일 |
| --- | --- | --- |
| Markdown 읽기·가벼운 수정 | Obsidian | Live Preview로 문서 흐름을 읽고 작은 수정 반영 |
| 복잡한 Git·code 검토 | VS Code 또는 Cursor | 큰 diff, hunk stage·revert, conflict, history 확인 |
| 문서 초안·갱신 | Claude Code 또는 Codex | 구현에 근거한 문서 초안, 계획·ADR 갱신, 변경 요약 |

Obsidian은 개발 repository 자체를 Vault로 열어 `README.md`, `CLAUDE.md`, `AGENTS.md`, `docs/`를 별도 복사본 없이 다루는 방안을 우선 검토한다. Obsidian Git 또는 GitUS는 changed file 표시, diff 품질, hunk stage, discard, conflict 처리와 성능을 실제 사용 환경에서 비교해 하나만 주력으로 정한다. 자동 commit·push는 review 경계를 흐릴 수 있으므로 기본으로 켜지 않는다.

## 검토 루프

```text
Claude Code / Codex가 Markdown 수정
→ Obsidian에서 문서 흐름과 의미 검토
→ Git diff로 변경 범위·구조·사실성 확인
→ 필요한 부분을 직접 보정
→ 검토한 hunk만 stage
→ manual commit
```

diff에서는 문장 자체보다 다음 세 종류의 변화를 분리해 읽는다.

- **구조 변경**: heading, section 이동, 목록 계층, code block이 의도대로 바뀌었는지 확인합니다.
- **의미 변경**: architecture 설명이 실제 구현과 맞는지, 결정되지 않은 내용을 확정된 사실처럼 쓰지 않았는지 확인합니다.
- **noise 변경**: 줄바꿈, whitespace, 문서 전체 재서식처럼 review를 어렵게 만드는 변경은 피합니다.

관련 없는 변경은 unstaged로 남기고, 검토한 변경만 hunk 단위로 stage합니다. 이는 Orca의 combined diff·hunk stage 중심 review loop와 같은 원칙이다. [Orca 리뷰·편집·브라우저 작업 흐름](../orca/orca-review-editing-and-browser-workflow.md)

## Agent의 Markdown 수정 규칙

Agent 지침에는 문서의 정본과 변경 범위를 보호하는 규칙을 둡니다.

```markdown
Markdown 문서를 수정할 때 기존 구조를 유지합니다.
명시적으로 요청하지 않은 section은 다시 쓰지 않습니다.
변경은 가능한 한 작게 만들고 formatting-only 변경을 피합니다.
heading hierarchy, 기존 용어, code identifier와 file path를 보존합니다.
현재 repository 구현으로 뒷받침할 수 있는 내용만 문서에 반영합니다.
```

이 규칙은 Agent가 문서를 전면 재작성하는 경향을 줄이는 데 목적이 있지만, 그 자체로 사실성을 보장하지는 않는다. 따라서 실제 code와 diff를 사람이 확인하는 단계가 필요하다. [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md)

## 도입 전 검증 항목

새 설정을 repository 전체에 적용하기 전에 임시 Markdown 파일 하나로 다음을 확인합니다.

1. Markdown 파일이 정상적으로 열리고 Live Preview가 필요한 수준인지 확인합니다.
2. AI가 바꾼 파일이 Git panel과 diff에 바로 나타나는지 확인합니다.
3. file·hunk 단위 stage와 discard가 가능한지, 기존 IDE의 Git 상태와 일치하는지 확인합니다.
4. `.obsidian/` 생성 여부와 추적 정책을 repository 규칙에 맞게 결정합니다.
5. `markdownlint-cli2`, Prettier의 Markdown format-on-save가 noise diff를 만들지 않는지 확인합니다.

특히 Prettier나 lint rule은 문서 내용보다 table 정렬, 줄바꿈, wrapping을 크게 바꿀 수 있다. Markdown format-on-save는 끈 상태에서 먼저 평가하고, 필요한 규칙만 선택적으로 적용한다.

## 초기 선택안과 미결정 사항

초기 선택안은 `Obsidian + Obsidian Git + 기존 VS Code/Cursor` 조합이다. 그러나 이는 최종 결론이 아니라 plugin의 diff·stage·discard 경험과 안정성을 확인하기 위한 시작점이다.

결정이 필요한 항목은 다음과 같다.

- repository root를 Obsidian Vault로 직접 열어도 되는가
- `.obsidian/`을 ignore할지 공유할지
- Obsidian Git과 GitUS 중 어떤 plugin을 주력으로 할지
- Markdown format-on-save와 markdownlint를 어느 범위까지 적용할지
- 문서 변경 commit을 code 변경 commit과 분리할지

## See Also

- [Orca 리뷰·편집·브라우저 작업 흐름](../orca/orca-review-editing-and-browser-workflow.md)
- [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md)
