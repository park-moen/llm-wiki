# 개인 AI Engineering Harness 주말 구축 시뮬레이션

> Sources: [Superpowers를 지속 사용하는 Harness 운영 전략](superpowers-continuous-use-harness-strategy.md); [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md); [gstack으로 AI 개발 Workflow 이해하기](gstack-ai-engineering-workflow.md); [Matt Pocock Skills의 Repository 설정 방식](matt-pocock-skills-repository-setup.md); [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md); [Claude Code Hooks](claude-code-hooks.md)
> Archived: 2026-09-15

## Overview

주말의 목표는 Superpowers, gstack과 Matt Pocock Skills를 모두 설치한 거대한 자동 개발 시스템을 만드는 것이 아니다. 회사 Blueprint를 기획 정본으로 유지하면서, 세 도구의 강점만 골라 하나의 작은 기능을 계획·구현·검증하는 **개인용 최소 harness**를 시험한다. Superpowers는 중심 개발 흐름, gstack은 engineering review와 browser QA, Matt Pocock Skills는 진단·설계·code review 도구로 사용한다. 개인 hook은 이 산문 workflow를 프로젝트 규칙, 권한과 기계적 검증으로 보강한다.

## 이번 실험에서 확인할 질문

1. Blueprint 산출물을 다시 기획하지 않고 구현 단계로 넘길 수 있는가?
2. 서로 다른 Skill 묶음이 같은 역할을 중복 수행하지 않는가?
3. Agent가 검증을 빠뜨려도 hook이 완료를 차단할 수 있는가?
4. Test 결과뿐 아니라 변경 범위와 검증 자산의 훼손도 확인할 수 있는가?
5. 전체 절차가 작은 실무 작업에도 감당할 만한 비용인가?

목표는 첫 주말에 보편적인 정답을 만드는 것이 아니라, 한 번의 실제 Run에서 유지할 부분과 버릴 부분을 찾는 것이다.

## 역할과 정본을 먼저 분리한다

```text
Blueprint
└─ 문제·요구사항·domain·화면·와이어프레임·추적성의 정본

개인 Harness Router
└─ 작업 유형과 위험도에 따라 필요한 Skill만 선택

Superpowers
└─ 구현 계획·작은 TDD 작업·review 흐름

gstack
└─ engineering 관점 재검토·실제 browser QA·회고

Matt Pocock Skills
└─ 어려운 bug 진단·module 설계·독립 code review

개인 Hook·Script
└─ 권한 차단·변경 감지·verify·완료 증거

ITNew Workflow
└─ Jira·commit·MR의 회사 규칙
```

기획 내용은 `docs/planning/`을 정본으로 둔다. 구현 Skill은 이 문서를 읽지만 PRD, domain model과 screen spec을 임의로 다시 정의하지 않는다. 기획 변경이 필요하면 구현 중 조용히 고치지 않고 Blueprint의 변경 관리 절차로 되돌린다.

## 첫 버전에 포함할 Skill

### Superpowers에서 빌릴 중심 흐름

| Skill 또는 단계 | 용도 | 첫 실험의 적용 |
|---|---|---|
| `writing-plans` | 기능을 작은 구현 단위로 분해 | 사용 |
| `executing-plans` | 확정된 plan을 순서대로 실행 | 사용 |
| `test-driven-development` | RED–GREEN feedback loop | 사용 |
| `systematic-debugging` | 원인 불명 bug 조사 | Bug preset에서만 사용 |
| `requesting-code-review` | 구현과 review 역할 분리 | 사용 |
| `receiving-code-review` | Review 의견을 재현한 뒤 반영 | 사용 |
| `verification-before-completion` | 완료 전 증거 확인 | 사용하되 hook으로 보강 |
| `using-git-worktrees` | 독립 작업 격리 | 첫 Run에서는 제외 |
| `brainstorming` | 문제와 solution 탐색 | Blueprint에 빈칸이 있을 때만 사용 |

### gstack에서 선택할 부분

| Skill | 용도 | 첫 실험의 적용 |
|---|---|---|
| `/plan-eng-review` | Data flow, 실패 경로와 test 계획 점검 | 사용 |
| `/review` | 구현 후 production 관점 review | 선택 사용 |
| `/qa-only` | Code를 바꾸지 않고 실제 browser 흐름 검사 | UI가 있는 경우 사용 |
| `/retro` | 반복된 실패와 workflow 마찰 기록 | Run 종료 후 사용 |
| `/office-hours`, `/plan-ceo-review` | 문제·제품 범위 탐색 | Blueprint와 겹치므로 제외 |
| Design 관련 Skill | 시각 설계 | Blueprint·Claude 디자인과 겹치므로 제외 |
| `/ship`, `/land-and-deploy` | Commit·push·PR·배포 | ITNew 규칙과 충돌하므로 제외 |

### Matt Pocock Skills에서 선택할 부분

| Skill | 용도 | 첫 실험의 적용 |
|---|---|---|
| `setup-matt-pocock-skills` | Issue tracker와 domain 문서 위치 연결 | Repository마다 처음 한 번 |
| `diagnosing-bugs` | 재현→가설→계측→수정 loop | Bug preset에서 사용 |
| `tdd` | 작은 vertical slice 구현 | Superpowers TDD와 하나만 주도권을 갖도록 연결 |
| `code-review` | Spec과 code 기준을 분리한 review | 사용 |
| `codebase-design` | Module 경계와 interface 점검 | 설계 위험이 큰 작업에서만 사용 |
| `research` | 신뢰도 높은 자료 조사 | 외부 근거가 꼭 필요한 경우만 사용 |
| `handoff` | 다음 세션에 작업 상태 전달 | 작업이 하루를 넘길 때 사용 |
| `to-spec`, `to-tickets`, `domain-modeling` | Spec·ticket·domain 생성 | Blueprint와 `itnew-plan`이 있으므로 제외 |
| `prototype` | 버릴 수 있는 설계 실험 | Blueprint 와이어프레임으로 답할 수 없을 때만 사용 |

Superpowers의 `test-driven-development`와 Matt Pocock의 `tdd`처럼 목적이 같은 Skill을 연속 호출하지 않는다. 하나를 주 실행자로 정하고 다른 하나는 빠진 기준을 보완하는 참고 자료로만 사용한다.

## 최소 Harness 구조

첫 주말에는 plugin 전체를 만들지 않고 다음 정도의 작은 구조로 시작한다.

```text
.agent-harness/
├── README.md                 # 목적, 정본과 책임 경계
├── routes.md                 # feature·bug·UI 작업별 Skill 선택
├── upstream-lock.md          # 사용한 Skill source와 version
├── policies/
│   ├── permissions.md        # push·merge·배포·외부 변경 승인
│   └── planning-source.md    # Blueprint 정본과 변경 절차
├── scripts/
│   ├── verify.sh             # test·lint·build 통합
│   ├── check-scope.sh        # 예상 밖 file과 기획 문서 변경 확인
│   └── collect-evidence.sh   # command·exit code·diff 요약 수집
└── runs/
    └── <date>-<task>/run.md  # 실제 Run과 회고
```

Host별 Skill 형식과 hook event는 바뀔 수 있으므로 핵심 정책과 검증 script는 특정 AI 도구 밖에 둔다. Claude Code나 Codex에는 이 파일을 읽고 실행하는 얇은 adapter만 둔다.

## 개인 Hook 설계

### `SessionStart`: Context 주입

- `docs/planning/`과 관련 `REQ`, `FEAT`, `SCR` 위치를 알려준다.
- Repository의 build·test·lint command를 알려준다.
- 현재 branch, 기준 branch와 미완료 변경을 표시한다.
- 고정한 upstream Skill version과 개인 override를 표시한다.

이 단계는 방향을 알려주는 산문 계층이다. Context 주입만으로 규칙이 강제되지는 않는다.

### `PreToolUse`: 위험 행동 차단

- 승인 없는 `git push`, `git merge`, `git tag`와 배포를 차단한다.
- `git reset --hard`, force push와 branch 삭제를 차단한다.
- Blueprint 산출물의 직접 수정은 경고하거나 변경 관리 절차로 돌린다.
- Secret 출력과 허용하지 않은 외부 서비스 변경을 차단한다.

기계적으로 판정할 수 있는 명령만 막는다. “설계가 나쁘다”처럼 판단이 필요한 항목은 hook에 넣지 않는다.

### `PostToolUse`: Drift와 빠른 feedback

- Code 수정 후 관련 test 또는 빠른 compile 검사를 안내한다.
- 기존 test 삭제·비활성화와 검사 범위 축소를 감지한다.
- `docs/planning/` 변경이 발생하면 Blueprint drift 후보로 기록한다.
- 예상 범위를 벗어난 file이 바뀌면 Run 기록에 남긴다.

모든 file 저장마다 전체 build를 실행하면 작업 흐름이 지나치게 느려진다. Post hook은 빠른 검사와 기록에 집중하고 전체 검증은 Stop 단계에서 수행한다.

### `Stop`: 완료 조건 재판정

Agent의 “완료했습니다”라는 문장과 무관하게 다음 조건을 검사한다.

```text
Target test 성공
+ 전체 verify 성공
+ 기존 test 삭제·비활성화 없음
+ 계획 밖 변경 설명 완료
+ 필요한 browser QA 증거 존재
+ 미해결 review 의견 기록
→ 완료 후보
```

실패하면 command 원문, exit code와 수정 가능한 원인을 돌려준다. 같은 변경 상태를 무한히 차단하지 않도록 변경 집합 hash, 재시도 상한과 사람에게 넘기는 탈출 경로를 둔다.

## 작업 유형별 Route

### 이미 기획된 새 기능

```text
Blueprint 관련 ID와 화면 확인
→ Superpowers writing-plans
→ gstack /plan-eng-review
→ Human plan checkpoint
→ TDD 기반 구현
→ Matt Pocock code-review
→ UI가 있으면 gstack /qa-only
→ Stop hook 전체 verify
→ ITNew commit·MR 절차
```

### 원인 불명의 Bug

```text
재현 조건과 영향 범위 확인
→ systematic-debugging 또는 diagnosing-bugs 중 하나 선택
→ 실패하는 regression test
→ 최소 수정
→ code-review
→ Stop hook 전체 verify
```

### UI·사용자 흐름 변경

```text
Blueprint screen spec·wireframe 확인
→ gstack /plan-eng-review
→ 구현과 component test
→ 접근성·반응형 확인
→ gstack /qa-only
→ Screenshot·DOM·URL evidence
→ Stop hook 전체 verify
```

## 실제로 돌려 볼 기능 시뮬레이션

첫 Run은 production 배포가 필요 없는 작은 기능을 고른다. 예시는 **관리자 사용자 목록에 활성 상태 필터 추가**다. Backend query, 화면 state와 browser QA가 모두 있어 여러 계층을 시험할 수 있지만 변경 범위는 작다.

### 입력 계약

- 관련 Blueprint `REQ`, `FEAT`, `SCR` ID
- 포함 범위: 활성·비활성·전체 필터, URL query 유지, 빈 결과 표시
- 제외 범위: 권한 체계 변경, database schema 변경, production 배포
- 완료 기준: API test, UI test, 전체 verify, local browser 흐름 통과

### 예상 Run

```text
1. SessionStart가 관련 Blueprint 문서와 검증 command를 주입
2. writing-plans가 API·UI를 작은 vertical slice로 분해
3. /plan-eng-review가 query 누락, 잘못된 enum과 빈 결과를 점검
4. 사람이 plan과 제외 범위를 승인
5. 실패하는 backend·frontend test 작성
6. 최소 구현 후 target test 실행
7. code-review가 spec 누락과 code quality를 각각 검사
8. /qa-only가 전체→활성→비활성 전환과 URL 복원을 확인
9. Stop hook이 전체 verify와 test 훼손 여부를 재판정
10. 결과를 run.md에 저장하고 /retro로 마찰을 기록
```

첫 Run에서는 Worktree, 여러 구현 agent, 자동 commit, push와 배포를 사용하지 않는다. 한 흐름을 사람이 이해할 수 있어야 병렬성과 자동화를 추가할 근거가 생긴다.

## 주말 실행 계획

### 토요일 오전: 조합 설계

1. 설치된 세 Skill 묶음의 실제 경로와 version을 기록한다.
2. 겹치는 Skill의 주도권을 표로 정한다.
3. Blueprint·ITNew Workflow와 충돌하는 command를 제외한다.
4. Feature, bug, UI 세 가지 route만 작성한다.

### 토요일 오후: 최소 Gate 구현

1. Repository의 단일 `verify` command를 만든다.
2. 위험 Git·배포 command를 막는 Pre hook을 만든다.
3. Test 삭제와 기획 문서 drift를 찾는 검사 script를 만든다.
4. Stop hook에 전체 verify와 loop guard를 연결한다.

### 일요일 오전: 작은 기능 한 번 실행

1. 실제 저위험 Issue 하나를 고른다.
2. 처음부터 끝까지 단일 agent로 실행한다.
3. 각 Skill의 입력·출력, 사람이 개입한 지점과 command 결과를 기록한다.
4. 자동 commit·push·배포 없이 local 결과까지만 확인한다.

### 일요일 오후: 회고와 축소

다음 표를 채우고 다음 버전을 결정한다.

| 질문 | 기록할 내용 |
|---|---|
| 실제로 도움 된 Skill | 빠뜨릴 뻔한 판단이나 bug |
| 중복된 Skill | 같은 질문·문서를 다시 만든 단계 |
| 잘못 차단한 hook | 정상 작업을 막은 조건 |
| 놓친 실패 | hook이나 verify가 잡지 못한 문제 |
| Human checkpoint | 사람이 결정해야 했던 trade-off |
| 비용 | 추가 시간, context와 review 부담 |
| 다음 변경 | 유지·삭제·script 이동·추가 검증 |

## 첫 버전의 성공 기준

- Blueprint 문서를 중복 생성하거나 조용히 덮어쓰지 않는다.
- 같은 목적의 Skill을 중복 실행하지 않는다.
- Agent가 verify를 실행하지 않아도 Stop hook이 검사한다.
- 승인 없는 push·merge·배포가 실행되지 않는다.
- Test 성공뿐 아니라 test 삭제와 범위 축소를 확인한다.
- 실행한 command, exit code, browser 증거와 남은 위험이 한 Run 문서에 남는다.
- 사용자는 왜 각 Skill이 호출됐고 결과가 무엇인지 자신의 말로 설명할 수 있다.

성공 기준을 모두 만족해도 즉시 전사 표준으로 확대하지 않는다. 서로 다른 유형의 실제 작업에서 반복 실행한 뒤 공통으로 유용한 규칙만 남긴다.

## 실패하기 쉬운 설계

### 모든 Skill을 한 번씩 호출한다

Skill 수가 많다고 검증이 강해지지 않는다. Context와 산출물이 중복되고 서로 다른 규칙이 충돌할 가능성이 커진다. 작업 위험도와 빈칸을 기준으로 route한다.

### Hook에 사람의 판단까지 넣는다

좋은 설계, 올바른 요구사항과 충분한 test는 단순 exit code로 판정할 수 없다. Hook은 관찰 가능한 최소 안전선을 맡고 의미 판단은 human checkpoint와 review에 남긴다.

### Upstream을 복사해 직접 수정한다

Fork 전체를 유지하면 upstream 변경과 개인 수정의 차이를 추적하기 어렵다. Source와 version을 고정하고 adapter와 override를 별도 파일로 유지한다. 설치 전에는 license와 자동 update 방식도 확인한다.

### 첫날부터 병렬 Agent와 자동 배포를 붙인다

잘못된 workflow를 빠르게 반복할 뿐이다. 단일 agent·local 검증에서 안정된 뒤 읽기 전용 reviewer, 독립 Worktree와 배포 순서로 확장한다.

## 최종 판단

이 조합은 충분히 시험할 가치가 있다. 다만 “Superpowers + gstack + Matt Pocock의 합집합”이 아니라 **Blueprint 이후 개발 과정의 빈칸을 채우는 선택적 조합**이어야 한다. 개인 harness의 핵심 자산은 Skill 목록이 아니라 정본의 위치, route 기준, 권한 경계, 독립된 완료 판정과 실제 Run에서 축적한 실패 기록이다.

## See Also

- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [Superpowers Brownfield 실전 가이드](superpowers-brownfield-field-guide.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md)
