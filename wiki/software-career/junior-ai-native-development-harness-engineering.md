# 취업 초기 주니어를 위한 AI-Native 개발 프로세스 2: Harness Engineering

> Sources: [취업 초기 주니어를 위한 AI-Native 개발 프로세스](junior-ai-native-development-process.md); [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md); [Superpowers Brownfield 실전 가이드](../ai-agents/superpowers-brownfield-field-guide.md); [AI Agent 산문 게이트와 결정적 게이트](../ai-agents/ai-agent-prose-vs-deterministic-gates.md); [AI Harness 실측과 선택 가이드](../ai-agents/ai-harness-audit-and-selection.md)
> Archived: 2026-08-16

## Overview

1편이 주니어가 AI와 개발할 때 지켜야 할 원칙을 정의했다면, 이 문서는 그 원칙을 Superpowers 같은 Skill 기반 harness에서 반복 가능한 작업 절차로 실행하는 방법을 정리한다. 핵심은 모든 Skill을 일률적으로 호출하는 것이 아니라 작업 위험도에 맞는 Skill을 선택하고, 각 단계 사이에 인간 checkpoint와 실제 검증 command를 두는 것이다. Superpowers는 workflow engine으로 사용하며 요구사항, test의 의미, merge와 production 책임은 사람이 유지한다.

## Harness Engineering의 의미

Prompt engineering은 AI에게 무엇을 할지 설명한다. Harness engineering은 AI가 어떤 순서로 일하고, 어디에서 멈추며, 무엇으로 검증하고, 누가 승인하는지를 설계한다.

1편의 방법론을 harness로 옮기려면 다음 계층이 필요하다.

| 계층 | 역할 |
|---|---|
| Context | 요구사항, 용어와 codebase 정보 제공 |
| Workflow | 조사·계획·TDD·구현 순서 |
| Verification | Test, build, lint와 static analysis |
| Review | Advisory subagent와 인간 review |
| Evidence | 실제 command, diff, 결정과 실패 기록 |

Superpowers는 주로 workflow를 제공한다. Skill에 TDD나 review가 적혀 있어도 실제 test 실행을 건너뛸 수 있다면 그것은 산문 지침이다. Repository command, CI 또는 hook처럼 모델이 무시해도 판정되는 검증 계층을 별도로 둬야 한다.

## 1편과 Superpowers Skill 연결

| 1편 프로세스 | Superpowers 또는 Harness 단계 |
|---|---|
| 위험도 판단 | 사람이 workflow와 Skill 선택 |
| 현재 상태 착륙 | `brainstorming` 전 baseline 실행 |
| Behavior로 조준 | `brainstorming` |
| 용어와 설계 결정 | `brainstorming`과 인간 승인 |
| 구현 계획 | `writing-plans` |
| RED–GREEN–REFACTOR | `test-driven-development` |
| 순차 구현 | `executing-plans` |
| 병렬 하위 작업 | `subagent-driven-development` 조건부 |
| Code review | `requesting-code-review` |
| Review 지적 검증 | `receiving-code-review` |
| Branch 완료 | `finishing-a-development-branch` |
| Worktree 격리 | `using-git-worktrees` 조건부 |
| 실패 원인 추적 | `systematic-debugging` 유형의 절차 |
| 지식 적립 | 별도 Run 기록 |

기본 구조는 다음과 같다.

```text
사람의 Task 정의
→ 현재 baseline 확인
→ brainstorming
→ 사람의 설계 승인
→ writing-plans
→ 사람의 계획 승인
→ executing-plans
   └── 각 behavior마다 TDD
→ requesting-code-review
→ Review 지적 재현
→ 전체 verify
→ 사람이 IDE에서 설명
→ finishing-a-development-branch
→ Merge 후 verify
```

## 작업 규모별 Skill 선택

모든 작업에 모든 Skill을 사용하지 않는다.

### 아주 작은 작업

기존 error message 오타처럼 의미와 영향이 명확한 작업은 범위 확인, 직접 수정 또는 single agent, 관련 test와 diff 확인으로 충분할 수 있다. 긴 `brainstorming`, plan과 Worktree는 구현보다 큰 비용이 될 수 있다.

### 일반적인 기능 작업

회원가입 email 중복 검사 같은 작업은 다음 흐름을 기본으로 한다.

```text
brainstorming
→ writing-plans
→ executing-plans
   └── test-driven-development
→ requesting-code-review
→ verify
```

### 독립적인 복수 작업

핵심 기능과 독립적인 API 문서 보강처럼 file과 domain 결정이 겹치지 않는 작업만 별도 Worktree를 검토한다.

```text
핵심 기능
→ 현재 branch + Main agent

독립 문서 작업
→ 별도 Worktree + Agent
```

### 서로 의존하는 기능

회원가입과 로그인처럼 같은 `Member`, `Account`, password와 인증 정책에 의존하는 기능은 순차 진행한다.

```text
회원가입 완료·merge·verify
→ 로그인 branch 시작
```

`subagent-driven-development`나 Worktree가 파일 상태는 분리해도 설계 충돌까지 해결하지는 않는다.

## 회원가입 기능에 적용하는 실제 Workflow

### 0. 사람이 Task Contract 작성

Superpowers를 호출하기 전에 목표와 경계를 준비한다.

```text
Task:
중복되지 않은 email로 회원가입한다.

포함:
- email과 password validation
- password encoding
- Member 저장
- 중복 email 차단

제외:
- 로그인
- JWT
- 계정 잠금
- email 인증

Domain language:
- Member: Service에 가입한 사람
- Account: 인증 정보를 소유하는 객체
- email: Member의 식별자

완료 조건:
- 정상 회원가입 성공
- 중복 email 실패
- 잘못된 email 실패
- password 평문 저장 금지
- 관련 test와 전체 build 통과
```

Harness가 요구사항을 대신 결정하게 하지 않는다.

### 1. Baseline 확인

`brainstorming`보다 먼저 application, build, test, lint 또는 static analysis, Git status와 현재 branch를 확인한다. 기존 실패가 있다면 대상과 관계를 기록해 새 회귀와 구분한다.

```text
현재 baseline:
- MemberServiceTest: PASS
- 전체 integration test: 기존 실패 1건
- 기존 실패는 PaymentIntegrationTest이며 이번 범위와 무관
```

### 2. `brainstorming`: 질문과 경계 찾기

```text
brainstorming:

회원가입 기능을 구현하기 전에 요구사항과 기존 code를 분석해줘.

목표:
- 현재 Member 생성 흐름 확인
- Member와 Account의 책임 구분
- 정상·실패 behavior 정리
- 아직 결정되지 않은 정책 발견
- 변경 가능한 최소 범위 제안

제약:
- 파일을 수정하지 않는다.
- 로그인과 JWT는 포함하지 않는다.
- 기존 code에서 확인한 사실과 제안을 구분한다.

출력:
1. 현재 Code path
2. Domain 용어
3. 확정된 요구사항
4. 미결정 사항
5. 회귀 후보
6. 추천하는 최소 vertical slice
```

사람은 code에서 찾은 사실과 agent의 제안을 구분하고, `Member`와 `Account`의 의미, 회사 convention과 scope를 직접 확인한다.

이 단계의 subagent는 읽기 전용으로 사용한다.

```text
Subagent A → 기존 회원 생성 Code path 조사
Subagent B → Security와 개인정보 위험 조사
Subagent C → 기존 test convention과 회귀 후보 조사
Main agent → 결과 종합
사람 → 실제 code에서 확인하고 결정
```

### 3. 사람의 설계 승인

다음 단계 전에 확정된 결정과 보류 사항을 분리한다.

```text
승인된 결정:
- Member는 email을 식별자로 사용한다.
- 중복 검사는 MemberService의 책임이다.
- database unique constraint를 함께 사용한다.
- password encoding은 Account 생성 전에 수행한다.
- 중복 email은 HTTP 409로 반환한다.

보류:
- 계정 잠금
- email 인증
```

설계가 합의되기 전에는 `writing-plans`로 넘어가지 않는다.

### 4. `writing-plans`: 검증 가능한 단계로 분할

```text
writing-plans:

승인된 회원가입 설계를 구현 계획으로 나눠줘.

원칙:
- 한 단계에는 하나의 behavior만 둔다.
- 각 단계에 먼저 작성할 실패 test를 명시한다.
- 예상 변경 파일과 실행할 test command를 명시한다.
- 로그인과 JWT는 포함하지 않는다.
- 추측 기반 공통 abstraction을 만들지 않는다.

각 단계 형식:
1. Behavior
2. RED test
3. 최소 구현
4. REFACTOR 범위
5. 예상 변경 파일
6. Verification command
7. 사람이 확인할 설계 질문
```

좋은 계획은 정상 회원가입, 중복 email, 잘못된 email, password encoding과 Controller error mapping처럼 독립적으로 확인 가능한 behavior로 나뉜다.

### 5. `executing-plans`: 순차 실행

취업 초기에는 `subagent-driven-development`보다 `executing-plans`를 기본으로 둔다.

```text
Task 1 실행 → 사람 확인
Task 2 실행 → 사람 확인
Task 3 실행 → 사람 확인
```

각 Task 사이에 구현 내용, RED evidence, GREEN command, 예상 밖의 변경 파일과 다음 Task의 의존성을 확인한다.

### 6. `test-driven-development`: 계획 내부에서 반복

TDD를 전체 구현 후의 검증 단계로 사용하지 않는다.

```text
Task: 중복 email 차단

RED
→ 같은 email을 두 번 등록하는 test 작성
→ 실패 이유 확인

GREEN
→ 중복 확인을 위한 최소 구현
→ Target test 통과

REFACTOR
→ 이름과 책임 정리
→ 관련 regression test 통과
```

사람은 구현 부재로 실패했는지, test setup 오류인지, 이미 기존 behavior가 있어 GREEN인지와 기대와 다른 exception으로 실패한 것인지를 실제 output에서 확인한다.

### 7. Advisory subagent의 Test 비판

```text
현재 회원가입 test를 읽기 전용으로 검토해줘.

확인:
- Acceptance criteria와 test가 대응하는가?
- 잘못된 구현도 통과할 수 있는가?
- Mock 때문에 실제 저장 behavior를 놓치는가?
- 중복 email의 concurrency 위험을 숨기지 않는가?
- 기존 test가 삭제 또는 약화됐는가?

파일을 수정하지 말고 근거와 file 위치를 보고해줘.
```

Main agent나 사람이 지적을 재현한 뒤 필요한 test만 보강한다.

### 8. `requesting-code-review`: 완료 선언과 Review 분리

```text
requesting-code-review:

승인된 회원가입 Task contract와 현재 diff를 비교해줘.

검토:
- Scope를 벗어난 변경
- Domain 용어 불일치
- Public interface 변경
- 불필요한 abstraction
- Transaction과 concurrency 위험
- Password 또는 개인정보 노출
- Test 삭제·비활성화
- Acceptance criteria 누락

각 지적에 file 위치, 재현 방법, 위험과 수정 방향을 포함한다.
직접 수정하지 않는다.
```

### 9. `receiving-code-review`: 지적 재현

Review agent의 지적도 바로 믿지 않는다.

```text
Review 지적
→ 실제 code 확인
→ Test 또는 재현 절차 실행
→ 유효성 판단
→ 수정
→ Regression test
```

Race condition 지적을 받았다고 즉시 복잡한 locking을 추가하지 않는다. Database constraint 존재 여부, 실제 재현 가능성, 현재 Issue 범위와 별도 Issue 분리 여부를 확인한다.

### 10. Deterministic verification

Superpowers는 작업 순서를 안내하고 repository command는 결과를 기계적으로 확인한다. 가능하면 검증 명령을 하나로 모은다.

```text
verify
├── format check
├── lint
├── type check 또는 compile
├── unit test
├── integration test
└── build
```

처음에는 사람이 직접 실행하고, 안정되면 CI나 hook으로 옮긴다. `verify`가 성공해도 test의 의미, test 삭제, scope, domain 일관성과 production 위험은 사람이 확인한다.

### 11. IDE에서 사람이 직접 설명

Skill이 모두 완료돼도 요청 흐름을 설명하지 못하면 개인 Definition of Done을 통과하지 못한 것이다.

```text
POST /members
→ MemberController.register()
→ MemberService.register()
→ 중복 검사
→ PasswordEncoder
→ MemberRepository.save()
→ HTTP 201
```

Transaction 위치, application 검사와 database constraint의 역할, encoding된 password 저장 위치, behavior별 test와 장애 조사 시작점을 직접 설명한다.

### 12. `finishing-a-development-branch`

Branch 완료 전에 Task contract, 설계 결정, 변경 파일, target test와 full verify 결과, review 지적과 해결 상태, known limitation과 recovery 방법을 모은다.

```text
현재 branch verify
→ Merge
→ 기준 branch에서 다시 verify
→ CI 확인
→ 배포 결과 확인
```

Worktree를 사용했다면 Worktree 자체가 아니라 그 안에서 checkout한 branch를 merge한 뒤 정리한다.

## 주니어용 Superpowers 기본 Preset

```text
기본:
brainstorming
→ writing-plans
→ executing-plans
   └── test-driven-development
→ requesting-code-review
→ 사람이 IDE 검토
→ finishing-a-development-branch
```

다음은 조건부로 사용한다.

```text
using-git-worktrees
→ 독립적인 작업 또는 학습용 실험

subagent-driven-development
→ 하위 작업이 독립적이고 사람이 각 diff를 감당할 수 있을 때

systematic-debugging
→ 예상하지 못한 실패나 기존 bug를 재현할 때

receiving-code-review
→ Review 지적을 검증하고 반영할 때
```

## Subagent 운영 규칙

초기 허용 범위는 Code path와 convention 조사, 요구사항 모호함 탐지, test 비판, security review, diff review와 regression 후보 탐색이다.

초기에는 다음을 맡기지 않는다.

- 독립적인 설계 결정
- Public interface 변경
- 인증 architecture 결정
- Database schema 변경
- 여러 module을 가로지르는 refactoring
- 별도 branch에서 핵심 기능 병렬 구현
- Review 없는 commit 또는 merge

숙련 후에는 test fixture, API 문서, 반복 DTO mapping, 확정된 validation pattern, parameterized test와 독립적인 lint·format 수정부터 제한적으로 위임한다.

## Worktree 도입 단계

### 단계 1: Worktree 없음

현재 feature branch, Main agent, advisory subagent와 순차 TDD로 핵심 기능을 개발한다.

### 단계 2: Worktree Skill 연습

폐기 가능한 작은 작업에서 Worktree path·branch·base 확인, 작은 변경, diff 비교와 제거까지 경험한다.

### 단계 3: 독립적인 보조 작업

핵심 회원가입 구현과 독립적인 API 문서 또는 test fixture 보강을 별도 Worktree에 둔다. Task, directory, branch와 base를 항상 함께 기록한다.

### 단계 4: 제한적 병렬 구현

Domain 결정과 수정 file이 독립적이고, public interface가 안정됐으며, 각각 test할 수 있고, 사람이 두 diff를 감당할 수 있고, merge 순서가 명확할 때만 사용한다.

## Harness 성장 순서

### Level 1: 수동 Workflow

Skill을 명시적으로 호출하고 사람이 checkpoint와 test command를 직접 실행하며 Run을 기록한다. 먼저 각 단계의 의미를 배운다.

### Level 2: 단일 Verify command

Repository에 맞는 대표 검증 command를 정한다.

### Level 3: CI Gate

Merge 전에 동일한 검증이 자동으로 실행되게 한다.

### Level 4: Local Hook

위험 명령, test 미실행과 완료 조건처럼 기계적으로 판정 가능한 항목만 차단한다. Hook에는 loop guard와 탈출 조건을 둔다.

### Level 5: 제한적 Multi-agent

Workflow, 검증 command와 인간의 review capacity가 안정된 후 write agent 수를 늘린다.

## Run 기록 형식

```markdown
# Run: 회원가입 email 중복 차단

- Repository / base commit:
- Branch / Worktree:
- Explicit skills:
- Expected files:
- Commands and gates:

## Decisions

- 확정한 Domain 용어
- 사람이 내린 설계 결정
- 보류한 결정

## Observed execution

- 실제로 호출된 Skill
- RED·GREEN·REFACTOR evidence
- Agent가 범위를 벗어난 지점
- 사람의 중단·승인·재계획

## Verification

- Target test
- Module test
- Full verify
- CI
- Review 결과

## Learning

- 유지할 harness 규칙
- 제거할 불필요한 단계
- AI가 잘못 가정한 내용
- 다음 작업에서 검증할 질문
```

다음 항목을 서로 구분한다.

```text
Skill을 호출했다
≠ Skill의 목적을 달성했다

TDD를 요청했다
≠ RED를 실제로 관찰했다

Code review를 요청했다
≠ 지적을 재현했다

CI 설정을 작성했다
≠ Remote CI가 통과했다
```

## 최종 구조

```text
사람
├── Task와 Domain language 정의
├── 설계 checkpoint 승인
├── Test 의미 판단
├── IDE에서 Code path 설명
└── Merge·배포 책임

Main agent
├── brainstorming
├── writing-plans
├── 순차 TDD 구현
└── 수정과 evidence 정리

Advisory subagent
├── Code path 조사
├── Test 비판
├── Security review
└── Diff review

Deterministic tool
├── Build
├── Test
├── Lint
├── Static analysis
└── CI

Worktree
└── 독립적인 보조 작업과 안전한 실험부터
```

Superpowers에 모든 책임을 넘기는 것이 아니라, 사람이 설계한 개발 process를 반복 가능하게 실행하는 workflow engine으로 사용한다. 현재 Wiki의 Superpowers 실전 기록은 특정 local repository의 품질 baseline 작업을 한 차례 관찰한 범위이며 remote CI와 모든 기능 Issue에 일반화할 수 없다. 이 문서는 1편의 방법론과 현재 관찰을 결합한 주니어용 적용안이다.

## See Also

- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
- [Claude Code Hooks](../ai-agents/claude-code-hooks.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
