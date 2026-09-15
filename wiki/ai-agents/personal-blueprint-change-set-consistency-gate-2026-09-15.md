# Personal Blueprint 변경 세트와 정합성 종료 Gate 설계

> Sources: [교체 가능한 Personal Blueprint Harness 설계](replaceable-personal-blueprint-harness-architecture-2026-09-15.md); [개인 AI Engineering Harness 주말 구축 시뮬레이션](personal-ai-engineering-harness-weekend-simulation-2026-09-15.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md); [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md); [효과적인 Software Design Document 작성법](../software-career/effective-software-design-document.md)
> Archived: 2026-09-15

## Overview

Personal Blueprint에서 기획 문서를 조금씩 수정할 때마다 `docs/planning/` 전체를 다시 검사하면 feedback은 빠르지만 작업 흐름과 token 비용이 커진다. 반대로 마지막까지 아무것도 확인하지 않으면 요구사항, 기능, 화면, 비즈니스 규칙과 테스트 문서 사이의 drift가 누적된다. 수정 중에는 변경 세트와 영향 후보만 가볍게 기록하고, Phase 또는 하나의 기획 변경을 닫을 때 전체 정합성 검사를 한 번 수행하는 **종료 gate**를 둔다. 완료 표시는 checkbox와 굵은 상태 문구로 사람이 읽기 쉽게 보여주되, 실제 통과 여부는 validation report와 상태 hash로 판정한다.

## 핵심 원칙

```text
수정 중
→ 변경 세트에 dirty file·관련 ID·변경 이유만 누적
→ 전체 검사는 실행하지 않음

기획 또는 변경 종료
→ 영향 문서 동기화
→ 전체 planning folder 기계 검사
→ 의미 정합성 review
→ 사람 확인
→ 검토 완료 상태 기록
```

편집 단계는 저렴하고 빠르게 유지하고, 변경을 완료 상태로 전환할 때만 비싼 검증을 집중한다.

## 변경 세트를 작업 단위로 사용한다

문서 한 줄을 바꿀 때마다 별도 검증 작업을 만들지 않는다. 같은 의도를 가진 수정들을 하나의 change set으로 묶는다.

```yaml
change_set:
  id: CHG-2026-09-15-01
  title: 회원 상태 필터 기본값 변경
  status: OPEN
  reason: 운영자는 활성 회원을 먼저 확인함
  related:
    - "회원 상태 필터 요구사항 (REQ-014)"
    - "회원 상태 필터 기능 (FEAT-009)"
    - "관리자 회원 목록 화면 (SCR-003)"
  dirty_files:
    - docs/planning/02-prd.md
    - docs/planning/04-spec.md
    - docs/planning/06-screen-spec.md
  pending_questions:
    - 기존 URL에 filter가 없을 때 active를 적용할지 결정 필요
```

변경 세트 상태는 다음 순서로 이동한다.

```text
OPEN
→ READY_TO_VALIDATE
→ VALIDATING
   ├─ 실패 → BLOCKED
   └─ 통과 → VALIDATED
→ CLOSED
```

- `OPEN`: 자유롭게 편집하며 영향 후보를 누적한다.
- `READY_TO_VALIDATE`: 사용자가 기획 또는 변경을 마쳤다고 선언했다.
- `VALIDATING`: 동기화와 정합성 검사가 실행 중이다.
- `BLOCKED`: 누락, 충돌이나 미결정 사항이 발견됐다.
- `VALIDATED`: 기계 검사와 필요한 review가 통과했다.
- `CLOSED`: 사람이 결과를 확인하고 변경을 닫았다.

## 매번 전체 검사하지 않는 Hook 구조

### `PostToolUse`: 기록만 한다

기획 문서가 수정될 때는 전체 검사를 실행하지 않는다.

- 변경된 file을 `dirty_files`에 추가한다.
- 문서에서 발견한 `REQ`, `FEAT`, `SCR`, `BR`, `TC`를 관련 ID 후보로 기록한다.
- 새 ID, 삭제된 ID와 관계 변경을 가벼운 event log로 남긴다.
- 이미 기록된 file과 ID는 중복 추가하지 않는다.

이 단계의 목적은 정답을 판정하는 것이 아니라 마지막 검사의 범위를 잃지 않는 것이다.

### 일반적인 `Stop`: 가볍게 알린다

변경 세트가 `OPEN`이면 대화를 끝낼 때 전체 검사를 강제하지 않는다. 대신 다음 상태만 알린다.

```text
⚠️ 열린 기획 변경이 있습니다.
- 변경 세트: CHG-2026-09-15-01
- 수정 문서: 3개
- 상태: 정합성 검토 대기
```

사용자가 아직 수정 중인데 매 turn마다 전체 검사를 실행하거나 종료를 막지 않는다.

### Finalize 요청 시: 전체 검사를 실행한다

사용자가 “기획 완료”, “변경 마무리”, “정합성 검사”처럼 변경 종료를 명시하거나 다음 Phase로 이동할 때 `READY_TO_VALIDATE`로 전환한다. 이때만 전체 종료 gate를 실행한다.

```text
변경 종료 요청
→ 영향 문서 동기화
→ 전체 folder 기계 검사
→ 의미 정합성 review
→ 검증 report 생성
→ 사람 확인
→ CLOSED
```

`Stop` hook은 `finalize_requested: true`인 경우에만 검증 실패를 이유로 종료를 차단한다. 같은 상태에서 무한히 차단하지 않도록 변경 집합 hash, 재시도 상한과 사람에게 넘기는 탈출 경로를 함께 둔다.

## 종료 시 수행할 두 종류의 검사

### 기계적 정합성 검사

Script나 validator가 재현 가능하게 판정할 수 있는 항목이다.

- ID 형식과 중복 여부
- 문서에서 참조한 ID의 실제 존재 여부
- `REQ → FEAT → SCR → BR → TC` 연결 누락
- 삭제한 ID를 참조하는 고아 link
- 필수 section과 metadata 누락
- Traceability Matrix와 원본 문서의 불일치
- 같은 이름에 서로 다른 ID가 붙은 후보
- `pendingChanges`와 열린 change set 잔존 여부
- 허용 범위 밖 파일 변경 여부

Dirty file만 검사하는 것으로 끝내지 않는다. 종료 시에는 영향 문서를 먼저 동기화한 뒤 `docs/planning/` 전체를 한 번 검사해 예상하지 못한 역방향 참조도 찾는다.

### 의미 정합성 Review

기계적으로 확정하기 어려운 항목은 AI review와 사람 확인으로 분리한다.

- PRD의 사용자 의도와 Spec의 동작이 같은가?
- Screen Spec과 와이어프레임이 같은 흐름을 표현하는가?
- 비즈니스 규칙 변경이 예외 흐름과 테스트에 반영됐는가?
- 같은 용어가 문서마다 다른 의미로 사용되지 않는가?
- 변경 이유와 decision log가 실제 선택을 설명하는가?
- 구현 착수 전에 결정해야 할 질문이 남아 있지 않은가?

AI reviewer의 “문제없음”은 결정적 통과가 아니다. 기계 검사 결과, 미결정 사항과 사람이 확인한 범위를 함께 남긴다.

## 사람이 읽을 수 있는 검토 상태 표시

`[]` checkbox와 `**굵은 상태**`는 빠르게 상태를 확인하는 데 유용하다. 다만 사람이 직접 문자를 바꾸거나 Agent가 근거 없이 완료 표시를 할 수 있으므로 표시 자체를 gate의 근거로 사용하지 않는다.

### 검토 대기

```markdown
> [!warning] 정합성 검토 대기
> 변경 세트 `CHG-2026-09-15-01`이 열려 있습니다.

- [ ] 영향 문서 동기화
- [ ] 전체 folder 기계 검사
- [ ] 의미 정합성 review
- [ ] 미결정 사항 확인

**상태: ⚠️ 검토 대기**
```

### 검토 완료

```markdown
> [!success] 정합성 검토 완료
> 변경 세트 `CHG-2026-09-15-01`을 검증했습니다.

- [x] 영향 문서 동기화
- [x] 전체 folder 기계 검사
- [x] 의미 정합성 review
- [x] 미결정 사항 확인

**상태: ✅ 검토 완료**

- 검증 보고서: `validation/CHG-2026-09-15-01.md`
- 검증 대상 hash: `<change-set-hash>`
- 남은 경고: 없음
```

Obsidian에서는 callout으로 눈에 띄게 표시하고, 다른 Markdown viewer에서는 checkbox와 굵은 상태가 fallback으로 남는다.

## 표시와 실제 상태를 분리한다

완료 상태의 정본은 문서에 적힌 `✅`가 아니라 별도 상태 파일과 validation report다.

```yaml
change_set:
  id: CHG-2026-09-15-01
  status: VALIDATED
  validated_at: 2026-09-15
  validated_hash: <change-set-hash>
  report: validation/CHG-2026-09-15-01.md
  mechanical_checks:
    status: passed
  semantic_review:
    status: reviewed
  human_checkpoint:
    status: confirmed
```

문서를 다시 수정해 hash가 달라지면 hook은 기존 완료 표시를 신뢰하지 않고 상태를 `OPEN` 또는 `STALE`로 되돌린다.

```text
검토 완료 hash ≠ 현재 변경 집합 hash
→ **상태: ⚠️ 검토 결과 만료**
→ checkbox를 다시 미완료 상태로 표시
```

이 구조가 있어야 Agent가 checkbox만 `[x]`로 바꾸고 검증을 건너뛰는 일을 발견할 수 있다.

## Personal Blueprint Core와 Adapter의 책임

정합성 종료 gate는 Superpowers, gstack이나 Matt Pocock Skills adapter에 두지 않는다. Provider를 교체해도 동일한 완료 조건이 유지되도록 Personal Blueprint core가 소유한다.

```text
외부 Skill Adapter
└─ 문서 생성·review·질문·QA 수행

Personal Blueprint Core
├─ Change set 상태 관리
├─ ID와 artifact 계약
├─ 전체 정합성 validator
├─ Validation report
└─ 완료 상태와 hash

Hook
└─ 변경 감지와 종료 gate 실행
```

새 Skill이 기획 문서를 수정하면 adapter 종류와 관계없이 같은 `dirty_files`와 ID event가 기록된다. 따라서 provider별로 별도 정합성 체계를 만들 필요가 없다.

## 권장 명령 흐름

실제 명령 이름은 host adapter가 결정하되 core의 의미는 다음처럼 유지한다.

```text
change start "회원 상태 필터 기본값 변경"
→ Change set OPEN

기획 문서 여러 차례 수정
→ Dirty file과 관련 ID만 누적

change status
→ 열린 변경과 검토 대기 상태 표시

change finalize
→ 동기화 + 전체 검사 + review + report

change close
→ Human checkpoint 후 CLOSED
```

Claude Code, Codex 또는 새로운 Skill 환경에서는 이 capability를 각 host의 command와 Skill 형식으로 연결한다. Core는 특정 slash command 이름에 의존하지 않는다.

## 종료 Gate 통과 조건

다음을 모두 만족해야 하나의 기획 변경을 완료로 표시한다.

- 영향 문서가 모두 반영됐다.
- 전체 planning folder의 기계적 정합성 검사가 통과했다.
- 의미 충돌 후보를 review했다.
- 미결정 사항이 없거나 명시적으로 보류됐다.
- 검증 보고서와 대상 hash가 남았다.
- 사람이 결과와 남은 경고를 확인했다.
- 검증 이후 문서가 다시 바뀌지 않았다.

검증 이후 문서가 수정되면 완료 상태는 자동으로 만료돼야 한다.

## 최종 판단

매번 전체 folder를 검사하는 대신 **수정 중에는 변경 정보를 누적하고, 변경을 닫을 때 전체 정합성을 한 번 검증하는 구조**가 비용과 안전성의 균형이 좋다. Checkbox, 굵은 상태와 Obsidian callout은 사람이 빠르게 상태를 이해하게 하고, validation report와 hash는 Agent가 표시만 조작해 완료를 가장하지 못하게 한다. 이 종료 gate를 Personal Blueprint core가 소유하면 어떤 외부 Skill로 교체하더라도 같은 정합성 기준을 유지할 수 있다.

## See Also

- [Superpowers를 지속 사용하는 Harness 운영 전략](superpowers-continuous-use-harness-strategy.md)
- [Claude Code Hooks](claude-code-hooks.md)
- [Matt Pocock Skills의 Repository 설정 방식](matt-pocock-skills-repository-setup.md)
