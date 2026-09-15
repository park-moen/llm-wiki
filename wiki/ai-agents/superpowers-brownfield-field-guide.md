# Superpowers Brownfield 실전 가이드

> Sources: Personal Superpowers Practice, 2026-08-11; Git Documentation, Unknown; Anthropic, Unknown; Jesse Vincent interview, YouTube (Unknown)
> Raw: [Issue 0 실행 기록](../../raw/ai-agents/2026-08-11-superpowers-issue-0-run.md); [git-worktree Documentation](../../raw/git/git-worktree-official-documentation.md); [Run parallel sessions with worktrees](../../raw/ai-agents/claude-code-worktree-isolation.md); [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md)
> Updated: 2026-08-16

## Overview

안동 Brownfield project의 Issue 0에서 Superpowers를 실제로 실행한 첫 관찰 기록이다. 저장소 조사부터 품질 baseline 구축, code review, local `develop` 병합과 Worktree 정리까지 초기 시뮬레이션과 유사하게 진행됐다. 다만 push하지 않았으므로 실제 GitLab Runner와 MR pipeline은 아직 검증하지 않았고, review 반영 과정에서 `receiving-code-review` Skill을 명시 호출하지 않은 차이도 있었다.

## 현재 검증 수준

> **관찰 범위:** 특정 local repository의 Issue 0 품질 baseline 작업 1회

확인된 것은 다음과 같다.

- `brainstorming`으로 기존 도구와 CI 결손을 조사했다.
- Issue 전용 branch와 Worktree를 만들었다.
- `writing-plans`와 `executing-plans`를 거쳐 lint·typecheck·test·format·CI verify 기반을 추가했다.
- `requesting-code-review`에서 발견한 문제를 재현하고 수정했다.
- 작업 branch를 local `develop`에 병합한 뒤 verify `exit 0`을 확인했다.
- 병합 후 Worktree와 작업 branch를 제거했다.

이 결과는 Issue 0에서 workflow가 작동했다는 관찰이지, 모든 repository와 모든 Superpowers Skill에 대한 일반 검증은 아니다.

## 원 설계의 의도와 실측의 대응

Jesse Vincent의 설명에서 Superpowers는 brainstorming으로 사람의 실제 의도를 끌어내고, lightweight spec을 작은 RED–GREEN task로 변환한 뒤 implementer·spec reviewer·quality reviewer의 역할을 분리하는 방법론이다. 따라서 Issue 0에서 관찰한 단계 이름만 실행됐는지보다 다음 목적이 실제로 달성됐는지를 봐야 한다.

- Brainstorming이 agent의 독단적 설계가 아니라 사람의 요구와 trade-off를 명확히 했는가?
- Plan이 다음 구현자가 project를 추측하지 않아도 될 만큼 작은 검증 단위였는가?
- Review가 구현자의 자기 확인과 분리됐는가?
- Agent의 완료 선언 외에 command output과 실제 behavior proof가 남았는가?

인터뷰 시점의 Superpowers에는 별도 behavioral testing 단계가 완전히 내장돼 있지 않다고 설명된다. Issue 0의 local verify 통과도 실제 사용자 behavior와 remote CI까지 자동으로 증명하지 않으므로, 해당 proof는 repository별 harness에서 별도로 붙인다.

## Issue 0에서 관찰한 흐름

```text
brainstorming
→ using-git-worktrees
→ writing-plans
→ executing-plans
→ requesting-code-review
→ review 지적 재현·수정
→ finishing-a-development-branch
→ local develop 병합
→ 병합 결과 verify
→ Worktree·branch 정리
```

실행 과정에서 기존 `tsc`와 build baseline을 먼저 확인했고, ESLint·typecheck·Vitest·Prettier와 CI verify를 추가했다. Vitest 스모크 테스트는 mutation 검증을 통해 누락된 동작을 발견해 보강했다. Review 단계에서는 format-only 변경을 독립적인 방법으로 확인하고 기존 `.gitlab-ci.yml` 문제를 Issue 범위에서 분리했다.

## Worktree 운영 결론

Worktree 자체를 병합하는 것이 아니라 Worktree가 checkout한 branch를 기준 branch에 병합한다.

```text
Issue branch의 작업 완료
→ local develop에 branch 병합
→ local develop에서 전체 검증
→ Worktree 제거
→ 다음 Issue를 갱신된 local develop에서 분기
```

개인 Integration branch나 push는 local 연습의 필수 조건이 아니다. 이번 실행에서는 Issue 0가 local `develop`에 병합됐으므로 다음 Issue branch를 local `develop`에서 만들면 Issue 0 변경이 포함된다.

Worktree는 작은 Task마다 새로 만들지 않고 독립적으로 review·병합할 Issue마다 하나를 사용하는 것이 단순하다.

## 다음 Issue를 시작할 때 확인할 것

- 기준이 `origin/develop`이 아니라 Issue 0가 병합된 local `develop`인지 확인한다.
- Issue 0 merge commit `8ae5c88`이 새 branch의 ancestor인지 확인한다.
- Issue 0에서 추가한 lint·typecheck·test·build 명령이 보이는지 확인한다.
- Worktree 생성 전후 `git status`가 clean한지 확인한다.

`worktree.baseRef: head` 설정은 현재 `HEAD`가 어느 branch인지에 따라 기준이 달라질 수 있다. 다음 Issue 프롬프트에는 `local develop`과 필요한 이전 merge commit을 명시해 기준점을 확인하는 편이 안전하다.

## 아직 검증되지 않은 부분

Push하지 않았기 때문에 다음 항목은 미검증 상태다.

- 실제 GitLab Runner 실행
- MR pipeline
- Remote review와 merge

따라서 현재 결과는 “CI 완료”보다 “local verify 통과와 CI 설정 정적 검토 완료”라고 기록하는 편이 정확하다.

또한 review 지적을 검증하고 수정했지만 `receiving-code-review` Skill을 직접 호출하지 않았다. 다음 Issue에서는 이 Skill을 명시적으로 사용해 호출 기록과 실제 행동이 일치하는지 관찰할 수 있다.

## 다음 실행에서 유지할 규칙

- 조사와 설계가 끝난 뒤 Issue 단위 Worktree를 만든다.
- 다음 Issue는 필요한 이전 Issue가 병합된 local 기준 branch에서 분기한다.
- 변경 후뿐 아니라 merge 결과에서도 전체 verify를 다시 실행한다.
- Review 지적은 즉시 수용하지 않고 재현한 뒤 수정한다.
- 대규모 format-only 변경은 기능 변경과 구분하고 무해성을 검증한다.
- Push하지 않은 경우 remote CI를 검증했다고 표현하지 않는다.

## See Also

- [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](superpowers-brownfield-practice-workflow-initial-design.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [Git Worktree](../git/git-worktree.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
