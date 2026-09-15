# Orca 설정·개인정보·문제 해결

> Sources: Stably, Inc., Unknown
> Raw: [Settings reference](../../raw/orca/settings.md); [Privacy & Telemetry](../../raw/orca/telemetry.md); [Troubleshooting & FAQ](../../raw/orca/troubleshooting.md); [Troubleshooting GitHub errors](../../raw/orca/github-errors.md)
> Updated: 2026-08-12

## Overview

Orca 운영 문제는 대개 UI 자체보다 실행 host의 CLI 설치·인증·PATH, Git ref·worktree 상태, remote relay prerequisite, provider rate limit에서 발생한다. Settings를 runtime ownership, permission, public exposure, experimental feature의 제어면으로 이해하면 진단이 빨라진다.

## Settings의 구조와 우선순위

Settings는 검색할 수 있으며 기능 문서에서 관련 pane으로 직접 연결된다. 처음 정착할 때는 다음 순서로 검토하는 편이 효율적이다.

1. **General**: CLI registration, update channel, external editor, UI scale
2. **Git·Repository**: base ref, commit signing, create hook, shared path, Source Control AI recipe
3. **Agents**: detected CLI, permission mode, account, status hook, skill freshness
4. **Terminal·Browser**: shell, clipboard, theme, profile, link routing, devtools
5. **Integrations**: GitHub, Linear, Jira, MCP, provider-specific usage credential
6. **Notifications·Shortcuts**: attention flow와 review action
7. **SSH·Remote Orca Servers**: execution host와 runtime routing
8. **Artifacts·Privacy**: public publishing과 telemetry
9. **Experimental·Plugins**: 안정화되지 않은 기능과 third-party code

Repository setting은 global setting과 분리해 본다. Base ref, setup hook, shared path, AI action recipe가 repo마다 다를 수 있다. Global default를 바꿔도 repository override가 남아 있으면 실제 동작이 달라진다.

## 업데이트 채널과 실험 기능

Stable update가 기본이며 Check for Updates의 modifier click으로 RC나 별도 build를 일회성으로 찾는다. Permanent RC opt-in을 전제로 하지 않는다. 문제가 생겨 older release를 설치하더라도 worktree data를 자동으로 downgrade하는 기능은 아니므로 version을 내릴 때 compatibility를 별도로 확인한다.

Chat UI, Agent Dashboard·Map, hibernation, Cloud VM, plugin system 같은 항목은 Experimental 또는 beta 성격을 가진다. 핵심 작업 흐름을 먼저 stable surface에서 익히고 한 번에 하나씩 켜야 원인 분리가 쉽다.

Plugin은 개별 동의 후 enable되며 worker가 local computer에서 실행된다. Marketplace source와 capability를 확인하고 third-party plugin을 untrusted software로 다룬다. SSH workspace action이 remote로 route되더라도 plugin worker 자체의 실행 위치는 local이라는 경계를 기억한다.

## Permission과 public exposure

### Agent permission

Global Yolo·Manual 선택은 custom override가 없는 agent에 적용된다. High-autonomy launch를 쓰더라도 secret, network credential, destructive Git action은 worktree isolation만으로 보호되지 않는다. Agent별 arguments와 environment를 함께 audit한다.

### OSC 52 clipboard

Terminal의 TUI clipboard write는 기본적으로 허용되어 local과 SSH TUI copy가 편리하다. 원격 terminal output이 OS clipboard를 쓰는 것을 원하지 않는 보안 환경에서는 이 setting을 끈다.

### Artifact publishing

Public artifact link publishing은 기본적으로 off이며 device-wide gate를 human이 켜야 한다. Gate를 끄더라도 이미 만든 public link가 자동 삭제되지는 않는다. Artifact list에서 기존 link를 audit하고 revoke해야 한다. CLI가 gate를 우회해 publishing permission을 부여할 수는 없다.

### Remote access

Remote Orca Server pairing link는 runtime 접근 credential이다. Private network에서 사용하고 grant를 관리한다. Agent account와 provider CLI는 client가 아니라 server host에 설치한다.

## Telemetry의 범위

Packaged build의 telemetry는 local random ID와 build·platform 정보, 제한된 lifecycle·feature event를 전송한다. 공식 문서상 file content, prompt, agent output, terminal output, repo·branch name, URL, path, commit message 같은 free-form content는 보내지 않는다. Raw error와 stack trace도 자동 전송하지 않으며 diagnostic bundle을 명시적으로 공유할 때만 incident detail이 전달된다.

Telemetry는 Settings의 Privacy toggle, `DO_NOT_TRACK=1`, `ORCA_TELEMETRY_DISABLED=1` 중 하나로 끌 수 있다. Environment variable은 해당 launch에 적용되고 제거하면 저장된 app preference가 다시 사용된다. Data destination은 PostHog Cloud의 United States region으로 문서화되어 있다.

Anonymous usage data와 public artifact, feedback attachment, diagnostic bundle은 서로 다른 channel이다. Telemetry를 꺼도 사용자가 직접 publish하거나 feedback에 첨부한 data까지 자동으로 사라지는 것은 아니다.

## 기본 진단 순서

문제가 생기면 다음 층을 위에서 아래로 확인한다.

1. **실행 위치**: Local, SSH, Remote Server 중 어느 host가 실제 process와 credential을 소유하는가?
2. **직접 CLI 실행**: Orca terminal에서 agent 또는 provider CLI를 직접 실행해 설치·auth·PATH 문제를 분리한다.
3. **Runtime state**: Tab의 Restart, CLI status, connection chip, worktree creation panel을 확인한다.
4. **Git state**: Start-from ref fetch 여부, duplicate worktree branch, external rebase·reset 뒤 refresh를 확인한다.
5. **Remote prerequisite**: Node, network, native build toolchain, SFTP capability, OpenSSH auth path를 확인한다.
6. **Provider quota·permission**: GitHub와 AI provider limit을 구분한다.
7. **Logs**: Help → Open Logs에서 local diagnostic evidence를 수집한다.

## 자주 발생하는 문제

### Agent가 시작되지 않음

같은 Orca terminal에서 CLI binary를 수동 실행한다. 여기서도 실패하면 agent 설치나 authentication 문제다. Settings에서 Orca가 보는 PATH와 detected agent를 확인하고 종료된 tab의 Restart를 시도한다.

### Diff가 오래되거나 잘못 보임

Diff toolbar를 refresh해 worktree를 다시 읽는다. External Git rebase·reset이 Orca refresh 사이에 들어갔는지 확인하고 compare base가 의도한 ref인지 본다.

### Worktree 생성 실패

Start-from ref가 fetch되지 않았거나 같은 branch가 이미 다른 worktree에 붙어 있을 수 있다. `git fetch origin`과 `git worktree list` 관점으로 상태를 확인하고 branch 이름 또는 기존 worktree를 정리한다.

### CLI command를 찾지 못함

General settings에서 bundled CLI를 register하고 shell PATH에 설치 위치가 포함되는지 확인한다. App과 shell이 서로 다른 environment를 볼 수 있으므로 일반 terminal에서만 command가 보이는 현상도 PATH 문제로 분류한다.

### SSH에서 file은 되지만 terminal이 안 됨

Remote relay의 native PTY module이 build되지 못했을 가능성이 있다. Remote Node, network, make, C++ compiler, `python3`를 확인하고 toolchain 설치 뒤 reconnect한다. File·Git·editor 연결 성공이 terminal readiness를 보장하지 않는다.

### Browser automation이 `browser_no_tab`을 반환

Selected worktree에 browser tab이 없다. UI에서 browser pane을 열거나 CLI로 tab을 만들고 URL을 연 뒤 snapshot한다. Active worktree selector가 의도한 target인지도 확인한다.

### Performance와 memory

열린 worktree마다 watcher와 session state가 있고 browser tab이 많은 split은 memory를 크게 쓴다. 사용하지 않는 browser와 worktree를 닫거나 supported agent에 hibernation을 적용한다. Resource Manager에서 CPU·memory·session과 daemon을 함께 본다.

## GitHub 오류를 별도 quota로 이해하기

Orca는 host의 `gh` CLI를 통해 GitHub와 통신한다. GitHub REST·GraphQL·Search budget은 Claude·Codex usage와 별개이며 같은 GitHub user의 Orca, terminal `gh`, agent, CI, extension이 quota를 공유한다.

PR panel error가 rate limit을 말하는데 Settings의 GitHub API Budget이 정상처럼 보일 수 있다. Budget UI가 읽는 rate-limit probe는 실제 REST request와 동작이 다를 수 있으므로 다음 우선순위를 따른다.

1. PR·Checks panel의 live request error
2. 같은 host에서 실행한 실제 `gh api user` 같은 REST call
3. Settings의 probe-based budget 표시

Rate limit이면 extra Orca instance와 polling automation을 줄이고 reset까지 기다린다. Orca는 primary bucket failure 뒤 잠시 같은 종류의 `gh` spawn을 막는 circuit breaker를 사용하고 last-known review state를 유지할 수 있다.

### Authentication과 access

`gh auth status`와 `gh api user`로 실제 identity를 확인한다. Shell profile의 stale `GITHUB_TOKEN`이나 `GH_TOKEN`이 keyring보다 우선되어 혼란을 만들 수 있다. Private organization은 SSO authorization과 scope가 필요할 수 있다. HTTP permission failure와 rate-limit response를 같은 403이라는 이유만으로 동일하게 처리하지 않는다.

Remote·SSH worktree에서는 GitHub login도 host별이다. Laptop에서 성공한 `gh`가 server에서 자동으로 성공하지 않는다. Git action이 실행되는 동일 host와 user context에서 진단해야 한다.

## 운영 baseline

- Stable channel과 최소한의 Experimental feature로 시작한다.
- Agent permission, artifact publishing, telemetry, remote pairing을 첫날에 명시적으로 결정한다.
- Repo별 base ref, setup hook, shared path, AI action override를 문서화한다.
- Provider usage와 GitHub API quota를 별도 dashboard로 해석한다.
- Remote 문제는 client UI보다 runtime host에서 CLI를 재현한다.
- 큰 변경 전 Help logs와 current worktree status를 확보한다.
- Plugin과 public link는 주기적으로 audit하고 불필요한 access를 revoke한다.

## See Also

- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
- [Orca 에이전트와 세션 운영](orca-agent-runtime-and-session-management.md)
- [Orca 원격 실행과 모바일 운영](orca-remote-and-mobile-runtime.md)
- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
