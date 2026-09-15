# Orca CLI Worktree Handoff 실전 기록 — 4레인 병렬 개발과 Claude Code 병렬 Agent 비교

> Source: 2026-08-19 안동 세계유산축전 홈페이지(fe) 저장소에서 Jira TNSP-1233 하위 작업 8건을 Orca 병렬 레인으로 실행한 세션
> Collected: 2026-08-19 ~ 2026-08-20
> 성격: 실전 field note. 아래 명령·수치·오류는 모두 실제 실행 결과이며, 추정은 "추정"으로 표시했다.
> Corrected: 2026-08-20. 사용자의 명시적 허가로 실제 실행 방식을 Orca orchestration이 아닌 Orca CLI handoff로 바로잡고, Claude Code subagent와 Agent Teams를 구분했다.

---

## 1. 무엇을 하려 했나

Jira 스토리 **TNSP-1233 「데모 시연 사용자 화면 피드백 반영」** 의 하위 작업 8건을 한 번에 병렬 처리하는 것이 목표였다.

| 키 | 요약 |
| --- | --- |
| TNSP-1236 | 메인 페이지 Hero 개편 |
| TNSP-1237 | 상단 메뉴 진입 경로 랜딩 페이지로 통합 |
| TNSP-1238 | 데모 1 축전 페이지 가독성·탐색 개선 |
| TNSP-1239 | 데모 2 Hero 표시 정리 |
| TNSP-1240 | 데모 2 Hero 자동 전환 규칙 개선 |
| TNSP-1241 | 데모 2 Hero 모바일 동작·레이아웃 개선 |
| TNSP-1242 | 축전 상세 내부 영역 흰색 배경 적용 |
| TNSP-1243 | 사용자 화면 브라우저·기기 검수 및 보완 |

---

## 2. 실제로 사용한 Orca CLI 명령

### 2.1 사전 조사

```bash
orca --help                    # 명령 목록
orca agent-context --json      # 기계 판독용 스키마
orca account list              # 관리 계정 (Claude 2, Codex 1)
orca repo list --json          # repo id / setup hook / gitUsername
orca worktree list --json      # 기존 worktree
orca worktree create --help    # 옵션 확인
```

`orca repo list --json` 에서 얻은 것 — repo id, `hookSettings.scripts.setup = "npm install"`, `setupRunPolicy = run-by-default`. 레인 생성 시 npm install 이 자동 실행된다는 뜻이다.

### 2.2 레인 생성 — Orca CLI handoff (핵심 명령)

```bash
orca worktree create \
  --repo name:fe \
  --name tnsp-1236-main-hero \
  --base-branch develop \
  --agent claude \
  --prompt "$(cat lane-a.txt)" \
  --no-parent \
  --json
```

`--prompt` 는 문자열 하나만 받으므로, 긴 지시는 파일로 써 두고 `"$(cat file)"` 로 넘겼다. 이 방식이 한글 이스케이프 깨짐 없이 동작했다(quoted heredoc 으로 파일 작성).

`--base-branch` 가 이번 배치의 축이었다. 레인마다 base 가 달랐다.

### 2.3 관제 (폴링)

```bash
# 커밋 진행도 — base 3개를 제외해 레인 고유 커밋만 셈
git -C <worktree> log --oneline <branch> --not develop feat/demo1 feat/demo2

# 미커밋 변경
git -C <worktree> status --porcelain | wc -l

# 터미널 생존 확인
orca terminal list --worktree path:<worktree> --json
```

`terminal list` 응답 필드: `handle, ptyId, incarnationId, orphaned, worktreeId, worktreePath, branch, tabId, leafId, title, connected, writable, lastOutputAt, preview`

실제로 쓸모 있던 세 필드:

- **`title`** — 앞에 브레일 스피너(`⠂ ⠐ ✳ ✢`)가 붙는다. 돌면 작업 중, 없으면 멈춤. 게다가 에이전트가 스스로 붙인 제목이 들어가 어느 티켓을 하는지 보인다.
- **`lastOutputAt`** — epoch ms. 몇 초 전에 움직였는지.
- **`preview`** — 마지막 출력. 질문을 던지고 멈춰 있는지 판별 가능.

### 2.4 레인에 중간 지시

```bash
orca terminal send --terminal term_<uuid> --text "..." --enter
```

터미널 handle 은 `terminal list` 의 첫 번째 항목이 에이전트 터미널, 두 번째가 일반 셸이었다(레인 생성 시 2개가 만들어짐).

---

## 3. 레인 배치 설계와 그 근거

병렬화의 단위를 **파일이 아니라 base 브랜치**로 잡은 것이 이번 배치의 핵심이었다.

조사 결과 데모 1과 데모 2는 라우트 분기나 환경변수 분기가 아니라 **별도 git 브랜치**였다.

- `feat/demo1` — 자연 스크롤 + GSAP 핀 (`HeroPin.tsx` 존재)
- `feat/demo2` — 가로 패널 확장 (`Panel.tsx`/`PanelGroup.tsx` 존재, `HeroPin.tsx` 삭제됨)

근거: `HANDOFF.md:18,21,45`, `docs/superpowers/specs/2026-08-13-demo-vercel-deployment-design.md:208`

최종 배치:

| 레인 | base | 담당 | 내부 순서 |
| --- | --- | --- | --- |
| A | develop | 1236 | — |
| B | develop | 1237 | — |
| C | feat/demo1 | 1238, 1242(데모1 몫) | 순차 |
| D | feat/demo2 | 1239, 1240, 1241, 1242(데모2 몫) | **순차 강제** |

레인 D 를 순차로 묶은 이유: 1240 과 1241 이 `PanelGroup.tsx` 의 같은 40줄 영역(112-148, 209-250)을 나눠 건드리고 타이머 게이트(`autoPlayOn` → `ticking`)를 공유한다. 병렬로 뗄 수 없었다.

이 판단은 실제로 값어치를 했다. 레인 D 의 1241 커밋 메시지:

> 패널 높이를 줄여 넷이 한 화면에 들어오게 한다. 접힘 5.5→5rem, 열림 26→21rem. (…) 26rem 이었던 이유는 패널 안에 회차명 타이틀이 있었기 때문인데 TNSP-1239 에서 그것을 없앴다.

즉 1239 를 먼저 끝냈기 때문에 1241 에서 높이를 줄일 수 있었다. 순서를 뒤집었으면 두 번 고쳤을 작업이다.

---

## 4. 발생한 문제

### 4.1 `--base-branch` 의 브랜치 생성 동작이 일관되지 않았다

같은 형식의 명령 4개를 실행했는데 결과가 갈렸다.

```
--base-branch develop     → refs/heads/park-moen/tnsp-1236-main-hero  (새 브랜치)
--base-branch develop     → refs/heads/park-moen/tnsp-1237-gnb-landing (새 브랜치)
--base-branch feat/demo1  → refs/heads/feat/demo1                      (기존 브랜치 그대로!)
--base-branch feat/demo2  → refs/heads/park-moen/tnsp-1239-demo2-hero  (새 브랜치)
```

레인 C 만 `feat/demo1` 을 직접 체크아웃했다. `--name tnsp-1238-demo1-hero` 를 줬는데도 그랬다. **시연 중인 브랜치에 에이전트가 직접 커밋하게 되는 상태**였다.

`feat/demo1` 이 직전까지 다른 worktree(hawksbill)에 점유돼 있다가 그 worktree 가 사라지면서 자유로워진 직후였다는 점이 관련 있을 것으로 **추정**되나 확정하지 못했다.

조치 — 작업 시작 전(working tree clean) 수동 교정:

```bash
git -C <lane-c> switch -c park-moen/tnsp-1238-demo1-hero
```

**교훈: `worktree create` 직후 `branch` 필드를 반드시 확인할 것.** `--json` 응답의 `result.worktree.branch` 를 읽어 기대와 다르면 즉시 교정.

### 4.2 Orca CLI handoff에는 완료 계약이 없었다

이번 레인은 `orca worktree create --agent claude --prompt ...`로 작업 전체를 넘긴 handoff였다. Run·Task·Dispatch를 만들거나 lifecycle preamble을 주입하지 않았으므로, 작업이 끝나도 coordinator에게 `worker_done`이 전달되는 완료 계약이 없었다. 이 실행 방식에서는 terminal과 Git 상태를 폴링해 완료를 판별했다.

이는 Orca orchestration 자체에 완료 통보가 없다는 뜻이 아니다. Supervised orchestration은 Run·Task·Dispatch를 연결하고 worker가 `worker_done`을 보내며 coordinator가 `check --wait`로 완료·질문·escalation을 기다리는 별도 방식이다.

이 세션에서는 `/loop` 를 동적 페이싱으로 걸어 10분 간격 3회, 25분 간격 1회 점검했다.

### 4.3 `.env` 가 레인에 복사되지 않는다

setup hook 이 `npm install` 만 실행하므로 gitignore 된 `.env` 는 레인에 없다. dev 서버를 띄우려면 수동 복사가 필요했다.

```bash
cp /path/to/main-repo/.env /path/to/lane/.env
```

**주의**: 이 저장소는 데모 브랜치들이 같은 Supabase DB 를 공유한다. `.env` 를 그대로 복사하면 4레인이 같은 DB 를 본다. 이번 배치는 스키마 변경이 없어 안전했지만, 마이그레이션이 있는 작업이면 별도 설계가 필요하다.

### 4.4 dev 서버 포트 충돌

레인마다 포트를 다르게 지정해야 한다. 프롬프트에 명시했다(3101/3102/3103/3104).

### 4.5 에이전트가 지시 범위를 넘어 자율 행동한다

프롬프트에 "커밋한다"까지만 쓰고 push/PR 을 언급하지 않았더니 레인마다 다르게 행동했다.

- 레인 A — 커밋만 하고 멈춤
- 레인 B — **push 하고 PR(#15)까지 자율 생성**
- 레인 C — 나중에 PR 지시를 받자 즉시 실행 (중단 지시가 도착하기 전에 완료)

**교훈: 프롬프트에 "어디서 멈출지"를 명시적으로 쓸 것.** 안 쓰면 에이전트가 합리적으로 끝까지 간다.

### 4.6 `agent-context --json` 에서 agent id 목록을 찾지 못함

`--agent` 에 넣을 수 있는 값의 목록을 스키마에서 추출하려 했으나 실패했다. `orca account list` 출력("Managed Claude accounts", "Managed Codex accounts")과 `--help` 예시(`--agent codex`)로부터 `claude` 를 추론해 사용했고 동작했다.

### 4.7 worktree 가 예고 없이 사라진 사건

레인 생성 직전 조회에서 `hawksbill [feat/demo1]` 이 보였는데, 레인 생성 후 디렉토리도 Orca 기록도 사라졌다. `.orca-worktree-trash` 디렉토리 타임스탬프(17:59)가 첫 `worktree create` 실행 시각(18:02)보다 앞서므로 CLI 가 지운 것은 아니다. 원인 미확정.

**교훈: `git worktree list` 는 디렉토리가 지워져도 prune 전까지 stale 항목을 보여준다.** 점유 여부를 믿기 전에 실제 디렉토리 존재를 확인할 것.

---

## 5. 사용 방식에서 어긋났던 지점

양쪽 모두의 실수를 사실대로 기록한다.

### 5.1 오케스트레이터(Claude) 쪽

**① 커밋 메시지 관례를 확인하지 않고 지시했다.**
프롬프트에 `[#TNSP-1236]` 형식을 예시로 넣었으나, 이 저장소의 실제 관례는 작업명 slug 였다.

```
최근 40개 커밋 prefix 집계:
  6 [#hero-frames]
  6 [#demo-deploy]
  3 [#hero-frames-admin-fixes]
  3 [#demo2-hero-admin-wiring]
  2 [#fix-demo1-device-qa]
  ...
  TNSP 키 사용: 0건
```

예시를 안 준 레인 D 만 `git log` 를 직접 확인하고 관례를 따랐다(`[#tnsp-1239-demo2-hero]`). **지시가 없을 때가 더 정확했던 역설.**

**② PR 정책을 먼저 정하지 않았다.** 4.5 의 원인.

**③ 로컬 확인 단계를 배치 설계에 넣지 않았다.** "PR 까지 내게 할까?" 를 물으면서 선택지에 "로컬 확인 후 PR" 을 넣지 않았다. 사용자가 뒤늦게 "PR 전에 내가 확인하고 수정할 텐데?" 라고 지적했을 때는 이미 PR #17 이 나간 뒤였다. **UI 작업은 눈으로 판정되므로 확인 단계가 기본값이어야 했다.**

### 5.2 사용자 쪽

**① Orca 앱에서 직접 한 행동을 오케스트레이터에게 알리지 않았다.**
레인 A 의 PR(#16)을 앱에서 직접 만들었는데, 오케스트레이터는 그 사실을 모른 채 레인 A 에 PR 생성 지시를 보내려던 참이었다. 이중 지시가 될 뻔했다.

hawksbill worktree 제거(4.7)도 마찬가지로 공유되지 않았다.

**② 지시 채널이 둘로 갈릴 뻔했다.** 오케스트레이터가 `terminal send` 로, 사용자가 Orca 앱 UI 로 같은 레인에 지시하면 레인이 모순된 명령을 받는다. 세션 후반에 "각 workspace 에서 수정 지시하고 최종 PR 만 여기서 관리" 로 역할을 갈라 해결했다.

**교훈: 앱에서 한 행동을 오케스트레이터에게 알리거나, 처음부터 지시 채널을 하나로 정할 것.**

---

## 6. Claude Code subagent와 Agent Teams를 구분한 비교

같은 세션에서 실제로 사용한 것은 Orca CLI handoff와 Claude Code `Agent` tool 기반 subagent였다. Claude Code Agent Teams는 실제로 실행하지 않았다. 따라서 아래에서 subagent 열은 실측이고, Agent Teams 열은 shared task list와 inter-agent messaging을 제공한다는 공식 기능 설명을 바탕으로 한 구조 비교다.

- 조사 단계 — Claude Code `Agent` 툴로 서브에이전트 **7개**(Explore 5개 + 재조사 2개)
- 구현 단계 — Orca 레인 **4개**

| 항목 | Orca CLI handoff 레인 | Claude subagent (`Agent` tool, 실제 사용) | Claude Agent Teams (미사용) |
| --- | --- | --- | --- |
| 작업 조정 | 외부 coordinator가 terminal·Git 상태를 직접 관제 | Parent가 subagent별 결과를 회수 | Lead가 shared task list와 message로 teammate 조정 |
| 완료 통보 | 계약 없음 → 이번에는 폴링 | `<task-notification>` 자동 도착 | Task 상태와 inter-agent messaging 사용 |
| 결과 회수 | 커밋·diff·PR을 직접 읽음 | 보고가 parent context로 들어옴 | Lead와 teammate 사이 message로 공유 |
| 사용자 가시성 | Orca 앱에서 terminal 관찰·직접 개입 | Parent session 안에서 결과 확인 | Lead가 팀을 관리하며 Orca 레인과는 별도 lifecycle |
| 지속성과 격리 | 별도 Orca Worktree와 process가 유지됨 | Subagent 실행 옵션과 parent session에 의존 | Teammate가 자동으로 Worktree에 격리되지 않으므로 파일 소유권 분리가 필요 |
| 중간 조정 | `terminal send` 또는 사람이 직접 개입 | 이번 세션에서는 완료 보고 중심으로 사용 | Inter-agent messaging으로 중간 소통 가능 |

### 6.1 이번 handoff에서 폴링은 단점이기만 한 것이 아니었다

이번 세션에서 폴링 덕에 잡은 것들:

1. **커밋 prefix 가 레인마다 갈린 것** — 완료 후 통보였다면 4개가 다 어긋난 뒤 발견
2. **레인 B 의 자율 PR** — 정책을 조기에 교정
3. **레인 C 의 골드 토큰 변경** — 아래 6.2 참조

### 6.2 폴링이 결정적이었던 사례 — 레인 간 기준 전파

TNSP-1242(흰 배경)는 레인 C(데모1)와 레인 D(데모2)로 쪼개져 있었고, 완료 조건에 **"데모 1과 데모 2 양쪽에 같은 기준으로 적용된다"** 가 있었다.

레인 C 가 아직 PR 도 내지 않고 완료 보고도 하지 않은 시점에, 폴링으로 중간 커밋을 읽다가 발견한 것:

> 골드를 `text-gold-accent`(#D4AF37) → `text-gold`(#A9803A)로 바꿈
> 이유: #D4AF37 은 흰 배경 위 대비 **2.10:1** 로 라벨 구실을 못 함. #A9803A 는 **3.60:1**

지시는 "골드 라벨 유지" 였으므로 이는 지시 이탈이지만 근거가 타당했다. 이 값을 **1241 을 작업 중이던 레인 D 에 즉시 전파**했다.

결과 — 레인 D 가 1242 를 마쳤을 때 토큰이 완전히 일치했다.

```
bg-paper · text-ink · text-ink-2 · text-ink-3 · text-ink-4
border-line-soft · border-line · text-gold · ring-paper
```

**완료 보고만 기다렸다면 이 개입은 늦었을 수 있다.** 레인 D 는 이미 `text-gold-accent` 를 흰 배경에 얹은 뒤였을 것이고, 사후에 두 데모를 맞추는 재작업이 발생했을 수 있다. 다만 중간 동기화 수단이 폴링뿐인 것은 아니다. Orca orchestration의 message·heartbeat·ask와 Agent Teams의 inter-agent messaging을 사용하면 완료 전에도 기준을 전달할 수 있다.

### 6.3 정리

- **짧고 독립적인 조사** → Claude subagent가 편리했다 (통보 자동, 보고 회수)
- **지속되는 구현과 사용자 관찰** → Orca Worktree가 적합했다
- **완료 계약이 필요한 Orca worker** → CLI handoff가 아니라 supervised orchestration을 사용해야 한다
- **레인끼리 기준을 맞춰야 함** → polling, orchestration message 또는 Agent Teams messaging 중 작업 구조에 맞는 중간 조정 수단이 필요하다

---

## 7. 실제 조합 — Claude subagent와 Orca CLI handoff의 하이브리드

이 세션은 의도치 않게 하이브리드로 굴러갔고, 그 조합이 각자의 강점에 맞았다.

```
[조사·분석]  Claude subagent (Agent tool 7개, 병렬 fan-out)
     ↓  자동 통보 → 결과가 오케스트레이터 컨텍스트로
[배치 설계]  오케스트레이터 (충돌 매트릭스, base 브랜치 라우팅)
     ↓
[구현]       Orca 레인 4개 (긴 작업, dev 서버, 사용자 개입 가능)
     ↓  폴링 → 중간 기준 전파
[통합]       오케스트레이터 (PR 검토, 병합 순서, 브랜치 하향)
     ↓
[검수]       Orca 레인 (브라우저·기기 — 예정)
```

### 7.1 조사에 Claude subagent를 쓴 것이 옳았던 이유

7개 에이전트가 각각 다른 파일군을 읽고 구조화된 보고만 돌려줬다. 파일 덤프가 컨텍스트에 들어오지 않았고, 완료 통보가 자동이라 폴링이 없었다. 조사는 서로 독립적이었으므로 중간 개입이 필요 없었다.

**다만 한 번 실패했다.** 데모 1/2 가 브랜치로 갈린다는 사실을 모른 채 조사를 지시해서, 데모 2 담당 에이전트가 데모 1 브랜치를 읽고 "패널·자동 전환 코드가 없다" 는 결론을 냈다. `feat/demo2` 를 명시해 재조사(2개)를 돌려 교정했다.

**교훈: 서브에이전트는 오케스트레이터의 컨텍스트를 상속하지 않는다.** 조사 대상이 체크아웃되지 않은 브랜치라면 `git show <branch>:<path>` / `git grep <pattern> <branch>` 를 쓰라고 프롬프트에 명시해야 한다.

### 7.2 만약 구현에 Agent Teams를 사용했다면

- Shared task list와 inter-agent messaging으로 lead가 완료와 중간 결정을 관리하기 쉬웠을 것이다.
- 6.2의 기준도 teammate message로 완료 전에 전파할 수 있었을 것이다.
- Teammate가 자동으로 Worktree에 격리되지는 않으므로 서로 다른 파일 집합을 맡기거나 별도 격리 전략이 필요했을 것이다.
- Orca 앱에서 각 Worktree의 dev server와 화면을 사용자가 직접 보며 개입하는 현재 흐름은 별도로 설계해야 했을 것이다.

### 7.3 만약 조사까지 Orca 레인으로 했다면

- 조사 레인 7개를 만들고 각각 폴링해야 했을 것이다
- 보고를 회수하려면 파일로 쓰게 하고 그걸 읽어야 했을 것이다
- 조사는 짧고 독립적이라 이 오버헤드가 순손실이다

### 7.4 권장 조합

| 작업 성격 | 도구 |
| --- | --- |
| 병렬 조사, 코드 탐색, 리뷰 | Claude subagent 또는 Agent Teams |
| 짧고 독립적이며 텍스트로 판정되는 작업 | Claude subagent |
| 긴 구현을 완전히 넘기고 사람이 직접 관찰 | Orca CLI handoff 레인 |
| 긴 구현을 coordinator가 추적하고 결과를 이어받음 | Orca supervised orchestration |
| Teammate끼리 공유 Task와 message가 필요 | Claude Code Agent Teams |
| 레인 간 기준 동기화 | Orchestration message·ask 또는 Agent Teams messaging과 human checkpoint |

---

## 8. 다음에 쓸 체크리스트

**레인 생성 전**

- [ ] base 브랜치를 작업별로 확정했는가 (파일 충돌보다 브랜치 분기가 상위 축일 수 있음)
- [ ] 같은 파일·같은 state 를 건드리는 작업을 한 레인에 순차로 묶었는가
- [ ] 커밋 메시지 관례를 `git log` 로 실측했는가
- [ ] **어디서 멈출지**(커밋 / push / PR)를 프롬프트에 명시했는가
- [ ] 로컬 확인 단계가 필요한 작업인가 (UI 라면 예)
- [ ] dev 서버 포트를 레인마다 다르게 지정했는가
- [ ] `.env` 등 gitignore 된 설정 파일 전달 방법을 정했는가
- [ ] DB 를 공유한다면 스키마 변경 금지를 명시했는가

**레인 생성 직후**

- [ ] `--json` 응답의 `result.worktree.branch` 가 기대와 같은가
- [ ] 기존 worktree 점유 상태를 디렉토리 존재로 실측했는가

**운영 중**

- [ ] 지시 채널이 하나인가 (오케스트레이터 또는 사용자, 둘 다 아님)
- [ ] 레인 간 공유해야 할 결정이 생겼을 때 즉시 전파했는가
- [ ] 사용자가 앱에서 한 행동이 오케스트레이터에게 공유되는가

**통합 시**

- [ ] 각 PR 의 base 가 올바른가
- [ ] 레인 간 기준(색상·토큰·네이밍)이 일치하는가
- [ ] 공통 변경을 파생 브랜치로 하향할 계획이 있는가

---

## 9. 이 세션의 최종 상태 (2026-08-20 기준)

| 티켓 | 상태 |
| --- | --- |
| TNSP-1236 | develop 머지 완료 (PR #16). 데모 브랜치 하향 대기 |
| TNSP-1237 | develop 머지 완료 (PR #15). 데모 브랜치 하향 대기 |
| TNSP-1238 | 커밋 완료, PR #17 (base `feat/demo1`, draft), CI 통과 |
| TNSP-1239~1241 | 커밋 완료, PR 미생성 (사용자 로컬 확인 대기) |
| TNSP-1242 | 양쪽 커밋 완료, 토큰 일치 확인됨 |
| TNSP-1243 | 미착수 (병합 후 진행) |

부수적으로 확인된 저장소 사실 — **`npm run lint` 실행 불가**. eslint 가 devDependencies 에도 설정 파일로도 없어 `next lint` 가 대화형 마법사를 띄운다. 두 레인 모두 독립적으로 이 사실을 발견하고 `npm run build` 로 대체했다.
