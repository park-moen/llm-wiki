# Claude Code Background Bash·Monitor와 Codex의 대응 기능

> Sources: Anthropic Claude Code Docs, Unknown; OpenAI ChatGPT Learn, Unknown
> Raw: [Background commands·Monitor](../../raw/ai-agents/claude-code-background-bash-monitor-tools-extract.md); [Interactive background Bash](../../raw/ai-agents/claude-code-interactive-background-bash-extract.md); [Codex background terminal commands](../../raw/ai-agents/codex-background-terminal-commands-extract.md); [Codex unified_exec](../../raw/ai-agents/codex-unified-exec-config-reference-extract.md)
> Updated: 2026-10-17

## Overview

Claude Code의 **Background Bash**는 명령을 계속 실행하면서 다른 작업을 할 수 있게 하고, **Monitor**는 실행 중 생기는 이벤트를 Claude에게 전달해 대화 중 반응하게 한다. Codex에도 **background terminal**과 재개 가능한 명령 실행이 있다. 다만 background 실행 지원과 Monitor 같은 이벤트 전달 보장은 구분해야 한다. 이 문서는 수집한 공식 문서에서 확인된 동작을 비교하며, 실제 설치 환경에서의 실행 검증 결과는 아니다.

## Background Bash: 명령을 실행해 두고 다른 작업 계속하기

개발 서버나 watch build처럼 오래 실행되는 명령에 적합하다. Claude가 Bash 호출에 `run_in_background: true`를 지정하면 명령이 비동기로 실행되고 background task ID가 반환된다. 사용자는 Claude에게 background 실행을 요청하거나, 실행 중인 일반 Bash 호출을 `Ctrl+B`로 전환할 수 있다. tmux에서는 prefix와 겹치므로 `Ctrl+B`를 두 번 누른다.

- `/tasks`에서 실행 중인 작업을 보고 중지한다.
- 출력은 파일에 기록되며 Claude가 `Read`로 읽는다.
- foreground 명령이 timeout에 도달하면 보통 background로 옮겨 실행을 이어간다. `sleep`으로 시작하는 명령은 예외다.
- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` 또는 bare mode에서는 background 기능이 꺼지므로 timeout에 명령이 중지된다.

기존 예제의 도구 이름은 주의해서 읽는다. 현재 수집본은 `TaskOutput`이 v2.1.277에서 제거되었으며 출력 파일을 `Read`로 읽는다고 설명한다. 중지는 `TaskStop`으로 할 수 있다.

### 얼마나 오래 유지되는가

명령을 시작한 주체와 실행 환경에 따라 수명이 달라진다.

| 상황 | 공식 문서의 동작 |
|---|---|
| main conversation 또는 background subagent가 시작한 명령 | 최종 응답 후에도 자체 종료·중지·시간 제한까지 실행 가능 |
| foreground subagent가 시작한 명령 | 해당 subagent의 실행이 끝나면 종료 |
| 사용자가 조작하는 local terminal·desktop app·VS Code 세션 | background 명령 자체의 시간 제한 없음 |
| unattended 실행: `-p`, Agent SDK, CI, cloud 등 | 기본 30분, background 시작 시 지정한 `timeout`으로 최대 2시간까지 설정 가능 |
| 단일 텍스트 prompt를 전달하는 `-p` 실행 | 기본 10분이며 `--input-format stream-json` 실행과 구분 |
| Claude Code 종료 | background task 정리 |

unattended 시간 제한은 v2.1.285 이상에 해당하며, v2.1.288 이전에는 모든 세션에 적용되었다. foreground에서 background로 옮겨진 명령은 이동 시점부터 제한을 센다. 환경 변수로 기본값과 최대값을 높일 수 있으므로 표는 기본 동작 기준이다.

따라서 최종 응답이 나왔다는 사실과 세션이 종료되었다는 사실을 구분해야 한다. background 실행을 세션 밖에서도 영구 유지되는 서비스로 해석하면 안 된다.

## Monitor: 변화를 이벤트로 받아 대화 중 반응하기

Monitor는 보통 작은 감시 script를 background로 실행하고 **stdout의 각 줄을 이벤트로 Claude에게 전달**한다. 사용자가 같은 세션에서 다른 작업을 이어가는 동안 Claude가 변화에 반응할 수 있다. 로그의 오류, 파일 변경, PR·CI 상태 변화, 장기 script 출력 감시에 적합하다.

Monitor가 항상 외부 서비스의 push를 받는 것은 아니다. script가 상태를 polling하고 변화가 생길 때만 출력할 수도 있다. 이미 WebSocket으로 이벤트를 제공하는 서버에는 `command` 대신 `ws` 입력으로 직접 연결할 수 있다.

### 기한과 사용 환경 제한

- 기본 감시 기한은 5분, 최대 30분이다. 단일 prompt를 받는 비대화형 `-p` 실행에서는 최대 10분이다.
- 기한이 끝나면 감시가 종료되고 Claude에게 알림이 전달되며, 필요하면 다시 시작할 수 있다.
- 취소를 요청하거나 세션을 종료하면 중지한다. Monitor를 시작한 subagent를 중지해도 함께 중지된다.
- script 실행에는 Bash의 `allow`·`deny` 규칙이 적용된다. WebSocket 연결은 별도의 승인을 거친다.
- Amazon Bedrock, Google Cloud의 Agent Platform, Microsoft Foundry에서는 사용할 수 없다.
- `DISABLE_TELEMETRY` 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`이 설정된 환경에서도 사용할 수 없다. Windows에서는 Git Bash가 필요하다.

Background Bash의 local 명령에 시간 제한이 없다는 설명을 Monitor에 적용하면 안 된다. Monitor에는 별도의 감시 기한이 있다.

## Codex에도 비슷한 기능이 있는가

**오래 실행되는 명령을 background terminal로 유지하는 기능은 있다.** 공식 설정 문서의 `features.unified_exec`는 재개 가능한 명령 실행이며 stable이고 모든 플랫폼에서 활성화된다고 설명한다. 과거 안내의 experimental 활성화 절차를 현재 필수 설정으로 그대로 적용할 근거는 없다.

다만 관리 정책이 재개 가능한 process와 `write_stdin`을 비활성화하면 일회성 `exec_command`만 남을 수 있다. 따라서 제품의 기능 지원과 자신의 환경에서 제공되는 실행 도구를 함께 확인해야 한다.

| 하고 싶은 일 | Claude Code | Codex 공식 문서에서 확인한 대응 |
|---|---|---|
| 장기 명령을 실행하면서 다른 작업 계속하기 | Background Bash | background terminal·`unified_exec` |
| 실행 상태와 최근 출력 확인 | `/tasks`, 출력 파일을 `Read`로 조회 | `/ps`: 명령과 최근 비어 있지 않은 출력 최대 세 줄 표시 |
| 실행 중인 작업 중지 | `/tasks` 또는 `TaskStop` | `/stop`: 현재 세션의 background terminal 전체 중지, `/clean`은 alias |
| 출력의 각 줄을 이벤트로 받아 대화 중 반응 | Monitor | 수집한 Codex 공식 문서에서는 같은 이벤트 전달 계약을 확인하지 못함 |
| WebSocket 메시지를 전용 감시 도구로 받기 | Monitor의 `ws` 입력 | 수집한 Codex 공식 문서에서는 직접 대응하는 기능을 확인하지 못함 |

`/ps` 목록은 `unified_exec`를 사용하는 경우에 나타나며 그렇지 않으면 비어 있을 수 있다. `/ps`의 최근 출력 미리보기를 전체 로그 streaming과 같다고 해석하지 않는다.

**해석의 한계:** 재개 가능한 명령 실행은 명령을 계속 실행하고 결과를 다시 확인할 수 있는 기반이다. 그것만으로 출력 한 줄마다 model이 자동으로 깨어나 반응하거나, idle 상태에서 완료 알림이 반드시 새 turn을 시작한다고 결론 내릴 수 없다. Codex에 Monitor와 동등한 기능이 없다고 단정하는 것도 이 수집 범위를 넘어선다.

## 요청할 때 목적을 나누기

다음은 도구 호출 문법이 아닌 요청 예시다.

- 장기 실행: “개발 서버를 background로 실행하고 다른 작업을 계속해 주세요.”
- 단일 조건 확인: “빌드가 끝나면 결과와 실패 로그를 확인해 주세요.”
- 반복 감시: “이 로그에서 새 오류가 생길 때마다 알려 주세요.”

Claude에서는 반복 감시와 장기 실행을 Monitor와 Background Bash의 역할에 맞춰 구분할 수 있다. Codex에서는 background 실행을 요청하되, 자동 이벤트 전달까지 필요하면 사용 중인 실행 환경이 그 동작을 제공하는지 별도로 확인한다.

## See Also

- [Claude Code Hooks](claude-code-hooks.md): 생명주기 이벤트에서 자동 실행하는 hook과 감시 도구의 역할 구분.
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md): model 외부 실행 환경이 도구 실행과 후속 처리를 담당하는 구조.
