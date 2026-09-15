# Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량

> Sources: Boris Cherny interview, YouTube (Unknown); Pasha interview·Beyond Coding (Unknown)
> Raw: [Building Claude Code with Boris Cherny transcript](../../raw/software-career/building-claude-code-boris-cherny.md); [Original YouTube source provenance](../../raw/software-career/building-claude-code-boris-cherny-source-provenance.md); [From Backend Engineer to Head of Mobile transcript](../../raw/software-career/from-backend-engineer-to-head-of-mobile-lessons-uber.md)
> Updated: 2026-08-16

## Overview

Boris Cherny가 설명한 Claude Code 팀의 workflow는 AI가 code를 많이 생성한다는 사실보다 **숙련도에 따라 작업 mode를 바꾸고, 빠른 생성 뒤에 여러 검증 계층을 배치한다**는 점이 중요하다. 새 codebase에서는 설명을 보며 따라가는 학습 mode를 권하고, 구조가 익숙해진 뒤에는 plan을 먼저 맞춘 다음 여러 checkout이나 Worktree에서 agent를 병렬 실행한다. 생성된 변경은 local test, deterministic 검사, AI review와 human review를 거쳐 production으로 이동한다.

이 workflow는 숙련된 개발자가 익숙한 codebase와 강한 내부 검증 환경에서 사용하는 개인 사례다. 주니어가 높은 PR 처리량이나 병렬 agent 수를 그대로 복사하기보다, 학습 mode에서 code path와 failure mode를 이해한 뒤 점진적으로 위임 범위를 넓히는 근거로 사용하는 편이 적절하다.

## 학습 Mode와 생산 Mode

인터뷰에서는 codebase에 익숙한지에 따라 두 가지 workflow를 구분한다.

### 새 Codebase: 설명을 따라가며 Mental Model 만들기

새 team이나 unfamiliar codebase에서는 Claude Code의 learn 또는 explanatory output style을 사용해 agent가 무엇을 하는지 따라가는 방식을 권한다. 이 단계의 목표는 최대 처리량이 아니라 file 구조, 호출 경로, convention과 변경 이유를 이해하는 것이다.

주니어에게는 다음 loop로 해석할 수 있다.

```text
Agent가 탐색
→ 설명과 근거 file 확인
→ IDE에서 같은 Code path 추적
→ Test와 실제 동작 확인
→ 이해한 내용을 자신의 말로 설명
```

### 익숙한 Codebase: Plan 정렬 후 병렬 실행

Codebase와 domain을 충분히 이해하면 역할이 달라진다. Boris는 plan mode에서 agent와 계획을 먼저 조정하고, plan이 좋아진 뒤 구현을 맡기는 방식이 중요하다고 설명한다. 여러 terminal checkout을 번갈아 사용하고, 별도 directory 관리 부담을 줄일 때는 desktop app의 built-in Worktree 지원을 사용한다.

이때 병렬화의 전제는 다음과 같다.

- 사람이 codebase와 domain convention을 이미 이해한다.
- 각 agent의 plan을 검토할 수 있다.
- Checkout 또는 Worktree가 작업 파일을 격리한다.
- 생성된 diff를 review하고 통합할 시간이 있다.
- Task가 서로 독립적이거나 merge 순서가 명확하다.

따라서 Worktree는 이해를 대신하는 장치가 아니다. 익숙한 영역에서 병렬 처리량을 높이는 격리 도구다.

## Plan이 구현보다 먼저다

인터뷰의 고생산성 workflow에서 사람이 가장 집중하는 지점은 모든 code line을 직접 입력하는 일이 아니라 **plan을 올바르게 만드는 과정**이다. Agent가 구현을 시작하기 전에 문제, scope, 가정과 접근 방식을 주고받는다.

좋은 plan은 다음을 드러내야 한다.

- 어떤 사용자 또는 system behavior를 바꾸는가?
- 어떤 file과 public interface가 영향을 받는가?
- 범위에 포함되지 않는 것은 무엇인가?
- 어떤 test와 command로 결과를 확인하는가?
- 어떤 domain·security 결정은 사람의 승인이 필요한가?
- Task가 병렬 실행 가능한가, 순차 의존적인가?

Plan mode를 사용하는 것만으로 올바른 계획이 보장되지는 않는다. 사람이 검토하지 않은 plan은 agent가 더 빠르게 잘못된 방향으로 가게 만들 수 있다.

## 여러 겹의 검증 구조

Claude Code 팀의 workflow는 AI가 code를 작성하므로 review가 사라지는 구조가 아니다. 오히려 생성량이 커진 만큼 검증을 여러 층으로 둔다.

```text
Agent의 local test와 self-verification
→ Type checker·linter·build 같은 deterministic 검사
→ CI의 AI code review
→ Human engineer의 두 번째 review
→ Production 반영
```

### Local test와 Self-verification

Agent는 관련 test를 실행하거나 새 test를 작성하고, Claude Code 자체를 변경할 때는 subprocess에서 tool을 직접 실행해 end-to-end로 확인한다. 중요한 점은 agent의 완료 설명이 아니라 실제 command와 실행 결과가 남는다는 것이다.

### Deterministic 검사

Non-deterministic AI review만으로는 매번 같은 defect를 잡는다고 보장할 수 없다. 그래서 type checker, linter와 build를 함께 사용한다. 반복되는 code review 지적은 lint rule로 옮겨 사람이 같은 comment를 계속 작성하지 않게 한다.

이 원칙은 다음과 같이 일반화할 수 있다.

```text
반복해서 관찰되는 기계적 문제
→ Lint·static analysis·test로 자동화

Domain 의미와 trade-off
→ Human review에 유지
```

### AI Review와 Human Review

AI review는 첫 번째 filter로 사용하고 engineer가 두 번째 pass와 최종 승인을 맡는다. AI reviewer의 false positive와 누락을 줄이기 위해 여러 agent가 독립적으로 검토하고 별도 agent가 중복·오탐을 정리하는 방식도 사용한다.

Agent 수가 많아져도 human-in-the-loop가 없어지는 것은 아니다. 사용자가 있고 security·privacy·enterprise 품질 기준이 있는 product에서는 사람이 production 변경을 승인한다.

## Agent Teams는 복잡한 작업에 조건부로 사용한다

인터뷰는 agent teams와 swarm을 모든 작업의 기본값으로 제안하지 않는다. 여러 독립 context window가 복잡한 문제에서 단일 agent보다 다양한 접근을 제공할 수 있지만 token 비용이 크고 task에 따라 이득이 달라진다.

적합한 경우는 다음과 같다.

- 단일 agent가 반복해서 막히는 복잡한 조사
- 여러 독립 가설을 동시에 검증해야 하는 문제
- 구현, test와 review를 독립 책임으로 나눌 수 있는 작업
- 여러 prototype을 만든 뒤 하나를 선택하는 탐색

반대로 같은 domain model이나 public interface를 동시에 수정하는 작업은 agent teams보다 순차 실행이 안전할 수 있다. 독립 context는 관점 다양성을 주지만 서로 다른 가정도 함께 만든다.

### 하나의 Feature를 역할별 Context로 나눈다

Pasha의 보완 사례에서는 서로 다른 terminal pane이 다른 feature를 하나씩 소유하기보다, 하나의 feature에서 planner, implementer와 reviewer 역할을 나눠 각 context의 책임을 제한한다. 이 구조는 다중 agent의 가치가 단순한 feature 병렬화에만 있지 않고, spec·implementation·review의 관점 분리에도 있음을 보여준다.

각 context를 분리해도 사람이 spec과 diff를 읽고 역할 사이의 불일치를 통합해야 한다. 여러 feature를 동시에 수정할 때는 별도 repository clone이나 Worktree로 file·branch 상태를 추가 격리한다.

## 문서보다 Prototype을 사용하는 Product Workflow

Claude Code 팀의 사례에서는 역할 경계가 넓고, engineer·designer·data scientist 같은 여러 직군이 code와 prototype을 직접 만든다. 큰 PRD로 미리 합의하기보다 대화와 작동하는 prototype을 반복해 product idea를 비교하는 문화가 설명된다.

이를 모든 조직에 그대로 적용할 수는 없다. 작은 team과 빠른 feedback에서는 prototype이 강력하지만, 여러 team이 의존하는 장기 system에서는 decision record, API contract, migration plan과 운영 문서가 여전히 필요하다.

실용적인 해석은 문서를 없애는 것이 아니라 다음처럼 문서의 역할을 좁히는 것이다.

```text
불확실한 Product idea
→ 빠른 Prototype으로 학습

합의된 Domain·Interface·운영 결정
→ 짧고 명확한 기록으로 고정
```

## AI 시대에도 남는 개발자 역량

### Methodical하고 Hypothesis-driven한 Debugging

인터뷰는 methodical하고 hypothesis-driven한 사고를 여전히 중요한 능력으로 본다. AI가 debugging을 도울 수 있어도 문제 재현, 가설, evidence와 반증을 구분하는 구조는 사람이 이해해야 한다.

```text
증상 관찰
→ 원인 가설
→ 가설을 구분하는 실험
→ 결과 확인
→ 다음 가설 또는 수정
```

오류가 날 때마다 prompt를 바꾸는 것은 hypothesis-driven debugging이 아니다.

### Types-first 사고

Language와 framework 선호에 대한 강한 논쟁의 가치는 줄어들 수 있지만, type signature와 interface로 system을 사고하는 능력은 별개의 문제다. Boris는 model이 작성한 code에서도 type을 먼저 생각한다고 설명한다.

```text
어떤 언어가 더 좋은가?
→ 상대적으로 덜 중요한 선택

어떤 input·output·invariant를 가진 interface인가?
→ 계속 중요한 설계 문제
```

### 아래 Layer 이해하기

현재 사용하는 layer 아래를 이해하라는 원칙도 유지된다. 과거에는 language runtime과 framework 내부가 주요 대상이었다면, AI coding 환경에서는 model의 tool use, context, permission과 failure mode도 이해해야 한다.

### Curiosity, Adaptability와 Generalist 역량

Model 능력과 유효한 workflow가 빠르게 바뀌므로 과거에 실패했던 방식을 영구적인 정답처럼 취급하지 않는다. 새로운 model에서 작은 실험으로 다시 확인하고, engineering뿐 아니라 product, design, business와 운영 맥락까지 이해하는 generalist 능력이 중요해진다.

## 주니어에게 적용하는 단계

### 1. 현재 Branch에서 Single agent 사용

새 codebase에서는 병렬 agent보다 설명 mode와 IDE 탐색을 우선한다. Agent가 찾은 file과 호출 경로를 직접 확인하고 작은 behavior를 TDD로 구현한다.

### 2. Plan review 능력 훈련

Agent가 작성한 plan에서 scope, domain assumption, 예상 file, test와 위험을 찾는다. 설명하지 못하는 plan은 실행하지 않는다.

### 3. Advisory subagent 도입

읽기 전용 code path 조사, test 비판, security 검토와 diff review부터 맡긴다. 핵심 설계와 production code 병렬 구현은 아직 위임하지 않는다.

### 4. 독립 Worktree 실험

문서, test 보강과 폐기 가능한 prototype처럼 독립적인 작업 하나를 Worktree에 둔다. Task·path·branch·base를 함께 기록하고 diff review부터 정리까지 경험한다.

### 5. 익숙한 영역만 제한적으로 병렬화

Codebase와 domain을 이해하고 각 plan과 diff를 감당할 수 있을 때 agent 수를 늘린다. 병렬 agent의 상한은 model 성능이 아니라 사람의 review와 integration capacity다.

## 기존 Wiki 원칙과의 관계

이 인터뷰는 code를 사람이 직접 입력해야만 software engineering이라는 주장과는 맞지 않는다. 구현을 크게 위임하면서도 plan, interface, deterministic verification, human review와 production 책임을 유지할 수 있음을 보여준다.

동시에 높은 생성량이 fundamentals를 불필요하게 만든다는 근거도 아니다. 이 workflow는 강한 test suite, lint rule, build, CI, security layer와 숙련된 human reviewer 위에서 작동한다. 따라서 주니어가 배워야 할 것은 고생산성 숫자보다 그 생산성을 안전하게 만드는 기반과 숙련도별 mode 전환이다.

## 한계

원본은 영어 자동 자막 transcript라 고유명사와 수치에 오인식 가능성이 있다. 인터뷰에 등장하는 model·product 기능과 개인 workflow는 빠르게 바뀔 수 있으며, 자기 보고식 생산성 수치는 독립 benchmark가 아니다. Claude Code 팀의 조직 문화도 다른 규모와 규제 환경에 그대로 일반화할 수 없다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [취업 초기 주니어를 위한 AI-Native 개발 프로세스](junior-ai-native-development-process.md)
- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
- [AI Coding Autonomy Experiment와 Human-in-the-loop](../ai-agents/ai-coding-autonomy-experiment.md)
- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
