# AI Agent Teams에서 Git Worktree를 사용할지 판단하기

> Sources: [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md); [Git Worktree](../git/git-worktree.md)
> Archived: 2026-08-10

## Overview

AI agent teams를 사용한다고 해서 기존의 `git add`, `git commit` 흐름을 버리고 Git Worktree로 바꾸는 것은 아니다. Worktree는 기존 Git 흐름에 병렬 작업 공간을 추가하는 보완 수단이다. 여러 agent가 같은 저장소를 동시에 수정할 때는 agent별 Worktree와 branch를 배정하는 편이 안전하지만, 읽기 전용 조사나 단일 agent의 순차 작업이라면 대개 필요하지 않다.

## 핵심 판단

두 기능은 해결하는 문제가 다르다.

- Agent teams 또는 multi-agent는 일을 나누고 작업자를 조정하며 결과를 통합한다.
- Git Worktree는 각 작업자의 checkout, 파일 변경, `HEAD`를 분리한다.
- Worktree 안에서도 평소처럼 `git add`와 `git commit`을 수행한다.
- 파일 격리는 직접 편집이 섞이는 일을 막지만, 서로 다른 branch가 같은 코드를 바꿨을 때 생기는 최종 merge 충돌까지 없애지는 않는다.

## 권장하는 경우

다음 중 하나라도 해당하면 agent별 Worktree 사용을 우선 검토한다.

- 두 개 이상의 agent가 같은 저장소를 동시에 수정한다.
- 수정 대상 파일이 겹칠 수 있다.
- 각 agent의 결과를 별도 branch나 PR로 검토하고 싶다.
- 한 작업이 오래 실행되는 동안 다른 기능이나 버그 수정을 계속해야 한다.

반대로 병렬 작업이 조사·분석뿐이거나, 한 agent가 순차적으로 수정하거나, 짧은 작업의 파일 소유권이 명확히 분리돼 있다면 단일 checkout으로도 충분하다.

## 최소 운영 방식

```bash
# agent마다 branch와 작업 공간 생성
git worktree add ../project-agent-a -b agent-a/task
git worktree add ../project-agent-b -b agent-b/task

# 각 agent는 자기 worktree에서 기존 Git 흐름 사용
cd ../project-agent-a
git add .
git commit -m "Implement task A"

# 완료 후 확인하고 정리
git worktree list
git worktree remove ../project-agent-a
```

작업을 배정할 때는 agent별 소유 파일과 완료 조건을 먼저 정한다. 각 Worktree는 fresh checkout이므로 dependency와 환경 설정을 준비하고, `.env` 같은 gitignored 파일을 별도로 다뤄야 한다. 작업이 끝나면 branch 통합과 Worktree 정리까지 완료한다.

## 결론

기본 규칙은 “병렬로 코드를 수정하는 agent마다 하나의 Worktree, 읽기 전용 또는 순차 작업에는 기존 checkout”으로 두는 것이 실용적이다. Worktree를 Git 명령의 대체재로 보지 않고 충돌 범위를 줄이는 격리 계층으로 사용하면 된다.

## See Also

- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [Git Worktree](../git/git-worktree.md)
