# Orca CLI·자동화·오케스트레이션

> Sources: Stably, Inc., Unknown
> Raw: [Orca CLI overview](../../raw/orca/cli-overview.md); [Orca CLI reference](../../raw/orca/cli-reference.md); [Orchestration](../../raw/orca/cli-orchestration.md); [Scheduled automations](../../raw/orca/cli-automations.md); [Computer use](../../raw/orca/cli-computer-use.md); [Worktree checkpoints](../../raw/orca/cli-worktree-checkpoints.md); [Orca skills registry & MCP](../../raw/orca/cli-skills.md)
> Updated: 2026-08-12

## Overview

`orca` CLI는 실행 중인 Orca runtime을 shell·script·agent에서 제어한다. Worktree, terminal, file, built-in browser, desktop app, emulator, issue, artifact, automation을 같은 runtime context에 연결한다. 단순 명령 전달과 추적 가능한 multi-agent orchestration을 구분하고, selector·JSON·read-before-write 원칙을 지키는 것이 안정적인 자동화의 핵심이다.

## CLI의 역할과 연결 확인

Desktop app의 Settings에서 CLI를 등록한 뒤 다음으로 executable과 runtime 연결을 확인한다.

```bash
command -v orca
orca status --json
```

CLI는 standalone Git wrapper가 아니라 Orca runtime과 통신한다. Headless에서는 `orca serve`가 runtime을 제공한다. Agent나 script가 output을 해석할 때는 human text 대신 `--json`을 사용한다.

## Selector와 명시적 대상 지정

많은 command는 긴 ID 대신 `active`, `current`, `id:...`, `path:...`, `branch:...`, `issue:...` selector를 받는다. 현재 shell이 Orca-managed worktree 안이라면 active/current가 편하지만, scheduler·remote script·coordinator처럼 실행 위치가 target과 다르면 explicit selector가 안전하다.

Remote runtime에서는 local shell path가 server에 없을 수 있으므로 server-side absolute path 또는 runtime이 돌려준 ID를 사용한다. UI label을 parsing하거나 worktree 이름이 유일하다고 가정하지 않는다.

## Worktree와 terminal automation

CLI는 repo 등록과 base ref 설정, worktree 생성·조회·삭제, agent launch와 initial prompt, setup hook 정책을 제어한다. Managed worktree 안에서 child worktree를 만들면 parent relationship을 추론할 수 있고 `--parent-worktree` 또는 `--no-parent`로 의도를 명시할 수 있다.

Terminal에는 list, show, read, send, wait, create, split, rename, switch, close가 있다. 안전한 agent driver는 다음 순서를 따른다.

1. `terminal list`로 runtime-scoped handle을 얻는다.
2. `terminal read`로 현재 prompt와 state를 본다.
3. 다음 input이 명확할 때만 `terminal send`한다.
4. `terminal wait --for tui-idle`로 settlement를 기다린다.
5. 긴 output은 cursor를 저장해 증분으로 읽는다.

Orca restart 뒤 handle이 stale할 수 있으므로 실패 시 list를 다시 실행해 reacquire한다. Multi-agent ownership과 completion tracking이 필요하면 terminal send를 반복하지 말고 orchestration으로 올린다.

## File과 browser 제어

File command는 selected worktree에서 source file, staged diff, changed file set을 Orca tab으로 연다. Browser command는 Chrome·Safari나 Orca desktop UI가 아니라 worktree의 embedded browser를 제어한다.

Browser automation의 기본은 **snapshot → act → snapshot**이다. Snapshot이 돌려준 element ref로 click·fill하고 navigation, tab switch, DOM change, stale-ref error 뒤에는 다시 snapshot한다. Console·network·screenshot·PDF·device emulation도 같은 worktree와 browser profile context를 사용한다.

이 패턴은 coordinate click보다 semantic element ref를 우선하고, action 뒤 state를 검증하게 한다. Logged-in identity가 중요하면 먼저 browser profile을 선택하고 cookie·storage isolation을 확인한다.

## Desktop computer use

`orca computer`는 built-in browser 밖의 native app을 accessibility tree와 screenshot으로 제어한다. Platform permission이 필요하며 macOS에서는 Accessibility와 Screen Recording을 확인한다.

Workflow는 browser와 비슷하다.

1. Running app과 window를 list한다.
2. `get-app-state`로 최신 accessibility tree와 screenshot을 얻는다.
3. Bundle ID, stable window ID, element index로 semantic action을 수행한다.
4. UI가 바뀌면 state를 다시 읽는다.

Element index는 최신 snapshot에만 유효하고 sparse할 수 있다. `elementCount`에서 index를 추측하면 안 된다. `click`, `set-value`, secondary action을 먼저 쓰고 accessibility targeting이 실패할 때만 coordinate fallback을 쓴다.

Secret은 command-line argument나 shell history에 넣지 않고 stdin flag로 전달한다. Screenshot이 필요 없으면 끄고, minimized window를 캡처해야 할 때만 restore option을 사용한다.

## Structured orchestration

Orchestration은 다음 객체로 multi-agent work를 추적한다.

- **Run**: namespace와 coordinator inbox
- **Task**: dependency와 acceptance 기준이 있는 작업 단위
- **Dispatch**: task를 worker에게 할당한 기록
- **Worker**: terminal 또는 worktree에서 실행되는 supervised agent
- **Message**: coordinator와 worker의 질문·응답·진행 보고
- **Decision gate**: human 또는 coordinator 판단이 필요한 경계

가벼운 prompt는 `orca terminal send`, completion contract가 필요한 worker는 orchestration dispatch, dependency graph와 coordinator loop가 필요한 큰 작업은 Run·Task·Worker를 사용한다. Ownership 전체를 다른 worktree에 넘기는 경우와 coordinator가 결과를 기다리는 supervision을 혼동하지 않는다.

### Supervised loop의 핵심

Coordinator는 task를 만들고 worker를 시작한 뒤 event를 기다린다. Worker는 임의의 terminal output만 남기는 것이 아니라 task ID에 대해 `worker_done` 또는 질문·blocker를 보고해야 한다. Coordinator는 결과와 acceptance evidence를 검토한 뒤 완료를 받아들이거나 follow-up을 보낸다.

Task dependency는 DAG로 표현해 선행 결과가 필요한 worker를 너무 일찍 시작하지 않는다. Decision gate가 열렸을 때는 user 판단 없이 임의로 통과시키지 않는다. Retry는 원래 placement를 자동 상속한다고 가정하지 말고 target을 명시한다.

Runtime-global orchestration reset은 다른 coordinator의 state도 건드릴 수 있으므로 active run이 없다는 확신 없이 실행하지 않는다. Command surface가 app version에 따라 변하므로 mutation 전 version-matched full guide를 읽는다.

## Scheduled automations

Automation은 schedule에 맞춰 prompt를 provider와 target repo/workspace에 실행한다. 안전한 첫 구성은 `--disabled`로 만든 뒤 inspect와 manual run을 거쳐 enable하는 것이다.

Trigger는 preset, cron, RRULE을 받을 수 있고 timezone을 명시할 수 있다. Target은 repo, existing workspace, project·host setup 중 작업 특성에 맞게 고른다. Cheap shell **precheck**가 non-zero면 run을 skipped로 기록해 불필요한 agent session을 막는다.

Existing workspace automation은 `--reuse-session`으로 이전 live terminal을 이어갈 수 있다. 지속 context가 오염을 만들 수 있는 작업은 fresh session을 유지한다. Missed-run grace는 host가 잠시 offline이었다 돌아왔을 때 오래된 schedule을 실행할 범위를 정한다.

권장 rollout은 다음과 같다.

1. Disabled automation 생성
2. `list`와 `show`로 schedule, prompt, provider, host 확인
3. Manual `run`으로 실제 workspace와 output 확인
4. Run history에서 실패·skip reason 검토
5. Enable 후 처음 몇 번의 schedule을 관찰

## Worktree checkpoint

각 worktree의 free-text comment는 UI와 CLI가 공유하는 현재 상태 요약이다. Agent는 의미 있는 단계가 끝났을 때 comment와 optional workspace status를 갱신한다. Blocker, 검증 완료, hypothesis 기각, review 전환 같은 순간이 적합하다.

기존 comment에 user goal이나 constraint가 있을 수 있으므로 `worktree current --json`으로 먼저 읽고 유효한 맥락을 보존한다. 첫 줄에는 방금 끝난 action, 위치, 다음 단계 또는 blocker를 넣어 sidebar만 봐도 판단할 수 있게 한다.

## Skills registry와 version drift 방지

Orca skill package는 짧은 discovery stub와 실행 중 CLI가 제공하는 version-matched live guide를 결합한다. Public stub는 언제 Orca 기능을 써야 하는지 알려 주고, 실제 flag와 command는 `orca skills get <topic>`에서 가져온다. Agent가 기억에 의존해 flag를 발명하지 않게 하는 구조다.

대표 skill은 `orca-cli`, `orchestration`, `computer-use`, `orca-linear`, iOS·Android emulator, per-workspace environment다. Desktop updater 또는 headless `orca skills install|update`로 관리한다. Global과 local placement, detected agent target을 구분하고 dry-run으로 실제 install command를 확인할 수 있다.

Orchestration state를 바꾸기 전에는 full guide, issue를 바꾸기 전에는 해당 integration guide를 load한다. MCP server는 Settings의 Integrations에 등록하고, 이를 지원하는 agent CLI 안에서 tool로 노출한다.

## 자동화 안전 원칙

- Machine-readable output에는 항상 `--json`을 우선한다.
- `active`가 명확하지 않은 background job에는 explicit selector를 쓴다.
- Terminal과 UI는 write 전에 최신 state를 읽는다.
- Browser·computer element ref는 화면 변화 뒤 재사용하지 않는다.
- Secret은 stdin과 OS credential store를 사용한다.
- Scheduled task는 disabled → inspect → manual run → enable 순서로 연다.
- Multi-agent 작업은 terminal prompt fan-out이 아니라 task ownership과 completion contract로 추적한다.
- Runtime-global reset, worktree removal, artifact publish 같은 넓은 mutation은 scope를 재확인한다.
- CLI flag는 설치된 Orca의 live skill guide에서 확인한다.

## See Also

- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
- [Orca 에이전트와 세션 운영](orca-agent-runtime-and-session-management.md)
- [Orca 원격 실행과 모바일 운영](orca-remote-and-mobile-runtime.md)
- [Orca 설정·개인정보·문제 해결](orca-operations-settings-privacy-troubleshooting.md)
