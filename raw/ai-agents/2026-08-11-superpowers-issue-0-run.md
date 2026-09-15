# Superpowers Brownfield Practice — Issue 0 실행 기록

> Source: 사용자 제공 Claude 실행 요약 스크린샷과 후속 설명
> Collected: 2026-08-11
> Published: 2026-08-11

## 실행 요약 스크린샷 전사

1. `/superpowers:brainstorming` — 저장소 조사 (ESLint·Prettier·테스트·Husky 전부 없음 확인)
2. `/superpowers:brainstorming` — `next lint` 실행 불가 상태 발견
3. `/superpowers:brainstorming` — CI가 `live-*` 태그에만 반응, MR 검증 0건 확인
4. `/superpowers:brainstorming` — Issue 0 설계 작성, 사용자 결정 5건 확정
5. `/superpowers:using-git-worktrees` — worktree 생성 (`feature/issue-0-quality-baseline`)
6. `/superpowers:using-git-worktrees` — `.claude/worktrees/` gitignore 안 됨 → 차단
7. `itnew-dev:itnew-commit` — git user 신원 미설정 발견 → 설정 후 커밋 정정
8. `/superpowers:writing-plans` — 구현 계획 작성 (1,535줄 / 과제 전부 실측)
9. `/superpowers:executing-plans` — 설계·계획 문서 커밋
10. `/superpowers:executing-plans` — baseline 재현 확인 (`tsc` 0, `build` 0)
11. `/superpowers:executing-plans` — Node 버전 고정 (`.nvmrc`, `engines`)
12. `/superpowers:executing-plans` — ESLint 9 도입, error 71건 중 규칙 1개만 warn 강등
13. `/superpowers:executing-plans` — `typecheck` 명령 추가
14. `/superpowers:executing-plans` — Vitest 스모크 테스트 23건 작성
15. `/superpowers:executing-plans` — mutation 검증 중 요일 배열 미검증 발견 → 보강
16. `/superpowers:executing-plans` — Prettier 설정 추가
17. `/superpowers:executing-plans` — `.prettierignore`의 `public/`이 앱 컴포넌트 22개 무검사 → 수정
18. `/superpowers:executing-plans` — Prettier 일괄 포맷 85파일, 무해성 3중 증명
19. `/superpowers:executing-plans` — `prebuild` + `verify` 통합 명령 추가
20. `/superpowers:executing-plans` — CI verify stage 신설 (배포 job 무수정)
21. `itnew-cicd:gitlab-ci-standards` — 사내 CI 표준 대조, 기존 결함 6건 발견
22. `/superpowers:executing-plans` — Landing note 작성
23. `/superpowers:executing-plans` — 게이트 4개 전부 결함 주입해 red 확인
24. `/superpowers:requesting-code-review` — 리뷰어 2명 병렬 투입
25. `/superpowers:requesting-code-review` — 포맷 커밋이 포맷뿐임을 2가지 독립 방법으로 증명받음
26. `(스킬 미호출, 수동)` — 리뷰 지적 3건 전부 재현 후 수정
27. `(스킬 미호출, 수동)` — ESLint가 `.claude/worktree`까지 lint하던 문제 차단
28. `(스킬 미호출, 수동)` — `ConfirmButton` 테스트 핵심 동작 미검증 보강 (25건)
29. `(스킬 미호출, 수동)` — 태그 파이프라인에도 verify 적용 + `interruptible`
30. `(스킬 미호출, 수동)` — 문서의 기존 결함 목록 2건 → 6건 완성
31. `/superpowers:finishing-a-development-branch` — `develop`의 `git blame` → `.gitlab-ci.yml` 변경 전 → 원복
32. `/superpowers:finishing-a-development-branch` — `develop`에 `--no-ff` 병합 (`8ae5c88`)
33. `/superpowers:finishing-a-development-branch` — 병합 결과에서 verify 재확인 (`exit 0`)
34. `/superpowers:finishing-a-development-branch` — worktree 제거, 브랜치 삭제
35. `update-config` — `worktree.baseRef: head` 설정

호출한 스킬 8개: `brainstorming` · `using-git-worktrees` · `writing-plans` · `executing-plans` · `requesting-code-review` · `finishing-a-development-branch` · `itnew-dev:itnew-commit` · `itnew-cicd:gitlab-ci-standards` · `update-config`

26~30번은 스킬 없이 처리했습니다. `superpowers:receiving-code-review`가 있는데 호출하지 않았습니다. 행동(지적을 검증 후 수용/반박)은 스킬 절차대로 했지만 절차를 따른 게 아니라 제 판단이었습니다.

## 사용자 후속 설명

- 개인 연습용 Integration branch는 만들지 않을 계획이다.
- Push 자체를 하지 않고 있다.
- Issue 0의 Worktree를 언제 병합하고, Issue 1 이후 매번 Worktree를 생성해야 하는지 의문이 생겼다.
- 매번 Worktree를 만들면 이전 Issue의 코드가 포함되지 않을 수 있다고 우려했다.

