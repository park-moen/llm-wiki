# 취업 초기 주니어를 위한 AI-Native 개발 프로세스

> Sources: [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md); [AI 시대 신입 개발자의 생존과 성장 판단](ai-era-junior-developer-survival-assessment-2026-08-13.md); [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md); [AI Coding Autonomy Experiment와 Human-in-the-loop](../ai-agents/ai-coding-autonomy-experiment.md); [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
> Archived: 2026-08-16

## Overview

취업 초기에는 AI가 많은 code를 만들게 하는 것보다 작은 변경을 요구사항부터 code, test와 운영 영향까지 이해하고 책임지는 능력을 먼저 키운다. 이 문서는 이를 **이해 가능한 한 조각 개발법**으로 정의하고, single main agent와 advisory subagent를 기본값으로 삼아 TDD, deterministic verification과 인간 review를 연결한다. Worktree와 write subagent는 독립적이고 낮은 위험의 작업부터 단계적으로 도입한다.

## 핵심 원칙

> 한 번에 하나의 작은 동작을 선택하고, 요구사항부터 code·test·운영 영향까지 자신이 설명할 수 있는 상태로 완료한다.

AI는 탐색과 구현을 가속하지만 사람은 다음을 소유한다.

- 문제와 요구사항의 의미
- Domain 용어
- 변경 범위
- Public interface
- Test가 의미 있는지에 대한 판단
- Merge와 배포 결정
- 장애 발생 시 복구 책임

## 역할 분담

| 역할 | 책임 |
|---|---|
| 사람 | 요구사항, 용어, 설계 결정, test 의미와 최종 review |
| Main agent | Code 탐색, 구현 후보, 반복 수정과 test 작성 보조 |
| Advisory subagent | 읽기 전용 조사, 위험 분석과 test·diff review |
| Deterministic tool | Build, test, type check, lint와 static analysis |
| 선임·동료 | 회사 고유의 architecture, domain 지식과 trade-off review |

기본 흐름은 다음과 같다.

```text
AI가 제안한다
→ 사람이 근거를 확인한다
→ Test와 도구가 검증한다
→ 사람이 설명하고 승인한다
```

## 작업별 표준 프로세스

### 0. 위험도 판단

AI에게 얼마나 맡길지는 작업 난이도보다 실패 비용과 변경 영향을 기준으로 정한다.

| 작업 | 기본 운영 |
|---|---|
| 문서, fixture와 단순 mapping | AI 위임 비중을 높일 수 있음 |
| 작은 CRUD behavior | Main agent와 TDD |
| Domain rule과 database schema | 사람이 설계 주도 |
| 로그인, 인증, 결제와 개인정보 | 사람이 세밀하게 검토 |
| 여러 module에 걸친 refactoring | 작은 작업으로 먼저 분할 |
| 장애 수정 | 원인과 재현을 사람이 먼저 이해 |

오래 운영할 code, 많은 사용자가 의존하는 interface와 실패 비용이 큰 기능일수록 더 작은 변경과 강한 review를 적용한다.

### 1. 착륙: 현재 상태 실행

새 repository나 처음 만지는 module에서는 바로 구현하지 않는다. Application과 test 실행 방법, 현재 test 상태, 주요 directory의 역할, 기능 entry point, 외부 API와 database 의존성을 확인한다.

현재 상태를 실행하지 못하면 변경 후 문제가 기존 실패인지 새 회귀인지 구분하기 어렵다. 기존 test가 실패한다면 그 사실부터 baseline으로 기록한다.

### 2. 조준: 사용자 동작 하나로 축소

회원 기능처럼 큰 요청을 회원가입, email 중복 검사, 비밀번호 저장과 로그인 같은 관찰 가능한 behavior로 나눈다.

```text
입력: 유효한 email과 password
행동: 회원가입 요청
결과: 회원이 저장되고 HTTP 201을 반환한다
```

한 작업에서는 하나의 정상 동작과 직접 연결된 주요 실패 동작만 다룬다.

### 3. 용어 고정

`User`, `Member`, `Account`, `Customer`가 같은 개념인지 다른 개념인지 확인하고, 사람·AI·대화·code에서 같은 용어를 같은 의미로 사용한다.

```text
Member
= Service에 가입한 사람

Account
= Member가 로그인할 때 사용하는 인증 정보
```

용어의 목적은 큰 문서를 만드는 것이 아니라 작업 도중 의미가 바뀌는 것을 막는 것이다.

### 4. Code path와 Blast radius 확인

전체 repository를 이해하려 하지 않고 요청과 연결된 실행 경로를 따라간다.

```text
POST /members
→ MemberController
→ MemberService.register()
→ MemberRepository
→ members table
```

같은 code를 공유하는 로그인, 회원 조회와 비밀번호 변경 같은 회귀 후보도 찾는다. 이 단계에는 읽기 전용 subagent가 적합하다.

```text
읽기 전용으로 회원가입 요청이 database 저장까지 이동하는 경로를 조사한다.

출력:
1. Entry point
2. 호출되는 class와 method
3. 공유 dependency
4. 회귀 가능성이 있는 기존 기능
5. 아직 확인할 수 없는 가정

파일은 수정하지 않는다.
```

### 5. 구현 전 설계 결정 기록

긴 설계 문서 대신 이번 작업에서 필요한 결정과 미결정 사항을 짧게 기록한다.

```text
결정:
- Member의 식별자는 email이다.
- email 중복은 허용하지 않는다.
- 비밀번호는 평문으로 저장하지 않는다.
- 중복 email은 HTTP 409로 반환한다.
- 로그인 기능은 이번 작업에 포함하지 않는다.

미결정:
- 계정 잠금 정책
- email 인증
```

미결정 사항을 AI가 임의로 채우지 못하게 한다. Public interface는 input, output, 실패 표현과 책임 layer를 자신의 말로 설명할 수 있어야 한다.

### 6. TDD로 작은 Feedback loop 구성

기존 동작을 바꾼다면 characterization test로 현재 동작을 먼저 고정한다. 새 동작은 실패 test부터 작성한다.

```text
RED
중복 email 회원가입 test가 실패한다

GREEN
중복을 차단하는 최소 code를 구현한다

REFACTOR
중복 검사 책임과 이름을 정리한다
```

AI가 test를 작성해도 요구사항을 검증하는지, 잘못된 구현도 통과할 수 있는지, mock만 확인하는지와 기존 test가 삭제·비활성화되지 않았는지를 사람이 검토한다.

### 7. Main agent에게 Behavior 하나만 위임

Main agent에게 기능 전체를 한 번에 맡기지 않는다.

```text
현재 목표:
중복 email 회원가입을 차단한다.

확정된 정책:
- email은 Member의 식별자다.
- 중복이면 DuplicateMemberEmailException을 발생시킨다.
- Controller에서 HTTP 409로 변환한다.

수정 가능:
- MemberService
- MemberRepository
- 관련 test

수정 금지:
- 로그인
- Security 설정
- database schema
- 기존 public interface

완료 조건:
- 실패 test의 RED를 확인한다.
- 최소 구현으로 GREEN을 만든다.
- 전체 Member test를 실행한다.
- 변경 파일과 test 결과를 보고한다.
```

Agent가 범위를 벗어나면 prompt를 계속 누적하기보다 작업을 중단하고 scope와 assumption을 다시 확인한다.

### 8. Advisory subagent의 독립 검토

취업 초기에는 subagent를 write worker보다 자문가로 사용한다.

구현 전에는 code path, 기존 convention, 요구사항의 모호함, 회귀 후보와 security 위험을 조사한다. Test 작성 후에는 요구사항과 test의 대응, 잘못된 구현의 통과 가능성, mock의 한계와 빠진 edge case를 검토한다. 구현 후에는 요구 범위를 벗어난 변경, 불필요한 abstraction, naming 불일치, error handling과 security 누락, test 삭제·약화를 찾는다.

Subagent의 지적도 정답으로 취급하지 않는다. 실제 code와 test에서 재현한 뒤 반영한다.

### 9. 기계적 검증

AI의 완료 메시지는 완료 증거가 아니다. 최소한 target test, 관련 module test, build, type check·lint·static analysis, 기존 test 삭제 여부와 예상 scope 대비 실제 `git diff`를 확인한다.

```text
Target test: PASS
Module test: PASS
Build: PASS
Changed files: 4
Deleted tests: 0
Known limitation: email 인증은 범위 밖
```

### 10. IDE에서 직접 Code path 추적

취업 초기에는 IDE에서 `Controller → Request DTO → Service → Domain rule → Repository → Database → Response 또는 Exception` 경로를 직접 따라간다.

요청 진입점, validation과 transaction 위치, domain rule의 책임 layer, 실패 response와 이를 보호하는 test를 자신의 말로 설명한다. AI가 작성한 code라도 여기까지 설명할 수 있어야 개인의 경험으로 전환된다.

### 11. 구체적인 사람 Review 요청

선임에게 전체를 막연히 봐 달라고 요청하지 않고, 자신이 내린 결정과 고민을 함께 전달한다.

```text
결정:
- email을 Member 식별자로 사용했다.
- 중복 검사는 Service에서 수행했다.
- 중복 응답은 HTTP 409로 정했다.

검토받고 싶은 부분:
- database unique constraint도 필요한가?
- 조회 후 저장 사이의 race condition은 어떻게 다루는가?
- Exception 이름이 기존 convention과 맞는가?
```

이렇게 질문해야 단순 승인이 아니라 설계 판단을 배울 수 있다.

### 12. 배포 이후 확인

Merge 후 가능한 범위에서 CI, 배포 결과, error log, monitoring 지표, 실제 API 응답과 rollback 가능성을 확인한다. Production 동작을 확인해야 code 작성 경험이 software engineering 경험으로 확장된다.

### 13. 지식 적립

작업마다 새로 이해한 내용, 설계 결정, AI가 틀린 부분, 직접 debug한 내용, 검증 evidence와 남은 위험을 짧게 기록한다.

```text
Task: 회원가입 email 중복 차단
새로 이해한 것: MemberService가 회원 생성 정책을 소유한다.
설계 결정: 중복 email은 HTTP 409로 처리한다.
AI가 틀린 부분: Controller에 중복 검사를 넣었다.
직접 Debug한 부분: Transaction 경계에서 중복 오류가 늦게 발생했다.
남은 위험: 동시 요청의 race condition은 unique constraint 검토가 필요하다.
```

생성한 code 양보다 알게 된 사실, 내린 결정, 발견한 문제와 검증 증거를 기록한다.

## Subagent 숙련 단계

### 단계 1: Advisory only

초기 기본값은 Main agent만 수정하고 subagent는 읽기 전용으로 사용하며, 현재 branch를 IDE에서 직접 추적하는 것이다.

다음 조건을 충족할 때까지 유지한다.

- AI가 만든 변경을 파일별로 설명할 수 있다.
- Test가 어떤 behavior를 보호하는지 설명할 수 있다.
- AI의 잘못된 가정을 발견한 경험이 있다.
- 간단한 bug를 직접 재현하고 고칠 수 있다.
- 선임의 review 지적을 이해하고 재현할 수 있다.

### 단계 2: Sequential bounded write

Test fixture, DTO mapping, 반복 validation, API 문서, parameterized test 변환처럼 입력과 pattern이 확정된 작업만 subagent에게 하나씩 맡긴다. 동시에 여러 write subagent를 실행하지 않고 Main agent가 매번 diff와 test를 검토한다.

### 단계 3: One isolated Worktree

독립성이 높은 작업 하나를 Worktree로 분리한다. Production 핵심 logic보다 문서, test coverage 보강, 독립적인 lint 수정, 작은 개발 도구 개선과 behavior를 바꾸지 않는 refactoring 실험이 적합하다.

목표는 속도가 아니라 다음 수명주기를 직접 익히는 것이다.

```text
생성
→ Branch·base 확인
→ Agent 실행
→ Diff review
→ Test
→ Merge 또는 폐기
→ Worktree 정리
```

### 단계 4: 제한적 병렬 Worktree

두 작업의 domain 결정과 file이 독립적이고, public interface가 안정됐으며, 각각 test할 수 있고, 각 diff를 사람이 감당할 수 있을 때만 병렬화한다. 처음에는 핵심 작업 하나와 보조 작업 하나 정도로 제한한다.

회원가입과 로그인처럼 같은 인증 model에 의존하는 기능은 순차적으로 진행한다.

## Worktree 학습 순서

### 읽기 전용 Worktree

다른 branch를 Worktree로 열어 path, checkout branch, Main Worktree와의 파일 분리, IDE에서 별도 project로 여는 방법과 안전한 제거를 익힌다.

### 폐기 가능한 실험

같은 작은 문제를 현재 branch와 별도 Worktree에서 다르게 구현해 diff를 비교하고 하나만 남긴다. 이 단계에서 Worktree는 생산 수단보다 안전한 실험 공간이다.

### 독립적인 실제 작업

Test 보강이나 문서처럼 main 기능과 충돌하지 않는 작업을 맡기고 task, Worktree, branch, base와 status를 함께 기록한다.

```text
Task: MEMBER-142 회원가입 예외 문서화
Worktree: project-member-142-docs
Branch: agent/member-142-docs
Base: develop
Status: review-ready
```

Branch와 directory 이름이 모호하면 Worktree 수를 늘리지 않는다.

## 개인 Definition of Done

- [ ] 요구사항을 정상·실패 behavior로 설명할 수 있다.
- [ ] 핵심 domain 용어의 의미가 code와 일치한다.
- [ ] Entry point부터 database 또는 외부 API까지 호출 흐름을 설명할 수 있다.
- [ ] 변경 범위와 회귀 후보를 알고 있다.
- [ ] Test가 어떤 behavior를 검증하는지 설명할 수 있다.
- [ ] AI가 만든 모든 변경 파일을 읽었다.
- [ ] 기존 test가 삭제·약화되지 않았다.
- [ ] Build, test와 정적 검증 결과를 직접 확인했다.
- [ ] 중요한 설계 결정과 대안을 설명할 수 있다.
- [ ] 장애가 나면 어디부터 조사할지 알고 있다.
- [ ] 이번 작업에서 새로 배운 사실을 기록했다.

모든 항목을 완벽하게 수행하지 못하더라도 확인하지 못한 항목을 모른다는 사실은 알고 있어야 한다.

## 일주일 단위 성장 루프

매주 다음 경험을 하나씩 확보한다.

- AI가 만든 code를 직접 설명한 작업
- AI의 잘못된 가정을 발견한 사례
- 직접 root cause를 추적한 bug
- 선임에게 한 설계 질문
- Test가 실제 defect를 잡은 사례
- 작은 refactoring 또는 직접 구현
- Worktree 생성부터 정리까지의 연습
- 업무 중 알게 된 domain 지식 기록

## 최종 운영 원칙

```text
핵심 기능
→ 현재 branch
→ Single Main agent
→ TDD
→ IDE에서 직접 흐름 추적
→ 사람이 최종 판단

Subagent
→ Advisory only
→ 읽기 전용 조사·test·diff review

Worktree
→ 학습과 독립적인 보조 작업부터
→ 초기에는 병렬 write 최소화

완료
→ AI의 보고가 아니라
→ 설명 가능성 + Test evidence + Human review
```

성장 기준은 AI가 없을 때보다 더 많은 code를 만드는지가 아니다. AI와 함께 작업하면서 이전보다 더 복잡한 변경을 이해하고 안전하게 책임질 수 있게 되는지가 핵심이다.

## See Also

- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [AI Agent 산문 게이트와 결정적 게이트](../ai-agents/ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](../ai-agents/ai-harness-audit-and-selection.md)
