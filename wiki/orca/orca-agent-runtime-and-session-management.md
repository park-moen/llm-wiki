# Orca 에이전트와 세션 운영

> Sources: Stably, Inc., Unknown
> Raw: [Supported agents](../../raw/orca/agents-supported.md); [Claude Code in Orca](../../raw/orca/agents-claude-code.md); [How to use GLM-5.2 in Orca ADE](../../raw/orca/agents-glm-agent.md); [Codex in Orca](../../raw/orca/agents-codex.md); [Cursor CLI in Orca](../../raw/orca/agents-cursor-cli.md); [Hot-swap Codex accounts](../../raw/orca/agents-codex-hot-swap.md); [Chat UI](../../raw/orca/agents-native-chat.md); [Agent session history](../../raw/orca/agents-session-history.md); [Agent hibernation](../../raw/orca/agents-hibernation.md); [Usage & rate-limit tracking](../../raw/orca/agents-usage-tracking.md); [Agent hooks & memory](../../raw/orca/agents-hooks-memory.md); [Agents feed](../../raw/orca/activity.md); [Notifications & Inbox](../../raw/orca/notifications.md)
> Updated: 2026-08-12

## Overview

Orca의 agent integration은 CLI process를 terminal에서 실행하는 공통 층과, 상태·usage·account·resume·hook을 인식하는 심층 층으로 나뉜다. 어떤 CLI든 terminal에서 사용할 수 있지만 combobox에 내장된 agent는 설치 안내와 launch preset을 가지며, Claude Code와 Codex 같은 일부 agent는 account switching과 usage tracking까지 연결된다. 안정적인 운영의 핵심은 agent의 실제 session store와 credential home은 각 CLI가 소유하고 Orca는 이를 조정한다는 경계를 이해하는 것이다.

## 공통 실행 모델과 권한

Agent combobox는 선택한 worktree를 `cwd`로 CLI process를 시작한다. 내장 agent들은 setup·launch preset을 제공하고, 지원 범위에 따라 OSC state, hook, usage, account 전환, restart·resume 기능이 추가된다. 직접 binary를 실행할 수도 있지만 상태 감지와 lifecycle integration이 약해질 수 있다.

새 launch에는 기본적으로 각 agent의 permission-bypass flag가 들어간다. 예를 들어 Claude는 `--dangerously-skip-permissions`, Codex는 `--dangerously-bypass-approvals-and-sandbox`를 사용한다. Worktree 자체를 격리 경계로 삼고 마지막 diff review에서 채택 여부를 결정하는 설계다.

위험 허용도가 다르면 **Settings → Agents → Agent Permissions**에서 전역 기본을 Yolo 또는 Manual로 바꾼다. Agent별 custom arguments나 environment가 이미 있으면 전역 전환이 그 override를 덮지 않는다. 따라서 조직 보안 정책을 적용할 때는 전역 mode만 믿지 말고 agent별 override도 함께 점검해야 한다.

## Claude Code, Codex, Cursor의 차이

### Claude Code

Orca는 host의 `~/.claude` 인증과 사용 상태를 읽고, worktree에서 Claude Code를 status hook과 함께 실행한다. Background subagent와 Agent Teams teammate는 parent 아래 child row로 나타날 수 있다. Repository의 `.claude/` 설정과 `CLAUDE.md`는 원래 Claude 규칙대로 동작한다.

### Codex

Orca는 `~/.codex`를 읽어 account와 session을 연결한다. System default는 외부 terminal의 bare `codex`와 같은 실제 home을 사용하고, Orca-managed extra account는 별도 home으로 분리한다. Running process는 시작할 때의 account home을 유지한다. Codex가 Task subagent를 만들면 child row로 보일 수 있지만 별도 pane을 소유하는 것은 아니다.

Windows에서는 host 설치와 WSL 설치를 구분한다. WSL account는 distro 내부의 격리 home을 쓰고 host에서 접근 가능한 경로로 mapping되며, launch·hot-swap·usage read도 선택한 distro를 통해 수행된다.

### Cursor CLI

Cursor CLI는 combobox launch, OSC state, exit 후 restart를 지원한다. Model 선택은 Cursor 자체 설정이 소유하며 Orca가 override하지 않는다. Runtime orchestration은 Orca가, model과 provider 설정은 agent harness가 맡는 제품 경계를 잘 보여준다.

### 다른 model을 쓰는 방식

GLM 계열처럼 Orca가 직접 제공하지 않는 model은 Claude Code, OpenCode, Cline, Kilo Code, Droid, OpenClaw 같은 harness에 provider와 model을 설정한 뒤 Orca가 그 harness를 worktree에서 실행하게 한다. Model access, API key, context 설정은 harness 또는 provider가 담당하고 Orca는 worktree·terminal·browser·review·session 관리만 담당한다.

## Multi-account와 hot-swap

Codex와 Claude account switcher는 새 session이 사용할 credential home을 빠르게 전환한다. 중요한 규칙은 다음과 같다.

- 새 launch는 현재 active account를 따른다.
- 이미 실행 중인 process는 시작할 때의 account를 유지한다.
- Account를 바꾼 뒤 기존 session에 새 account를 적용하려면 restart가 필요하다.
- Usage readout은 active account를 기준으로 바뀐다.
- Managed account는 System default credential을 덮어쓰지 않는다.

Managed Codex account는 실제 `~/.codex/config.toml` 설정을 runtime home으로 mirror한다. Source config가 비어 있거나 읽을 수 없으면 마지막으로 성공한 설정을 유지하므로, edit가 무시된 것처럼 보일 때는 account UI의 warning과 source path를 먼저 본다.

## Terminal UI와 Chat UI

Raw terminal이 session의 source of truth다. 실험적 Chat UI는 같은 PTY 위에 structured transcript와 composer를 얹는다. 지원 agent에서는 slash command·skill discovery, model/options control, attachment, structured question card를 제공하지만 transcript fidelity와 terminal parity는 계속 조정되는 영역이다.

따라서 일상적인 prompt와 응답 검토에는 Chat UI가 편하고, agent 고유 TUI 동작·OSC signal·세밀한 command를 확인할 때는 terminal로 전환하는 것이 안전하다. Chat UI와 terminal은 서로 다른 session이 아니라 같은 pane의 두 view다.

**Continue in New Session**은 기존 transcript에서 bounded handoff context를 만들어 새 agent session을 시작하고 원본 session은 그대로 둔다. 이는 provider의 native resume와 다르다. 긴 session을 정리해 다른 agent나 새 context로 넘길 때 사용한다.

## Session history와 resume

Agent Session History는 각 CLI가 disk에 남긴 transcript store를 scan한다. Workspace, Project, All scope로 좁히고 agent, sort, group, empty-session filter를 조정할 수 있다. Session detail에는 working directory, branch, model, message 정보와 첫 prompt·최근 turn이 나타난다.

Resume는 새 terminal을 열고 agent 고유 command를 실행한다. Codex의 `codex resume`, Claude의 `claude --resume`처럼 provider별 방식이 다르며, Orca는 원래 `cwd`와 session identifier를 연결한다. Raw log 열기, log path·session ID·resume command 복사도 가능하다.

Remote workspace를 보고 있더라도 history scan과 resume의 실행 host는 구분해야 한다. 공식 문서 기준으로 panel의 resume action은 local workspace에서 실행되며, remote에서 재개해야 한다면 command를 복사해 해당 host에서 실행한다. Session transcript가 삭제되거나 provider가 identifier를 더 이상 받아들이지 않으면 새 prompt로 열릴 수 있다.

## Hibernation과 resource 관리

실험적 hibernation은 완료된 background agent terminal을 멈추고 worktree를 다시 열 때 resume한다. Waiting for input, active foreground, recent input/output, mobile control 중, unsettled orchestration dispatch, live subagent가 있는 terminal은 잠들지 않는다. 여러 agent pane이 있는 worktree는 부분적으로만 멈추지 않고 함께 처리한다.

기본 idle window는 공식 문서에서 **30 minutes**이며 Settings에서 조정할 수 있다. Resume 가능한 agent만 대상으로 하고, session을 재개할 수 없는 CLI는 계속 실행한다. 짧은 window는 memory를 절약하지만 workspace 전환 때 resume 비용이 늘고, 긴 window는 즉시성을 유지하지만 idle PTY가 쌓인다.

Manual sleep과 automatic hibernation은 terminal process를 정리한다는 점에서는 비슷하지만 session record를 삭제하는 기능이 아니다. Reopen 시 저장된 launch command, arguments, private environment와 resume 정보로 복구를 시도한다.

## Usage와 rate limit 해석

Orca는 Claude Code, Codex 및 일부 agent가 local disk에 기록한 usage state를 읽어 status bar와 roster에 보여준다. 별도 provider API call을 하는 것이 아니므로 agent가 local state를 갱신한 시점만큼만 최신이다. Active account와 다른 configured account의 usage를 구분해 본다.

Warning과 reset window는 작업 배분에 유용하지만 provider billing console을 대체하지 않는다. Stats의 estimated cost 중 inferred pricing으로 표시된 값은 Orca local price table 기반 추정치다. 실제 spend 판단은 provider console을 우선한다.

## Hooks, memory, status

- Repository의 `.claude/`, `.codex/` config와 hook은 해당 agent가 원래 하던 방식으로 실행된다.
- `CLAUDE.md`, `AGENTS.md`는 agent 소유 memory file이며 Orca는 일반 file처럼 보여 줄 뿐 내용을 대체하지 않는다.
- Worktree 생성 뒤 dependency install이나 local setup이 필요하면 **Settings → Repository → Hooks**에 setup command를 둔다.
- Orca-managed status hook은 working·waiting·done signal을 UI에 보낸다. Settings 또는 `orca agent hooks status|on|off --json`으로 제어한다.
- Hook endpoint는 disk에서 다시 읽히므로 app restart 전부터 살아 있던 process도 새 runtime endpoint로 report할 수 있다.

## Notifications와 Agents feed

Agent가 working에서 idle로 바뀌면 system notification, sound, worktree chip을 통해 완료를 알릴 수 있다. Header bell은 모든 worktree의 unread event를 모으고 선택하면 해당 pane으로 이동한다. Category와 sound는 Settings에서 조정한다.

Agents feed는 completion, blocking question, unread state, worktree creation, 최근 응답 preview를 시간순으로 모으는 catch-up surface다. Running agent는 위쪽에 유지되어 진행 중인 작업을 구분할 수 있다. Notification이 즉각적인 interruption이라면 feed는 자리를 비운 뒤 전체 상태를 훑는 inbox다.

## 실전 운영 원칙

1. Agent binary를 직접 입력하기보다 combobox에서 시작해 상태와 restart integration을 보존한다.
2. Permission mode를 조직 정책에 맞춘 뒤 worktree 격리만으로 모든 위험이 사라진다고 가정하지 않는다.
3. Multi-account 전환은 running session 이동이 아니라 다음 launch의 기본 account 변경으로 이해한다.
4. Chat UI가 애매하게 보이면 terminal을 source of truth로 확인한다.
5. 긴 작업은 worktree comment, notification, feed로 attention을 관리하고 무작정 모든 terminal을 열어 두지 않는다.
6. Session history와 hibernation을 쓰기 전에 사용하는 agent의 native resume 지원 여부를 확인한다.

## See Also

- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
- [Orca 설정·개인정보·문제 해결](orca-operations-settings-privacy-troubleshooting.md)
