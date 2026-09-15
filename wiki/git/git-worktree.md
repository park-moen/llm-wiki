# Git Worktree

> Sources: 또 만드는 한톨, 2025-09-13; Git Documentation, Unknown
> Raw: [Git Worktree로 브랜치 사이를 효율적으로 거닐기](../../raw/git/2025-09-13-git-worktree-efficient-branch-workflow.md); [git-worktree Documentation](../../raw/git/git-worktree-official-documentation.md)
> Updated: 2026-08-10

## Overview

Git Worktree는 하나의 Git 저장소와 이력을 공유하면서 브랜치나 커밋별로 독립된 작업 디렉터리를 운용하는 방법이다. 진행 중인 작업을 임시 commit이나 stash로 치워두고 브랜치를 전환하는 대신, 다른 디렉터리로 이동해 코드 리뷰·핫픽스·병렬 빌드 같은 작업을 수행할 수 있다.

## 브랜치 전환과 다중 clone의 문제

하나의 작업 디렉터리에서 브랜치를 계속 전환하면 현재 작업을 임시 commit이나 stash로 보존하고 나중에 복원해야 한다. 브랜치마다 의존성이 달라 프로젝트 생성이나 clean build가 필요하다면 전환 비용도 커질 수 있다.

저장소를 여러 번 clone하면 디렉터리별로 독립된 작업 공간을 얻지만 다음 부담이 생긴다.

- clone마다 전체 저장소를 내려받아 네트워크와 디스크를 추가로 사용한다.
- 각 clone의 작업 내용과 최신화 시점을 따로 관리해야 한다.
- 한 clone의 로컬 변경은 원격 저장소를 통하거나 직접 옮기기 전까지 다른 clone과 공유되지 않는다.

## Worktree의 모델

Main Worktree는 처음 `git clone`했을 때 만들어지는 기본 작업 디렉터리다. 여기에 연결된 추가 worktree들은 같은 Git 저장소와 이력을 공유하면서도 서로 다른 디렉터리에서 독립적으로 작성·빌드·테스트할 수 있다.

공식 Git 문서의 모델에서는 저장소에 하나의 main worktree와 0개 이상의 linked worktree가 연결된다. linked worktree마다 `HEAD` 같은 worktree 전용 관리 정보가 따로 있지만 브랜치 참조와 저장소 이력은 공유한다. 즉 작업 디렉터리와 현재 브랜치 상태는 격리하면서 저장소 자체를 중복 clone하지 않는다.

따라서 진행 중인 기능 작업을 그대로 둔 채 별도 worktree에서 핫픽스를 처리하거나, 팀원의 여러 브랜치를 각각의 디렉터리에 열어 코드 리뷰할 수 있다.

## 기본 사용 흐름

기존 `develop` 브랜치를 기준으로 상위 디렉터리에 hotfix worktree를 만든다.

```bash
git worktree add ../hotfix develop
```

원본 프로젝트 내부가 아니라 상위 디렉터리에 만들면 worktree 디렉터리 자체가 기존 프로젝트의 변경 사항으로 잡히는 일을 피할 수 있다. 생성된 worktree는 다음 명령으로 확인한다.

```bash
git worktree list
```

이후 디렉터리로 이동해 별도의 hotfix 브랜치를 만들고 작업한다.

```bash
cd ../hotfix
```

## 브랜치 제약

모든 worktree가 같은 Git 저장소를 바라보기 때문에 하나의 브랜치는 하나의 worktree에서만 접근하도록 제한된다. 다른 worktree가 이미 사용 중인 브랜치를 checkout하려면 충돌하게 된다.

`--force`로 강제할 수 있지만 원본 글은 로컬 파일 손실 위험을 이유로 권장하지 않는다. 여러 worktree에서 비슷한 작업이 필요하다면 worktree마다 별도 브랜치를 만드는 방식을 권장한다.

## 주요 명령어

```bash
# worktree 목록 보기
git worktree list

# 새 worktree 추가
git worktree add {path}

# 기존 branch로 worktree 추가
git worktree add {path} {branch}

# 새로운 branch를 만들지 않고 detached worktree 추가
git worktree add -d {path}

# worktree 삭제
git worktree remove {path}

# 수동 삭제 뒤 남은 관리 정보 정리
git worktree prune

# 디렉터리를 수동으로 옮겨 연결이 깨졌을 때 복구
git worktree repair {path}
```

`git worktree add {path}`에서 브랜치를 생략하면 기본적으로 경로의 마지막 이름을 사용하는 새 브랜치가 만들어진다. 삭제는 변경 파일과 추적되지 않은 파일이 없는 clean worktree에만 기본 허용된다.

## 한계와 주의점

공식 문서는 다중 checkout 기능을 여전히 experimental로 표시하고 submodule 지원이 불완전하다고 밝힌다. 특히 submodule을 포함하는 superproject를 여러 worktree로 checkout하는 방식은 권장하지 않는다.

## 활용 사례

- 진행 중인 기능 개발을 유지한 채 별도 디렉터리에서 긴급 핫픽스 처리
- 여러 팀원의 PR 브랜치를 각각의 worktree로 열어 코드 리뷰
- 브랜치마다 다른 의존성을 반복해서 다시 생성하지 않고 독립적인 빌드 환경 유지
- 한 worktree에서 빌드하는 동안 다른 worktree에서 별도 작업 진행

## See Also

- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
