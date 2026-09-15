# Claude Code Hooks

> Sources: Anthropic Claude Code Docs, Unknown
> Raw: [Hooks reference](../../raw/ai-agents/claude-code-hooks-reference.md); [Automate actions with hooks](../../raw/ai-agents/claude-code-hooks-guide.md)
> Updated: 2026-08-10

## Overview

Claude Code hook은 session, prompt, tool 호출, turn 종료 같은 생명주기 지점에서 자동 실행되는 handler다. Claude가 필요성을 판단해 호출하는 기능이 아니라 runtime이 event에 반응해 실행한다. 따라서 반복 작업 자동화, 명령 차단, 검사 결과 주입, 알림과 감사 기록처럼 정해진 시점에 자동 실행할 동작에 적합하다.

## Skill과 Hook의 차이

| 구분 | Skill | Hook |
|---|---|---|
| 핵심 역할 | Claude에게 추가 지침과 실행 절차를 제공 | 생명주기 event에 handler를 자동 연결 |
| 시작 주체 | 사용자 또는 Claude가 skill을 호출 | Claude Code runtime이 event 발생 시 실행 |
| 적합한 문제 | 방법론, domain 지식, 여러 단계 workflow | 자동 format, policy 검사, 알림, logging, 차단 |
| 보장 범위 | 지침을 읽고 적용하는 모델 행동에 의존 | matcher가 맞으면 handler 실행이 시도됨 |

둘은 경쟁 관계가 아니다. Skill이 작업 방법을 설명하고, hook이 중요한 경계에서 자동 검사하는 식으로 함께 쓸 수 있다. Skill이나 agent의 frontmatter 안에 hook을 선언해 해당 component가 활성화된 동안만 실행할 수도 있다.

## 동작을 이해하는 세 단계

Hook 설정은 다음 세 층으로 해석하면 된다.

1. **Hook event**: 언제 실행할지 정한다. 예: `PreToolUse`, `Stop`.
2. **Matcher group**: 어떤 대상에서 실행할지 좁힌다. 예: `Bash`, `Edit|Write`.
3. **Hook handler**: 실제로 무엇을 실행할지 정한다.

```text
event 발생
  → matcher 확인
    → handler 실행
      → exit code 또는 JSON 결과 해석
        → 계속, 차단, 입력 변경, context 추가
```

Matcher가 없는 event도 있다. 지원하지 않는 event에 matcher를 추가하면 무시될 수 있으므로 event별 reference를 확인해야 한다.

## 자주 쓰는 Event

| Event | 시점 | 대표 용도 | 주의점 |
|---|---|---|---|
| `SessionStart` | session 시작·재개 | 동적 context와 환경 준비 | 정적 지침은 `CLAUDE.md`가 더 단순함 |
| `UserPromptSubmit` | prompt 처리 전 | prompt 검사와 context 추가 | 차단하면 prompt 자체가 거부됨 |
| `PreToolUse` | tool 실행 전 | 위험 명령 차단, 입력 변경, 승인 결정 | 실제 실행 전 차단 가능한 핵심 지점 |
| `PermissionRequest` | 사용자 승인을 묻기 전 | 제한적인 자동 승인·거부 | matcher를 너무 넓히면 위험함 |
| `PostToolUse` | tool 성공 후 | format, lint, 결과 feedback | 이미 수행된 동작을 되돌릴 수 없음 |
| `PostToolUseFailure` | tool 실패 후 | 실패 분석과 추가 feedback | 실패 원인을 input JSON에서 확인 |
| `Stop` | Claude가 응답을 끝내려 할 때 | 완료 조건과 test 여부 확인 | 반복 차단에 대한 탈출 조건이 필요함 |
| `SessionEnd` | session 종료 | 정리, 통계, 상태 저장 | 종료 시간 예산을 고려해야 함 |

공식 reference에는 subagent, task, compaction, configuration, worktree, MCP elicitation 등 더 많은 event가 있다. 새 hook을 만들 때는 이름만 보고 추측하지 말고 해당 event의 input schema와 decision control 지원 여부를 확인한다.

## Handler 종류와 결정성

현재 handler는 `command`, `http`, `mcp_tool`, `prompt`, `agent` 형태를 지원한다.

- `command`: shell command나 executable을 실행한다. 기계적 검사를 구현할 때 가장 직접적이다.
- `http`: event JSON을 endpoint로 보내 중앙 감사·검증 service와 연결한다.
- `mcp_tool`: 이미 연결된 MCP server의 tool을 호출한다.
- `prompt`: 단일 model 평가로 yes/no 판단을 받는다.
- `agent`: file 검색이나 command 실행이 필요한 판단을 subagent에 맡긴다. 공식 문서상 experimental이다.

여기서 중요한 구분은 **hook이 자동으로 발화하는 것**과 **판정이 결정론적인 것**은 같지 않다는 점이다. `command`가 고정 규칙을 검사하면 결정적 gate가 될 수 있지만, `prompt`와 `agent` handler의 판단은 LLM에 의존한다. 따라서 “hook을 썼다”는 사실만으로 전체 policy가 결정적이라고 결론 내리면 안 된다.

## 설정 위치와 공유 범위

| 위치 | 범위 | 공유 여부 |
|---|---|---|
| `~/.claude/settings.json` | 사용자의 모든 project | machine local |
| `.claude/settings.json` | 해당 project | repository에 공유 가능 |
| `.claude/settings.local.json` | 해당 project | local 전용 |
| Managed policy settings | 조직 전체 | 관리자 통제 |
| plugin의 `hooks/hooks.json` | plugin 활성 기간 | plugin과 함께 배포 |
| skill·agent frontmatter | component 활성 기간 | component와 함께 배포 |

여러 settings level의 hook은 서로 대체되는 것이 아니라 합쳐진다. `/hooks`는 현재 적용된 hook과 출처를 확인하는 읽기 전용 browser다. 개별 hook만 비활성화하는 설정은 없으며, 전체 비활성화에는 `disableAllHooks`를 사용한다. Managed hook은 하위 설정에서 끌 수 없다.

## 입력과 출력

Command hook은 event 정보를 JSON으로 `stdin`에서 받고, exit code·`stdout`·`stderr`로 결과를 돌려준다. HTTP hook은 같은 JSON을 request body로 받고 response body로 결과를 돌려준다.

- exit `0`: 성공이다. `stdout`에 JSON이 있으면 구조화된 제어 정보로 해석한다.
- exit `2`: 해당 event가 차단을 지원하면 blocking error다. `stderr`의 이유가 Claude에 전달된다.
- 그 밖의 non-zero: 대부분의 event에서 오류를 기록하지만 실행은 계속된다.

Policy를 강제하려면 Unix 관습의 exit `1`이 아니라 Claude Code가 정의한 차단 방식이 무엇인지 확인해야 한다. 모든 event가 차단을 지원하는 것도 아니다. 구조화된 JSON을 사용할 때는 exit `0`이어야 하며 `stdout`에는 JSON object만 출력해야 한다.

`PreToolUse`는 `permissionDecision`으로 allow, deny, ask 같은 결정을 내리거나 `updatedInput`으로 tool 입력을 바꿀 수 있다. 반면 `PostToolUse`는 이미 실행된 작업을 취소하지 못하고 feedback만 줄 수 있다. 즉, 같은 검사라도 배치하는 event에 따라 강제력이 달라진다.

## 동기 Hook과 비동기 Hook

기본 hook은 handler가 끝날 때까지 Claude의 진행을 기다리게 한다. 오래 걸리는 test나 외부 작업은 command handler에 `async: true`를 둘 수 있지만, 비동기 hook은 이미 흐름이 계속된 뒤이므로 `decision`, `permissionDecision`, `continue`로 현재 행동을 차단하거나 제어할 수 없다.

따라서 강제 gate는 동기로 짧게 유지하고, 오래 걸리는 관측·알림은 비동기로 분리하는 편이 안전하다.

## Stop Hook과 무한 반복 방지

`Stop` hook이 종료를 막으면 Claude는 이유를 받고 계속 작업한다. 같은 조건이 해결되지 않으면 다시 종료를 시도하고 또 차단될 수 있다.

최신 공식 동작에는 두 가지 보호가 있다.

- input의 `stop_hook_active`는 이미 Stop hook 때문에 계속된 상태인지를 알려준다.
- Claude Code는 진행 없이 연속으로 여덟 번 차단되면 hook을 override하고 turn을 끝낸다.

이 보호는 최후의 안전장치이지 정책 설계를 대신하지 않는다. Hook은 `stop_hook_active`를 확인하고, 해결 가능한 조건과 명시적인 탈출 경로를 가져야 한다. 변경 집합 hash처럼 project 상태를 기록하는 개인 guard는 “동일 상태를 한 번만 차단”하는 더 세밀한 정책으로 추가할 수 있다.

## 보안과 실패 특성

Command hook은 현재 사용자의 전체 권한으로 실행된다. 따라서 신뢰하지 않는 repository의 project hook을 그대로 실행하거나, JSON 입력을 검증하지 않고 shell 문자열에 삽입하면 file 삭제·secret 노출·command injection 위험이 생긴다.

- 입력을 검증하고 shell quoting을 엄격히 한다.
- 가능한 경우 shell 해석을 피하는 exec form의 `args`를 사용한다.
- matcher와 조건을 가장 좁게 만든다.
- shared hook은 script를 code review하고 최소 권한으로 설계한다.
- `PostToolUse`로 사후 발견할 문제와 `PreToolUse`에서 사전 차단할 문제를 구분한다.
- `if` filtering은 parsing이 불가능한 Bash command에서 fail-open으로 handler를 실행할 수 있으므로, hard allow/deny 자체는 permission system을 우선 검토한다.

## 학습 순서

1. `Notification`이나 `PostToolUse` logging처럼 차단하지 않는 hook으로 event와 JSON input을 관찰한다.
2. `/hooks`와 `claude --debug`로 설정 출처, matcher, `stdout`/`stderr`를 확인한다.
3. `PreToolUse`에서 좁은 matcher를 사용해 한 가지 위험 동작을 차단해 본다.
4. exit code 방식과 exit `0` + JSON 방식을 각각 시험한다.
5. 마지막에 `Stop` 완료 gate를 추가하고 정상 통과, 한 번 차단, 반복 차단 경로를 모두 test한다.

처음부터 복잡한 TDD gate를 만드는 것보다 event별로 input을 기록하고 차단 가능 여부를 확인하는 작은 실험이 이해에 더 도움이 된다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
