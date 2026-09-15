# Git 태그와 GitLab 태그 기반 배포

> Sources: Git Project, Unknown; GitLab, Unknown; Semantic Versioning, Unknown
> Raw: [git-tag Documentation](../../raw/git/git-tag-official-documentation.md); [Tags - GitLab Docs](../../raw/git/gitlab-tags-and-tag-pipelines.md); [Protected tags - GitLab Docs](../../raw/git/gitlab-protected-tags.md); [Create a tag from the command line - GitLab Docs](../../raw/git/gitlab-tag-creation-commands.md); [Tag information in the GitLab UI](../../raw/git/gitlab-tag-ui-information.md); [Semantic Versioning 2.0.0](../../raw/git/semantic-versioning-2.0.0.md)
> Updated: 2026-08-10

## Overview

Git tag는 저장소 이력의 중요한 지점을 이름으로 표시하며, GitLab에서는 tag 생성이 CI/CD pipeline의 실행 조건이 될 수 있다. 배포용 tag를 운영하려면 tag 종류와 생성 방법뿐 아니라 `.gitlab-ci.yml`의 tag 조건, protected tag 권한, 팀의 버전 규칙을 함께 이해해야 한다. 화면에서 commit에 배포처럼 보이는 tag가 붙어 있는 것만으로 실제 배포 실행 여부를 확정할 수는 없으며 CI/CD 설정과 pipeline 이력을 확인해야 한다.

## Lightweight tag와 annotated tag

Lightweight tag는 특정 commit을 직접 가리키는 이름이며 별도 정보를 담지 않는다. Annotated tag는 생성일, tagger의 이름과 이메일, 메시지, 선택적인 암호학적 서명을 담는 tag object다. Git 공식 문서는 annotated tag를 release 용도에, lightweight tag를 개인적이거나 임시적인 객체 표시 용도에 둔다.

Annotated tag는 다음과 같이 만들 수 있다.

```bash
git tag -a v1.0 -m "Version 1.0"
```

생성한 tag들을 upstream으로 push하는 GitLab 문서의 예시는 다음과 같다.

```bash
git push origin --tags
```

`git tag`는 기본적으로 같은 이름이 이미 있으면 실패하지만 `--force`로 기존 tag를 교체할 수 있고 `--delete`로 삭제할 수도 있다. 그러나 이미 공개한 tag를 다른 commit으로 다시 붙이면 사람마다 같은 tag 이름을 서로 다른 내용으로 인식할 수 있다. 공개된 release·deployment tag는 이동시키지 않고 새 이름으로 발행하는 것이 신뢰 가능한 운영 방식이다.

## GitLab tag pipeline

GitLab은 `CI_COMMIT_TAG` predefined variable로 pipeline이 tag에 의해 시작됐는지 식별한다. 이 변수는 job의 `rules:if` 또는 pipeline 수준의 `workflow`에서 사용할 수 있다.

```yaml
deploy-production:
  script:
    - ./deploy-production.sh
  rules:
    - if: $CI_COMMIT_TAG
```

이 예시는 tag pipeline에서 job을 포함시키는 최소 형태다. 실제로 특정 이름의 tag만 배포하려면 프로젝트의 `.gitlab-ci.yml`에 추가된 정규식이나 다른 조건을 확인해야 한다. 별도 규칙이 없는 job은 새 tag의 pipeline에도 포함될 수 있으며, tag pipeline은 tag가 commit을 대상으로 할 때 만들어진다.

따라서 tag가 commit 목록에 표시된다는 사실과 해당 tag가 운영 배포를 실행한다는 사실은 구분해야 한다. 다음 항목을 함께 확인해야 배포 흐름을 판별할 수 있다.

- `.gitlab-ci.yml`의 `CI_COMMIT_TAG`, `rules`, `workflow`
- tag pipeline에서 실제로 실행된 build·deploy job
- GitLab의 tag 화면에 표시되는 pipeline 상태와 artifact

## Protected tag

Protected tag는 누가 tag를 생성할 수 있는지 통제하고 생성된 tag의 우발적인 변경이나 삭제를 방지한다. 개별 이름이나 wildcard로 보호 규칙을 지정할 수 있다. Protected tag를 생성하거나 삭제하려면 해당 tag의 `Allowed to create` 목록에 포함되어야 한다.

Protected tag 생성 권한은 해당 tag에서 CI/CD pipeline을 시작하고 관련 job의 동작을 실행할 수 있는 권한에도 영향을 준다. 배포 tag가 pipeline trigger라면 tag 생성 권한은 곧 배포 시작 권한이 될 수 있으므로, 배포 이름 패턴과 허용 역할을 함께 검토해야 한다.

Branch와 tag에 같은 이름을 사용하면 서로 다른 commit을 가리킬 수 있어 checkout 과정에서 혼동과 운영 문제가 발생할 수 있다. GitLab 문서는 branch와 같은 이름의 tag를 피하도록 권고한다.

## 버전 규칙

Semantic Versioning의 일반 version core는 `X.Y.Z` 형식이며 각각 major, minor, patch를 뜻한다.

- `PATCH`: 하위 호환되는 bug fix
- `MINOR`: 하위 호환되는 public API 기능 추가
- `MAJOR`: public API의 하위 호환성이 깨지는 변경

Pre-release identifier는 patch 뒤에 hyphen으로, build metadata는 plus sign으로 덧붙인다. 이미 release한 version의 내용은 수정하지 않고 변경 사항을 새 version으로 release해야 한다.

숫자가 네 구간인 tag나 환경 이름이 앞에 붙은 tag는 그 자체로 표준 Semantic Versioning의 version core와 일치하지 않을 수 있다. 이런 tag는 오류라고 단정하기보다 팀의 custom versioning convention으로 보고 각 구간과 접두사의 의미, 증가 조건을 확인해야 한다.

## 저장소를 파악할 때 확인할 질문

- 어떤 tag 이름이 개발·검증·운영 환경에 대응하는가?
- tag를 만들면 자동으로 pipeline이 시작되는가, 수동 deploy 승인이 필요한가?
- tag는 어느 branch 또는 commit에서 만들어야 하는가?
- 배포 tag는 annotated tag인가?
- 배포 tag 이름에 protected tag 규칙이 적용되는가?
- version의 각 숫자와 접두사는 무엇을 의미하는가?
- 잘못된 배포와 rollback 시 기존 tag를 재사용하지 않고 어떤 새 이력으로 남기는가?

## 권장 학습 순서

1. Git object와 ref 관점에서 tag가 commit을 가리키는 방식을 익힌다.
2. Lightweight tag와 annotated tag의 차이, tag 생성·push·조회 방식을 실습한다.
3. `.gitlab-ci.yml`에서 `CI_COMMIT_TAG`, `rules`, `workflow`를 읽는 방법을 익힌다.
4. Protected tag와 배포 권한의 연결을 확인한다.
5. 팀의 tag naming·versioning·rollback 규칙을 문서화한다.

## See Also

- [Git Worktree](git-worktree.md)
