# Superpowers 기반 Brownfield 연습 워크플로 초기 설계

> Sources: [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md); [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
> Archived: 2026-08-11

## Overview

기존 repository를 clone한 뒤 Claude에서 Superpowers Skill을 명시적으로 호출하며 brownfield 작업을 연습하기 위한 초기 시뮬레이션 설계다. 각 Issue에 모든 Skill을 일률적으로 강제하기보다, Issue의 불확실성·병렬성·변경 위험에 따라 필요한 Skill을 선택하고 TDD를 계획 실행 내부의 반복 방식으로 배치한다. 이 문서는 실사용 전 가설을 보존한 시점 고정 Archive이며, 이후 실제 실행 기록은 별도 원본으로 수집해 살아 있는 실전 가이드로 발전시킨다.

## 연습 목적과 사용 규칙

연습 환경은 다음과 같이 가정한다.

- 기존 repository를 local에 clone해 brownfield 상황에서 작업한다.
- 회사 계정의 Claude를 사용한다.
- Superpowers 사용 여부를 명확히 남기기 위해 요청 앞에 Skill 이름을 붙인다.
- Skill이 자동으로 선택됐다고 추정하지 않고, 실제 호출과 산출물을 기록한다.
- 성공 사례뿐 아니라 중단, 재계획, 과도한 추상화와 test 실패도 보존한다.

호출 기록은 다음처럼 명시한다.

```text
brainstorming: 안동 Festival 리뉴얼의 정보 구조와 Public/Admin 공통 모델을 설계해줘.
```

```text
writing-plans: 합의한 설계를 독립적으로 검증할 수 있는 구현 단계로 나눠줘.
```

```text
test-driven-development: Admin 수정 내용이 Public 페이지에 반영되지 않는 문제를 재현하고 수정해줘.
```

이 규칙은 Skill 이름을 기록하기 위한 개인 운영 규칙이며, 실제 Skill이 어떤 조건에서 무엇을 실행하는지는 향후 Claude transcript와 repository 결과로 검증해야 한다.

## Issue마다 적용할 기본 흐름

모든 Issue에 동일한 Skill 목록을 기계적으로 적용하지 않는다. 기본 골격은 다음과 같다.

```text
brainstorming
→ writing-plans
→ using-git-worktrees                 조건부
→ executing-plans 또는 subagent-driven-development
   └── 구현 단위마다 test-driven-development
→ requesting-code-review
→ finishing-a-development-branch
```

중요한 점은 `test-driven-development`를 구현 완료 후의 검증 단계로 두지 않는 것이다. TDD는 계획을 실행하는 과정에서 작은 단위로 반복한다.

```text
RED      실패하는 test로 요구 동작 표현
GREEN    test를 통과하는 최소 구현
REFACTOR 중복과 구조 개선
```

`subagent-driven-development`와 `executing-plans`는 작업 방식에 따라 선택한다. 서로 독립적인 하위 작업을 나눌 수 있으면 전자를 검토하고, 같은 code와 설계에 강하게 의존하는 작업은 후자로 순차 실행한다.

## 초기 Issue 의존 구조

최초 시뮬레이션은 다음 작업을 대상으로 한다.

- Test code 작성
- 안동 Festival 통합 후 축전 개요·개막식·세부 프로그램·체험 참여 detail page 리뉴얼
- Public 페이지 수정과 동기화되지 않는 Admin 개선
- 페이지마다 중복 작성된 UI를 component로 추출하고 design system 설계
- 공통 component를 추출할 때 Storybook Story 추가

이를 단순한 Issue 번호 순서가 아니라 dependency 기준으로 다시 배열한다.

```text
Issue 0: 최소 품질 파이프라인
    ↓
Issue 1: 기존 동작 Characterization Test
    ↓
Issue 2-A: Public/Admin 공통 Festival content model·contract
    ↓
Issue 2: Public 안동 Festival detail page 리뉴얼
    ↓
Issue 3: 동일 contract를 사용하는 Admin 개선과 동기화
    ↓
Issue 4: 중복 component 추출·design system·Storybook
```

`Issue 2-A`가 중요한 이유는 Public 페이지가 임의의 데이터 구조를 먼저 만들고 Admin이 나중에 따라가는 흐름을 피하기 위해서다. 축전 개요·개막식·세부 프로그램·체험 참여의 공통 content model 또는 API contract를 먼저 합의하고 Public과 Admin이 같은 구조를 사용하도록 한다.

## Issue 0: 최소 품질 파이프라인

기능 구현 전에 기존 실패와 새 회귀를 구분할 수 있는 최소 baseline을 만든다.

```text
lint
typecheck
unit test runner
build
CI에서 위 명령 실행
```

`ESLint`, typecheck, test runner와 CI는 먼저 준비하는 편이 좋다. 반면 `Husky`와 `lint-staged`는 local에서 빠르게 문제를 발견하게 해주는 편의 계층으로 보고 CI보다 우선하지 않는다. 초기부터 높은 coverage 기준이나 복잡한 test architecture를 완성하려 하지 않는다.

Agent가 반복 실행할 수 있는 대표 명령을 명확히 만드는 것이 첫 목표다.

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

## Issue 1: Characterization Test

기존 code의 test는 새 요구사항을 RED로 표현하기 전에 현재 동작을 고정하는 안전망 역할을 한다. Characterization Test는 바람직한 동작이 아니라 변경 전 code가 실제로 하는 동작을 기록한다.

우선 보호할 후보는 다음과 같다.

- 현재 안동 Festival 데이터가 화면에 표시되는 흐름
- route와 detail page 접근
- Public과 Admin이 데이터를 읽고 저장하는 흐름
- Admin 변경이 Public에 반영되는 핵심 경로
- 이후 공통화할 가능성이 있는 기존 component 상태

전체 codebase의 coverage를 먼저 높이는 것이 아니라 Issue 2~4의 blast radius에 안전망을 둔다. 기존 동작을 기록하는 test는 현재 code에서 먼저 통과할 수 있으므로 모든 Characterization Test가 RED로 시작해야 하는 것은 아니다.

## Issue 2: Public Festival 리뉴얼

먼저 사용자 관점의 정보 구조와 Admin에서 수정해야 할 데이터를 구분한다.

```text
안동 Festival
├── 축전 개요
├── 개막식
├── 세부 프로그램
└── 체험 참여
```

`brainstorming`에서는 각 detail page의 목적, URL, 공통 정보, 고유 정보, responsive layout과 Admin 편집 범위를 다룬다. `writing-plans`에서는 content model, route, page shell과 각 detail page를 독립적으로 검증할 수 있는 단계로 나눈다.

본격적인 design system을 미리 완성하지는 않는다. 처음부터 분명한 page shell이나 section heading은 좁은 책임의 임시 공통 component로 시작할 수 있지만, 실제 사용 사례 없이 범용 props를 예측하지 않는다.

## Issue 3: Admin 동기화 개선

이 작업은 Admin UI 수정만이 아니라 Admin부터 Public rendering까지 이어지는 데이터 경로의 일관성 문제다.

```text
Admin form model
→ API request
→ Backend schema
→ 저장소
→ Public page query
→ UI rendering
```

먼저 실패하는 integration 또는 regression test로 문제를 재현한다.

```text
1. Admin에서 개막식 정보를 수정한다.
2. 저장한다.
3. Public 개막식 페이지를 조회한다.
4. 변경 내용이 표시되어야 한다.
```

RED에서는 동기화 실패를 확인하고, GREEN에서는 양쪽이 같은 schema·API·data source를 사용하도록 최소 수정한다. REFACTOR에서는 중복 mapping과 변환 logic을 정리한다.

## Issue 4: 중복 추출과 Design System

중복을 관찰하기 전에 완성된 design system을 설계하면 추측 기반 abstraction이 되기 쉽다. 반대로 모든 페이지를 의도적으로 복사한 뒤 한꺼번에 정리할 필요도 없다.

```text
첫 번째 등장
→ 해당 page에 구현

두 번째 유사 패턴
→ 실제 차이와 공통점을 비교

의미와 동작까지 동일하고 재사용이 예상됨
→ 공통 component 추출 검토
```

Issue 2와 Issue 3에서 반복되는 UI를 확인한 뒤 다음 순서로 진행한다.

```text
중복 inventory
→ 같은 개념인지 확인
→ component 책임과 API 설계
→ 지원할 상태를 Story로 정의
→ 공통 component 구현
→ unit·interaction·visual 검증
→ Public/Admin을 순차 migration
```

겉모양만 같고 의미나 interaction이 다르면 하나의 component로 합치지 않을 수 있다.

## Storybook Story와 TDD의 관계

Story는 component의 상태와 사용 예시를 표현하지만 그 자체만으로 자동 test가 되지는 않는다. `play` function, Storybook test runner, visual regression, accessibility 검사와 CI 판정이 연결될 때 실행 가능한 검증 역할을 한다.

따라서 Story는 TDD를 보조할 수 있지만 unit·integration·end-to-end test를 모두 대체하지 않는다.

기존 UI를 안전하게 refactoring하려면 추출 전에 page-level Story나 screenshot을 만들어 시각적 Characterization 자료로 활용할 수 있다. 새 공통 component는 중복을 발견하고 책임과 props를 결정한 뒤 Story를 구현보다 먼저 또는 함께 작성한다.

```text
중복 발견
→ component API 결정
→ Default·상태별 Story 정의
→ component 구현
→ interaction/unit test
→ 기존 page에 적용
```

실제 중복을 발견하기 전에 미래의 모든 component와 Story를 예측해 만드는 것은 피한다.

## Worktree 적용 기준

Worktree는 모든 Issue를 동시에 진행하게 만드는 자동 병렬화 도구가 아니다. 여러 작업자가 같은 repository를 동시에 수정할 때 checkout과 branch 상태를 분리하는 격리 계층이다.

```text
독립적인 병렬 code 수정
→ Issue별 Worktree 검토

같은 schema·component에 의존
→ dependency 순서대로 실행

읽기 전용 조사 또는 단일 agent의 순차 작업
→ 기존 checkout으로 충분
```

Issue 2·3·4는 같은 content model과 component에 의존하므로 무리하게 병렬화하지 않는다. Worktree가 파일 상태는 분리해도 논리적으로 같은 설계를 수정하는 branch 사이의 통합 충돌까지 없애지는 않는다.

## 실사용 기록 방법

향후 각 Claude 작업은 다음 항목을 남긴다.

```markdown
# Run: {Issue와 작업명}

- Date:
- Repository / base commit:
- Issue:
- Explicit skill invocation:
- Expected outcome:
- Files expected to change:
- Commands and gates:

## Observed execution

- 실제로 실행된 단계
- 생성된 plan·test·code·review 산출물
- 사람의 개입과 승인
- 중단·재시도·우회

## Result

- Test 결과
- Review 결과
- 변경된 파일
- 의도와 다른 동작

## Learning

- 유지할 규칙
- 폐기할 가정
- 다음 실행에서 검증할 질문
```

특히 “Skill을 호출했다”와 “Skill의 의도대로 실제 단계가 수행됐다”를 분리해 기록한다. Prompt에 이름이 있었다는 사실만으로 Worktree 생성, RED 관찰, review 수행을 완료했다고 판정하지 않는다. Command output, file diff, test result와 review artifact를 함께 본다.

## Wiki 발전 방식

이 Archive는 초기 가설을 보존하므로 이후 결과로 덮어쓰지 않는다. 실제 실험이 시작되면 Claude transcript, plan, test 결과, review 내용과 개인 회고를 새 `raw/` 원본으로 추가한다. 각 원본을 순차 Ingest하면서 별도의 `Superpowers Brownfield 실전 가이드`를 만들고 갱신한다.

```text
현재 Archive
→ 실험 전 가설

실행별 raw 기록
→ 실제 관찰과 증거

살아 있는 Wiki 가이드
→ 반복 실행에서 확인된 사용법·실패 패턴·판단 기준
```

가이드에는 검증 수준을 구분한다.

- **제안**: 아직 실행하지 않은 설계
- **관찰**: 특정 repository와 Issue에서 실제 발생
- **반복 확인**: 여러 실행에서 같은 결과가 나타남
- **환경 의존**: Claude version, repository 구조나 팀 규칙에 따라 달라짐

이 구분을 유지하면 개인 경험을 일반 규칙으로 너무 빨리 확대하는 일을 줄일 수 있다.

## See Also

- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
