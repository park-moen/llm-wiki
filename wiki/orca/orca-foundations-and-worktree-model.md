# Orca 핵심 모델과 첫 멀티에이전트 세션

> Sources: Stably, Inc., Unknown
> Raw: [What is Orca?](../../raw/orca/what-is-orca.md); [Install](../../raw/orca/install.md); [Your first 3-agent session](../../raw/orca/first-session.md); [Worktrees](../../raw/orca/model-worktrees.md); [Tabs, panes & split layouts](../../raw/orca/model-tabs-panes-splits.md); [Agents & sessions](../../raw/orca/model-agents-sessions.md); [Session restore](../../raw/orca/model-session-restore.md); [Quick Open & Jump Palette](../../raw/orca/model-quick-open.md); [Terminal](../../raw/orca/terminal.md); [Race three agents on the same task](../../raw/orca/recipes-parallel-agents.md); [Jump between 10 worktrees](../../raw/orca/recipes-jump-worktrees.md)
> Updated: 2026-08-12

## Overview

Orca는 모델이나 Git 대체물이 아니라, 여러 AI 코딩 에이전트를 실제 `git worktree` 단위로 격리해 동시에 운용하는 데스크탑 IDE다. 핵심 루프는 저장소 추가 → 작업별 worktree 생성 → 에이전트 실행 → 필요하면 여러 접근을 병렬 비교 → diff 검토 → commit·push·review 생성이다. Orca를 깊게 이해하려면 화면 기능보다 먼저 **작업 하나가 독립 checkout, branch, terminal, browser, review context를 가진다**는 worktree-native 모델을 익혀야 한다.

## 제품의 정체성과 적합한 사용자

Orca는 Claude Code, Codex, Cursor CLI, OpenCode 등 이미 사용하는 CLI 에이전트를 한곳에서 실행한다. 에이전트별 구독과 인증은 사용자가 가져오며, 로컬·SSH·자체 서버·사용자 소유 cloud 환경에서 실행한다. 따라서 다음 상황에 특히 맞는다.

- 같은 문제에 여러 에이전트나 여러 접근법을 동시에 투입하고 결과를 비교한다.
- AI가 만든 변경을 diff 단위로 진지하게 검토하고 수정 루프를 돌린다.
- stash와 branch 전환을 반복하지 않고 여러 작업을 열어 둔다.
- 에이전트가 laptop 밖에서 계속 실행되더라도 같은 IDE 경험을 유지한다.

반대로 no-code 도구, 모델 제공 서비스, managed VPS 상품을 기대하면 제품의 역할을 잘못 잡은 것이다. Orca는 코드·Git·review를 직접 다루는 개발자에게 AI를 레버리지로 제공한다.

## Worktree-native 작업 모델

저장소에는 새 작업이 기본적으로 출발하는 **base ref**가 있고, 각 worktree에는 실제 분기 기준인 **start-from ref**가 있다. worktree마다 독립 branch와 파일 트리, 에이전트 terminal, editor·browser tab이 생긴다. 이 격리가 여러 에이전트가 서로의 파일을 덮어쓰지 않게 하는 핵심 안전장치다.

한 작업의 표준 수명주기는 다음과 같다.

1. 작업 이름, start-from ref, 실행 host, agent, 연결할 issue 또는 review를 정해 worktree를 만든다.
2. 해당 worktree에 귀속된 terminal, editor, browser에서 구현한다.
3. start-from ref 기준 diff를 보고 AI 주석과 attribution을 이용해 검토한다.
4. stage, commit, push, hosted review 생성을 진행한다.
5. 끝난 worktree를 archive하거나 branch와 함께 삭제한다.

생성 중 `git fetch`와 `git worktree add`는 background에서 실행되므로 다른 작업을 계속할 수 있다. 생성 상태, 취소, 실패 후 재시도는 해당 worktree tab에서 확인한다.

### Start-from ref와 stacked work

대부분은 base ref에서 시작하지만 다른 local branch, 특정 commit, remote branch에서도 시작할 수 있다. 이미 review 중인 branch 위에 후속 작업을 쌓을 때 특히 유용하다. Orca는 workspace 이름이나 연결된 작업 항목으로 branch 이름을 만들며, 허용되는 생성 경로에서는 Advanced 영역에서 명시적인 branch 이름을 줄 수 있다.

### Gitignored 데이터의 세 가지 처리법

새 worktree는 깨끗한 checkout이므로 dependency, cache, local secret 같은 gitignored 경로가 자동으로 따라오지 않는다. 용도에 따라 다음 방식을 구분한다.

- **Worktree Shared Paths**: 사용자·저장소 설정에서 지정하고 primary checkout의 경로를 새 worktree에 materialize한다.
- `orca.yaml`의 `worktree.sharedDirectories`: 저장소가 공유 정책을 버전 관리한다. 크고 재생성 가능한 gitignored directory에 적합하며 worktree 간에 공유한다.
- `.worktreeinclude`: gitignored file 또는 directory를 새 worktree로 복사한다. `.env`처럼 worktree가 자기 사본을 가져야 할 때 적합하다. literal path만 지원하며 tracked·missing·non-gitignored 경로는 대상이 아니다.

공유와 복사를 섞을 때는 이미 공유된 경로를 다시 복사하지 않는다는 점이 중요하다. dependency tree에는 공유, 변경 가능성이 있는 local config에는 복사를 우선하는 식으로 정책을 명시하면 새 worktree 준비 비용과 오염 위험을 함께 낮출 수 있다.

## 첫 멀티에이전트 세션

공식 문서가 제시하는 입문 흐름은 Orca의 전체 사용법을 축약한다.

1. **Add Repo**로 local checkout을 등록하고 base ref를 확인한다.
2. 저장소 옆 **+**에서 task 이름과 start-from ref를 정해 worktree를 만든다.
3. agent combobox에서 Claude Code, Codex, Cursor CLI 등 원하는 CLI를 실행한다.
4. 같은 prompt를 별도 worktree의 여러 agent에게 주어 서로 다른 branch와 diff를 만든다.
5. worktree tab을 pane 가장자리로 끌어 split하고 진행 상태를 함께 본다.
6. 결과를 diff에서 비교하고 가장 좋은 접근에 review note를 돌려보낸 뒤 commit·push한다.

여기서 병렬화의 단위는 같은 checkout의 여러 process가 아니라 독립 worktree의 독립 branch다. 승자를 고른 뒤 나머지는 삭제할 수 있으므로 실험 비용이 낮고, 선택한 결과만 review 흐름으로 승격시킨다.

## Tabs, panes, splits

Worktree 안의 terminal, file, diff, browser는 모두 tab이다. Tab을 pane 가장자리로 drag하면 오른쪽 또는 아래로 split하며 nested layout도 만들 수 있다. 여러 worktree의 tab을 함께 배치하는 tab group도 가능해, 한 화면에서 구현 agent, test terminal, browser, diff를 역할별로 고정할 수 있다.

Pinned boundary는 tab 이동이나 열기 동작이 의도하지 않은 pane을 침범하지 않게 한다. 병렬 에이전트 비교에서는 agent별 terminal을 나란히 두고, 선택한 worktree에서는 diff와 browser를 별도 pane에 두는 구성이 이해하기 쉽다.

## Agent 상태와 attention 관리

Orca는 지원 agent가 보내는 terminal title signal과 hook을 이용해 working, waiting for input, done, blocked, idle 상태를 표시한다. Plain shell처럼 인식되지 않는 process에는 표시가 없다. 상태 표시가 필요하면 binary를 직접 입력하기보다 agent combobox에서 실행해야 한다.

실험적 Agent Dashboard는 여러 worktree의 agent를 Needs You, Working, Done, Idle 관점으로 모은다. Agent Map은 project·worktree·agent 및 orchestration lineage를 topology로 보여준다. 핵심은 모든 terminal을 계속 바라보는 것이 아니라, attention이 필요한 session만 골라내는 것이다.

Orca는 지원 agent를 기본적으로 높은 자율 권한 flag와 함께 실행한다. 공식 설계 의도는 disposable worktree를 sandbox처럼 쓰고, 결과를 diff에서 검토한 뒤 채택하거나 버리는 것이다. 이 권한 모델이 조직 정책과 맞지 않으면 Settings의 Agent Permissions 또는 agent별 launch arguments를 먼저 Manual 쪽으로 조정해야 한다.

## Session restore와 daemon

Orca app 창을 닫아도 background daemon이 살아 있으면 agent PTY는 계속 실행된다. 다시 열면 worktree, tab·split layout, scrollback, focus와 실행 중 process에 다시 연결한다. App crash나 updater relaunch도 daemon이 살아 있으면 같은 방식으로 복구한다.

Host reboot처럼 daemon 자체가 종료되면 layout과 마지막 scrollback은 돌아오지만 agent process는 사라진다. 즉 UI 재시작 내구성과 machine 재시작 내구성은 다르다. 완전히 새 화면으로 시작하려면 종료 전에 worktree를 명시적으로 닫아야 하며, 별도의 clean-session launch mode를 전제로 하면 안 된다.

## Navigation이 병목이 되지 않게 하는 법

- `Cmd-P`: 현재 worktree의 file을 빠르게 연다. tracked match 뒤에 gitignored file도 검색한다.
- Tab strip의 `+`: 열린 tab, file, URL, agent를 한 입력창에서 찾는다.
- `Cmd-J`: 모든 worktree와 tab을 가로질러 이동한다. Host·project filter, recent session, review metadata 검색을 지원하며 sidebar filter로 숨겨진 worktree도 query가 있으면 찾는다.
- Worktree 결과에서 `Shift-Enter`: 현재 pane을 바꾸지 않고 새 split으로 연다.

Worktree 수가 늘면 sidebar를 눈으로 스캔하지 말고 이름 규칙, project grouping, pin, sleep filter, Jump Palette를 결합해야 한다. 작업 이름은 branch와 검색 키가 되므로 짧고 의미 있게 짓는 편이 좋다.

## Terminal을 agent 작업면으로 쓰기

Terminal은 xterm.js 기반이며 다른 tab과 동일하게 split할 수 있다. Agent tab은 identity와 live state를 표시하며, 종료 뒤 Restart control로 같은 working directory에서 다시 시작한다. Search, bounded context copy, OSC 52 clipboard, theme import, native modifier-aware key 입력을 지원한다.

자주 쓰는 개발 명령이나 launch prompt는 **Quick Commands**로 저장할 수 있다. Global scope와 Project scope를 구분하고, tab bar에서는 새 terminal을 열어 실행하며 terminal context menu에서는 현재 terminal에 삽입할 수 있다. Remote runtime에서도 command의 저장 위치와 실제 실행 위치가 다를 수 있으므로 “Saved on”과 현재 workspace host를 함께 확인해야 한다.

## 정착을 위한 권장 순서

1. Stable build를 설치하고 agent CLI 인증이 Orca가 실행되는 host에 존재하는지 확인한다.
2. 하나의 repo에서 base ref와 worktree naming을 먼저 안정화한다.
3. 작은 task를 서로 다른 worktree의 여러 agent에게 맡겨 diff 비교와 삭제까지 끝내 본다.
4. `.worktreeinclude`, shared directories, setup hook으로 새 worktree 준비를 재현 가능하게 만든다.
5. Split layout보다 먼저 상태 표시, notification, Jump Palette로 attention 흐름을 익힌다.
6. 기본 permission mode가 자신의 위험 허용도와 맞는지 검토한 뒤 agent 수를 늘린다.

## See Also

- [Orca 에이전트와 세션 운영](orca-agent-runtime-and-session-management.md)
- [Orca 리뷰·편집·브라우저 작업 흐름](orca-review-editing-and-browser-workflow.md)
- [Orca 원격 실행과 모바일 운영](orca-remote-and-mobile-runtime.md)
- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
