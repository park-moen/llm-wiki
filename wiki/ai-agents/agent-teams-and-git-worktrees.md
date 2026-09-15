# AI Agent Teams와 Git Worktree

> Sources: OpenAI, Unknown; Anthropic, Unknown; Git Documentation, Unknown; Boris Cherny interview, YouTube, Unknown; Pasha interview·Beyond Coding, Unknown
> Raw: [Model guidance — Multi-agent](../../raw/openai/model-guidance-multi-agent.md); [Run agents in parallel](../../raw/ai-agents/claude-code-run-agents-in-parallel.md); [Run parallel sessions with worktrees](../../raw/ai-agents/claude-code-worktree-isolation.md); [git-worktree Documentation](../../raw/git/git-worktree-official-documentation.md); [Building Claude Code with Boris Cherny transcript](../../raw/software-career/building-claude-code-boris-cherny.md); [Original YouTube source provenance](../../raw/software-career/building-claude-code-boris-cherny-source-provenance.md); [From Backend Engineer to Head of Mobile transcript](../../raw/software-career/from-backend-engineer-to-head-of-mobile-lessons-uber.md)
> Updated: 2026-08-16

## Overview

Multi-agent와 Git Worktree는 서로 대체하는 기능이 아니다. Multi-agent 또는 agent teams는 작업 분배와 결과 통합을 담당하고, Worktree는 동시에 코드를 수정하는 작업자의 파일과 브랜치 상태를 분리한다. 여러 agent가 독립적으로 조사만 하거나 서로 다른 파일을 맡는다면 Worktree가 필수는 아니지만, 같은 저장소를 병렬로 수정한다면 충돌 위험과 작업 경계를 기준으로 도입할 가치가 커진다.

## 조정과 격리는 별개의 문제

OpenAI 공식 모델 가이드는 multi-agent를 여러 subagent를 병렬로 조정하고 결과를 종합하는 기능으로 설명한다. 독립적인 작업 흐름으로 깔끔하게 나눌 수 있는 복잡한 작업에서 수행 시간을 줄이고 성능을 높일 수 있다는 설명이다. 이 문서는 multi-agent의 작업 조정 역할을 설명할 뿐, Git Worktree를 사용하라는 지침은 제공하지 않는다.

Claude Code 공식 문서는 이 둘을 더 직접적으로 구분한다. Subagent와 agent teams는 작업 자체를 조정하고, Worktree는 각 세션의 파일 변경을 별도 checkout으로 격리한다. 특히 agent teams의 teammate가 자동으로 Worktree에 격리되는 것은 아니므로, Worktree를 쓰지 않는다면 각 teammate가 서로 다른 파일 집합을 소유하도록 작업을 나눠야 한다.

## 기존 Git 작업 흐름과의 관계

Worktree는 `git add`, `git commit`, branch, merge를 대체하지 않는다. 각 Worktree는 독립된 작업 디렉터리와 `HEAD`를 가지지만 같은 저장소의 Git 이력과 remote를 공유하고, Worktree 안에서도 `git commit`이 동작한다. 따라서 Worktree는 기존 Git 흐름 위에 병렬 작업 공간을 추가하는 기능으로 이해해야 한다.

일반적인 흐름은 다음과 같다.

```bash
# agent별 새 브랜치와 worktree 생성
git worktree add ../project-agent-a -b agent-a/task
git worktree add ../project-agent-b -b agent-b/task

# 각 worktree 안에서 기존 Git 흐름 사용
cd ../project-agent-a
git add .
git commit -m "Implement task A"

# 작업 공간 확인 및 정리
git worktree list
git worktree remove ../project-agent-a
```

## 언제 사용하는가

| 상황 | 판단 | 이유 |
|---|---|---|
| 여러 agent가 같은 저장소를 동시에 수정 | Worktree 권장 | 작업 디렉터리와 브랜치 상태를 분리한다. |
| 작업 범위가 겹치거나 같은 파일을 수정할 가능성이 큼 | Worktree 권장 | 한 세션의 편집이 다른 세션의 파일 상태에 직접 섞이는 일을 막는다. 최종 merge 충돌 가능성 자체는 남는다. |
| agent별 독립 branch와 PR이 필요 | Worktree 권장 | 각 작업 공간과 branch의 대응 관계가 명확해진다. |
| 조사·분석처럼 파일을 수정하지 않는 병렬 작업 | 대개 불필요 | 격리할 파일 변경이 없다. |
| 한 agent만 순차적으로 수정 | 대개 불필요 | 단일 작업 디렉터리로 충분하다. |
| 서로 다른 파일을 명확히 소유하고 짧게 작업 | 선택 사항 | 파일 분할만으로도 충돌 위험을 낮출 수 있다. |

## 운영 비용과 주의점

- Worktree는 fresh checkout이므로 각 디렉터리에서 dependency 설치나 build 설정이 필요할 수 있다.
- `.env` 같은 gitignored 파일은 자동으로 따라오지 않으므로 안전한 별도 설정이 필요하다.
- 작업이 끝난 branch의 통합과 Worktree 정리 책임이 생긴다.
- 여러 agent를 동시에 실행하면 token 사용량도 함께 증가한다. 이는 Worktree 자체의 비용이 아니라 병렬 agent 실행의 비용이다.
- Worktree는 파일을 격리하지만 논리적으로 같은 코드를 바꾼 branch 사이의 merge 충돌까지 제거하지는 않는다.

## 실용적인 선택 기준

처음부터 모든 agent에 Worktree를 강제하기보다는, 병렬로 파일을 수정하는 agent마다 하나씩 배정하는 규칙이 단순하다. 먼저 업무를 독립된 단위로 나누고 파일 소유권을 정한 뒤, 수정 범위가 겹치거나 장시간 실행되는 작업만 별도 Worktree에 둔다. 작은 읽기 전용 조사나 순차 작업은 기존 checkout에서 수행해 관리 비용을 줄일 수 있다.

## 숙련도에 따른 학습 Mode와 생산 Mode

Boris Cherny가 설명한 개인 workflow는 Worktree 사용 여부보다 **codebase 친숙도**를 먼저 구분한다. 새 codebase에서는 learn 또는 explanatory mode로 agent의 행동을 따라가며 구조를 익히고, 익숙한 codebase에서는 plan을 먼저 조정한 뒤 여러 checkout이나 managed Worktree에서 agent를 병렬 실행한다.

```text
새 Codebase
→ 설명을 따라가며 Code path·convention 학습
→ Single agent와 작은 변경 우선

익숙한 Codebase
→ Plan 검토
→ 독립 Task 분리
→ Checkout·Worktree 격리
→ 병렬 실행과 Diff review
```

인터뷰의 여러 terminal checkout은 한 숙련 개발자의 workflow이며 보편적인 기본값이 아니다. Managed Worktree 기능은 command-line directory 관리 부담을 줄이지만, 각 plan과 diff를 이해하고 통합해야 하는 사람의 attention 비용까지 제거하지는 않는다. 병렬 agent의 상한은 생성 가능한 task 수보다 사람이 review하고 merge할 수 있는 task 수로 정한다.

## Feature 분할과 역할 분할을 구분한다

Parallel context는 반드시 agent별로 다른 feature를 맡기는 구조가 아니다. Pasha의 사례는 하나의 feature를 plan, implementation과 review로 나눠 서로 다른 terminal context에 두는 방식을 보여준다.

```text
역할 분할
→ 하나의 Feature를 서로 다른 관점으로 검토
→ Context 오염을 줄이지만 통합 책임은 사람에게 남음

Feature 분할
→ 여러 변경을 동시에 진행
→ Repository clone·Worktree·branch 격리가 추가로 필요
```

인터뷰이는 개인의 기존 Git 습관 때문에 여러 repository clone을 Worktree보다 편하게 사용했다. 이는 Worktree의 일반적 열세를 의미하지 않는다. 중요한 것은 어떤 도구를 선택하든 feature·branch·directory·context의 대응 관계를 사람이 파악할 수 있어야 한다는 점이다.

## See Also

- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](../software-career/claude-code-team-ai-native-development-workflow.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [Git Worktree](../git/git-worktree.md)
- [AI를 활용한 개발자 성장과 Career 판단](../software-career/ai-assisted-engineering-growth-and-career-judgment.md)
