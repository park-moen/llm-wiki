# Orca 원격 실행과 모바일 운영

> Sources: Stably, Inc., Unknown
> Raw: [Ways to run Orca](../../raw/orca/ways-to-run.md); [SSH worktrees](../../raw/orca/ssh.md); [Remote Orca Servers](../../raw/orca/remote-servers.md); [Mobile companion](../../raw/orca/mobile.md); [Work on a remote machine over SSH](../../raw/orca/recipes-remote-worktrees.md)
> Updated: 2026-08-12

## Overview

Orca의 실행 위치는 local desktop, SSH target, Remote Orca Server, per-workspace environment로 나뉜다. 이들은 단순 접속 방식이 아니라 project·file·worktree·terminal·agent process·credential의 소유 위치가 다르다. 원격 정착에서는 UI의 위치보다 runtime과 상태를 소유하는 host를 먼저 확인해야 한다.

## 네 가지 실행 방식

### Local desktop

UI, file, worktree, agent, browser가 같은 machine에서 실행된다. Setup이 가장 단순하고 짧은 작업과 빠른 iteration에 적합하다. Laptop 성능과 sleep에 직접 영향을 받는다.

### SSH target

Laptop의 Orca가 remote host에 연결하고 그곳에서 Git worktree와 agent를 실행한다. Editor, diff, orchestration UI는 laptop에 있지만 source file과 process는 remote에 있다. 이미 repo·toolchain·credential이 준비된 dev box, GPU host, VPS를 한 client에서 운용할 때 적합하다.

### Remote Orca Server

Remote machine에서 Orca desktop 또는 `orca serve` runtime을 계속 실행하고 laptop·web·mobile client가 그 runtime에 연결한다. Server가 project, worktree, terminal, tab, provider account, agent session을 소유한다. 여러 client와 automation이 같은 persistent runtime을 공유해야 할 때 적합하다.

### Per-workspace environment

Repository의 `orca.yaml`과 lifecycle script가 worktree마다 disposable VM, sandbox, container를 만들고 suspend·resume·destroy한다. Provider account, image, billing은 사용자가 소유한다. Task별 clean isolation과 재현 가능한 compute가 필요할 때 적합하다.

## SSH와 Remote Orca Server의 결정 기준

SSH에서는 laptop Orca가 runtime owner에 가깝고 remote host는 연결된 execution target이다. Remote Orca Server에서는 server의 Orca runtime이 상태의 중심이고 client는 UI다.

- 기존 dev box를 laptop 한 대에서 조작: **SSH**
- Laptop이 sleep해도 같은 server session을 mobile·web에서 이어감: **Remote Orca Server**
- Remote에 Orca runtime을 설치·관리하고 싶지 않음: **SSH**
- Automation과 여러 client가 server-owned state를 공유: **Remote Orca Server**
- Task마다 새 환경을 만들고 끝나면 폐기: **Per-workspace environment**

한 설치에서 mode를 섞을 수 있다. 짧은 edit는 local, GPU task는 SSH, 장기 agent는 server, 불신 코드 실행은 ephemeral recipe로 보내는 식이다.

## SSH target 설정과 동작

Settings의 SSH 영역에서 host, user, port, identity를 직접 넣거나 OpenSSH config에서 host를 가져온다. Proxy·jump host·multiplexing은 advanced connection에서 조정한다. macOS와 Linux에서는 connection reuse가 기본이므로 host 정책이 막을 때만 끄는 편이 낫다.

Worktree 생성 시 **Run on**에서 target을 고르면 remote에 실제 worktree가 만들어지고 agent와 terminal이 remote에서 실행된다. File event와 Git state는 Orca UI로 전달된다. Disconnect가 agent process를 자동 종료하지 않으며 reconnect 후 다시 attach한다.

Remote PTY는 relay lease를 통해 desktop app을 닫아도 유지될 수 있다. Reconnect grace가 지나거나 remote relay가 종료되는 상황과 app UI만 닫힌 상황은 구분해야 한다.

### Remote prerequisite와 제한

- Repo worktree에는 remote `git`이 필요하다.
- Agent CLI와 provider login은 remote host에 설치·인증되어야 한다.
- Linux remote terminal은 relay의 native module 때문에 build toolchain이 필요할 수 있다. Toolchain이 없더라도 file·Git·editor 연결은 성공하고 terminal만 실패할 수 있다.
- Kerberos/GSSAPI와 hardware-backed OpenSSH key는 system OpenSSH transport를 사용한다.
- Remote folder download는 recursive transfer capability가 있어야 하며 desktop client 전용이다.
- VS Code Remote-SSH handoff는 지원되는 VS Code launcher와 SSH worktree에 한정되고 Remote Orca Server active runtime에는 같은 방식으로 적용되지 않는다.

Right sidebar의 Ports 기능은 remote listening port를 local로 forward한다. Remote privileged port는 local에서 다른 port로 remap될 수 있으므로 실제 local endpoint를 UI에서 확인해야 한다.

## Remote Orca Server 구성

권장 경로는 server와 client 모두 Orca desktop 및 Tailscale을 사용해 private network에서 pair하는 것이다. Server의 **Advertise this app as a server**에서 reachable address를 골라 access link를 만들고 client의 **Add Server**에 붙인다. Headless host는 `orca serve`를 사용할 수 있다.

Pairing URL은 해당 runtime 접근 권한이므로 password처럼 다룬다. 공식 문서는 beta 기능으로 설명하며 public internet에 직접 노출하기보다 같은 tailnet이나 통제된 LAN을 권장한다.

Agent CLI, `git`, account, skill은 server computer에 있어야 한다. Laptop의 Claude·Codex login은 자동으로 server에 전달되지 않는다. Headless host에서는 `orca account add`와 `orca skills install` 같은 host-local CLI로 준비한다.

Client가 server를 추가했다고 모든 새 project가 자동으로 server로 향하는 것은 아니다. Active Server 설정과 worktree의 run target을 명시적으로 확인한다. 이 구분은 local credential과 server credential을 혼동하거나 의도하지 않은 host에 repository를 만드는 실수를 줄인다.

## Per-workspace environment recipe

Cloud VM 기능은 Orca가 hosting을 판매하는 것이 아니라 user-provided provider를 lifecycle script로 조정하는 기능이다. Recipe는 create·suspend·resume·destroy와 연결 정보 출력을 정의하고 Orca는 pairing URL 또는 SSH detail로 접속한다.

Workspace create 목록에 recipe가 보이려면 project의 primary checkout에 있는 `orca.yaml`에 `environmentRecipes`가 있어야 한다. Feature branch에서 recipe를 개발하고 doctor할 수는 있지만 create-time discovery는 primary checkout을 기준으로 한다.

이 mode는 agent가 실행할 code를 host와 분리하고, task별 dependency drift를 줄이며, 끝난 작업의 compute를 폐기하는 데 유리하다. 반면 provider billing, image maintenance, credential bootstrap, recipe failure recovery는 사용자가 책임진다.

## Mobile companion의 위치

Mobile은 독립 runtime이 아니라 paired host의 client다. Phone에서 agent 상태를 보고 terminal·Chat UI에 input을 보내고 workspace를 만들거나 source control·browser 상태를 확인한다. 실제 file, CLI, credential, process는 paired host에 남는다.

따라서 mobile 활용의 전제는 host가 awake·online이고 reachable하며, 같은 session을 소유하는 Orca runtime이 살아 있는 것이다. 장기 agent를 phone에서 관리하려면 laptop sleep에 의존하기보다 always-on Remote Orca Server가 더 자연스럽다.

Mobile에서 account switch와 usage를 보더라도 account는 host-local이다. 여러 host를 pair했다면 어느 host의 account·workspace를 보고 있는지 먼저 확인한다. Terminal setting이나 Quick Command도 client UI와 execution host의 소유 경계를 구분해야 한다.

## 보안과 운영 체크리스트

- Pairing link는 secret으로 취급하고 잘못 공유했으면 grant를 revoke한다.
- Server와 client는 private network path에 둔다.
- Agent·GitHub·cloud provider credential은 실제 runtime owner host에만 설치한다.
- SSH key passphrase의 보존 기간과 session 종료 시 clear 동작을 확인한다.
- Remote에서 public artifact나 browser profile을 쓸 때 local과 별도 storage임을 기억한다.
- Worktree를 만들기 전에 Create dialog의 **Run on**을 확인한다.
- Long-running process에는 reconnect·reboot·daemon 종료가 각각 어떤 영향을 주는지 구분한다.
- Ephemeral environment는 destroy script와 비용 회수 경로까지 검증한 뒤 확대한다.

## 권장 도입 순서

1. Local에서 worktree와 review loop를 먼저 익힌다.
2. 기존 dev box 하나를 SSH target으로 연결해 file·terminal·port forwarding을 검증한다.
3. Laptop sleep과 multi-client가 문제일 때만 Remote Orca Server를 추가한다.
4. 항상 켜진 server에 agent account와 skill을 host-local로 설치한다.
5. Isolation 요구가 명확한 repository에만 per-workspace recipe를 도입하고 lifecycle을 수동 검증한다.
6. Mobile은 runtime 대체가 아니라 attention·recovery client로 사용한다.

## See Also

- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
- [Orca 에이전트와 세션 운영](orca-agent-runtime-and-session-management.md)
- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
- [Orca 설정·개인정보·문제 해결](orca-operations-settings-privacy-troubleshooting.md)
